# 现货交易API术语表

**声明:** 此术语表只适用现货交易(`SPOT`); 用于合约、期权或者其他币安API相应的术语可能有不一样的表达。

## A

`ACK`
* `newOrderRespType`的枚举值, 设置时下单的返回值只包括下面的字段: `symbol`, `orderId`, `orderListId`, `clientOrderId`, 和 `transactTime`。

`aggTrade`
* 归集交易信息; 此交易信息归集了在同一个时间同一个 `taker` 的订单生成的相同价格的交易信息。

allocation
* 在这里，分配指的是将资产从交易所转移到个人账户的过程（e.g. 当一个订单通过 SOR 成交而不是直接交易）。

`allocationId`
* 此字段是一个唯一识别码，用来标识针对某个交易对上进行的分配(allocation)。

`allocationType`
* 参考 [ 分配类型 ](../rest-api_CN.md#allocationtype) 

`askPrice`
* `ticker` 请求返回的来自“卖"方的最低价格。

`askQty`
* `ticker` 请求返回的“卖"方以最低价格提供的总数量。

`asks`
* 卖单

`avgPrice`
* 表示相应N分钟之内的平均价格。

---

## B

`baseAsset`
* 基准资产; 指代交易对中的第一个资产(比如 `BTCUSDT`中的`BTC`), 表示被出售或者买进的资产。

`baseAssetPrecision`
* 基准资产精度; 接口 `GET /api/v3/exchangeInfo` 中的一个字段, 代表了基准资产(`baseAsset`)可以允许的最多小数位数。

`baseCommissionPrecision`
* 接口 `GET /api/v3/exchangeInfo` 中用来表示基准资产手续费可以允许的最多小数位数。

`bidPrice`
* `ticker` 请求返回的来自“买"方的最高价格。

`bidQty`
* `ticker` 请求返回的“买"方以最高价格提供的总数量。

`bids`
* 买单。

`BREAK`
* 交易对的一个交易状态, 用来表示某交易对无法交易。 处于此状态的交易对无法产生市场行情数据。

`BUY`
* `side` 的一个枚举值, 用来表示用户期望购买一个资产(比如 `BTC`)。

---

## C

`CANCELED`
* 订单的一个状态, 用来表示订单被用户取消。

`clientOrderId`
* 用于下单的接口 `POST /api/v3/order`, 用户可以用此字段来设置自定义值, 便于用来跟踪订单。

`commission`
* 交易费

`commissionAsset`
* 用于计算交易费的资产。

`cancelReplaceMode`
* 撤消挂单再下单接口(`POST /api/v3/order/cancelReplace`)的一个参数, 用来定义如果取消订单的请求失败之后，是否继续下新的订单。

`cummulativeQuoteQty`
* 订单的成交交易记录里面所有价格(`price`)乘以数量(`qty`)的和。

---

## D

Data Source
* 发送请求后得到数据的地方, 比如数据库, 缓存等。

---

## E

`executedQty`
* 订单中成交的数量。

`EXPIRED`
* 订单的一个状态, 用来表示订单因为交易规则而取消, 也可能是直接被交易所取消。

`EXPIRED_IN_MATCH`
* 订单的一个状态，用来表示订单由于 STP 而过期 （e.g. 带有 `EXPIRE_TAKER` 的订单与订单簿上属于同账户或同 `tradeGroupId` 的订单撮合）。

---

## F

`filters`
* 过滤器; 用于定义交易规则。

`FOK`/ Fill or Kill
* `timeInForce` 的枚举值, 用于下单时要求订单全部成交，不然就取消。

`free`
* 用户的可用余额, 可以用来交易或者提取的金额。

`FULL`
* `newOrderRespType`的枚举值, 设置在下单接口时, 请求会返回所有的交易信息, 包括了成交记录(`fills`)。

---

## G

`GTC`/ Good Til Canceled
* `timeInForce` 的枚举值, 表示订单会一直有效, 直到全部成交或者被取消。

---

## H

`HALT`
* 交易对的一个交易状态, 可以用来表示交易处于紧急暂停状态。 此时市场信息还会生成。

---

## I

`intervalNum`
* 表示间隔时间, 例如如果`interval`的值是`SECOND`, 并且`intervalNum`是5, 那么表示为每5秒钟间隔。

`IOC` / Immediate or Canceled
* `timeInForce` 的枚举值, 表示订单会尽量的成交，而不能成交的部分则会被交易所取消。

`isBestMatch`
* 表示交易的价格是不是当时的最优价。

`isBuyerMaker`
* 表示交易双方的买家是否是市场的做市商(`Maker`)。

`isWorking`
* 表示订单是否出现在订单薄上。

---

## K

`kline`
* K线; 包括了一定时期内的开盘价, 收盘价, 最高价，最低价，交易量，以及其他的市场数据。 通常也被成为蜡烛图。

---

## L

Last Prevented Quantity
* 最后被阻止交易的数量。这仅在订单因 STP 触发而过期时可见。

`lastPrice`
* 最新一笔交易的成交价格。

`lastQty`
* 最新一笔交易的成交量。

`LIMIT`
* 限价; 一种订单形式，其订单的成交价格会是指定价格，或者更好的价格。

`LIMIT_MAKER`
* 一种订单形式, 其订单会保证成为做市订单(`MAKER`), 不会立刻成交进而成为`TAKER`。

`limitClientOrderId`
* OCO订单下单接口(`POST /api/v3/order/oco`)的一个参数, 方便用户自定义ID来标记OCO里的`LIMIT_MAKER`订单。

`listClientOrderId`
* OCO订单下单接口(`POST /api/v3/order/oco`)的一个参数, 方便用户自定义ID来标记OCO订单。

`listenKey`
* 系统提供的一个Key, 以方便用户来获取WebSocket中的用户相关信息。

`locked`
* 表示用户的某个资产余额中当前锁定在挂单或者被其他系统占用的数量。

---

## M

`MARKET`
* 一个订单的类型; 其订单会在系统中尽可能的全部成交，除非市场没有流动性，无法成交部分会被交易取消。

Matching Engine
* 在数据源(`Data Source`)的部分指代的是请求获得数据的地方。
* 也可以指代的是处理所有请求，撮合所有订单的后台系统。

Memory
* 数据源(`Data Source`)中指代数据存储在系统内部的缓冲。

---

## N

`NEW`
* 一个订单的状态, 表示订单成功被发送到了交易引擎。

`newClientOrderId`
* 一个订单相关(下单，撤销订单, 等)请求中的参数; 在请求的返回的时候, 此值会被设置为`clientOrderId`;

Notional value
* 订单的名义价值, 值为`price` * `qty`。

---

## O

`OCO`
* 二选一订单(`One-Cancels-the-Other`); 订单支持用户同时提交一些列订单, 比如现价单(`LIMIT_MAKER`)和止盈止损订单(`STOP_LOSS` or `STOP_LOSS_LIMIT`)。 当执行其中一个订单时，另一个订单将自动取消。

Order Book
* 订单薄; 包括了当前市场上买卖挂单。

`orderId`
* 订单数据里用来唯一标识的ID。

`origQty`
* 发送订单请求中的原始数量。

`origClientOrderId`
* 在查询或者取消订单请求中, 用户设置在 `clientOrderId` 的值。

---

## P

`PARTIALLY_FILLED`
* 订单的一种状态, 表示订单被部分成交。

`preventedQuantity`
* 因为 STP 导致订单失效的数量。

`preventedMatchId`
* 与 `symbol` 结合使用时，可用于查询因为 STP 导致订单失效的过期订单。

---

## Q

`quantity`
* 订单量; 买卖订单时候基本资产(`base asset`)的数量。

`quoteAsset`
* 定价资产; 在交易对中的第二个资产, 比如交易对`BTCUSDT`中的`USDT`;

`quoteAssetPrecision`
* 接口 `GET /api/v3/exchangeInfo` 中用来指明`quoteAsset`允许的最多小数位数。

`quoteCommissionPrecision`
* 接口 `GET /api/v3/exchangeInfo` 中用来指明交易费用是`quoteAsset`允许的最多小数位数。

`quoteOrderQty`
* 市价单(`MARKET`)的下单接口中用于下反向市价单中的数量。

`quoteQty`
* 名义价值; 为订单中数量(`qty`)乘以价格(`price`)。

---

## R

`recvWindow`
* API中的一个参数, 值为毫秒; 用以设定请求从`timestamp`开始的有效期。

`RESULT`
* `newOrderRespType`的一个枚举值。 用于下单的接口，会返回除了成交部分(`fills`)的所有值。

Reverse `MARKET` order
* 反向市价单; 下市价单的时候使用`quoteOrderQty`, 而不是`quantity`。

---

## S

Self Trade Prevention (STP)
* 自我交易预防; 此功能能阻止订单与来自同一账户或者同一 `tradeGroupId` 下的账户的订单撮合交易。

`selfTradePreventionMode`
* 如果发生自我交易情况，此参数用来通知系统如何处理订单。

`SELL`
* 方向(`side`)的一个枚举值, 用于用户希望卖出某一资产。

Smart Order Routing (SOR)
* 智能订单路由; 使用可互换的定价资产(`quote asset`)来提高流动性. [请阅读 SOR 常见问题](./sor_faq_cn.md) 来了解更多详情。

`SPOT`
* 现货交易; 此种交易时候，买卖相应的资产会立刻到账。

`stopClientOrderId`
* 用于下OCO订单的接口(i.e. `POST /api/v3/order/oco`); 此ID可以用来标识OCO中 `STOP_LOSS` 或 `STOP_LOSS_LIMIT` 的订单。

`stopPrice`
* 用于设置逻辑订单(比如`STOP_LOSS`, `TAKE_PROFIT`)中的触发价; 此价格被触发后，订单会被放置到订单薄里面(`OrderBook`)。
* 用于设置追踪止盈止损订单中的触发价; 此价格被触发后, 订单会被开始追踪。

`STOP_LOSS`
* 止损单; 一种逻辑订单，市场价格达到`stopPrice`的时候, 此订单会以市价单(`MARKET`)的形式执行。

`STOP_LOSS_LIMIT`
* 限价止损单; 一种逻辑订单，市场价格达到`stopPrice`的时候, 此订单会以限价单(`LIMIT`)的形式执行。

`strategyId`
* 策略单ID; 用以关联此订单对应的交易策略。

`strategyType`
* 策略单类型; 用以显示此订单对应的交易策略。

`symbol`
* 交易对; 由交易对象(`base asset`)和定价资产(`quote asset`)组成。

---

## T

`TAKE_PROFIT`
* 止盈订单; 当市场价格触及`stopPrice`价, 此订单会以市价单(`MARKET`)被执行。

`TAKE_PROFIT_LIMIT`
* 限价止盈订单; 当市场价格触及`stopPrice`价, 此订单会以限价单(`LIMIT`)被执行。

`ticker`
* 用以汇报过去一段时间内的价格变动等市场信息。

`time`
* 对于交易/分配查询：交易/分配执行的时间。
* 订单查询：订单创建时间。

`timeInForce`
* 定义订单的时效性, 用以表明订单会在orderbook中的时长。
* 支持的值包括了:  `GTC`, `IOC`, 和 `FOK`。

`tradeGroupId`
* 属于同一个交易组的账户组。

`TRADING`
* 一种交易状态, 表明某交易对可以进行交易。

`trailingDelta`
* 用以定义追踪止盈止损订单被触发的价格差。

`trailingTime`
* 追踪单被激活和跟踪价格变化的时间。

`transactTime`
* 订单的更新时间： 下单， 成交或者取消。此字段（和所有时间戳有关的字段一样）的单位为毫秒++

---

## U

`uiKlines`
* 为了前端展示而优化的K线。

`updateTime`
* 订单的最后更新时间, 单位为毫秒。

User Data Stream
* 通过WebSocket推送及时的个人用户信息, 包括了账户余额的变动, 订单的更新等。

---

## W

`weightedAveragePrice`
* 成交量加权平均价; 将过去N分钟内所有交易的价格按各自的成交量加权而算出的平均价。

`workingFloor`
* 工作平台； 该字段用于定义订单是通过 SOR 还是由订单提交到的订单薄（order book）成交的。

`workingTime`
* 指示订单何时添加到了 order book。
---

## X

`X-MBX-ORDER-COUNT-XX`
* 请求的返回Header里面一个自定义值, 用来表明当前用户下单限制额的所剩额度。

`X-MBX-USED-WEIGHT-XX`
* 请求的返回Header里面一个自定义值, 用来表明当前IP在一定时间内所剩的请求额度。
