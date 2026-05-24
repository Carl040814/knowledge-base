# Besu 执行客户端

这是开始为 Besu（以太坊执行客户端的 Java 实现）贡献代码时需要了解的重要信息摘要。

代码仓库: https://github.com/hyperledger/besu/
文档: https://besu.hyperledger.org

## 代码目录说明

### 模块
+  这是一个多模块 [gradle](https://gradle.org/) 项目。你可以查看 settings.gradle 来了解所有模块：
    + 每个模块有自己的 build.gradle：
		+ 可以指定其模块名称：`archiveBaseName`
		+ 可以指定其依赖项但不带版本号。
	+ 每个模块有自己的源代码在此: /src/main/java
+ **顶级模块有**：
	+ `config`:
		+ 大多数配置在此组装和验证。
		+ 可以找到创世 (genesis) 信息。
	+ `besu`:
		+ 所有 CL 参数在此定义。
		+ main 方法所在位置。
+ **补充模块有**：
	+ `crypto`:
		+ 与密码学密钥相关的所有内容。
	+ `data types`:
		+ Besu 使用的数据类型。
	+ `metrics`:
		+ 不与 Besu 内部紧密绑定的 OpenTelemetry/Prometheus。
	+ `ethereum`:
		+ 不是一个模块，但包含模块：
			+ api :
				+ 你想与以太坊、世界状态进行的所有交互。
			+ core:
				+ 存储数据、quorum 设置。
	+ `evm`:
		+ EVM 行为
		+ 在此模块中可以找到每个操作码操作的实现。
+ **企业模块有：**
	+ enclave, plugin-api, privacy-contracts

### Gradle
+ gradlew（文件）：
	+ 一个 bash 脚本，会检查 gradle 是否已安装（如果没有，会为你安装 wrapper 并下载整个发行版）
	+ Gradle 本身作为 wrapper 的一部分管理，通过调用此脚本来使用。
+ gradle（文件夹）：
	+ gradle-wrapper.properties：
		+ distributionURL：指向调用 ./gradlew 时要使用的发行版
	+ versions.gradle（文件）：
		+ 在此定义所有模块的版本。由 gradlew 使用。
+ build.gradle（文件）：
	+ plugins:
		+ `spotless`: 代码格式化，检查许可等。
			+ 命令: `./gradlew spotlessApply`
		+ `errorprone`: Java 最佳实践合规。
			+ 命令: `./gradlew errorProne`
	+ distribution:
		+ 定义了构建输出的位置，将所有项目合并为一个应用程序：
			+ .tar .zip 发行版。可以在 builder/distributions 下看到 .tar 或 .zip。
+ build（文件夹）：
	+ 不是一个模块。
	+ distributions（文件夹）：
		+ Besu 发行版的位置。
		+ 如果深入进入 build/distribution/besu-{version}-SNAPSHOT/lib，可以看到每个组件和库的版本。

### 测试
+ 单元测试 (Unit tests):
	+ 每个模块在 src/test/java 下有单元测试
+ 集成测试 (Integration tests):
	+ 每个模块在 src/integration-test/java 下有集成测试：
	+ 较为少见。
	+ 在代码内部之外运行。
	+ 涉及更昂贵的运行。
+ 验收测试 (Acceptance tests):
	+ 位于 acceptance-test-module 下。
	+ 运行多个 Besu 节点以在它们之间创建共识算法，并执行调整和传播区块的任务。
+ 参考测试 (Reference tests):
	+ 取自以太坊测试，由以太坊基金会借出。
	+ 对所有客户端相同：https://github.com/ethereum/tests
	+ 以 JSON 存储：
		+ 位置：`ethereum/referencetests/`
+ 其他信息：
	+ JUnit 4

### 开发任务
+ 一些有用的命令：
	+ `git pull --recurse-submodules`
	+ `./gradlew spotlessApply`
	+ `./gradlew check`（每次向 Besu 仓库提交 PR 时，CI 会运行此命令）
	+ `./gradlew assemble`
	+ 如果你想将你的 MetaMask 连接到本地 Besu 节点，应使用以下选项运行 Besu：
		+ bin/besu --network=dev --rpc-http-enabled --rpc-http-cors-origins=chrome-extension://nkbihfbeogaeaoehlefnkodbefgpgknn
		+ RPC URL: http://localhost:8545

### 重要类
+ `BesuControllerBuilder`:
	+ 管理将要使用并用于设置客户端的所有组件。
	+ 在 build 方法上，可以看到是否构建了正确的域对象。
	+ 返回一个 BesuController。
+ `BesuCommand`:
	+ 表示主要的 Besu CLI 命令。
+ `ForkIdManager`:
    + 负责构建和表示最新同步的分叉。
	+ 我们总是需要知道自己在链中的位置。此检查持续进行。
	+ 在 EthProtocolManager 中创建，EthProtocolManager 在 BesuControllerBuilder 中创建。
+ `ProtocolSchedule`:
	+ 跟踪链的特定区块编号范围内所有配置项。
+ `ProtocolSpec`:
	+ 允许你配置协议内部运作的每个方面。
+ `MainnetProtocolSpec`:
	+ 可以找到自 frontier 以来的每个规范。
	+ 每个新规范在前一个规范的基础上构建，根据需要添加或更改内容。
+ `MainnetEVMs`:
	+ 为主网硬分叉向 EVM 提供适当的操作。
	+ 一个聚合状态，大多数时候你会在此向规范本身添加新功能。
	+ 新操作将在此注册。
+ `JsonRpcMethodsFactory`:
	+ RPC 方法的构建器类。
	+ 从此处可以了解如何创建新的 RPC 方法。


### 重要库
+ https://doc.libsodium.org/:
	+ Sodium 是一个现代、易于使用的软件库，用于加密、解密、签名、密码哈希等。
+ https://github.com/nss-dev/nss:
	+ Network Security Services (NSS) 是一组旨在支持跨平台开发安全客户端和服务器应用程序的库。NSS 支持 TLS 1.2、TLS 1.3、PKCS #5、PKCS#7、PKCS #11、PKCS #12、S/MIME、X.509 v3 证书和其他安全标准。

## 参考

+ https://www.youtube.com/watch?v=4pCxwuNRaKg
+ https://github.com/hyperledger/besu
+ https://wiki.hyperledger.org/display/besu
