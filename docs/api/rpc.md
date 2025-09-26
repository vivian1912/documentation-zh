# gRPC API

## 概述

本文档提供了 TRON 区块链网络中远程过程调用 gRPC API 的详细参考。这些 API 允许开发者与 TRON 网络交互，执行账户管理、交易广播、智能合约操作、资源管理等多种功能。

**注意：** SolidityNode 已被弃用。现在 FullNode 支持 SolidityNode 的所有 gRPC。新的开发者应仅部署 FullNode。

**API 定义参考:** 本文档介绍了各主要 API 的功能。如需查看最完整和权威的 API 技术定义，请访问 [api/api.proto](https://github.com/tronprotocol/protocol/blob/master/api/api.proto)。

## 账户管理

### 获取账户信息

```
rpc GetAccount (Account) returns (Account) {}
```

### 根据 ID 获取账户信息

```
rpc GetAccountById (Account) returns (Account) {}
```

### 创建账户

```
rpc CreateAccount2 (AccountCreateContract) returns (TransactionExtention) {}
```

### 更新账户名称

```
rpc UpdateAccount2 (AccountUpdateContract) returns (TransactionExtention) {}
```

### 设置账户 ID

```
rpc SetAccountId (SetAccountIdContract) returns (Transaction) {}
```

### 更新账户权限

```
rpc AccountPermissionUpdate (AccountPermissionUpdateContract) returns (TransactionExtention) {}
```

## 交易操作

### TRX 转账

```
rpc CreateTransaction2 (TransferContract) returns (TransactionExtention) {}
```

### 广播交易

```
rpc BroadcastTransaction (Transaction) returns (Return) {}
```

**描述:**
此 RPC 用于将已签名的交易信息发送到节点。交易经过超级代表 (Super Representative，简称 SR) 验证后，将被广播到整个网络。

### 查询交易信息 (按交易 ID)

```
rpc GetTransactionById (BytesMessage) returns (Transaction) {}
```

### 查询交易费用与区块信息 (按交易 ID)

```
rpc GetTransactionInfoById (BytesMessage) returns (TransactionInfo) {}
```

### 查询待处理池中的交易信息

```
rpc GetTransactionFromPending (BytesMessage) returns (Transaction) {};
```

### 查询待处理池交易 ID 列表

```
rpc GetTransactionListFromPending (EmptyMessage) returns (TransactionIdList) {};
```

### 查询待处理池的大小

```
rpc GetPendingSize (EmptyMessage) returns (NumberMessage) {};
```

### 查询指定区块的交易数目

```
rpc GetTransactionCountByBlockNum (NumberMessage) returns (NumberMessage) {};
```

### 查询交易的当前权限权重

```
rpc GetTransactionSignWeight (Transaction) returns (TransactionSignWeight) {};
```

### 查询交易的权限批准列表

```
rpc GetTransactionApprovedList (Transaction) returns (TransactionApprovedList) {};
```


### 查询指定地址发出的交易

```
rpc GetTransactionsFromThis2 (AccountPaginated) returns (TransactionListExtention) {}
```

### 查询指定地址接收的交易

```
rpc GetTransactionsToThis2 (AccountPaginated) returns (TransactionListExtention) {}
```

## 代币 (TRC-10) 操作

### 发行代币

```
rpc CreateAssetIssue2 (AssetIssueContract) returns (TransactionExtention) {}
```

### 更新代币信息

```
rpc UpdateAsset2 (UpdateAssetContract) returns (TransactionExtention) {}
```

**描述:**
代币更新只能由代币发行人发起，用于更新：

- 代币描述
- URL
- 每个账户的最大带宽消耗
- 总带宽消耗

### 代币转账

```
rpc TransferAsset2 (TransferAssetContract) returns (TransactionExtention) {}
```

### 参与代币发行

```
rpc ParticipateAssetIssue2 (ParticipateAssetIssueContract) returns (TransactionExtention) {}
```

### 查询所有已发行代币列表

```
rpc GetAssetIssueList (EmptyMessage) returns (AssetIssueList) {}
```

### 查询给定账户发行的代币

```
rpc GetAssetIssueByAccount (Account) returns (AssetIssueList) {}
```

### 按代币名称查询代币信息

```
rpc GetAssetIssueByName (BytesMessage) returns (AssetIssueContract) {}
```

### 按时间戳查询代币列表

```
rpc GetAssetIssueListByTimestamp (NumberMessage) returns (AssetIssueList){}
```

### 分页查询所有代币列表

```
rpc GetPaginatedAssetIssueList (PaginatedMessage) returns (AssetIssueList) {}
```

### 解冻代币

```
rpc UnfreezeAsset2 (UnfreezeAssetContract) returns (TransactionExtention) {}
```

### 按名称查询代币

```
rpc GetAssetIssueListByName (BytesMessage) returns (AssetIssueList) {}
```

### 按 ID 查询代币

```
rpc GetAssetIssueById (BytesMessage) returns (AssetIssueContract) {}
```


## 超级代表 (SR) 与投票

### 投票给超级代表候选人

```
rpc VoteWitnessAccount2 (VoteWitnessContract) returns (TransactionExtention) {}
```

### 查询超级代表的佣金比例

```
rpc GetBrokerageInfo (BytesMessage) returns (NumberMessage) {}
```

### 查询未领取的奖励

```
rpc GetRewardInfo (BytesMessage) returns (NumberMessage) {}
```

### 更新佣金比例

```
rpc UpdateBrokerage (UpdateBrokerageContract) returns (TransactionExtention) {}
```

### 申请成为超级代表

```
rpc CreateWitness2 (WitnessCreateContract) returns (TransactionExtention) {}
```

**描述:**
申请成为 TRON 的超级代表候选人。

### 更新超级代表候选人信息

```
rpc UpdateWitness2 (WitnessUpdateContract) returns (TransactionExtention) {}
```

**描述:** 更新超级代表的网站 URL。

### 查询所有超级代表候选人

```
rpc ListWitnesses (EmptyMessage) returns (WitnessList) {}
```

### 区块奖励提取

```
rpc WithdrawBalance2 (WithdrawBalanceContract) returns (TransactionExtention) {}
```

### 创建 Proposal

```
rpc ProposalCreate (ProposalCreateContract) returns (TransactionExtention) {}
```

### 给 Proposal 投票或取消投票

```
rpc ProposalApprove (ProposalApproveContract) returns (TransactionExtention) {}
```

### 删除 Proposal

```
rpc ProposalDelete (ProposalDeleteContract) returns (TransactionExtention) {}
```


## 资源管理 (TRX 质押与委托)

### 质押 TRX (旧版本，已废弃)

该接口已废弃。请使用 `FreezeBalanceV2`。
```
rpc FreezeBalance (FreezeBalanceContract) returns (Transaction) {}
```

### 解质押 TRX (旧版本，已废弃)

```
rpc UnfreezeBalance (UnfreezeBalanceContract) returns (Transaction) {}
```

### 质押 TRX V2

```
rpc FreezeBalanceV2 (FreezeBalanceV2Contract) returns (TransactionExtention) {}
```

### 解质押 TRX V2

```
rpc UnfreezeBalanceV2 (UnfreezeBalanceV2Contract) returns (TransactionExtention) {}
```

### 委托资源

```
rpc DelegateResource (DelegateResourceContract) returns (TransactionExtention) {}
```

### 取消委托资源

```
rpc UnDelegateResource (UnDelegateResourceContract) returns (TransactionExtention) {}
```

### 取消所有解质押

```
rpc CancelAllUnfreezeV2 (CancelAllUnfreezeV2Contract) returns (TransactionExtention) {}
```

### 获取账户剩余的解质押次数

```
rpc GetAvailableUnfreezeCount (GetAvailableUnfreezeCountRequestMessage)
      returns (GetAvailableUnfreezeCountResponseMessage) {}; 
```

### 获取一个地址的指定资源类型的可委托资源量，单位为 sun

```
rpc GetCanDelegatedMaxSize (CanDelegatedMaxSizeRequestMessage) returns (CanDelegatedMaxSizeResponseMessage) {}; 
```

### 获取指定时间可提取的到期的解质押数量

```
rpc GetCanWithdrawUnfreezeAmount (CanWithdrawUnfreezeAmountRequestMessage)
      returns (CanWithdrawUnfreezeAmountResponseMessage) {}; 
```

### 提取到期的解质押

用户执行 `/wallet/unfreezebalancev2` 交易并等待 N 天（N 为网络参数）后，可调用此 API 取回资金。

```
rpc WithdrawExpireUnfreeze (WithdrawExpireUnfreezeContract) returns (TransactionExtention) {}; 
```

## 智能合约操作

### 部署智能合约

```
rpc DeployContract (CreateSmartContract) returns (TransactionExtention) {}
```

### 触发智能合约

```
rpc TriggerContract (TriggerSmartContract) returns (TransactionExtention) {}
```

### 预估能量消耗

```
rpc EstimateEnergy (TriggerSmartContract) returns (EstimateEnergyMessage) {}
```

### 获取智能合约

```
rpc GetContract (BytesMessage) returns (SmartContract) {}
```

### 更新智能合约中的`consume_user_resource_percent`（用户能量消耗占比）

```
rpc UpdateSetting (UpdateSettingContract) returns (TransactionExtention) {}
```

### 更新智能合约中的 `origin_energy_limit`

```
rpc UpdateEnergyLimit (UpdateEnergyLimitContract) returns (TransactionExtention) {}
```




## 市场交易 (Market Order)

### 创建市场卖单

```
rpc MarketSellAsset (MarketSellAssetContract) returns (TransactionExtention) {};
```

### 取消市场订单      

```
rpc MarketCancelOrder (MarketCancelOrderContract) returns (TransactionExtention) {};
```

### 获取账户的所有市场订单        

```
rpc GetMarketOrderByAccount (BytesMessage) returns (MarketOrderList) {};
```

### 获取所有市场交易对         

```
rpc GetMarketPairList (EmptyMessage) returns (MarketOrderPairList) {};
``` 

### 获取交易对的所有市场订单        

```
rpc GetMarketOrderListByPair (MarketOrderPair) returns (MarketOrderList) {};
```

### 获取交易对的所有市场价格        

```
rpc GetMarketPriceByPair (MarketOrderPair) returns (MarketPriceList) {};
```

### 按 ID 获取市场订单      

```
rpc GetMarketOrderById (BytesMessage) returns (MarketOrder) {}; 
``` 

### 创建交易对      

```
rpc ExchangeCreate (ExchangeCreateContract) returns (TransactionExtention) {}; 
``` 

### 给交易对注入资金     

```
rpc ExchangeInject (ExchangeInjectContract) returns (TransactionExtention) {}; 
``` 

### 对交易对撤资    

```
rpc ExchangeWithdraw (ExchangeWithdrawContract) returns (TransactionExtention) {}; 
``` 

### 在交易对中进行交易    

```
rpc ExchangeTransaction (ExchangeTransactionContract) returns (TransactionExtention) {}; 
``` 

### 查询所有交易对

```
rpc ListExchanges (EmptyMessage) returns (ExchangeList) {}
```

### 分页查询交易对

```
rpc GetPaginatedExchangeList (PaginatedMessage) returns (ExchangeList) {}
```

### 按 ID 查询交易对

```
rpc GetExchangeById (BytesMessage) returns (Exchange) {}
```

## 区块链数据查询

### 获取当前区块信息

```
rpc GetNowBlock2 (EmptyMessage) returns (BlockExtention) {}
```

### 按区块高度获取区块

```
rpc GetBlockByNum2 (NumberMessage) returns (BlockExtention) {}
```

### 获取区块信息 (按区块 ID)

```
rpc GetBlockById (BytesMessage) returns (Block) {}
```

### 获取总交易数量（已废弃，固定返回0）

```
rpc TotalTransaction (EmptyMessage) returns (NumberMessage) {}
```

### 查询账户在特定区块高度的余额

```
rpc GetAccountBalance (AccountBalanceRequest) returns (AccountBalanceResponse){}; 
``` 

### 获取账户带宽       

```
rpc GetAccountNet (Account) returns (AccountNetMessage){}; 
```

### 获取账户资源（能量，带宽）

```
rpc GetAccountResource (Account) returns (AccountResourceMessage){}; 
```
**注意**：只有在配置文件里设置了 `storage.balance.history.lookup= true` 的节点才支持查
询账户历史余额。支持此接口的官方节点可在 [此处](../using_javatron/backup_restore.md/#fullnode) 查阅。


### 获取区块内账号余额变更

```
rpc GetBlockBalanceTrace (BlockBalanceTrace.BlockIdentifier) returns (BlockBalanceTrace) {}; 
```

### 获取累计销毁的 TRX 数量

```
rpc GetBurnTrx (EmptyMessage) returns (NumberMessage) {}; 
```

### 获取指定区块的所有交易回执信息

```
rpc GetTransactionInfoByBlockNum (NumberMessage) returns (TransactionInfoList) {}; 
```

### 获取指定区块的信息       

默认返回最新区块的区块头，可查询指定的区块头信息，或者包含交易体的区块信息

```
rpc GetBlock (BlockReq) returns (BlockExtention) {}; 
```

## 治理相关

### 查询所有提议

```
rpc ListProposals (EmptyMessage) returns (ProposalList) {}
```

### 分页查询提议

```
rpc GetPaginatedProposalList (PaginatedMessage) returns (ProposalList) {}
```

### 按 ID 查询提议

```
rpc GetProposalById (BytesMessage) returns (Proposal) {}
```

### 查询所有委员会能设置的参数

```
rpc GetChainParameters (EmptyMessage) returns (ChainParameters) {}
```



## 其他类型

### 查询下一次维护时间

```
rpc GetNextMaintenanceTime (EmptyMessage) returns (NumberMessage) {}
```

### 查询节点信息

```
rpc GetNodeInfo (EmptyMessage) returns (NodeInfo) {}
```

### 获取带宽单价的变化的历史记录

```
rpc GetBandwidthPrices (EmptyMessage) returns (PricesResponseMessage) {}
```

### 获取能量单价的变化的历史记录

```
rpc GetEnergyPrices (EmptyMessage) returns (PricesResponseMessage) {}
```

### 获取交易备注费用变化的历史记录

```
rpc GetMemoFee (EmptyMessage) returns (PricesResponseMessage) {}
```

### 获取区块引用

```
rpc getBlockReference (EmptyMessage) returns (BlockReference) {}
```

### 获取网络动态参数

```
rpc GetDynamicProperties (EmptyMessage) returns (DynamicProperties) {}
```

### 获取节点状态信息

```
rpc GetStatsInfo (EmptyMessage) returns (MetricsInfo) {}
```

### 获取 peer 信息

```
rpc ListNodes (EmptyMessage) returns (NodeList) {}
```

### 查询指定区间的区块信息

```
rpc GetBlockByLimitNext2 (BlockLimit) returns (BlockListExtention) {}
```
