# 开源精选（Python / Java）

拉取时间：以文件 mtime 为准（云电脑 Asia/Shanghai）。仅供学习对照，遵守各仓库许可证；未声明 LICENSE 的仓库**不要再分发**，只本地阅读。

## 成功拉取

### Java（`java/`）

| 本地文件 | 来源 | 许可证 |
|---|---|---|
| `JavaGuide-ioc-and-aop.md` | [Snailclimb/JavaGuide](https://github.com/Snailclimb/JavaGuide) `docs/system-design/framework/spring/ioc-and-aop.md` | Apache-2.0 |
| `toBeBetterJavaer-springboot-transaction.md` | [itwanger/toBeBetterJavaer](https://github.com/itwanger/toBeBetterJavaer) `docs/src/springboot/transaction.md` | 仓库未声明 LICENSE（仅本地学习） |
| `java-design-patterns-README-节选.md` | [iluwatar/java-design-patterns](https://github.com/iluwatar/java-design-patterns) README 前 250 行 | MIT（见上游 LICENSE.md） |
| `java-eight-part-README.md` | [CoderLeixiaoshuai/java-eight-part](https://github.com/CoderLeixiaoshuai/java-eight-part) README | 仓库未声明 LICENSE（仅本地学习） |

### Python（`python/`）

| 本地文件 | 来源 | 许可证 |
|---|---|---|
| `awesome-python-README.md` | [vinta/awesome-python](https://github.com/vinta/awesome-python) | CC BY 4.0（LICENSE 文件） |
| `fastapi-README.md` | [tiangolo/fastapi](https://github.com/tiangolo/fastapi) | MIT |
| `Python-100-Days-README.md` | [jackfrued/Python-100-Days](https://github.com/jackfrued/Python-100-Days) | 仓库未声明 LICENSE（仅本地学习） |
| `Python-100-Days-14-函数和模块.md` | 同上 `Day01-20/14.函数和模块.md` | 同上 |

## 失败 / 降级项

| 尝试 | 结果 |
|---|---|
| JavaGuide `springboot-introduce.md` 旧路径 | 404，改拉 `ioc-and-aop.md` |
| JavaGuide `java-basic-questions-01.md` | 体积过大（~71KB 且偏题海），**未整篇拉**；基础改由自建 `Java/01-*.md` 覆盖 |
| JavaGuide `springboot-knowledge-and-questions-summary.md` | 近空 stub（~1KB），跳过 |
| toBeBetterJavaer 旧 `docs/sidebar.md` 路径 | 404，改用 `docs/src/springboot/transaction.md` |
| Python-100-Days 根目录 `LICENSE` | 404；API 亦无 SPDX |
| design patterns 完整 README | 过大，只保留前 250 行索引 |
| GitMCP `search_generic_code` 对 JavaGuide/toBeBetterJavaer 的 Spring 查询 | 无命中；改用 GitHub API + raw curl |

## 怎么用

1. 先读自建笔记 `Java/`、`Python/`、`前端/`
2. 对照本目录原文加深（IoC、事务、FastAPI、React/antd/Storybook 等）
3. 看 `对照补缺.md` 知道还缺啥（偏 Java/Python）

数据库开源精选仍在：`/workspace/面试八股/07-开源精选/`（本目录不改动那边文件）。

## frontend（前端，追加）

目录：`开源精选/frontend/`

| 文件 | 来源 | 许可证 | 状态 |
|---|---|---|---|
| `react-README.md` | [facebook/react](https://github.com/facebook/react) | MIT | 成功（README 全文） |
| `storybook-README.md` | [storybookjs/storybook](https://github.com/storybookjs/storybook) | MIT | 成功（README 全文） |
| `ant-design-README.md` | [ant-design/ant-design](https://github.com/ant-design/ant-design) | MIT | 成功（README 全文） |
| `nextjs-README.md` | [vercel/next.js](https://github.com/vercel/next.js) `packages/next/README.md` | MIT | 成功（根 README 404，改拉 package README） |
| `awesome-react-README-节选.md` | [enaqx/awesome-react](https://github.com/enaqx/awesome-react) | CC BY 4.0 | 成功（节选前 180 行） |

拉取说明：优先 GitMCP `fetch_generic_url_content`；失败或需落盘时用 `curl` raw。仅 README，非整库。

## devops（运维）

见 [`devops/README.md`](./devops/README.md)。

## agent（AI Agent，追加）

目录：`开源精选/agent/`

详见该目录 [`README.md`](./agent/README.md) 与 [`对照补缺.md`](./agent/对照补缺.md)。

含 LangGraph / LangChain / OpenAI Agents / LlamaIndex / awesome-agent-quickstart / Datawhale easy-langent / tour-of-agents；AutoGen README 仅作 archived 历史参考。

