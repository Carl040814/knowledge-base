# 简洁序列化 (Simple Serialize, SSZ)

## 概述 (Overview)


简洁序列化 (Simple Serialize, SSZ) 是专门为以太坊信标链 (Ethereum's Beacon Chain) 设计的序列化 (serialization) 和 [默克尔化 (Merkleization)](/wiki/CL/merkleization.md) 方案。在共识层 (Consensus Layer, CL) 中，除了 [对等节点发现协议 (peer discovery protocol)](https://github.com/ethereum/devp2p) 之外，SSZ 在所有地方都取代了执行层 (Execution Layer, EL) 所使用的 [RLP 序列化 (RLP serialization)](/wiki/EL/RLP.md)。其开发和采用旨在提高以太坊共识层 (CL) 的效率、安全性和可扩展性。

本文档主要介绍 SSZ 序列化。您可以在 [默克尔化维基页面 (merkleization wiki page)](/wiki/CL/merkleization.md) 了解更多关于 SSZ 默克尔化的信息。


## SSZ 工具 (SSZ Tools)

有许多适用于 SSZ 的工具。这里是 SSZ 工具的 [完整列表 (full list)](https://github.com/ethereum/consensus-specs/issues/2138)。以下是一些常用的工具：

- [py-ssz](https://github.com/ethereum/py-ssz)
- [dafny](https://github.com/ConsenSys/eth2.0-dafny)
- [remerkleable](https://github.com/protolambda/remerkleable)
- [fastssz](https://github.com/ferranbt/fastssz/)
- [rust-ssz](https://github.com/ralexstokes/ssz-rs)

## SSZ 与 RLP 序列化对比 (SSZ VS RLP Serialization)

| 评估标准 (CRITERIA)   | 紧凑性 (COMPACT) | 表达能力 (EXPRESSIVENESS) | 哈希化 (HASHING)  | 索引 (INDEXING) |
|------------|---------|----------------|----------|----------|
| RLP        | 是 (Yes) | 灵活 (Flexible) | 可能 (Possible) | 否 (No)   |
| SSZ        | 否 (No)  | 是 (Yes)        | 是 (Yes)      | 较差 (Poor) |

*表：由 [Piper Merriam](https://twitter.com/pipermerriam) 制作的 SSZ 与 RLP 对比表。*

**表达能力 (Expressiveness)**：
- **SSZ**：直接支持所有必要的数据类型，无需额外的抽象层 (abstraction layers)。这使得 SSZ 在处理以太坊权益证明 (Proof of Stake, PoS) 中使用的复杂数据结构时，本质上更加直观和健壮。
- **RLP**：仅限于动态长度字节串 (dynamic length byte strings) 和列表 (lists)。其他数据类型只能通过抽象层来支持，这可能会引入复杂性并降低效率。

**哈希化 (Hashing)**：
- **SSZ**：便于高效地对对象进行哈希 (hashing) 和重新哈希 (re-hashing)，这对于需要频繁更新数据状态的操作特别有利，例如分片 (sharding) 和无状态客户端 (stateless clients) 中的操作。这种效率对于维护区块链完整性 (blockchain integrity) 和性能至关重要。
- **RLP**：虽然可以进行哈希，但无法提供相同的性能优化，特别是在数据结构发生微小修改时。

**索引 (Indexing)**：
- **SSZ**：尽管索引被描述为“较差”，但 SSZ 支持在不进行完全反序列化 (deserialization) 的情况下，对已序列化数据进行一定程度的直接访问，这有利于区块链内的某些操作。
- **RLP**：不支持高效的索引，访问内部数据时可能会导致 `O(N)` 复杂度，这对于大规模网络上的性能来说可能是一个重大缺陷。

**数据类型兼容性 (Data Type Compatibility)**：
- **SSZ**：设计为与以太坊协议中中使用的数据类型和结构完全兼容，从而增强了其在共识机制 (consensus mechanisms) 和网络运行中的实用性。
- **RLP**：虽然灵活，但需要额外的层来支持各种数据类型，这可能会导致实现过程中的低效和复杂性增加。

**确定性序列化 (Deterministic Serialization)**：
- **SSZ**：提供确定性的序列化结果，确保相同的数据结构每次都序列化为完全相同的字节序列，这对于共识的可靠性至关重要。
- **RLP**：RLP 也提供确定性的序列化结果。


出于这些原因，以太坊正在做出巨大努力，以完全迁移到 SSZ 序列化，并停止使用 RLP 序列化。


## SSZ 的工作原理 - 基本类型 (How SSZ Works - Basic Types)

以下是 SSZ 如何处理基本类型的序列化 and 反序列化：

```mermaid
flowchart TD
    A[开始序列化 (Start Serialization)] --> B[选择数据类型 (Choose Data Type)]
    B --> C[无符号整数 (Unsigned Integer)]
    B --> D[布尔值 (Boolean)]
    
    C --> E["将整数转换为 \n 小端序字节数组 (Convert Integer to Little-Endian Byte Array)"]
    E --> F[无符号整数序列化输出 (Serialized Output for Integer)]
    
    D --> G["将布尔值转换为字节 \n (True转换为0x01, False转换为0x00) \n (Convert Boolean to Byte)"]
    G --> H[布尔值序列化输出 (Serialized Output for Boolean)]
    
```

*图：基本类型的序列化流程。*


```mermaid
flowchart TD
    A[开始反序列化 (Start Deserialization)] --> B[确定数据类型 (Determine Data Type)]
    B --> C[无符号整数 (Unsigned Integer)]
    B --> D[布尔值 (Boolean)]
    
    C --> E[读取小端序字节数组 (Read Little-Endian Byte Array)]
    E --> F[重构原始整数值 (Reconstruct Original Integer Value)]
    F --> G[反序列化整数输出 (Deserialized Integer Output)]
    
    D --> H[读取字节 (Read Byte)]
    H --> I["将字节翻译为布尔值 \n (0x01翻译为True, 0x00翻译为False) \n (Translate Byte to Boolean)"]
    I --> J[反序列化布尔值输出 (Deserialized Boolean Output)]
    
```

*图：基本类型的反序列化流程。*

### 无符号整数 (Unsigned Integers)

SSZ 中的无符号整数 (`uintN`) 中的 `N` 可以是 8、16、32、64、128 或 256 位中的任意一种。这些整数被直接序列化为它们的小端序 (little-endian) 字节表示，这种形式非常适合大多数现代计算机架构，并有助于在字节级别进行更容易的操作。

**无符号整数的序列化过程 (Serialization Process for Unsigned Integers)：**

- **输入 (Input)**：获取一个类型为 `uintN` 的无符号整数。
- **转换为字节 (Convert to Bytes)**：将该整数转换为长度为 `N/8` 的字节数组。例如，`uint16` 代表 2 个字节。
- **应用小端序格式 (Apply Little-Endian Format)**：按小端序顺序排列字节，其中最低有效字节存储在最前面。
- **输出 (Output)**：生成的字节数组就是该整数的序列化形式。

**示例 (Example)：**
- 作为 `uint16` 的整数 `1025` 将被序列化为十六进制的 `01 04`。首先，将 `1025` 转换为十六进制，得到 `0x0401`。在小端序格式中，最低有效字节 (Least Significant Byte, LSB) 最先出现。因此，小端序中的 `0x0401` 是 `01 04`。字节数组 `[01, 04]` 就是序列化输出。

**无符号整数的反序列化过程 (Deserialization Process for Unsigned Integers)：**

- **输入 (Input)**：读取表示已序列化 `uintN` 的字节数组。
- **读取小端序字节 (Read Little-Endian Bytes)**：按小端序顺序解释字节以重构整数值。
- **输出 (Output)**：将字节数组转换回整数。

**示例 (Example)：**
- 字节数组 `01 04`（十六进制）被反序列化为整数 `1025`。读取第一个字节 `01` 作为整数的低位部分，读取 `04` 作为高位部分。为了便于人类阅读，重新组装为大端序 (big-endian) 格式时，它会转换回十六进制的 `0401`，这等于十进制的 1025。

### 布尔值 (Booleans)

SSZ 中的布尔值非常简单，每个布尔值都用单个字节表示。

**布尔值的序列化过程 (Serialization Process for Booleans)：**

- **输入 (Input)**：获取一个布尔值 (`True` 或 `False`)。
- **转换为字节 (Convert to Byte)**：
   - 如果布尔值为 `True`，则将其序列化为 `01`（十六进制）。
   - 如果布尔值为 `False`，则将其序列化为 `00`。
- **输出 (Output)**：生成的单个字节就是布尔值的序列化形式。

**示例 (Example)：**
- `True` 变为 `01`。
- `False` 变为 `00`。

**布尔值的反序列化过程 (Deserialization Process for Booleans)：**

- **输入 (Input)**：读取单个字节。
- **解释字节 (Interpret the Byte)**：
   - 字节 `01` 表示 `True`。
   - 字节 `00` 表示 `False`。
- **输出 (Output)**：该字节对应的布尔值。

**示例 (Example)：**
- 字节 `01` 被反序列化为 `True`。
- 字节 `00` 被反序列化为 `False`。

我们可以按照 [说明 (instructions)](https://eth2book.info/capella/appendices/running/)，使用以太坊权益证明规格 (Ethereum PoS spec) 的 Python 代码来运行 SSZ 序列化 and 反序列化命令，并验证上述字节数组。

```python
>>> from eth2spec.utils.ssz.ssz_typing import uint64, boolean
# 序列化 (Serializing) 
>>> uint64(1025).encode_bytes().hex()
'0104000000000000'
>>> boolean(True).encode_bytes().hex()
'01'
>>> boolean(False).encode_bytes().hex()
'00' 

# 反序列化 (Deserializing) 
>>> print(uint64.decode_bytes(bytes.fromhex('0104000000000000')))
1025
>>> print(boolean.decode_bytes(bytes.fromhex('01')))
1
>>> print(boolean.decode_bytes(bytes.fromhex('00')))
0
```

## SSZ 在复合类型上的工作原理 (How SSZ Works on Composite Types)

### 向量 (Vectors)

SSZ 中的向量 (Vectors) 用于处理同构元素的固定长度 (fixed-length) 集合。以下是 SSZ 如何处理向量序列化和反序列化的详细分解。

**向量的 SSZ 序列化 (SSZ Serialization for Vectors)**

```mermaid
flowchart TD
    A[开始序列化 (Start Serialization)] --> B[定义具有类型和长度的向量 (Define Vector with Type and Length)]
    B --> C[序列化每个元素 (Serialize Each Element)]
    C --> D["将每个元素转换为 \n 字节数组 (小端序) \n (Convert to Byte Array (Little-Endian))"]
    D --> E[拼接所有字节数组 (Concatenate All Byte Arrays)]
    E --> F[输出序列化后的向量 (Output Serialized Vector)]
    
```

*图：向量的 SSZ 序列化流程。*


**固定长度定义 (Fixed-Length Definition)**：
- 向量是用特定的长度和它们可以容纳的元素类型来定义的，例如 `Vector[uint64, 4]` 表示一个包含四个 64 位无符号整数的向量。

**元素序列化 (Element Serialization)**：
- 向量中的每个元素都是根据其类型独立序列化的。
- 对于整数或布尔值等基本类型，这意味着将每个元素转换为其字节表示。
- 如果元素是复合类型，则每个元素都根据其特定的序列化规则进行序列化。

**拼接 (Concatenation)**：
- 每个元素的序列化输出按它们在向量中出现的顺序进行拼接 (concatenated)。
- 由于向量的长度和每个元素的大小都是已知且固定的，因此序列化输出中不需要额外的元数据（如长度前缀 (length prefixes)）。

**示例 (Example)：**
- 对于包含元素 `[256, 512, 768]` 的 `Vector[uint64, 3]`，每个元素长度为 64 位或 8 字节。序列化过程如下：

**将每个整数转换为小端序字节数组 (Convert Each Integer to Little-Endian Byte Array)**：
- 作为 `uint64` 的 `256` 变为 `00 01 00 00 00 00 00 00`。
- 作为 `uint64` 的 `512` 变为 `00 02 00 00 00 00 00 00`。
- 作为 `uint64` 的 `768` 变为 `00 03 00 00 00 00 00 00`。

**拼接这些字节数组 (Concatenate These Byte Arrays)**：
- 生成的拼接字节数组将是 `00 01 00 00 00 00 00 00 00 02 00 00 00 00 00 00 00 03 00 00 00 00 00 00`。

**序列化输出 (Serialized Output)**：
- `00 01 00 00 00 00 00 00 00 02 00 00 00 00 00 00 00 03 00 00 00 00 00 00`。


**向量的 SSZ 反序列化 (SSZ Deserialization for Vectors)**

```mermaid
flowchart TD
    A[开始反序列化 (Start Deserialization)] --> B[接收序列化字节流 (Receive Serialized Byte Stream)]
    B --> C["根据元素大小识别并拆分字节流 \n (Identify and Split Byte Stream Based on Element Size)"]
    C --> D["将每个字节段反序列化 \n 为其原始类型 \n (Deserialize Each Byte Segment to Original Type)"]
    D --> E[将元素重新组装为向量 (Reassemble Elements into Vector)]
    E --> F[输出反序列化后的向量 (Output Deserialized Vector)]
    
```

*图：向量的 SSZ 反序列化流程。*


**利用固定长度 (Fixed-Length Utilization)**：
- 反序列化器利用预先定义的向量长度和类型来解析已序列化的数据。
- 它确切地知道每个元素占用多少个字节，以及向量中包含多少个元素。

**元素反序列化 (Element Deserialization)**：
- 字节流被分割为与每个元素大小相对应的片段 (segments)。
- 每个片段都根据向量中元素的类型独立进行反序列化。

**重构 (Reconstruction)**：
- 元素被重构为它们的原始形式（例如，将字节数组转换回整数或其他指定的类型）。
- 然后将这些元素进行聚合，以重新形成原始向量。

**示例 (Example)：**
- 给定 `Vector[uint64, 3]` 的序列化数据：
- 序列化字节数组：`00 01 00 00 00 00 00 00 00 02 00 00 00 00 00 00 00 03 00 00 00 00 00 00`。

**将数据解析为片段 (Parse the Data into Segments)**：
- 每个片段包含 8 字节。
- 第一个片段：`00 01 00 00 00 00 00 00` → 代表整数 256。
- 第二个片段：`00 02 00 00 00 00 00 00` → 代表整数 512。
- 第三个片段：`00 03 00 00 00 00 00 00` → 代表整数 768。

**将每个片段从小端序字节数组转换回整数 (Convert Each Segment from a Little-Endian Byte Array Back to an Integer)**：
- 使用小端序格式，读取每个字节数组并将其转换回相应的 `uint64` 整数。

**重构 (Reconstruction)**：
- 重构后的向量为 `[256, 512, 768]`。

我们可以在 Python 中运行并验证它，如下所示：

```python
>>> from eth2spec.utils.ssz.ssz_typing import uint8, uint16, Vector
>>> Vector[uint16, 3](256, 512, 768).encode_bytes().hex()
'000100000000000000020000000000000003000000000000'
>>> print(Vector[uint64, 3].decode_bytes(bytes.fromhex('000100000000000000020000000000000003000000000000')))
Vector[uint64, 3]<<len=3>>(256, 512, 768)
>>> 
```

### 列表 (Lists)

SSZ 中的列表 (Lists) 对于在指定的最高长度限制 (`N`) 内管理同构元素的可变长度 (variable-length) 集合至关重要。这种灵活性允许动态管理数据结构（例如交易集或可变状态组件），以适应网络不断变化的需求。

**列表的 SSZ 序列化 (SSZ Serialization for Lists)**

```mermaid
flowchart TD
    A[开始序列化 (Start Serialization)] --> B[定义具有类型和最大长度的列表 (Define List with Type and Max Length)]
    B --> C[序列化每个元素 (Serialize Each Element)]
    C --> D["将每个元素转换为 \n 字节数组 (小端序) \n (Convert to Byte Array (Little-Endian))"]
    D --> E[拼接所有字节数组 (Concatenate All Byte Arrays)]
    E --> F[可选：包含长度元数据 (Optional: Include Length Metadata)]
    F --> G[输出序列化后的列表 (Output Serialized List)]
    
```

*图：列表的 SSZ 序列化流程。*

**定义列表 (Define the List)**：
- SSZ 中的列表是用特定的元素类型和最大长度定义的，记作 `List[type, N]`。这一定义不仅约束了列表的最大容量，还决定了元素应当如何被序列化。

**元素序列化 (Element Serialization)**：
- 列表中的每个元素都根据其类型进行序列化。对于 `uint64` 元素，序列化过程包括将每个整数转换为字节数组。

**拼接已序列化的元素 (Concatenate Serialized Elements)**：
- 序列化后的元素输出按顺序拼接。序列化数据的总长度会根据序列化时存在的实际元素数量而发生变化。

**包含长度元数据 (Include Length Metadata - 可选)**：
- 根据具体实现的要求，列表的长度可能会显式地包含在已序列化数据的开头，以协助反序列化过程中的解析和验证。

**示例 (Example)**：
- 对于一个包含元素 `[1024, 2048, 3072]` 的 `List[uint64, 5]`，序列化过程包括：
- 将每个整数转换为小端序格式的字节数组：`00 04 00 00 00 00 00 00`，`00 08 00 00 00 00 00 00`，`00 0C 00 00 00 00 00 00`。
- 拼接这些数组得到：`00 04 00 00 00 00 00 00 00 08 00 00 00 00 00 00 00 0C 00 00 00 00 00 00`。

**列表的 SSZ 反序列化 (SSZ Deserialization for Lists)**

```mermaid
flowchart TD
    A[开始反序列化 (Start Deserialization)] --> B[接收序列化字节流 (Receive Serialized Byte Stream)]
    B --> C["根据元素大小识别并拆分字节流 \n (对于uint64为8字节) \n (Split Byte Stream Based on Element Size)"]
    C --> D[将每个字节段反序列化为uint64 (Deserialize Each Byte Segment to uint64)]
    D --> E[将元素重新组装为列表 (Reassemble Elements into List)]
    E --> F[输出反序列化后的列表 (Output Deserialized List)]
    
```

*图：列表的 SSZ 反序列化流程。*

**接收已序列化的数据 (Receive Serialized Data)**：
- 输入是该列表的已序列化字节流，其中包含每个元素的字节数组序列。

**解析并反序列化每个元素 (Parse and Deserialize Each Element)**：
- 根据元素类型（例如 `uint64`），将已序列化的流解析为 8 字节的片段。
- 将每个字节数组从小端序格式转换回 `uint64`。

**重新组装列表 (Reassemble the List)**：
- 反序列化后的元素被重新组装以重建原始列表。

**示例 (Example)**：
- 给定 `List[uint64, 5]` 的已序列化数据 `00 04 00 00 00 00 00 00 00 08 00 00 00 00 00 00 00 0C 00 00 00 00 00 00`：
- 将数据拆分为 8 字节的片段：`00 04 00 00 00 00 00 00`，`00 08 00 00 00 00 00 00`，`00 0C 00 00 00 00 00 00`。
- 将每个片段从小端序转换为整数：`1024`，`2048`，`3072`。
- 重建后的列表为 `[1024, 2048, 3072]`。

我们可以运行并验证上述示例的 SSZ，如下所示：

```python
>>> from eth2spec.utils.ssz.ssz_typing import uint8, List, Vector
>>> List[uint64, 5](1024, 2048, 3072).encode_bytes().hex()
'00040000000000000008000000000000000c000000000000'
>>> Vector[uint64, 3](1024, 2048, 3072).encode_bytes().hex()
'00040000000000000008000000000000000c000000000000'
>>> print(List[uint64, 5].decode_bytes(bytes.fromhex('00040000000000000008000000000000000c000000000000')))
List[uint64, 5]<<len=3>>(1024, 2048, 3072)
>>> 
```

在 SSZ 中，当可变大小的对象 (variable sized objects) 包含在另一个对象中时，其编码方式与固定大小的向量不同，因此存在少量的开销。例如，下面的 `Alice` 和 `Bob` 对象具有不同的编码：

```python
>>> from eth2spec.utils.ssz.ssz_typing import uint8, Vector, List, Container
>>> class Alice(Container):
...     x: List[uint8, 3] # 可变大小 (Variable sized)
>>> class Bob(Container):
...     x: Vector[uint8, 3] # 固定大小 (Fixed sized)
>>> Alice(x = [1, 2, 3]).encode_bytes().hex()
'04000000010203'
>>> Bob(x = [1, 2, 3]).encode_bytes().hex()
'010203'
>>> 
```

### 位向量 (Bitvectors)

SSZ 中的位向量 (Bitvectors) 用于管理布尔值的固定长度序列，通常表示为位 (bits)。这种数据结构对于紧凑地存储二进制 data 或标志 (flags) 特别高效，这在以太坊应用中非常常见，用于指示状态条件、权限或其他二进制设置。

**位向量的 SSZ 序列化 (SSZ Serialization for Bitvectors)**

```mermaid
flowchart TD
    A[开始序列化 (Start Serialization)] --> B[定义大小为 N 的位向量 (Define Bitvector of Size N)]
    B --> C[将位打包为字节 (Pack Bits into Bytes)]
    C --> D["在每个字节中，位按从 \n LSB 到 MSB 的顺序排列 \n (Bits from LSB to MSB)"]
    D --> E[如果 N % 8 != 0，则添加填充 (Add Padding)]
    E --> F[输出序列化后的字节数组 (Output Serialized Byte Array)]
    
```

*图：位向量的 SSZ 序列化流程。*

**定义位向量 (Define the Bitvector)**：
- SSZ 中的位向量由其长度 `N` 定义，它指定了位的数量。例如，`Bitvector[256]` 表示一个包含 256 位的位向量。

**将位转换为字节 (Convert Bits to Bytes)**：
- 位向量中的每一位都代表一个布尔值，其中 `0` 对应 `False`，`1` 对应 `True`。
- 这些位被打包成字节，每个字节中最低有效位 (LSB) 排在最前面。这意味着位向量中的第一位对应于第一个字节的 LSB。

**字节数组的形成 (Byte Array Formation)**：
- 通过将每 8 位打包进一个字节，直到所有位都被处理完，从而将这些位序列化为字节数组。
- 如果 `N` 不是 8 的倍数，最后一个字节将包含少于 8 位的数据，并在最高有效位 (MSB) 位置填充零 (padded with zeros)。

**示例 (Example)**：对于一个模式为 `1011010010` 的 `Bitvector[10]`：
- 前 8 位 (`10110100`) 组成第一个字节。
- 剩余的 2 位 (`10`) 填充六个零以组成第二个字节：`10000000`。
- 序列化输出为十六进制的 `B4 80`。

**位向量的 SSZ 反序列化 (SSZ Deserialization for Bitvectors)**

```mermaid
flowchart TD
    A[开始反序列化 (Start Deserialization)] --> B[接收序列化字节数组 (Receive Serialized Byte Array)]
    B --> C[读取每个字节 (Read Each Byte)]
    C --> D[将字节转换为位 (Convert Bytes to Bits)]
    D --> E["遵守每个字节中从 \n LSB 到 MSB 的顺序 \n (LSB to MSB Order in Each Byte)"]
    E --> F[若有填充，丢弃填充位 (Discard Padding if Present)]
    F --> G[重构位向量 (Reconstruct Bitvector)]
    G --> H[输出反序列化后的位向量 (Output Deserialized Bitvector)]
    
```

*图：位向量的 SSZ 反序列化流程。*

**读取已序列化的字节数组 (Read Serialized Byte Array)**：
- 从对位向量进行编码的字节数组开始。

**从字节中提取位 (Extract Bits from Bytes)**：
- 将每个字节转换回位。请记住，在每个字节中，位是以 LSB 优先的顺序存储的。
- 如果位向量'的长度 `N` 不是 8 的倍数，请丢弃最后一个字节中无关的填充位 (padding bits)。

**重构位向量 (Reconstruct the Bitvector)**：
- 将提取出的位重新组装为原始位向量格式，并遵守指定的长度 `N`。

**示例 (Example)**：给定 `Bitvector[10]` 的已序列化数据 `B4 80`：
- 将 `B4`（二进制为 `10110100`）和 `80`（二进制为 `10000000`）转换回位。
- 从二进制序列中提取前 10 位：`1011010010`。
- 重构后的位向量为 `1011010010`。

您可以在 Python 中运行并验证它，如下所示：

```python
>>> from eth2spec.utils.ssz.ssz_typing import Bitvector
>>> Bitvector[8](0,0,1,0,1,1,0,1).encode_bytes().hex()
'b4'
>>> Bitvector[8](0,0,0,0,0,0,0,1).encode_bytes().hex()
'80'
```

请注意，在功能上，我们可以使用 `Vector[boolean, N]` 或 `Bitvector[N]` 来表示位列表。然而，在实践中，后者的序列化结果将比前者短多达八倍，因为前者对每个位都使用一个完整的字节。

```python
>>> from eth2spec.utils.ssz.ssz_typing import Vector, Bitvector, boolean
>>> Bitvector[5](1,0,1,0,1).encode_bytes().hex()
'15'
>>> Vector[boolean,5](1,0,1,0,1).encode_bytes().hex()
'0100010001'
```

### 位列表 (Bitlists)

SSZ 中的位列表 (Bitlists) 与位向量类似，但旨在处理具有指定最大长度 (`N`) 的布尔值可变长度序列。

**位列表的 SSZ 序列化 (SSZ Serialization for Bitlists)**

```mermaid
flowchart TD
    A[开始序列化 (Start Serialization)] --> B[定义大小为 N 的位列表 (Define Bitlist of Size N)]
    B --> C[将位打包为字节 (Pack Bits into Bytes)]
    C --> D[添加哨兵位 (Add Sentinel Bit)]
    D --> E[必要时填充最后一个字节 (Pad Final Byte if Necessary)]
    E --> F[输出序列化后的字节数组 (Output Serialized Byte Array)]
    
```

*图：位列表的 SSZ 序列化流程。*


**定义位列表 (Define the Bitlist)**：
- 位列表由其最大长度 `N` 定义，这决定了可以包含的位的上限。然而，实际的位数可以少于 `N`。

**将位打包为字节 (Pack Bits into Bytes)**：
- 位列表中的每一位都代表一个布尔值，其中 `0` 对应 `False`，`1` 对应 `True`。
- 与位向量类似，这些位被序列化为字节数组，在每个字节中从 LSB 到 MSB 进行打包。

**添加哨兵位 (Add Sentinel Bit)**：
- 为了标记位列表的结束并将其真实长度与最大容量区分开来，在位序列的末尾添加了一个哨兵位 (sentinel bit)（`1`）。这对于确保反序列化过程能够准确识别位列表的长度至关重要。

**字节数组的形成与填充 (Byte Array Formation and Padding)**：
- 包含哨兵位后，位会被打包成字节。如果总位数（包括哨兵位）不能被 8 整除，则会对最后一个字节进行必要的填充，以使其完整。


**位列表的 SSZ 反序列化 (SSZ Deserialization for Bitlists)**

```mermaid
flowchart TD
    A[开始反序列化 (Start Deserialization)] --> B[接收序列化字节数组 (Receive Serialized Byte Array)]
    B --> C[将字节转换为位 (Convert Bytes to Bits)]
    C --> D[识别并移除哨兵位 (Identify and Remove Sentinel Bit)]
    D --> E[移除填充位 (Remove Padding Bits)]
    E --> F[重构原始位列表 (Reconstruct Original Bitlist)]
    F --> G[输出反序列化后的位列表 (Output Deserialized Bitlist)]
    
```

*图：位列表的 SSZ 反序列化流程。*

**接收已序列化的字节数组 (Receive Serialized Byte Array)**：
- 从对位列表（包含哨兵位）进行编码的字节数组开始。

**从字节中提取位 (Extract Bits from Bytes)**：
- 将每个字节转换回位，并遵守顺序（LSB 到 MSB）。
- 对已序列化数据中的每个字节继续此过程。

**识别并移除哨兵位 (Identify and Remove the Sentinel Bit)**：
- 在提取位时，从位序列的末尾定位第一个 `1`（哨兵位），以确定位列表数据的实际结束位置。
- 哨兵位之后的防干扰位全部被视为填充位并予以忽略。

**重构位列表 (Reconstruct the Bitlist)**：
- 将提取出的位（不包括哨兵位和任何填充）重新组装为原始位列表格式。

您可以像下面这样运行 Bitlist 的编码：

```python
>>> from eth2spec.utils.ssz.ssz_typing import Bitlist
>>> Bitlist[100](0,0,0).encode_bytes().hex()
'08'
```

由于引入了哨兵位，如果位列表的实际长度是 8 的倍数（与最大长度 `N` 无关），我们需要一个额外的字节来对其进行序列化。而位向量则不需要。

```python
>>> Bitlist[8](0,0,0,0,0,0,0,0).encode_bytes().hex()
'0001'
>>> Bitvector[8](0,0,0,0,0,0,0,0).encode_bytes().hex()
'00'
```

### 容器 (Containers)

SSZ 中的容器 (Containers) 是用于将多个字段组合成单个复合类型的基本结构。容器内的每个字段都可以是 SSZ 支持的任何类型，包括 `uint64` 等基本类型，或者其他容器、向量或列表等更复杂的类型。容器类似于编程语言中的结构体 (structures) 或对象 (objects)，这使得它们对于在以太坊中表示复杂的嵌套数据结构不可或缺。

**容器的 SSZ 序列化 (SSZ Serialization for Containers)**

```mermaid
flowchart TD
    A[开始序列化 (Start Serialization)] --> B[定义容器模式 (Define Container Schema)]
    B --> C[根据类型序列化每个字段 (Serialize Each Field According to Type)]
    C --> D["序列化基本类型\n (uint64, 布尔值等)\n (Serialize Basic Types)"]
    C --> E["序列化复合类型\n (其他容器、列表、向量)\n (Serialize Composite Types)"]
    D --> F[拼接字段的序列化输出 (Concatenate Serialized Outputs of Fields)]
    E --> F
    F --> G[输出序列化后的容器 (Output Serialized Container)]
    
```

*图：容器的 SSZ 序列化流程。*

**定义容器 (Define the Container)**：
- SSZ 中的容器由其模式 (schema) 定义，该模式指定了其字段的类型和顺序。这种模式至关重要，因为它决定了数据应该如何进行序列化和反序列化。

**序列化每个字段 (Serialize Each Field)**：
- 容器中的每个字段按照模式定义的顺序进行序列化。
- 每个字段的序列化方法取决于其类型：
- **基本类型**直接转换为它们的字节表示。
- **复合类型**（其他容器、列表、向量）根据它们各自的规则递归地进行序列化。

**拼接已序列化的字段 (Concatenate Serialized Fields)**：
- 将所有字段的序列化输出拼接起来，形成容器的完整序列化数据。
- 如果某个字段是可变大小的（如列表或具有可变长度的向量），其序列化数据将包含长度前缀，或者可以使用偏移量 (offsets) 来指示数据的起点，这取决于具体实现和类型的细节。

**容器的 SSZ 反序列化 (SSZ Deserialization for Containers)**

```mermaid
flowchart TD
    A[开始反序列化 (Start Deserialization)] --> B[接收序列化容器数据 (Receive Serialized Container Data)]
    B --> C["根据容器模式解析数据 \n (Parse Data According to Container Schema)"]
    C --> D[根据类型反序列化字段 (Deserialize Fields Based on Type)]
    D --> E[反序列化基本类型 (Deserialize Basic Types)]
    D --> F[反序列化复合类型 (Deserialize Composite Types)]
    E --> G[使用反序列化字段重构容器 (Reconstruct Container)]
    F --> G
    G --> H[输出反序列化后的容器 (Output Deserialized Container)]
    
```

*图：容器的 SSZ 反序列化流程。*

**读取已序列化的数据 (Read Serialized Data)**：
- 从代表该容器的已序列化字节流开始。

**根据模式解析已序列化的数据 (Parse Serialized Data According to Schema)**：
- 根据容器的模式，将序列化数据解析为其组成字段。
- 这需要了解每个字段的类型和大小，以便正确地提取和反序列化每一个字段。

**反序列化每个字段 (Deserialize Each Field)**：
- 每个字段的数据根据其类型进行反序列化。
- 反序列化可能涉及将字节数组转换回整数、对嵌套容器进行解码，或从序列化形式重建列表和向量。

**重构容器 (Reconstruct the Container)**：
- 随着每个字段被反序列化，通过将每个字段放回其定义的相应位置来重构容器。

**示例 (Example)**：

让我们通过信标链中 `IndexedAttestation` 容器的特定示例来深入了解 SSZ 序列化和反序列化过程。本例将概述如何处理和运行复杂的嵌套容器，特别是那些同时涉及固定大小和可变大小数据类型的容器。

`IndexedAttestation` 容器结构如下：

```python
class IndexedAttestation(Container):
    attesting_indices: List[ValidatorIndex, MAX_VALIDATORS_PER_COMMITTEE]
    data: AttestationData
    signature: BLSSignature
```

It contains an `AttestationData` container,

```python
class AttestationData(Container):
    slot: Slot
    index: CommitteeIndex
    beacon_block_root: Root
    source: Checkpoint
    target: Checkpoint
```

which in turn contains two `Checkpoint` containers,

```python
class Checkpoint(Container):
    epoch: Epoch
    root: Root
```    

**IndexedAttestation 容器结构**

`IndexedAttestation` 容器包括多个字段，其中一些是固定大小的基本类型，另一些是复合类型，包括另一个容器 (`AttestationData`) 和列表（如 `attesting_indices`）。

其结构如下：

- **attesting_indices**：`List[ValidatorIndex, MAX_VALIDATORS_PER_COMMITTEE]`（可变大小）
- **data**：`AttestationData`（复合容器）
- **signature**：`BLSSignature`（固定大小）

**AttestationData 容器结构**

- **slot**：`Slot`（固定大小）
- **index**：`CommitteeIndex`（固定大小）
- **beacon_block_root**：`Root`（固定大小）
- **source**：`Checkpoint`（复合容器）
- **target**：`Checkpoint`（复合容器）

**Checkpoint 容器结构**
- **epoch**：`Epoch`（固定大小）
- **root**：`Root`（固定大小）

**序列化过程 (Serialization Process)**

- **序列化固定和可变组件 (Serialize Fixed and Variable Components)**
  - `IndexedAttestation` 的序列化涉及根据每个组件的类型对其进行序列化：

- **序列化固定大小元素 (Serialize Fixed-Size Elements)**
  - 每个固定大小的元素（`Slot`、`CommitteeIndex`、`Epoch`、`Root`、`BLSSignature`）都会被序列化为其相应的字节格式，对于数值类型通常为小端序。

- **序列化可变大小元素 (Serialize Variable-Size Elements)**
  - 对 `List[ValidatorIndex, MAX_VALIDATORS_PER_COMMITTEE]` 进行序列化时，首先记录列表的长度，然后是每个索引的序列化形式。
  - 如果列表或其他可变大小元素为空或未达到最大容量，它只消耗实际存在的数据所需的空间，加上可能的一些长度或偏移量元数据。

- **拼接已序列化的数据 (Concatenate Serialized Data)**
  - 所有已序列化的字节将按照容器结构指定的顺序进行拼接。固定大小的字段将直接按顺序放置，而可变大小的字段可能会将偏移量或长度作为序列化的一部分包含在内。

**示例序列化输出 (Example Serialization Output)**

```python
from eth2spec.utils.ssz.ssz_typing import *
from eth2spec.capella import mainnet
from eth2spec.capella.mainnet import *

attestation = IndexedAttestation(
    attesting_indices = [33652, 59750, 92360],
    data = AttestationData(
        slot = 3080829,
        index = 9,
        beacon_block_root = '0x4f4250c05956f5c2b87129cf7372f14dd576fc152543bf7042e963196b843fe6',
        source = Checkpoint (
            epoch = 96274,
            root = '0xd24639f2e661bc1adcbe7157280776cf76670fff0fee0691f146ab827f4f1ade'
        ),
        target = Checkpoint(
            epoch = 96275,
            root = '0x9bcd31881817ddeab686f878c8619d664e8bfa4f8948707cba5bc25c8d74915d'
        )
    ),
    signature = '0xaaf504503ff15ae86723c906b4b6bac91ad728e4431aea3be2e8e3acc888d8af'
                + '5dffbbcf53b234ea8e3fde67fbb09120027335ec63cf23f0213cc439e8d1b856'
                + 'c2ddfc1a78ed3326fb9b4fe333af4ad3702159dbf9caeb1a4633b752991ac437'
)

print(attestation.encode_bytes().hex())
```

表示此 `IndexedAttestation` 对象的最终序列化数据块 (blob of data) 是（十六进制）：

```code
e40000007d022f000000000009000000000000004f4250c05956f5c2b87129cf7372f14dd576fc15
2543bf7042e963196b843fe61278010000000000d24639f2e661bc1adcbe7157280776cf76670fff
0fee0691f146ab827f4f1ade13780100000000009bcd31881817ddeab686f878c8619d664e8bfa4f
8948707cba5bc25c8d74915daaf504503ff15ae86723c906b4b6bac91ad728e4431aea3be2e8e3ac
c888d8af5dffbbcf53b234ea8e3fde67fbb09120027335ec63cf23f0213cc439e8d1b856c2ddfc1a
78ed3326fb9b4fe333af4ad3702159dbf9caeb1a4633b752991ac437748300000000000066e90000
00000000c868010000000000
```

**序列化输出的分步解析 (Breakdown of the Serialization Output)**

为了清楚地解释本例中 `IndexedAttestation` 容器的序列化过程和已序列化数据的结构，让我们将序列化拆分为其各个组成部分，并理解每一部分在字节流中是如何表示的。这种拆解有助于说明 SSZ 格式如何管理复杂的数据结构。

**第 1 部分：固定大小元素 (Part 1: Fixed Size Elements)**

**可变大小列表 (`attesting_indices`) 的 4 字节偏移量 (4-byte Offset for Variable Size List (`attesting_indices`))**：
- **字节偏移量 (Byte Offset)**：`00`
- **值 (Value)**：`e4000000`
- **解释 (Explanation)**：这表示已序列化字节流中 `attesting_indices` 列表的起点。十六进制值 `e4` 转换为十进制为 `228`，意味着该列表从字节流开头的第 `228` 字节处开始。

**插槽 (Slot) (uint64)**：
- **字节偏移量 (Byte Offset)**：`04`
- **值 (Value)**：`7d022f0000000000`
- **解释 (Explanation)**：代表序列化为 64 位无符号整数的 `slot` 字段。小端序格式下的十六进制 `7d022f00` 转换为十进制为 `3080829`，即插槽号。

**委员会索引 (Committee Index) (uint64)**：
- **字节偏移量 (Byte Offset)**：`0c`
- **值 (Value)**：`0900000000000000`
- **解释 (Explanation)**：这是 `index` 字段，表示作为 64 位无符号整数的委员会索引。值 `09` 表示委员会索引 `9`。

**信标区块根 (Beacon Block Root) (Bytes32)**：
- **字节偏移量 (Byte Offset)**：`14`
- **值 (Value)**：`4f4250c05956f5c2b87129cf7372f14dd576fc152543bf7042e963196b843fe6`
- **解释 (Explanation)**：这是一个存储为 `Bytes32` 的 256 位哈希，表示信标区块的根哈希。

**源检查点时期 (Source Checkpoint Epoch) (uint64) 与根 (Root) (Bytes32)**：
- **时期字节偏移量 (Epoch Byte Offset)**：`34`
- **时期值 (Epoch Value)**：`1278010000000000`
- **根字节偏移量 (Root Byte Offset)**：`3c`
- **根值 (Root Value)**：`d24639f2e661bc1adcbe7157280776cf76670fff0fee0691f146ab827f4f1ade`
- **解释 (Explanation)**：源检查点包含一个 `epoch`（96274）和一个 `root`。该根是另一个 256 位哈希。

**目标检查点时期 (Target Checkpoint Epoch) (uint64) 与根 (Root) (Bytes32)**：
- **时期字节偏移量 (Epoch Byte Offset)**：`5c`
- **时期值 (Epoch Value)**：`1378010000000000`
- **根字节偏移量 (Root Byte Offset)**：`64`
- **根值 (Root Value)**：`9bcd31881817ddeab686f878c8619d664e8bfa4f8948707cba5bc25c8d74915d`
- **解释 (Explanation)**：与源检查点类似，目标检查点包括一个 `epoch`（96275）和一个 `root`，详细说明了该见证的预期目标。

**签名 (Signature) (BLSSignature/Bytes96)**：
- **字节偏移量 (Byte Offset)**：`84`
- **值 (Value)**：由于长度原因拼接在多行中（共 96 字节）。
- **解释 (Explanation)**：这是见证的密码学签名 (cryptographic signature)，用于验证其真实性。

**第 2 部分：可变大小元素 (Part 2: Variable Size Elements)**

**参与见证的索引 (Attesting Indices) (List[uint64, MAX_VALIDATORS_PER_COMMITTEE])**：
- **字节偏移量 (Byte Offset)**：`e4`
- **值 (Value)**：`748300000000000066e9000000000000c868010000000000`
- **解释 (Explanation)**：这表示对该区块进行见证的验证者索引列表。它从偏移量 `228` 开始，包含诸如 `33652`、`59750` 和 `92360` 的索引。


## 资源 (Resources)
- [Simple serialize](https://ethereum.org/en/developers/docs/data-structures-and-encoding/ssz/)
- [SSZ specs](https://github.com/ethereum/consensus-specs/blob/dev/ssz/simple-serialize.md)
- [eth2book - SSZ](https://eth2book.info/capella/part2/building_blocks/ssz/#ssz-simple-serialize)
- [Go Lessons from Writing a Serialization Library for Ethereum](https://rauljordan.com/go-lessons-from-writing-a-serialization-library-for-ethereum/)
- [Interactive SSZ serializer/deserializer](https://www.ssz.dev/)
- [SSZ encoding diagrams by Protolambda](https://github.com/protolambda/eth2-docs#ssz-encoding)
- [SSZ explainer by Raul Jordan](https://rauljordan.com/go-lessons-from-writing-a-serialization-library-for-ethereum/)
- [SSZ Specifications](https://github.com/ethereum/consensus-specs/blob/v1.3.0/ssz/simple-serialize.md)
- [Why Ethereum Clients prefer SSZ over RLP?](https://etherworld.co/2023/01/25/why-ethereum-clients-prefer-ssz-over-rlp/)