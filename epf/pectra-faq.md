<!-- markdownlint-disable MD013 -->

# Pectra 升级问答 (Pectra FAQ)

**什么是 Pectra？**
Pectra（Prague - Electra，布拉格-埃勒特拉）是以太坊网络的一次升级，于 **2025 年 5 月 7 日 10:05 (UTC)** 在以太坊主网的纪元 `364032` 处激活。完整的 EIP 列表以及功能介绍可以在 [此处](https://notes.ethereum.org/@ethpandaops/mekong#What-is-in-the-Mekong-testnet) 找到。

**本指南适用于谁？**
适用于对即将到来的 Pectra 升级感兴趣的应用开发者 (App developers)、质押者 (Stakers) 和节点运营商 (Node operators)。

## 整体情况 (Overall)

**常见问题 (FAQ)**：

### **问**：什么是 Prague/Electra？

**答**：Prague 和 Electra 是即将到来的以太坊硬分叉 (hard fork) 的名称。包含的 EIP 可以在 [此处](https://eips.ethereum.org/EIPS/eip-7600) 找到。Prague 是执行层客户端 (execution client) 端的分叉名称，而 Electra 是共识层客户端 (consensus layer client) 端的升级名称。

Pectra 中包含 3 个主要功能以及一些较小的 EIP。它们是：[EIP-7251](https://eips.ethereum.org/EIPS/eip-7251)，也称为最大有效余额 (Max effective balance, MaxEB)；[EIP-7702](https://eips.ethereum.org/EIPS/eip-7702)；以及 [EIP-7002](https://eips.ethereum.org/EIPS/eip-7002)，也称为执行层触发退出 (Execution Layer triggered exits)。

[MaxEB](https://eips.ethereum.org/EIPS/eip-7251) 功能将允许用户拥有大于 32 ETH 的有效余额 (effective balance)。这将允许用户将许多验证者合并（或存入新的验证者）为一个高达 2048 ETH 的验证者。使用此功能的要求是设置 `0x02` 提款凭证 (withdrawal credentials)。用户可以直接使用 `0x02` 凭证进行存款，或者从 `0x01` 迁移到 `0x02`。

通过 [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702)，用户钱包将能够将控制权委托给智能合约 (smart contract)。这种模式开启了全新的钱包和应用交互设计空间，为未来完整的账户抽象 (account abstraction) 解决方案铺平了道路。

[执行层 (EL) 触发退出](https://eips.ethereum.org/EIPS/eip-7002) 是一项新功能，它允许在 `0x01` 或 `0x02` 提款凭证中设置的执行层地址直接在执行层 (EL) 中执行退出操作，而无需依赖预签名的 BLS 密钥。该功能主要针对质押池，使它们能够使用智能合约完全控制验证者的生命周期。

## 用户与开发者 (Users/Devs)

有关 Pectra 的更多资源和数据可以在 [https://pectra.wtf/](https://pectra.wtf/) 找到。

**常见问题 (FAQ)**：

### **问**：什么是 EIP-7702，它与账户抽象 (Account abstraction) 有什么关系？

虽然 [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702) 并不完全是账户抽象，但它确实提供了执行抽象 (execution abstraction)，即为外部拥有账户 (Externally Owned Accounts, EOAs) 添加了额外的功能。这允许您的 EOA 执行诸如发送交易批次和委托给其他密码学密钥方案（如密钥/Passkeys）等操作。它通过将与 EOA 关联的代码设置为协议级别的代理 (proxy) 指定来实现这一点。完整的规范可以在 [此处](https://eips.ethereum.org/EIPS/eip-7702) 找到。它引入了一种新的交易类型，在单笔交易期间临时将特定的合约代码授权给 EOA，从而允许 EOA 扮演智能合约账户的角色。这为用户启用了多种用例，包括交易打包、Gas 赞助和权限降级。

#### **问**：我在哪里可以找到 EIP-7702 的规范？作为钱包开发者我该如何使用它？

[EIP-7702](https://eips.ethereum.org/EIPS/eip-7702) 的规范可以在 [此处](https://eips.ethereum.org/EIPS/eip-7702) 找到。作为钱包开发者开始使用它，您需要确定要与 EOA 一起使用的智能合约钱包核心。请密切关注钱包[应该如何初始化](https://eips.ethereum.org/EIPS/eip-7702#front-running-initialization)。一旦确定了要使用的核心钱包，您就需要公开诸如 `eth_sendTransaction` 以及其他针对 [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702) 特定功能（如批量交易）的自定义方法。

#### **问**：作为用户，我该如何使用 EIP-7702？

要获得作为早期账户抽象尝试的 EIP-7702 的[好处](https://ethereum.org/en/roadmap/account-abstraction/)，您需要使用支持它的钱包。一旦您选择的钱包支持账户抽象，您就能够使用它。

#### **问**：我必须等待我的钱包支持 EIP-7702 吗？

很遗憾，是的。在您的钱包集成 [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702) 之前，将无法使用它提供的新功能。

#### **问**：作为智能合约开发者 (smart contract dev)，关于 EIP-7702 我需要了解什么？

作为智能合约开发者，您应该知道，在布拉格升级之后，以太坊上的大多数用户现在将能够以比以前更复杂的、在以前不可行的方式与链进行交互。已经开发了许多标准来绕过 EOA 的限制，例如 [ERC-2612 许可许可机制 (Permit)](https://eips.ethereum.org/EIPS/eip-2612)。

#### **问**：作为安全工程师/审计员 (security engineer/auditor)，关于 EIP-7702 我需要了解什么？

作为安全工程师 / 审计员，您必须意识到，以前当 `msg.sender == tx.origin` 时框架不能被重入的假设已不再成立。这意味着该检查不再适用于重入防范或闪电贷保护。

#### **问**：EIP-2537 BLS 预编译 (BLS precompile) 在 Pectra 中添加了什么？

[EIP-2537](https://eips.ethereum.org/EIPS/eip-2537) 将 BLS12-381 曲线上的操作作为预编译引入到以太坊。BLS12-381 预编译使得高效的 BLS 签名验证成为可能。这对于需要验证多个签名的应用程序（例如证明检查系统）非常有用。

#### **问**：我该如何使用 `BLOCKHASH` 操作码 (BLOCKHASH OPCODE)？

最新的 8192 个区块哈希现在存储在 `BLOCKHASH` 系统合约中并可供访问。`BLOCKHASH` 操作码语义与以前保持相同，只是现在可以指定大端 (big-endian) 编码的区块号。还可以通过 `eth_call` RPC 方法调用区块哈希系统合约，将相关的区块号作为调用数据 (calldata) 传入。

#### **问**：什么是系统合约 (system contracts)？

系统合约是以合约形式定义的接口，这对于发生某些以太坊功能是必不可少的。采用合约方式而不是每个客户端都实现该逻辑，是为了简化维护并允许在未来进行升级，且开销最小。

## 质押者 (Stakers)

**常见问题 (FAQ)**：

### **问**：存款有什么变化？

进行和提交存款的过程不会改变。您可以继续使用与之前相同的工具。然而，以太坊上处理存款的机制将经历改进。这一改进由 [EIP-6110](https://eips.ethereum.org/EIPS/eip-6110) 描述，并将允许几乎立即处理存款。

#### **问**：我需要等待多久才能包含我的存款？

在包含 [EIP-6110](https://eips.ethereum.org/EIPS/eip-6110) 的更改之后，在网络的常规最终确定期间，存款应在 20 分钟内显示。然而，您的验证者要被激活仍然存在存款队列，该 EIP 仅仅确保链能更快速、更安全地看到存款，并不会影响验证者激活的速度。

#### **问**：什么是 `0x02` 提款凭证 (withdrawal credentials)？

在 Pectra 分叉之前，以太坊接受两种类型的提款凭证：`0x00` 和 `0x01`。主要变化是 `0x01` 包含一个接收部分和全部提款的执行层地址。`0x02` 提款凭证是 Pectra 升级中引入的新型提款凭证。`0x02` 提款凭证将允许通过更大金额的存款或通过现有验证者的合并，来实现大于 32 ETH 且小于 2048 ETH 的最大有效余额。`0x02` 提款凭证还启用了使用执行层提款地址退出验证者的能力，从而允许通过执行层完全控制验证者。

#### **问**：我如何切换到 `0x02` 提款凭证？它对我有什么帮助？

验证者可以通过 2 种方式拥有 `0x02` 提款凭证：

1. 当您直接使用 `0x02` 提款凭证存入新的验证者时。
2. 当您通过向合并请求地址发送交易，将现有的验证者合并到 `0x02` 提款凭证时。

虽然 `0x01` 和 `0x02` 提款凭证都允许您从执行层地址控制验证者退出，但只有 `0x02` 允许验证者拥有大于 32 ETH 且小于 2048 ETH 的最大有效余额。这意味着您可以运行一个验证者，并拥有一个余额高达 2048 ETH 的单个验证者。

#### **问**：我可以直接存入具有 `0x02` 凭证的验证者吗？

是的，您可以直接存入具有 `0x02` 凭证的验证者。这将允许您拥有一个余额高达 2048 ETH 的单个验证者。

要立即尝试存款，您可以使用 ethstaker 的 [`staking-cli`](https://github.com/eth-educators/ethstaker-deposit-cli)，它已经通过 `--compounding` 标志支持 `0x02` 凭证。您还可以通过 `--amount` 标志指定高于或低于 32 ETH 的存款金额。
官方的 [`staking-deposit-cli`](https://github.com/ethereum/staking-deposit-cli) 将在 Pectra 主网以太坊分叉前的未来几个月内支持 `0x02` 提款凭证。

生成的存款数据文件可以像往常一样发送到网络的 Launchpad 以进行存款交易。
对于测试网，您可以使用 Dora 中的 [`Submit Deposits`](https://dora.mekong.ethpandaops.io/validators/deposits/submit) 页面来提交生成的存款。对官方 Launchpad 的支持也将在未来几个月内推出。

#### **问**：我有一个具有 `0x00` 凭证的验证者，我该如何迁移到 `0x02`？

没有直接从 `0x00` 迁移到 `0x02` 的方法。您需要首先通过 BLS 更改操作将您的验证者从 `0x00` 迁移到 `0x01` 提款凭证，然后将您的验证者合并到 `0x02` 提款凭证。您也可以选择在存款期间退出验证者并使用 `0x02` 提款凭证进行新的存款。

#### **问**：我有一个具有 `0x01` 凭证的验证者，我该如何迁移到 `0x02`？

您可以通过自我合并 (self consolidation)（源和目标都指向您的验证者）将您的验证者合并到 `0x02` 提款凭证。这将把您的提款凭证更改为 `0x02`，并允许您拥有一个余额高达 2048 ETH 的单个验证者。对于新存款，`staking-cli` 将在 Pectra 主网以太坊分叉前的未来几个月内支持 `0x02` 提款凭证。

#### **问**：什么是 MaxEB？

[EIP-7251](https://eips.ethereum.org/EIPS/eip-7251) 或 MaxEB 将 `MAX_EFFECTIVE_BALANCE` 提高到 2048 ETH，同时保持 32 ETH 的最低质押余额。在 MaxEB 之前，任何想要向共识贡献大量 ETH 的实体都必须运行多个验证者，因为每个验证者的上限为 32 ETH。[EIP-7251](https://eips.ethereum.org/EIPS/eip-7251) 将允许大型质押运营商将其 ETH 合并到更少的验证者中，使用相同的质押但验证者数量减少高达 64 倍。它还允许将独立质押者的 ETH 复利 (compound) 到其现有的验证者中，并在无需使用确切的验证者数量的情况下增加其奖励。例如，35 ETH 将被协议视为验证者的有效余额，而不是留下 3 ETH 作为无效余额并等到 64 ETH 才能开启 2 个验证者。总体而言，合并验证者将允许共识网络中的证明更少，从而减轻节点的带宽使用。

#### **问**：我该如何合并我的验证者？

要合并您的验证者，您首先需要确保源验证者和目标验证者都已分配了 `0x01` 或 `0x02` 凭证。
提款凭证使用 `0x00` 前缀或指向不同执行层地址的验证者不能进行合并。

要合并两个验证者，从您的提款地址向合并系统合约发送一笔交易，其中包含您希望合并的源验证者和目标验证者的公钥。

我们预计此功能将在未来几个月内添加到各种工具中。
要立即进行测试，您可以使用 Dora 中的 [提交合并 (Submit Consolidations)](https://dora.mekong.ethpandaops.io/validators/submit_consolidations) 页面。连接用作您的验证者提款地址的钱包，您应该能够选择您的验证者并构建适当的合并交易。

源和目标指向同一个验证者的合并称为自我合并 (self-consolidation)。
在这种情况下，验证者不会被退出，资金也不会被移动。它将只是被简单地分配为 `0x02` 凭证。

#### **问**：合并验证者有什么要求？

在执行合并时，验证者在信标链上必须是活跃 (active) 状态。这意味着它们不能处于正在退出、等待激活或除活跃之外的任何其他状态。
源地址必须授权合并交易，且目标地址必须是具有 `0x02` 凭证的验证者。具有不同提款凭证的验证者也可以进行合并。

#### **问**：我原来的独立验证者会发生什么？

在合并期间，存在一个源验证者和一个目标验证者。源验证者将完全退出，其余额随后转移到目标验证者。目标验证者将拥有源验证者和目标验证者的余额之和，并将继续履行其信标链职责，没有任何变化。

#### **问**：余额何时会出现在我合并后的验证者上？

一旦源验证者完全退出并停止履行所有职责，余额将被计入目标验证者。

#### **问**：如果我将一个具有 `0x01` 凭证的验证者与另一个具有 `0x00` 凭证的验证者合并会发生什么？

合并请求将被视为无效且不予处理。如果两个验证者不包含指向完全相同执行层地址的 `0x01` 提款凭证，它将失败。

#### **问**：如果我合并已退出的验证者会发生什么？

合并将失败，因为在执行合并时，验证者必须在信标链上是活跃的。

#### **问**：合并系统合约的 ABI 是什么？

EIP-7251 合并合约部署在 `0x0000BBdDc7CE488642fb579F8B00f3a590007251` 处，源代码在此处：[ethereum/sys-asm/src/consolidations/main.eas](https://github.com/ethereum/sys-asm/blob/main/src/consolidations/main.eas)。
合并被放入队列中，并以每区块 2 个的速度出队。
该合约不是用 Solidity 编写的，也没有典型的 Solidity ABI，以免将 API 固化。

伪代码 (pseudo code) 中的功能：

```pseudo
function(bytes input) {
    if input.length == 0 {
        return required_fee
    }
    if input.length == 96 {
        if msg.value < required_fee {
            return error
        }
        source := input[0:48]
        target := input[48:96]
        store_consolidation(msg.sender, source, target)
        emit event(msg.sender, source, target)
        return
     }
     return error
}
```

输入由 96 字节组成，包含合并的源和目标的 BLS 公钥。
源和目标必须都设置了 `0x01` 或 `0x02` 提款凭证。
可以再次调用无输入数据的合约来计算所需的合并费用。
资金将从源合并到目标。
EIP-7251 包含一个关于如何从 Solidity 与该[合约进行交互](https://eips.ethereum.org/EIPS/eip-7251#fee-overpayment)的示例。

#### **问**：我如何从我的 `0x02` 验证者中提取部分 ETH？

您可以发起由执行层 (EL) 触发的部分提款，从 `0x02` 验证者中提取部分 ETH。
向提款系统合约（待 Electra 在主网激活时确定最终地址）发送一笔交易，附带您的验证者公钥 (`pubkey`) 和金额 (`amount`)（正数且非零的 Gwei 金额）。
与合并类似，该交易必须从您验证者提款凭证中设置的提款地址发送。

我们预计此功能将在未来几个月内添加到各种工具中。
对于现在的测试，您可以使用 Dora 中的 [提交提款 (Submit Withdrawals)](https://dora.mekong.ethpandaops.io/validators/submit_withdrawals) 页面。连接用作您验证者提款地址的钱包，您应该能够选择您的验证者并构建适当的提款交易。

#### **问**：我可以从我的验证者中提取多少 ETH？

只要在提款完成时验证者包含大于 32 ETH，您就可以提取超出完整验证者金额的部分。例如，如果您当前有 34 ETH 并请求部分提款，则最多可以提取 2 ETH。
您也可以通过在请求中指定金额为 `0` 来请求全部提款。在发送此类全部提款请求时，您的验证者将被退出，其余额被全部提取。

#### **问**：提款系统合约的 ABI 是什么？

EIP-7002 合约部署在 `0x00000961Ef480Eb55e80D19ad83579A64c00700`，源代码在此处：[ethereum/sys-asm/src/withdrawals/main.eas](https://github.com/ethereum/sys-asm/blob/main/src/withdrawals/main.eas)。
提款被放入队列中，每区块最多有 16 个提款出队。
该合约不是用 Solidity 编写的，也没有典型的 Solidity ABI，以免将 API 固化。
提款合约在伪代码中的功能：

```pseudo
function(bytes input) {
    if input.length == 0 {
        return required_fee
    }
    if input.length == 56 {
        if msg.value < required_fee {
            return error
        }
        pk := input[0:48]
        amount := input[48:56]
        store_withdrawal(msg.sender, pk, amount)
        emit event(msg.sender, pk, amount)
        return
    }
    return error
}
```

如您所见，输入由 56 字节组成，包含我们想要提款的 BLS 公钥以及我们想要提取的金额。
您可以调用无输入数据的合约来查询提款所需的费用。
一个关于如何从 Solidity 与其进行交互的例子可以在 [此处](https://eips.ethereum.org/EIPS/eip-7002#fee-overpayment) 找到。

#### **问**：如果我的验证者具有 `0x02` 凭证且降至 32 ETH 以下，ETH 余额会发生什么？

表现正常的验证者即使您发起了部分提款请求，其余额也不会降至 32 ETH 以下。这只有在验证者受到处罚 (penalty) 时才会发生。除了收益减少外，什么都不会发生。然而，如果余额降至 16 ETH 以下，验证者将被退出，余额将被转移到执行层提款地址。

#### **问**：如果我的验证者具有 `0x02` 凭证且超过 2048 ETH，ETH 余额会发生什么？

余额将继续在验证者处累积，直到触发下一次部分提款。然而，验证者的最大有效余额为 2048 ETH，超出部分的余额在信标链中将被视为无效。

#### **问**：在 32 ETH 和 2048 ETH 之间，我可以赚取哪些余额的收益？

有效余额以 1 ETH 为步长递增。这意味着累计余额需要达到阈值，有效余额才会改变。例如，如果累计余额为 33.74 ETH，有效余额将为 33 ETH。如果累计余额增加到 33.75 ETH，那么有效余额仍将是 33 ETH。因此，34.25 ETH 的累计余额将对应 34 ETH 的有效余额。

#### **问**：我可以在我的 `0x02` 验证者中追加存入 ETH 吗？

您可以将其他验证者合并到该 `0x02` 验证者中以增加其余额，或者进行新的存款。

#### **问**：如果我进行合并，且我的验证者具有 `0x02` 凭证，且总余额超过了 2048 ETH，ETH 余额会发生什么？

余额将继续在验证者处累积，直到触发下一次部分提款。然而，验证者的最大有效余额为 2048 ETH，超出部分的余额在信标链中将被视为无效。超过 2048 ETH 的部分将在部分提款扫描 (partial withdrawal sweep) 到来时被提取。
