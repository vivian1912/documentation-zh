# TRON P2P 网络架构与核心机制

TRON 网络的核心是其高效、去中心化的 **P2P（对等）网络** 架构。作为 TRON 区块链的最底层模块，P2P 网络的设计与实现直接决定了整个系统的稳定性和性能。本文档将深入解析 TRON P2P 网络的四大核心功能：节点发现、节点连接、区块同步以及区块和交易广播。


## P2P 网络概述

P2P 网络是一种分布式网络，其参与者在不依赖中心化服务器的情况下，直接共享计算、存储和网络连接等硬件资源。在 P2P 网络中，每个节点既是信息的提供者，又是信息的消费者，这种模式极大地提高了资源的利用率和网络的鲁棒性。

### 区块链与 P2P

P2P 是区块链架构中的 **网络层**。它为节点间的去中心化信息传播、验证和通信提供了基础。每个节点通过维护一份共同的区块链数据副本来保持同步，从而确保了整个网络的共识。

这种去中心化的 P2P 架构为 TRON 网络带来了显著优势：

  * **防止单点攻击**: 网络中没有中心化的故障点。
  * **高容错性**: 即使部分节点宕机，网络仍能正常运行。
  * **良好的可扩展性**: 网络的扩展能力不受中心化服务器的限制。

TRON架构图如下：

![image](https://raw.githubusercontent.com/tronprotocol/documentation-zh/master/images/network_architecture.png)

TRON 网络的 P2P 模块可分为以下四个主要功能：

1.  [节点发现](#node-discovery)
2.  [节点连接](#node-connection)
3.  [区块同步](#block-sync)
4.  [区块和交易广播](#broadcasting)


<a id="node-discovery"></a>
## 节点发现

节点发现是新节点加入 TRON 网络的第一步。TRON 采用一种基于 **Kademlia 算法** 的结构化 P2P 网络。这种算法能够让节点在没有中心化目录的情况下，快速高效地找到其他节点。

### Kademlia 算法原理

Kademlia 是一种分布式哈希表（**DHT**）实现，它通过计算节点 ID 的异或距离来确定节点间的“远近”，并以此组织路由表。

TRON 对 Kademlia 算法的实现要点包括：

  * **节点 ID**: 每个节点启动时会随机生成一个 512 位的 ID。
  * **节点距离**: 两个节点间的距离是通过它们 ID 的异或运算计算得出的。公式为：`距离 = 256 - 节点ID异或结果的前导零个数`。如果计算结果为负数，距离等于0。
  * **K 桶**: 节点的路由表，用于存储远端节点信息。根据与本节点的距离远近，将远端节点划分为 256 个“桶”，每个桶最多容纳 16 个节点。

### 节点发现协议消息

TRON 节点发现协议基于 **UDP**，使用以下四种消息类型进行通信：

  * `DISCOVER_PING`: 用于探测目标节点是否在线。
  * `DISCOVER_PONG`: 对 `DISCOVER_PING` 的响应。
  * `DISCOVER_FIND_NODE`: 用于查找距离特定目标 ID 最近的其他节点。
  * `DISCOVER_NEIGHBORS`: 对 `DISCOVER_FIND_NODE` 的响应，最多返回 16 个节点。

### 节点发现流程

#### 1\. 初始化 K 桶

节点启动时，会读取配置文件中配置的种子节点（`seed nodes`）和数据库中已知的对等节点，向它们发送 `DISCOVER_PING` 消息。如果收到 `DISCOVER_PONG` 响应，节点会将其添加到对应的 K 桶中。如果 K 桶已满，则会向 K 桶中最旧的节点发起挑战，成功后用新节点替换旧节点。

#### 2\. 定期发现更多节点

TRON 节点发现服务会周期性地执行两个定时任务来持续更新 K 桶：

  * **`DiscoverTask`**: 每 **30 秒** 执行一次，用于发现更多距离本节点较近的节点。执行流程如下图所示:
![image](https://raw.githubusercontent.com/tronprotocol/documentation-zh/master/images/network_discovertask.png)
  * **`RefreshTask`**: 每 **7.2 秒** 执行一次，通过查找距离一个随机 ID 最近的节点来扩充本地 K 桶。执行流程如下图所示:
![image](https://raw.githubusercontent.com/tronprotocol/documentation-zh/master/images/network_refreshtask.png)

这两个任务在每次运行时，会执行8轮节点发现，每轮会向 K 桶中距离目标 ID 最近的 **3 个** 节点发送 `DISCOVER_FIND_NODE` 消息，以请求更多邻居节点。

#### 3\. 接收并更新 K 桶

当节点收到远端节点回复的 `DISCOVER_NEIGHBORS` 消息后，它会依次向收到回复的邻居节点发送 `DISCOVER_PING`。如果成功收到 `DISCOVER_PONG` 响应，节点会尝试将这个新节点加入到相应的 K 桶中。如果 K 桶已满，会向桶中某个节点发起挑战（即发送 `DISCOVER_PING`），如果挑战成功（没有收到 `PONG` 响应），则将旧节点移除，加入新节点。

![image](https://raw.githubusercontent.com/tronprotocol/documentation-zh/master/images/network_updatek.png)

通过这些周期性的任务，每个节点都能持续构建和维护一个动态的、健康的路由表。


<a id="node-connection"></a>
## 节点连接

在发现潜在节点后，下一步是建立 **TCP 连接**。TRON 网络对节点进行分类管理，以实现高效稳定的连接。

### 节点分类

TRON 节点将对等节点（`peers`）分为以下几类：

  * **Active Nodes**: 在配置文件中指定，节点启动后会主动尝试与其建立连接。如果连接建立失败，则在每个TCP连接定时任务中都会重新尝试与其建立连接。
  * **Passive Nodes**: 在配置文件中指定，被动接受这些节点的连接请求。
  * **Trust Nodes**: 包括 `Active` 和 `Passive` 节点。TRON 会优先信任这些节点，跳过部分连接检查。
  * **Bad Nodes**: 被列入黑名单的节点。当收到异常协议报文时，该节点会被标记为 `badNodes`，有效期为 **1 小时**。
  * **Recently Disconnected Nodes**: 30 秒内断开连接的节点，用于防止短时间内重复连接。

### 建立 TCP 连接

TRON 节点会定期运行一个名为 `poolLoopExecutor` 的定时任务，用于建立 TCP 连接。其工作流程如下：

![image](https://raw.githubusercontent.com/tronprotocol/documentation-zh/master/images/network_connect.png)

1.  **确定连接列表**: 首先检查 `active nodes` 中尚未成功连接的节点，并将其加入列表。
2.  **筛选邻居节点**: 计算还需要连接的数量，并从已发现的邻居列表中，根据 **节点过滤策略** 和 **节点打分策略** 筛选出最佳的节点。
3.  **建立连接**: 向最终列表中的节点发起 TCP 连接。

#### 1\. 节点过滤策略

在尝试连接之前，会过滤掉不符合条件的节点，包括：

  * 自身节点。
  * 位于 `recentlyDisconnectedNodes` 或 `badNodes` 列表中的节点。
  * 已建立连接的节点。
  * IP 连接数达到上限（`maxConnectionsWithSameIp`）的节点。

> **注意**: `trust nodes` 会忽略部分过滤规则，始终被优先考虑建立连接。


![image](https://raw.githubusercontent.com/tronprotocol/documentation-zh/master/images/network_filterrule.png)

#### 2\. 节点打分策略

节点会根据多个维度对潜在的连接对象进行打分，分数越高，优先级越高。

| 维度 | 评分规则 |
| :--- | :--- |
| **丢包率** | 丢包率越低，分数越高。最高 **100** 分。 |
| **网络延迟** | 延迟越低，分数越高。最高 **20** 分。 |
| **TCP 流量** | 流量越大，分数越高。最高 **20** 分。 |
| **断开连接次数** | 次数越多，分数越低（负分）。一次断开连接扣 **10** 分。 |
| **握手成功次数** | 曾成功握手过的节点得 **20** 分，否则为 **0**。 |
| **处罚状态** | 若节点处于处罚状态（如断开连接时间不足 60 秒、在 `badNodes` 列表中、链信息不一致），则直接得 **0** 分。 |

### 握手与保活

当 TCP 连接建立成功后，双方会进行握手，交换 `P2P_HELLO` 消息，以验证彼此的区块链信息（如 `p2p version`、`genesis block` 等）。若所有检查通过，握手成功，连接将进入区块同步或数据广播阶段；否则，连接将被断开。

为了保持信道活跃，节点会定期发送 `P2P_PING` 消息。如果在超时时间内没有收到 `P2P_PONG` 响应，连接将被断开。


<a id="block-sync"></a>
## 区块同步

新节点在成功连接并握手后，如果发现远端节点的区块链比本地的更长，根据最长链原则，会触发区块同步的处理流程`syncService.startSync`。同步过程中的报文交互如下图：

![image](https://raw.githubusercontent.com/tronprotocol/documentation-zh/master/images/network_syncflow.png)

1.  **请求链摘要**: 节点 A 向节点 B 发送 `SYNC_BLOCK_CHAIN` 消息，包含本地链的摘要信息（如最高固化块、最高非固化块等）。
2.  **返回缺失清单**: 节点 B 收到摘要后，与自己的链进行比对，计算出节点 A 缺失的区块 ID 列表，并通过 `BLOCK_CHAIN_INVENTORY` 消息返回，每次最多携带 **2000 个** 区块 ID。
3.  **请求缺失区块**: 节点 A 收到清单后，通过 `FETCH_INV_DATA` 消息异步请求缺失的区块，每次最多请求 **100 个**。如果还有剩余区块，将触发新一轮同步。
4.  **发送区块**: 节点 B 收到 `FETCH_INV_DATA` 请求后，通过 `BLOCK` 消息将区块数据发送给节点 A。
5.  **处理**: 节点 A 收到 `BLOCK` 消息后，异步处理该区块。



### 链摘要与缺失区块清单

节点在区块同步时，会通过**链摘要**和**缺失区块清单**来确定需要拉取的区块。

* **链摘要**：由一组有序的区块 blockID 构成，通常包括最高固化块、最高非固化块，以及二分方式选取的中间区块。
* **缺失区块清单**：邻居节点根据链摘要与自身链进行对比，识别出对方缺失的区块，返回连续的区块 blockID 以及剩余块数。

下面根据几个不同的区块同步场景示例，说明链摘要和节点缺少的区块清单的生成。

#### 普通场景

本地头块高度为 **1018**，固化块高度为 **1000**。
由于节点 A 与节点 B 初次建立连接，共同块高度为 **0**。
节点 A 通过二分法生成的链摘要为：`1000、1010、1015、1017、1018`。

节点 B 收到链摘要后，与自身链对比，发现节点 A 缺少区块：`1018、1019、1020、1021`。
随后，节点 A 根据缺失区块清单，请求同步 `1019、1020、1021`。

![image](https://raw.githubusercontent.com/tronprotocol/documentation-zh/master/images/network_sync1.png)

#### 切链场景

**场景一**
本地主链头块高度为 **1018**，固化块高度为 **1000**，共同块高度为 **0**。
节点 A 的链摘要为：`1000、1010、1015、1017、1018`。

节点 B 收到后发现链分叉，对比后确认共同块为 **1015**。
因此节点 A 缺少的区块清单为：`1015、1016'、1017'、1018'、1019'`。
节点 A 随后请求同步 `1018'、1019'`。

![image](https://raw.githubusercontent.com/tronprotocol/documentation-zh/master/images/network_sync2.png)

**场景二**
本地主链头块高度为 **1018**，固化块高度为 **1000**，共同块高度为 **1017'**（位于分叉链上）。
节点 A 的链摘要为：`1000、1009、1014、1016'、1017'`。

节点 B 对比后生成缺失区块清单：`1017'、1018'、1019'`。
节点 A 随后请求同步 `1018'、1019'`。

![image](https://raw.githubusercontent.com/tronprotocol/documentation-zh/master/images/network_sync3.png)


<a id="broadcasting"></a>
## 区块与交易广播

当超级代表节点生成新区块，或节点收到新的交易时，会发起广播流程。广播和转发的流程基本一致。报文交互如下图所示：

![image](https://raw.githubusercontent.com/tronprotocol/documentation-zh/master/images/network_broadcastflow.png)


其中涉及到的消息类型包括：

  * `INVENTORY`: 包含区块或交易 ID 的广播清单。
  * `FETCH_INV_DATA`: 请求获取清单中的数据。
  * `BLOCK`: 区块数据。
  * `TRXS`: 交易数据。

具体流程如下：

1.  **发送清单**: 节点 A 将待广播的区块或交易 ID 打包成 `INVENTORY` 消息，发送给邻居节点 B。
2.  **放入队列**: 节点 B 收到 `INVENTORY` 后，需要检查对方节点的状态，如果可以接收该消息，则将清单中的区块/交易 ID 放入待获取队列 `invToFetch`。如果是区块清单，会立即发送 `FETCH_INV_DATA` 消息请求数据。
3.  **发送数据**: 节点 A 收到 `FETCH_INV_DATA` 后，会检查是否发送过清单消息给对方，如果发送过，则根据清单数据，将具体的区块（`BLOCK`）或交易（`TRXS`）数据发送给节点 B。
4.  **处理与转发、**: 节点 B 收到数据后，处理并验证，然后触发转发流程，将其广播给自己的邻居节点。


## 总结

本文详细介绍了 TRON P2P 网络的底层实现细节，涵盖了节点发现、连接、区块同步以及数据广播的核心机制。通过对这些模块的深入理解，开发者能够更好地认识 `java-tron` 网络的健壮性和高效性，并在此基础上进行二次开发与优化。