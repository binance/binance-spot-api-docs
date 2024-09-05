# 自我交易预防 (Self Trade Prevention - STP) 常见问题

## 什么是 Self Trade Prevention - STP?

自我交易预防是指阻止订单与来自同一账户或者同一 `tradeGroupId` 账户的订单交易。

## 什么是自我交易（self-trade）?

在以下任一情况下都可能发生自我交易：

* 属于同一账户的订单之间交易。
* 属于相同 `tradeGroupId` 的账户的订单之间交易。

## STP 触发时会发生什么？

如果订单会触发自我交易，系统将执行四种可能的模式：

`NONE` - 此模式使订单免于自我交易预防。

`EXPIRE_TAKER` - 此模式通过立即使吃单者(taker)的剩余数量过期来预防交易。

`EXPIRE_MAKER` - 此模式通过立即使潜在挂单者(maker)的剩余数量过期来预防交易。

`EXPIRE_BOTH` - 此模式通过立即同时使吃单和挂单者的剩余数量过期来预防交易。

STP 的发生取决于 **Taker 订单** 的 STP 模式。 <br> 因此，订单薄上的订单的 STP 模式不再有效果，并且将在所有未来的订单处理中被忽略。

## 什么是交易组 Id（Trade Group Id）?

属于同一 `tradeGroupId` 的账户被视为同一交易组。相同交易组成员提交的订单有 STP 资格。

每个账户可以从 `GET /api/v3/account`（REST API）或 `account.status`（Websocket API）确认账户是否属于同一个 `tradeGroupId`。

`tradeGroupId` 也存在 `GET /api/v3/preventedMatches`（Rest API）或 `myPreventedMatches`（Websocket API）的响应中。

如果该值为 `-1`，这表示账户未设置 `tradeGroupId`，因此 STP 只能发生在同一账户的订单之间。

## 什么是 Prevented Match?

当一个或多个订单因 STP 而过期时，这会创建一个被阻止的撮合交易事务。

通过 Rest API 的 `GET /api/v3/preventedMatches` 或 Websocket API 的 `myPreventedMatches` 可以查询到有哪些被阻止的撮合交易。

请求的响应示例：

```javascript
[
  {
    "symbol": "BTCDUSDT",                       //交易对
    "preventedMatchId": 8,                      //被阻止撮合交易的Id
    "takerOrderId": 12,                         //吃单者的订单Id
    "makerOrderId": 10,                         //挂单者的订单Id
    "tradeGroupId": 1,                          //交易组的Id。（如果账户不属于交易组，则为 -1）
    "selfTradePreventionMode": "EXPIRE_BOTH",   //订单过期的 STP 模式。
    "price": "50.00000000",                     //撮合交易的价格。
    "takerPreventedQuantity": "1.00000000",     //吃单者的剩余数量。 仅在 STP 模式为 EXPIRE_TAKER 或 EXPIRE_BOTH 时出现。
    "makerPreventedQuantity": "10.00000000",    //挂单者的剩余数量。 仅在 STP 模式为 EXPIRE_MAKER 或 EXPIRE_BOTH 时出现。
    "transactTime": 1663190634060               //订单因 STP 而过期的时间。
  }
]
```

## 什么是 "prevented quantity"?

STP事件会导致挂单的数量失效; STP的模式 `EXPIRE_TAKER`, `EXPIRE_MAKER` 以及 `EXPIRE_BOTH` 会使挂单中剩余的数量全部失效，从而使整个订单失效。


`Prevented quantity` 表示订单中因为STP事件失效的数量. 用户WebSocket数据流中可能有如下两个字段:

```javascript
{
  "A":"3.000000", // Prevented Quantity
  "B":"3.000000"  // Last Prevented Quantity
}
```

`B` 代表着 `TRADE_PREVENTION` 交易类型, 其值表示本次STP事件导致失效的订单数量.

`A` 代表着某订单因为STP事件导致的累计失效订单数量. 对于 `EXPIRE_TAKER`, `EXPIRE_MAKER` 以及 `EXPIRE_BOTH` 模式, 其值总是和 `B` 一样.

由于 STP 而过期的订单的 API 响应也将有一个 `preventedQuantity` 字段，指示在订单由于 STP 而过期的累计数量。

如果订单是处于挂单状态, 如下的公式成立:

```
executed quantity + prevented quantity < original order quantity
执行的订单数量 + 被过期的数量 < 订单的原始数量
```

如果订单状态是 `EXPIRED_IN_MATCH` 或者 `FILLED`, 如下的等式成立:

```
executed quantity + prevented quantity = original order quantity
执行的订单数量 + 被过期的数量 = 订单的原始数量
```


## 如何知道有那些交易对支持 STP?

交易对可以配置为允许不同的 STP 模式集并采用不同的默认 STP 模式。

`defaultSelfTradePreventionMode` - 如果用户在下单时不提供，订单将使用此 STP 模式。

`allowedSelfTradePreventionModes` - 交易对允许的下单 STP 模式集。

例如，如果交易对有以下配置：

```json
"defaultSelfTradePreventionMode": "NONE",
"allowedSelfTradePreventionModes": [
    "NONE",
    "EXPIRE_TAKER",
    "EXPIRE_BOTH"
  ]
```

这表示如果用户在没有提供 `selfTradePreventionMode` 的情况下发送订单，发送的订单有 `NONE` 的值。

如果用户想明确提及模式，可以传 `NONE`，`EXPIRE_TAKER`，或 `EXPIRE_BOTH`。

如果用户尝试为此交易对的订单指定 `EXPIRE_MAKER`，将会收到错误消息：

```json
{
    "code": -1013,
    "msg": "This symbol does not allow the specified self-trade prevention mode."
}
```

## 如何知道订单因为 STP 而过期？

订单的状态会是 `EXPIRED_IN_MATCH`.

## STP 的一些示例:

假设以下示例的所有订单都是在同一个账户下发送。

**情况 A - 用户发送一个带有 selfTradePreventionMode:`NONE` 的新订单，该订单将与订单薄上已有的另一个订单撮合。**


```
Maker 订单: symbol=BTCUSDT side=BUY type=LIMIT quantity=1 price=1 selfTradePreventionMode=NONE
Taker 订单: symbol=BTCUSDT side=SELL type=LIMIT quantity=1 price=1 selfTradePreventionMode=NONE
```

**结果:** : 没有 STP 被触发，订单会撮合。

Maker 订单的状态

```json
{
  "symbol": "BTCUSDT",
  "orderId": 2,
  "orderListId": -1,
  "clientOrderId": "FaDk4LPRxastaICEFE9YTf",
  "price": "1.000000",
  "origQty": "1.000000",
  "executedQty": "1.000000",
  "cummulativeQuoteQty": "1.000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.000000",
  "icebergQty": "0.000000",
  "time": 1670217090310,
  "updateTime": 1670217090330,
  "isWorking": true,
  "workingTime": 1670217090310,
  "origQuoteOrderQty": "0.000000",
  "selfTradePreventionMode": "NONE"
}
```

Taker 订单的状态

```json
{
  "symbol": "BTCUSDT",
  "orderId": 3,
  "orderListId": -1,
  "clientOrderId": "Ay48Vtpghnsvy6w8RPQEde",
  "transactTime": 1670207731263,
  "price": "1.000000",
  "origQty": "1.000000",
  "executedQty": "1.000000",
  "cummulativeQuoteQty": "1.000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "SELL",
  "workingTime": 1670207731263,
  "fills": [
    {
      "price": "1.000000",
      "qty": "1.000000",
      "commission": "0.000000",
      "commissionAsset": "USDT",
      "tradeId": 1
    }
  ],
  "selfTradePreventionMode": "NONE"
}
```


**情况 B - 用户发送带有 `EXPIRE_MAKER` 的订单，该订单将与订单薄上已有的订单撮合。**

```
Maker 订单 1: symbol=BTCUSDT side=BUY type=LIMIT quantity=1.2 price=1.2 selfTradePreventionMode=NONE
Maker 订单 2: symbol=BTCUSDT side=BUY type=LIMIT quantity=1.3 price=1.1 selfTradePreventionMode=NONE
Maker 订单 3: symbol=BTCUSDT side=BUY type=LIMIT quantity=8.1 price=1   selfTradePreventionMode=NONE
Taker 订单 1: symbol=BTCUSDT side=SELL type=LIMIT quantity=3 price=1    selfTradePreventionMode=EXPIRE_MAKER
```

**结果:** : 由于 STP，订单薄上的订单将会过期，taker 订单将继续在订单薄。

Maker 订单 1
```json
{
  "symbol": "BTCUSDT",
  "orderId": 2,
  "orderListId": -1,
  "clientOrderId": "wpNzhSclc16pV8g5THIOR3",
  "price": "1.200000",
  "origQty": "1.200000",
  "executedQty": "0.000000",
  "cummulativeQuoteQty": "0.000000",
  "status": "EXPIRED_IN_MATCH",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.000000",
  "icebergQty": "0.000000",
  "time": 1670217957437,
  "updateTime": 1670217957498,
  "isWorking": true,
  "workingTime": 1670217957437,
  "origQuoteOrderQty": "0.000000",
  "selfTradePreventionMode": "NONE",
  "preventedMatchId": 0,
  "preventedQuantity": "1.200000"
}
```

Maker 订单 2

```json
{
  "symbol": "BTCUSDT",
  "orderId": 3,
  "orderListId": -1,
  "clientOrderId": "ZT9emqia99V7x8B6FW0pFF",
  "price": "1.100000",
  "origQty": "1.300000",
  "executedQty": "0.000000",
  "cummulativeQuoteQty": "0.000000",
  "status": "EXPIRED_IN_MATCH",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.000000",
  "icebergQty": "0.000000",
  "time": 1670217957458,
  "updateTime": 1670217957498,
  "isWorking": true,
  "workingTime": 1670217957458,
  "origQuoteOrderQty": "0.000000",
  "selfTradePreventionMode": "NONE",
  "preventedMatchId": 1,
  "preventedQuantity": "1.300000"
}
```

Maker 订单 3
```json
{
  "symbol": "BTCUSDT",
  "orderId": 4,
  "orderListId": -1,
  "clientOrderId": "8QZ3taGcU4gND59TxHAcR0",
  "price": "1.000000",
  "origQty": "8.100000",
  "executedQty": "0.000000",
  "cummulativeQuoteQty": "0.000000",
  "status": "EXPIRED_IN_MATCH",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.000000",
  "icebergQty": "0.000000",
  "time": 1670217957478,
  "updateTime": 1670217957498,
  "isWorking": true,
  "workingTime": 1670217957478,
  "origQuoteOrderQty": "0.000000",
  "selfTradePreventionMode": "NONE",
  "preventedMatchId": 2,
  "preventedQuantity": "8.100000"
}
```

Taker 订单的响应

```json
{
  "symbol": "BTCUSDT",
  "orderId": 5,
  "orderListId": -1,
  "clientOrderId": "WRzbhp257NhZsIJW4y2Nri",
  "transactTime": 1670217957498,
  "price": "1.000000",
  "origQty": "3.000000",
  "executedQty": "0.000000",
  "cummulativeQuoteQty": "0.000000",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "SELL",
  "workingTime": 1670217957498,
  "fills": [],
  "preventedMatches": [
    {
      "preventedMatchId": 0,
      "makerOrderId": 2,
      "price": "1.200000",
      "makerPreventedQuantity": "1.200000"
    },
    {
      "preventedMatchId": 1,
      "makerOrderId": 3,
      "price": "1.100000",
      "makerPreventedQuantity": "1.300000"
    },
    {
      "preventedMatchId": 2,
      "makerOrderId": 4,
      "price": "1.000000",
      "makerPreventedQuantity": "8.100000"
    }
  ],
  "selfTradePreventionMode": "EXPIRE_MAKER"
}
```


**情况 C - 用户发送带有 `EXPIRE_TAKER` 的订单，该订单将与订单薄上已有的订单撮合。**

```
Maker 订单 1: symbol=BTCUSDT side=BUY type=LIMIT quantity=1.2 price=1.2  selfTradePreventionMode=NONE
Maker 订单 2: symbol=BTCUSDT side=BUY type=LIMIT quantity=1.3 price=1.1  selfTradePreventionMode=NONE
Maker 订单 3: symbol=BTCUSDT side=BUY type=LIMIT quantity=8.1 price=1    selfTradePreventionMode=NONE
Taker 订单 1: symbol=BTCUSDT side=SELL type=LIMIT quantity=3 price=1 selfTradePreventionMode=EXPIRE_TAKER
```
**结果:** : 已经在订单薄上的订单将保留，而taker订单将过期。

Maker 订单 1
```json
{
  "symbol": "BTCUSDT",
  "orderId": 2,
  "orderListId": -1,
  "clientOrderId": "NpwW2t0L4AGQnCDeNjHIga",
  "price": "1.200000",
  "origQty": "1.200000",
  "executedQty": "0.000000",
  "cummulativeQuoteQty": "0.000000",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.000000",
  "icebergQty": "0.000000",
  "time": 1670219811986,
  "updateTime": 1670219811986,
  "isWorking": true,
  "workingTime": 1670219811986,
  "origQuoteOrderQty": "0.000000",
  "selfTradePreventionMode": "NONE"
}
```

Maker 订单 2

```json
{
  "symbol": "BTCUSDT",
  "orderId": 3,
  "orderListId": -1,
  "clientOrderId": "TSAmJqGWk4YTB2yA9p04UO",
  "price": "1.100000",
  "origQty": "1.300000",
  "executedQty": "0.000000",
  "cummulativeQuoteQty": "0.000000",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.000000",
  "icebergQty": "0.000000",
  "time": 1670219812007,
  "updateTime": 1670219812007,
  "isWorking": true,
  "workingTime": 1670219812007,
  "origQuoteOrderQty": "0.000000",
  "selfTradePreventionMode": "NONE"
}
```

Maker 订单 3
```json
{
  "symbol": "BTCUSDT",
  "orderId": 4,
  "orderListId": -1,
  "clientOrderId": "L6FmpCJJP6q4hCNv4MuZDG",
  "price": "1.000000",
  "origQty": "8.100000",
  "executedQty": "0.000000",
  "cummulativeQuoteQty": "0.000000",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.000000",
  "icebergQty": "0.000000",
  "time": 1670219812026,
  "updateTime": 1670219812026,
  "isWorking": true,
  "workingTime": 1670219812026,
  "origQuoteOrderQty": "0.000000",
  "selfTradePreventionMode": "NONE"
}
```

Taker 订单的状态

```json
{
  "symbol": "BTCUSDT",
  "orderId": 5,
  "orderListId": -1,
  "clientOrderId": "kocvDAi4GNN2y1l1Ojg1Ri",
  "price": "1.000000",
  "origQty": "3.000000",
  "executedQty": "0.000000",
  "cummulativeQuoteQty": "0.000000",
  "status": "EXPIRED_IN_MATCH",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "SELL",
  "stopPrice": "0.000000",
  "icebergQty": "0.000000",
  "time": 1670219812046,
  "updateTime": 1670219812046,
  "isWorking": true,
  "workingTime": 1670219812046,
  "origQuoteOrderQty": "0.000000",
  "selfTradePreventionMode": "EXPIRE_TAKER",
  "preventedMatchId": 0,
  "preventedQuantity": "3.000000"
}
```


**情况 D - 用户发送带有 `EXPIRE_BOTH` 的订单，该订单将与订单薄上已有的订单撮合。**

```
Maker 订单: symbol=BTCUSDT side=BUY type=LIMIT quantity=1 price=1 selfTradePreventionMode=NONE
Taker 订单: symbol=BTCUSDT side=SELL type=LIMIT quantity=3 price=1 selfTradePreventionMode=EXPIRE_BOTH
```

**结果:** 两个订单都将过期。

Maker 订单

```json
{
  "symbol": "ABCDEF",
  "orderId": 2,
  "orderListId": -1,
  "clientOrderId": "2JPC8xjpLq6Q0665uYWAcs",
  "price": "1.000000",
  "origQty": "1.000000",
  "executedQty": "0.000000",
  "cummulativeQuoteQty": "0.000000",
  "status": "EXPIRED_IN_MATCH",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.000000",
  "icebergQty": "0.000000",
  "time": 1673842412831,
  "updateTime": 1673842413170,
  "isWorking": true,
  "workingTime": 1673842412831,
  "origQuoteOrderQty": "0.000000",
  "selfTradePreventionMode": "NONE",
  "preventedMatchId": 0,
  "preventedQuantity": "1.000000"
}
```

Taker 订单

```json
{
  "symbol": "ABCDEF",
  "orderId": 5,
  "orderListId": -1,
  "clientOrderId": "qMaz8yrOXk2iUIz74cFkiZ",
  "transactTime": 1673842413170,
  "price": "1.000000",
  "origQty": "3.000000",
  "executedQty": "0.000000",
  "cummulativeQuoteQty": "0.000000",
  "status": "EXPIRED_IN_MATCH",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "SELL",
  "workingTime": 1673842413170,
  "fills": [],
  "preventedMatches": [
    {
      "preventedMatchId": 0,
      "makerOrderId": 2,
      "price": "1.000000",
      "takerPreventedQuantity": "3.000000",
      "makerPreventedQuantity": "1.000000"
    }
  ],
  "selfTradePreventionMode": "EXPIRE_BOTH",
  "tradeGroupId": 1,
  "preventedQuantity": "3.000000"
}
```

**情况 E - 用户在订单薄上有一个带有 `EXPIRE_MAKER` 的订单，然后发送一个带有 `EXPIRE_TAKER` 的新订单，该订单将与订单薄上的订单撮合。**


```
Maker 订单: symbol=BTCUSDT side=BUY type=LIMIT quantity=1 price=1 selfTradePreventionMode=EXPIRE_MAKER
Taker 订单: symbol=BTCUSDT side=SELL type=LIMIT quantity=1 price=1 selfTradePreventionMode=EXPIRE_TAKER
```

**结果:** 将使用 taker 订单的 STP 模式，因此 taker 订单将过期。

Maker 订单
```json
{
    "symbol": "ABCDEF",
    "orderId": 0,
    "orderListId": -1,
    "clientOrderId": "jFUap8iFwwgqIpOfAL60GS",
    "price": "1.000000",
    "origQty": "1.000000",
    "executedQty": "0.000000",
    "cummulativeQuoteQty": "0.000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.000000",
    "icebergQty": "0.000000",
    "time": 1670220769261,
    "updateTime": 1670220769261,
    "isWorking": true,
    "workingTime": 1670220769261,
    "origQuoteOrderQty": "0.000000",
    "selfTradePreventionMode": "EXPIRE_MAKER"
}
```

Taker 订单
```json
{
    "symbol": "ABCDEF",
    "orderId": 1,
    "orderListId": -1,
    "clientOrderId": "zxrvnNNm1RXC3rkPLUPrc1",
    "transactTime": 1670220800315,
    "price": "1.000000",
    "origQty": "1.000000",
    "executedQty": "0.000000",
    "cummulativeQuoteQty": "0.000000",
    "status": "EXPIRED_IN_MATCH",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL",
    "workingTime": 1670220800315,
    "fills": [],
    "preventedMatches": [
        {
            "preventedMatchId": 0,
            "makerOrderId": 0,
            "price": "1.000000",
            "takerPreventedQuantity": "1.000000"
        }
    ],
    "selfTradePreventionMode": "EXPIRE_TAKER",
    "preventedQuantity": "1.000000"
}
```


**情况 F - 用户发送带有 `EXPIRE_MAKER` 的市价订单，该订单将与订单薄上已有的订单撮合。**


```
Maker 订单: symbol=ABCDEF side=BUY type=LIMIT quantity=1 price=1  selfTradePreventionMode=NONE
Taker 订单: symbol=ABCDEF side=SELL type=MARKET quantity=1 selfTradePreventionMode=EXPIRE_MAKER
```

**结果:** 由于 STP，订单薄上的订单会过期，状态为 `EXPIRED_IN_MATCH`。
由于订单薄上的流动性低，新订单也已过期但状态为 `EXPIRED`。

Maker 订单

```json
{
  "symbol": "ABCDEF",
  "orderId": 2,
  "orderListId": -1,
  "clientOrderId": "7sgrQQInL69XDMQpiqMaG2",
  "price": "1.000000",
  "origQty": "1.000000",
  "executedQty": "0.000000",
  "cummulativeQuoteQty": "0.000000",
  "status": "EXPIRED_IN_MATCH",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.000000",
  "icebergQty": "0.000000",
  "time": 1670222557456,
  "updateTime": 1670222557478,
  "isWorking": true,
  "workingTime": 1670222557456,
  "origQuoteOrderQty": "0.000000",
  "selfTradePreventionMode": "NONE",
  "preventedMatchId": 0,
  "preventedQuantity": "1.000000"
}
```

Taker 订单
```json
{
  "symbol": "ABCDEF",
  "orderId": 3,
  "orderListId": -1,
  "clientOrderId": "zqhsgGDEcdhxy2oza2Ljxd",
  "transactTime": 1670222557478,
  "price": "0.000000",
  "origQty": "1.000000",
  "executedQty": "0.000000",
  "cummulativeQuoteQty": "0.000000",
  "status": "EXPIRED",
  "timeInForce": "GTC",
  "type": "MARKET",
  "side": "SELL",
  "workingTime": 1670222557478,
  "fills": [],
  "preventedMatches": [
    {
      "preventedMatchId": 0,
      "makerOrderId": 2,
      "price": "1.000000",
      "makerPreventedQuantity": "1.000000"
    }
  ],
  "selfTradePreventionMode": "EXPIRE_MAKER"
}
```
