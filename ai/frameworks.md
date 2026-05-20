# [Frameworks 框架](https://aiweb3.school/zh/handbook/ai/frameworks)

> 约 7 分钟阅读 · 2026-05-12

**核心原则**：框架解决工程组织问题，不定义产品价值，不保证模型输出正确。

**第一性原理**：先画清楚输入、状态、工具、输出、评估和失败路径，再看哪些部分值得交给框架。不要让产品逻辑迁就框架。

**常见框架对比**：

| 框架 | 定位 | 适合场景 | 说明 |
|------|------|---------|------|
| **LangChain** | LLM 应用组件库 | 快速接入模型、prompt、工具、RAG | 适合组合能力，不适合决定产品边界。小心"抽象太早" |
| **LangGraph** | 工作流/状态机 | 多轮工具调用、重试、人工确认、分支恢复 | 任务简单时太重；流程复杂时显式 graph 更可靠 |
| **OpenAI Agents SDK** | Agent 执行引擎 | 带 tool/handoff/guardrails/tracing 的 Agent 应用 | 帮你执行，但你要定义边界 |
| **DSPy** | Prompt/LM pipeline 优化 | 有明确数据集和评估指标的分类/抽取/QA 任务 | 不是手调 prompt，而是用 optimizer 改进策略 |
| **Hermes** | CLI Agent + 工具调用生态 | 面向工具调用和结构化输出的 Agent 运行时 | 见 [Hermes Docs](https://hermes-agent.nousresearch.com) |

---

