# 系统合约

TRON 网络支持多种多样的交易类型，例如 TRX 转账、TRC-10 代币转账、智能合约部署与调用，以及 TRX 质押等。每种交易类型都由一个特定的系统合约来定义和执行。开发者可以通过调用不同的 API 接口来创建这些交易。

举例来说，部署智能合约需要调用 `wallet/deploycontract` 接口，其对应的合约类型为 `CreateSmartContract`。而质押 TRX 获取资源则需调用 `wallet/freezebalancev2` 接口，其合约类型为 `FreezeBalanceV2Contract`。

本文档将详细介绍 TRON 网络中主要的系统合约类型及其包含的参数。


## 1. 账户管理合约

### AccountCreateContract
*创建新账户*

```
message AccountCreateContract {
   bytes owner_address = 1;
   bytes account_address = 2;
   AccountType type = 3;
}
```

* `owner_address`: 合约发起人地址。
* `account_address`: 待创建的账户地址。
* `type`: 账户类型。
    * `0`: 普通账户。
    * `1`: 创世块中的初始账户。
    * `2`: 智能合约账户。

### AccountUpdateContract
*更新账户名称*

```
// Update account name. Account name is unique now.
message AccountUpdateContract {
   bytes account_name = 1;
   bytes owner_address = 2;
}
```

* `owner_address`: 合约发起人地址。
* `account_name`: 账户名称。

### SetAccountIdContract
*设置账户 ID*

```
// Set account id if the account has no id. Account id is unique and case insensitive.
message SetAccountIdContract {
   bytes account_id = 1;
   bytes owner_address = 2;
}
```

* `owner_address`: 合约发起人地址。
* `account_id`: 账户 ID，唯一且不区分大小写。


## 2. TRX 转账与资源合约

### TransferContract
*TRX 转账*

```
message TransferContract {
   bytes owner_address = 1;
   bytes to_address = 2;
   int64 amount = 3;
}
```

* `owner_address`: 合约发起人地址。
* `to_address`: 目标账户地址。
* `amount`: 转账金额，单位为 **sun** (1 TRX = 1,000,000 sun)。

### FreezeBalanceV2Contract
*质押 TRX 获取资源（Stake 2.0）*

```
message FreezeBalanceV2Contract {
   bytes owner_address = 1;
   int64 frozen_balance = 2;
   ResourceCode resource = 3; // 类型可以是 Bandwidth 或 Energy
}
```

* `owner_address`: 质押者地址。
* `frozen_balance`: 质押的 TRX 数量。
* `resource`: 质押 TRX 以获取的资源类型。

### UnfreezeBalanceV2Contract
*解质押 TRX（Stake 2.0）*

```
message UnfreezeBalanceV2Contract {
   bytes owner_address = 1;
   int64 unfreeze_balance = 2;
   ResourceCode resource = 3;
}
```

* `owner_address`: 解质押者地址。
* `unfreeze_balance`: 解质押的 TRX 数量。
* `resource`: 解质押的资源类型。

### WithdrawExpireUnfreezeContract
*提取已到期解质押的 TRX 本金*

```
message WithdrawExpireUnfreezeContract {
   bytes owner_address = 1;
}
```

* `owner_address`: 提取本金的账户地址。

### DelegateResourceContract
*代理资源（委托）*

```
message DelegateResourceContract {
   bytes owner_address = 1;
   ResourceCode resource = 2;
   int64 balance = 3;
   bytes receiver_address = 4;
   bool  lock = 5;
   int64 lock_period = 6;
}
```

* `owner_address`: 资源代理者地址。
* `resource`: 代理的资源类型。
* `balance`: 代理的资源份额，单位为 **sun**。
* `receiver_address`: 资源接收者地址。
* `lock`: 是否将代理操作锁定 3 天。
* `lock_period`: 代理锁定期。可以是 (0, 86400] 之间的任何时间，单位是区块个数。仅当 `lock` 为 `true` 时生效。

### UnDelegateResourceContract
*取消资源代理（取消委托）*

```
message UnDelegateResourceContract {
   bytes owner_address = 1;
   ResourceCode resource = 2;
   int64 balance = 3;
   bytes receiver_address = 4;
}
```

* `owner_address`: 取消代理发起者地址。
* `resource`: 解除代理的资源类型。
* `balance`: 解除代理的资源份额。
* `receiver_address`: 资源接收者地址。

### *（已废弃）* FreezeBalanceContract
*质押 TRX 获取资源（Stake 1.0）*

```
message FreezeBalanceContract {
   bytes owner_address = 1;
   int64 frozen_balance = 2;
   int64 frozen_duration = 3;
   ResourceCode resource = 10;
   bytes receiver_address = 15;
}
```

* `owner_address`: 合约发起人地址。
* `frozen_balance`: 质押的 TRX 数量。
* `frozen_duration`: 质押时间（单位：天）。
* `resource`: 质押 TRX 获取的资源类型。
* `receiver_address`: 接收资源的账户地址。

### *（已废弃）* UnfreezeBalanceContract
*解质押 TRX（Stake 1.0）*

```
message UnfreezeBalanceContract {
   bytes owner_address = 1;
   ResourceCode resource = 10;
   bytes receiver_address = 13;
}
```

* `owner_address`: 合约发起人地址。
* `resource`: 解锁资源的类型。
* `receiver_address`: 接收资源的账户地址。

### WithdrawBalanceContract
*提取超级代表奖励*

```
message WithdrawBalanceContract {
   bytes owner_address = 1;
}
```

* `owner_address`: 合约发起人地址。


## 3. TRC-10 代币合约

### TransferAssetContract
*TRC-10 代币转账*

```
message TransferAssetContract {
   bytes asset_name = 1;
   bytes owner_address = 2;
   bytes to_address = 3;
   int64 amount = 4;
}
```

* `asset_name`: TRC-10 代币 ID。
* `owner_address`: 合约发起人地址。
* `to_address`: 目标账户地址。
* `amount`: 转账的代币数量。

### AssetIssueContract
*发行 TRC-10 代币*

```
message AssetIssueContract {
   message FrozenSupply {
     int64 frozen_amount = 1;
     int64 frozen_days = 2;
   }
   bytes owner_address = 1;
   bytes name = 2;
   bytes abbr = 3;
   int64 total_supply = 4;
   repeated FrozenSupply frozen_supply = 5;
   int32 trx_num = 6;
   int32 num = 8;
   int64 start_time = 9;
   int64 end_time = 10;
   int64 order = 11; // the order of tokens of the same name
   int32 vote_score = 16;
   bytes description = 20;
   bytes url = 21;
   int64 free_asset_net_limit = 22;
   int64 public_free_asset_net_limit = 23;
   int64 public_free_asset_net_usage = 24;
   int64 public_latest_free_net_time = 25;
}
```

* `owner_address`: 代币发行者地址。
* `name`: 代币名称。
* `abbr`: 代币缩写。
* `total_supply`: 发行总量。
* `frozen_supply`: 质押的代币数量和质押天数列表。
* `trx_num`: 1 TRX 可兑换的代币数量。
* `num`: 兑换 1 TRX 所需的代币数量。
* `start_time`: ICO 开始时间。
* `end_time`: ICO 结束时间。
* `order`: *（已废弃）*。
* `vote_score`: *（已废弃）*。
* `description`: 代币描述。
* `url`: 代币官方网址。
* `free_asset_net_limit`: 每个账户使用该代币时的免费带宽额度。
* `public_free_asset_net_limit`: 所有账户共享的免费带宽总额度。
* `public_free_asset_net_usage`: 所有账户已使用的免费带宽。
* `public_latest_free_net_time`: 最近一次使用共享免费带宽的时间。

### ParticipateAssetIssueContract
*参与 TRC-10 代币 ICO*

```
message ParticipateAssetIssueContract {
   bytes owner_address = 1;
   bytes to_address = 2;
   bytes asset_name = 3;
   int64 amount = 4;
}
```

* `owner_address`: 购买者地址。
* `to_address`: 代币发行者地址。
* `asset_name`: 代币 ID。
* `amount`: 用于购买代币的 TRX 数量，单位为 **sun**。

### UnfreezeAssetContract
*解除已发布的代币质押*

```
message UnfreezeAssetContract {
   bytes owner_address = 1;
}
```

* `owner_address`: 代币发行者地址。

### UpdateAssetContract
*更新通证参数*

```
message UpdateAssetContract {
   bytes owner_address = 1;
   bytes description = 2;
   bytes url = 3;
   int64 new_limit = 4;
   int64 new_public_limit = 5;
}
```

* `owner_address`: 通证发行者地址。
* `description`: 通证的新描述。
* `url`: 通证的新网址。
* `new_limit`: 单个调用者可消耗的新带宽上限。
* `new_public_limit`: 所有调用者可消耗的新公共带宽上限。



## 4. 超级代表与投票合约

### VoteWitnessContract
*为超级代表候选人投票*

```
message VoteWitnessContract {
   message Vote {
     bytes vote_address = 1;
     int64 vote_count = 2;
   }
   bytes owner_address = 1;
   repeated Vote votes = 2;
   bool support = 3;
}
```

* `owner_address`: 投票人地址。
* `votes`: 投票列表。
    * `vote_address`: 超级代表候选人地址。
    * `vote_count`: 投给该候选人的票数。
* `support`: 是否支持。该参数目前恒为 `true`。

### WitnessCreateContract
*创建超级代表候选人*

```
message WitnessCreateContract {
   bytes owner_address = 1;
   bytes url = 2;
}
```

* `owner_address`: 候选人地址。
* `url`: 候选人网站地址。

### WitnessUpdateContract
*更新超级代表候选人网址*

```
message WitnessUpdateContract {
   bytes owner_address = 1;
   bytes update_url = 12;
}
```

* `owner_address`: 候选人地址。
* `update_url`: 新的网站地址。



## 5. 提议与治理合约

### ProposalCreateContract
*创建提议（Proposal）*

```
message ProposalCreateContract {
   bytes owner_address = 1;
   map<int64, int64> parameters = 2;
}
```

* `owner_address`: Proposal 发起人地址。
* `parameters`: Proposal 内容，以键值对形式表示。

### ProposalApproveContract
*为 Proposal 投票。投票为赞同，不投即为不赞同。*

```
message ProposalApproveContract {
   bytes owner_address = 1;
   int64 proposal_id = 2;
   bool is_add_approval = 3; // add or remove approval
}
```

* `owner_address`: 投票人地址。
* `proposal_id`: Proposal ID。
* `is_add_approval`: 是否赞成 Proposal。`true` 表示赞成，`false` 表示取消赞成。

### ProposalDeleteContract
*删除 Proposal*

```
message ProposalDeleteContract {
   bytes owner_address = 1;
   int64 proposal_id = 2;
}
```

* `owner_address`: Proposal 删除人地址。
* `proposal_id`: Proposal ID。



## 6. 智能合约管理合约

### CreateSmartContract
*创建智能合约*

```
message CreateSmartContract {
   bytes owner_address = 1;
   SmartContract new_contract = 2;
   int64 call_token_value = 3;
   int64 token_id = 4;
}
```

* `owner_address`: 合约发起人地址。
* `new_contract`: 智能合约的 ABI 和字节码等信息。
* `call_token_value`: 随合约部署转入的 TRC-10 代币数量。
* `token_id`: 转入的 TRC-10 代币 ID。

### TriggerSmartContract
*触发智能合约*

```
message TriggerSmartContract {
   bytes owner_address = 1;
   bytes contract_address = 2;
   int64 call_value = 3;
   bytes data = 4;
   int64 call_token_value = 5;
   int64 token_id = 6;
}
```

* `owner_address`: 合约发起人地址。
* `contract_address`: 目标合约地址。
* `call_value`: 随合约调用转入的 TRX 数量，单位为 **sun**。
* `data`: 合约方法的编码参数。
* `call_token_value`: 随合约调用转入的 TRC-10 代币数量。
* `token_id`: 转入的 TRC-10 代币 ID。

### UpdateSettingContract
*更新智能合约的资源消耗百分比*

```
message UpdateSettingContract {
   bytes owner_address = 1;
   bytes contract_address = 2;
   int64 consume_user_resource_percent = 3;
}
```

* `owner_address`: 合约发起人地址。
* `contract_address`: 目标合约地址。
* `consume_user_resource_percent`: 更新后的用户资源消耗百分比。

### UpdateEnergyLimitContract
*调整智能合约的能量上限*

```
message UpdateEnergyLimitContract {
   bytes owner_address = 1;
   bytes contract_address = 2;
   int64 origin_energy_limit = 3;
}
```

* `owner_address`: 合约发起人地址。
* `contract_address`: 目标合约地址。
* `origin_energy_limit`: 调整后合约部署者提供的能量上限值。

### ClearABIContract
*清除智能合约的 ABI*

```
message ClearABIContract {
   bytes owner_address = 1;
   bytes contract_address = 2;
}
```

* `owner_address`: 合约发起人地址。
* `contract_address`: 待清除 ABI 的合约地址。

### UpdateBrokerageContract
*更新超级代表分红比例*

```
message UpdateBrokerageContract {
   bytes owner_address = 1;
   int32 brokerage = 2;
}
```

* `owner_address`: 超级代表地址。
* `brokerage`: 新的分红比例，范围从 0 到 100，表示百分比。


## 7. 账户权限管理

如需了解账户权限管理的详细信息，请参考 [账户权限管理](https://tronprotocol.github.io/documentation-zh/mechanism-algorithm/multi-signatures/)。
