# 共识层规范

> :warning: 本文是一个[存根 (stub)](https://en.wikipedia.org/wiki/Wikipedia:Stub)，请通过[贡献](/contributing.md)来帮助完善 wiki 并扩展它。

以太坊网络最初作为一个工作量证明 (Proof-of-Work) 区块链启动，意图在其引导阶段后切换到权益证明 (Proof-of-Stake)。研究产生了一种结合 Casper 和 GHOST 的共识机制，在 [Gasper 论文](https://arxiv.org/abs/2003.03052)中发布。

基于其设计，用 Python 编写了一份规范。[Pyspec](https://github.com/ethereum/consensus-specs) 是一个可执行规范，作为共识层开发者的参考。它也被用作客户端实现的参考，以及为客户端创建测试用例向量。

## 资源

[如何使用可执行共识 Pyspec，Hsiao-Wei Wang | Devcon Bogotá](https://www.youtube.com/watch?v=ZDUfYJkTeYw)