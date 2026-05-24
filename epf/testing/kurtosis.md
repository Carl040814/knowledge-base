# 用于本地开发网的 Kurtosis (Kurtosis for Local Devnets)

## 概述 (Overview) 
[**Kurtosis**](https://docs.kurtosis.com/) 是一个开发和测试平台，旨在高效地打包和启动容器化服务 (containerized services) 的环境。Kurtosis 可作为多容器测试环境 (multi-container test environments) 的构建系统，帮助开发人员搭建可在本地轻松复现的以太坊网络实例。构建在 Kurtosis 之上的 [**以太坊软件包 (Ethereum package)**](https://github.com/ethpandaops/ethereum-package) 允许使用 Docker 或 Kubernetes 快速搭建一个可定制、可扩展且私有的以太坊测试网。它支持所有主要的执行层 (Execution Layer, EL) 和共识层 (Consensus Layer, CL) 客户端，高效地管理本地端口映射 (port mappings) 和服务连接，用于以太坊核心基础设施的验证和测试。

本文简要介绍了 Kurtosis，如何安装和使用它，以及一些用于以太坊开发网 (devnets) 的基本命令。

## 安装 Kurtosis (Installing Kurtosis)

在安装 Kurtosis 之前，请确保您的系统上预先安装了以下依赖项：

- Docker（运行 Kurtosis 容器所需）
- Git（用于克隆代码库）

您可以按照 [此处](https://docs.kurtosis.com/install) 的官方安装指南安装 Kurtosis。

安装完成后，运行以下命令验证您的设置：

```sh
kurtosis version
```

有关升级说明，请参阅 [Kurtosis 升级指南 (Kurtosis upgrade guide)](https://docs.kurtosis.com/upgrade)。

## Kurtosis 引擎 (Kurtosis Engine)

Kurtosis 引擎 (Kurtosis engine) 是运行开发网基础设施的核心服务。一旦您启动开发网，它就会自动启动。然而，这里有一些用于与引擎交互的有用命令：

```sh
# 启动引擎
kurtosis engine start

# 停止引擎
kurtosis engine stop

# 检查引擎的状态
kurtosis engine status
```

## 快速入门：以太坊软件包 (Quick Start: Ethereum Package)

您可以按照 [ethereum-package GitHub 页面](https://github.com/kurtosis-tech/ethereum-package) 的说明，使用以太坊软件包快速搭建默认的以太坊开发网。

安装后启动开发网的最短路径是运行带默认配置的软件包：

```sh
kurtosis run github.com/ethpandaops/ethereum-package
```

运行中的隔离区 (enclave) 状态将会显示出来：

![Kurtosis quick start terminal](./img/kurtosis-quick-start.png)

运行以下命令打开 Kurtosis 网页界面 (web interface)：

```sh
kurtosis web
```

![Kurtosis web interface](./img/kurtosis-web.png)

## Kurtosis 隔离区 (Kurtosis Enclave)

Kurtosis 中的 **隔离区 (enclave)** 是部署服务的隔离环境。它允许开发人员创建多个开发网而互不干扰。关键命令包括：

```sh
# 列出已存在的隔离区
kurtosis enclave ls

# 检查某个隔离区
kurtosis enclave inspect <enclave-name>

# 示例
kurtosis enclave inspect my-testnet

# 删除某个隔离区
kurtosis enclave rm <enclave-name> -f
```

在网页界面中通过点击隔离区的名称来探索它：

![Kurtosis enclave in web interface](./kurtosis-web-enclave.png)

您可以同时运行多个隔离区，但要注意您机器的资源以避免性能问题。此外，在同时管理多个隔离区时，为每个隔离区分配一个自定义名称会非常有用。

```sh
# 为隔离区分配自定义名称
kurtosis --enclave my-testnet run github.com/ethpandaops/ethereum-package
```

## 清理资源 (Cleaning Up Resources)

在测试之后，您可能希望以新参数重新启动开发网，或者清理资源以释放磁盘空间。使用以下命令删除已停止的隔离区以及已停止的引擎容器：

```sh
kurtosis clean -a
```

> [!CAUTION]
> 所有的隔离区都将被删除。

## 自定义开发网 (Customizing Devnets)

Kurtosis 中的开发网是通过保存在本地或网络中的 YAML 文件进行配置的。这些文件允许自定义参数，例如参与者 (participants)、网络参数、附加服务。如果文件中未指定某个参数，则使用默认值。所有可能配置参数的详尽列表可以在 [此处](https://github.com/ethpandaops/ethereum-package?tab=readme-ov-file#configuration) 找到。常见的做法是在本地构建客户端 Docker 镜像，并通过 `cl_image` 或 `el_image` 参数将它们添加到配置文件中以进行测试。

示例文件：

```yaml
participants:
 - el_type: geth   
   el_image: geth:latest   
   cl_type: teku
   cl_image: consensys/teku:develop
additional_services:
  - dora
  - prometheus_grafana
network_params:
  seconds_per_slot: 4
global_log_level: debug
```

要运行带这些参数的 Kurtosis，使用 `--args-file` 标志。对于远程文件，提供目标文件的原始 URL：

```sh
# 本地文件
kurtosis run github.com/ethpandaops/ethereum-package --args-file ./custom.yaml

# 远程文件
kurtosis run github.com/ethpandaops/ethereum-package --args-file https://raw.githubusercontent.com/ethpandaops/ethereum-package/main/.github/tests/mix.yaml
```

您可以在 [此处](https://github.com/ethpandaops/ethereum-package?tab=readme-ov-file#configuration) 找到所有参数的详细描述。

## 自定义以太坊软件包 (Customizing Ethereum Package)

您可以克隆以太坊软件包的代码库并在其之上进行开发。要运行您自己的修改，使用以下命令：

```sh
kurtosis run .

# 示例
kurtosis run . --args-file ./custom.yaml
```

## 工具链 (Tooling)

Kurtosis 提供了几种内置工具来与以太坊开发网交互。这里有一些例子：

- [**dora**](https://github.com/ethpandaops/dora)：轻量级的信标链 (Beacon Chain) 浏览器。
- [**xatu**](https://github.com/ethpandaops/xatu)：用于数据管道的集中式以太坊网络监控工具。
- [**assertoor**](https://github.com/ethpandaops/assertoor)：用于编写断言 (assertions) 以验证网络行为。

示例 YAML 文件：

```yaml
- el_type: nethermind
  cl_type: prysm
additional_services:
  - dora
  - assertoor
```

完整服务列表可以在 [此处](https://ethpandaops.io/posts/kurtosis-deep-dive/#tooling) 找到。

其中一些工具有网页界面。要打开界面，使用用户服务列表中提供的链接：

![Kurtosis tools](./kurtosis-tools.png)

Dora 是信标链最常用的工具之一。以下是其界面的样子：

![Dora](./kurtosis-dora.png)

## 日志处理 (Working with Logs)

阅读日志是调试 (debugging) 和监控开发网的重要部分。可以使用命令行界面 (CLI) 访问日志：

```sh
kurtosis service logs <enclave-id> <service-name> -f

# 示例
kurtosis service logs my-testnet cl-1-teku-geth -f
```

也可以将日志写入文件以进行进一步分析：

```sh
kurtosis service logs <enclave-name> <instance> > <file-name>

# 示例
kurtosis service logs my-testnet cl-1-teku-geth > kurtosis.log
```

或者，可以使用 Docker 日志来检查 Kurtosis 容器：

```sh
docker logs <container-id>
```

## 参考文献 (References)

- [Ethereum Package GitHub Repository](https://github.com/kurtosis-tech/ethereum-package)
- [Kurtosis GitHub Repository](https://github.com/kurtosis-tech/kurtosis)
- [Kurtosis Official Documentation](https://docs.kurtosis.com)
- [Kurtosis Deep Dive (EthPandaOps blog post)](https://ethpandaops.io/posts/kurtosis-deep-dive/)
- [Kurtosis Workshop by EthPandaOps Team (Youtube)](https://www.youtube.com/watch?v=mywpmp2sPt0)
- [3-minute introduction of running local devnet using ethereum-package by James (Prysm)](https://www.loom.com/share/f7d32d0af14f4f24b63bf3cebfdc796a)
