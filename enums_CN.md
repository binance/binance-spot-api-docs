# 枚举定义

这将适用于 REST API 和 WebSocket API。

## 交易对的状态（status）

* `TRADING` - 正常交易中
* `END_OF_DAY` - 收盘
* `HALT` - 交易终止(该交易对已下线)
* `BREAK` - 交易暂停

<a id="account-and-symbol-permissions"></a>

## 账户与交易对权限（permissions）

* `SPOT` - 现货
* `MARGIN` - 杠杆
* `LEVERAGED` - 杠杆代币
* `TRD_GRP_002` - 交易组 002
* `TRD_GRP_003` - 交易组 003
* `TRD_GRP_004` - 交易组 004
* `TRD_GRP_005` - 交易组 005
* `TRD_GRP_006` - 交易组 006
* `TRD_GRP_007` - 交易组 007
* `TRD_GRP_008` - 交易组 008
* `TRD_GRP_009` - 交易组 009
* `TRD_GRP_010` - 交易组 010
* `TRD_GRP_011` - 交易组 011
* `TRD_GRP_012` - 交易组 012
* `TRD_GRP_013` - 交易组 013
* `TRD_GRP_014` - 交易组 014
* `TRD_GRP_015` - 交易组 015
* `TRD_GRP_016` - 交易组 016
* `TRD_GRP_017` - 交易组 017
* `TRD_GRP_018` - 交易组 018
* `TRD_GRP_019` - 交易组 019
* `TRD_GRP_020` - 交易组 020
* `TRD_GRP_021` - 交易组 021
* `TRD_GRP_022` - 交易组 022
* `TRD_GRP_023` - 交易组 023
* `TRD_GRP_024` - 交易组 024
* `TRD_GRP_025` - 交易组 025

## 订单状态（status）

状态 |描述
-----------|--------------
`NEW` | 该订单被交易引擎接受。
`PENDING_NEW` | 该订单处于待处理 (`PENDING`) 阶段，直到其所属订单组（order list） 中的 `working order` 完全成交。
`PARTIALLY_FILLED` | 部分订单已被成交。
`FILLED`| 订单已完全成交。
`CANCELED` | 用户撤销了订单。
`PENDING_CANCEL` | 撤销中(目前并未使用)
`REJECTED`       | 订单没有被交易引擎接受，也没被处理。
`EXPIRED` | 该订单根据订单类型的规则被取消（例如，没有成交的 LIMIT FOK 订单, LIMIT IOC 或部分成交的 MARKET 订单）<br/>或者被交易引擎取消（例如，在强平期间被取消的订单，在交易所维护期间被取消的订单）
`EXPIRED_IN_MATCH` | 表示订单由于 STP 而过期。（例如，带有 `EXPIRE_TAKER` 的订单与账簿上同属相同帐户或相同 `tradeGroupId` 的现有订单匹配）

## 订单组（order list）状态 （状态类型集 listStatusType）

状态 |描述
-----------|--------------
`RESPONSE` | 在 ListStatus 用于响应失败的操作时会被使用。（例如，下订单组或取消订单组）
`EXEC_STARTED` | 订单组已被下达或订单组状态有更新。
`UPDATED`  | 订单组里的某个订单的 clientOrderId 被改变。
`ALL_DONE` | 订单组执行结束，因此不再处于活动状态。

## 订单组（order list）中的订单状态 （订单状态集 listOrderStatus）

状态 |描述
-----------|--------------
`EXECUTING` | 订单组已被下达或订单组状态有更新。
`ALL_DONE`| 订单组执行结束，因此不再处于活动状态。
`REJECT` | 在 ListStatus 用于响应在下单阶段或取消订单组期间的失败操作时会被使用，

## 订单组的类型

* `OCO`
* `OTO`

<a id="allocationtype"></a>

## 分配类型

* `SOR`

<a id="ordertypes"></a>

## 订单类型（orderTypes, type）

* `LIMIT` - 限价单
* `MARKET` - 市价单
* `STOP_LOSS` - 止损单
* `STOP_LOSS_LIMIT` - 限价止损单
* `TAKE_PROFIT` - 止盈单
* `TAKE_PROFIT_LIMIT` - 限价止盈单
* `LIMIT_MAKER` - 限价做市单

<a id="orderresponsetype"></a>

## 订单返回类型 （newOrderRespType）

* `ACK`
* `RESULT`
* `FULL`

## 工作平台

* `EXCHANGE` - 常规交易
* `SOR` - 智能订单路由

<a id="side"></a>

## 订单方向 (side)

* `BUY` - 买入
* `SELL` - 卖出

<a id="timeinforce"></a>

## 生效时间 （timeInForce）

这里定义了订单在失效前的有效时间。

状态 |描述
-----------|--------------
`GTC` | 成交为止 <br/> 订单会一直有效，直到被成交或者取消。
`IOC` | 无法立即成交的部分就撤销 <br/> 订单在失效前会尽量多的成交。
`FOK` | 无法全部立即成交就撤销 <br/> 如果无法全部成交，订单会失效。


## 速率限制种类（rateLimitType）

* REQUESTS_WEIGHT - 单位时间请求权重之和上限

```json
{
    "rateLimitType": "REQUEST_WEIGHT",
    "interval": "MINUTE",
    "intervalNum": 1,
    "limit": 6000
}
```

* ORDERS - 单位时间下单次数上限

```json
{
    "rateLimitType": "ORDERS",
    "interval": "SECOND",
    "intervalNum": 1,
    "limit": 10
}
```

* RAW_REQUESTS - 单位时间请求次数上限

```json
{
    "rateLimitType": "RAW_REQUESTS",
    "interval": "MINUTE",
    "intervalNum": 5,
    "limit": 61000
}
```

## 速率限制间隔 （interval）

* SECOND
* MINUTE
* DAY

<a id="stpmodes"></a>

## STP 模式

请参阅 [自我交易预防 (Self Trade Prevention - STP) 常见问题](faqs/stp_faq_CN.md)。

* `NONE`
* `EXPIRE_MAKER`
* `EXPIRE_TAKER`
* `EXPIRE_BOTH`
* `DECREMENT`
* `TRANSFER`
