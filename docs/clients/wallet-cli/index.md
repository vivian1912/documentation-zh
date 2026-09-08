# wallet-cli

wallet-cli 是 [TRON 网络](https://tron.network)的命令行钱包，提供 Java 和 TypeScript 两种实现：Java 版面向交互式使用，
TypeScript 版面向脚本和自动化集成。

本仓库包含**两套独立实现**，目的相同但面向不同的使用者：

- **[Java](java/index.md)**——最早的完整参考实现，采用交互式命令行（REPL）。
- **[TypeScript](typescript/index.md)**——面向自动化的重写版本，采用标准子命令，并提供稳定的
  JSON 输出，适合脚本、CI 和 AI 智能体调用。

两者管理同一类钱包；在 TRON 网络上，同一助记词通过两种实现派生出的地址完全相同。它们支持的 TRON 功能基本一致，
主要区别在于安装方式和操作方式——此外 TypeScript 版还支持 **EVM 网络**（以太坊、BNB Smart Chain 及其测试网），
Java 版则不支持。选定其一后，请阅读对应文档了解详细用法；本页概述两者的特点，帮助你选择。

## 概览对比

| | [**Java**](java/index.md)——原始实现 | [**TypeScript**](typescript/index.md)——面向自动化的重写版 |
| ---------------------- | ---------------------------------- | -------------------------------------------------- |
| **它是什么** | 成熟、功能完整的参考 CLI。 | 较新的重写版本，专注于程序化集成。 |
| **运行时** | JVM——使用 Gradle 构建，以 `.jar` 运行。使用 [Trident](https://github.com/tronprotocol/trident) SDK。 | [Node.js](https://nodejs.org) **20+**。 |
| **安装** | `git clone` + `./gradlew build`（见[环境准备](java/index.md#setup)） | `npm install -g @tron-walletcli/wallet-cli` |
| **操作方式** | **交互式命令行**——启动后在 `>` 提示符中输入命令。 | **非交互式子命令**——在 shell 中执行 `wallet-cli <command>`。只有输入敏感信息时才会出现交互式提示。 |
| **命令风格** | PascalCase 动词：`RegisterWallet`、`SendCoin`、`GetBalance`。金额以 **SUN** 计（1 TRX = 1,000,000 SUN）。 | 名词—动词子命令：`create`、`tx send`、`account balance`，配合 `--flags`。 |
| **面向脚本的输出** | 供人阅读的文本。 | 通过 `-o json` 输出稳定 JSON（[`wallet-cli.result.v1`](typescript/machine-interface.md)），配合固定退出码（`0`/`1`/`2`）。 |
| **配置 / 网络** | `config.conf`（网络类型 + FullNode），或运行时使用 `SwitchNetwork`。主网 · Nile · Shasta · 自定义。 | `--network` 参数 / `config` 命令。使用 CAIP-2 网络 id，并带有简短别名：`tron:728126428`（`tron`）· `tron:3448148188`（`nile`）· `tron:2494104990`（`shasta`）· `eip155:1`（`ethereum`）· `eip155:11155111`（`sepolia`）· `eip155:56`（`bsc`）· `eip155:97`（`bsc-testnet`）。 |
| **签名** | 软件 keystore · Ledger。 | 加密的本地 keystore · Ledger。敏感信息不会从命令行参数或环境变量读取。 |
| **功能范围** | 钱包与转账、质押、投票与奖励、治理、合约、TRC10，以及链上交易所。 | HD 钱包、TRX/TRC20/TRC10 转账、质押与代理、投票与奖励、治理提案与超级代表运营、合约调用/部署/治理、TRC10 发行、链上 Bancor 交易所、多重签名、GasFree 转账、消息签名，以及链上查询。 |
| **适用场景** | 需要交互式操作和完整 TRON 功能的用户。 | 脚本、CI 流水线和 AI 智能体集成。 |
| **完整文档** | [Java CLI](java/index.md) | [TypeScript CLI](typescript/index.md) |

## Java 快速体验

Java 版仅支持交互式操作。完成构建并启动命令行后，在提示符中输入命令：

```console
$ git clone https://github.com/tronprotocol/wallet-cli.git
$ cd wallet-cli && ./gradlew build && cd build/libs
$ java -jar wallet-cli.jar        # 打开交互式提示符
> RegisterWallet 123456           # 创建 keystore（密码 123456）
> Login                           # 解锁
> GetAddress                      # 你的 TRON 地址
> GetBalance                      # TRX 余额
```

有关环境准备（config.conf、节点连接）、A–Z 命令列表，以及 GasFree、多重签名等功能的详细说明，
请参见 **[Java CLI](java/index.md)**；也可以直接跳转到[环境准备](java/index.md#setup)、
[快速上手](java/index.md#quickstart)、[命令](java/index.md#commands)，或
[GasFree](java/index.md#contracts-gasfree--chain-data)。

## TypeScript 快速体验

通过 npm 安装后，可以直接在 shell 中执行子命令：

```console
$ npm install -g @tron-walletcli/wallet-cli
$ wallet-cli create --label main               # 提示设置主密码（master password）
$ wallet-cli account balance --network tron:3448148188
$ wallet-cli account balance -o json           # 输出符合 wallet-cli.result.v1 规范的 JSON
```

每条命令都有独立的参考页，同时提供 JSON 接口规范、退出码和 AI 智能体调用说明。请从
**[TypeScript CLI](typescript/index.md)** 开始，然后：

- [快速上手](typescript/guide/getting-started.md)——创建钱包并发送第一笔交易
- [命令参考](typescript/commands/index.md)——全部命令，A–Z
- [机器接口](typescript/machine-interface.md)——JSON 响应结构、退出码、脚本安全

## 我该用哪一个？

- 编写脚本、运行 CI 或构建 AI 智能体时，建议使用 [TypeScript 版](typescript/index.md)，它提供稳定的
  JSON 输出和确定的退出码。
- 希望在 `>` 提示符中保持长时间会话，并在整个会话中只解锁一次钱包时，建议使用
  [Java 版](java/index.md)。
- 只需在本机转账 TRX/token 或进行质押时，两者均可；TypeScript CLI 通过 npm 安装，无需本地构建。
