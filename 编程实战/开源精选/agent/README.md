# 开源精选 · Agent

拉取时间：2026-11-06（Asia/Shanghai）  
路径：`/workspace/编程实战/开源精选/agent/`  
仅供本地学习对照；遵守各仓库许可证，勿擅自再分发。

## 成功拉取

| 本地文件 | 来源 | 许可证（据上游） | 说明 |
|---|---|---|---|
| `langgraph-README.md` | [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) | MIT | 有状态 Agent 编排 |
| `langchain-README.md` | [langchain-ai/langchain](https://github.com/langchain-ai/langchain) | MIT | LLM/Agent 组件平台 |
| `openai-agents-python-README.md` | [openai/openai-agents-python](https://github.com/openai/openai-agents-python) | 见上游 | 官方 Agents SDK |
| `llamaindex-README.md` | [run-llama/llama_index](https://github.com/run-llama/llama_index) | MIT | 偏 RAG/数据应用 |
| `awesome-agent-quickstart-README.md` | [ababdotai/awesome-agent-quickstart](https://github.com/ababdotai/awesome-agent-quickstart) | MIT | 多框架 helloworld 索引 |
| `easy-langent-README.md` | [datawhalechina/easy-langent](https://github.com/datawhalechina/easy-langent) | CC BY-NC-SA 4.0 | **中文** LangChain/LangGraph 课 |
| `tour-of-agents-README.md` | [ahumblenerd/tour-of-agents](https://github.com/ahumblenerd/tour-of-agents) | MIT | 从零 ~60 行讲清 Agent 循环 |
| `autogen-README.md` | [microsoft/autogen](https://github.com/microsoft/autogen) | 见上游 | **已 archived**；仅历史参考 |

## 失败 / 注意

| 尝试 | 结果 |
|---|---|
| GitMCP 拉 `tour-of-agents` `main` 分支 raw | 失败；改用 `master` + curl 成功 |
| GitHub `search_repositories` 宽泛 query | 一次空结果；改用 `langgraph in:name` 命中官方仓 |
| AutoGen | 拉取成功，但 README 明确 archived → 新项目勿依赖 |
| 完整教程仓库（notebook 全集） | **未整库克隆**；只拉 README，避免体积过大 |

## 推荐阅读顺序（开源原文）

1. `tour-of-agents-README.md` — 先建立「循环」心智模型  
2. `langgraph-README.md` + `langchain-README.md`  
3. `openai-agents-python-README.md`  
4. `easy-langent-README.md` — 中文路径与章节地图  
5. `awesome-agent-quickstart-README.md` — 横向扫框架  
6. `autogen-README.md` — 了解历史与迁移提示即可  

对照缺口：[`对照补缺.md`](./对照补缺.md)
