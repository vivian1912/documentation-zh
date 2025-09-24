# 智能合约

## 概述

智能合约是一种旨在自动执行合约条款的计算化交易协议。智能合约和普通合约一样，定义了参与者相关的条款和奖惩机制。一旦合约被启动，便能按照设定的条款执行，并自动检查所承诺的条款实施情形。

TRON 完全兼容以太坊（Ethereum）的智能合约生态，支持使用 Solidity 语言编写的合约。开发流程如下：
1. 在本地或在线 IDE 中编写并编译 Solidity 合约。
2. 将合约部署到 TRON 公链。
3. 部署后的合约会在被触发时于 TRON 网络的所有节点上自动执行。

您可以在 [TRON Solidity 代码库](https://github.com/tronprotocol/solidity/releases) 中获取最新版本。
 

## TRON 智能合约特性

TRON 虚拟机（TRON Virtual Machine, 简称 TVM）基于以太坊 Solidity 语言实现，在兼容以太坊虚拟机（Ethereum Virtual Machine，简称 EVM）功能的基础上，结合 TRON 自身特性存在部分差异。

### 智能合约的定义

TVM 运行的智能合约兼容以太坊合约特性。TRON 协议采用 **Protocol Buffers** 来封装和描述一个智能合约，其数据结构定义如下：

```
message SmartContract {
  message ABI {
    message Entry {
      enum EntryType {
        UnknownEntryType = 0;
        Constructor = 1;
        Function = 2;
        Event = 3;
        Fallback = 4;
        Receive = 5;
        Error = 6;
      }
      message Param {
        bool indexed = 1;
        string name = 2;
        string type = 3;
      }
      enum StateMutabilityType {
        UnknownMutabilityType = 0;
        Pure = 1;
        View = 2;
        Nonpayable = 3;
        Payable = 4;
      }

      bool anonymous = 1;
      bool constant = 2;
      string name = 3;
      repeated Param inputs = 4;
      repeated Param outputs = 5;
      EntryType type = 6;
      bool payable = 7;
      StateMutabilityType stateMutability = 8;
    }
    repeated Entry entrys = 1;
  }
  bytes origin_address = 1;
  bytes contract_address = 2;
  ABI abi = 3;
  bytes bytecode = 4;
  int64 call_value = 5;
  int64 consume_user_resource_percent = 6;
  string name = 7;
  int64 origin_energy_limit = 8;
  bytes code_hash = 9;
  bytes trx_hash = 10;
}
```
字段说明：

  - `origin_address`: 合约创建者地址
  - `contract_address`: 合约地址
  - `abi`: 合约中所有函数的接口信息
  - `bytecode`: 合约字节码
  - `call_value`: 调用合约时传入的 TRX 数量（以 sun 为单位）
  - `consume_user_resource_percent`: 开发者设定的、调用者需承担的资源消耗百分比
  - `name`: 合约名称
  - `origin_energy_limit`: 开发者设定的、部署者为单笔交易承担的能量（Energy）上限。该值必须大于 0。对于在部署时未提供能量上限参数的旧合约，其 `origin_energy_limit` 值会存储为 `0`，但在计算资源时，系统仍会按照 1000 万的能量上限来处理。开发者后续可以通过调用 `updateEnergyLimit` 接口来明确设定此值，且新值必须大于 0。

开发者可以通过 `CreateSmartContract` 和 `TriggerSmartContract` 这两种 gRPC 消息类型来创建和调用智能合约。

### 合约函数的使用

#### Constant 函数与非 Constant 函数

根据是否更改链上状态，合约函数可分为两类：

  - **Constant 函数**: 使用 `view`、`pure` 或 `constant` 修饰的函数。这类调用在节点本地执行并直接返回结果，不会产生交易广播。
  - **非 Constant 函数**: 会修改链上状态的函数，例如执行转账或更改合约内部变量。这类调用必须通过提交一笔交易来完成，并由网络共识确认。

> **注意**：若在合约内部使用 `CREATE` 指令（动态创建合约），即使该函数被 `view` 或 `pure` 等关键字修饰，仍会被视为非 constant 函数，并以交易的形式处理。

#### 消息调用 (Message Calls)

在合约执行期间，可以通过消息调用与其他合约交互，或向任何账户（合约或非合约）转帐 TRX。每次消息调用都包含发起者、接收者、数据、转账金额、扣费，以及返回值等属性，并可以递归生成新的消息调用。

当合约发起内部消息调用时，可以灵活控制能量分配：

* 为该调用指定可使用的能量上限；
* 为当前合约的后续执行预留部分能量。

通常使用 `{gas: gasleft() - 预留能量}` 的方式来设定该次内部调用可消耗的能量上限。

如果在调用过程中发生异常（例如 `OutOfEnergyException`）：

* 调用会返回 `false`，但不会抛出异常；
* 若为该调用设置了能量上限，则最多只会消耗分配给它的能量。若未显式设定，则该调用会消耗掉执行合约时剩余的全部能量。


#### 委托调用/代码调用与库（`delegatecall`/`callcode`/Library）

委托调用 (`delegatecall`) 是一种特殊类型的消息调用。它和一般的消息调用的区别在于，目标地址的合约代码在**发起者合约的上下文**中执行，且 `msg.sender` 和 `msg.value` 保持不变。这使得合约可在运行时动态加载外部代码，同时保持自身存储、地址和余额都指向发起调用的合约，只有代码是从被调用地址获取。

这种机制使得 Solidity 可以实现 **库（Library）** 的能力。开发者可以将可复用的代码部署为独立的库合约，然后通过 `delegatecall` 在调用合约的上下文中执行这些代码。一个典型的应用是使用库来实现复杂的数据结构（如链表、集合等），从而让多个主合约共享同一套功能逻辑，同时避免代码冗余。

#### `CREATE` 指令

`CREATE` 指令用于在合约内部动态地创建新的合约并生成新地址。与以太坊不同，TVM 生成新地址的方式是**基于当前智能合约的交易 ID 与一个内部调用计数器（`nonce`）的哈希组合**。其中，`nonce` 定义为**根调用下的创建序号**，即在一次合约执行中，每次使用 `CREATE` 指令都会生成一个递增的编号：第一次为 1，第二次为 2，以此类推。

> **注意**：通过 `CREATE` 指令创建的合约并不会自动保存 ABI，如果需要记录 ABI，则应使用 gRPC 接口 `deploycontract` 来部署合约。

#### TVM 内置功能

TRON 智能合约内置了多种功能，以支持常见的链上操作。最常见的是 **转账**，它可以出现在不同场景中，例如：

- 在构造函数（`constructor`）执行时随交易进行转账
- 在合约函数调用的过程中进行转账
- 使用 `transfer`、`send`、`call`、`callcode`、`delegatecall` 等方式进行转账

> 注意：在 TRON 智能合约中，在 TRON 智能合约中，如果转账目标地址尚未激活，不能通过智能合约转账的形式激活目标地址账户。这一点既不同于以太坊的处理方式，也不同于 TRON 系统合约的逻辑。

除了转账功能外，合约还可以执行更复杂的链上操作，包括：

- 为超级节点投票
- 质押（Stake）TRX
- 资源代理

TVM 同时保持了对以太坊大部分内置函数的兼容性，使开发者能够在已有经验的基础上快速迁移和开发合约。

## Solidity 中的 TRON 地址使用规范

在 Solidity 中正确处理地址是在 TRON 上开发智能合约的关键。核心需要理解的是 TRON 地址与以太坊地址在字节长度上的差异，以及 TRON 虚拟机（TVM）为兼容 Solidity 而做的内部处理。

**核心差异：20字节 vs 21字节**

- 以太坊地址：是一个 20 字节（40 个十六进制字符）的值。
- TRON 地址：是一个 21 字节的值。它由一个字节的地址前缀（通常是 `0x41`）和后面 20 字节的核心地址组成。

在 Solidity 中处理 TRON 地址时，需要遵循以下规范。

#### 地址转换

当您需要将一个 TRON 地址传入合约时，推荐的做法是将其作为一个 `uint256` 整数传入，然后在合约内部转换为 `address` 类型。

```
/**
 * @dev 将 uint256 格式的 TRON 地址转换为 Solidity 的 address 类型。
 * @param tronAddress uint256 格式的 TRON 地址，以 0x 开头，后接 HexString
 * @return address Solidity 的地址类型。
 */
function convertFromTronInt(uint256 tronAddress) public view returns(address) {
    return address(tronAddress);
}
```
这个和在以太坊中其他类型转换成 `address` 类型语法相同。


#### 地址判断

在 Solidity 中进行地址常量的比较时，必须使用 20 字节的格式，否则将导致编译器报错。TVM 在执行时会自动处理前缀 `0x41`。例如：

```
function compareAddress(address tronAddress) public view returns (uint256) {
    // 错误写法：包含 0x41 前缀，会导致编译器报错。
    // if (tronAddress == 0x41ca35b7d915458ef540ade6068dfe2f44e8fa733c) { ... }

    // 正确写法：使用 20 字节的地址格式。
    if (tronAddress == 0xca35b7d915458ef540ade6068dfe2f44e8fa733c) {
        return 1;
    } else {
        return 0;
    }
}
```
> **说明**：虽然代码中常量为 20 字节，但当 `tronAddress` 参数从外部（如 `wallet-cli`）传入一个完整的 21 字节 TRON 地址时，TVM 依然能正确完成判断，即依然能正确返回 `1`。

#### 地址赋值

与地址判断类似，在为 `address` 类型的变量赋常量值时，也必须省略 `0x41` 前缀，使用 20 字节的格式。例如：

```
function assignAddress() public pure {
    // 错误写法：包含 0x41 前缀，会导致编译器报错。
    // address newAddress = 0x41ca35b7d915458ef540ade6068dfe2f44e8fa733c;

    // 正确写法：
    address newAddress = 0xca35b7d915458ef540ade6068dfe2f44e8fa733c;
    // ... 后续操作
}
```
> 说明：
> - 当使用 `newAddress` 变量进行转账等链上操作时，TVM 会自动为其补上 `0x41` 前缀，以构成一个有效的 TRON 地址。
> - `TLLM21wteSPs4hKjbxgmH1L6poyMjeTbHm` 此类 Base58 格式的地址是钱包等客户端使用的字符串地址，不能直接在 Solidity 代码中使用，需要先解码为十六进制格式（HexString）。


## 全局变量与单位

在 Solidity 智能合约中，可以使用一系列全局变量来获取关于区块链、交易和调用的信息。

#### 货币单位

类似于 Solidity 对 `ether` 的支持，TVM 原生支持 `trx` 和 `sun` 两个货币单位，换算关系为 `1 trx = 1,000,000 sun`。这两个单位均为小写且大小写敏感。

  * 我们推荐使用 **TronIDE** 或 **TronBox** 进行 TRON 智能合约的开发，这两款工具均完整支持 `trx` 和 `sun` 单位。




#### 区块与交易相关的全局变量

以下是在 TRON 智能合约中常用的全局变量和函数，其中部分变量的行为与以太坊有所不同。

**区块信息**

  - `block.blockhash(uint blockNumber) returns (bytes32)`: 获取指定区块的哈希值。仅对最近的 256 个区块有效（不含当前区块）
  - `block.coinbase (address)`: 产出当前区块的超级节点地址
  - `block.number (uint)`：当前区块的高度（即区块号）
  - `block.timestamp (uint)`：当前区块的时间戳（以秒为单位）
  - `now (uint)`：当前区块的时间戳，与 `block.timestamp` 等价

**调用与交易信息**

- `msg.data (bytes)`：完整的调用数据 (`calldata`)
- `msg.sender (address)`：消息的发送者（当前调用的发起者）
- `msg.sig (bytes4)`：`calldata` 的前 4 字节，即函数标识符
- `msg.value (uint)`：随消息发送的 `sun` 的数量
- `tx.origin (address)`：交易的原始发起者

**Gas 与特殊变量**

- `gasleft() returns (uint256)`：剩余的 gas
- `block.difficulty (uint)`：当前区块的难度。在 TRON 网络中，此值恒定为 0，不推荐使用
- `block.gaslimit (uint)`：当前区块的 gas 限额。在 TRON 网络中不支持使用，暂时设置为 0
- `tx.gasprice (uint)`：交易的 gas 价格。在 TRON 网络中，此值恒定为 0，不推荐使用

