# Self Trade Prevention (STP) FAQ

## What is Self Trade Prevention?

Self Trade Prevention (or STP) prevents orders of users, or the user's `tradeGroupId` to match against their own.

## What defines a self-trade?

A self-trade can occur in either scenario:

* The order traded against the same account.
* The order traded against an account with the same `tradeGroupId`.

## What happens when STP is triggered?

There are four possible modes for what the system will do if an order could create a self-trade.

`NONE` - This mode exempts the order from self-trade prevention. Accounts or Trade group IDs will not be compared, no orders will be expired, and the trade will occur.

`EXPIRE_TAKER` - This mode prevents a trade by immediately expiring the taker order's remaining quantity.

`EXPIRE_MAKER` - This mode prevents a trade by immediately expiring the potential maker order's remaining quantity.

`EXPIRE_BOTH` - This mode prevents a trade by immediately expiring both the taker and the potential maker orders' remaining quantities.

The STP event will occur depending on the STP mode of the **taker order**. <br> Thus, the STP mode of an order that goes on the book is no longer relevant and will be ignored for all future order processing.

## What is a Trade Group Id?

Different accounts with the same `tradeGroupId` are considered part of the same "trade group". Orders submitted by members of a trade group are eligible for STP according to the taker-order's STP mode.

A user can confirm if their accounts are under the same `tradeGroupId` from the API either from `GET /api/v3/account` (REST API) or `account.status` (Websocket API) for each account.

The field is also present in the response for `GET /api/v3/preventedMatches` (Rest API) or `myPreventedMatches` (Websocket API).

If the value is `-1`, then the `tradeGroupId` has not been set for that account, so the STP may only take place between orders of the same account.

## What is a Prevented Match?

When one or more orders are expired due to STP, this creates a prevented match.

This is not to be confused with a trade, as no orders will match.

This is a record of what orders could have self-traded.

This can be queried through the endpoint `GET /api/v3/preventedMatches` on the Rest API or `myPreventedMatches` on the Websocket API.

This is a sample of the output request for reference:

```javascript
[
  {
    "symbol": "BTCDUSDT",                       //Symbol of the orders
    "preventedMatchId": 8,                      //Identifies the prevented match of the expired order(s) for the symbol.
    "takerOrderId": 12,                         //Order Id of the Taker Order
    "makerOrderId": 10,                         //Order Id of the Maker Order
    "tradeGroupId": 1,                          //Identifies the Trade Group Id. (If the account is not part of a trade group, this will be -1.)
    "selfTradePreventionMode": "EXPIRE_BOTH",   //STP mode that expired the order(s).
    "price": "50.00000000",                     //Price at which the match occurred.
    "takerPreventedQuantity": "1.00000000",     //Taker's remaining quantity. Only appears if the STP mode is EXPIRE_TAKER or EXPIRE_BOTH.
    "makerPreventedQuantity": "10.00000000",    //Maker's remaining quantity. Only appears if the STP mode is EXPIRE_MAKER or EXPIRE_BOTH.
    "transactTime": 1663190634060               //Time the order(s) expired due to STP.
  }
]
```

## What is "prevented quantity?"

STP events expire quantity from open orders. The STP modes `EXPIRE_TAKER`, `EXPIRE_MAKER`, and `EXPIRE_BOTH` expire all remaining quantity on the affected orders, resulting in the entire open order being expired.

Prevented quantity is the amount of quantity that is expired due to STP events for a particular order. User stream execution reports for orders involved in STP may have these fields:

```javascript
{
  "A":"3.000000", // Prevented Quantity
  "B":"3.000000"  // Last Prevented Quantity
}
```

`B` is present for execution type `TRADE_PREVENTION`, and is the quantity expired due to that individual STP event.

`A` is the cumulative quantity expired due to STP over the lifetime of the order. For `EXPIRE_TAKER`, `EXPIRE_MAKER`, and `EXPIRE_BOTH` modes this will always be the same value as `B`.

API responses for orders which expired due to STP will also have a `preventedQuantity` field, indicating the cumulative quantity expired due to STP over the lifetime of the order.

While an order is open, the following inequality holds true:

```
executed quantity + prevented quantity < original order quantity
```

When an order has status `EXPIRED_IN_MATCH` or `FILLED`, the followiung equation will hold true:

```
executed quantity + prevented quantity = original order quantity
```

## How do I know which symbol uses STP?

Symbols may be configured to allow different sets of STP modes and take different default STP modes.

`defaultSelfTradePreventionMode` - Orders will use this STP mode if the user does not provide one on order placement.

`allowedSelfTradePreventionModes` - Defines the allowed set of STP modes for order placement on that symbol.

For example, if a symbol has the following configuration:

```json
"defaultSelfTradePreventionMode": "NONE",
"allowedSelfTradePreventionModes": [
    "NONE",
    "EXPIRE_TAKER",
    "EXPIRE_BOTH"
  ]
```

Then that means if a user sends an order with no `selfTradePreventionMode` provided, then the order sent will have the value of `NONE`.

If a user wants to explicitly mention the mode they can pass the enum `NONE`, `EXPIRE_TAKER`, or `EXPIRE_BOTH`.

If a user tries to specify `EXPIRE_MAKER` for orders on this symbol, they will receive an error:

```json
{
    "code": -1013,
    "msg": "This symbol does not allow the specified self-trade prevention mode."
}
```

## How do I know if an order expired due to STP?

The order will have the status `EXPIRED_IN_MATCH`.

## STP Examples:

For all these cases, assume that all orders for these examples are made on the same account.

**Scenario A- A user sends a new order with selfTradePreventionMode:`NONE` that will match with another order of theirs that is already on the book.**

```
Maker Order: symbol=BTCUSDT side=BUY  type=LIMIT quantity=1 price=1 selfTradePreventionMode=NONE
Taker Order: symbol=BTCUSDT side=SELL type=LIMIT quantity=1 price=1 selfTradePreventionMode=NONE
```

**Result**: No STP is triggered and the orders will match.

Order Status of the Maker Order

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

Order Status of the Taker Order

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


**Scenario B- A user sends an order with `EXPIRE_MAKER` that would match with their orders that are already on the book.**

```
Maker Order 1: symbol=BTCUSDT side=BUY  type=LIMIT quantity=1.2 price=1.2 selfTradePreventionMode=NONE
Maker Order 2: symbol=BTCUSDT side=BUY  type=LIMIT quantity=1.3 price=1.1 selfTradePreventionMode=NONE
Maker Order 3: symbol=BTCUSDT side=BUY  type=LIMIT quantity=8.1 price=1   selfTradePreventionMode=NONE
Taker Order 1: symbol=BTCUSDT side=SELL type=LIMIT quantity=3   price=1   selfTradePreventionMode=EXPIRE_MAKER
```

**Result**: The orders that were on the book will expire due to STP, and the taker order will go on the book.

Maker Order 1
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

Maker Order 2

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

Maker Order 3
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

Output of the Taker Order

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


**Scenario C - A user sends an order with `EXPIRE_TAKER` that would match with their orders already on the book.**

```
Maker Order 1: symbol=BTCUSDT side=BUY  type=LIMIT quantity=1.2 price=1.2  selfTradePreventionMode=NONE
Maker Order 2: symbol=BTCUSDT side=BUY  type=LIMIT quantity=1.3 price=1.1  selfTradePreventionMode=NONE
Maker Order 3: symbol=BTCUSDT side=BUY  type=LIMIT quantity=8.1 price=1    selfTradePreventionMode=NONE
Taker Order 1: symbol=BTCUSDT side=SELL type=LIMIT quantity=3   price=1    selfTradePreventionMode=EXPIRE_TAKER
```
**Result**: The orders already on the book will remain, while the taker order will expire.

Maker Order 1
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

Maker Order 2

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

Maker Order 3
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

Output of the Taker order

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


**Scenario D- A user has an order on the book, and then sends an order with `EXPIRE_BOTH` that would match with the existing order.**

```
Maker Order: symbol=BTCUSDT side=BUY  type=LIMIT quantity=1 price=1 selfTradePreventionMode=NONE
Taker Order: symbol=BTCUSDT side=SELL type=LIMIT quantity=3 price=1 selfTradePreventionMode=EXPIRE_BOTH
```

**Result:** Both orders will expire.

Maker Order

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

Taker Order

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

**Scenario E - A user has an order on the book with `EXPIRE_MAKER`, and then sends a new order with `EXPIRE_TAKER` which would match with the existing order.**

```
Maker Order: symbol=BTCUSDT side=BUY  type=LIMIT quantity=1 price=1 selfTradePreventionMode=EXPIRE_MAKER
Taker Order: symbol=BTCUSDT side=SELL type=LIMIT quantity=1 price=1 selfTradePreventionMode=EXPIRE_TAKER
```

**Result**: The taker order's STP mode will be used, so the taker order will be expired.

Maker Order
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

Taker Order
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


**Scenario F - A user sends a market order with `EXPIRE_MAKER` which would match with an existing order.**

```
Maker Order: symbol=ABCDEF side=BUY  type=LIMIT  quantity=1 price=1  selfTradePreventionMode=NONE
Taker Order: symbol=ABCDEF side=SELL type=MARKET quantity=1          selfTradePreventionMode=EXPIRE_MAKER
```

**Result**: The existing order expires with the status `EXPIRED_IN_MATCH`, due to STP.
The new order also expires but with status `EXPIRED`, due to low liquidity on the order book.

Maker Order

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

Taker Order
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
