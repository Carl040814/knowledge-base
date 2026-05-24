# 执行层规范 (Execution Layer Specification)

执行层 (Execution Layer, EL) 最初在以太坊 黄皮书 (Yellow Paper) 中进行了规范，因为当时它涵盖了整个以太坊。目前最新的规范是 [EELS Python 规范 (EELS Python spec)](https://ethereum.github.io/execution-specs/)。

> - [黄皮书，Paris 版本 705168a – 2024-03-04](https://ethereum.github.io/yellowpaper/paper.pdf)（注：此版本已过时，未包含合并后的更新）
> - [Python 执行层规范](https://ethereum.github.io/execution-specs/)
> - EIP 提案[请参阅仓库的 Readme](https://github.com/ethereum/execution-specs)

本页面提供了执行层规范的概述、其架构以及 pyspec 的背景信息。

## 状态过渡函数 (State transition function)

从 EELS 的视角来看，执行层专门聚焦于执行 状态过渡函数 (State Transition Function, STF)。该角色主要回答两个核心问题[¹]：

- 是否可以将区块追加到区块链的末尾？
- 状态因此会发生什么改变？

简要概述：
<img src="https://epf.wiki/images/el-specs/stf_eels.png" width="800"/>

上图展示了黄皮书中的区块级状态过渡函数：

$$
\begin{equation}
\sigma_{t+1} \equiv \Pi(\sigma_t, B)
\qquad (2)  \nonumber
\end{equation}
$$

在该公式中，每个符号代表了与区块链状态过渡相关的特定概念：

- $\sigma_{t+1}$ 代表应用当前区块后的**区块链状态**，通常称为“新状态”。
- $\Pi$ 表示 [区块级状态过渡函数](https://github.com/ethereum/execution-specs/blob/0f9e4345b60d36c23fffaa69f70cf9cdb975f4ba/src/ethereum/shanghai/fork.py#L145)，它负责通过应用当前区块中包含的交易，将区块链从一个状态过渡到下一个状态。
- $\sigma_t$ 代表添加当前区块前的**[区块链](https://github.com/ethereum/execution-specs/blob/0f9e4345b60d36c23fffaa69f70cf9cdb975f4ba/src/ethereum/shanghai/fork.py#L73)状态**，也称为“前一状态”。
- $B$ 象征着被发送到执行层进行处理的**[当前区块](https://github.com/ethereum/execution-specs/blob/0f9e4345b60d36c23fffaa69f70cf9cdb975f4ba/src/ethereum/shanghai/fork_types.py#L217)**。

此外，至关重要的一点是，不要将 $\sigma$ 与 Python 规范中定义的 `State` 类混淆。系统的状态并非存储在特定位置，而是通过应用状态折叠函数 (state collapse function) 动态推导出来的。这突显了区块链状态过渡的数学模型与软件规范中实际实现细节之间的概念分离。

<img src="https://epf.wiki/images/el-specs/state.png" width="800"/>

上图中的标识符在黄皮书（Paris 版本）中代表的含义如下：

| 标识符 | 公式编号 | 黄皮书公式 | 说明 |
| :--- | :--- | :--- | :--- |
| 1 | [7](https://ethereum.github.io/yellowpaper/paper.pdf#page=4) | $$TRIE(L_I^*(sigma[a]_s)) \equiv \sigma[a]_s $$ | 在使用函数 $L_I((k,v)) \equiv (KEC(k), RLP(v))$ 映射每个节点后，这给出了账户存储 Trie 的根，即右侧的 $\sigma[a]_s$。左侧的公式是指映射账户存储 $\sigma[a]_s$ 的底层键值对。这是两个不同的对象：左侧的 $\sigma[a]_s$ 和代表根哈希 (root hash) 的右侧 $\sigma[a]_s$。 |
| 2 | | [第 4 页](https://ethereum.github.io/yellowpaper/paper.pdf#page=4)，第 2 段 | 账户状态 $\sigma[a]$ 在黄皮书中进行了描述。 |
| 3 | [10](https://ethereum.github.io/yellowpaper/paper.pdf#page=4) | $$L_s(\sigma) \equiv \{p(a) : \sigma[a] \neq \empty \} $$ | 这是世界状态折叠函数，应用于所有被认为非空的账户。 |
| 4 & 5 | [39](https://ethereum.github.io/yellowpaper/paper.pdf#page=7) | $$TRIE(L_s(\sigma)) = P(B_H)_{H_{stateRoot}} $$ | 该公式将父区块 (parent block) 的状态根区块头定义为由 TRIE 函数给出的根，其中 $P(B_H)$ 是父区块。 |
| 6 | [35b](https://ethereum.github.io/yellowpaper/paper.pdf#page=7) | $$H_{stateRoot} \equiv TRIE(L_s(\Pi(\sigma, B))) $$ | 这给出了当前区块的状态根 (state root)。 |

代码文档中指定的状态过渡函数具体步骤如下：

1. **检索区块头 (Retrieve the Header)**：获取添加到链中的最新区块（称为父区块）的区块头。
2. **超额 Blob Gas 验证 (Excess Blob Gas Validation)**：从父区块头计算超额 Blob Gas (excess blob gas)，并确保其与当前区块头的参数 `excess_blob_gas` 相匹配。
3. **区块头验证 (Header Validation)**：将当前区块的区块头与父区块的区块头进行比较和验证。
4. **Ommer 字段检查 (Ommers Field Check)**：验证当前区块中的 ommers 字段是否为空。注：“ommers”是中性词，用以取代之前使用的“uncles（叔区块）”。
5. **区块执行 (Block Execution)**：执行区块内的交易，产生以下输出：
   - **Gas 消耗量 (Gas Used)**：执行区块中所有交易消耗的总 Gas。
   - **Trie 根 (Trie Roots)**：区块中包含的所有交易和收据的 Trie 根。
   - **日志布隆过滤器 (Logs Bloom)**：区块内所有交易产生日志的布隆过滤器 (bloom filter)。
   - **状态 (State)**：在 python 执行规范中指定，执行所有交易后的状态。
6. **区块头参数验证 (Header Parameters Verification)**：确认执行区块返回的参数存在于区块头中。这包括将状态的根与区块头中的 `state_root` 字段进行比较。
7. **区块追加 (Block Addition)**：如果所有检查均成功，则将区块追加到区块链中。
8. **修剪旧区块 (Pruning Old Blocks)**：从区块链中移除早于最近 255 个区块的旧区块。
9. **错误处理 (Error Handling)**：如果任何验证检查失败，抛出“无效区块 (Invalid Block)”错误。否则，返回 `None`。

## 区块头验证 (Block Header Validation)

区块头验证的过程在黄皮书和 python 规范中得到了严格定义，根据以太坊协议规则验证区块完整性，例如哈希验证、Gas 使用、时间戳准确性等。此验证确保每个区块符合规范中定义并在客户端中实现的以太坊协议。在同步和追加区块期间，验证是区块链的一项不可或缺的功能，通过独立验证当前和历史数据来起作用。

黄皮书中指定的区块头 $H$ 的[有效性 (validity)](https://github.com/ethereum/execution-specs/blob/0f9e4345b60d36c23fffaa69f70cf9cdb975f4ba/src/ethereum/shanghai/fork.py#L269) 采用了一系列标准，以确保每个区块符合以太坊的协议要求。父区块，表示为 $P(H)$，是验证当前区块头 $H$ 所必需的。有效性的关键条件包括：

$$V(H) \equiv H_{gasUsed} \leq H_{gasLimit} \qquad (57a)$$
$$\land$$
$$H_{gasLimit} < P(H)_{H_{gasLimit'}} + floor(\frac{P(H)_{H_{gasLimit'}}}{1024} ) \qquad (57b)$$
$$\land $$
$$H_{gasLimit} > P(H)_{H_{gasLimit'}} - floor(\frac{P(H)_{H_{gasLimit'}}}{1024} ) \qquad (57c)$$
$$\land$$
$$H_{gasLimit} > 5000\qquad (57d)$$
$$\land  $$
$$H_{timeStamp} > P(H)_{H_{timeStamp'}} \qquad (57e)$$
$$\land$$
$$H_{numberOfAncestors} = P(H)_{H_{numberOfAncestors'}} + 1 \qquad (57f)$$
$$\land$$
$$length(H_{extraData}) \leq 32_{bytes} \qquad (57g)$$
$$\land$$
$$H_{baseFeePerGas} = F(H) \qquad (57h)$$
$$\land$$
$$H_{parentHash} = KEC(RLP( P(H)_H )) \qquad (57i) $$
$$\land$$
$$H_{ommersHash} = KEC(RLP(())) \qquad (57j)$$
$$\land$$
$$H_{difficulty} = 0\qquad (57k)$$
$$\land $$
$$H_{nonce} = 0x0000000000000000 \qquad (57l)$$
$$\land$$
$$H_{withdrawalHash} \neq nil \qquad (57n)$$
$$\land$$
$$H_{blobGasUsed} \neq nil \qquad (57o)$$
$$\land$$
$$H_{blobGasUsed} \leq  MaxBlobGasPerBlock_{=786432}  \qquad (57p)$$
$$\land $$
$$H_{blobGasUsed} \% GasPerBlob_{=2^{17}} = 0  \qquad (57q)$$
$$\land $$
$$H_{excessBlobGas} = CalcExcessBlobGas(P(H)_H) \qquad (57r)$$
$$\land $$
$$
\begin{aligned}
CalcExcessBlobGas(P(H)_H) \equiv \nonumber \\
\begin{aligned}
&\begin{cases}
0,  \qquad \text{if} \space P(H)_{blobGasUsed} < TargetBlobGasPerBlock \\
P(H)_{blobGasUsed} - TargetBlobGasPerBlock
\end{cases}
\quad (57s)
\end{aligned}
\end{aligned}
$$
$$\land $$
$$
\begin{aligned}
P(H)_{blobGasUsed} \equiv  P(H)_{H_{excessBlobGas}} + P(H)_{H_{blobGasUsed}} \\
TargetBlobGasPerBlock =  393216
\end{aligned}
\quad (57t)
$$

- **Gas 消耗量 (Gas Usage)**：区块使用的 Gas 量 $H_{gasUsed}$ 绝不能超过 Gas 限制 $H_{gasLimit}$，从而确保交易适应区块的容量限制 (57a)。
- **Gas 限制约束 (Gas Limit Constraints)**：区块的 Gas 限制必须保持在相对于父区块 Gas 限制 ${P(H)_{H_{gasLimit'}}}$ 的指定范围内，允许逐渐变化而非突变 (57b, 57c)。
- **最小 Gas 限制 (Minimum Gas Limit)**：5000 的最小 Gas 限制确保了基础水平的交易处理能力 (57d)。
- **时间戳验证 (Timestamp Verification)**：每个区块的时间戳 $H_{timeStamp}$ 必须大于其父区块的时间戳 $P(H)_{H_{timeStamp'}}$，以确保按时间顺序排列 (57e)。
- **祖先与额外数据 (Ancestry and Extra Data)**：区块通过 $H_{numberOfAncestors}$ 字段维护世系，并将额外数据 $H_{extraData}$ 的大小限制为 32 字节 (57f, 57g)。
- **经济模型合规性 (Economic Model Compliance)**：基础费用 $H_{baseFeePerGas}$ 是根据 EIP-1559 中确立的规则计算的，反映了网络当前对交易处理的需求 (57h)。这与 a, b, c, d & h 共同定义了经济模型的一部分。

### 区块头验证与以太坊经济模型 (Header validation and the Ethereum economic model)

正如 [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) 中所概述的那样，以太坊经济模型引入了一系列旨在增强网络效率和稳定性的机制：

- **针对波动性降低的目标 Gas 限制 (Targeted Gas Limit for Reduced Volatility)**：通过将 Gas 目标值设置为最大 Gas 限制的一半，以太坊旨在减少满区块可能引起的波动，从而确保更具可预测性的交易处理环境。
- **防止不必要的延迟 (Prevention of Unnecessary Delays)**：该模型寻求通过优化交易处理时间来消除给用户带来的不当延迟，从而改善网络上的整体用户体验。
- **稳定区块奖励发放 (Stabilizing Block Reward Issuance)**：区块奖励的发放有助于提高系统的稳定性，为参与者提供更具可预测性的经济格局。
- **可预测的基础费用调整 (Predictable Base Fee Adjustments)**：EIP-1559 引入了一种可预测的基础费用变化机制，该特性对钱包尤为有利。这种可预测性有助于提前准确估算交易成本，简化交易创建流程。
- **基础费用销毁和优先费用 (Base Fee Burn and Priority Fee)**：在此模型下，验证者有权保留优先费用（小费）作为激励，而基础费用则被销毁，实际上将其从流通中移除。这种方法作为对抗以太坊通货膨胀的对策，通过随着时间的推移减少整体供应量来促进更健康的经济环境。

其他检查确保了传统的兼容性和安全性，例如将 ommer（叔区块）哈希和难度字段设置为预定义值，这反映了从工作量证明 (Proof of Work, PoW) 到权益证明 (Proof of Stake, PoS) 的过渡 (57j-57l)。

这些标准构成了以太坊经济模型的一部分，特别受到 EIP-1559 的影响，它引入了动态基础费用 (base fee) 机制。该机制旨在优化网络使用率和费用可预测性，增强用户体验 and 经济稳定性。此外，[EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) 引入了一种新型交易——Blob 交易 (blob transactions)，增强了来自 EIP-1559 的经济模型。

让我们更深入地探索这一点，并试图更好地理解这些公式中发生的事情，这些在 python 规范或黄皮书中并不容易直观看到。

让我们先展开 57h，它在黄皮书中被指定为：

$$
\begin{equation}
F(H) \equiv
\begin{cases}
10^9 & \text{if } H_{number} = F_{London} \nonumber \\
P(H)_{H_{baseFeePerGas}} & \text{if } P(H)_{H_{gasUsed}} = \tau \nonumber \\
P(H)_{H_{baseFeePerGas}} - \nu & \text{if } P(H)_{H_{gasUsed}} < \tau \nonumber \\
P(H)_{H_{baseFeePerGas}} + \nu & \text{if } P(H)_{H_{gasUsed}} > \tau
\end{cases}
\qquad (45)
\end{equation}
$$

$$
\tau \equiv \frac {P(H)_{H_{gasLimit}}}  {\rho} \qquad (46)
$$

$$
\rho \equiv 2 \qquad (47)
$$

$$
\nu^* \equiv
\begin{cases}
\frac{P(H)_{H_{baseFeePerGas}} \times (\tau - P(H)_{H_{gasUsed}})}{\tau} & \text{if } P(H)_{H_{gasUsed}} < \tau \\
\frac{P(H)_{H_{baseFeePerGas}} \times (P(H)_{H_{gasUsed}} - \tau)}{\tau} & \text{if } P(H)_{H_{gasUsed}} > \tau
\end{cases} \qquad (48)
$$

$$
\nu \equiv
\begin{cases}
\left\lfloor \frac{\nu^*}{\xi} \right\rfloor & \text{if } P(H)_{H_{gasUsed}} < \tau \\
\max\left(\left\lfloor \frac{\nu^*}{\xi} \right\rfloor, 1\right) & \text{if } P(H)_{H_{gasUsed}} > \tau
\end{cases} \qquad (49)
$$

$$
\xi \equiv 8 \qquad (50)
$$

| 符号 | 所代表的含义 | 数值 | 说明 |
| :--- | :--- | :--- | :--- |
| $F(H)$ | 每个 Gas 的基础费用 | | 由发送方作为总费用的一部分支付。基础费用最终被执行层销毁，并从系统中移除。 |
| $\nu$ | 基础费用增加或减少的幅度 | | 与父区块的 Gas 消耗量与 Gas 目标值之间的差值成正比。 |
| $\tau$ | Gas 目标值 | $\frac {P(H)_{H_{gasLimit}}}  {\rho}_{= 2}$ | 旨在降低波动性，Gas 目标值设定为 Gas 限制的一半，以缓和每个区块的交易吞吐量。 |
| $\rho$ | 弹性乘数 | 2 | 有助于调整 Gas 目标值，以维持网络响应能力、容量和价格可预测性。 |
| $\xi$ | 基础费用最大变化分母 | 8 | 它控制基础费用的最大变化率，确保逐渐调整。 |

此外，黄皮书对这些对象的类型进行了一些关键定义，这些定义将用于对这些等式进行推理：

首先，它为我们提供了无界的区块限制，即这些限制可以无限延伸：

$$
H_{\text{gasUsed}} , H_{\text{gasLimit}}, H_{\text{baseFeePerGas}} \in \mathbb{N} \qquad (41)
$$

然后，它为我们提供了交易参数的类型。这些被限制在 $2^{256}$ 或大约 $10^{77}$ 的最大值内，这是这些数字可以达到的最大值：

$$T_{\text{maxPriorityFeePerGas}} , T_{\text{maxFeePerGas}}, T_{\text{gasLimit}}, T_{\text{gasPrice}} \in\mathbb{N}_{256}$$

#### 补充见解：自然数的意义 (Additional Insights: The Significance of Natural Numbers)

以太坊协议将区块级参数——如已用 Gas ($H_{gasUsed}$)、Gas 限制 ($H_{gasLimit}$) 和每 Gas 基础费用 ($H_{baseFeePerGas}$)——指定为自然数 $\mathbb{N}$。这一决定绝非凭空做出；它在区块链的底层经济学中嵌入了一层直观的逻辑。

- **自然数与区块链逻辑**

自然数从 0 开始并无限延伸，为理解和操作这些参数提供了一个直接的框架。与在任意两点之间包含不可数无穷大的实数不同，自然数允许精确、离散的步骤——这使得它们成为精度至关重要的区块链交易的理想选择。该属性简化了对操作这些参数的函数的推理，有助于对交易成本和网络容量进行精确计算和预测。

- **简单性与精确性**

考虑递增的简单性：每个自然数都可以被视为 (0 + 1 + 1 + ... + 1) 的和，为智能合约或交易处理中的递增或递减提供了清晰的路径。自然数的这种原子性质，以 0 和后继者 (+ 1) 作为基石，使得在以太坊区块链中构建稳健且可证明的逻辑成为可能，换句话说，自然数使得证明更容易。

- **与实数（小数）的对比**

与实数的无限可分性相反，以太坊经济模型中自然数的离散性质确保了操作保持在可计算的范围内。这种区别对于维持网络效率和安全性至关重要，避免了与处理实数相关的计算复杂性和潜在漏洞。

**交易参数与有界自然数**

此外，以太坊在有界自然数子集 $\mathbb{N}_{256}$ 内指定了交易参数，例如每 Gas 最大优先费用和小费。此边界上限为 $2^{256}$ 或大约 $10^{77}$，在为交易处理允许极宽的数值范围与确保这些数值保持在安全、可管理的限制之内之间取得了平衡。

#### 逐个区块的 Gas 价格动态 (Dynamics of Gas Price Block to Block)

让我们通过探索在从最小可能（5,000 单位）到设定的 Gas 限制的一系列 Gas 使用场景中的影响，深入研究 Gas 价格计算函数的动态。我们的焦点是理解该函数在单个区块的范围内的表现。

我们旨在分析“计算每 Gas 基础费用”函数，这对于理解以太坊的 Gas 定价机制至关重要。以下 R 代码片段说明了该函数的实现：

<img src="https://epf.wiki/images/el-specs/gasused-basefee.png" width="800"/>

图表中的观察结果：

- 该函数表现出阶梯状的线性递增，在中心点处的方差最大。这反映了 Gas 目标值设置为 Gas 限制的一半（在本例中为 15,000 单位）。
- 当基础费用从 100 开始时，基础费用的最大向上变化约为 12.5%，在图表的最右侧观察到。这代表了可能的最大增幅。
- 当基础费用从 100 开始时，基础费用的最大向下变化约为 10%，在图表的最左侧观察到。这代表了可能的最大降幅。
- 精确达到 Gas 目标会导致基础费用增加 1%。稍微超过目标（例如，在 15,000 和 17,000 单位的已用 Gas 之间）仍仅导致 1% 的增加，这说明了该函数围绕目标设计的弹性。

#### 扩展模拟：对 Gas 限制和费用的长期影响 (Extended Simulation: Long-term Effects on Gas Limit and Fee)

在可视化了 Gas 价格计算函数在一定已用 Gas 范围内的即时影响后，让我们来考虑其长期效应。具体而言，这种动态如何在成千上万个区块中影响以太坊网络，特别是在每个区块都达到其 Gas 限制的最大需求条件下？

以下图表模拟了在 100,000 个区块中的此场景，假设存在恒定的最大需求，以预测 Gas 限制和基础费用的演变：

<img src="https://epf.wiki/images/el-specs/gas-limit-max.png" width="800"/>

模拟的观察结果揭示了几个关键见解：

- **基础费用敏感性 (Base Fee Sensitivity)**：以 wei 衡量的基础费用迅速升级，在持续的最大需求下，可能在仅仅 200 个区块内达到 1 ETH。
- **达到上限的潜力 (Potential to Hit Upper Limits)**：在持续的高需求下，基础费用可能会在不到 2,000 个区块内接近其理论最大值。
- **无界的 Gas 限制增长 (Unbounded Gas Limit Growth)**：与基础费用不同，Gas 限制本身不受限制，允许持续增长以适应不断增长的网络需求。
- **市场动态与均衡 (Market Dynamics and Equilibrium)**：现实世界的外部需求增加首先反映在区块超过其 Gas 目标值，导致基础费用上涨。然而，随着 Gas 限制的逐渐增加，Gas 目标值（Gas 限制的一半）也会上升，最终使需求与更高的基础费用达到稳定，从而达成新的均衡。

为了使我们的分析具有前瞻性，我们在更细微的水平上检查模型的底层支撑。具体而言，我们关注改变模型核心常数的影响，特别是弹性乘数 ($\rho$) 和基础费用最大变化分母 ($\xi$)。这些常数预计不会在分叉内发生改变，但可以在未来的协议升级中重新指定：

让我们从 $\xi$ 开始：

<img src="https://epf.wiki/images/el-specs/xi.png" width="800"/>

这是区块之间的快照，就像我们的第一张图一样，它代表了跨协议升级由 $\xi$ 额外参数化的经济模型的极小切片。

$\xi$ 对基础费用的影响：

- **拐点和步宽变异性 (Inflection point & step width variability)**：“拐点”或拐角处在 $\xi$ 发生变化时变得尤为明显，随着 $\xi$ 的增加而变得更宽。因此，增加 $\xi$ 会导致更宽的步长，表明费用调整更为平缓。相反，减少 $\xi$ 会导致更窄的步长和更剧烈的费用变化。
- **敏感性 (Sensitivity)**：在拐点之外，基础费用调整曲线的斜率会发生显着变化。随着 $\xi$ 的减小，我们观察到费用调整率急剧增加，表明敏感性增高。
- **目标范围内的线性趋势 (Linear Trend within Target Range)**：曲线的中心部分，特别是以太坊协议当前 $\xi$ 值的浅绿色线所示，展示了随着交易接近或超过 Gas 目标值，费用调整大多呈现线性趋势。

接下来，我们将注意力转向弹性乘数 ($\rho$)，这是以太坊经济模型中另一个关键常数，它直接影响 Gas 限制调整的灵活性和响应能力。为了理解其影响，我们结合基础费用最大变化分母 ($\xi$) 的变化，探索了 $\rho$ 从 1 到 6 的取值范围。

<img src="https://epf.wiki/images/el-specs/rho-xi.png" width="800"/>

$\rho$ 和 $\xi$ 对基础费用的影响：

- **瞬时分析**：与我们最初的观察类似，此图提供了一个细微的视角，展示了调节 $\rho$ 和 $\xi$ 如何具体塑形每个区块的经济模型行为，尤其是在协议升级的背景下。
- **$\rho$ 的独特影响**：每个子图代表了改变 $\rho$ 值的影响。作为弹性乘数，$\rho$ 显著地移动了基础费用调整曲线中的拐点，突出了其在网络对交易量波动的敏感度方面的作用。
- **$\rho$ 和 $\xi$ 之间的相互作用**：弹性乘数 ($\rho$) 不仅移动了拐点，而且还调节了由于基础费用最大变化分母 ($\xi$) 改变而引起的调整敏感度。这种相互作用强调了以太坊在面对不同需求时维持网络效率和稳定性所保持的微妙平衡。

<img src="https://epf.wiki/images/el-specs/gas-header.png" width="800"/>

#### Blob Gas 价格动态 (Dynamics of Blob Gas Price)

Blob Gas 价格的动态在以下场景中建模，从零开始，在接下来的区块中将每个区块使用的 Gas 以 1000 的恒定因子递增。

- 图 E：说明了 Blob Gas 与其价格之间的关系。所有图表的代码都在附录中。

<img src="https://epf.wiki/images/el-specs/blob-gas-and-price.png" width="800"/>

- 图 F：对数据进行了归一化，以突出显示相对于 Gas 使用的费用动态。
  <img src="https://epf.wiki/images/el-specs/blob-gas-and-price-norm.png" width="800"/>

- 当父区块的 Gas 使用量低于目标值（~400K，对应于大约 400KB 或每区块 3 个 Blob）时，Blob Gas 价格保持在 1。大约 800K 的最大值映射到大约 800KB 或每区块 6 个 Blob。
- 超过目标不会立即影响 Gas 价格，但超额 Gas (excess gas) 开始累积。
- 持续的需求增长导致累积的超额 Gas 超过阈值，从而触发 Gas 价格呈指数级增长，作为一种调节措施。
- 如果前一个区块的 Gas 使用量降至目标以下，则累积的超额 Gas 可以在一个区块中被清除，从而重置调整机制。

## 区块执行过程 (Block Execution Process)

在初始区块头验证之后，区块进入执行阶段 ([apply_body](https://github.com/ethereum/execution-specs/blob/804a529b4b493a61e586329b440abdaaddef9034/src/ethereum/cancun/fork.py#L437))。及早进行区块头检查使状态过渡函数 (STF) 能够潜在地向共识层 (CL) 返回“无效有效载荷 (Invalid Payload)”消息，而无需继续进行区块/交易执行这一计算密集型阶段。

1. **将 `blobGasUsed` 初始化为 0**。这为区块中交易使用的 Gas 设置了起始点。
2. **将 `gasAvailable` 设置为 $H_{gasLimit}$**。这将区块执行的可用 Gas 初始化为区块的 Gas 限制。
3. **初始化其他执行组件**：这包括设置收据的 Trie、提现的 Trie 和区块日志元组（其行为类似于不可变列表），确保可用的 Gas 与区块的 Gas 限制一致。
4. **通过指定信标根地址 BEACON ROOTS ADDRESS 的执行层常量，访问信标区块根合约代码**：
   - 此特性在 Cancun 中引入，并在 [EIP-4788](https://eips.ethereum.org/EIPS/eip-4788) 中有详细说明，它支持将信标链区块根用作构建共识层状态证明的密码学累加器 (cryptographic accumulators)。这为 EVM 提供了一种信任最小化的方式来访问共识层数据，支持诸如质押池 (staking pools)、再质押操作 (restaking operations)、智能合约桥和 MEV 缓解措施等应用。想了解更多，可以观看规范创作者的[介绍视频](https://www.youtube.com/watch?v=GriLSj37RdI)。
5. **构建一个系统交易消息 (System Transaction Message)**：以系统地址 (**System Address**) 为呼叫方，信标根地址 (**BEACON ROOTS ADDRESS**) 为目标，包括 $H_{parentBeaconBlockRoot}$ 和检索到的合约代码。这在 Cancun 中引入了一个“系统合约 (system contract)”，这是一种有状态的智能合约，不同于无状态的预编译合约 (precompiles)，只有系统地址可以插入数据。
6. **设置虚拟机环境并处理消息调用**，将 $H_{parentBeaconBlockRoot}$ 存储在合约's 的存储中，以便交易稍后通过提供时隙的时间戳进行检索。
7. **删除在先前步骤中涉及的空账户**，以清理状态。
8. **处理区块内的交易**：
   - 交易被解码并添加到交易 Trie 中以进行执行。
   - **执行[交易](/wiki/EL/transaction)**：这对区块执行过程至关重要，涉及：
     1. 使用签名组件 $T_v, T_r, T_s$ 恢复交易发送方的地址。
     2. 验证交易的固有有效性 (intrinsic validity)。
     3. 计算有效 Gas 价格 (effective gas price)。
     4. 初始化执行环境。
     5. 在虚拟机内**执行解码后的交易**，包括根据当前状态进行验证、计算 Gas，并在成功后应用状态更改。
9. **处理经信标链验证的验证者提现** ([EIP-4895](https://eips.ethereum.org/EIPS/eip-4895))：
   - 遍历每个[提现](https://github.com/ethereum/execution-specs/blob/119208cf1a13d5002074bcee3b8ea4ef096eeb0d/src/ethereum/shanghai/fork_types.py#L178)，将其添加到 Trie 中。
   - 将提现额度从 Gwei 转换为 Wei，并入账到指定地址。
   - 销毁空提现账户以保持干净的状态。

### 环境初始化 (Environment initialization)

$$
I_{caller} = T_{Sender_{address}}, \nonumber \\
I_{origin} = T_{Sender_{address}}, \nonumber \\
I_{blockHashes} = blockHashes_{Last255}, \nonumber \\
I_{coinbase} = H_{coinbase}, \nonumber \\
I_{number} = H_{number}, \nonumber \\
I_{gaslimit} = Header_{gasLimit} - cumulativeGasUsed, \nonumber \\
I_{baseFeePerGas} = H_{baseFeePerGas}, \nonumber \\
I_{gasPrice} = effectiveGasPrice, \nonumber \\
I_{time} = H_{timeStamp}, \nonumber \\
I_{prevRandao} = H_{prevRandao}, \nonumber \\
I_{state} = state, \nonumber \\
I_{chainId} = H_{chainId}, \nonumber \\
I_{traces} = [], \nonumber \\
I_{excessBlobGas} = excessBlobGas, \nonumber \\
I_{blobVersionedHashes} = T_{blobVersionedHashes}, \nonumber \\
$$

| 变量 | 描述 |
| :--- | :--- |
| $I_{caller}$ | 发起代码执行的地址；通常是交易的发送方。 |
| $I_{origin}$ | 发起此执行上下文的交易的原始发送方地址。 |
| $I_{blockHashes}$ | 最近 255 个区块的哈希集合。 |
| $I_{coinbase}$ | 区块奖励和交易费用的受益人地址。 |
| $I_{number}$ | 当前区块在区块链中的顺序编号。 |
| $I_{gasLimit}$ | 可用于执行交易的最大 Gas 量，扣除当前区块中已使用的 Gas。 |
| $I_{baseFeePerGas}$ | 每单位 Gas 的基础费用，随着区块空间需求而调整的动态参数。 |
| $I_{gasPrice}$ | 有效 Gas 价格，受当前网络状况和交易紧迫性影响。 |
| $I_{time}$ | 标记区块生产时的时间戳，自 Unix 纪元以来的秒数。 |
| $I_{prevRandao}$ | 前一个 RANDAO（随机性）值，有助于信标链区块生产中的熵贡献。 |
| $I_{state}$ | 当前状态，包含所有账户余额、存储和合约代码。 |
| $I_{chainId}$ | 区块链的标识符，确保交易是为特定链签名的。 |
| $I_{traces}$ | 执行追踪的占位符，供未来使用或调试。 |
| $I_{excessBlobGas}$ | 从父区块计算得出，它代表为 Blob 交易分配的盈余 Gas。 |
| $I_{blobVersionedHashes}$ | 附加到当前交易的 Blob 版本化哈希的有序列表。 |

## Gas 记账 (Gas Accounting)

### 固有 Gas 计算 (Intrinsic Gas Calculation)

固有 Gas (Intrinsic Gas) 代表交易开始执行所需的最低 Gas。此成本涵盖了 EVM 所需的计算资源以及与数据传输相关的成本。固有 Gas 从交易的 $T_{gasLimit}$ 中扣除，以在 EVM 内建立执行上下文。

针对 Shanghai 规范进行了更新，固有 Gas 公式如下，其中 $T$ 代表交易 (Transaction)，$G$ 代表 Gas 成本 (Gas Cost)：

$$
g_0 \equiv
$$

$$
\begin{aligned}
G_{\text{initCodeWordCost}} \times
&\begin{cases}
\text{length}, & \text{if length} \mod 32 = 0 \\
\text{length} + 32 - (\text{length} \mod 32), & \text{otherwise}
\end{cases}\\
&\qquad\text{if } \text{CALLDATA} = T_{\text{initializationCode}}
\end{aligned}
$$

$$+$$

$$
\begin{aligned}
&\begin{cases}
\sum_{i \in \{T_{\text{inputData}}\}}
\begin{cases}
G_{\text{txdatazero}} & \text{if } i = 0 \\
G_{\text{txdatanonzero}} & \text{otherwise}
\end{cases}
\end{cases}\\
&\qquad\text{if } \text{CALLDATA} = T_{inputData} \lor T_{initializationCode}
\end{aligned}
$$

$$+$$

$$
\{ \begin{array}{ll}
G_{\text{txcreate}} & \text{if } T_{to} = \emptyset \\
0 & \text{otherwise}
\end{array}
$$

$$+$$

$$
G_{\text{transaction}}
$$

$$+$$

$$
\sum_{j=0}^{ length(T_{accessList}) - 1} \left( G_{\text{accesslistaddress}} + length(T_{accessList}[j]_s) *  G_{\text{accessliststorage}} \right)
$$

#### 固有 Gas 组件 (Intrinsic Gas Components)：

| 组件 | 描述 |
| :--- | :--- |
| $g_0$ | 代表交易的总固有 Gas 成本，涵盖初始代码执行和数据传输。 |
| $G_{\text{transaction}}$ | 每笔交易的基础成本，设定为 21000 Gas。 |
| $T_{\text{initializationCode}}$ | 当 $T_{to} = 0_{\text{Bytes}}$ 时，CALLDATA 被视为 $T_{\text{initializationCode}}$。成本按 32 字节间隔进行规范化。 |
| $T_{inputData}$ and $T_{initializationCode}$ | 统称为 $T_{inputData}$ 和 $T_{initializationCode}$，代表交易的 CallData 参数。如果 $T_{to} \neq 0_{Bytes}$，CALLDATA 将被视为合约入口点的输入。处理 CALLDATA 的 Gas 成本定义为非零字节每字节 16 Gas，零字节每字节 4 Gas，由于处理增加而影响区块大小并可能导致网络延迟。该 Gas 成本模型是基于区块创建率、链增长率和网络延迟的平衡，最初是为工作量证明系统优化的。使该模型适应权益证明仍是一个研究机会和未来优化的领域。这些参数定义为无限大小的字节数组，初始化成本设为每个非零字节 16 Gas，每个零字节 4 Gas。 |
| $G_{\text{txCreate}}$ | 合约创建交易需要额外的 32000 Gas。 |
| $G_{\text{accesslistaddress}}, G_{\text{accessliststorage}}$ | 访问列表中指定的每个地址和存储键的额外 Gas 成本，有利于优化状态访问。 |

### 有效 Gas 价格与优先费用 (Effective Gas Price & Priority Fee)

下面的公式被修改以包含 Blob 交易（$T_{type} = 3$）：

$$ p \equiv effectiveGasPrice \equiv
\begin{aligned}
&\begin{cases}
T_{gasPrice}, & \text{if} \space T_{type} = 0 \lor 1\\
priorityFee + H_{baseFeePerGas} , & \text{if} \space T_{type} = 2 \lor 3
\end{cases}\\
\end{aligned} \qquad (62)
$$

$$ f \equiv priorityFee \equiv
\begin{aligned}
&\begin{cases}
T_{gasPrice} - H_{baseFeePerGas}, & \text{if} \space T_{type} = 0 \lor 1\\
min(T_{maxPriorityFeePerGas} , T_{maxFeePerGas} -  H_{baseFeePerGas}) , & \text{if} \space T_{type} = 2 \lor 3
\end{cases}\\
\end{aligned}
$$

| 变量 | 描述 |
| :--- | :--- |
| effectiveGasPrice | 在交易执行期间，交易签署方每消耗一单位 Gas 将支付的 Wei 数量。 |
| priorityFee | 在交易执行期间，交易受益人每消耗一单位 Gas 将收到的 Wei 数量。 |

### 有效 Gas 费用 (Effective Gas Fee)

$$effectiveGasFee \equiv effectiveGasPrice \times T_{gasLimit} $$
作为预付成本 (upfront cost) 的一部分扣除。

### 总 Blob Gas (Total Blob Gas)

$$totalBlobGas  \equiv  (G_{gasPerBlob = 2^{17}} \times length(T_{blobVersionedHashes}) ) $$

### Blob Gas 价格 (Blob Gas Price)

Blob Gas 价格是通过一个基于网络中产生的超额 Blob Gas 进行调整的公式决定的。公式如下：

$$
blobGasPrice  \\  \approx  \\
factor_{minBlobBaseFee = 1} \times e^{numerator_{excessBlobGas} / denominator_{blobGaspriceUpdateFraction = 3338477}}
$$

- 如果超额 Gas 没有累积，该公式对低于当前最大每区块 Blob Gas（设定为 786432）的任何输入返回 1。
- 然而，当目标在多个区块中被突破时，它开始增加，这导致超额 Blob Gas 参数开始累积，从而触发 Blob Gas 价格呈指数级增长。
- 目标值设定为每区块最大 Blob Gas 的约一半 (393216)，该函数在达到目标的十倍时开始显示增加到 2 的值，此后呈指数级上升。

### Blob Gas 费用 (Blob Gas Fee)

$$blobGasFee \equiv totalBlobGas \times blobGasPrice $$

### 最大 Gas 费用 (Max Gas Fee)

$$
 maxGasFee \equiv
\begin{aligned}
&\begin{cases}
T_{gasLimit} \times  T_{gasPrice}    , & \text{if} \space T_{type} = 0 \lor 1\\
T_{gasLimit} \times  T_{maxFeePerGas}   , & \text{if} \space T_{type} = 2 \\
(T_{gasLimit} \times  T_{maxFeePerGas}) +  maxBlobFee  , & \text{if} \space T_{type} =  3
\end{cases}\\
\end{aligned}
$$

$$
maxBlobFee \equiv
T_{maxFeePerBlobGas} \times totalBlobGas
$$

### 预付成本 (Up-Front Cost)

$$
v_0 \equiv upfrontCost \equiv  effectiveGasFee + blobGasFee
$$

## 交易执行 (Transaction Execution)

以太坊网络内执行交易的过程受交易级状态过渡函数支配：

$$\Upsilon(\sigma_t, T_{index}) \qquad (4)$$

一旦调用 $\Upsilon$，系统首先验证交易的固有有效性。一旦通过验证，[以太坊虚拟机](/wiki/EL/evm) (EVM) 根据交易的指令启动状态修改。

### 交易固有有效性 (Transaction Intrinsic Validity)

交易的固有有效性是通过一系列检查确定的：

$$
\begin{align}
(65)\quad Sender(T) &\neq EMPTY( \sigma ) \nonumber \\
(66)\quad T_n &\equiv \sigma[Sender(T)]_n \nonumber \\
(67)\quad T_g &\le H_l - cumulativeGasUsed \nonumber \\
(68)\quad v_0 &\le \sigma[Sender(T)]_b \nonumber \\
(69)\quad T_{maxFeePerGas} &\ge H_{baseFeePerGas} \nonumber \\
(70)\quad T_{maxFeePerBlobGas} &\ge blobGasPrice \nonumber \\
\end{align}
$$

#### 限制条件 (Constraints)：
* **Sender Exists (65)**: 交易发送方必须是有效且非空的账户。
* **Nonce Match (66)**: 交易 nonce 必须与发送方账户中记录的 nonce 相匹配。
* **Block Gas Limit (67)**: 交易 Gas 限制不能超过当前区块的剩余可分配 Gas 空间。
* **Sufficient Balance (68)**: 发送方的余额必须能够支付预付费用（Gas 费用 + 任何附加的 Blob 费用）。
* **Base Fee Compliance (69)**: 交易的最大 Gas 费用必须等于或高于区块的 `base_fee`。
* **Blob Fee Compliance (70)**: 交易的最大 Blob 费用必须大于或等于区块的 `blob_gas_price`。

## 以太坊虚拟机执行 (EVM Execution)

在交易通过固有验证后，EVM 实例化以运行字节码。EVM 可以通过形式化状态机模型来理解。

### 形式化状态机 (Formal State Machine)

虚拟机可以建模为一个由状态转换规则决定的形式化状态机。其系统状态由两个主要部分组成：
- 长期世界状态 $\sigma$：包含所有以太坊账户的全局、持久状态。
- 瞬时机器状态 $\mu$：在特定执行周期内用于单笔交易的临时执行状态。

机器状态 $\mu$ 定义为一个元组：

$$\mu \equiv (g, pc, memory, activeWords, stack)$$

其中：
* $g \in \mathbb{N}$ 表示当前可用于执行的剩余 Gas。
* $pc \in \mathbb{N}$ 是程序计数器 (program counter)，指向当前正在执行的操作码。
* $memory$ 是一个大小无限的零初始化字节数组，用于临时存储。
* $activeWords \in \mathbb{N}$ 跟踪内存中活动字 (active words) 的数量（32 字节字）。
* $stack$ 是一个大小最大为 1024 元素的 256 位字堆栈。

### 单个执行循环 (Single Execution Cycle)

在每个执行周期开始时，虚拟机会根据当前的程序计数器从合约字节码中检索要执行的下一个操作码：

$$
\begin{align}
w \equiv \text{currentOperation}(\mu, \text{byteCode}) \equiv \nonumber \\
\begin{cases}
\text{byteCode}[\mu_{programCounter}], & \text{if } \mu_{programCounter} < \text{length}(\text{byteCode}) \nonumber \\
\text{STOP}, & \text{otherwise}
\end{cases}
\end{align}
$$

此逻辑通过访问字节码数组中 programCounter 位置的字节来获取当前操作。如果 programCounter 超过字节码的长度，则发出 STOP 操作以停止执行。

考虑黄皮书中作为说明性示例的 add 运算符定义：
$$\mu'_{stackContents}[0] \equiv \mu_{stackContents}[0] + \mu_{stackContents}[1]$$

此表示暗示了堆栈中的左侧加法和移除，类似于队列操作。然而，传统的堆栈操作是从右侧添加和移除项目的。将其转换为基于堆栈的操作：

$$
Add \Rightarrow
$$

$$
x =  Pop(\mu_{stackContents}) \\
y =  Pop(\mu_{stackContents_{itemsRemoved=1}}) \\
result = x + y  \\
Push(\mu_{stackContents_{itemsRemoved=2}}, result)
$$

$$
\Rightarrow \mu_{stackContents^{itemsAdded_{\alpha}=1}_{itemsRemoved_{\delta}=2}}
$$

当转换为代码时，符号 $\mu_{s}[number]$ 转换为 $\mu_{stackContents}[stackLength - 1 - number]$，这符合传统对堆栈操作的理解。

黄皮书优雅地记录了基于堆栈的操作，并提供了一个框架来解释执行周期内的这些操作。它指定堆栈项目是从数组的左数较低索引部分进行操作的，未受影响的项目保持恒定：

$$
\begin{align}
\Delta \equiv \alpha^{itemsAdded}_w - \delta^{itemsRemoved}_w \quad (160) \nonumber\\
& \nonumber \\
\mu'_{stackContents}.length \equiv \mu_{stackContents}.length + \Delta \quad (161) \nonumber \\
& \nonumber \\
\forall x \in [\alpha^{itemsAdded}_w , \mu'_{stackContents}.length) : \mu'_{stackContents}[x] \equiv \mu_{stackContents}[x - \Delta] \quad (162) \nonumber
\end{align}
$$

公式 162 表明，对于指定范围内的每个 x，修改后的堆栈镜像了位置 $x - \Delta$ 处的原始堆栈，从而有效地跟踪了操作后堆栈项的原始位置。例如，将一个项目 [2] 添加到现有堆栈 [10] 中会得到 [2,10]，其中原始项目的最新位置与 $x = Delta$ 对齐，从而在操作后维持了堆栈顺序的完整性。

#### 单个执行周期

$$
O((\sigma, \mu, A, I)) \equiv (\sigma', \mu', A', I) \quad (159)\\
$$

其中 $O$ 代表执行周期，封装了状态机内单个周期的结果。此周期可以修改 $\mu$ 的所有组件，并对 $\mu_{gas}$ 和 $\mu_{programCounter}$ 的变化做出了明确的规定：

##### 单个执行周期的结果程序计数器 (Resultant Program Counter of a Single Execution Cycle)

以下公式概述了执行周期如何一次处理一条指令：

$$
\mu'_{programCounter} \equiv
\begin{cases}
J_{JUMP}(\mu) \space \text{if }  currentOperation = JUMP \\
J_{JUMP1}(\mu) \space \text{if }  currentOperation = JUMP1 \\
N(\mu_{programCounter}, currentOperation) \space \text{otherwise}
\end{cases}
$$

$$
\text{Where, } \\
NextValidInstruction_N(i_{=programCounter}, w_{=currentOperation}) \equiv \\
\begin{cases}
&\\
programCounter + NumberOfBytes(currentOperation + Data_{currentOperation}) - NumberOfBytes(Operation_{PUSH1}) + 2 \\
\qquad  \text{if } currentOperation \in [PUSH1,PUSH32] \\
&\\
programCounter + 1 \space \text{otherwise}
\end{cases}
$$

- 在这里，如果操作是 $JUMP$，$J_{JUMP}$ 函数会将程序计数器设置为堆栈顶部的值。
- 对于 $JUMP1$ 操作，$J_{JUMP1}$ 函数仅在堆栈中的相邻值不为 0 时将程序计数器设置为堆栈顶部的值。否则，它将程序计数器增加 1。如果当前操作既不是 $JUMP$ 也不是 $JUMP1$，程序计数器将由 NextValidInstruction 函数递增。

NextValidInstruction 函数决定，如果当前操作在所有 PUSH 操作的范围内，我们将程序计数器递增到紧随当前操作字节之后的字节，并考虑与该操作相关联的数据。该数据可以从 1 到 32 字节不等，具体取决于特定的 PUSH 操作。如果该操作不是 PUSH 操作，我们只需将程序计数器递增 1，前进到代码的下一个字节。此过程突显了 PUSH 指令负责将数据从代码加载到堆栈上。

当程序计数器执行跳转操作时，它必须靶向有效的跳转目的地。$ValidJumpDestinations_{D}$ 函数指定了所有有效跳转目的地的集合。

$$
ValidJumpDestinations_{D}(byteCode) \equiv \\
ValidJumpDestinations_{D_J}(byteCode,index) \equiv \\
\begin{cases}
\{\}  \quad \text{  if } index \geq Length(byteCode) \\
&\\
{i} \cup ValidJumpDestinations_{D_J}(byteCode,NextValidInstruction(index, byteCode[index])) \\
\space \qquad \text{if }  byteCode[index] = JUMPDEST \\
&\\
ValidJumpDestinations_{D_J}(byteCode,NextValidInstruction(index, byteCode[index])) \space \text{otherwise}
\end{cases}
$$

这表明如果该索引处的字节码对应于 JUMPDEST 操作，我们就在集合中包含该索引。我们通过使用由 $NextValidInstruction$ 函数确定的索引递归调用 $ValidValidJumpDestinations_{D_J}(byteCode, index)$ 函数来继续添加这些索引。

##### 单个执行周期中的结果 Gas 消耗 (Resultant Gas Consumption in a Single Execution Cycle)

$$
\mu'_{gas} \equiv \mu_{gas} - C_{gasCost}(\sigma, \mu, AccruedSubState, Environment_I)
$$

Gas 成本函数虽然不是特别复杂，但包含了针对不同操作的各种情况。它在黄皮书的附录 H 中得到了简洁的定义。从本质上讲，它通过将当前操作的成本与循环前后内存中活动字的成本差异（内存扩展成本）相加，来计算当前周期的总成本。

不同的客户端处理 Gas 成本的方式不同。在 PySpec 中，各种类型的成本处理都集成到了操作中，而在 Geth 中，Gas 成本在操作执行之前进行处理。此外，Geth 还区分了用于内存扩展的[动态](https://github.com/ethereum/go-ethereum/blob/7bb3fb1481acbffd91afe19f802c29b1ae6ea60c/core/vm/interpreter.go#L257)成本以及与操作基础成本相关联的[恒定](https://github.com/ethereum/go-ethereum/blob/7bb3fb1481acbffd91afe19f802c29b1ae6ea60c/core/vm/interpreter.go#L224) Gas。两种类型的成本均使用 [UseGas](https://github.com/ethereum/go-ethereum/blob/7bb3fb1481acbffd91afe19f802c29b1ae6ea60c/core/vm/contract.go#L161) 函数进行扣除。

#### 程序执行 $\Xi$ (Program Execution $\Xi$) :

$$(\sigma^{'}_{resultantState}, gas_{remaining}, A^{resultantAccruedSubState}, \omicron^{Output})$$ $$\equiv \Xi(\sigma,gas,A^{accruedSubState}, Environment_I)$$

程序执行函数是由函数 X 正式定义的，唯一的区别是 $\Xi$ 调用了 X，并在输出元组中排除了 $Environment_I$。

##### 递归执行函数 X

X 编排了整个代码的执行。这通常由客户端实现为循环遍历代码的主循环。然而，其定义是递归的：

$$
X((\sigma,\mu,AccruedSubState,Environment_I)) \equiv  \nonumber \\
\begin{cases}
&\\
(\empty, \mu, AccruedSubState, Environment_I) \\ \qquad \text{if } Z_{exceptionalHalting}(\sigma, \mu, AccruedSubState, Environment_I) \\
&\\
(\empty, \mu', AccruedSubState, Environment_I, output ) \\ \qquad \text{if }  currentOperation_w = REVERT \\
&\\
O(\sigma, \mu', AccruedSubState, Environment_I) . output  \\ \qquad \text{if }  output  \neq \empty \\
&\\
X(O(\sigma, \mu', AccruedSubState, Environment_I)) \\ \qquad \text{otherwise}
&\\
\end{cases}
$$

$$
\text{Where}, \\
\mu'_{returnData} \equiv \mu'_{outputFromNormalHalting} \equiv output \equiv H_{normalHaltingFunction}(\mu,Environment_I)
$$

$$
O(\sigma,\mu,A,I).output \equiv O(\sigma,\mu,A,I,output)
$$

$$
\mu' \equiv \mu \text{ except:} \\
\mu'_{gas} \equiv \mu_{gas} - C_{gasCostFunction}(\sigma,\mu,A,I) \\
\mu'_{activeWordsInMemory} \equiv 32 * M_{memoryExpansionForRangeFunction}(\mu_{activeWordsInMemory}, \mu_{stackContents}[0], \mu_{stackContents}[1])
$$

1. 如果满足异常停机 (Exceptional Halting) 的条件，返回一个由空状态、机器状态、累加子状态、环境和空输出组成的元组。
2. 如果当前操作是 $REVERT$，则返回一个由空状态、扣除 Gas 后的机器状态、累加子状态、环境和机器输出组成的元组。
3. 如果机器输出不为空，执行迭代器函数 $O$ 消耗该输出。

- 例如，如果当前操作是系统操作，如 CALL、CALLCODE、[DELEGATECALL](https://github.com/ethereum/execution-specs/blob/9c24cd78e49ce6cb9637d1dabb679a5099a58169/src/ethereum/cancun/vm/instructions/system.py#L542) 或 STATICCALL，这些调用会调用[通用调用函数](https://github.com/ethereum/execution-specs/blob/9c24cd78e49ce6cb9637d1dabb679a5099a58169/src/ethereum/cancun/vm/instructions/system.py#L267)，建立一条新消息和一个子 EVM 进程。此进程的输出随后被[写回父 EVM 进程的内存中](https://github.com/ethereum/execution-specs/blob/9c24cd78e49ce6cb9637d1dabb679a5099a58169/src/ethereum/cancun/vm/instructions/system.py#L325)，实际上在 $O$ 的一次迭代中消耗了输出，这可能会在下一次迭代中被使用。

4. 在所有其他情况下，我们只需继续递归调用迭代器函数。简而言之，这意味着我们继续主解释器循环。

##### 正常停机 H (Normal Halting H)

$H_{normalHaltingFunction}$ 定义了 EVM 在正常情况下的停机行为：

$$
H_{normalHaltingFunction}(\mu, Environment_I) \equiv
$$

$$
\begin{cases}
H_{RETURN}(\mu) & \text{if } \text{currentOperation} \in \{ \text{RETURN}, \text{REVERT} \} \\
() & \text{if } \text{currentOperation} \in \{ \text{STOP}, \text{SELFDESTRUCT} \} \\
\empty & \text{otherwise}
\end{cases}
$$

其中：

- $H_{RETURN}(\mu) \equiv \mu'$
- $\Delta_{expansion}$ 计算如下：
  - $\Delta_{expansion} \equiv 32 \times M_{memoryExpansionForRangeFunction}(length(\mu_{memoryContents}), startPos, memorySize)$
  - $\Delta_{expansion} \in \mathbb{N}$
- $\mu'$ 定义为：
  - $\mu' \equiv \mu$ 除了：
    - $\mu'_{memoryContents} \equiv \mu_{memoryContents} + [0_{\text{word}_{256\text{bit}}} ... 0_{\text{word}_{256\text{bit}}}]_{\text{length}=\Delta_{expansion}}$
    - $\mu'_{output} \equiv \mu'_{memoryContents}[startPos : startPos + memorySize]$
    - $\mu'_{gas} \equiv \mu_{gas} - \text{memoryExpansionCost}$
    - $\mu'_{running} \equiv false$

其中：

- $startPos \equiv  \mu_{stackContents}[0]$
- $memorySize \equiv  \mu_{stackContents}[1]$

函数 $M_{memoryExpansionForRangeFunction}(s,f,l)$ 确定容纳指定范围所需的内存扩展：

$$
M_{memoryExpansionForRangeFunction}(s,f,l) \equiv
$$

$$
\begin{cases}
S & \text{if } l = 0 \\
\text{max}(s, \lceil (f + l) / 32 \rceil) & \text{otherwise}
\end{cases}
$$

本质上，$H_{normalHaltingFunction}$ 首先根据堆栈最顶端的两项设置输出的起始索引和长度。如果需要内存扩展来容纳输出，它会相应地扩展内存，必要时产生内存扩展成本。最后，它将 EVM 的输出设置为指定的内存范围。

##### 异常停机 Z (Exception Halting Z)

### $T$ 执行阶段 4：临时状态 $\sigma_p$ (Provisional State $\sigma_p$)

TODO

### $T$ 执行阶段 5：预最终状态 $\sigma^*$ (Pre-Final State $\sigma^*$)

TODO

### $T$ 执行阶段 6：最终状态 $\sigma'$ (Final State $\sigma'$)

TODO

## 区块整体有效性 (Block Holistic Validity)

区块整体有效性指作为一个单一的原子状态过渡的区块的正确性，这是通过组合所有交易的有序执行并验证其聚合效应是否与区块头中声明的承诺一致而获得的。

尽管黄皮书通过不同的函数指定了交易执行和区块有效性，但区块整体有效性是由它们的组合产生的，并通过在区块级别重建和验证执行结果而得到正式确立。

---

### 状态初始化 (State Initialization)

区块的执行从派生自父区块的初始状态开始。函数 $\Gamma$ 将一个区块映射到其初始执行状态：

$$
\Gamma(B) \equiv 
\begin{cases} 
\sigma_0 & \text{if } P(B_H) = \emptyset \\
\sigma_i \text{ such that } \text{TRIE}(LS(\sigma_i)) = P(B_H)_H & \text{otherwise}
\end{cases}
$$

| 符号 | 描述 |
| :--- | :--- |
| $\sigma_0$ | 创世状态。 |
| $P(B_H)_H$ | 父区块的状态根。 |
| $LS$ | 状态映射函数（世界状态）。 |

这确保了所有节点均从同一个密码学承诺状态开始执行。

---

### 区块过渡函数 (Block Transition Function)

令 $B_T$ 表示区块中交易的有序列表。交易按顺序执行，其中每笔交易都在其前驱交易所产生的状态上运行。这种顺序执行由区块过渡函数 $\Pi$ 捕获：

$$\Pi(\sigma, B) \equiv \sigma'$$

其中 $\sigma'$ 是通过按顺序应用所有交易（包括在区块级别定义的任何执行后状态过渡）获得的最终后交易状态。因此，交易有效性与上下文相关，并且对顺序和累积效应非常敏感。

### Gas 累加与收据 (Gas Accumulation and Receipts)

每笔交易执行都会产生一份包含执行状态、日志和累计 Gas 使用量的收据。令 $R[n]$ 表示执行第 $n$ 笔交易后使用的累计 Gas。Gas 累加递归定义如下：

$$
R[n] \equiv 
\begin{cases} 
0 & \text{if } n < 0 \\
\Upsilon^g(\sigma[n-1], B_T[n]) + R[n-1] & \text{otherwise}
\end{cases}
$$

其中 $\Upsilon^g$ 提取在状态 $\sigma[n-1]$ 下执行交易 $B_T[n]$ 消耗的 Gas。

---

### 承诺验证 (Commitment Verification)

交易收据的有序序列和最终世界状态使用 Merkle-Patricia Trie 进行承诺。仅当这些计算出的根与区块头匹配时，区块才有效：

1. **收据根 (Receipts Root)**：$B_{H_r} = \text{TRIE}(LS(R))$
2. **状态根 (State Root)**：$B_{H_s} = \text{TRIE}(LS(\sigma'))$

**有效性要求 (Validity Requirements)**：
* 计算出的收据根必须与区块头匹配。
* 最终状态根必须与区块头匹配。
* 累计 Gas 使用量必须符合区块 Gas 限制限制 ($R[n] \le B_{H_l}$)。

区块有效性是**原子性**的。区块级过渡函数 $\Phi$ 将初始状态和区块映射到完整的区块结果。当且仅当所有重建、执行、累加和承诺检查都成功时，区块才被接受。对于区块内的交易，不存在部分接受的概念。

---

**实现参考 (Implementation Reference)**：
上述语义是基于 [以太坊黄皮书](https://ethereum.github.io/yellowpaper/paper.pdf)：
- **第 12 节**：区块最终化
- **第 12.1 节**：执行提现
- **第 12.2 节**：交易验证
- **第 12.3 节**：状态验证

### Gas 记账示例 (Gas Accounting Examples)

到目前为止，我们已经讨论了在各种场景下合并后的执行层 Gas 机制。让我们通过一些示例将它们结合起来。

注：每种交易类型都有不同的参数和费用处理行为。

### 示例 1：简单的 ETH 转账 (Simple ETH Transfer)
#### 支持的交易类型
- **类型 0 (Type 0)**：传统交易
- **类型 1 (Type 1)**：传统 + 访问列表 ([EIP-2930](https://eips.ethereum.org/EIPS/eip-2930))
- **类型 2 (Type 2)**：EIP-1559 交易 ([EIP-1559](https://eips.ethereum.org/EIPS/eip-1559))

#### 交易参数（由发送方定义）

| 交易类型 | 参数 | 描述 |
| :--- | :--- | :--- |
| 类型 0 / 1 / 2 | `gasLimit` | 交易可消耗的最大 Gas |
| 类型 0 / 1 | `gasPrice` | 支付给提议者的全额 Gas 价格 |
| 类型 2 | `maxFeePerGas` | 最大总费用每单位 Gas（包括基础费用 + 优先费用） |
| 类型 2 | `maxPriorityFeePerGas` | 鼓励区块包含的可选小费 |

#### 区块参数（由协议定义）

| 交易类型 | 参数 | 描述 |
| :--- | :--- | :--- |
| 类型 2 | `baseFee` | 动态每单位基础 Gas 价格（由协议销毁） |

#### 预付预留额 (Upfront Reservation)
此时，交易已准备好在区块内进行处理。最初，预留一部分预付费，即从发送方扣除。
| 交易类型 | 公式 |
| :--- | :--- |
| 类型 0 / 1 | `gasLimit × gasPrice` |
| 类型 2 | `gasLimit × maxFeePerGas` |

#### 执行阶段 (Execution Phase)
在初始扣除之后，确定交易的执行成本，并将 Gas 销毁、奖励给提议者或退还给发送方。

| 交易类型 | 有效 Gas 价格 | 实际成本 |
| :--- | :--- | :--- |
| 类型 0 / 1 | `gasPrice` | `gasUsed × gasPrice` |
| 类型 2 | `baseFee + min(maxPriorityFeePerGas, maxFeePerGas − baseFee)` | `gasUsed × effectiveGasPrice` |

注：
- 对于类型 0/1，全额费用直接支付给提议者。
- 对于类型 2，`baseFee` 被销毁，小费 `min(maxPriorityFeePerGas, maxFeePerGas - baseFee)` 归提议者所有。`effectiveGasPrice` 确保总 Gas 成本保持在 `maxFeePerGas` 之内，如果 `baseFee` 很高，则可能会减少小费。

退款金额通过 `reserved − actualCost` 计算，并退还给发送方。

### 示例 2：Blob 交易 (Blob Transaction)

携带 Blob 的交易既支付通常的 EVM Gas 费用，也支付单独的用于大数据 Blob 的 Blob Gas 费用。请注意，EIP-1559 之前的交易类型没有 Blob。在此示例中，我们将仅讨论与 Blob 相关的费用。

#### Blob 交易类型
- **类型 3 (Type 3)**：EIP-4844 交易 ([EIP-4844](https://eips.ethereum.org/EIPS/eip-4844))

#### 交易参数
- `blobVersionedHashes` – 标识每个数据 Blob。
- `totalBlobGas` – 计算为 `GasPerBlob × numberOfBlobs`。
- `maxFeePerBlobGas` – 发送方将支付的最大每 Blob Gas 单位的 Gwei 数量。

#### 区块参数
- `blobGasPrice` – 动态的每区块 Blob Gas 单位价格。

最初，预留一部分预留额，即从发送方扣除。
- `reserved_blob  = totalBlobGas × maxFeePerBlobGas`

#### 执行成本
- `blobFee = totalBlobGas × blobGasPrice` 并由协议完全销毁。

#### 退款给发送方的计算
- `refund_blob = reserved_blob − blobFee` 并退还给发送方。

这些示例应有助于理解交易生命周期中 Gas 是如何处理的。

## 附录 (Appendix)

### 代码 A (Code A)

```R
##imports

library(plotly)
library(dplyr)

## values for xi and rho
## this is how '<-' assignment works in R

ELASTICITY_MULTIPLIER <- 2
BASE_FEE_MAX_CHANGE_DENOMINATOR <- 8

## Slightly modified function from the spec

calculate_base_fee_per_gas <- function(parent_gas_limit, parent_gas_used, parent_base_fee_per_gas, max_change_denom = BASE_FEE_MAX_CHANGE_DENOMINATOR , elasticity_multiplier = ELASTICITY_MULTIPLIER) {

  #  %/% == // (in python) == floor

  parent_gas_target <- parent_gas_limit %/% elasticity_multiplier
  if (parent_gas_used == parent_gas_target) {
    expected_base_fee_per_gas <- parent_base_fee_per_gas
  } else if (parent_gas_used > parent_gas_target) {
    gas_used_delta <- parent_gas_used - parent_gas_target
    parent_fee_gas_delta <- parent_base_fee_per_gas * gas_used_delta
    target_fee_gas_delta <- parent_fee_gas_delta %/% parent_gas_target
    base_fee_per_gas_delta <- max(target_fee_gas_delta %/% max_change_denom, 1)
    expected_base_fee_per_gas <- parent_base_fee_per_gas + base_fee_per_gas_delta
  } else {
    gas_used_delta <- parent_gas_target - parent_gas_used
    parent_fee_gas_delta <- parent_base_fee_per_gas * gas_used_delta
    target_fee_gas_delta <- parent_fee_gas_delta %/% parent_gas_target
    base_fee_per_gas_delta <- target_fee_gas_delta %/% BASE_FEE_MAX_CHANGE_DENOMINATOR
    expected_base_fee_per_gas <- parent_base_fee_per_gas - base_fee_per_gas_delta
  }
  return(expected_base_fee_per_gas)
}
```

在 R 中定义模型后，我们通过在已用 Gas 场景范围内模拟该函数来继续：

```R
parent_gas_limit <- 30000  # Fixed for simplification

## lets see the effect on 100 to see the percentage effect this function has on fee
parent_base_fee_per_gas <- 100

## note gas used can not go below the minimum limit of 5k ,
## therefore we can just count from 5k to 30k by ones for complete precision

seq_parent_gas_used <- seq(5000, parent_gas_limit, by = 1) # creates a vector / column

## add the vector / column to the data frame

data <- expand.grid(parent_gas_used = seq_parent_gas_used)

## apply the function we created above and collect it in a new column

data$expected_base_fee <- mapply(calculate_base_fee_per_gas, parent_gas_limit, data$parent_gas_used, parent_base_fee_per_gas)
```

准备工作已完成，现在我们通过绘制散点图进行绘制和观察，这将揭示该函数在给定约束范围内的形状。

```R
fig <- plot_ly(data, x = ~parent_gas_used, y = ~expected_base_fee, type = 'scatter', mode = 'markers')  # scatter plot

## %>% is a pipe operater from dplyr , used extensively in R codebases it's like the pipe | operator used in shell

fig <- fig %>% layout(xaxis = list(title = "Parent Gas Used"),
                      yaxis = list(title = "Expected Base Fee "))

## display the plot
fig
```

### 代码 B (Code B)

```r
library(forcats)
library(ggplot2)
library(scales)
library(viridis)

## Initial parameters
initial_gas_limit <- 30000000
initial_base_fee <- 100
num_blocks <- 100000

## Sequence of blocks
blocks <- 1:num_blocks

max_natural_number <- 2^256

## Calculate gas limit for each block
gas_limits <- numeric(length = num_blocks)
expected_base_fee <- numeric(length = num_blocks)
gas_limits[1] <- initial_gas_limit
expected_base_fee[1] <- initial_base_fee

for (i in 2:num_blocks) {

   # apply max change to gas_limit at each block
    gas_limits[i] <- gas_limits[i-1] + gas_limits[i-1] %/% 1024


  # Check if the previous expected_base_fee has already reached the threshold
  if (expected_base_fee[i-1] >= max_natural_number) {
    # Once max_natural_number is reached or exceeded, stop increasing expected_base_fee
    expected_base_fee[i] <- expected_base_fee[i-1]
  } else {
    # Calculate expected_base_fee normally until the threshold is reached
    expected_base_fee[i] <- calculate_base_fee_per_gas(gas_limits[i-1], gas_limits[i], expected_base_fee[i-1])
  }
}

## Create data frame for plotting
data <- data.frame(Block = blocks, GasLimit = gas_limits, BaseFee = expected_base_fee)

## Saner labels
label_custom <- function(labels) {
  sapply(labels, function(label) {
    if (is.na(label)) {
      return(NA)
    }
    if (label >= 1e46) {
      paste(format(round(label / 1e46, 2), nsmall = 2), "× 10^46", sep = "")
    } else if (label >= 1e12) {
      paste(format(round(label / 1e12, 2), nsmall = 2), "T", sep = "")  # Trillions
    } else if (label >= 1e9) {
      paste(format(round(label / 1e9, 1), nsmall = 1), "Billion", sep = "")  # Million
    } else if (label >= 1e6) {
      paste(format(round(label / 1e6, 1), nsmall = 1), "Mil", sep = "")  # Million
    } else if (label >= 1e3) {
      paste(format(round(label / 1e3, 1), nsmall = 1), "k", sep = "")  # Thousand
    } else {
      as.character(label)
    }
  })
}

## Bin the ranges we want to observe
data_ranges <- data %>%
  mutate(Range = case_when(
    Block <= 1000 ~ "1-1000",
    Block <= 10000 ~ "1001-10000",
    Block <= 100000 ~ "10001-100000"
  ))

## Rearrange the bins to control where the plots are displayed
data_ranges$Range <- fct_relevel(data_ranges$Range, "1-1000", "1001-10000", "10001-100000")

## Grammar of graphics we can just + the features we want in the plot
plot <- ggplot(data_ranges, aes(x = Block, y = GasLimit, color = BaseFee)) +
  geom_line() +
  facet_wrap(~Range, scales = "free") +  # Using free to allow each facet to have its own x-axis scale
  labs(title = "Gas Limit Over Different Block Ranges",
       x = "Block Number",
       y = "Gas Limit") +
  scale_x_continuous(labels = label_custom) +  # Use custom label function for x-axis
  scale_y_continuous(labels = label_custom) +  # Use custom label function for y-axis
  scale_color_gradientn(colors = viridis(8), trans = "log10",
                        breaks = c(1e3, 1e10, 1e20, 1e40, 1e60, 1e76),
                        labels = c("100", "10^10", "10^20", "10^40", "10^60", "10^76")) +
  theme_bw()

## To view
plot

## Save to file
ggsave("plot_gas_limit.png", plot, width = 7, height = 5)
```

### 代码 C (Code C)

```r
## we are observing the effects of this parameter
## it's set at 8 but lets see its effect in the range of [2,4, .. ,8, .. ,12]
seq_max_change_denom <- seq(2, 12, by = 2)

parent_gas_limit <- 3 * 10^6
seq_parent_gas_used <- seq(5000, parent_gas_limit, by = 100)

parent_base_fee_per_gas <- 100

data <- expand.grid( parent_gas_used = seq_parent_gas_used, base_fee_max_change_denominator = seq_max_change_denom)

data$expected_base_fee <- mapply(calculate_base_fee_per_gas, parent_gas_limit, data$parent_gas_used, parent_base_fee_per_gas, data$  base_fee_max_change_denominator)
```

数据准备完毕，现在进行绘制：

```r
plot <- ggplot(data, aes(x = parent_gas_used, y = expected_base_fee, color = as.factor(base_fee_max_change_denominator))) +
    geom_point() +
    scale_color_brewer(palette = "Spectral") +
    theme_minimal() +
    labs(color = "Base Fee Max Change Denominator") +
    theme_bw()

plot
```

### 代码 D (Code D)

```r
seq_elasticity_multiplier <- seq(1, 6, by = 1)
seq_max_change_denom <- seq(2, 12, by = 2)

parent_gas_limit <- 3 * 10^6
seq_parent_gas_used <- seq(5000, parent_gas_limit, by = 500)

parent_base_fee_per_gas <- 100

data <- expand.grid( parent_gas_used = seq_parent_gas_used, base_fee_max_change_denominator = seq_max_change_denom, elasticity_multiplier = seq_elasticity_multiplier)

data$expected_base_fee <- mapply(calculate_base_fee_per_gas, parent_gas_limit, data$parent_gas_used, parent_base_fee_per_gas, data$base_fee_max_change_denominator, data$  elasticity_multiplier)

plot <- ggplot(data, aes(x = parent_gas_used, y = expected_base_fee, color = as.factor(base_fee_max_change_denominator))) +
    geom_point() +
    facet_wrap(~elasticity_multiplier) +  #  we break the plots out by the this facet
    scale_color_brewer(palette = "Spectral") +
    theme_minimal() +
    labs(color = "Base Fee Max Change Denominator") +
    theme_bw()

ggsave("rho-xi.png", plot, width = 14, height = 10)
```

### 代码 E (Code E)

```r
library(ggplot2)
library(tidyr)

## fake exponential or taylor series expansion function
fake_exponential <- function(factor, numerator, denominator) {
    i <- 1
    output <- 0
    numerator_accum <- factor * denominator
    while(numerator_accum > 0){
      output <- output + numerator_accum
      numerator_accum <- (numerator_accum * numerator) %/% (denominator * i)
      i <- i + 1
    }
    output %/% denominator
}

## Blob Gas Target
target_blob_gas_per_block <- 393216

## Blob Gas Max Limit
max_blob_gas_per_block <- 786432

 # Used in header Verificaton
 calc_excess_blob_gas <- function(parent_excess_blob_gas, parent_gas_used) {
   if (parent_gas_used  + parent_excess_blob_gas < target_blob_gas_per_block) {
     return(0)
   } else {
     return(parent_excess_blob_gas + parent_gas_used - target_blob_gas_per_block)
   }
 }

## This is how EL determines the Blob Gas Price
cancun_blob_gas_price <- function(excess_blob_gas) {
  fake_exponential(1, excess_blob_gas, 3338477)
}

## we got from zero to Max each step increasing by 1000
parent_gas_used <- seq(0, max_blob_gas_per_block, by = 1000)
## A column of the same Length
excess_blob_gas <- numeric(length = length(parent_gas_used))
excess_blob_gas[1] <- 0

## We get the T+1(time + 1) excess gas by using values from before
for (i in 2:length(parent_gas_used)) {
  excess_blob_gas[i] <- calc_excess_blob_gas(excess_blob_gas[i - 1],
                                             parent_gas_used[i - 1])
}

data_blob_price <- expand.grid(parent_gas_used = parent_gas_used)
data_blob_price$excess_blob_gas <- excess_blob_gas

## Apply the EL gas price function
data_blob_price$  blob_gas_price <- mapply(cancun_blob_gas_price,
                                         data_blob_price$excess_blob_gas)

## Each row represents a block
data_blob_price$BlockNumber <- seq_along(data_blob_price$parent_gas_used)

## we collapse the 3 columns into 1 Parameter Column
data_long <- pivot_longer(data_blob_price,
                          cols = c(parent_gas_used,
                                   excess_blob_gas,
                                   blob_gas_price),
                          names_to = "Parameter",
                          values_to = "Value")

ggplot(data_long, aes(x = BlockNumber, y = Value)) +
  geom_line() +
  facet_wrap(~ Parameter, scales = "free_y") +   # We break the charts out based on the Parameter Column
  theme_minimal() +
  scale_y_continuous(labels = scales::label_number()) +
  labs(title = "Dynamic Trends in Blob Gas Consumption & Price Over Time",
       x = "Block Number",
       y = "Parameter Value") +
  geom_text(data = subset(data_long, Parameter == "blob_gas_price" &
                            BlockNumber == min(BlockNumber)),
            aes(label = "blobGasPrice = 1", y = 0),
            vjust = -1, hjust = -0.1, size = 3)
```

### 代码 F (Code F)

```r
normalize <- function(x) {
  return((x - min(x)) / (max(x) - min(x)))
}

data_blob_price$parent_gas_used_normalized <- normalize(data_blob_price$parent_gas_used)
data_blob_price$excess_blob_gas_normalized <- normalize(data_blob_price$excess_blob_gas)
data_blob_price$blob_gas_price_normalized <- normalize(data_blob_price$blob_gas_price)

ggplot(data_blob_price, aes(x = BlockNumber)) +
  geom_line(aes(y = parent_gas_used_normalized, color = "Parent Gas Used")) +
  geom_line(aes(y = excess_blob_gas_normalized, color = "Excess Blob Gas")) +
  geom_line(aes(y = blob_gas_price_normalized, color = "Blob Gas Price")) +
  theme_minimal() +
  labs(title = "Normalized Trends Over Blocks", x = "Block Number", y = "Normalized Value", color = "Parameter")
```

### 文档格式化代码 (Code for formatting document)

由于在这个文档中格式化会打乱 LaTeX 代码，下面的脚本可以正确格式化 KaTeX 文档。

```bash
#!/bin/bash

sed -i.bck -E ':a;N;$!ba;s/\$\$([^$]+)\$\$/```code2 \1```/g; s/\$([^$]+)\$/```code1 \1```/g' $1
prettier --write $1
sed -i -E ':a;N;$!ba;s/```code1([^`]*)```/\$\1\$/g' $1
sed -i -E ':a;N;$!ba;s/```code2([^`]*)```/\$\$\1\$\$/g' $1
sed -i -E ':a;N;$!ba;s/`code1([^`]*)`/\$\1\$/g' $1
sed -i -E ':a;N;$!ba;s/`code2([^`]*)`/\$\$\1\$\$/g' $1
sed -i -E 's/(\$+)\s*([^$]+?)\s*(\$+)/\1\2\3/g' $1
```

### 资源 (Resources)
- https://archive.devcon.org/archive/watch/6/eels-the-future-of-execution-layer-specifications/?tab=YouTube
- [EIP‑1559](https://eips.ethereum.org/EIPS/eip-1559) • [archived](https://web.archive.org/web/20230101000000/https://eips.ethereum.org/EIPS/eip-1559)
- [EIP‑4844](https://eips.ethereum.org/EIPS/eip-4844) • [archived](https://web.archive.org/web/20230701000000/https://eips.ethereum.org/EIPS/eip-4844)
- [Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) • [archived](https://web.archive.org/web/20240310000000/https://ethereum.github.io/yellowpaper/paper.pdf)
- [EL Specs](https://github.com/ethereum/execution-specs) • [archived](https://web.archive.org/web/20240501000000/https://github.com/ethereum/execution-specs)

> [!NOTE]
> 本 PR 中的所有主题均可在独立分支上开展协作。
