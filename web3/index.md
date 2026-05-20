# Web3 / 区块链

> AI × Web3 School Week 1 — 模块 B 学习笔记
>
> 参考：Handbook [钱包](https://aiweb3.school/zh/handbook/web3/wallet) | [智能合约](https://aiweb3.school/zh/handbook/web3/smart-contract)

## [钱包（Wallet）](https://aiweb3.school/zh/handbook/web3/wallet)

> 约 6 分钟阅读 · 2026-05-12

### 核心概念链

```
助记词 → 私钥 → 地址 → 签名 → 交易 → 链上状态
```

### 知识节点

| 节点 | 解释 |
|------|------|
| **EOA** | Externally Owned Account，由私钥控制的外部账户。大多数钱包默认创建这类账户。优点：简单通用；缺点：私钥丢失难恢复、权限难细分、自动化能力弱 |
| **Mnemonic（助记词）** | 一组单词恢复钱包私钥。**任何网页/客服/AI Agent 要求输入助记词都应视为危险** |
| **Transaction（交易）** | 对链上状态的正式请求。按钮只是发起，需经：钱包确认 → 签名 → 广播 → 打包 → 执行。每一步都可能失败 |
| **Gas** | 链上执行成本。gas 不够交易失败但费用不退。需要理解 gas limit 和 gas price |
| **Explorer（区块浏览器）** | 查交易哈希、状态、Gas 消耗、合约地址、区块高度 |

### 三类钱包交互（风险等级）

| 类型 | 风险 | 说明 |
|------|------|------|
| **读操作** | 低 | 查余额、读合约状态，无需签名 |
| **签名** | 中 | 登录验证、消息签名，不改变链上状态 |
| **交易** | **高** | 转账、合约调用、授权，需要人工确认 + gas |

### 安全原则

- 地址 ≠ 匿名（链上公开可查）
- 签名 ≠ 普通登录（可能在授权具体动作）
- AI Agent **不应直接接触私钥/助记词**
- 涉及签名/转账/合约写入必须保留人工确认

---

## [智能合约（Smart Contract）](https://aiweb3.school/zh/handbook/web3/smart-contract)

> 约 8 分钟阅读 · 2026-05-12

### 核心原则

智能合约能让任何人检查规则、调用接口、验证状态变化。这个开放性带来了**可组合性**，也带来了**攻击面**。

### 知识节点

| 节点 | 解释 |
|------|------|
| **Solidity** | EVM 生态最常见的合约语言。关键概念：storage / msg.sender / modifier / event / external call / revert / 权限控制 |
| **EVM** | Ethereum Virtual Machine，合约执行环境。决定 gas 机制、storage 成本、外部调用风险 |
| **ABI** | Application Binary Interface，描述合约的函数/参数/返回值/事件。Agent 理解合约能力的重要上下文，但 ABI ≠ 安全说明书 |

### 最小合约示例（Counter）

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

### 合约 vs 普通后端

| 特性 | 智能合约 | 普通后端 |
|------|---------|---------|
| 状态 | 链上公开 | 数据库私有 |
| 升级 | 部分不可更改 | 随意部署 |
| 执行 | 需 gas，公开 | 免费，封闭 |
| 审计 | 合约代码可直接检查 | 需后端权限 |

---

## [网络（Network）](https://aiweb3.school/zh/handbook/web3/network)

> 约 3 分钟阅读 · 2026-05-12

| 节点 | 解释 |
|------|------|
| **L1（Layer 1）** | 基础链，如 Ethereum 主网。安全性最高，Gas 最贵 |
| **L2（Layer 2）** | 二层网络，如 Arbitrum、Optimism、Base。在 L1 之上提供更低成本、更快速度 |
| **测试网（Testnet）** | 如 Sepolia、Holesky。学习实验应先在测试网完成 |
| **桥（Bridge）** | 在不同网络间转移资产 |
| **RPC** | Remote Procedure Call，与链通信的接口 |

### 选择网络的要点

- 学习和实验优先用测试网
- Agent 的交易先在测试网模拟再上主网
- 不同网络的 Gas 和确认时间差异很大
