# [开发工具（Dev Tooling）](https://aiweb3.school/zh/handbook/tracks/dev-tooling)

> 约 6 分钟阅读 · 2026-05-12

**核心原则**：AI×Web3 开发工具不只"让 AI 写合约"，而是帮开发者理解协议、解释交易、生成测试、检查部署风险。

Web3 开发摩擦：文档分散、ABI 难读、calldata 不直观、测试难覆盖、部署不可逆。AI 降低认知负担但不能替代安全判断。

## 第一性原理

AI 开发工具的价值不是生成更多代码，而是减少错误理解和不可逆操作。

- **文档→可检索上下文**：回答必须回到官方文档、源码或 EIP
- **交易→可解释对象**：用户需看到函数、参数、资产变化和风险
- **部署→检查流程**：AI 生成 checklist，关键步骤必须可复核

## 知识节点

| 节点 | 说明 |
|------|------|
| **Docs-to-Agent** | 协议文档/SDK/EIP/README 变 Agent 知识库。处理：文档版本、来源优先级（官方>博客）、代码可运行性、引用追踪、更新同步 |
| **Contract Reading Assistant** | 解释合约：职责/依赖、谁可调管理函数、哪些函数动资产、依赖 oracle/owner/proxy、upgrade/pause/fee/黑名单、关键 event。安全结论需结合 Slither/测试/审计 |
| **Transaction Interpreter** | calldata→人类可读：调了哪个合约/函数、参数含义、哪些资产转出/授权、是否无限授权/代理调用/多跳 swap、失败原因、是否和用户意图一致 |
| **Test Generator** | 不只 happy path。覆盖：权限测试、状态转换、数值边界、资产安全（重入/重复提款）、Oracle 依赖、升级迁移。AI 生成草稿，开发者验证业务不变量 |
| **Deployment Checklist** | 合约地址/chain id/RPC/deployer 验证、constructor 参数、gas 策略、源码验证、权限配置、多签设置、监控告警、回滚预案 |

---
