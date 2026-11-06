# 数据库面试八股 · 目录

路径：`/workspace/面试八股/`

面向能写简单项目、正在刷面试的同学。本地笔记 + 开源精选对照看。

---

## 文件夹结构

```
面试八股/
├── README.md                 ← 你在这里
├── 01-总览/                  关系型 vs 非关系型，三库定位
├── 02-MySQL专题/             专题细讲 + 全览合集
├── 03-PostgreSQL/            PG 八股
├── 04-SQLServer/             SQL Server 八股
├── 05-对比/                  三库速查表
├── 06-实战/                  实现 / 分析 / 排障 / AI 辅助
└── 07-开源精选/              GitHub 择优摘录 + 对照补缺
```

---

## 建议学习顺序

1. `01-总览` → 建立地图  
2. `02-MySQL专题` → 先 5 个基础专题，再按全览进入 8 篇补强专篇  
3. `03-PostgreSQL`、`04-SQLServer` → 扩库  
4. `05-对比` → 临考翻  
5. `06-实战` → 边学边练  
6. `07-开源精选` → 对照补缺、扩展阅读  

---

## 各目录文件

### 01-总览
| 文件 | 说明 |
|---|---|
| `00-数据库总览-关系型对比.md` | 关系型/非关系型 + MySQL/PG/SQL Server 定位 |

### 02-MySQL专题
| 文件 | 说明 |
|---|---|
| `MySQL-存储引擎.md` | InnoDB vs MyISAM |
| `MySQL-事务隔离级别.md` | 四级隔离与脏读等 |
| `MySQL-B+树索引.md` | 为何用 B+ 树 |
| `MySQL-聚簇与非聚簇索引.md` | 聚簇、二级索引、回表 |
| `MySQL-最左前缀原则.md` | 联合索引用法 |
| [MySQL-面试八股全览.md](02-MySQL专题/MySQL-面试八股全览.md) | 高频合集 + 补强主题导航 |
| [MySQL-redo-binlog与两阶段提交.md](02-MySQL专题/MySQL-redo-binlog与两阶段提交.md) | 三类日志、WAL、prepare/commit 崩溃判断、双 1 与组提交 |
| [MySQL-一条SQL的执行过程.md](02-MySQL专题/MySQL-一条SQL的执行过程.md) | Server/引擎边界、查询与更新链路、执行计划和分段排障 |
| [MySQL-索引下推与BufferPool.md](02-MySQL专题/MySQL-索引下推与BufferPool.md) | ICP 与覆盖索引对比、页缓存与刷脏、命中率和容量评估 |
| [MySQL-深分页与分布式主键.md](02-MySQL专题/MySQL-深分页与分布式主键.md) | 延迟关联、复合游标、并发分页边界与自增/号段/UUID/Snowflake |
| [MySQL-死锁与select-for-update.md](02-MySQL专题/MySQL-死锁与select-for-update.md) | 锁范围、双会话死锁实验、监控证据、整笔重试与幂等 |
| [MySQL-IN-EXISTS与COUNT.md](02-MySQL专题/MySQL-IN-EXISTS与COUNT.md) | 半连接与 NULL、COUNT 语义、外连接和条件计数陷阱 |
| [MySQL-高可用读写分离与MHA.md](02-MySQL专题/MySQL-高可用读写分离与MHA.md) | 复制与读写路由、写后读、半同步、MHA 切换和旧主隔离 |
| [MySQL-类型与DDL注意点.md](02-MySQL专题/MySQL-类型与DDL注意点.md) | 字符/金额/时间类型、DELETE/TRUNCATE/DROP 与在线变更风险 |

### 03-PostgreSQL
| 文件 | 说明 |
|---|---|
| `PostgreSQL-面试八股.md` | 与 MySQL 差异 + 高频题 |

### 04-SQLServer
| 文件 | 说明 |
|---|---|
| `SQLServer-面试八股.md` | 锁、聚集索引、T-SQL、Always On 等 |

### 05-对比
| 文件 | 说明 |
|---|---|
| `三库对比速查.md` | 隔离/索引/锁/分页/选型对照表 |

### 06-实战
| 文件 | 说明 |
|---|---|
| `实战-开发实现技巧.md` | 建表、索引、事务、分页、防注入 |
| `实战-分析数据.md` | EXPLAIN、慢日志、核对 SQL |
| `实战-排查故障定位错误.md` | 连接/锁/延迟/磁盘等问题定位 |
| `实战-用AI辅助数据库工作.md` | 提问模板、人审、安全红线 |

### 07-开源精选
| 路径 | 说明 |
|---|---|
| `README.md` | 来源、许可证、刷法 |
| `对照补缺.md` | 开源有、本地缺的主题 |
| `mysql/` | csguide / yongxinz / Qiu-Ye / 面渣节选 |
| `postgresql/` | 缺口说明（暂无合适中文专篇） |
| `sqlserver-or-sql/` | CrackingTheSQLInterview（跨库） |

---

## 刷法

- 每篇先能口述 2～4 句，再看要点表。  
- 实战按清单在本地 Docker MySQL 练一遍。  
- 开源精选用来补缺，不替代本地笔记。  

## 本轮补强范围

已按 [开源对照补缺](07-开源精选/对照补缺.md) 补上前 10 项：日志/两阶段提交、SQL 链路、ICP、Buffer Pool、深分页、分布式主键、死锁、FOR UPDATE、IN/EXISTS/COUNT、高可用；次优先补类型与 DDL。原始开源摘录保持不动，技术细节另用官方文档核验，示例未连接真实数据库执行。
