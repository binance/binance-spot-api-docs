# Self Trade Prevention (STP) FAQ

**Disclaimer:**
* The commissions and prices used here are fictional and do not imply anything about the actual setup on the live exchange.

### What is Self Trade Prevention?

Self Trade Prevention (or STP) prevents orders of users, or the user's `tradeGroupId` to match against their own.

### What defines a self-trade?

A self-trade can occur in either scenario:

* The order traded against the same account.
* The order traded against an account with the same `tradeGroupId`.

### What happens when STP is triggered?

There are five possible modes for what the system does when an order would create a self-trade.

`NONE` - This mode exempts the order from self-trade prevention. Accounts or Trade group IDs will not be compared, no orders will be expired, and the trade will occur.

`EXPIRE_TAKER` - This mode prevents a trade by immediately expiring the taker order's remaining quantity.

`EXPIRE_MAKER` - This mode prevents a trade by immediately expiring the potential maker order's remaining quantity.

`EXPIRE_BOTH` - This mode prevents a trade by immediately expiring both the taker and the potential maker orders' remaining quantities.

`DECREMENT`  - This mode increases the `prevented quantity` of *both* orders by the amount of the prevented match. The smaller of the two orders will expire, or both if they have the same quantity.

`TRANSFER` - If orders are from the same account, then the behavior is the same as `DECREMENT`.
If orders are from different accounts with the same `tradeGroupId`, then in addition to the behavior of `DECREMENT`, the `last prevented quantity` and its notional are transferred between the two accounts.

STP behavior is typically determined by the STP mode of the **taker order** only. The exception is that for STP `TRANSFER` to occur, both the maker and taker orders must specify STP mode `TRANSFER`. If the taker order specifies STP mode `TRANSFER`, but the maker order specifies a different STP mode, then the STP behavior is `DECREMENT`.

In summary:

| Taker Order STP Mode | Maker Order STP Mode | Effective STP Mode |
| :---- | :---- | :---- |
| `TRANSFER` | `TRANSFER` | `TRANSFER` |
| `TRANSFER` | `EXPIRE_MAKER`, `EXPIRE_TAKER`, `EXPIRE_BOTH`, `NONE`, `DECREMENT` | `DECREMENT` |
| `EXPIRE_MAKER`, `EXPIRE_TAKER`, `EXPIRE_BOTH`, `NONE`, `DECREMENT`  | ANY STP MODE | STP mode of the Taker Order |

### What is a Trade Group Id?

Different accounts with the same `tradeGroupId` are considered part of the same "trade group". Orders submitted by members of a trade group are eligible for STP according to the taker-order's STP mode.

A user can confirm if their accounts are under the same `tradeGroupId` from the API either from `GET /api/v3/account` (REST API) or `account.status` (WebSocket API) for each account.

The field is also present in the response for `GET /api/v3/preventedMatches` (REST API) or `myPreventedMatches` (WebSocket API).

If the value is `-1`, then the `tradeGroupId` has not been set for that account, so the STP may only take place between orders of the same account.

### What is a Prevented Match?

When a self-trade is prevented, a prevented match is created. The orders in the prevented match have their prevented quantities increased and one or more orders expire.

This is not to be confused with a trade, as no orders will match.

This is a record of what orders could have self-traded.

This can be queried through the endpoint `GET /api/v3/preventedMatches` on the REST API or `myPreventedMatches` on the WebSocket API.

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
    "takerPreventedQuantity": "1.00000000",     //Taker's remaining quantity before the STP. Only appears if the STP mode is EXPIRE_TAKER, EXPIRE_BOTH or DECREMENT.
    "makerPreventedQuantity": "10.00000000",    //Maker's remaining quantity before the STP. Only appears if the STP mode is EXPIRE_MAKER, EXPIRE_BOTH, or DECREMENT.
    "transactTime": 1663190634060               //Time the order(s) expired due to STP.
  }
]
```

### What is "prevented quantity?"

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

While an order is open, the following equation holds true:

```
original order quantity - executed quantity - prevented quantity = quantity available for further execution
```

When an order's available quantity goes to zero, the order will be removed from the order book and the status will be one of `EXPIRED_IN_MATCH`, `FILLED`, or `EXPIRED`.

### How do I know which symbol uses STP?

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

If a user wants to explicitly specify the mode they can pass the enum `NONE`, `EXPIRE_TAKER`, or `EXPIRE_BOTH`.

If a user tries to specify `EXPIRE_MAKER` for orders on this symbol, they will receive an error:

```json
{
    "code": -1013,
    "msg": "This symbol does not allow the specified self-trade prevention mode."
}
```

### How do I know if an order expired due to STP?

The order will have the status `EXPIRED_IN_MATCH`.

### STP Examples

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
  "symbol": "BTCUSDT",
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
  "symbol": "BTCUSDT",
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
    "symbol": "BTCUSDT",
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
    "symbol": "BTCUSDT",
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
Maker Order: symbol=BTCUSDT side=BUY  type=LIMIT  quantity=1 price=1  selfTradePreventionMode=NONE
Taker Order: symbol=BTCUSDT side=SELL type=MARKET quantity=1          selfTradePreventionMode=EXPIRE_MAKER
```

**Result**: The existing order expires with the status `EXPIRED_IN_MATCH`, due to STP.
The new order also expires but with status `EXPIRED`, due to low liquidity on the order book.

Maker Order

```json
{
  "symbol": "BTCUSDT",
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
  "symbol": "BTCUSDT",
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

**Scenario G- A user sends a limit order with `DECREMENT` which would match with an existing order.**

```
Maker Order: symbol=BTCUSDT side=BUY  type=LIMIT quantity=6 price=2  selfTradePreventionMode=NONE
Taker Order: symbol=BTCUSDT side=SELL type=LIMIT quantity=2 price=2  selfTradePreventionMode=DECREMENT
```

**Result**: Both orders have a preventedQuantity of 2. Since this is the taker order’s full quantity, it expires due to STP.

Maker Order

```json
{
  "symbol": "BTCUSDT",
  "orderId": 23,
  "orderListId": -1,
  "clientOrderId": "Kxb4RpsBhfQrkK2r2YO2Z9",
  "price": "2.00000000",
  "origQty": "6.00000000",
  "executedQty": "0.00000000",
  "cummulativeQuoteQty": "0.00000000",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.00000000",
  "icebergQty": "0.00000000",
  "time": 1741682807892,
  "updateTime": 1741682816376,
  "isWorking": true,
  "workingTime": 1741682807892,
  "origQuoteOrderQty": "0.00000000",
  "selfTradePreventionMode": "DECREMENT",
  "preventedMatchId": 4,
  "preventedQuantity": "2.00000000"
}
```

Taker Order

```json
{
  "symbol": "BTCUSDT",
  "orderId": 24,
  "orderListId": -1,
  "clientOrderId": "dwf3qOzD7GM9ysDn9XG9AS",
  "price": "2.00000000",
  "origQty": "2.00000000",
  "executedQty": "0.00000000",
  "cummulativeQuoteQty": "0.00000000",
  "status": "EXPIRED_IN_MATCH",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "SELL",
  "stopPrice": "0.00000000",
  "icebergQty": "0.00000000",
  "time": 1741682816376,
  "updateTime": 1741682816376,
  "isWorking": true,
  "workingTime": 1741682816376,
  "origQuoteOrderQty": "0.00000000",
  "selfTradePreventionMode": "DECREMENT",
  "preventedMatchId": 4,
  "preventedQuantity": "2.00000000"
}
```

**Scenario H- A user sends a limit order with `TRANSFER` which would match with an existing order under the same tradeGroupId.**

Balances before order placement

Maker's Balances

```json
{
  "balances": [
    {
      "asset": "BTC",
      "free": "20000.00000000",
      "locked": "0.00000000"
    },
    {
      "asset": "USDT",
      "free": "20000.00000000",
      "locked": "0.00000000"
    }
  ]
}
```

Taker's Balances

```json
{
  "balances": [
    {
      "asset": "BTC",
      "free": "20000.00000000",
      "locked": "0.00000000"
    },
    {
      "asset": "USDT",
      "free": "20000.00000000",
      "locked": "0.00000000"
    }
  ]
}
```

```
Maker Order: symbol=BTCUSDT side=BUY  type=LIMIT quantity=0.6 price=0.2  selfTradePreventionMode=TRANSFER tradeGroupId=1
Taker Order: symbol=BTCUSDT side=SELL type=LIMIT quantity=0.2 price=0.2  selfTradePreventionMode=TRANSFER tradeGroupId=1
```

**Result:** Both orders have a preventedQuantity of 0.2. Since this is the taker’s full quantity, it expires due to STP.

Maker Order

```json
{
    "symbol": "BTCUSDT",
    "orderId": 12,
    "orderListId": -1,
    "clientOrderId": "zEyu9HGqiT5YUaXXhKr1MR",
    "price": "0.20000000",
    "origQty": "0.60000000",
    "executedQty": "0.00000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.00000000",
    "icebergQty": "0.00000000",
    "time": 1762852466582,
    "updateTime": 1762852522145,
    "isWorking": true,
    "workingTime": 1762852466582,
    "origQuoteOrderQty": "0.00000000",
    "selfTradePreventionMode": "TRANSFER",
    "preventedMatchId": 3,
    "preventedQuantity": "0.20000000"
}
```

Taker Order

```json
{
    "symbol": "BTCUSDT",
    "orderId": 13,
    "orderListId": -1,
    "clientOrderId": "6T06cph3Et2yFNnGpHdejh",
    "transactTime": 1762852522145,
    "price": "0.20000000",
    "origQty": "0.20000000",
    "executedQty": "0.00000000",
    "origQuoteOrderQty": "0.00000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "EXPIRED_IN_MATCH",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL",
    "workingTime": 1762852522145,
    "fills": [],
    "preventedMatches": [
        {
            "preventedMatchId": 3,
            "makerSymbol": "BTCUSDT",
            "makerOrderId": 12,
            "price": "0.20000000",
            "takerPreventedQuantity": "0.20000000",
            "makerPreventedQuantity": "0.20000000"
        }
    ],
    "selfTradePreventionMode": "TRANSFER",
    "tradeGroupId": 1,
    "preventedQuantity": "0.20000000"
}
```

Balances after self-trade prevention:

Maker Balances

```json
{
  "balances": [
    {
      "asset": "BTC",
      "free": "20000.20000000",
      "locked": "0.00000000"
    },
    {
      "asset": "USDT",
      "free": "19999.88000000",
      "locked": "0.08000000"
    }
  ]
}
```

Taker's Balances

```json
{
  "balances": [
    {
      "asset": "BTC",
      "free": "19999.80000000",
      "locked": "0.00000000"
    },
    {
      "asset": "USDT",
      "free": "20000.04000000",
      "locked": "0.00000000"
    }
  ]
}
```
