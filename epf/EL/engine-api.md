# 引擎 API (Engine API)

> [!WARNING]
> 本页面的 Glamsterdam 部分涵盖了活跃的研究领域（EIP-7732 ePBS 和 EIP-7928 BAL）。这些部分在阅读时可能已经过时，并随着设计空间的演变在未来进行更新。

**预备阅读：** [EL 架构 (EL Architecture)](/wiki/EL/el-architecture.md)

引擎 API (Engine API) 是共识层 (Consensus Layer, CL) 与执行层 (Execution Layer, EL) 之间经过身份验证的通信接口，在合并 (The Merge) 时引入。CL 通过该接口驱动 EL 的区块构建 (block building)、验证和分叉选择 (fork choice)。它告知 EL 哪条链是规范链 (canonical chain)，请求其构建新区块，并将接收到的区块发送以进行验证。

## 架构 (Architecture)

### 网络隔离 (Network Isolation)

引擎 API 在**专用端口（默认：8551）**上运行，与面向公众的 JSON-RPC API（默认：8545）严格隔离。这种隔离是一项安全要求。共享端口将允许公共流量或蓄意的拒绝服务 (Denial of Service, DoS) 洪水攻击耗尽共识对话，导致遗漏提案 (proposals) 和证明 (attestations)。

### 通信协议 (Communication Protocols)

| 协议 (Protocol) | 身份验证 (Authentication) | 说明 (Notes) |
|---|---|---|
| HTTP | 每次请求均需使用 JWT | 标准；无状态 |
| WebSocket | 仅在初始握手时使用 JWT | 持久连接；无单帧验证 |
| IPC | 无 | 仅限同机通信；文件系统权限提供隔离 |

### 辅助 eth_ 方法 (Ancillary eth_ Methods)

规范要求 EL 在经过身份验证的引擎 API 端口上公开以下所有九个 `eth_` 方法，以便 CL 无需建立单独连接即可查询链状态：

`eth_blockNumber`, `eth_call`, `eth_chainId`, `eth_getCode`, `eth_getBlockByHash`, `eth_getBlockByNumber`, `eth_getLogs`, `eth_sendRawTransaction`, `eth_syncing`

`eth_getLogs` 对于 CL 监控存款合约（pre-Electra）至关重要。`eth_call` 使 CL 能够验证 EIP-7002 的取款凭证，而无需广播交易。

## 身份验证 (Authentication)

CL 和 EL 共享一个 256 位（32 字节）十六进制编码的 JWT 密钥，该密钥通过指向 `jwt.hex` 文件的 `--jwt-secret` 命令行界面 (Command Line Interface, CLI) 标志进行配置。如果省略，EL 将为该会话自动生成一个随机密钥，并将其写入其数据目录中的 `jwt.hex` 中。

**算法 (Algorithm)**：EL 必须强制使用 **HS256** (HMAC-SHA256)。任何指定 `alg: none` 的 JWT 都必须立即被拒绝，以防止身份验证绕过。

**重放保护 (Replay protection)**：每个 JWT 必须包含一个 `iat`（签发时间）声明 (claim)。EL 必须拒绝任何令牌的 `iat` 与 EL 本地时钟偏差超过 **±60 秒**的请求。这可以防止捕获的令牌被重放以引发链重组 (reorganizations)。

为了遥测 (telemetry) 目的，可以包含可选的声明（`id`，`clv`），但不对其进行访问控制验证。

## 能力协商 (Capability Negotiation)

### engine_exchangeCapabilities

`engine_exchangeCapabilities` 没有版本后缀。它是唯一没有版本后缀的引擎 API 方法。它允许客户端发现彼此支持的方法版本。

- EL **必须**支持此方法；CL 可以选择性调用它。
- 每一方都发送一个支持的方法名称及其版本后缀的数组（例如 `["engine_newPayloadV3", "engine_newPayloadV4"]`）。
- EL 用自己的列表进行响应。`engine_exchangeCapabilities` 本身**绝不能**出现在响应中。
- 如果从未调用此方法，EL 不得记录错误（以保证向后兼容性）。

### engine_getClientVersionV1

一种可选方法，允许 CL 和 EL 识别彼此的软件。每一侧都返回一个 `ClientVersionV1` 结构，其中包含两个字母的客户端代码、易于阅读的名称、版本字符串和 4 字节的提交哈希 (commit hash)。紧凑的标识符旨在适合 32 字节的信标区块涂鸦 (beacon block graffiti) 字段，以便进行网络多样性跟踪。

| 代码 | EL 客户端 | 代码 | CL 客户端 |
|---|---|---|---|
| `BU` | Besu | `GR` | Grandine |
| `EG` | Erigon | `LH` | Lighthouse |
| `EJ` | EthereumJS | `LS` | Lodestar |
| `EX` | Ethrex | `NB` | Nimbus |
| `GE` | Geth | `PM` | Prysm |
| `NM` | Nethermind | `TK` | Teku |
| `RH` | Reth | | |
| `TE` | Trin-Execution | | |

如果从未调用此方法，EL 不得记录错误。

## 核心方法 (Core Methods)

### engine_forkchoiceUpdatedV3

更新 EL 的规范链视图，并选择性地启动区块构建。

**参数 (Parameters)：**
- `forkchoiceState`：`{headBlockHash, safeBlockHash, finalizedBlockHash}`
- `payloadAttributes`（可选）：`{timestamp, prevRandao, suggestedFeeRecipient, withdrawals, parentBeaconBlockRoot}`。如果提供，EL 开始构建负载并返回一个 `payloadId`。

**返回 (Returns)：** `{payloadStatus, payloadId}`

`engine_forkchoiceUpdatedV3` 仅返回 `VALID`（有效）、`INVALID`（无效）或 `SYNCING`（同步中）。它绝不会返回 `ACCEPTED`（已接受）。分叉选择更新是重组或延伸规范链的权威命令，因此 EL 在执行更新之前必须完全解析头区块的状态。`ACCEPTED` 是 `engine_newPayload` 所独有的。

### 负载状态值 (Payload Status Values)

| 状态 | 返回自 | 含义 |
|---|---|---|
| `VALID` | 两者 | 区块及所有祖先区块已完整下载并完成 EVM 验证 |
| `INVALID` | 两者 | 违反共识规则；`latestValidHash` 标识用于分叉恢复的最高有效祖先区块 |
| `SYNCING` | 两者 | 本地缺失所需的祖先数据；EL 已开始通过 p2p 获取 |
| `ACCEPTED` | 仅限 newPayload | 区块哈希有效，所有交易长度非零，负载**没有**延伸规范链（它在侧支上），祖先在本地可用。EVM 执行故意推迟，直到分叉选择可能会转向该分支。 |

**ACCEPTED 与 SYNCING 的对比**：`SYNCING` 意味着 EL 无法接受该区块，因为其链历史记录缺失。`ACCEPTED` 意味着祖先链完好无损。EL 选择不运行完整的以太坊虚拟机 (Ethereum Virtual Machine, EVM) 验证，因为目前这是一个非规范的侧分支。如果 LMD-GHOST 稍后转向此分支，EL 将执行推迟的状态过渡 (state transition)。

### engine_newPayloadV4

将接收到的信标区块中的执行负载 (execution payload) 交付给 EL 进行验证。

**参数：**
- `executionPayload`：完整区块
- `expectedBlobVersionedHashes`：有序的 blob 版本化哈希列表
- `parentBeaconBlockRoot`：父信标区块根 (EIP-4788)
- `executionRequests`：EL 生成的针对 CL 的请求（Electra 及更高版本）

**验证：**
- 执行所有交易并验证所产生的状态根
- **Blob 哈希验证**：从负载中每个携带 blob 的交易中提取 `blob_versioned_hashes`，保持包含顺序，将其拼接，并与 `expectedBlobVersionedHashes`进行比较。任何不匹配或顺序差异都会返回 `INVALID`。

**返回：** `VALID`, `INVALID`, `SYNCING`, 或 `ACCEPTED`

### engine_getPayloadV4

检索 EL 在调用带有 `payloadAttributes` 的 `forkchoiceUpdated` 后构建的负载。

**参数：** `payloadId` 是由 `engine_forkchoiceUpdatedV3` 返回的 8 字节 ID。

**返回：**
- `executionPayload`：组装好的区块
- `blockValue`：归属于 `feeRecipient` 的总优先费用（以 wei 为单位，256 位数量）
- `blobsBundle`：`{commitments, proofs, blobs}`。包含 48 字节的 KZG 承诺、48 字节的 KZG 证明和 131,072 字节的 blob 数据，以便 CL 构建 blob 边车 (blob sidecar)。
- `shouldOverrideBuilder`：EL 的一个布尔值**建议**，即应使用本地构建的负载，而不是外部的 MEV-Boost 竞价。CL 可以采纳也可能不采纳。它是 CL 决策的输入之一，而不是命令。EL 使用实现定义的启发式方法来设置它（例如，一个高价值交易一直被构建者竞价排除在外）。如果 EL 没有实现启发式方法，它必须默认返回 `false`。
- `executionRequests`：EL 生成的请求（Electra 及更高版本）

## 执行请求 (Execution Requests, EIP-7685)

在 V4（Prague/Electra）中引入，执行请求允许 EVM 智能合约触发共识层状态变化。每个请求都是 `request_type ++ request_data`，其中 `request_type` 是 1 字节的前缀。

**Electra 中支持的类型：**

| 类型 | EIP | 描述 |
|---|---|---|
| `0x00` | EIP-6110 | **存款请求 (Deposit requests)**：EL 直接将存款事件推送到 CL，取代 CL 日志轮询。将验证者激活时间从 ~12 小时缩短到 ~13 分钟。每个负载的最大值：`MAX_DEPOSIT_REQUESTS_PER_PAYLOAD = 8,192`。 |
| `0x01` | EIP-7002 | **提现请求 (Withdrawal requests)**：持有 `0x01` 提现凭证的智能合约可以从 EVM 触发部分或全部验证者退出。通过在 `0x00000961Ef480Eb55e80D19ad83579A64c007002` 的预部署合约进行处理。目标：2个/区块；最大值：16个/区块。费用：`fake_exponential(1_wei, excess, 17)`，在正常负载下接近 1 wei，在持续需求下呈指数级昂贵。系统调用 gas：30,000,000（排除在区块 gas 计费之外）。 |
| `0x02` | EIP-7251 | **合并请求 (Consolidation requests)**：将多个 32 ETH 验证者合并为一个复合验证者（有效余额最高可达 2048 ETH）。目标：1个/区块；最大值：2个/区块。挂起队列硬限：`PENDING_CONSOLIDATIONS_LIMIT = 262,144` (2^18)。 |

**排序和验证规则**：`executionRequests` 数组必须：
- 按 `request_type` 进行**严格升序排序**
- 每个 `request_type` 字节**最多出现一次**（无重复）
- 没有 `request_data` 为空的元素（长度 <= 1 字节的元素必须被排除）

任何违反规则的行为都会返回 `-32602: Invalid params`。EL 还会计算一个 `requests_hash`（对排序后的列表进行 SHA-256 哈希），并与区块头进行验证；不匹配则返回 `INVALID`。对于没有请求的区块，哈希默认使用 `sha256("") = 0xe3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`。

## 版本历史 (Version History)

| 版本 | EL 分叉 | CL 分叉 | 关键变化 |
|---|---|---|---|
| V1 | Paris | Bellatrix | 初始合并后版本：`forkchoiceUpdated`, `newPayload`, `getPayload` |
| V2 | Shanghai | Capella | 在负载属性和 executionPayload 中添加了 `withdrawals`（提现） |
| V3 | Cancun | Deneb | 添加了 `blobVersionedHashes`, `parentBeaconBlockRoot`；executionPayload 增加了 `blobGasUsed`, `excessBlobGas`；`getPayload` 返回 `BlobsBundleV1` |
| V4 | Prague | Electra | 添加了 `executionRequests` (EIP-7685) |

## 时隙生命周期 (Slot Lifecycle)

引擎 API 在严格的 12 秒时隙 (slot) 心跳内运行。对于提议者 (proposer) 节点：

| 时间 | 操作 |
|---|---|
| **t = 0s** | 时隙开始。CL 调用带有 `payloadAttributes` 的 `engine_forkchoiceUpdatedV3`。EL 返回 `payloadId`。 |
| **t = 1-3s** | EL 构建负载：选择交易池 (mempool) 中的交易，执行它们，计算状态根。CL 选择性地查询外部 MEV 中继 (relays)。 |
| **t = 3s** | CL 调用 `engine_getPayloadV4`，将负载包装到 `BeaconBlock`（信标区块）中，签名并广播。 |
| **t = 4s** | **证明截止时间。** 其他验证者必须在此时间点前收到区块，调用 `engine_newPayloadV4`，并收到 `VALID`。迟到的区块将不会被证明，并且无法获得包含收益。 |
| **t = 4-12s** | CL 调用 `engine_forkchoiceUpdatedV3`（不带 `payloadAttributes`）将新区块设置为头。 |

非提议者节点跳过前三个步骤，仅执行 `engine_newPayloadV4`，然后执行 `engine_forkchoiceUpdatedV3`。

### 乐观同步 (Optimistic Sync)

在重负载期间，CL 可能会乐观地导入信标区块，而无需等待完整的 EVM 执行。如果 EL 稍后返回 `INVALID`，则 CL 将重组回 `latestValidHash`。最大乐观导入深度受限于 `SAFE_SLOTS_TO_IMPORT_OPTIMISTICALLY`（默认：**128 个时隙**，~25.6 分钟），可通过 `--safe-slots-to-import-optimistically` 进行配置。这可以防止“分叉选择毒化”攻击，即恶意节点在链尖端提供结构有效但计算无效的区块。

## 错误处理 (Error Handling)

### JSON-RPC 错误码 (JSON-RPC Error Codes)

| 代码 | 名称 | 触发条件 |
|---|---|---|
| `-32700` 至 `-32603` | 标准 JSON-RPC | 解析错误、无效请求、找不到方法、无效参数、内部错误 |
| `-32000` | 服务器错误 | 通用 EL 故障；响应**必须**包含带有诊断上下文的 `data.err` 字符串（例如 `"LevelDB read failure"`） |
| `-38001` | 未知负载 | 调用 `engine_getPayload` 时使用的 `payloadId` 没有处于活动状态的构建进程或已超时 |
| `-38002` | 无效的分叉选择状态 | `forkchoiceState` 哈希在逻辑上不一致（例如 `safeBlockHash` 不是 `headBlockHash` 的祖先） |
| `-38003` | 无效的负载属性 | `payloadAttributes` 字段在结构上无效或缺少分叉所需的字段 |
| `-38004` | 请求过大 | 数组参数超出了硬编码的内存限制 |
| `-38005` | 不支持的分叉 | 负载时间戳与 EL 的活动分叉不一致（例如，在 Deneb 激活之前提交了 Deneb 格式的负载） |

### 失败模式 (Failure Modes)

**`SYNCING`**：CL 重试 `engine_forkchoiceUpdatedV3`，直到 EL 追上。不要证明或在该区块上构建。

**`INVALID`**：CL 将该区块及所有后代区块标记为无效，将分叉选择还原为 `latestValidHash`。

**EL 无法访问**（端口关闭、JWT 不匹配、崩溃）：CL 无法提议或验证。在连接恢复之前，验证者将错过所有职责。

## 未来升级 (Future Upgrades)

### Fusaka (Fulu/Osaka, 2025年12月3日，epoch 411392)

Fusaka 升级包含 13 个 EIP，涵盖数据可用性 (Data Availability, DA) 扩展、执行性能和协议清理。引擎 API 的关键影响：

**数据可用性 (Data availability)：**
- **EIP-7594 (PeerDAS)**：节点对小的列子集进行采样，而不是下载完整的 blob，从而实现 blob 吞吐量的安全提升。`engine_getPayloadBodiesByHashV2` 和 `engine_getPayloadBodiesByRangeV2` 已更新以支持 PeerDAS 单元格证明 (cell proof) 结构。
- **EIP-7918**：Blob 基础费用地板与执行基础费用成比例挂钩，防止 blob 费用在低需求期间降至 1 wei。
- **EIP-7892 (BPO forks)**：仅限 Blob 参数 (Blob Parameter Only, BPO) 分叉允许在 Fusaka 之后通过轻量级网络调整来扩展 blob 数量，而无需触发完整的硬分叉。BPO1 和 BPO2 在 Fusaka 之后不久激活，分别将 blob 目标提高到 10/15 和 14/21。

**执行性能 (Execution performance)：**
- **EIP-7935**：默认区块 gas 限制提高到 **60M**（由客户端协同完成，非共识规则）。
- **EIP-7825**：交易 gas 限制上限为 **2^24 = 16,777,216 gas**。没有哪单笔交易可以占满整个区块。
- **EIP-7934**：RLP 执行区块大小上限为 **8 MiB**（`MAX_RLP_BLOCK_SIZE = 10 MiB - 2 MiB 安全余量`）。CL 传播协议已经丢弃超过 10 MiB 的区块；2 MiB 的余量为信标区块预留了空间，使得有效的 EL 负载限制为 8 MiB。
- **EIP-7883**：MODEXP 预编译合约重新定价。最低 gas 从 200 提高到 **500**，通用成本公式**翻三倍**，大指数乘数从 8 提高到 **16**。
- **EIP-7823**：MODEXP 输入每个输入限制为 **8192 位（1024 字节）**（基数、指数、模数）。所有历史上真实的用法都在此限制之内。
- **EIP-7917**：确定性提议者展望。提议者日程表会提前一个完整 epoch 得知，从而改善 MEV 和 PBS 协调。

### Glamsterdam (Fusaka 之后)

**EIP-7732 (ePBS)** 是提议的 Glamsterdam CL 重头戏。它将 PBS 从协议外的 MEV-Boost 边车移入共识层，消除了对可信中继的依赖。引擎 API 的时隙生命周期将发生重大改变：提议者在 t=3s 时仅接收已签名的**构建者承诺 (builder commitment)**，并在没有执行负载的情况下对其进行广播；然后构建者在 t=4s 证明截止时间之后向网络揭示完整的 `ExecutionPayloadEnvelope`。这可防止提议者窃取构建者的 MEV，同时保留证明的能力。新的引擎 API 方法分别处理承诺验证与完整 EVM 执行。

**EIP-7928 (区块级访问列表)** 是提议的 Glamsterdam EL 重头戏。每个区块都将包含执行过程中访问的所有地址和存储插槽的显式映射，从而实现：
- 在 EVM 执行前并行预取状态
- 在 CPU 核心之间并行执行无冲突的交易
- 为无状态和 ZK 客户端提供免执行的状态验证

引擎 API 将通过让 EL 执行负载、在内部计算访问列表、并与负载头中的 `blockAccessList` 进行验证来验证此行为。不匹配则返回 `INVALID`。这两个 EIP 都是草案提案。

## 资源 (Resources)

- [引擎 API 规范 (execution-apis)](https://github.com/ethereum/execution-apis/tree/main/src/engine)
- [EIP-3675: 升级到权益证明](https://eips.ethereum.org/EIPS/eip-3675)
- [EIP-7685: 通用 EL 请求](https://eips.ethereum.org/EIPS/eip-7685)
- [EIP-7002: 执行层可触发的提现](https://eips.ethereum.org/EIPS/eip-7002)
- [EIP-7607: 硬分叉元数据 - Fusaka](https://eips.ethereum.org/EIPS/eip-7607)
- [EIP-7732: 封装式提议者-构建者分离](https://eips.ethereum.org/EIPS/eip-7732)
- [EIP-7928: 区块级访问列表](https://eips.ethereum.org/EIPS/eip-7928)
- [引擎 API 可视化指南](https://hackmd.io/@danielrachi/engine_api)
