# 知识库

> 个人学习知识沉淀，持续更新。涵盖所有方向的学习内容。
>
> 建于：2026-05-19

## 目录

- [AI / 机器学习](#ai--机器学习)
- [Web3 / 区块链](#web3--区块链)
- [编程 / 开发](#编程--开发)
- [工具 / 效率](#工具--效率)
- [语言学习](#语言学习)
- [其他](#其他)

---

## AI / 机器学习

### AI × Web3 School — 入门

来源：[Handbook 首页](https://aiweb3.school/zh/handbook/)

AI x Web3 不是把两个 buzzword 拼在一起。一个真实系统往往会同时涉及：

- 模型如何理解任务、组织上下文、调用工具
- Agent 如何从"回答问题"进入"执行流程"
- 钱包和账户如何表达权限、额度、时间和撤销条件
- 链上交易如何模拟、确认、记录和审计
- 支付、身份、声誉、治理如何接入自动化系统
- 隐私、安全和可验证性如何在产品里落地

**核心观点**：只从 AI 侧看会低估资产和权限风险；只从 Web3 侧看会忽略模型、上下文和工具编排的复杂度。

**Handbook 四层结构**：

1. **AI 基础** — LLM、Prompt、Context、RAG、Agent、Frameworks、Vibe Coding、MCP、Evaluation、Fine-tuning、Inference
2. **Web3 基础** — Cryptography、Wallet、Smart Contract、Dev Stack、Network、Account Abstraction、DeFi、Oracle、Indexing、Security
3. **AI × Web3 Bridge** — Chain-aware Context、Web3 Tool Use、Agent Workflow、Agent Wallet、Machine Payment、Settlement & Escrow、Agent Identity、Agent Trust & Reputation、AI Oracle、Verifiable AI、AI Security、AI Privacy、AI Sovereignty、Governance AI、Decentralized AI
4. **前沿探索** — Agentic Commerce、Dev Tooling、Wallet / Permission、AI Security、Governance、Open Track

### LLM 大语言模型 — 核心概念

**本质**：基于上下文进行概率生成，预测最合理的下一个 token 序列。

- **Token**：最小读取单位 ≈ 0.75 英文词 / 1 中文字
- **Embedding**：文字 → 数学向量，表达语义相似度
- **Transformer**：核心架构，"注意力机制"让模型关注重要输入
- **Hallucination（幻觉）**：自信编造不存在的信息，需外部核实

**擅长**：语言理解、代码生成、推理、创意
**不擅长**：精确事实记忆、确定计算、跨会话状态保持

### Prompt → Workflow → Agent 区别

| 层次 | 决策者 | 路径 | 风险 |
|------|--------|------|------|
| Prompt | 人 | 单次问答 | 低 |
| Workflow | 预定流程 | 固定路径，模型是节点 | 中 |
| Agent | 模型自主 | 动态规划+工具调用 | 高 |

### Agent 核心技术组件

- **Tool Calling**：模型输出结构化请求 → 框架执行 → 回传结果
- **MCP（模型上下文协议）**：LLM 与外部工具的统一连接协议
- **Skills**：可复用的高层指令集，支持自动发现和动态生成
- **Tracing**：可视化 agent 执行链
- **Guardrails**：输入输出验证规则，不符合则中止
- **Handoff**：子任务完成后移交控制权
- **错误恢复**：执行失败时的重试、回退和人工介入
- **长期记忆**：跨 session 存储与召回

**什么时候用 Agent？** 目标开放、需多工具协作、中间结果决定下一步、需跨会话记忆。
**什么时候不适合？** 一次性问答用 Prompt、流程固定用脚本、高合规要求用人工审核节点。

### Prompt 深度理解

**核心原则**：Prompt 不是咒语，是把模糊任务变成模型能稳定执行的工作说明。

**第一性原理**：prompt 负责表达任务，**系统负责执行约束**。Prompt 不能保证模型永远按规则行动。

**Instruction 四段式写法**：
1. 角色 + 任务说明
2. 做什么 + 不做什么（边界）
3. 遇到不确定信息怎么处理
4. 输出格式要求

**Few-shot（少样本示例）**：放少量示例让模型模仿判断方式和输出格式。适合"风格和边界很难一句话说清"的任务。注意：旧示例可能过时误导模型。

**Structured Output（结构化输出）**：让模型按 JSON / schema 返回，后续系统才能检查、拒绝、记录和回归测试。

**Prompt Injection（提示注入）**：恶意用户/文档往 prompt 里注入冲突指令。防御：输入净化 + 权限分离 + 输出验证。

### Agent 深度理解

**核心定义**：Agent 是能围绕目标持续调用工具、读取状态、调整步骤的 AI 系统。

**核心分工**：模型负责提出候选行动，系统负责限制行动空间，用户负责批准高风险边界。

**三大支柱**：
- **权限分级**：读数据 vs 写数据库 vs 发送请求 vs 改配置 — 不同风险等级
- **可观测性**：任务进度、工具结果、失败原因、用户确认都要记录
- **停止条件**：达到目标、超出预算、信息不足、风险越界、用户拒绝，都应停下来

**Tool Use 设计要点**：输入 schema、权限范围、是否只读、外部副作用、调用前后记录、哪些需要人工确认。

**Planning**：把目标拆成步骤。每一步需暴露：工具、读写类型、能否自执行、是否需要用户确认、失败能否重试。

**Agent 失败模式**：循环卡住、工具滥用、幻觉传递（一个工具的幻觉传给下一个工具）、过度信任自身输出、权限蔓延。

## Web3 / 区块链

<!-- TODO -->

## 编程 / 开发

<!-- TODO -->

## 工具 / 效率

<!-- TODO -->

## 语言学习

<!-- TODO -->

## 其他

<!-- TODO -->
