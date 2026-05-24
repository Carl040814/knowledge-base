# 以太坊执行层数据结构

# 区块头 (Block Header)

| 名称 | 描述 | EIP / 分叉 |
|------- | ------- | ------- |
| parent_hash | 前一区块的哈希值 | |
| ommers_hash | ommer（又称 uncle）列表的哈希值 | |
| beneficiary | 接收区块奖励的矿工或验证者地址 | |
| state_root | 执行所有交易后世界状态树 (world state trie) 的根哈希 | |
| transactions_root | 此区块交易树 (transaction trie) 的根哈希 | |
| receipts_root | 此区块收据树 (receipt trie) 的根哈希 | |
| logs_bloom | 汇总此区块收据中所有日志的布隆过滤器 (bloom filter) | |
| difficulty | PoW：区块难度。PoS：未使用。 | |
| number | 区块编号（链中的高度） | |
| gas_limit | 此区块允许的最大 gas 量 | |
| gas_used | 此区块所有交易使用的总 gas | |
| timestamp | 区块被提议时的 Unix 时间戳 | |
| extra_data | 用于附加数据的任意 32 字节字段（例如矿工 ID） | |
| prev_randao | PoW 时期称为 `mix_hash`，用于 nonce 验证。PoS：验证者的随机种子 | [EIP-4399](https://eips.ethereum.org/EIPS/eip-4399) / Paris |
| nonce | PoW：挖矿难题的解。PoS：未使用 | |
| base_fee_per_gas | 每 gas 的最低基础费用 | [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) / London |
| withdrawals_root | 提款列表树 (withdrawals trie) 的根哈希 | [EIP-4895](https://eips.ethereum.org/EIPS/eip-4895) / Shanghai |
| base_fee_per_blob_gas | 每 blob gas 的最低基础费用 | [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) / Dencun |
| blob_gas_used | 区块中使用的总 blob gas | [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) / Dencun |
| excess_blob_gas | 未使用的 blob gas 滚动计数器 | [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) / Dencun |
| parent_beacon_block_root | 父信标区块 (beacon block) 的 SSZ 根 | [EIP-4788](https://eips.ethereum.org/EIPS/eip-4788) / Dencun |
| request_root | 执行层生成的跨层请求 (cross-layer requests) 的根哈希 | [EIP-7685](https://eips.ethereum.org/EIPS/eip-7685) / Pectra |

# 区块体 (Block Body)

| 名称 | 描述 | EIP/分叉 |
|------- | ------- | ------- |
| transactions[] | 按顺序在区块中执行的交易列表。可以有不同类型的交易 | |
| ommers[] | PoS：空。PoW：ommer（uncle）列表 | |
| withdrawals[] | 验证者的 ETH 提款列表 | [EIP-4895](https://eips.ethereum.org/EIPS/eip-4895) / Shanghai |
| requests[] | 执行产生的跨层请求列表 | [EIP-7685](https://eips.ethereum.org/EIPS/eip-7685) / Pectra |

# 状态树 (State Trie)
以区块头中的 `state_root` 为根的树。
每个叶节点编码为 `keccak(address1) -> RLP(nonce, balance, storage_root, code_hash)`

| 名称 | 描述 |
|------- | ------- |
| nonce | 已发送（EOA）或已创建（合约）的交易数量 |
| balance | 账户的 ETH 余额，以 wei 计 |
| storage_root | 账户存储树 (storage trie) 的根哈希 |
| code_hash | 合约代码的 keccak256 哈希。EOA 为空哈希或委托指示符 |

# 收据树 (Receipts Trie)
以区块头中的 `receipts_root` 为根的树

收据包含以下字段：
| 名称 | 描述 | EIP / 分叉 |
|------- | ------- | ------- |
| type | 收据类型字节前缀 | [EIP-2718](https://eips.ethereum.org/EIPS/eip-2718) / Berlin |
| status | 指示交易状态。1 = 成功，0 = 失败。Byzantium 之前称为 `state_root` | [EIP-658](https://eips.ethereum.org/EIPS/eip-658) / Byzantium |
| cumulative_gas_used | 区块中直到并包括此交易的总 gas 使用量 | |
| logs_bloom | 汇总此交易所有日志的 256 字节布隆过滤器 | |
| logs[] | 执行期间发出的事件日志列表 | |

`logs[]` 字段中的每个项包含以下内容：
| 名称 | 描述 |
|------- | ------- |
| address | 发出日志的合约地址 |
| topics[] | 索引主题 (topics) 数组（包含事件签名） |
| data | ABI 编码的非索引事件数据 |

# 提款树 (Withdrawals Trie)
以区块头中的 `withdrawals_root` 为根的树

提款的字段有：
| 名称 | 描述 |
|------- | ------- |
| index | 单调递增的索引 |
| validator_index | 进行提款的验证者索引 |
| address | 接收提款的地址 |
| amount | 金额，以 gwei 计 |

# 交易树 (Transaction Trie)
以区块头中的 `transactions_root` 为根的树。交易可以有多种类型。

## 传统交易 (Legacy Transaction)（类型 0x00）
| 名称 | 描述 | EIP/分叉 |
|------- | ------- | ------- |
| nonce | 发送者在此交易之前已发送的交易数量 | |
| gas_price | 传统定价模型：每单位 gas 价格 | |
| gas_limit | 此交易允许消耗的最大 gas | |
| to | 接收者地址，合约创建时为空 | |
| value | 要转移的 ETH 金额，以 wei 计 |
| data | 合约交互的 calldata 或创建的初始化代码 |
| chain_id | 链的 ID | [EIP-155](https://eips.ethereum.org/EIPS/eip-155) / Spurious Dragon |
| v, r, s | 签名组件（用于恢复发送者地址） | |

## 访问列表交易 (Access List Transaction)（类型 0x01）
字段与类型 0 相同，但增加了 `access_list`。
编码为 `RLP(chain_id, nonce, gas_price, gas_limit, to, value, data, access_list, v, r, s)`

| 名称 | 描述 | EIP/分叉 |
|------- | ------- | ------- |
| access_list | 要访问的地址/存储键列表 | [EIP-2930](https://eips.ethereum.org/EIPS/eip-2930) / Berlin |

## 动态费用交易 (Dynamic-Fee Transaction)（类型 0x02）
编码为 `RLP(chain_id, nonce, max_priority_fee_per_gas, max_fee_per_gas, gas_limit, to, value, data, access_list, v, r, s)`，通过用新字段替换 `gas_price` 来扩展类型 1 交易。

| 名称 | 描述 | EIP/分叉 |
|------- | ------- | ------- |
| max_priority_fee | 给验证者的小费，激励其优先包含此交易 | [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) / London |
| max_fee_per_gas | 每单位 gas 愿意支付的最高费用 | [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) / London |

## 携带 Blob 的交易 (Blob-Carrying Transaction)（类型 0x03）
编码为 `RLP(chain_id, nonce, max_priority_fee_per_gas, max_fee_per_gas, gas_limit, to, value, data, access_list, max_fee_per_blob_gas, blob_versioned_hashes, v, r, s)`

与类型 2 交易相同，只是增加了新的 blob 相关字段。

| 名称 | 描述 | EIP/分叉 |
|------- | ------- | ------- |
| max_fee_per_blob_gas | 每 blob gas 单位愿意支付的最高费用 | [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) / Dencun |
| blob_versioned_hashes | 每个 blob 的哈希列表 | [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) / Dencun |

## 为 EOA 设置代码的交易 (Set Code Transaction for EOAs)（类型 0x04）
扩展类型 3 交易，新增 `authorization_list` 字段，内容如下。

| 名称 | 描述 | EIP/分叉 |
|------- | ------- | ------- |
| chain_id | 链的 ID | [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702) / Prague |
| address | 要委托的智能合约地址 | [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702) / Prague |
| nonce | 发送者在此交易之前已发送的交易数量 | [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702) / Prague |
| y_parity, r, s | 密码学签名元素 | [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702) / Prague |
