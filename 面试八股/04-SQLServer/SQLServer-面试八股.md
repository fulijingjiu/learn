# SQL Server 面试八股

> 学习笔记 · 面试八股  
> 重点：和 MySQL / PostgreSQL 的差异 + 高频口述。  
> 对照：[`00-数据库总览-关系型对比.md`](./00-数据库总览-关系型对比.md) · [`三库对比速查.md`](./三库对比速查.md)

---

## 1. 一句话定位

**SQL Server** 是微软的关系型数据库（商业为主，有 Express 等免费版）。和 **Windows / .NET / Azure** 生态绑定深，企业内部系统、ERP、.NET 项目里很常见。默认隔离级别一般是 **读已提交 Read Committed（RC）**。

### 面试口述版

> SQL Server 是微软栈的默认库，.NET 和 Windows 运维多。默认 RC。和开源的 MySQL/PG 比，授权与生态是主要差异。

---

## 2. 与 MySQL / PG 差异总表（常见说法）

| 维度 | SQL Server | MySQL InnoDB | PostgreSQL |
|---|---|---|---|
| 许可 | 商业为主 | 开源 | 开源 |
| 默认隔离 | RC | RR | RC |
| 表组织 | 常强调 **聚集索引表**；也可堆表 | 主键聚簇 | 堆表 + 索引 |
| 语言方言 | **T-SQL** | SQL + MySQL 方言 | SQL + PG 方言 |
| 自增 | **IDENTITY** | AUTO_INCREMENT | SEQUENCE / IDENTITY |
| 高可用概念 | **Always On**（AG 等） | 主从 / Group Replication 等 | 流复制 / 各类方案 |
| 工具链 | SSMS、SSIS、SSRS、Agent 作业 | 生态分散（各云控制台、中间件） | psql、扩展生态 |

---

## 3. 锁与隔离（常考）

- 默认 **RC**：一般避免脏读；不可重复读、幻读在默认下仍可能出现（概念同其它库）。
- 可调到 RR、Serializable；还有 **快照隔离 Snapshot Isolation**、**读已提交快照 Read Committed Snapshot（RCSI）** 等选项（靠版本存储减锁冲突——常见说法）。
- 锁粒度：行、页、表等；有锁升级（大量行锁可能升到表锁——常见说法）。
- 提示（Hint）：如 `NOLOCK`（脏读风险，面试要能说出代价）等——生产慎用。

### 面试口述版

> SQL Server 默认读已提交。可以用快照类隔离减轻读写阻塞。`NOLOCK` 能减少等待但可能脏读，面试里要说明风险，不能当银弹。

---

## 4. 聚集索引表（Clustered Index）

- **聚集索引 Clustered index**：决定数据行的物理排序（口语：叶子就是数据）。
- 一张表通常 **一个** 聚集索引；常建在主键上。
- 没有聚集索引时是 **堆表 Heap**。
- **非聚集索引 Nonclustered**：叶子指向行定位符（有聚集索引时通常是聚集键）。

这点和 InnoDB「主键即聚簇」很像，术语用微软的 Clustered / Nonclustered。

### 面试口述版

> SQL Server 里聚集索引决定行怎么排，一张表一般一个。主键常常就是聚集索引。非聚集索引找到键以后还要回表拿其它列，优化思路和 MySQL 覆盖索引类似。

---

## 5. 执行计划（Execution Plan）

- 看估计/实际计划：图形化在 SSMS；也有 SET STATISTICS / 计划 XML 等。
- 关注：是否 Index Seek / Scan、预估行数准不准、有没有昂贵的排序/哈希连接。
- 统计信息（Statistics）过期会导致计划变差——运维常考点。

### 面试口述版

> 慢查询我会看执行计划，看有没有该走的索引 Seek，以及统计信息是否过期。这和 MySQL 的 EXPLAIN 是同一类事。

---

## 6. 备份与恢复（概念级）

| 类型 | 英文 | 口语 |
|---|---|---|
| 完整备份 | Full backup | 整库一份 |
| 差异备份 | Differential | 相对上次完整备份的变化 |
| 日志备份 | Log backup | 事务日志；配合完整恢复模式做时间点恢复 |

恢复模式（常见说法）：简单 / 完整 / 大容量日志——影响日志备份与能否 point-in-time restore。

### 面试口述版

> 完整 + 差异 + 日志备份是常规组合。要按时间点恢复，一般要用完整恢复模式并持续做日志备份。

---

## 7. Always On（概念级）

- **Always On** 是微软高可用/灾备品牌叙事；面试常提 **Availability Group（AG，可用性组）**：多副本、可同步/异步、可读从副本等（视配置）。
- 说到「自动故障转移、读写分离读副本」即可，不必背安装步骤。

### 面试口述版

> Always On 可用性组用来做高可用和可读副本，类似「主从 + 故障转移」这一层，只是 SQL Server 体系里的标准说法。

---

## 8. T-SQL 特点（面试点到）

- 变量：`DECLARE @id INT`
- 流程：`IF` / `WHILE`；临时表 `#temp`、表变量 `@t`
- 常见分页（较新写法）：`OFFSET ... FETCH NEXT ...`；老项目常见 `ROW_NUMBER()` 窗口
- `MERGE`、`OUTPUT` 子句等（知道存在即可）
- 与标准 SQL 大体兼容，细节和函数名与 MySQL/PG 不同（如字符串、日期函数）

### 面试口述版

> T-SQL 是 SQL Server 的方言，变量、临时表、分页语法和 MySQL 不太一样。分页现在常用 OFFSET FETCH，老代码可能是 ROW_NUMBER。

---

## 9. IDENTITY（身份标识）

- **IDENTITY(seed, increment)**：自增列常见写法。
- 类似 MySQL 的 AUTO_INCREMENT；注意：回滚/失败也可能造成**号洞**（常见说法）。
- 需要显式插入时用 `SET IDENTITY_INSERT`（有权限与场景限制）。

### 面试口述版

> SQL Server 自增用 IDENTITY，事务回滚也可能跳号，别当连续业务单号唯一方案。

---

## 10. 常见运维 / 面试点（短列表）

- 作业调度：**SQL Server Agent**
- 权限：登录 Login vs 数据库用户 User；角色
- 索引维护：碎片、重建/重组（常见运维话题）
- 阻塞与死锁：看等待、死锁图（面试能说到「查谁锁谁」即可）
- 云上：Azure SQL 等托管形态（知道与本地 SQL Server 有亲缘关系）

### 面试口述版

> 运维上常问备份策略、Agent 作业、索引碎片和死锁排查。企业环境还会问权限模型和 Always On。

---

## 11. 高频问答速背

**Q：为什么公司用 SQL Server？**  
> 已有微软授权与 .NET 团队；要和 Windows、报表、集成服务绑在一起。

**Q：和 MySQL 默认隔离差别？**  
> SQL Server 默认 RC，MySQL InnoDB 默认 RR；幻读/不可重复读的默认表现不一样。

**Q：聚集 vs 非聚集？**  
> 聚集决定数据排序且通常一个；非聚集另开索引结构，可能回表。

---

## 12. 复习挂钩

- 隔离三个坏现象：[`MySQL-事务隔离级别.md`](./MySQL-事务隔离级别.md)（概念通用）  
- 聚簇思想对照：[`MySQL-聚簇与非聚簇索引.md`](./MySQL-聚簇与非聚簇索引.md)  
- 三库表：[`三库对比速查.md`](./三库对比速查.md)
