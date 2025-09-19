# RPC List

**For the specific definition of API, please refer to the following link:**
[api/api.proto](https://github.com/tronprotocol/protocol/blob/master/api/api.proto)

!!! note
    SolidityNode is deprecated. Now a FullNode supports all RPCs of a SolidityNode. New developers should deploy FullNode only.

## Get account information

```protobuf
rpc GetAccount (Account) returns (Account) {}
```
Nodes: Fullnode and SolidityNode

## TRX transfer

```protobuf
rpc CreateTransaction (TransferContract) returns (Transaction) {}
```
Nodes: Fullnode

## Broadcast transaction

```protobuf
rpc BroadcastTransaction (Transaction) returns (Return) {}
```
Nodes: Fullnode

Description:
Transfer, vote, issuance of token, or participation in token offering. Sending signed transaction information to node, and broadcasting it to the entire network after SR(Super Representatives) verification.

## Create an account

```protobuf
rpc CreateAccount (AccountCreateContract) returns (Transaction) {}
```
Nodes: FullNode

## Account name update
```protobuf
rpc UpdateAccount (AccountUpdateContract) returns (Transaction) {}
```
Nodes: Fullnode

## Vote for super representative candidates
```protobuf
rpc VoteWitnessAccount (VoteWitnessContract) returns (Transaction) {}
```
Nodes: FullNode

## Query the ratio of brokerage of the super representative
```protobuf
rpc GetBrokerageInfo (BytesMessage) returns (NumberMessage) {}
```
Nodes: FullNode

## Query unclaimed reward
```protobuf
rpc GetRewardInfo (BytesMessage) returns (NumberMessage) {}
```
Nodes: FullNode

## Update the ratio of brokerage
```protobuf
rpc UpdateBrokerage (UpdateBrokerageContract) returns (TransactionExtention) {}
```
Nodes: FullNode

## Issue a token
```protobuf
rpc CreateAssetIssue (AssetIssueContract) returns (Transaction) {}
```
Nodes: FullNode

## Query of list of super representative candidates
```protobuf
rpc ListWitnesses (EmptyMessage) returns (WitnessList) {}
```
Nodes: FullNode and SolidityNode

## Application for super representative
```protobuf
rpc CreateWitness (WitnessCreateContract) returns (Transaction) {}
```
Nodes: FullNode

Description:
To apply to become TRON’s Super Representative candidate.

## Information update of Super Representative candidates
```protobuf
rpc UpdateWitness (WitnessUpdateContract) returns (Transaction) {}
```
Nodes: FullNode

Description: Update the website url of the SR.

## Token transfer
```protobuf
rpc TransferAsset (TransferAssetContract) returns (Transaction){}
```
Node: FullNode

## Participate a token
```protobuf
rpc ParticipateAssetIssue (ParticipateAssetIssueContract) returns (Transaction) {}
```
Nodes: FullNode

## Query the list of nodes connected to the ip of the api
```protobuf
rpc ListNodes (EmptyMessage) returns (NodeList) {}
```
Nodes: FullNode and SolidityNode

## Query the list of all issued tokens
```protobuf
rpc GetAssetIssueList (EmptyMessage) returns (AssetIssueList) {}
```
Nodes: FullNode and SolidityNode

## Query the token issued by a given account
```protobuf
rpc GetAssetIssueByAccount (Account) returns (AssetIssueList) {}
```
Nodes: FullNode and SolidityNode

## Query the token information by token name
```protobuf
rpc GetAssetIssueByName (BytesMessage) returns (AssetIssueContract) {}
```
Nodes: FullNode and Soliditynode

## Query the list of tokens by timestamp
```protobuf
rpc GetAssetIssueListByTimestamp (NumberMessage) returns (AssetIssueList){}
```
Nodes: SolidityNode

## Get current block information
```protobuf
rpc GetNowBlock (EmptyMessage) returns (Block) {}
```
Nodes: FullNode and SolidityNode

## Get a block by block height
```protobuf
rpc GetBlockByNum (NumberMessage) returns (Block) {}
```
Nodes: FullNode and SolidityNode

## Get the total number of transactions
```protobuf
rpc TotalTransaction (EmptyMessage) returns (NumberMessage) {}
```
Nodes: FullNode and SolidityNode

## Query the transaction by transaction id
```protobuf
rpc getTransactionById (BytesMessage) returns (Transaction) {}
```
Nodes: SolidityNode

## Query the transaction by timestamp
```protobuf
rpc getTransactionsByTimestamp (TimeMessage) returns (TransactionList) {}
```
Nodes: SolidityNode

## Freeze TRX
该接口已废弃。
```protobuf
rpc FreezeBalance (FreezeBalanceContract) returns (Transaction) {}
```
Nodes: FullNode

## Unfreeze TRX
```protobuf
rpc UnfreezeBalance (UnfreezeBalanceContract) returns (Transaction) {}
```
Nodes: FullNode

## Block producing reward redemption
```protobuf
rpc WithdrawBalance (WithdrawBalanceContract) returns (Transaction) {}
```
Nodes: FullNode

## Unfreeze token balance
```protobuf
rpc UnfreezeAsset (UnfreezeAssetContract) returns (Transaction) {}
```
Nodes: FullNode

## Query the next maintenance time
```protobuf
rpc GetNextMaintenanceTime (EmptyMessage) returns (NumberMessage) {}
```
Nodes: FullNode

## Query the transaction fee & block information
```protobuf
rpc GetTransactionInfoById (BytesMessage) returns (TransactionInfo) {}
```
Nodes: SolidityNode

## Query block information by block id
```protobuf
rpc GetBlockById (BytesMessage) returns (Block) {}
```
Nodes: FullNode

## Update token information
```protobuf
rpc UpdateAsset (UpdateAssetContract) returns (Transaction) {}
```
Nodes: Fullnode

Description:
Token update can only be initiated by the token issuer to update token description, url, maximum bandwidth consumption by each account and total bandwidth consumption.

## Query the list of all the tokens by pagination
```protobuf
rpc GetPaginatedAssetIssueList (PaginatedMessage) returns (AssetIssueList) {}
```
Nodes: FullNode and SolidityNode

## Deploy a smart contract
```protobuf
rpc DeployContract (CreateSmartContract) returns (TransactionExtention) {}
```
Nodes: FullNode and SolidityNode

## Trigger a smart contract
```protobuf
rpc TriggerContract (TriggerSmartContract) returns (TransactionExtention) {}
```
Nodes: FullNode

## Create an market order
```
rpc MarketSellAsset (MarketSellAssetContract) returns (TransactionExtention) {};
```
Nodes: FullNode
 
## Cancel the order      
```    
rpc MarketCancelOrder (MarketCancelOrderContract) returns (TransactionExtention) {};
```
Nodes: FullNode 

## Get all orders for the account        
```
rpc GetMarketOrderByAccount (BytesMessage) returns (MarketOrderList) {};
```
Nodes: FullNode 

## Get all trading pairs         
```
rpc GetMarketPairList (EmptyMessage) returns (MarketOrderPairList) {};
```
Nodes: FullNode 

## Get all orders for the trading pair        
```
rpc GetMarketOrderListByPair (MarketOrderPair) returns (MarketOrderList) {};
```
Nodes: FullNode 

## Get all prices for the trading pair        
```
rpc GetMarketPriceByPair (MarketOrderPair) returns (MarketPriceList) {};
```
Nodes: FullNode 

## Get order by id      
```
rpc GetMarketOrderById (BytesMessage) returns (MarketOrder) {}; 
```
Nodes: FullNode 
    
## perform a historical balance lookup       
```
rpc GetAccountBalance (AccountBalanceRequest) returns (AccountBalanceResponse){}; 
```
Nodes: FullNode 

**注意**：只有在配置文件里设置了`storage.balance.history.lookup= true`的节点才支持查
询账户历史余额。支持此接口的官方节点可在[此处](../using_javatron/backup_restore.md/#fullnode)查阅。

## fetch all balance-changing transactions in a block      
```
rpc GetBlockBalanceTrace (BlockBalanceTrace.BlockIdentifier) returns (BlockBalanceTrace) {}; 
```
Nodes: FullNode 

## get the burn trx amount       
```
rpc GetBurnTrx (EmptyMessage) returns (NumberMessage) {}; 
```
Nodes: FullNode and SolidityNode

## Freeze TRX
```protobuf
rpc FreezeBalanceV2 (FreezeBalanceV2Contract) returns (TransactionExtention) {}
```
Nodes: FullNode

## UnFreeze TRX
```protobuf
rpc UnfreezeBalanceV2 (UnfreezeBalanceV2Contract) returns (TransactionExtention) {}
```
Nodes: FullNode

## Withdraw Staked TRX
```protobuf
rpc WithdrawExpireUnfreeze (WithdrawExpireUnfreezeContract) returns (TransactionExtention) {}
```
Nodes: FullNode

## Delegate Resource
```protobuf
rpc DelegateResource (DelegateResourceContract) returns (TransactionExtention) {}
```
Nodes: FullNode

## UnDelegate Resource
```protobuf
rpc UnDelegateResource (UnDelegateResourceContract) returns (TransactionExtention) {}
```
Nodes: FullNode

## Query transaction information in the pending pool
```
rpc GetTransactionFromPending (BytesMessage) returns (Transaction) {};
```
Nodes: FullNode

## Query the pending pool transaction id list
```
rpc GetTransactionListFromPending (EmptyMessage) returns (TransactionIdList) {};
```
Nodes: FullNode

## Query the size of the pending pool
```
rpc GetPendingSize (EmptyMessage) returns (NumberMessage) {};
Nodes: FullNode
```

##  Cancel UnFreeze
```protobuf
rpc CancelAllUnfreezeV2 (CancelAllUnfreezeV2Contract) returns (TransactionExtention) {}
```
Nodes: FullNode

##  Get bandwidth unit price
```protobuf
rpc GetBandwidthPrices (EmptyMessage) returns (PricesResponseMessage) {}
```
Nodes: FullNode

##  Get energy unit price
```protobuf
rpc GetEnergyPrices (EmptyMessage) returns (PricesResponseMessage) {}
```
Nodes: FullNode

##  Get transaction memo fee
```protobuf
rpc GetMemoFee (EmptyMessage) returns (PricesResponseMessage) {}
```
Nodes: FullNodes
