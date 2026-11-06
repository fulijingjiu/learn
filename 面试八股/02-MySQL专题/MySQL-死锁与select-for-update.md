# MySQL：死锁、锁等待与 SELECT FOR UPDATE

> 范围：MySQL 8.0 / 8.4、InnoDB；未特别说明时讨论 RR。加锁范围跟实际访问索引、查询条件、隔离级别有关。以下双会话实验只在独立练习库手动进行，本文没有执行任何 SQL。

## 1. 概念：等待不一定是死锁

| 情形 | 举例 | 结果 |
|---|---|---|
| 普通锁等待 | B 想写 A 已锁住的 id=1，A 不在等 B | A 提交后 B 可继续；等太久可能超时 |
| 死锁 | A 占 id=1 等 id=2，B 占 id=2 等 id=1 | 等待成环，需打破环 |
| 元数据锁（MDL）等待 | 长事务访问过表，DDL 等待改结构 | 不等同于 InnoDB 行锁等待，需看 metadata_locks 等证据 |

InnoDB 通常启用死锁检测，发现环后选择牺牲事务回滚，释放锁让其他事务继续。被回滚的不一定是“最后来的那笔”或“业务最不重要的一笔”。只调大超时不能修复死锁环。

## 2. FOR UPDATE 是什么，为什么需要它？

它是**锁定读 / 当前读**：读取可用于当前修改的记录版本，对访问到的相应索引记录加排他锁；发生并发未提交写入时可能等待。它不是按普通 SELECT 的旧快照读完就算。

```sql
START TRANSACTION;
SELECT available FROM stock WHERE sku_id = 1001 FOR UPDATE;
-- 应用判断库存，并在同一个事务、同一个连接里执行后续修改
UPDATE stock SET available = available - 1
WHERE sku_id = 1001 AND available > 0;
COMMIT;
```

事务显式开始后，锁通常持有到提交/回滚。`autocommit=1` 下单条 SELECT 结束就不能指望保护之后另一次业务操作；连接池拿了另一个连接也不是原事务。

如果只是“库存足够则减一”，直接使用上面的**条件 UPDATE**并检查影响行数通常更简洁。只有业务确实要在锁保护下读取再做复杂判断时，才增加锁定读；不要为所有查询加 `FOR UPDATE`。

## 3. 会锁行还是锁表？别背错误二分法

InnoDB 记录锁加在**索引记录**上；没有显式主键也有聚簇组织方式。典型情况：

| 访问情形 | RR 下的常见表现 | 需要补的限定 |
|---|---|---|
| 完整唯一键等值，且命中记录 | 记录锁，不需要锁前面的间隙 | 多列唯一键必须用到完整唯一条件 |
| 唯一键等值，但记录不存在 | 可能锁住该值所在间隙 | “没查到所以没锁”不成立 |
| 非唯一索引等值 / 范围扫描 | 常见 next-key / gap 锁，范围可能超出最终结果行 | 边界、索引记录分布与执行计划影响锁范围 |
| 无合适索引而全表扫描 | 可能锁住大量扫描的聚簇记录及相关间隙 | **不是自动把行锁升级成一把表锁** |
| 走二级索引的排他锁访问 | 还可能锁对应聚簇索引记录 | 只看二级索引会漏锁 |

RC 下通常不对普通搜索/索引扫描采用 RR 那样的 gap locking，但外键检查、重复键检查等仍有例外，也仍可能死锁。切 RC 不能代替正确事务设计。

`FOR UPDATE` 不会让所有普通 SELECT 都被阻塞：RC/RR 的一致性读通常通过 MVCC 读可见版本，但 DDL、其他锁定读或写可能受阻。

## 4. 两个会话复现最简单的死锁

先在练习库准备 `lock_demo(id INT PRIMARY KEY, value INT)`，包含 id=1、2 两行。按表格顺序交错执行：

| 步骤 | 会话 A | 会话 B |
|---|---|---|
| 1 | `START TRANSACTION;` | `START TRANSACTION;` |
| 2 | `UPDATE lock_demo SET value=value+1 WHERE id=1;` | |
| 3 | | `UPDATE lock_demo SET value=value+1 WHERE id=2;` |
| 4 | `UPDATE lock_demo SET value=value+1 WHERE id=2;`（等待） | |
| 5 | | `UPDATE lock_demo SET value=value+1 WHERE id=1;`（形成环） |
| 6 | 检查哪边收到 1213；结束剩余事务 | 检查结果，显式清理事务 |

预期：检测开启时，其中一笔事务报 deadlock，另一笔可继续。不要在生产为了“看日志”制造死锁，也不要让实验结束后还有事务未关闭。

## 5. 排障：拿证据 → 画等待图 → 修复

只读检查示例（需要相应监控权限）：

```sql
SHOW ENGINE INNODB STATUS;
SELECT * FROM performance_schema.data_lock_waits;
SELECT ENGINE_TRANSACTION_ID, OBJECT_SCHEMA, OBJECT_NAME,
       INDEX_NAME, LOCK_TYPE, LOCK_MODE, LOCK_STATUS, LOCK_DATA
FROM performance_schema.data_locks;
SELECT trx_id, trx_started, trx_state, trx_mysql_thread_id, trx_query
FROM information_schema.innodb_trx;
```

1. `SHOW ENGINE ...` 找 `LATEST DETECTED DEADLOCK`，它通常只保留最近一次，别把旧死锁当当前原因。
2. 从事务、SQL、索引名、已持有锁、正在等的锁，画出 A→B→A；同一事务之前执行过的 SQL 也要找。
3. 结合 `SHOW CREATE TABLE`、索引和计划解释“为什么碰到这些记录”；只看最终 WHERE 条件不够。
4. 若卡的是 DDL，查看 `performance_schema.metadata_locks`、进程状态和长事务，不要只查 data_locks。
5. 需要持续记录所有死锁时，由负责人评估 `innodb_print_all_deadlocks` 等设置；本文不直接修改全局参数或终止连接。

MySQL 8 的主要观测入口与老资料的 `INFORMATION_SCHEMA.INNODB_LOCKS / INNODB_LOCK_WAITS` 不同，照抄老命令可能找不到表。

## 6. 应用怎么兜底？

| 错误 | 通常含义 | 应用策略 |
|---|---|---|
| 1213 / SQLSTATE 40001 | 检测到死锁，牺牲事务被回滚 | 在新事务重试整个业务单元，次数有限、有退避 |
| 1205 | 等锁超时 | 默认通常回滚当前语句，不一定回滚整个事务；显式回滚后决定是否整笔重试 |

- 多资源按稳定顺序获取，例如所有转账都先处理较小账户 ID，再处理较大 ID。
- 事务短小；减少扫描范围，补合适索引；批量任务控制单批规模。
- 重试前确保幂等：唯一业务请求号、状态迁移条件、事务外消息/支付等副作用单独处理。
- “出错后原事务继续执行剩下半段”很危险；不要吞异常后假装成功。
- `NOWAIT` 可让冲突立即失败，`SKIP LOCKED` 可跳过已锁记录，常用于队列抢任务；会改变结果语义，不适合账务完整性查询，且不是免死锁开关。

## 7. 面试口述版

> 死锁是事务等待成环，不是普通的排队；InnoDB 检测后会回滚其中一笔。我先看最近死锁日志和 data_locks/data_lock_waits，结合事务之前的 SQL 与索引画等待图。FOR UPDATE 是当前读，锁范围取决于访问路径和隔离级别，没有索引可能锁大量记录，但不是自动升级表锁。修复靠统一加锁顺序、短事务、合适索引，再对幂等业务做有限整笔重试。

## 延伸阅读

- 开源：[yongxinz Q2、Q12、Q15](../07-开源精选/mysql/yongxinz-mysql-interview.md)、[Qiu-Ye 锁](../07-开源精选/mysql/Qiu-Ye-InterviewFAQ-database-README.md)。原文“没索引就表锁”是简化说法，本篇已修正。
- 官方核验：[不同语句的加锁范围](https://dev.mysql.com/doc/refman/8.4/en/innodb-locks-set.html)、[Deadlock handling](https://dev.mysql.com/doc/refman/8.4/en/innodb-deadlocks-handling.html)。
