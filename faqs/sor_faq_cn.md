# 智能指令路由 (SOR)

**声明:** 

* 这里使用的符号和数值是虚构的，并不意味着真实交易所中的设置。
* 为简单起见，本文档中的示例不包括佣金。

## 什么是智能指令路由 (SOR)?

**智能订单路由**（Smart Order Routing，简称SOR）允许客户通过使用具有相同基础资产(`base asset `)和可互换报价资产(`interchangeable quote assets`)的其他订单簿(order books)中的流动性来潜在获得更好的流动性。可互换报价资产是具有固定的1比1兑换率的报价资产，例如与同一法定货币挂钩的稳定币。

请注意，尽管报价资产(quote assets)是可互换的，但在出售基础资产(base asset)时，您将始终收到订单中交易对(symbol)对应的报价资产(quote asset)。

当您使用`SOR`下单时，它会在SOR配置的订单簿(order books)里，寻找每个订单簿的最佳价格水平，并在可能的情况下从中交易。

**注意：** 如果使用SOR的订单无法根据符合条件的订单簿流动性完全成交，IOC限价单(`LIMIT IOC`)或市价单(`MARKET`)将立即过期，而GTC限价单(`LIMIT GTC`)将把剩余数量放置在您最初提交订单的订单簿(order book)上。

示例 1:

让我们考虑一个包含交易对`BTCUSDT`，`BTCUSDC`和`BTCUSDP`的SOR配置，并给出以下这些符号的卖出(`ASK`)方向的订单簿:

```
BTCUSDT quantity 3 price 30,800
BTCUSDT quantity 3 price 30,500

BTCUSDC quantity 1 price 30,000
BTCUSDC quantity 1 price 28,000

BTCUSDP quantity 1 price 35,000
BTCUSDP quantity 1 price 29,000
```

如果您以价格为31000、数量为0.5的限价挂单买入`BTCUSDT`，并且在`BTCUSDT`的订单簿中找到最佳的卖出价格为30,500 USDT，您将花费15,250 USDT 并收到0.5 BTC。

如果您通过使用SOR下达了一笔GTC限价买单(`LIMIT GTC BUY`)，购买`BTCUSDT`，数量为 0.5，价格为 31000，您将与SOR涵盖的所有交易对中最佳的卖出价格相匹配，即`BTCUSDC`，价格为 28,000。您将花费 14,000 USDT（不是 USDC！），并收到 0.5 BTC。

```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 2,
  "orderListId": -1,
  "clientOrderId": "sBI1KM6nNtOfj5tccZSKly",
  "transactTime": 1689149087774,
  "price": "31000.00000000",
  "origQty": "0.50000000",
  "executedQty": "0.50000000",
  "cummulativeQuoteQty": "14000.00000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "workingTime": 1689149087774,
  "fills": [
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "28000.00000000",
      "qty": "0.50000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": -1,
      "allocId": 0
    }
  ],
  "workingFloor": "SOR",
  "selfTradePreventionMode": "NONE",
  "usedSor": true
}
```
 
示例 2:

使用示例1中同样的订单薄:

```
BTCUSDT quantity 3 price 30,800
BTCUSDT quantity 3 price 30,500

BTCUSDC quantity 1 price 30,000
BTCUSDC quantity 1 price 28,000

BTCUSDP quantity 1 price 35,000
BTCUSDP quantity 1 price 29,000
```

如果您下达一笔GTC限价买单(`LIMIT GTC BUY`)购买`BTCUSDT`，数量为 5，价格为 31,000，则：

* 与 `BTCUSDT` 订单簿中价格为 30,500 USDT 的 3 个 `BTCUSDT` 成交，以 91,500 USDT 的价格购买 3 个 BTC。
* 然后与 `BTCUSDT` 订单簿中价格为 30,800 USDT 的 3 个 `BTCUSDT` 成交，以 61,600 USDT 的价格购买 2 个 BTC。

总计您花费了 153,100 USDT 并获得了 5 BTC。

如果您通过使用SOR下达相同的GTC限价买单(`LIMIT GTC BUY`)购买 `BTCUSDT`，数量为 5，价格为 31,000，则：

* 与 `BTCUSDC` 订单簿中价格为 28,000 的 1 个 BTCUSDC 成交，以 28,000 USDT 的价格购买 1 个 BTC。
* 与 `BTCUSDP` 订单簿中价格为 29,000 的 1 个 BTCUSDP 成交，以 29,000 USDT 的价格购买 1 个 BTC。
* 与 `BTCUSDC` 订单簿中价格为 30,000 的 1 个 BTCUSDC 成交，以 30,000 USDT 的价格购买 1 个 BTC。
* 与 `BTCUSDT` 订单簿中价格为 30,500 的 3 个 BTCUSDT 成交，以 61,000 USDT 的价格购买 2 个 BTC。

总计您花费了 148,000 USDT 并获得了 5 BTC。

```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 2,
  "orderListId": -1,
  "clientOrderId": "tHonoNjWfOSaKiTygN3bfY",
  "transactTime": 1689146154686,
  "price": "31000.00000000",
  "origQty": "5.00000000",
  "executedQty": "5.00000000",
  "cummulativeQuoteQty": "148000.00000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "workingTime": 1689146154686,
  "fills": [
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "28000.00000000",
      "qty": "1.00000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": -1,
      "allocId": 0
    },
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "29000.00000000",
      "qty": "1.00000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": -1,
      "allocId": 1
    },
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "30000.00000000",
      "qty": "1.00000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": -1,
      "allocId": 2
    },
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "30500.00000000",
      "qty": "2.00000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": -1,
      "allocId": 3
    }
  ],
  "workingFloor": "SOR",
  "selfTradePreventionMode": "NONE",
  "usedSor": true
}
```

示例 3:

使用示例1和2中同样的订单薄:

```
BTCUSDT quantity 3 price 30,800
BTCUSDT quantity 3 price 30,500

BTCUSDC quantity 1 price 30,000
BTCUSDC quantity 1 price 28,000

BTCUSDP quantity 1 price 35,000
BTCUSDP quantity 1 price 29,000
```

如果您通过使用SOR下市价买单(`MARKET` `BUY`) 购买 `BTCUSDT`，数量为 11，但在所有符合条件的订单簿中总共只有 10 个BTC可供交易。一旦所有SOR配置中的订单簿都耗尽了，剩余的数量1将过期。

```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 2,
  "orderListId": -1,
  "clientOrderId": "jdFYWTNyzplbNvVJEzQa0o",
  "transactTime": 1689149513461,
  "price": "0.00000000",
  "origQty": "11.00000000",
  "executedQty": "10.00000000",
  "cummulativeQuoteQty": "305900.00000000",
  "status": "EXPIRED",
  "timeInForce": "GTC",
  "type": "MARKET",
  "side": "BUY",
  "workingTime": 1689149513461,
  "fills": [
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "28000.00000000",
      "qty": "1.00000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": -1,
      "allocId": 0
    },
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "29000.00000000",
      "qty": "1.00000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": -1,
      "allocId": 1
    },
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "30000.00000000",
      "qty": "1.00000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": -1,
      "allocId": 2
    },
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "30500.00000000",
      "qty": "3.00000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": -1,
      "allocId": 3
    },
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "30800.00000000",
      "qty": "3.00000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": -1,
      "allocId": 4
    },
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "35000.00000000",
      "qty": "1.00000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": -1,
      "allocId": 5
    }
  ],
  "workingFloor": "SOR",
  "selfTradePreventionMode": "NONE",
  "usedSor": true
}
```

示例 4:

假设有一个包含 `BTCUSDT`, `BTCUSDC` 和 `BTCUSDP` 交易对的SOR配置, 以及下面这些交易对买方(`BID`)的订单簿：

```
BTCUSDT quantity 5 price 29,500

BTCUSDC quantity 5 price 35,000
BTCUSDC quantity 5 price 30,000

BTCUSDP quantity 5 price 28,000
```

如果您在`BTCUSDT`下一笔GTC限价卖单(`LIMIT GTC SELL`) 订单，价格是29000, 卖出10 BTC，那么您将卖出 5 个 BTC 并获得 147,500 USDT。由于`BTCUSDT`订单簿上没有更好的价格可用，订单的剩余（未成交）数量将会以29,000的价格保持在订单簿上。

如果您通过使用SOR下GTC限价卖单(`LIMIT GTC SELL`) 卖出 `BTCUSDT`，则会：

* 与 `BTCUSDC` 订单簿中价格为 35,000 的 5 个 BTCUSDC 成交，以 175,000 USDT 的价格出售 5 个 BTC。
* 与 `BTCUSDC` 订单簿中价格为 30,000 的 5 个 BTCUSDC 成交，以 150,000 USDT 的价格出售 5 个 BTC。

总计您卖出 10 个 BTC 并获得 325,000 USDT。

```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 1,
  "orderListId": -1,
  "clientOrderId": "W1iXSng1fS77dvanQJDGA5",
  "transactTime": 1689147920113,
  "price": "29000.00000000",
  "origQty": "10.00000000",
  "executedQty": "10.00000000",
  "cummulativeQuoteQty": "325000.00000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "SELL",
  "workingTime": 1689147920113,
  "fills": [
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "35000.00000000",
      "qty": "5.00000000",
      "commission": "0.00000000",
      "commissionAsset": "USDT",
      "tradeId": -1,
      "allocId": 0
    },
    {
      "matchType": "ONE_PARTY_TRADE_REPORT",
      "price": "30000.00000000",
      "qty": "5.00000000",
      "commission": "0.00000000",
      "commissionAsset": "USDT",
      "tradeId": -1,
      "allocId": 1
    }
  ],
  "workingFloor": "SOR",
  "selfTradePreventionMode": "NONE",
  "usedSor": true
}
```

**概要：SOR的目标是潜在地在具有可互换报价资产(`interchangeable quote assets`)的订单簿之间获得更好的流动性。更好的流动性可以使订单以更好的价格，并更充分成交。**

## 什么交易对支持SOR?

当前SOR配置可以在交易所信息接口查询(Restful接口`GET /api/v3/exchangeInfo`, Websocket API的 `exchangeInfo`).

```json
  "sors": [
    {
      "baseAsset": "BTC",
      "symbols": [
        "BTCUSDT",
        "BTCUSDC",
        "BTCUSDP"
      ]
    }
  ]
```

## 如何下SOR订单?

通过Rest API接口 `POST /api/v3/sor/order`.

通过WebSocket API接口 `sor.order.place`.

## 在API响应里, 有个字段workingFloor是什么意思?

这是一个用于确定订单的最后更新操作（成交、过期或作为新订单下达等）的术语。

如果 workingFloor 是 SOR，这表示您的订单与SOR配置中的其他符合条件的订单簿进行了交互。

如果 workingFloor 是 EXCHANGE，这表示您的订单在您发送该订单的订单簿上进行了交互。

## 如果查询订单是否使用过SOR?

您可以像查询任何其他订单一样来查询。主要的区别是对于使用SOR的订单，在响应中会有两个额外的字段：`usedSor` 和 `workingFloor`。

## 什么是资产分配?

**资产分配**是从交易所转移资产到您的账户。例如，当SOR从符合条件的订单簿中获取流动性时，您的订单将通过资产分配来填充。在这种情况下，您不直接进行交易，而是通过SOR代表您进行交易，并接收对应于SOR为您进行的交易的资产分配。

```javascript
[
  {
    "symbol": "BTCUSDT",            // Symbol the order was submitted to
    "allocationId": 0,    
    "allocationType": "SOR",
    "orderId": 2,       
    "orderListId": -1,
    "price": "30000.00000000",      // Price of the fill
    "qty": "5.00000000",            // Quantity of the fill
    "quoteQty": "150000.00000000",
    "commission": "0.00000000",
    "commissionAsset": "BTC",
    "time": 1688379272280,          // Time the allocation occurred
    "isBuyer": true,
    "isMaker": false,
    "isAllocator": false
  }
]
````

## 如何获取使用SOR的订单成交细节？

当SOR订单与非提交订单的订单簿进行交易时，订单将通过资产分配（allocation）而不是交易(trade)来成交。使用SOR下达的订单可能同时拥有资产分配和交易。

在API响应中，您可以查看`fills`字段。资产分配具有`allocId`和`matchType`: "ONE_PARTY_TRADE_REPORT"，而交易将具有非负的`tradeId`。

您可以使用以下方式查询资产分配和交易：

查询资产分配：使用Rest API接口 `GET /api/v3/myAllocations` 或 WebSocket API 的 `myAllocations`。

查询交易：使用Rest API接口 `GET /api/v3/myTrades` 或 WebSocket API 的 `myTrades`。

## 什么交易对支持SOR?

当前SOR配置可以在交易所信息接口查询(Rest API接口`GET /api/v3/exchangeInfo`, WebSocket API的 `exchangeInfo`).

```json
  "sors": [
    {
      "baseAsset": "BTC",
      "symbols": [
        "BTCUSDT",
        "BTCUSDC",
        "BTCUSDP"
      ]
    }
  ]
```


