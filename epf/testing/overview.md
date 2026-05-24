# 测试 (Testing)

以太坊客户端实现 (Ethereum client implementations) 在不同层面上进行着持续的测试，以确保网络的安全性和稳定性。对于去中心化网络而言，确保所有客户端正确通信、表现一致并从而对协议所定义的交易结果达成一致是必不可少的。在单个状态转换 (state transition) 中的微小差异都会导致网络分裂 (network split)，从而导致最终性失败 (finalization failure) 并给用户带来许多问题。为了实现稳定性，以太坊客户端必须针对标准化测试用例套件进行严格的测试。

这些测试验证了客户端对[执行层 (execution) 规范](/wiki/EL/el-specs.md)和[共识层 (consensus) 规范](/wiki/CL/cl-specs.md)的遵守情况，保证所有客户端以完全相同的方式解释和执行交易。这种严密的测试还可以作为一种主动的漏洞检测工具，防止发生网络分叉 (network forks)（即对规范区块链状态 (canonical blockchain state) 产生分歧）。

## 资源 (Resources)

### 视频指南 (Walkthrough)
- [测试与安全概述 (Testing & Security Overview)](https://www.youtube.com/watch?v=PQVW5dJ8J0c)

### 通用测试套件 (Common test suite)
- [pytest：Python 测试框架 (Python test framework)](https://docs.pytest.org/en/8.0.x/)
- [Ethereum Tests：所有实现的通用测试 (Common tests for all implementations)](https://github.com/ethereum/tests)
- [Hive：以太坊端到端测试工具套件 (Ethereum end-to-end test harness)](https://github.com/ethereum/hive)

### 执行层测试 (Execution layer tests)
- [Execution Spec Tests：执行层客户端的测试用例 (Test cases for execution clients)](https://github.com/ethereum/execution-spec-tests)
- [FuzzyVM：EVM 的差异化模糊测试器 (Differential fuzzer for EVM)](https://github.com/MariusVanDerWijden/FuzzyVM)
- [retesteth：测试生成工具 (Test generation tool)](https://github.com/ethereum/retesteth)
- [EVM 实验室实用工具 (EVM lab utilities)](https://github.com/ethereum/evmlab)
- [Go evmlab：受 EVMLAB 启发的 EVM 实验室 (Evm laboratory inspired by EVMLAB)](https://github.com/holiman/goevmlab)
- [执行层 API 集合 (Collection of execution APIs)](https://github.com/ethereum/execution-apis)

### 共识层测试 (Consensus layer test)
- [共识层规范测试 (Consensus Spec Tests)](https://github.com/ethereum/consensus-specs/tree/dev/tests)

### 链测试工具 (Tools for testing chains)
- [Assertoor：以太坊测试网测试工具 (Ethereum Testnet Testing Tool)](https://github.com/ethpandaops/assertoor)
- [Ethereum package：测试网部署工具 (Testnet deployer)](https://github.com/kurtosis-tech/ethereum-package)（推荐初学者使用）

### 仪表板 (Dashboards)
- [Hive 测试结果 (Hive test results)](https://hivetests.ethdevops.io/)