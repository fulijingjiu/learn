# PostgreSQL 相关开源摘录索引

## 说明

`yongxinz/backend-interview` **没有**独立的 PostgreSQL Markdown 目录（仅 MySQL / Redis / MongoDB 等），因此本目录不伪造 PG 专题全文。

跨库对比请主要看：

- `../sqlserver-or-sql/xoraus-CrackingTheSQLInterview-README.md`（MIT，英文，多处标注 MySQL / PostgreSQL / SQL Server 差异）
- 本地笔记：`../../PostgreSQL-面试八股.md`、`../../三库对比速查.md`

## 从 xoraus 指南里优先扫的 PG 相关点（主题清单，非答案抄录）

- BOOLEAN 真类型（对比 MySQL TINYINT）
- FULL OUTER JOIN / INTERSECT / EXCEPT
- `JSONB` 与索引
- 分页与日期函数方言
- CTE / Window Functions（PG 与 MySQL 8+ / SQL Server 都强）
- 隔离级别默认多为 Read Committed（对照本地笔记）

## 失败/缺口记录

- 未能从 yongxinz 拉到 PostgreSQL 专篇（仓库无对应文件）
- 未另找第三方中文 PG 八股整库拷贝，避免超范围乱拷
