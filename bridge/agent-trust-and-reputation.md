# [Agent 信任与声誉（Agent Trust & Reputation）](https://aiweb3.school/zh/handbook/bridge/agent-trust-and-reputation)

> 约 6 分钟阅读 · 2026-05-12

**核心原则**：当 Agent 声称自己能完成任务时，用户和其他 Agent 如何判断它是否可靠、历史是否真实、失败是否有代价。信任不是一个分数，而是一组可追溯、可比较、可解释的证据。

## 第一性原理

Agent 的可信度应该来自可验证行为，而不是自我声明。

- **声誉要绑定身份**：没有稳定身份，历史记录无法积累
- **评价要绑定任务**：泛泛五星评价不如具体任务结果
- **惩罚要真实**：没有成本的承诺很容易被滥用

## 知识节点

| 节点 | 说明 |
|------|------|
| **Reputation** | Agent 历史表现信号：成功率、响应时间、争议率、退款率、用户评分、验证通过次数、stake 数量。按任务类型拆开，处理时间衰减 |
| **Review** | 绑定任务 ID、交付物、付款记录和评价者身份。质量>数量。防互刷：评价者本身也需身份和声誉 |
| **Attestation** | 某主体对 Agent/任务/结果的可验证声明。结构化：issuer/subject/claim/evidence/expiration/revocation。需过期+撤销机制 |
| **Stake** | Agent 押上的经济保证。让承诺有成本。不代表能力，只说明失败时可能有被罚没资金。小团队可能 stake 少但质量高 |
| **Slashing** | 违约/作弊时罚没抵押。需定义：违约证据、挑战窗口、仲裁者、误罚处理、申诉机制。主观任务先 dispute 再 slashing |
| **Validation** | 对 Agent 能力或任务结果的验证：自动测试/benchmark/人工审核/多 Agent 复核/TEE。区分"能力验证"和"任务结果验证" |
| **ERC-8004** | Agent 身份+声誉+验证 registry 标准草案。提供身份/反馈/验证信号的公共承载层。不保证 Agent 一定好，但保证记录连续性 |

## 在 AI × Web3 中的位置

连接 Agent Identity、Settlement & Escrow、Machine Payment。没有信任层 Agent 无法安全委托。真正可靠系统同时看：身份、任务历史、评价者、证明、stake、争议记录和上下文适配度。

## 最小实践

Agent 任务声誉记录：定义任务→记录 Agent id/用户 id/输入 hash/交付 hash/付款→设计 review 字段（准确性/及时性/返工）→设计 validation 字段→设计 refund/slashing 条件。

---
