# [安全（Security）](https://aiweb3.school/zh/handbook/web3/security)

> 约 5 分钟阅读 · 2026-05-12

**核心原则**：Web3 安全不是上线前找人审一次代码，而是从合约设计、权限、测试、交易模拟、监控、应急暂停到权限撤销的一整套工程流程。

## 第一性原理

链上系统默认暴露在公开对抗环境里，任何可调用路径都要按攻击面看待。

普通后端可以靠权限、网络隔离、人工回滚和数据库修复兜底。合约不同：代码公开，状态公开，资金公开，攻击者可以反复模拟和抢跑。

- **权限必须最小化**：owner、admin、upgrade、pause、withdraw 都要有清楚边界
- **执行前要模拟**：交易能不能成功、会改什么资产、会调用哪些合约，都要尽量提前看见
- **上线后要监控**：安全不是部署那一刻结束

## 知识节点

| 节点 | 解释 |
|------|------|
| **Reentrancy** | 合约在外部调用未完成前被再次调用，导致状态被重复利用。防护：Checks-Effects-Interactions（先检查→再更新状态→最后外部调用）、reentrancy guard |
| **Access Control** | 决定谁能执行敏感动作（mint/burn/pause/upgrade/withdraw）。检查权限不只问"有没有 onlyOwner"，更要问：owner 是谁（EOA/多签/治理）、有无 timelock、角色能否相互授予、权限变更是否发 event、私钥泄漏最坏结果 |
| **Audit** | 外部安全审查，不是安全保证书。看 report 时关注：覆盖范围、已修复/已接受风险、是否覆盖经济机制和外部依赖、上线代码是否和审计代码一致 |
| **Simulation** | 交易发出前的预演。能挡住：链 ID 错、合约地址错、授权额度异常、滑点过大、余额不足。不能替代安全审计 |
| **Monitoring** | 上线后的安全感知层。监控：大额转账、权限变更、合约升级、价格异常、TVL 流出、Agent 连续触发高风险工具。关键是"监控 + 响应" |

## 在 AI × Web3 中的位置

AI 会让安全边界更复杂。设计原则：模型可以建议，工具返回事实，policy 限制权限，simulation 预演结果，human check 确认高风险动作，monitoring 记录执行后果。

## 最小实践

1. 选一笔公开的合约调用交易
2. 查看 from、to、method、value、token transfers、logs、gas used
3. 判断是否改变权限、资产或关键协议参数
4. 写出如果由 Agent 发起，执行前需做哪些 simulation 和 human check
5. 写出上线后应监控哪些 event 或异常指标

---
