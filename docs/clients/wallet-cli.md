<a id="pending-pool"></a>
### Pending Pool
下面是Pending Pool相关API：

- [wallet/gettransactionfrompending](#walletgettransactionfrompending)
- [wallet/gettransactionlistfrompending](#walletgettransactionlistfrompending)
- [wallet/getpendingsize](#walletgetpendingsize)

#### wallet/gettransactionfrompending
作用：查询pending pool中的交易信息
```
curl -X POST  http://127.0.0.1:8090/wallet/gettransactionfrompending -d
'{
  "value": "txId"
}'
```
参数说明：
- `value`: 交易id，默认为hexString格式

返回值：完整的交易对象。如果交易不在等待池中，返回空对象。

#### wallet/gettransactionlistfrompending
作用：获取当前交易等待池中所有交易的ID列表。
```
curl -X get  http://127.0.0.1:8090/wallet/gettransactionlistfrompending
```
参数说明：无

返回值：一个包含了所有等待中交易ID的数组。

#### wallet/getpendingsize
作用：查询当前交易等待池中的交易数量。
```
curl -X get  http://127.0.0.1:8090/wallet/getpendingsize
```
参数说明：无

返回值：一个包含等待池大小的对象。

## FullNode Solidity HTTP API


### 账户资源

#### walletsolidity/getaccount
作用：查询并返回一个指定TRON账户的完整链上信息(包括余额、资源、权限、资产在内的所有账户详情)。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getaccount -d '{"address": "41E552F6487585C2B58BC2C9BB4492BC1F17132CD0"}'
```
参数：
* `address` 需要查询的账户地址。
* `visible` 设置地址格式,`true`为Base58Check，`false`或省略则为HexString。

返回值：Account对象

#### walletsolidity/getdelegatedresource
作用：Stake1.0中查询指定账户（代理方）为另一个特定账户（接收方）所代理的资源（能量或带宽）详情。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getdelegatedresource -d '
{
"fromAddress": "419844f7600e018fd0d710e2145351d607b3316ce9",
"toAddress": "41c6600433381c731f22fc2b9f864b14fe518b322f"
}'
```
参数说明：
- `fromAddress`：是要查询的账户地址，默认为hexString格式
- `toAddress`：代理对象的账户地址，默认为hexString格式
- `visible` 设置地址格式,`true`为Base58Check，`false`或省略则为HexString。

返回值：
- 账户的资源代理的列表，列表的元素为DelegatedResource

#### walletsolidity/getdelegatedresourceaccountindex
作用：Stake1.0中查询一个指定账户代理及被代理资源关系列表。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getdelegatedresourceaccountindex -d '
{
"value": "419844f7600e018fd0d710e2145351d607b3316ce9",
}'
```
参数：
- `value`：是要查询的账户地址，默认为hexString格式。
- `visible` 设置地址格式,`true`为Base58Check，`false`或省略则为HexString。

返回值：
- 账户的资源代理概况，结构为DelegatedResourceAccountIndex。

#### walletsolidity/getaccountbyid
作用：通过accountId查询一个账号的信息。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getaccountbyid -d '{"account_id":"6161616162626262"}'
```
参数说明：account_id 默认为hexString格式

返回值：Account对象

#### walletsolidity/getavailableunfreezecount

作用：查询一个指定账户当前还可发起**解质押操作的剩余次数**。由于TRON网络规定每个账户最多只能同时保有**32笔**处于14天锁定期内的解质押操作，此接口可用于在调用 `unfreezebalancev2` 之前，预先检查是否还有可用的“解质押额度”。
```
curl -X POST http://127.0.0.1:8090/walletsolidity/getavailableunfreezecount -d
'{
  "owner_address": "TZ4UXDV5ZhNW7fb2AMSbgfAEZ7hWsnYS2g",
  "visible": true
}
'
```

参数：
- `owner_address`: 需要查询的账户地址。
- `visible` 设置地址格式,`true`为Base58Check，`false`或省略则为HexString。

返回值：
- 该接口返回一个包含剩余次数的JSON对象。

#### walletsolidity/getcanwithdrawunfreezeamount

作用：查询在某时间点可以提取的解质押本金数量。
```
curl -X POST http://127.0.0.1:8090/walletsolidity/getcanwithdrawunfreezeamount -d
'{
  "owner_address": "TZ4UXDV5ZhNW7fb2AMSbgfAEZ7hWsnYS2g",
  "timestamp": 1667977444000,
  "visible": true
}
'
```

参数：
- `owner_address`: 交易发起者账号的地址。
- `timestamp`: 查询在该时间戳时，可提取的本金数量，单位为毫秒。
- `visible` 设置地址格式,`true`为Base58Check，`false`或省略则为HexString。

返回值：
- 该接口返回一个包含可提取金额的JSON对象。


#### walletsolidity/getcandelegatedmaxsize

作用：查询目标地址中指定类型资源的可代理数量，单位为sun
```
curl -X POST http://127.0.0.1:8090/walletsolidity/getcandelegatedmaxsize -d
'{
  "owner_address": "TZ4UXDV5ZhNW7fb2AMSbgfAEZ7hWsnYS2g",
  "type": 0,
  "visible": true
}
'
```

参数：
- `owner_address`: 需要查询的账户地址。
- `type`: 资源类型，0为带宽，1为能量。
- `visible` 设置地址格式,`true`为Base58Check，`false`或省略则为HexString。

返回值：
- 该接口返回一个包含可代理份额最大值的JSON对象。

#### walletsolidity/getdelegatedresourcev2

作用：查询在 Stake 2.0 机制下，指定账户（代理方）为另一个特定账户（接收方）所代理的资源详情。
```
curl -X POST http://127.0.0.1:8090/walletsolidity/getdelegatedresourcev2 -d
'{
  "fromAddress": "TZ4UXDV5ZhNW7fb2AMSbgfAEZ7hWsnYS2g",
  "toAddress": "TPswDDCAWhJAZGdHPidFg5nEf8TkNToDX1",
  "visible": true
}
'
```

参数：
- `fromAddress`: 代理账户地址。
- `toAddress`: 资源的接收账户地址。
- `visible` 设置地址格式,`true`为Base58Check，`false`或省略则为HexString。

返回值：
- 该接口返回一个 delegatedResource 数组，包含了两者在Stake 2.0下的所有资源代理记录。

#### walletsolidity/getdelegatedresourceaccountindexv2
作用：查询在Stake2.0阶段，某地址的资源委托索引。返回两个列表，一个是该帐户将资源委托给的地址列表(toAddress)，另一个是将资源委托给该帐户的地址列表(fromAddress)
```
curl -X POST http://127.0.0.1:8090/walletsolidity/getdelegatedresourceaccountindexv2 -d
'{
  "value": "TZ4UXDV5ZhNW7fb2AMSbgfAEZ7hWsnYS2g",
  "visible": true
}
'
```

参数：
- `value`: 账户地址。
- `visible` 设置地址格式,`true`为Base58Check，`false`或省略则为HexString。

返回值：
- 该接口返回一个包含双向代理关系列表的JSON对象。包含两个列表，一个是该帐户将资源委托给的地址列表(toAddress)，另一个是将资源委托给该帐户的地址列表(fromAddress)

### 投票和SR

#### walletsolidity/listwitnesses
作用：查询当前的所有witness列表。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/listwitnesses
```
参数说明：无

返回值：返回所有witness信息列表。

### TRC10 通证

#### walletsolidity/getassetissuelist
作用：查询所有Token列表
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getassetissuelist
```
参数说明：无

返回值：返回所有 Token 列表。

#### walletsolidity/getpaginatedassetissuelist
作用：分页查询全网的TRC-10 Token 列表。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getpaginatedassetissuelist -d '{"offset": 0, "limit":10}'
```
参数：
 - `offset`：分页查询的起始索引。
 - `limit`：本次查询期望返回的TRC-10 Token 数量。

返回值：一个包含了分页结果的TRC-10 Token 对象的数组。

#### walletsolidity/getassetissuebyname
作用：根据名称查询TRC-10 Token。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getassetissuebyname -d '{"value": "44756354616E"}'
```
参数：
 - `value`：TRC-10 Token 名称，默认为HexString格式。
返回值：TRC-10 Token 对象。

注意：Odyssey-v3.2开始，推荐使用getassetissuebyid或者getassetissuelistbyname替换此接口，因为从3.2开始将允许 TRC-10 Token 名称相同。如果存在相同的 TRC-10 Token 名称，此接口将会报错。

#### walletsolidity/getassetissuelistbyname
作用：根据名称查询所有匹配的 TRC-10 Token 列表。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getassetissuelistbyname -d '{"value": "44756354616E"}'
```
参数：
 - `value`：TRC-10 Token 名称，默认为HexString格式。

返回值：一个包含所有同名 TRC-10 Token 对象的数组。

#### walletsolidity/getassetissuebyid
作用：根据ID查询 TRC-10 Token
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getassetissuebyid -d '{"value": "1000001"}'
```
参数：
- `value`：TRC-10 Token 的 ID。

返回值：指定的 TRC-10 Token 对象。


### 区块

#### walletsolidity/getnowblock
作用：查询最新块。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getnowblock
```
参数说明：无

返回值：solidityNode 上的最新的区块对象。

#### walletsolidity/getblockbynum
作用：通过指定的区块高度查询完整的区块信息。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getblockbynum -d '{"num" : 100}'
```
参数：
- num：区块高度 (整型)。

返回值：指定高度的区块对象 (Block object)。

#### walletsolidity/getblockbyid
作用：通过指定的区块ID（哈希）查询完整的区块信息。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getblockbyid-d '{"value":
"0000000000038809c59ee8409a3b6c051e369ef1096603c7ee723c16e2376c73"}'
```
参数：
- value：区块的ID (hash)。

返回值：指定ID的区块对象 (Block object)。

#### walletsolidity/getblockbylimitnext
作用：分页查询指定高度范围内的区块列表。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getblockbylimitnext -d '{"startNum": 1, "endNum": 2}'
```
参数说明：

- `startNum`：起始块高度，包含此块
- `endNum`：截止块高度，不包含此此块

返回值：一个包含多个区块对象的数组 (Block[])。

#### walletsolidity/getblockbylatestnum
作用：查询solidityNode从最新区块开始，倒序的N个区块。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getblockbylatestnum -d '{"num": 5}'
```
参数：
- num：需要查询的区块数量。

返回值：一个包含多个区块对象的数组 (Block[])。

#### wallet/getnodeinfo
作用：查看当前节点自身的运行状态和信息。
```
curl -X GET http://127.0.0.1:8091/wallet/getnodeinfo
```
参数说明：无

返回值：一个包含节点版本、网络状况、区块同步状态等信息的对象。


### 交易

#### walletsolidity/gettransactionbyid
作用：通过交易ID（哈希）查询一个已上链交易的完整信息。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/gettransactionbyid -d '{"value" : "309b6fa3d01353e46f57dd8a8f27611f98e392b50d035cef213f2c55225a8bd2"}'
```
参数：
- value：交易ID (hash)。

返回值：完整的交易对象 (Transaction object)。如果交易不存在或未确认，返回空对象。

#### walletsolidity/gettransactioncountbyblocknum
作用：查询指定区块高度上包含的交易总数。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/gettransactioncountbyblocknum -d '{"num" : 100}'
```
参数：
- num：区块高度。

返回值：一个包含交易数量的对象，如 {"count": 50}。

#### walletsolidity/gettransactioninfobyid
作用：根据交易ID（哈希）查询交易的摘要信息，如费用、所在区块等。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/gettransactioninfobyid -d '{"value" : "309b6fa3d01353e46f57dd8a8f27611f98e392b50d035cef213f2c55225a8bd2"}'
```
参数：
- value：交易ID (hash)。

返回值：交易的摘要信息对象 (TransactionInfo object)，包含交易费用、所在区块高度、区块时间戳、合约执行结果等。

#### walletsolidity/gettransactioninfobyblocknum
作用：获取指定区块高度上所有交易的摘要信息列表。
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/gettransactioninfobyblocknum -d '{"num" : 100}'
```
参数：
- num：区块高度。

返回值：一个包含多个交易摘要信息对象的列表。


### 去中心化交易所

#### walletsolidity/getexchangebyid
作用：根据id查询交易对
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/getexchangebyid -d {"id":1}
```
参数说明：
- `id`：交易对id

返回值：返回指定的交易对对象。

#### walletsolidity/listexchanges
作用：查询所有交易对
```
curl -X POST  http://127.0.0.1:8091/walletsolidity/listexchanges
```
参数说明：无

返回值：一个包含所有交易对对象的数组。


