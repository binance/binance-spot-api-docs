# 更新日志 (2022-12-28)

## 2022-12-28

* 现货 WebSocket API 文档已更新，添加了如何使用 RSA key 签署请求。

## 2022-12-26

* 现货的 Websocket API 发布到生产系统中。
* 现货的 Websocket API 可以通过URL: `wss://ws-api.binance.com/ws-api/v3` 来访问。

---

## 2022-12-15

* 添加新的RSA签名验证方式
    * 文档已更新以显示如何创建 RSA keys。
    * 建议在生成 API key 时使用 RSA keys。
    * 我们接受`PKCS#8`（BEGIN PUBLIC KEY）。
    * 稍后将添加有关如何上传 RSA public key 的更多详细信息。
* 现货 WebSocket API 现在可以在 SPOT 测试网上使用。
    * WebSocket API 允许通过 WebSocket 连接下订单、取消订单等。
    * WebSocket API 是一个 **独立** 于 WebSocket 市场数据流的服务。 即，下订单和收听市场数据需要两个独立的 WebSocket 连接。
    * WebSocket API 与 REST API 相同过滤器和速率限制规则。
    * WebSocket API 与 REST API 提供相同的功能，接受相同的参数，返回相同的状态和错误代码。

**WEBSOCKET API 会晚些时候在生产系统中可用。**

## 2022-12-13

REST API

错误代码 `-1003` 的一些错误消息已更改
* 之前错误消息: `Too much request weight used; current limit is %s request weight per %s %s. Please use the websocket for live updates to avoid polling the API.` 改成了：
```
Too much request weight used; current limit is %s request weight per %s. Please use WebSocket Streams for live updates to avoid polling the API.
```
* 之前错误消息: `Way too much request weight used; IP banned until %s. Please use the websocket for live updates to avoid bans.` 改成了：
```
Way too much request weight used; IP banned until %s Please use WebSocket Streams for live updates to avoid bans.
```


## 2022-12-05

**备注：** 这些更新正在逐步部署到我们所有的服务器，大约需要一周时间才能完成。

WEBSOCKET

* `!bookTicker` 在 **2022-12-07** 下线。 请改用按 symbol 的最优挂单信息的数据流（`<symbol>@bookTicker`）。
    * 可以通过一个连接订阅多个 `<symbol>@bookTicker` 数据流。 （例如`wss://stream.binance.com:9443/stream?streams=btcusdt@bookTicker/bnbbtc@bookTicker`）

REST API

* 新的错误代码 `-1135`
    * 如果参数是无效的 JSON 格式，则会出现此错误代码。
* 新的错误代码 `-1108`
    * 如果发送的参数的值太长，可能会导致溢出，则会发生此错误。
    * 此错误代码可能出现在以下接口：
        * `POST /api/v3/order`
        * `POST /api/v3/order/cancelReplace`
        * `POST /api/v3/order/oco`
* `GET /api/v3/aggTrades` 更新
    * 之前的规则: `startTime` 和 `endTime` 必须结合使用，并且只能相隔一个小时。
    * 新的规则: `startTime` 和 `endTime` 可以单独使用，一个小时的限制已被取消。
        * 仅使用 startTime 时，如果limit的值为N, 将返回从此时间开始的N条交易。
        * 仅使用 endTime 时，如果limit的值为N, 将返回到此时间的N条交易.
        * 如果不提供 `limit`，无论是组合使用还是单独发送，服务器端都将使用默认的 `limit`。
* `GET /api/v3/myTrades` 更新
    * 修复了 `symbol` + `orderId` 组合会返回所有交易，可能会超过`LIMIT`的默认值`500`。
    * 之前的行为： API 将根据发送的参数组合发送特定的错误消息。 例如：

        ```json
            {
                "code": -1106,
                "msg": "Parameter X was sent when not required."
            }
        ```

    * 新的行为: 如果接口不支持可选参数组合，那么服务器会返回一般性的错误:

        ```json
            {
                "code": -1128,
                "msg": "Combination of optional parameters invalid."
            }
        ```
    * 添加一个新的参数组合: `symbol` + `orderId` + `fromId`.
    * 下面的参数组合不再支持:
        * `symbol` + `fromId` + `startTime`
        * `symbol` + `fromId` + `endTime`
        * `symbol` + `fromId` + `startTime` + `endTime`
    * 当前支持的所有参数组合：
        * `symbol`
        * `symbol` + `orderId`
        * `symbol` + `startTime`
        * `symbol` + `endTime`
        * `symbol` + `fromId`
        * `symbol` + `startTime` + `endTime`
        * `symbol`+ `orderId` + `fromId`

**备注：** 这些新字段将在发布日期后大约一周出现。

* `GET /api/v3/exchangeInfo` 更新
    * 新字段 `defaultSelfTradePreventionMode` 和 `allowedSelfTradePreventionModes`
* 下单，查询订单和撤销订单接口的更新:
    * 响应中会出现新的字段 `selfTradePreventionMode`。
    * 以下接口会受到影响:
        * `POST /api/v3/order`
        * `POST /api/v3/order/oco`
        * `POST /api/v3/order/cancelReplace`
        * `GET /api/v3/order`
        * `DELETE /api/v3/order`
        * `DELETE /api/v3/orderList`
* `GET /api/v3/account` 更新
    * 响应中会出现新的字段 `requireSelfTradePrevention`.
* 以下接口的响应中会出现新字段 `workingTime`（指示订单何时添加到了订单薄）：
    * `POST /api/v3/order`
    * `GET /api/v3/order`
    * `POST /api/v3/order/cancelReplace`
    * `POST /api/v3/order/oco`
    * `GET /api/v3/order`
    * `GET /api/v3/openOrders`
    * `GET /api/v3/allOrders`
* 如果`trailingDelta`作为参数提供给了`TAKE_PROFIT`，`TAKE_PROFIT_LIMIT`，`STOP_LOSS`或 `STOP_LOSS_LIMIT`的订单，那么下面接口中会出现`trailingTime`, 用来表示追踪单被激活和跟踪价格变化的时间: 
    * `POST /api/v3/order`
    * `GET /api/v3/order`
    * `GET /api/v3/openOrders`
    * `GET /api/v3/allOrders`
    * `POST /api/v3/order/cancelReplace`
    * `DELETE /api/v3/order`
* 字段 `commissionRates` 会在 `GET /api/v3/acccount` 的响应中出现。


USER DATA STREAM

* eventType `executionReport` 有新的字段
    * `V` - `selfTradePreventionMode` 
    * `D` - `trailing_time`  (追踪单被激活时会出现)
    * `W` - `workingTime`   (如果 `isWorking`=`true` 会出现)


---

## 2022-12-02

* 新增一个用于访问市场信息的RESTful API URL: `https://data.binance.com`.
* 新增一个用于访问市场信息的WebSocket URL: `wss://data-stream.binance.com`.

---

## 2022-09-30


`!bookTicker`的WebSocket推送的变更.

* 全市场最优挂单信息推送(`!bookTicker`)计划在**2022年11月**下线, 具体下线的时间会在后面通告.
* 请使用按Symbol的最优挂单信息推送(`<symbol>@bookTicker`).
* 多个 `<symbol>@bookTicker` 可以订阅在一个WebSocket连接上.
    * 比如 `wss://stream.binance.com:9443/stream?streams=btcusdt@bookTicker/bnbbtc@bookTicker`

___


## 2022-09-15

这些变动会是滚动发布，可能需要几天才会部署到所有服务器.

* 接口 `GET /api/v3/exchangeInfo` 的变动
    * 添加一个新的参数 `permissions` , 用于查询适用于相应权限的所有交易对.
    * 如果查询时不提供此参数, 则默认值是 `["SPOT","MARGIN","LEVERAGED"]`.
        * 这表示如果请求 `GET /api/v3/exchangeInfo` 时候没有任何参数, 则会返回拥有权限是 `SPOT`, `MARGIN` , `LEVERAGED` 的交易对.
        * 如果要查询其他交易权限, 比如`TRD_GRP_004`等, 需要在查询参数里设置(比如`permissions`=`TRD_GRP_004`).
    * 此参数不可以同时和 `symbol` 或者 `symbols` 使用.


---

## 2022-08-23

此变动会滚动发布, 可能需要一段时间更新到所有服务器上。

* 接口 `GET /api/v3/ticker` 与 `GET /api/v3/ticker/24hr` 变动
    * 添加新可选参数 `type`
    * `type` 可接受的参数值有 `FULL` 与 `MINI`
        * `FULL` 是默认值， 也是原来接口所返回的响应
        * `MINI` 省略了以下字段: `priceChangePercent`, `weightedAvgPrice`, `bidPrice`, `bidQty`, `askPrice`, `askQty` 与 `lastQty`
* 添加新错误代码 `-1008`
    * 每当服务器的请求超载时都会发送此消息
    * 此错误代码只会在 SPOT API 里出现
* 接口 `GET /api/v3/account` 添加新参数 `brokered`
* 添加新接口: `GET /api/v3/uiKlines`
* 添加新k线间隔: `1s`


---

## 2022-08-08

REST API

* 接口 `POST /api/v3/order` 与 `POST /api/v3/order/cancelReplace` 变动
    * 添加新可选参数 `strategyId` 与 `strategyType`
        * `strategyId` 是用于将订单标识为某策略的参数。 
        * `strategyType` 是用于标识在执行的策略。(例如：如果所有订单属于现货网格策略，订单可设置为`strategyType=1000000`)
* 接口 `POST /api/v3/order/oco` 变动
    * 添加新可选参数  `limitStrategyId`, `limitStrategyType`, `stopStrategyId`, `stopStrategyType`
    * 这些是OCO订单里两个leg的策略元数据
    * `limitStrategyType` 和 `stopStrategyType` 都不能低于 `1000000`
* 接口 `GET /api/v3/order`, `GET /api/v3/openOrders` 与 `GET /api/v3/allOrders` 变动
    * 新增参数 `strategyId` 与 `strategyType` 必须在下单时填上字段才会在回应JSON里返回
* 接口 `DELETE /api/v3/order` 与 `DELETE /api/v3/openOrders` 变动
    * 新增参数 `strategyId` 与 `strategyType` 必须在下单时填上字段才会在回应JSON里返回


USER DATA STREAM

* eventType `executionReport` 新增参数
    * `j` 代表 `strategyId`
    * `J` 代表 `strategyType`
    * 必须在下单时填上字段才会在回应里返回

---

## 2022-06-20

接口 `GET /api/v3/ticker` 变动

* 权重从每`symbol` 5 降低到 2.
* 每次请求最多可以有100个交易对.
    * 如果`symbols`请求超过100个交易对, 会收到如下错误信息:
    ```json
    {
     "code": -1101,
     "msg": "Too many values sent for parameter 'symbols', maximum allowed up to 100." 
    }
    ```
* 单请求的权重上限为100.
    * 比如，如果请求的交易对超过50个，请求的权重是100.

---

## 2022-06-15

**注意:** 此变动不会立刻可用, 会在后面几天上线。


SPOT API

* 添加新接口 `GET /api/v3/ticker` 
    * 基于 `windowSize` 返回最近的价格变动。
    * 无需像 `GET /api/v3/ticker/24hr` 提供symbols参数。
    * 如果不提供 `windowSize` 参数，默认值是`1d`。
    * 响应和 `GET /api/v3/ticker/24hr` 相似，但不包括以下数据：`prevClosePrice`, `lastQty`, `bidPrice`, `bidQty`, `askPrice`, `askQty`
* 添加新接口 `POST /api/v3/order/cancelReplace`
    * 撤消当前的挂单并在同样的交易对上下新订单。
    * 过滤器会在**撤单前**做判断。
        * 例如，`MAX_NUM_ORDERS` 是 10，如果目前挂单也是10，调用 `POST /api/v3/order/cancelReplace`会失败。撤单与下单的操作都不会被执行。 
    * 更新将在几天后上线，升级完毕后才会开启此功能。
* `GET /api/v3/exchangeInfo` 在`symbols`列表里返回新数据`cancelReplaceAllowed`。
* 添加新的过滤器 `NOTIONAL`
    * 基于`minNotional` 与 `maxNotional` 值来限制名义价值 (`price * quantity`)
* 添加新的过滤器 `EXCHANGE_MAX_NUM_ICEBERG_ORDERS`
    * 账号最大冰山挂单数

WEBSOCKETS

* 新的symbol ticker流, 可以选择 `1h` 或者 `4h`时间窗口：
    * 单个交易对: `<symbol>@ticker_<window-size>`
    * 市场所有交易对: `!ticker_<window-size>@arr`


---


## 2022-05-23
* Order Book 深度的变动
    * 之前深度的数量在一些极端情况下会出现负数.
    * 之后深度数量不会溢出, 而是限制在64位的最大值, 这表示深度的数量达到，或者超过了最大值. 最大值和交易对的`base asset`的精度有关. 比如如果精度是8位小数，最大值则为92,233,720,368.54775807.
    * 原有的深度价位, 在修复上线后, 需要价位上有变动, 才能体现新的修复.
* 哪里有影响?
    * 现货深度接口
        * `GET /api/v3/depth`
    * Websocket Streams
        * `<symbol>@depth`
        * `<symbol>@depth@100ms`
        * `<symbol>@depth<levels>`
        * `<symbol>@depth<levels>@100ms`

* `MAX_POSITION` 的更新
    * 如果一个订单的数量(`quantity`) 可能导致持有仓位溢出, 会触发过滤器 `MAX_POSITION`.

---


* `GET api/v3/aggTrades` 更新
    * 如果同时提供 `startTime` 和 `endTime`, 旧的记录会返回.
* 如果接口 `GET /api/v3/myTrades` 中没有提供参数 `symbol`, 错误消息变为:

```json
{
"code": -1102,
"msg": "Mandatory parameter 'symbol' was not sent, was empty/null, or malformed." 
}
```
* 下面的接口提供参数 `symbols` 用于查询多个symbol.
    * `GET /api/v3/ticker/24hr`
    * `GET /api/v3/ticker/price`
    * `GET /api/v3/ticker/bookTicker`

* 上面接口的权重取决于请求 `symbols` 的数量, 具体请看下面的列表:

|接口|Symbols的数量|权重|
|-----|-----|----|
| `GET /api/v3/ticker/price`|Any| 2|
|`GET /api/v3/ticker/bookTicker`|Any|2|
|`GET /api/v3/ticker/24hr`|1-20|1|
|`GET /api/v3/ticker/24hr`|21-100|20|
|`GET /api/v3/ticker/24hr`| >= 101|40|



---

## 2022-04-13

REST API

* 现货交易支持追踪止损(Trailing Stop)订单.
    * 追踪止损通过一个新的参数`trailingDelta`来设置基于市场价的一个自动触发价格.
    * 只适用于订单类型: `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`.
    * 参数`trailingDelta`的单位为基点(BIPS).
        * 比如一个`STOP_LOSS`卖单设置`trailingDelta`为100, 那么订单会在当前市场价格从下单后的最高点下降1%的时候被触发。(100 / 10,000 => 0.01 => 1%)
    * 用于OCO订单的时候, 如果市场变动触发了`STOP_LOSS`订单, 那么此止损订单变成追踪止损订单.
    * 当参数`trailingDelta`和`stopPrice`一起使用时, 一旦`stopPrice`条件被触发，系统会开始追踪当前的价格变动. 从`stopPrice`价格开始，到基于`trailingDelta`值之间变动.
    * 如果没有提供`stopPrice`, 系统开始追踪价格从最新价到基于`trailingDelta`值之间变动.
* `POST /api/v3/order` 变动
    * 添加新可选参数 `trailingDelta`
* `POST /api/v3/order/test` 变动
    * 添加新可选参数 `trailingDelta`
* `POST /api/v3/order/oco` 变动
    * 添加新可选参数 `trailingDelta`
* 添加新的过滤器 `TRAILING_DELTA`
    * 用于限定 `trailingDelta` 的最大和最小值.

USER DATA STREAM

* User Data Stream 的`executionReport`添加新参数
  * "d" 代表`trailingDelta`

## 2022-04-12

**Note:** 下面的变更会在后面几天上线.


* `GET api/v3/allOrders` 如果没有提供 `symbol`, 则返回错误信息:
    ```json
    {
     "code": -1102,
     "msg": "Mandatory parameter 'symbol' was not sent, was empty/null, or malformed."
    }
    ```
* 修复一个错误信息中的拼写错误。 如果账号被禁用了相应的权限(比如提款，交易等), 则服务器返回错误:
    ```json
    "This action is disabled on this account."
    ```
* 在市场数据(market data)审计中，发现了一些现货的聚合交易数据(aggTrades)中的问题.
    * 丢失的记录已经被补回.
    * 重复的记录被标记成无效，具体的值设置成如下:
        * p = '0' // price
        * q = '0' // qty
        * f = -1 // ﬁrst_trade_id
        * l = -1 // last_trade_id

## 2022-02-28

* 在接口`GET /api/v3/exchangeInfo`中添加新字段`allowTrailingStop`.


## 2022-02-24

* 现货规则`PRICE_FILTER`里面的 `(price-minPrice) % tickSize == 0` 改成 `price % tickSize == 0`
* 新添加了一个规则 `PERCENT_PRICE_BY_SIDE`.
* 接口 `GET api/v3/depth` 的变动:
    * `limit` 原先必须是固定值(比如 5, 10, 20, 50, 100, 500, 1000, 5000), 现在可以是在1-5000之间的任意的正整数, 服务器会返回指定的limit数量。(比如如果设置limit=3, 会返回前3个最好的卖价和买价)
    * 如果`limit`超过5000, 服务器也最多返回5000条记录.
    * 相应的, 此接口的权重变成:

|Limit|Request Weight
------|-------
1-100|  1
101-500| 5
501-1000| 10
1001-5000| 50

* GET `api/v3/aggTrades` 接口的变动:
    * 当同时提供参数 `startTime` 和 `endTime`, 最旧的订单会优先返回.

---


# 2021-12-29
* 移除交易对类型枚举
* 新增权限枚举

## 2021-11-01
* 新增接口 `GET /api/v3/rateLimit/order`
    * 回传用户在当前时间区间内的下单总数
    * 此接口的权重为20

## 2021-09-14
* 添加一个基于OpenAPI规范的RESTful API接口定义的[YAML文件](https://github.com/binance/binance-api-swagger)

## 2021-08-12
* GET `api/v3/myTrades` 添加新的参数 `orderId`


## 2021-05-12
* 在文档中添加接口的数据来源说明
* 在每个接口中添加相应的数据源
* GET `api/v3/exchangeInfo` 现在支持单个或者多个交易对查询

## 2021-04-26

从 **April 28, 2021 00:00 UTC** 开始,下面接口的权重有如下变动:

* `GET /api/v3/order` 权重改为 2
* `GET /api/v3/openOrders` 权重改为 3
* `GET /api/v3/allOrders` 权重改为 10
* `GET /api/v3/orderList` 权重改为 2
* `GET /api/v3/openOrderList` 权重改为 3
* `GET /api/v3/account` 权重改为 10
* `GET /api/v3/myTrades` 权重改为 10
* `GET /api/v3/exchangeInfo` 权重改为 10

## 2021-01-01

**USER DATA STREAM**

* 移除`outboundAccountInfo`事件.

---

## 2020-11-27

为了优化性能，除了当前的`api.binance.com`，新加了一些API的集群。如果访问`api.binance.com`有性能问题，也可以尝试访问:

* https://api1.binance.com/api/v3/*
* https://api2.binance.com/api/v3/*
* https://api3.binance.com/api/v3/*

## 2020-09-09

用户数据 STREAM

* `outboundAccountInfo`事件不再推荐使用。
* `outboundAccountInfo`事件以后会被删除(具体时间未定) **请使用 `outboundAccountPosition` 事件.**
* `outboundAccountInfo`只推送余额不为0，以及余额刚变成0的资产。

---

## 2020-05-01
* 从2020-05-01 UTC 00:00开始, 所有交易对都会有最多200个挂单的限制, 体现在过滤器[MAX_NUM_ORDERS](./rest-api_CN.md#max_num_orders-%E6%9C%80%E5%A4%9A%E8%AE%A2%E5%8D%95%E6%95%B0)上.
  * 已经存在的挂单不会被移除或者撤销。
  * 单交易对(`symbol`)的挂单数量达到或超过200的账号, 无法在此交易对上下新的订单, 除非挂单数量低于200。
  * OCO订单在被触发成`LIMIT`订单, 或者被触发成`STOP_LOSS`(或者`STOP_LOSS_LIMIT`)前, 被认为是2个挂单量. 一旦OCO订单被触发, 就只被算作一个挂单。

---

## 2020-04-23

WEB SOCKET 连接限制

* Websocket服务器每秒最多接受5个消息。消息包括:
	* PING帧
	* PONG帧
	* JSON格式的消息, 比如订阅, 断开订阅.
* 如果用户发送的消息超过限制，连接会被断开连接。反复被断开连接的IP有可能被服务器屏蔽。
* 单个连接最多可以订阅 **1024** 个Streams。

---
## 2020-03-24

* 添加过滤器 `MAX_POSITION`.
    * 这个过滤器定义账户允许的基于`base asset`的最大仓位。一个用户的仓位可以定义为如下资产的总和:
        * `base asset`的可用余额
        * `base asset`的锁定余额
        * 所有处于open的买单的数量总和

    * 如果用户的仓位大于最大的允许仓位，买单会被拒绝。

---

## 2018-11-13
### Rest API
  * 账户交易权限被禁时允许进行撤单操作。
  * 增加了新的过滤器: `PERCENT_PRICE`, `MARKET_LOT_SIZE`, `MAX_NUM_ICEBERG_ORDERS`。
  * 增加了 `RAW_REQUESTS` 频率限制. 该限制仅统计请求次数，不统计请求权重。
  * /api/v3/ticker/price 无symbol参数时，权重增加到2。
  * /api/v3/ticker/bookTicker 无symbol参数时，权重增加到2。
  * DELETE /api/v3/order 现在会返回订单撤销前所处的末次状态。
  * `MIN_NOTIONAL` 新增两个参数: `applyToMarket` (是否对市价单生效) and `avgPriceMins` (对市价单生效时，估算金额时使用过去几分钟的平均价格?). 
  *  /api/v1/exchangeInfo 中的限制增加了`intervalNum`. `intervalNum`表示该限制针对多少时间间隔进行统计. 例如: `intervalNum`= 5, `interval` = minute, 表示该限制对每5分钟内的行为进行统计。
  
#### 如何计算过去n分钟平均价格:
  1. [对过去n分钟所有订单的数量\*价格求和] / 过去n分钟所有订单的数量
  2. 如果过去n分钟没有交易发生，则继续向前追溯，直到找到第一个交易，以此价格作为过去n分钟平均价格。
  3. 如果该交易对之前从未发生过交易，则无平均价格，亦即无法在该交易对下市价单，必须至少有一个（双方均未限价单）的交易成交后才可以下市价单。
  4. 当前系统使用的平均价格可以通过接口 `https://api.binance.com/api/v3/avgPrice?symbol=<symbol>`查询
     例如
     https://api.binance.com/api/v3/avgPrice?symbol=BNBUSDT

### User data stream
  * 成交报告中增加了 `末次成交金额` (`Y`)，等于 `末次成交量` * `末次成交价格` (`L` * `l`).

## 2018-07-18
### Rest API
  *  新过滤器: `ICEBERG_PARTS`
  *  `POST api/v3/order` 中 `newOrderRespType` 参数的缺省值更改; `MARKET`  `LIMIT` 默认为 `FULL`, 其他默认为 `ACK`.
  *  POST api/v3/order `RESULT` 与 `FULL` 响应中增加 `cummulativeQuoteQty`
  *  GET api/v3/openOrders 无symbol调用权重下降为 40.
  *  GET api/v3/ticker/24hr 无symbol调用权重下降为 40.
  *  GET /api/v1/trades amount最大可取到1000.
  *  GET /api/v1/historicalTrades amount最大可取到1000.
  *  GET /api/v1/aggTrades amount最大可取到1000.
  *  GET /api/v1/klines amount最大可取到1000.
  *  订单查询结果返回中增加 `updateTime` 字段，代表该订单末次更新(创建、成交、过期、取消、拒绝等等)时间; `time` 仅表示创建时间.
  *  订单查询结果中增加 `cummulativeQuoteQty`字段. 负值表示尚无任何成交，该字段不可用.
  *  `REQUESTS` 限制更名为 `REQUEST_WEIGHT`. 避免名字造成的误解。

### User data stream
  *  订单报告与成交报告中增加`cummulativeQuoteQty` 字段 (`Z`). 表示已经成交的金额， 即已经花费的金额(买入订单)或已经收到的金额(卖出订单)，均未计算手续费. 此功能增加之前成交的订单在历史订单接口中查询到的该字段可能小于零. 
  *  `cummulativeQuoteQty`/`cummulativeQty` 可以用来计算该订单已经成交部分的平均价格。
  *  成交报告中增加了 `O`字段 (订单创建时间) 


## 2018-01-23
  * GET /api/v1/historicalTrades权重降为 5
  * GET /api/v1/aggTrades 权重降为 1
  * GET /api/v1/klines 权重降为 1
  * GET /api/v1/ticker/24hr 不带symbol参数的权重降为 symbols总数 / 2
  * GET /api/v3/allOrders 权重降为 5
  * GET /api/v3/myTrades 权重降为 5
  * GET /api/v3/account 权重降为 5
  * GET /api/v1/depth limit=500 权重降为 5
  * GET /api/v1/depth limit=1000 权重降为 10
  * websocket 用户增加 -1003 error code

## 2018-01-20
  * GET /api/v1/ticker/24hr 单symbol参数调用权重降为 1
  * GET /api/v3/openOrders 不带symbol参数的权重降为 symbols总数 / 2
  * GET /api/v3/allOrders  权重降为  15
  * GET /api/v3/myTrades  权重降为  15
  * GET /api/v3/order  权重降为  1
  * 自成交现在会在myTrades结果中有两条记录。

## 2018-01-14
  * GET /api/v1/aggTrades 权重改为 2
  * GET /api/v1/klines 权重改为 2
  * GET /api/v3/order 权重改为 2
  * GET /api/v3/allOrders 权重改为 20
  * GET /api/v3/account 权重改为 20
  * GET /api/v3/myTrades 权重改为 20
  * GET /api/v3/historicalTrades 权重改为 20
