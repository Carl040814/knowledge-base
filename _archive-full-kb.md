# 知识库（归档快照）

> 本文件为 2026-05-19 创建时的原始归档。内容已全部拆分为独立文件。
>
> **迁移日期**：2026-05-20
>
> **迁移说明**：原始单文件 knowledge-base.md → 按类别拆分为独立文件，每章一个 .md，统一格式（核心原则 → 第一性原理 → 知识节点表 → 安全/实践 → 最小实践）。

## 当前结构

```
knowledge-base/
├── README.md                      # 总索引
├── _archive-full-kb.md            # 本文件（归档快照）
├── ai/
│   ├── index.md                   # AI 目录索引
│   ├── agent.md                   # Agent 智能体
│   ├── context.md                 # Context 上下文
│   ├── course-overview.md         # 课程概览
│   ├── evaluation.md              # Evaluation 评估
│   ├── fine-tuning.md             # Fine-tuning 微调
│   ├── frameworks.md              # Frameworks 框架
│   ├── inference.md               # Inference 推理服务
│   ├── llm.md                     # LLM 大语言模型
│   ├── mcp.md                     # MCP 模型上下文协议
│   ├── prompt.md                  # Prompt 提示词
│   ├── rag.md                     # RAG 检索增强生成
│   └── vibe-coding.md             # Vibe Coding 氛围编程
├── web3/
│   ├── index.md                   # Web3 目录索引
│   ├── account-abstraction.md     # 账户抽象
│   ├── cryptography.md            # 密码学
│   ├── defi.md                    # 去中心化金融
│   ├── dev-stack.md               # 开发栈
│   ├── indexing.md                # 索引
│   ├── network.md                 # 网络
│   ├── oracle.md                  # 预言机
│   ├── security.md                # 安全
│   ├── smart-contract.md          # 智能合约
│   └── wallet.md                  # 钱包
├── programming/
│   └── index.md
├── tools/
│   └── index.md
├── language/
│   └── index.md
└── other/
    └── index.md
```

## 图表

### AI 知识地图

```
LLM ──→ Prompt ──→ Context ──→ RAG
  │                              │
  └────→ Agent ←─────────────────┘
           │
           ├──→ Frameworks
           ├──→ MCP
           ├──→ Vibe Coding
           ├──→ Evaluation
           ├──→ Fine-tuning
           └──→ Inference
```

### Web3 知识地图

```
Cryptography ──→ Wallet ──→ Smart Contract ──→ Dev Stack
                                         │
                   ┌─────────────────────┤
                   ↓                     ↓
              Network              Security
                   │
     ┌─────────────┼─────────────┐
     ↓             ↓             ↓
Account        DeFi          Oracle
Abstraction       │             │
     │             ↓             ↓
     └──────→ Indexing ←─────────┘
```

### AI × Web3 Bridge（预告）

```
AI 基础 ──┐                    ┌── Web3 基础
          │                    │
          └──→ Chain-aware ──→ Web3 Tool Use
               Context      ──→ Agent Wallet
                            ──→ Machine Payment
                            ──→ Agent Identity
                            ──→ AI Oracle
                            ──→ Verifiable AI
                            ──→ ...
```

---

> **来源**：[AI × Web3 School Handbook](https://aiweb3.school/zh/handbook/)
>
> GitHub：[Carl040814/knowledge-base](https://github.com/Carl040814/knowledge-base)
