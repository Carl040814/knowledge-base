# 共识层规范测试 (Consensus-spec tests)

> :warning: 本文是一个待完善的草稿 (stub)，欢迎通过 [做出贡献](/contributing.md) 并对其进行扩展来帮助维基。

[consensus-spec-tests](https://github.com/ethereum/consensus-spec-tests) 代码库包含了从以太坊共识规范中生成的一系列测试向量 (test vectors)。
这些测试用于验证共识客户端实现 (consensus client implementations) 是否符合 [以太坊共识层规范 (Ethereum consensus-specs)](https://github.com/ethereum/consensus-specs)。

客户端开发者使用这些测试来确认协议的更改在所有共识客户端中均已得到一致的实现。

---

> ### 实践说明 (Practical notes)
>
> `consensus-spec-tests` 代码库非常庞大，可能会占用数吉字节 (gigabytes) 的磁盘空间。
> 当运行多个依赖相同测试向量的共识客户端时，技术上可以通过创建**符号链接 (symlinks)**，将每个客户端所需的测试目录指向单个共享的 `consensus-spec-tests` 副本，从而节省空间。
>
> 然而，这种方法伴随着风险：
> - 不同的客户端可能依赖**不同版本**的测试套件。
>   使用错误的版本可能会导致**假阴性 (false negatives)**（即使客户端实现正确，测试也会失败）。
>     ![consensus-spec-tests-different-versions](./img/consensus-spec-tests-different-versions.png)
>    
> - 手动切换 (`git checkout`) 到客户端所需的测试版本并重新运行测试，可能会导致 **解压 SSZ Snappy 文件出现问题**，从而导致测试失败。
>     ![consensus-spec-tests-incorrect-unpacking-SSZ](./img/consensus-spec-tests-incorrect-unpacking-SSZ.png)
> > 参见具体的 [问题 (issue)](https://github.com/sntntn/grandine/issues/15) 示例。
> >
> > 因此，**推荐的做法**是为每个共识客户端**保留单独的副本**，即使这需要占用更多的磁盘空间。




相关资源：  
- [consensus-spec-tests 仓库](https://github.com/ethereum/consensus-spec-tests)  
- [以太坊共识层规范 (Ethereum consensus-specs)](https://github.com/ethereum/consensus-specs)（测试生成的真理之源）  
- [简洁序列化 (SimpleSerialize, SSZ) 规范](https://ethereum.github.io/consensus-specs/ssz/simple-serialize/)  
