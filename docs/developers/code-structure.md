# Java-tron 模块结构

`Java-tron` 是由 Java 编写的 TRON 网络核心客户端，实现了 TRON 白皮书中定义的全部核心功能，包括共识机制、加密算法、数据库管理、TRON 虚拟机（TVM）以及网络通信等。开发者可以通过启动 Java-tron 快速运行一个 TRON 网络节点。

本文将系统解析 Java-tron 的代码结构，深入介绍各功能模块的职责与组织方式，为开发者后续进行源码分析和功能扩展提供参考。

## 核心模块
`Java-tron` 采用模块化的设计理念，确保了代码的清晰性、可维护性和可扩展性。当前，`Java-tron` 主要包含以下七个核心模块：

* [protocol](#1)
* [common](#2)
* [chainbase](#3)
* [consensus](#4)
* [actuator](#5)
* [crypto](#6)
* [framework](#7)

接下来，我们将逐一介绍这些模块的功能及其内部结构。

<a id="1"></a>
### 1. `protocol` 模块

在区块链这样的分布式网络环境中，简洁高效的数据交互协议至关重要。`protocol` 模块定义了 `java-tron` 客户端中的关键通信协议：

*   节点间的通信协议
*   节点内部各模块间的通信协议
*   对外提供的服务协议

上述协议统一采用了 [`Google Protobuf`](https://developers.google.com/protocol-buffers)（Protocol Buffers）作为数据交互格式。相比 JSON 和 XML，Protobuf 具有体积更小、解析更快、类型更安全等优势，同时具备良好的结构定义能力与可扩展性。借助 Protobuf 编译器，可基于定义的协议文件自动生成多种语言（如 Java、C++、Python 等）的序列化与反序列化代码，便于实现跨语言集成。Protobuf 是 java-tron 实现跨平台、跨语言通信的核心基础，支撑了节点间、模块间及对外服务接口的高效数据传输。



`protocol` 模块的源代码位于 [java-tron 仓库的 protocol 目录](https://github.com/tronprotocol/java-tron/tree/develop/protocol)，其核心目录结构如下：

```
|-- protos
    |-- api
    |   |-- api.proto
    |   |-- zksnark.proto
    |-- core
        |-- Discover.proto
        |-- Tron.proto
        |-- TronInventoryItems.proto
        |-- contract
```

*   `protos/api/`：定义了 `java-tron` 节点对外提供的 gRPC 接口及其相关数据结构。
*   `protos/core/`：定义了节点间以及节点内部各模块间通信所需的数据结构。
    *   `Discover.proto`：定义了节点发现机制相关的数据结构。
    *   `TronInventoryItems.proto`：定义了节点间区块和交易传输相关的数据结构。
    *   `contract/`：定义了智能合约相关的数据结构定义。
    *   `Tron.proto`：定义了其他重要的核心数据结构，其中包括账户、区块、交易、资源（如**带宽**和**能量**）、超级代表、投票和 Proposal 等相关数据结构。



<a id="2"></a> 
### 2. `common` 模块

`common` 模块封装了 `java-tron` 中常用的公共组件和工具类，旨在提升代码复用性，便于其他模块统一调用。该模块包含异常处理机制、指标监控工具等通用功能。

源代码位于 [java-tron 仓库的 common 目录](https://github.com/tronprotocol/java-tron/tree/develop/common)，其核心目录结构如下所示：


```
|-- /common/src/main/java/org/tron
    |-- common
    |   |-- args
    |   |-- cache
    |   |-- config
    |   |-- cron
    |   |-- entity
    |   |-- es
    |   |-- exit
    |   |-- log
    |   |-- logsfilter
    |   |-- math
    |   |-- parameter
    |   |-- prometheus
    |   |-- runtime/vm
    |   |-- setting
    |   |-- utils
    |-- core
        |-- config
        |-- db
        |-- db2
        |-- exception
        |-- vm/config
        |-- Constant.java
```

*   `common/prometheus`：集成了 Prometheus 指标监控工具，用于采集并暴露 `java-tron` 节点的运行时指标，便于系统监控与性能分析。
*   `common/utils`：提供了一系列基础数据类型的封装类和实用工具方法（如数据转换、格式化等），用于提升开发效率，提高代码可读性。
*   `core/config`：包含与节点配置相关的类，负责管理和加载 `java-tron` 节点的各类运行参数。
*   `core/exception`：定义了 TRON 网络中的异常处理机制，支持统一的错误捕获与反馈，提升系统健壮性与可维护性。
*   `common/args`：节点基础参数定义类。
*   `common/cache`：定义数据库缓存相关类。
*   `common/config`：数据库备份配置类。
*   `common/es`：服务执行管理器。
*   `common/cron`：Cron 表达式处理类。
*   `common/entity`：定义节点、对等节点信息。
*   `common/exit`：服务退出管理类。
*   `common/log`：日志服务。
*   `common/logsfilter`：事件插件配置类及各种事件定义类。
*   `common/math`：数学计算封装类。
*   `common/parameter`：节点启动参数配置类。
*   `common/runtime/vm`：tvm字段类型处理类。
*   `common/setting`：RocksDb配置类。
*   `core/db` 和 `core/db2`：数据库相关基础类。

<a id="3"></a> 
### 3. `chainbase` 模块

`chainbase` 模块是 java-tron 在数据库层的抽象封装。考虑到 Proof of Work (PoW)、Proof-of-Stake (PoS)、Delegated Proof-of-Stake (DPoS) 等基于概率的共识算法可能出现的链分叉（切链）情况，chainbase 提供了一个支持状态回退的数据库接口标准。该接口要求底层数据库实现状态回滚机制、Checkpoint 容灾机制等能力，以保障系统在发生链重组时仍能保持数据一致性与可靠性。

此外，chainbase 模块具备良好的接口抽象设计，任何符合该接口规范的数据库都可作为区块链系统的底层存储，从而为开发者提供更大的灵活性。当前，java-tron 默认提供了 LevelDB 和 RocksDB 两种实现。

chainbase 模块源代码位于 [java-tron 仓库的 chainbase 目录](https://github.com/tronprotocol/java-tron/tree/develop/chainbase)，其核心目录结构如下：


```
|-- chainbase.src.main.java.org.tron
    |-- common
    |   |-- bloom
    |   |-- error
    |   |-- overlay/message
    |   |-- runtime
    |   |-- storage
    |   |   |-- leveldb
    |   |   |-- rocksdb
    |   |-- utils
    |   |-- zksnark
    |-- core
        |-- actuator
        |-- capsule
        |-- config/args
        |-- db
        |   |-- RevokingDatabase.java
        |   |-- TronStoreWithRevoking.java
        |   |-- ......
        |-- db2
        |   |-- common
        |   |-- core
        |       |-- SnapshotManager.java
        |       |-- ......
        |-- net/message
        |-- service
        |-- store
        |-- ChainBaseManager.java
```

*   `common/`：封装了一些通用组件，如异常处理类和工具类。
    *   `storage/leveldb/`：基于 LevelDB 实现的数据库存储逻辑。
    *   `storage/rocksdb/`：基于 RocksDB 实现的数据库存储逻辑。
*   `core/`：`chainbase` 模块的核心代码。
    *   `capsule/`：封装各类核心数据结构（如 `AccountCapsule`、`BlockCapsule`），用于提供账户、区块等对象的读写操作接口。
    *   `store/`：实现了各种业务数据库 （如 `AccountStore` 和 `ProposalStore`）。
        *   `AccountStore` ：账户数据库，其数据库名称为 `account`，用于存储所有账户信息；
        *   `ProposalStore` ：Proposal 数据库，其数据库名称为 `proposal`，用于存储所有 Proposal 信息。
    *   `db/` 和 `db2/`：实现了可回退数据库。
        *   `db/` ：包含早期的 `AbstractRevokingStore`。
        *   `db2/` ：包含更稳定的和可扩展的 `SnapshotManager`。是当前 java-tron 中的主要回退机制实现。其中的关键类和接口包括：
            *   `RevokingDatabase.java`：数据库容器接口，负责管理所有可回退数据库，`SnapshotManager` 是该接口的一个具体实现。
            *   `TronStoreWithRevoking.java`：支持回退机制数据库的通用基类，`BlockStore`、`TransactionStore` 等具体业务数据库均基于该类实现。



<a id="4"></a> 
### 4. `consensus` 模块

共识机制是区块链系统中至关重要的核心模块，决定了网络中节点如何就交易和区块的有效性达成一致。常见的共识算法包括工作量证明（PoW）、权益证明（PoS）、委托权益证明（DPoS）以及实用拜占庭容错（PBFT）等。在联盟链或受限信任网络中，也常采用 Paxos、Raft 等一致性算法。

共识机制的选择应与具体业务场景相匹配，比如对共识效率敏感的实时游戏类就不适合采用 PoW，而对实时性要求极高的交易所来说 PBFT 可能是首选。所以支持可替换的共识是非常有必要的，同时也是实现特定应用区块链的重要一环，consensus 模块最终目标是能够让应用开发者能够像配置参数那样简单的切换共识机制。

`consensus` 模块源代码位于 [java-tron 仓库的 consensus 目录](https://github.com/tronprotocol/java-tron/tree/develop/consensus)，其核心目录结构如下：

```
|-- consensus/src/main/java/org/tron/consensus
    |-- Consensus.java
    |-- ConsensusDelegate.java
    |-- base
    |   |-- ConsensusInterface.java
    |   |-- ......
    |-- dpos
    |-- pbft
```

`consensus` 模块将共识过程抽象为一组核心接口，统一定义在 `ConsensusInterface` 中，主要包括以下方法：

* `start`：启动共识服务，支持自定义启动参数；
* `stop`：停止共识服务；
* `receiveBlock`：定义接收区块时的共识处理逻辑；
* `validBlock`：定义区块验证的共识逻辑；
* `applyBlock`：定义区块应用（写入状态）的处理逻辑。

目前，`java-tron` 基于该接口提供了两种共识机制的实现：DPoS（位于 dpos/ 目录）和 PBFT（位于 pbft/ 目录）。

开发者也可以根据具体业务需求，自行实现 `ConsensusInterface` 接口，以集成其他共识算法，从而实现更灵活的链设计与部署。


<a id="5"></a> 
### 5. `actuator` 模块

以太坊开创性地引入了虚拟机和智能合约开发范式，极大推动了区块链应用的发展。然而，在某些复杂场景下，智能合约在灵活性和性能方面仍存在局限。为满足更高性能和定制化的需求，java-tron 提供了构建应用链（Application Chain）的能力，并为此设计了独立的 `actuator` 模块。

`actuator` 模块是交易的执行器，支持将应用逻辑直接植入链中，而无需在虚拟机中运行。开发者可将应用视为由不同类型的交易组成，每种交易类型对应一个专属的 actuator，负责该类交易的具体执行逻辑，从而实现高度定制化的链上处理能力。

`actuator` 模块源代码位于 [java-tron 仓库的 actuator 目录](https://github.com/tronprotocol/java-tron/tree/develop/actuator)，其核心目录结构如下：


```
|-- actuator/src/main/java/org/tron/core
    |-- actuator
    |   |-- AbstractActuator.java
    |   |-- ActuatorCreator.java
    |   |-- ActuatorFactory.java
    |   |-- TransferActuator.java
    |   |-- VMActuator.java
    |   |-- ......
    |-- utils
    |-- vm
```
**`actuator` 模块核心结构说明**

*   `actuator/`：定义了 TRON 网络中各种类型交易的执行器，每类交易对应一个特定的执行器。例如：
    *   `TransferActuator` ：处理 TRX 转账交易；
    *   `FreezeBalanceV2Actuator` ：处理质押 TRX 以获取资源（带宽或能量）交易。
*   `utils/`：封装了交易执行过程中的辅助工具类。
*   `vm/`：包含虚拟机相关逻辑。

**Actuator 接口定义**

`actuator` 模块通过定义统一的 Actuator 接口，规范了交易执行逻辑。该接口包含以下核心方法：

* `execute()`：执行交易的具体操作，如状态变更、业务逻辑处理等；
* `validate()`：校验交易合法性，防止无效或恶意交易；
* `getOwnerAddress()`：获取交易发起方的地址；
* `calcFee()`：计算交易所需手续费。


开发者也可以根据自身业务需求，通过实现 `Actuator` 接口来定义和处理自定义的交易类型。



<a id="6"></a> 
### 6. `crypto` 模块

`crypto` 是 java-tron 中一个相对独立但至关重要的模块，承担了整个系统的加密与数据安全工作。目前支持的加密算法包括 `SM2` 和 `ECKey` 。

`crypto` 模块源代码位于 [java-tron 仓库的 crypto 目录](https://github.com/tronprotocol/java-tron/tree/develop/crypto)，其核心目录结构如下：

```
|-- crypto/src/main/java/org/tron/common/crypto
    |-- Blake2bfMessageDigest.java
    |-- ECKey.java
    |-- Hash.java
    |-- SignInterface.java
    |-- SignUtils.java
    |-- SignatureInterface.java
    |-- cryptohash
    |-- jce
    |-- sm2
    |-- zksnark
```

*   `sm2` 和 `jce`：分别实现了 SM2 和 ECKey 加密与签名算法，广泛用于交易签名、身份验证等核心安全场景。
*   `zksnark`：引入零知识证明（`Zero-Knowledge Proof`）算法，为更高级别的隐私保护能力奠定基础。

该模块为整个 TRON 网络提供了安全基础，确保交易数据的完整性、不可抵赖性与隐私性，是支撑链上信任的重要支柱。


<a id="7"></a> 
### 7. `framework` 模块

`framework` 是 java-tron 的核心模块，也是整个节点的启动入口。它负责各个子模块的初始化、业务逻辑的分发，以及对外服务的统一接入，涵盖了节点网络通信、区块广播、同步与管理等核心功能。

`framework` 模块源代码位于 [java-tron 仓库的 framework 目录](https://github.com/tronprotocol/java-tron/tree/develop/framework)，其核心目录结构如下：

```
|-- framework/src/main/java/org/tron
    |-- common
    |   |-- application
    |   |-- backup
    |   |-- logsfilter
    |   |-- net
    |   |-- overlay
    |   |   |-- client
    |   |   |-- discover
    |   |   |-- message
    |   |   |-- server
    |   |-- runtime
    |   |-- zksnark
    |-- core
    |   |-- Wallet.java
    |   |-- capsule
    |   |-- config
    |   |-- consensus
    |   |-- db
    |   |-- metrics
    |   |-- net
    |   |-- services
    |   |-- trie
    |   |-- zen
    |-- keystore
    |-- program
    |   |-- FullNode.java
    |-- tool
```

*   `program/FullNode.java`：java-tron 的程序入口，负责初始化各项系统组件及对外接口服务（HTTP、gRPC、JSON-RPC 等），以支持外部系统接入 TRON 网络。 
*   `core/services`：对外服务的统一出口.
    * `http/`: 定义所有 HTTP 接口的控制逻辑
    * `json-rpc/` : 实现 JSON-RPC 接口，兼容以太坊的 RPC 标准，支持轻量化访问节点功能
*   `common/overlay/discover`：实现基于 Kademlia 的节点发现协议，用于发现并连接其他 TRON 网络节点。
*   `common/overlay/server`：管理已连接节点、处理区块同步、交易传播等逻辑，保障节点间的区块一致性和消息传递效率。
*   `core/net`：处理网络消息。其子目录 `/service` 包含了交易和区块的广播、区块抓取和同步逻辑。
*   `core/db/Manager.java`：数据处理的核心调度类，负责交易与区块的验证、应用、持久化等操作，是链上数据处理的关键枢纽。

