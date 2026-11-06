# MySQL：索引下推（ICP）与 Buffer Pool

> 范围：MySQL 8.0 / 8.4、InnoDB。两件事都与 IO 有关：ICP 尽量少读整行，Buffer Pool 尽量让页读取命中内存；不要混为查询结果缓存。

## 1. 概念：扫描、过滤、回表是三步

假设表 `customers` 有主键 `id`、联合二级索引 `(tenant_id, name)`，还有未在该索引中的 `address`：

```sql
SELECT id, name, address
FROM customers
WHERE tenant_id = 42 AND name LIKE '%明%';
```

- `tenant_id=42` 能定位联合索引中的一段范围。
- `name LIKE '%明%'` 不能靠 name 的有序性直接定位更小的连续区间，但 name 的值**在索引记录中**，仍可以拿来判断。
- `address` 不在该二级索引中；最终留下的候选需要拿主键回聚簇索引读取整行，即回表。

## 2. 为什么 ICP 能省 IO？

| 路径 | 大致过程 | 不满足 name 条件的候选 |
|---|---|---|
| 无 ICP | 读二级索引 → 回表取行 → Server 判断剩余条件 | 也可能先取整行再丢弃 |
| 有 ICP | 读二级索引 → 引擎先判断可下推的索引条件 → 通过才回表 | 提前丢弃，减少取整行 |
| 覆盖索引 | 查询所需列和条件都能由索引满足 | 通常无需为输出字段回表 |

若某个 tenant 有 1 万条索引候选，其中只有 300 条匹配 name，ICP 有机会把整行读取从大量候选缩小到通过者；这只是示意，**不是实测结果**。它可能仍扫描那 1 万条索引记录，并没有神奇地把模糊查询变成精确定位。

## 3. EXPLAIN 怎么识别？

- `Using index condition`：常见 ICP 标记，通常仍需要读取通过过滤的整行。
- `Using index`：常见覆盖索引标记，不是“任何使用索引的查询”都会出现。
- `Using where`：仍有条件过滤，不能仅凭它判好坏。
- 看 `key`、访问类型、估计/实际扫描量一起判断；不要只截一个 Extra 值。

InnoDB 的 ICP 针对需要读取完整行的二级索引访问，聚簇索引记录本身已有整行，不靠它减少整行 IO。包含子查询、存储函数等的条件不一定可下推；有无 ICP 取决于访问方式、条件及版本，不能说“联合索引一定开”。

在**同一个测试会话**中可切换 `optimizer_switch` 的 `index_condition_pushdown` 标志对比，完成后恢复；生产不要为演示修改全局优化器配置。

## 4. Buffer Pool 是什么？

它是 InnoDB 缓存数据页、索引页等内容的主要内存区域。典型页大小为 16 KiB，但可随实例配置变化，不能把“页”当成“行”。

```text
索引定位要访问某页
  ├─ 命中 Buffer Pool → 在内存读取，仍要做条件/可见性判断
  └─ 未命中 → 读磁盘页放入池 → 再读取
写入 → 改内存页 → 脏页 → 后台或压力驱动刷盘
```

| 容易混淆的组件 | 缓存/保存什么 | 生命周期与作用 |
|---|---|---|
| Buffer Pool | 数据页、索引页等 | 减少物理读，承接脏页 |
| redo log buffer | redo 记录 | 日志写入前的内存缓冲，不是数据页池 |
| OS page cache | 操作系统文件缓存 | 是否参与具体数据文件 IO 与 flush 配置有关 |
| 应用缓存 / Redis | 应用定义的键值或查询结果 | 要额外处理一致性与过期 |
| 旧 Query Cache | SQL 结果 | MySQL 8.0 已移除 |

## 5. 页怎么淘汰、脏页怎么处理？

- **Free list** 管理可用页框；空闲页不足时要找可淘汰页。
- **LRU** 管理冷热页。InnoDB 使用带 old/new 区域的改进策略，降低一次大扫描把热点全挤掉的风险，不是简单“每读一次都塞到最前”。
- **Flush list** 帮助组织脏页刷盘。脏页不能直接扔掉，刷盘前必须满足 WAL。
- **Checkpoint** 推进恢复起点并使更老的 redo 空间可复用。刷脏跟提交不是一对一：已提交页可以没刷，未提交修改也可能已刷，恢复由日志和事务状态保障。
- 扫描巨大冷表、热点工作集超过内存、磁盘跟不上刷脏，都可能导致延迟。仅扩大 Buffer Pool 不能解决锁等待或错误 SQL。

老资料中的 AHI、change buffer 默认开启、固定内存结构是有版本边界的；例如 8.4 的相关默认值不同，不要背成所有版本都一样。

## 6. 怎么量化，而不是拍脑袋加内存？

只读命令：

```sql
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool%';
SHOW GLOBAL STATUS LIKE 'Innodb_log_waits';
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SHOW ENGINE INNODB STATUS;
```

在同一负载窗口取两次累计值，粗略页命中率：

```text
1 - ΔInnodb_buffer_pool_reads / ΔInnodb_buffer_pool_read_requests
```

分母为 0 时不计算，实例重启导致计数重置也要排除。这个比率只是观察信号，不是 SQL 延迟的充分解释；还要看物理读取次数、磁盘时延、`wait_free` 增量、脏页和查询扫描量。高命中率下仍可能因为扫描数百万内存记录而 CPU 高。

设置容量要给操作系统、连接内存、排序缓冲及同机服务留空间。专用数据库服务器常见较大内存占比的经验，不适合直接套到 2 GiB 容器；要按容器实际限制核算，避免 OOM。

## 7. 面试口述版

> ICP 是把能用索引列判断的剩余条件放到存储引擎，先过滤再回表，减少整行读取；它和覆盖索引不同，不代表完全不回表。Buffer Pool 缓存页，读命中后少做磁盘 IO，写入先改内存脏页并用 redo 保护。排查时我看计划与扫描量，再看一段时间的页命中率、物理读和刷脏压力，而不是只把内存调大。

## 8. 易错点

- 范围列后面的字段可能不能继续缩小定位范围，但仍可能参与 ICP、覆盖或过滤；“完全不能用”太绝对。
- 回表不等于每次必有一次磁盘 IO：聚簇页可能已在缓存中，但多次访问仍有 CPU/并发成本。
- Buffer Pool 重启预热记录的是待加载页的标识，不是替代 redo 的持久化数据备份。

## 延伸阅读

- 开源：[yongxinz Q27](../07-开源精选/mysql/yongxinz-mysql-interview.md)、[Qiu-Ye InnoDB 内存池与 checkpoint](../07-开源精选/mysql/Qiu-Ye-InterviewFAQ-database-README.md)。
- 主题入口：[面渣 MySQL 目录：27、51](../07-开源精选/mysql/itwanger-sanfene-mysql-索引.md)（仅目录/节选）。
- 官方核验：[ICP 适用条件](https://dev.mysql.com/doc/refman/8.4/en/index-condition-pushdown-optimization.html)、[Buffer Pool](https://dev.mysql.com/doc/refman/8.4/en/innodb-buffer-pool.html)。
