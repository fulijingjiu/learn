# 开源精选 · 数据库面试资料

> 给 jing Fu 自学用：从公开 GitHub 仓库**择优**拉取的 Markdown。  
> 本地已有笔记（`../02-MySQL专题/` 等）**不删不改内容**；这里只做补充阅读。  
> 拉取日期：2026-11-06（Asia/Shanghai）

---

## 怎么和本地笔记对照刷

1. 先按 [`../README.md`](../README.md) 把本地专题 + 全览过一遍（能口述）。  
2. 打开本目录 [`对照补缺.md`](./对照补缺.md)，按「本地缺口」逐条补：到开源文件里找对应标题，用自己的话记回本地专题（不要整段粘答案）。  
3. MySQL 深挖顺序建议：`csguide`（短）→ `yongxinz-mysql-interview` → `yongxinz-mvcc` / `index-b-plus` → `Qiu-Ye FAQ` → `sanfene 索引`（回原仓库看图文）→ `xoraus`（英文跨库）。  
4. PostgreSQL / SQL Server：本地笔记为主；跨方言细节看 `sqlserver-or-sql/xoraus-...`。

---

## 来源清单与许可证

| 源 | 落地文件 | 原路径 | 许可证 | 备注 |
|---|---|---|---|---|
| [csguide-dabai/interview-guide](https://github.com/csguide-dabai/interview-guide) | [`mysql/csguide-八股文骚套路之MySQL.md`](./mysql/csguide-八股文骚套路之MySQL.md) | `八股文骚套路之MySQL.md` | 未声明 SPDX | 全文约 9KB |
| [yongxinz/backend-interview](https://github.com/yongxinz/backend-interview) | [`mysql/yongxinz-mysql-interview.md`](./mysql/yongxinz-mysql-interview.md) 等 3 个 | `MySQL/*.md` | **MIT** | 每源最多 3 个文件 |
| [Qiu-Ye/InterviewFAQ-database](https://github.com/Qiu-Ye/InterviewFAQ-database) | [`mysql/Qiu-Ye-InterviewFAQ-database-README.md`](./mysql/Qiu-Ye-InterviewFAQ-database-README.md) | `README.md` | 未声明 SPDX | 小而全 FAQ |
| [xoraus/CrackingTheSQLInterview](https://github.com/xoraus/CrackingTheSQLInterview) | [`sqlserver-or-sql/xoraus-CrackingTheSQLInterview-README.md`](./sqlserver-or-sql/xoraus-CrackingTheSQLInterview-README.md) | `README.md` | **MIT** | 英文跨库指南 ~90KB |
| [itwanger/toBeBetterJavaer](https://github.com/itwanger/toBeBetterJavaer) | [`mysql/itwanger-sanfene-mysql-索引.md`](./mysql/itwanger-sanfene-mysql-索引.md) | `docs/src/sidebar/sanfene/mysql.md` | 未声明 SPDX | 原文件 ~308KB，**仅目录+文首节选** |

每个落地文件文首都有「来源说明」块（仓库、路径、许可证、原链接）。

---

## 目录结构

```
07-开源精选/
  README.md                 # 本文件
  对照补缺.md               # 开源有、本地缺的主题清单
  mysql/                    # MySQL 相关摘录
  postgresql/               # PG：目前仅为索引与缺口说明
  sqlserver-or-sql/         # SQL 跨方言 / SQL Server 相关
```

---

## 失败 / 未拉项

| 尝试 | 结果 |
|---|---|
| yongxinz 的 PostgreSQL 专篇 | **失败**：仓库无独立 PG Markdown（仅 README 偶提）；见 `postgresql/README-索引与缺口.md` |
| itwanger sanfene `mysql.md` 全文 | **刻意不全文**：>200KB，改为索引+节选；完整请点原链接 |
| csguide / Qiu-Ye / itwanger 许可证文件 | 仓库无标准 LICENSE 或 GitHub `license=null`，已在文首标注 |
| 整库 git clone | 未做：按原则只拉相关 md，避免乱拷 |

---

## 使用与版权提醒

- 开源摘录仅供个人学习；再分发请遵守各仓库许可证（MIT 可按条款使用；未声明许可证的请优先在线阅读原仓库）。  
- 不要把开源长文整段背进面试；本地笔记的「口述版」才是你的表达。
