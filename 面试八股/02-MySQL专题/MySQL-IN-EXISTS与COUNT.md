# MySQL：IN、EXISTS、NOT IN 与 COUNT 的语义和优化

> 范围：MySQL 8.0 / 8.4。先答“算什么”，再答“跑多快”。优化器会改写，不能用手写双层循环替代真实计划。

## 1. 概念：成员判断与存在判断

| 表达式 | 问的是什么 | 结果特点 |
|---|---|---|
| `x IN (子查询)` | x 是否等于候选中的某个值 | 涉及 NULL 时可能是 UNKNOWN |
| `EXISTS (子查询)` | 子查询是否产生至少一行 | TRUE / FALSE，不关心那行 SELECT 出来的具体值 |
| `x NOT IN (...)` | x 是否不等于所有候选 | NULL 陷阱容易让期望结果消失 |
| `NOT EXISTS (...)` | 是否没有满足相关条件的行 | 常用于反关联；仍要明确定义 NULL 业务含义 |

例如找“至少有一个已支付订单”的用户：

```sql
SELECT u.id
FROM users AS u
WHERE EXISTS (
  SELECT 1 FROM orders AS o
  WHERE o.user_id = u.id AND o.status = 'PAID'
);
```

也可以写 `u.id IN (SELECT user_id FROM orders WHERE status='PAID')`。这类存在性查询不会因为一个用户有 10 个订单就自动输出 10 次；直接 JOIN 却可能放大行数，改写时要保持语义。

## 2. 为什么“小表 IN，大表 EXISTS”不能死记？

这是老式执行模型下的经验口诀，现代 MySQL 可能使用半连接（semijoin）、物化、索引探测，部分 NOT 查询还可用反连接（antijoin）。写成相关子查询不代表真在应用层反复建连接，也不等于必然逐行扫描整张内表。

实际检查：

1. 先确认两种写法处理 NULL、重复值时是否等价。
2. 看真实筛选后的行数，不只看表总行数。
3. 看相关键索引，例如 `orders(user_id,status)` 可服务按用户探测；若从已支付订单集合起步，另一列顺序也可能更合适。
4. 用 `EXPLAIN`，在受控环境再用 `EXPLAIN ANALYZE` 对比扫描量、循环次数和耗时。
5. 不强行“全部改成 EXISTS”；清晰表达需求，再依据证据优化。

## 3. NOT IN 的 NULL 陷阱

```sql
SELECT 2 NOT IN (1, NULL);  -- 结果是 NULL（UNKNOWN），不是 TRUE
```

它相当于判断 `2<>1 AND 2<>NULL`，后者是 UNKNOWN。WHERE 只保留 TRUE，所以你可能一行都拿不到。

| 表达式 | 结果 |
|---|---|
| `1 IN (1,NULL)` | TRUE，已有一个确定匹配 |
| `2 IN (1,NULL)` | UNKNOWN，既没有匹配，又无法排除 NULL |
| `NULL IN (1,NULL)` | UNKNOWN，NULL 不按普通等值比较相等 |

所以不是“集合含 NULL，IN 永远为假”，更不是“NULL 能与 NULL 用等号匹配”。

找“没有订单”的用户，常写：

```sql
SELECT u.id
FROM users AS u
WHERE NOT EXISTS (
  SELECT 1 FROM orders AS o WHERE o.user_id = u.id
);
```

若用 `NOT IN`，至少要处理子查询的 NULL，例如 `WHERE user_id IS NOT NULL`。但**外层值本身为 NULL 时，两种写法仍可能不同**：相关等值比较不匹配，NOT EXISTS 可能保留该行；NOT IN 往往得到 UNKNOWN。要按业务加 `u.some_key IS NOT NULL` 或明确 NULL 安全比较语义，不能无脑替换。

`EXISTS(SELECT NULL ...)` 只要真的返回行也为 TRUE；SELECT 1 是清晰表达存在性，不是比 SELECT * 固有快几倍。

## 4. COUNT 统计的到底是什么？

假设某列数据是 `10、NULL、10`：

| 表达式 | 结果 | 解释 |
|---|---|---|
| `COUNT(*)` | 3 | 统计结果集行数，包括含 NULL 的行 |
| `COUNT(1)` | 3 | 常量 1 非 NULL，每行计一次 |
| `COUNT(col)` | 2 | 只统计该表达式非 NULL 的行 |
| `COUNT(DISTINCT col)` | 1 | 非 NULL 值去重后统计 |
| `COUNT(NULL)` | 0 | 每行表达式都是 NULL |

没有 GROUP BY 的空输入上，COUNT 返回一行 0；有 GROUP BY 时，没有任何分组就可能没有结果行。不要把这两种结果混为一谈。

## 5. 为什么 InnoDB 不能始终 O(1) 统计全表？

不同事务通过 MVCC 可能看到不同数量的可见行，所以不能只维护一个“对所有事务都精确”的全局行数直接返回。

- 无额外 WHERE/GROUP BY 的 `COUNT(*)`，InnoDB 通常遍历最小的可用二级索引；没有二级索引则扫描聚簇索引，仍需考虑可见性。
- 官方明确：InnoDB 对 `COUNT(*)` 和 `COUNT(1)` 处理相同，不存在“1 比星号必然快”的规律。
- 带 WHERE 时，索引是否支持筛选、扫描量多大通常比写 * 还是 1 更重要。
- `information_schema` / `SHOW TABLE STATUS` 中 InnoDB 行数可为估计，不能拿来算财务金额或精确分页总数。
- 预聚合表、缓存计数能换性能，但必须定义更新原子性、补偿、一致性与热点；不是随手加 Redis 就精确。

## 6. 实战常见坑

### A. LEFT JOIN 统计“没有订单”的用户

```sql
SELECT u.id, COUNT(o.id) AS order_count
FROM users AS u
LEFT JOIN orders AS o ON o.user_id = u.id
GROUP BY u.id;
```

无订单时仍有一条右侧补 NULL 的连接结果，`COUNT(*)` 会算成 1；`COUNT(o.id)` 在右侧 id 为非空主键时才代表实际匹配订单数。若要只统计已支付订单，把条件放 ON 内，随便移到 WHERE 可能把外连接变成内连接效果。

### B. 条件计数不能写错

```sql
COUNT(CASE WHEN status = 'PAID' THEN 1 END)
-- 或
SUM(CASE WHEN status = 'PAID' THEN 1 ELSE 0 END)
```

`COUNT(status='PAID')` 会连 FALSE=0 也计入，只要表达式非 NULL；它不是“只数真”。SUM 对空输入可能返回 NULL，需要时 `COALESCE(...,0)`。

### C. 只要知道有没有，不必数完

```sql
SELECT EXISTS(SELECT 1 FROM orders WHERE user_id = 42);
```

表达存在性即可。若用于“检查用户名没占用再插入”，仍有并发竞态，必须由唯一约束兜底；EXISTS 不是锁。

## 7. 面试口述版

> IN 判断值是否属于集合，EXISTS 判断子查询有没有行；性能要看索引和优化器改写，不能只背小表大表口诀。NOT IN 遇到 NULL 可能变成 UNKNOWN，所以反关联常用 NOT EXISTS，但也要明确外层 NULL 的含义。COUNT(*) 和 COUNT(1) 在 InnoDB 中没有固有性能差别，COUNT(列) 不统计 NULL。InnoDB 的精确计数受 MVCC 可见性影响，通常还要扫描记录。

## 延伸阅读

- 开源：[yongxinz Q19](../07-开源精选/mysql/yongxinz-mysql-interview.md)；循环类比仅帮助入门，不代表现代优化器执行保证。
- 已读开源节选：[面渣 MySQL：7、12 节](../07-开源精选/mysql/itwanger-sanfene-mysql-索引.md)。
- 官方核验：[COUNT 语义与 InnoDB 优化](https://dev.mysql.com/doc/refman/8.4/en/aggregate-functions.html#function_count)、[Semijoin transformations](https://dev.mysql.com/doc/refman/8.4/en/semijoins-antijoins.html)。
