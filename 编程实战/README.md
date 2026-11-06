# 编程实战学习资料

路径：`/workspace/编程实战/`

Java / Python / 前端 **并存**，互不覆盖。

## 目录

| 目录 | 内容 |
|---|---|
| `Java/` | 基础/框架/实战/Debug/片段 + MyBatis 缓存与占位符 |
| `Python/` | 基础/框架/实战/Debug/片段 + SQLAlchemy 会话/事务 |
| `前端/` | React / Node.js / 组件库 / Storybook / 实战 / Debug / 模式，01～07 均加厚 |
| `运维部署/` | Linux / Docker / K8s / Rancher / Nginx / 跳板机·FinalShell / 排障 |
| [Agent/](Agent/README.md) | 概念/框架/工具、MCP 协议、checkpoint 恢复与评测 |
| `开源精选/` | GitHub 择优 README / 章节（java / python / frontend / devops / agent） |

## 前端入口（本次新增）

从这里开始：[`前端/README.md`](./前端/README.md)

建议顺序：React → Node → 组件库 → Storybook → 实战 → Debug → 代码片段；对照 `开源精选/frontend/`。

## 开源精选

见 [`开源精选/README.md`](./开源精选/README.md)。

## 相关

- 数据库八股：`/workspace/面试八股/`

## 运维部署

路径：[`运维部署/`](./运维部署/)

覆盖：Linux、跳板机/FinalShell、Docker/Dockerfile、K8s/Rancher、Nginx、部署与排障。

开源摘录：[`开源精选/devops/`](./开源精选/devops/)

建议顺序：`运维部署/README.md` 内目录 → 对照 devops 开源示例。

## Agent（本次追加）

路径：[`Agent/`](./Agent/)

覆盖：Agent 概念与架构、常用框架对比、实战技巧、Debug、代码模式。

开源摘录：[`开源精选/agent/`](./开源精选/agent/)

建议顺序：`Agent/README.md` → 01～06 → 对照开源 README 与 `对照补缺.md`。


## 对照补缺后的详细专题

- [Java：MyBatis 缓存与占位符](Java/06-MyBatis缓存与占位符.md)：会话/事务边界、跨 Mapper 缓存一致性、#{} 与 ${}。
- [Python：SQLAlchemy 会话与生命周期](Python/06-SQLAlchemy会话与生命周期.md)：FastAPI 请求资源、flush/commit、异步并发和对象加载。
- [前端 01～07](前端/README.md)：补机制、常见坑、口述、使用前提与延伸阅读。
- [Docker](运维部署/02-Docker与Dockerfile.md) / [K8s](运维部署/03-K8s与Rancher.md)：补安全配置、探针/资源与分层排障。
- [Agent：MCP](Agent/06-MCP与工具协议.md) / [Debug 与评测](Agent/04-Debug与排障.md)：补协议、审批、幂等、恢复与最小测试模板。

原开源摘录保留不变；机制细节结合官方文档核验，所有代码/命令为学习示例，未对真实服务执行。
