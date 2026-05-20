# [智能合约（Smart Contract）](https://aiweb3.school/zh/handbook/web3/smart-contract)

> 约 8 分钟阅读 · 2026-05-12

**核心原则**：智能合约能让任何人检查规则、调用接口、验证状态变化。这个开放性带来了**可组合性**，也带来了**攻击面**。

## 第一性原理

合约代码公开、状态公开、资金公开。任何人都可以调用公开接口，攻击者可以反复模拟和抢跑。部署前要充分测试，部署后难回滚。

## 知识节点

| 节点 | 解释 |
|------|------|
| **Solidity** | EVM 生态最常见的合约语言。关键概念：storage / msg.sender / modifier / event / external call / revert / 权限控制 |
| **EVM** | Ethereum Virtual Machine，合约执行环境。决定 gas 机制、storage 成本、外部调用风险 |
| **ABI** | Application Binary Interface，描述合约的函数/参数/返回值/事件。Agent 理解合约能力的重要上下文，但 ABI ≠ 安全说明书 |

## 最小合约示例（Counter）

```solidity
contract Counter {
    uint256 public count;          // 链上状态变量
    event CountChanged(uint256);   // 事件日志（可索引）
    function increment() external { // 写操作：需签名+gas
        count += 1;
        emit CountChanged(count);
    }
}
```

- 读 `count`：无需签名
- 调 `increment()`：需要钱包签名 + 支付 gas

## 合约 vs 普通后端

| 特性 | 智能合约 | 普通后端 |
|------|---------|---------|
| 状态 | 链上公开 | 数据库私有 |
| 升级 | 部分不可更改 | 随意部署 |
| 执行 | 需 gas，公开 | 免费，封闭 |
| 审计 | 合约代码可直接检查 | 需后端权限 |

---
