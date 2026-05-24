# JSON-RPC

JSON-RPC 规范 (Specification) 是一种基于 [OpenRPC](https://www.open-rpc.org/docs/getting-started) 的，以 JSON 编码的远程过程调用 (Remote Procedure Call, RPC) 协议。它允许在远程服务器 (Remote Server) 上调用函数并返回结果。
它是执行 API (Execution API) 规范的一部分，该规范提供了一组与以太坊区块链 (Ethereum Blockchain) 进行交互的方法。
更为人所知的是，它是用户通过客户端 (Client) 与网络进行交互的方式，甚至共识层 (Consensus Layer, CL) 和执行层 (Execution Layer, EL) 也是通过 Engine API 使用它来进行交互。
本节提供了 JSON-RPC 方法的描述。

## API 规范 (API Specification)

JSON-RPC 方法按命名空间 (Namespaces) 分组，并指定为方法前缀。尽管它们的目的各不相同，但它们都具有相同的通用结构，并且在所有实现中的行为必须相同：

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "<prefix_methodName>",
  "params": [...]
}
```

其中：
- `id`：请求的唯一标识符。
- `jsonrpc`：JSON-RPC 协议的版本。
- `method`：要调用的方法。
- `params`：方法的参数。如果方法不需要任何参数，它可以是一个空数组。其他参数如果没有提供，可能会有默认值。

### 命名空间 (Namespaces)

每个方法都由一个命名空间前缀 (Namespace Prefix) 和方法名称组成，两者用下划线分隔。

以太坊客户端 (Ethereum Clients) 必须实现规范所要求的、用于与网络交互的最基本 RPC 方法集。在此基础上，还有一些客户端特有的方法，用于控制节点 (Node) 或实现额外的独特功能。请始终参考列出可用方法 and 命名空间的客户端文档，例如注意 [Geth](https://geth.ethereum.org/docs/interacting-with-geth/rpc) 和 [Reth](https://reth.rs/jsonrpc/intro/) 文档中的不同命名空间。

以下是最常见命名空间的示例：

| **命名空间 (Namespace)** | **描述 (Description)**                                                                                      | **敏感度 (Sensitive)** |
| ------------- | ---------------------------------------------------------------------------------------------------- | ------------- |
| eth           | eth API 允许您与以太坊 (Ethereum) 进行交互。                                                    | 可能         |
| web3          | web3 API 为 web3 客户端 (web3 Client) 提供实用工具函数 (Utility Functions)。                                         | 否            |
| net           | net API 提供对节点网络信息 (Network Information of the Node) 的访问。                                      | 否            |
| txpool        | txpool API 允许您检查交易池 (Transaction Pool)。                                           | 否            |
| debug         | debug API 提供了几种检查以太坊状态 (Ethereum State) 的方法，包括 Geth 风格的追踪 (Geth-style Traces)。   | 否            |
| trace         | trace API 提供了几种检查以太坊状态 (Ethereum State) 的方法，包括 Parity 风格的追踪 (Parity-style Traces)。 | 否            |
| admin         | admin API 允许您配置您的节点。                                                     | 是           |
| rpc           | rpc API 提供有关 RPC 服务器 (RPC Server) 及其模块的信息。                                | 否            |

敏感 (Sensitive) 意味着它们可用于设置节点，例如 *admin*，或者访问存储在节点中的账户数据 (Account Data)，就像 *eth* 一样。

现在，让我们看一些方法，以了解它们是如何构建的以及它们的作用：

#### Eth

Eth 可能是最常用的命名空间，它提供对以太坊网络 (Ethereum Network) 的基本访问，例如，它是钱包 (Wallets) 读取余额 (Balance) 和创建交易 (Transactions) 所必需的。
这里只提供了一个简要的方法列表，完整列表可以在 [以太坊 JSON-RPC 规范 (Ethereum JSON-RPC Specification)](https://ethereum.github.io/execution-apis/api-documentation/) 中找到。

| **方法 (Method)**                           |           **参数 (Params)**            | **描述 (Description)**                                                                                                                             |
| ------------------------------------ |:-------------------------------:| ------------------------------------------------------------------------------------------------------------------------------------------- |
| eth_blockNumber                      |       无强制参数       | 返回最新区块 (Block) 的编号                                                                                                 |
| eth_call                             |       交易对象 (Transaction Object)        | 立即执行一个新的消息调用 (Message Call)，而不在区块链上创建交易                                                   |
| eth_chainId                          |       无强制参数       | 返回当前链 ID (Chain ID)                                                                                                                |
| eth_estimateGas                      |       交易对象 (Transaction Object)        | 发起调用或交易，这不会被添加到区块链中，并返回所消耗的 Gas，这可用于估算所使用的 Gas (Gas Used) |
| eth_gasPrice                         |       无强制参数       | 返回当前每个 Gas 的价格（以 wei 为单位）                                                                                                    |
| eth_getBalance                       |      地址 (Address), 区块编号 (Block Number)      | 返回给定地址账户的余额                                                                                     |
| eth_getBlockByHash                   |      区块哈希 (Block Hash), 完整交易列表 (Full Txs)       | 通过哈希返回区块的信息                                                                                                   |
| eth_getBlockByNumber                 |     区块编号 (Block Number), 完整交易列表 (Full Txs)      | 通过区块编号返回区块的信息                                                                                           |
| eth_getBlockTransactionCountByHash   |           区块哈希 (Block Hash)            | 返回与给定区块哈希匹配的区块中的交易数量                                                    |
| eth_getBlockTransactionCountByNumber |          区块编号 (Block Number)           | 返回与给定区块编号匹配的区块中的交易数量                                                  |
| eth_getCode                          |      地址 (Address), 区块编号 (Block Number)      | 返回区块链中给定地址处的代码 (Code)                                                                                           |
| eth_getLogs                          |          过滤器对象 (Filter Object)          | 返回与给定过滤器对象匹配的所有日志 (Logs) 数组                                                                                 |
| eth_getStorageAt                     | 地址 (Address), 位置 (Position), 区块编号 (Block Number) | 返回给定地址处存储位置 (Storage Position) 的值                                                                                |

#### Debug (调试)

*debug* 命名空间提供了检查以太坊状态的方法。它是对原始数据 (Raw Data) 的直接访问，这对于某些用例（如区块浏览器 (Block Explorers) 或研究目的）可能是必需的。其中一些方法可能需要在节点上进行大量计算，并且在非归档节点 (Non-archival Node) 上请求历史状态通常是不可行的。因此，公共 RPC 提供商通常会限制此命名空间或仅允许安全的方法。
以下是调试方法的常规示例：

| **方法 (Method)**               |      **参数 (Params)**       | **描述 (Description)**                                                 |
|--------------------------|:---------------------:|-----------------------------------------------------------------|
| debug_getBadBlocks       |  无强制参数  | 返回客户端见过的近期坏区块 (Bad Blocks) 数组 |
| debug_getRawBlock        |     区块编号 (Block Number)      | 返回一个 RLP 编码 (RLP-encoded) 的区块                                    |
| debug_getRawHeader       |     区块编号 (Block Number)      | 返回一个 RLP 编码 (RLP-encoded) 的区块头 (Header)                                   |
| debug_getRawReceipts     |     区块编号 (Block Number)      | 返回一个 EIP-2718 二进制编码的收据 (Receipts) 数组            |
| debug_getRawTransactions |        交易哈希 (Tx Hash)        | 返回一个 EIP-2718 二进制编码的交易 (Transactions) 数组        |

#### Engine (引擎)

[Engine API](https://hackmd.io/@danielrachi/engine_api) 与上述通用的以太坊 JSON-RPC 方法不同。执行客户端 (Execution Clients) 在一个独立的、经过身份验证的端点 (Endpoint) 上提供 Engine API，而不是在普通的 HTTP JSON-RPC 端口上，因为它不是面向用户的 API。它的唯一目的是促进客户端之间的通信 (Inter-client Communication)，在共识客户端和执行客户端之间交换有关共识 (Consensus)、分叉选择 (Fork Choice) 和区块验证 (Block Validation) 的信息。

客户端间通信通过基于 HTTP 的 JSON-RPC 接口运行，并使用 JSON Web 令牌 (JSON Web Token, JWT) 进行安全保护。JWT 将发送方身份验证为合法的共识层客户端 (Consensus Layer Client)，但它不提供流量加密。此外，Engine JSON-RPC 端点只能由共识层访问，确保恶意外部各方无法与其交互。

下表列出了核心的 Engine API 方法，并简要描述了它们的目的及其接受的参数：
| **方法 (Method)**                               |               **参数 (Params)**               | **描述 (Description)**                                                           |
|------------------------------------------|:--------------------------------------:|---------------------------------------------------------------------------|
| engine_exchangeTransitionConfigurationV1 |        共识客户端配置 (Consensus Client Config)         | 在共识层 (CL) 和执行层 (EL) 之间交换配置详细信息                                            |
| engine_forkchoiceUpdatedV1*              |  分叉选择状态 (Forkchoice State), 有效载荷属性 (Payload Attributes)  | 更新分叉选择状态，并可选地启动有效载荷构建 (Payload Building)                                            |
| engine_getPayloadBodiesByHashV1*         |           区块哈希数组 (Block Hash Array)           | 给定区块哈希，返回相应的执行有效载荷体 (Execution Payload Bodies) |
| engine_getPayloadV1*                     |  分叉选择状态 (Forkchoice State), 有效载荷属性 (Payload Attributes)  | 获取由执行层 (EL) 构建的执行有效载荷 (Execution Payload)                      |
| debug_newPayloadV1*                      |                交易哈希 (Tx Hash)                 | 返回用于调试目的的执行有效载荷验证详细信息                                      |

标有星号 (*) 的方法有多个版本，以支持网络升级和不断演进的协议特性。[以太坊 JSON-RPC 规范 (Ethereum JSON-RPC Specification)](https://ethereum.github.io/execution-apis/api-documentation/) 提供了关于这些方法的详细文档。

## 编码 (Encoding)

对于 JSON-RPC 方法 of 参数编码，存在一个约定，即十六进制编码 (Hex Encoding)。
* 数量 (Quantities) 表示为使用 "0x" 前缀的十六进制值。
  * 例如，数字 65 表示为 "0x41"。
  * 数字 0 表示为 "0x0"。
  * 一些无效的用法是 "0x" 和 "ff"。因为第一种情况后面没有数字，第二种情况没有以 "0x" 为前缀。
* 未格式化的数据 (Unformatted Data)，如哈希 (Hashes)、账户地址 (Account Addresses) 或字节数组 (Byte Arrays)，也同样使用 "0x" 前缀进行十六进制编码。
  * 例如：0x400（十进制中的 1014）
  * 无效的情况是 0x0400，因为不允许有前导零。

## 传输无关 (Transport Agnostic)

这里值得一提的是，JSON-RPC 是传输无关的，这意味着它可以在任何传输协议 (Transport Protocol) 上使用，例如 HTTP、WebSockets (WSS)，甚至进程间通信 (Inter-Process Communication, IPC)。
它们的区别可以总结如下：
* **HTTP** 传输提供单向的“响应-请求”模型，该模型在发送响应后会关闭连接。
* **WSS** 是一种双向协议，这意味着连接将保持打开状态，直到节点或用户显式关闭它。它允许基于订阅模型的通信，例如事件驱动的交互。
* **IPC** 传输协议用于在同一台机器上运行的进程之间进行通信。它比 HTTP 和 WSS 更快，但不适用于远程通信，例如，它可以通过本地 JS 控制台 (Console) 使用。
  
## 工具链 (Tooling)

有几种方法可以使用 JSON-RPC 方法。其中之一是使用 `curl` 命令。例如，要获取最新的区块编号，可以使用以下命令：

```bash
curl <node-endpoint> \
-X POST \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```
请注意 *params* 字段是如何为空的，因为该方法将 "latest" 作为其默认值传递。

另一种方法是在 Javascript/TypeScript 中使用 `axios` 库。例如，要获取地址余额，可以使用以下代码：

```typescript
import axios from 'axios';

const node = '<node-endpoint>';
const address = '<address>';

const response = await axios.post(node, {
  jsonrpc: '2.0',
  method: 'eth_getBalance',
  params: [address, 'latest'],
  id: 1,
  headers: {
    'Content-Type': 'application/json',
  },
});
```
您可能会注意到，JSON-RPC 方法被封装在一个 POST 请求中，参数在请求体 (Request Body) 中传递。
这是一种使用 OSI 应用层 (OSI Application Layer)——HTTP 协议在客户端和服务器之间交换数据的不同方式。

无论哪种方式，与以太坊网络进行交互最常用的方法是使用 web3 库，例如用于 Python 的 web3py，或者用于 JS/TS 的 web3.js/ethers.js：

#### web3py

```python
from web3 import Web3

# 设置 HTTPProvider
w3 = Web3(Web3.HTTPProvider('http://localhost:8545'))

# API
w3.eth.get_balance('0xaddress')
```

#### ethers.js

```typescript
import { ethers } from "ethers";

const provider = new ethers.providers.JsonRpcProvider('http://localhost:8545');

await provider.getBlockNumber();
```

通常，所有的 web3 库都会对 JSON-RPC 方法进行封装，提供一种更友好的方式来与执行层 (Execution Layer) 进行交互。请在您首选的编程语言中寻找相关库，因为语法可能会有所不同。

### 延伸阅读 (Further Reading)
* [以太坊 JSON-RPC 规范 (Ethereum JSON-RPC Specification)](https://ethereum.github.io/execution-apis/api-documentation/)
* [执行 API 规范 (Execution API Specification)](https://github.com/ethereum/execution-apis/tree/main)
* [JSON-RPC | Infura 文档 (Infura docs)](https://docs.metamask.io/services/reference/ethereum/json-rpc-methods/)
* [reth 书籍 | JSON-RPC (reth book | JSON-RPC)](https://reth.rs/jsonrpc/intro/)
* [OpenRPC](https://www.open-rpc.org/docs/getting-started)
* [Engine API | Mikhail | 第 21 讲 (Lecture 21)](https://youtu.be/fR7LBXAMH7g)
