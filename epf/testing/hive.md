# Hive 测试 (Hive testing)

> :warning: 本文是一个待完善的草稿 (stub)，欢迎通过 [做出贡献](/contributing.md) 并对其进行扩展来帮助维基。

[Hive](https://github.com/ethereum/hive) 是一个端到端测试工具套件 (end-to-end testing harness)，允许模拟器 (simulators) 在具有不同测试场景（模拟）的单一网络中[启动各种客户端 (clients)](https://github.com/ethereum/hive/blob/master/docs/clients.md)。这是一种在相同参数下同时测试所有客户端以确保没有客户端发生故障的简便得多的方法。这些测试还具有非常具体的边缘情况 (edge cases)，可以针对任何可能的边缘情况和模拟/场景进行修改。由于这种简便性，Hive 在测试即将到来的分叉 (forks) 和网络升级 (network upgrades) 中发挥着至关重要的作用。

如果您感到好奇并想运行一些您自己的测试和模拟，请[安装 Hive (install Hive)](https://github.com/ethereum/hive/blob/master/docs/commandline.md)！

这里是 [Hive 模拟器 (Hive Simulators) 列表](https://github.com/ethereum/hive/tree/master/simulators)

[Hive 模拟器编程指南 (The Hive Simulator Programming guide)](https://github.com/ethereum/hive/blob/master/docs/simulators.md) 还解释了如何编写 Hive 模拟器。
