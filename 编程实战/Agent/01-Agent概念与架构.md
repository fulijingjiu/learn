# 01 · Agent 概念与架构

## Agent vs Chatbot

| | Chatbot | Agent |
|---|---|---|
| 目标 | 对话、回答 | 完成任务（可能多步） |
| 能力 | 主要生成文本 | 文本 + **工具调用 Tool Calling** |
| 控制权 | 用户每问一答 | 模型可循环：思考→行动→观察 |
| 风险 | 胡话 | 胡话 + **误操作外部系统** |

## 核心积木（面试可口述）

1. **LLM（大脑）**：推理与决策，不保证正确。
2. **Tools（手脚）**：函数 / API / SQL / 浏览器 / 代码执行器。签名要清晰。
3. **Planning（规划）**：拆任务、排序、决定停不停。可以是隐式（模型自己想）或显式（todo 列表）。
4. **Memory（记忆）**
   - 短期：当前对话 messages
   - 长期：向量库 / 档案 / 跨会话偏好
5. **Orchestration（编排）**：单 Agent 循环，或多 Agent 分工（supervisor / handoff）。

## 经典循环：ReAct（Reason + Act）

口述版：

> 想一步（Thought）→ 选工具（Action）→ 看结果（Observation）→ 再想，直到能给最终答案。

伪代码见 `05-代码片段与模式.md`。

## 多代理 Multi-Agent（概念级）

常见模式：

- **Router / Supervisor**：一个主管分派给专家子 Agent
- **Handoff**：A 做完交给 B（像客服转接）
- **并行子任务**：调研 Agent 同时搜多个主题再汇总

代价：延迟↑、费用↑、调试更难。能用单 Agent + 好工具就先别上多 Agent。

## Agent ≠ RAG ≠ Workflow

- 只有「查文档再答」→ 先做 **RAG**，不一定要 Agent。
- 步骤固定（先审后发）→ **Workflow / LangGraph 固定图** 更可控。
- 步骤依赖中间结果、要动态选工具 → 才上 **Agent**。

## 和 IDE Agent（如 Cursor）的关系

产品里的「编程 Agent」也是同一套：模型 + 读文件/终端/搜索等工具 + 循环。你在 Cursor 里看到的 tool 调用轨迹，就是生产 Agent 要学会的可观测性雏形。
