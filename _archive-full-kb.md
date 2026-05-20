# 知识库

> 个人学习知识沉淀，持续更新。涵盖所有方向的学习内容。
>
> 建于：2026-05-19

## 目录

- [AI / 机器学习](ai/index.md)
  - [AI × Web3 School 课程概览](ai/course-overview.md)
  - [LLM 大语言模型](ai/llm.md)
  - [Prompt 提示词](ai/prompt.md)
  - [Context 上下文](ai/context.md)
  - [RAG 检索增强生成](ai/rag.md)
  - [Agent 智能体](ai/agent.md)
  - [Frameworks 框架](ai/frameworks.md)
  - [Vibe Coding 氛围编程](ai/vibe-coding.md)
  - [MCP 模型上下文协议](ai/mcp.md)
  - [Evaluation 评估](ai/evaluation.md)
  - [Fine-tuning 微调](ai/fine-tuning.md)
  - [Inference 推理服务](ai/inference.md)
- [Web3 / 区块链](web3/index.md)
- [编程 / 开发](programming/index.md)
- [工具 / 效率](tools/index.md)
- [语言学习](language/index.md)
- [其他](other/index.md)

---

## AI / 机器学习

### AI × Web3 School 课程概览

**来源**：[Handbook 首页](https://aiweb3.school/zh/handbook/)

AI x Web3 不是把两个 buzzword 拼在一起。一个真实系统往往会同时涉及：模型理解任务、Agent 执行流程、钱包权限管理、链上交易审计、支付身份治理、隐私安全可验证性。

**核心观点**：只从 AI 侧看会低估资产和权限风险；只从 Web3 侧看会忽略模型、上下文和工具编排的复杂度。

**Handbook 四层结构**：

| 层级 | 内容 | 链接 |
|------|------|------|
| **AI 基础** | LLM、Prompt、Context、RAG、Agent、Frameworks、Vibe Coding、MCP、Evaluation、Fine-tuning、Inference | [AI 基础](https://aiweb3.school/zh/handbook/) |
| **Web3 基础** | Cryptography、Wallet、Smart Contract、Dev Stack、Network、Account Abstraction、DeFi、Oracle、Indexing、Security | [Web3 基础](https://aiweb3.school/zh/handbook/) |
| **AI × Web3 Bridge** | Chain-aware Context、Web3 Tool Use、Agent Workflow、Agent Wallet、Machine Payment、Settlement & Escrow、Agent Identity、Trust & Reputation、AI Oracle、Verifiable AI、AI Security、AI Privacy、AI Sovereignty、Governance AI、Decentralized AI | [Bridge](https://aiweb3.school/zh/handbook/) |
| **前沿探索** | Agentic Commerce、Dev Tooling、Wallet/Permission、AI Security、Governance、Open Track | [前沿](https://aiweb3.school/zh/handbook/) |

---

### [LLM 大语言模型](ai/llm.md)

> 约 7 分钟阅读 · 2026-03-31

**本质**：基于上下文进行概率生成，预测最合理的下一个 token 序列。

**知识节点**：

| 节点 | 解释 |
|------|------|
| **Token** | 最小读取单位 ≈ 0.75 英文词 / 1 中文字。模型以 token 为单位接收和生成文本 |
| **Embedding** | 文字 → 数学向量，表达语义相似度。相似的词在向量空间中距离更近 |
| **Transformer** | 核心架构，注意力机制（Attention）让模型关注输入中重要的部分，而非平均对待每个词 |
| **Hallucination（幻觉）** | 模型自信地编造不存在的信息。这是概率生成的固有特性，**不是 bug**，必须通过外部手段验证 |

**模型能力边界**：
- ✅ **擅长**：语言理解、代码生成、推理、创意写作
- ❌ **不擅长**：精确事实记忆（会编造）、确定性计算、跨会话状态保持

**第一性原理**：LLM 是一个文本续写系统，不是知识库或数据库。它基于训练数据学到的模式进行预测，因此：
- 训练数据里没有的信息，它会"合理补全"（即编造）
- 上下文窗口之外的"记忆"不存在
- 工具调用和 RAG 是用来弥补这些不足的，不是附加功能

---

### [Prompt 提示词](ai/prompt.md)

> 约 6 分钟阅读 · 2026-05-12

**核心原则**：Prompt 不是咒语，是把模糊任务变成模型能稳定执行的工作说明。

**第一性原理**：Prompt 负责表达任务，**系统负责执行约束**。Prompt 不能保证模型永远按规则行动。只要输入里混入恶意文档、错误上下文或冲突指令，模型就可能偏离原始意图。

**知识节点**：

| 节点 | 解释 |
|------|------|
| **Instruction** | 给模型的任务规则。实际写法拆成四段：① 角色+任务 ② 做什么/不做什么 ③ 不确定信息处理 ④ 输出格式 |
| **Few-shot** | 在 prompt 里放少量示例，让模型模仿判断方式和输出格式。适合"风格和边界难用文字说清"的任务。注意：旧示例可能过时误导模型 |
| **Structured Output** | 让模型按 JSON/schema 返回固定结构。后续代码才能检查、拒绝、记录和回归测试 |
| **Prompt Injection** | 恶意用户/文档向 prompt 注入冲突指令。防御：输入净化 + 权限分离 + 输出验证 |

**Instruction 四段式写法**：
1. **角色 + 任务说明**：你是一个交易解释助手，需要解释链上交易的风险
2. **做什么 + 不做什么**：解释交易但不代替用户确认；可以分析风险但不能给出投资建议
3. **不确定信息怎么处理**：如无法确认合约来源，标记`[unverified]`并说明
4. **输出格式要求**：以 JSON 返回，包含 `risk_level`、`description`、`unverified_claims` 字段

---

### [Context 上下文](ai/context.md)

> 约 6 分钟阅读 · 2026-05-12

**核心原则**：LLM 不会自动连接你的数据库、文件系统、API 或项目文档。它只会基于当前上下文生成输出。如果上下文错了、缺了、过期了，模型再强也会给出不可靠答案。

**第一性原理**：Context 不是简单的长文本拼接，而是一套**信息治理问题**。要为每类信息标注来源、时效、权限和可信度。否则模型会把"用户说的""网页写的""链上查到的""系统规定的"混在同一层处理。

**知识节点**：

| 节点 | 解释 |
|------|------|
| **Context Window** | 模型一次请求能处理的最大范围。窗口大 ≠ 效果好 —— 模型可能被无关内容分散注意力，也可能引用过期段落。长上下文应配合检索、摘要和结构化数据使用 |
| **Context Engineering** | 设计上下文进入模型的方式：选择数据源、排序、裁剪、标注来源、隔离不可信内容。稳定 Agent 上下文应包括：系统指令、工具 schema、用户输入、历史状态、当前权限/预算、外部知识检索结果 |
| **Memory** | 跨请求保留的信息（用户偏好、历史任务等）。注意：用户过去允许 ≠ 现在仍然允许。Memory 不能替代实时授权 |
| **Knowledge Base** | 系统可检索的外部知识库（产品文档、SDK、FAQ、runbook 等） |

---

### [RAG 检索增强生成](ai/rag.md)

> 约 5 分钟阅读 · 2026-05-12

**核心原则**：RAG 不是让回答更长，而是让回答**有来源、有版本、有边界**。

**第一性原理**：一个 RAG 系统至少有三次判断——文档怎么切、查询时取哪些内容、生成时如何引用和拒答。任何一层做错，模型都会拿着错误材料说得很顺。

**知识节点**：

| 节点 | 解释 |
|------|------|
| **Chunking** | 把长文档切成可检索片段。切太小语义断裂，切太大噪声多。推荐按结构切：标题、API endpoint、函数说明、FAQ。每个 chunk 保留来源 URL、更新时间 |
| **Vector DB** | 存储 embedding 并按相似度检索。但向量相似 ≠ 答案正确。DB 里应存 metadata：来源、版本、可信等级、是否废弃。检索时先过滤再排序 |
| **Retriever** | 根据用户问题取回候选材料。不能只看语义相似度，还要知道用户问的是哪个产品、哪个版本、哪段时间 |
| **Rerank** | 初步检索后对候选材料重新排序。对技术文档问答和资产风险评估场景通常值得加 |
| **Citation** | 把答案里的关键结论连接回来源。至少要说明：结论来源、原文片段、更新日期、可信等级 |

---

### [Agent 智能体](ai/agent.md)

> 约 6 分钟阅读 · 2026-05-12

**核心定义**：Agent 是能围绕目标持续调用工具、读取状态、调整步骤的 AI 系统。关键不在"像人"，而在于能不能在明确权限和可审计流程里把建议推进到行动。

**核心分工**：模型负责提出候选行动 → 系统负责限制行动空间 → 用户负责批准高风险边界。

**知识节点**：

| 节点 | 解释 |
|------|------|
| **Tool Use** | Agent 调用外部能力（搜索、API、代码执行、支付接口等）。工具设计要明确：输入 schema、权限范围、只读还是写入、外部副作用、调用前后记录、哪些需要人工确认 |
| **Planning** | 把目标拆成步骤。每一步需暴露：用什么工具、读还是写、能否自动执行、是否需要用户确认、失败可否重试 |
| **MCP** | 协议化工具接入方式——详见 [MCP 章节](ai/mcp.md) |
| **Skills** | 可复用高层指令集，支持自动发现和动态生成 |
| **Tracing** | 可视化 agent 执行链——每一步调用什么工具、输入输出、耗时、谁触发的 |
| **Guardrails** | 输入输出验证规则，不符合则中止——详见 [Evaluation 章节](ai/evaluation.md) |
| **Handoff** | 子任务完成后移交控制权 |
| **错误恢复** | 执行失败时的重试、回退和人工介入 |
| **长期记忆** | 跨 session 存储与召回——详见 [Context 章节](ai/context.md) |

**Agent 三大支柱**：
1. **权限分级**：读数据 vs 写数据库 vs 发送请求 vs 改配置 —— 不同风险等级
2. **可观测性**：任务进度、工具结果、失败原因、用户确认都要记录
3. **停止条件**：达到目标、超出预算、信息不足、风险越界、用户拒绝都应停下来

---

### [Frameworks 框架](ai/frameworks.md)

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

### [Vibe Coding 氛围编程](ai/vibe-coding.md)

> 约 7 分钟阅读 · 2026-05-12

**核心原则**：Vibe Coding 不是凭感觉写代码。实际可用的做法正好相反：你要**更清楚**地管理 repo、任务、上下文、测试和提交边界。

**知识节点**：

| 节点 | 解释 |
|------|------|
| **Vibe Coding 定义** | 快速和 AI 共同探索代码的方式：描述目标 → Agent 生成方案/patch → 运行/检查/迭代收敛 |
| **Claude Code / Codex CLI** | 把 AI Agent 放进本地开发环境，可读/写/运行命令 |
| **Model / API Config** | 管理模型选择、API key、工具权限、上下文、成本和日志 |
| **GitHub / gh** | 把 AI 生成的改动放回版本控制、issue、PR 和 review 流程 |

**安全守则**：
- 不要让它无边界地"重写整个项目"
- 每次 AI 生成改动后必须审查 diff
- 涉及合约部署/权限变更/签名操作的动作必须人工确认

---

### [MCP 模型上下文协议](ai/mcp.md)

> 约 5 分钟阅读 · 2026-05-12

**核心原则**：MCP 解决的不是"模型会不会更聪明"，而是"模型如何以可描述、可复用、可管理的方式使用外部能力"。

**知识节点**：

| 节点 | 解释 |
|------|------|
| **Server** | 提供能力的一侧，暴露 resources / tools / prompts |
| **Client** | 连接模型和 MCP server 的一侧 |
| **Tool Schema** | 描述工具名字、用途、参数、返回值和约束 |

**三大支柱**：
1. **Schema 精确**：没有 schema，模型调用工具会变成自然语言猜参数
2. **授权隔离**：协议能描述能力，但真正的授权、审计和隔离要由系统实现
3. **错误明确**：工具失败、超时、权限不足必须明确返回

---

### [Evaluation 评估](ai/evaluation.md)

> 约 5 分钟阅读 · 2026-05-12

**核心原则**：Evaluation 把"感觉效果不错"变成"系统可持续改进"的方法。

**知识节点**：

| 节点 | 解释 |
|------|------|
| **Harness** | 运行 eval 的框架，核心价值：**可重复** |
| **Golden Set** | 一组认真标注的测试样本，30~100 条高质量样本 |
| **LLM-as-Judge** | 用模型给模型输出评分，适合开放式答案 |
| **Regression Set** | 每修一个重要 bug，都应变成 regression 样本 |
| **Task-level Eval** | 评估整条链路是否完成任务 |

---

### [Fine-tuning 微调](ai/fine-tuning.md)

> 约 5 分钟阅读 · 2026-05-12

**核心原则**：Fine-tuning 不是"效果不好就训练一下"。它适合让模型更稳定地学习格式/风格/领域任务，**不适合**补实时知识或修权限边界。

**知识节点**：

| 节点 | 解释 |
|------|------|
| **SFT（监督微调）** | 输入+期望输出样本，让模型学习任务。对数据质量非常敏感 |
| **LoRA** | 参数高效微调，训练较小的适配参数，降低成本和显存需求 |
| **PEFT** | 参数高效微调统称（LoRA 是其中一种） |

---

### [Inference 推理服务](ai/inference.md)

> 约 5 分钟阅读 · 2026-05-12

**核心原则**：同一模型在不同部署方式上的表现完全不同。你的选择是一套延迟、成本、质量、隐私和运维复杂度的平衡。

**知识节点**：

| 节点 | 解释 |
|------|------|
| **API Model** | 通过云端服务调用模型。速率限制、超时、重试、账单控制 |
| **Local Model** | 在自己设备上运行模型。适合隐私要求高、离线场景 |
| **Quantization** | 降低模型精度（FP16→INT8/INT4）以节省显存 |
| **Serving** | 把模型部署成服务：并发、队列、缓存、版本管理 |

---

## Web3 / 区块链

[→ 查看详情](web3/index.md)

## 编程 / 开发

[→ 查看详情](programming/index.md)

## 工具 / 效率

[→ 查看详情](tools/index.md)

## 语言学习

[→ 查看详情](language/index.md)

## 其他

[→ 查看详情](other/index.md)
