# Smart Order Routing (SOR)

**Disclaimer:** 

* The symbols and values used here are fictional, and do not imply anything about the actual setup on the live exchange. 
* For simplicity, the examples in this document do not include commission.

## What is Smart Order Routing (SOR)?

**Smart Order Routing (SOR)** allows you to potentially get better liquidity by filling an order with liquidity from other order books with the same base asset and interchangeable quote assets. **Interchangeable quote assets** are quote assets with fixed 1 to 1 exchange rate, such as stablecoins pegged to the same fiat currency.

Note that even though the quote assets are interchangeable, when selling the base asset you will always receive the quote asset of the symbol in your order.

When you place an order using SOR, it goes through the eligible order books, looks for best price levels for each order book in that SOR configuration, and takes from those books if possible.

**Note:** If the order using SOR cannot fully fill based on the eligible order books' liquidity, `LIMIT IOC` or `MARKET` orders will immediately expire, while `LIMIT GTC` orders will place the remaining quantity on the order book you originally submitted the order to.

### Example 1

Let's consider a SOR configuration containing the symbols `BTCUSDT`, `BTCUSDC` and `BTCUSDP`, and the following `ASK` (`SELL` side) order books for those symbols:

```
BTCUSDT quantity 3 price 30,800
BTCUSDT quantity 3 price 30,500

BTCUSDC quantity 1 price 30,000
BTCUSDC quantity 1 price 28,000

BTCUSDP quantity 1 price 35,000
BTCUSDP quantity 1 price 29,000
```

If you send a `LIMIT GTC BUY` order for `BTCUSDT` with `quantity=0.5` and `price=31000`, you would match with the best SELL price on the BTCUSDT book at 30,500. You would spend 15,250 USDT and receive 0.5 BTC.

If you send a `LIMIT GTC BUY` order _using SOR_ for `BTCUSDT` with `quantity=0.5` and `price=31000`, you would match with the best SELL price across _all symbols in the SOR_, which is BTCUSDC at price 28,000. You would spend 14,000 USDT (_not_ USDC!) and receive 0.5 BTC.

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
 
### Example 2

Using the same order book as Example 1:

```
BTCUSDT quantity 3 price 30,800
BTCUSDT quantity 3 price 30,500

BTCUSDC quantity 1 price 30,000
BTCUSDC quantity 1 price 28,000

BTCUSDP quantity 1 price 35,000
BTCUSDP quantity 1 price 29,000
```

If you send a `LIMIT GTC BUY` order for `BTCUSDT` with `quantity=5` and `price=31000`, you would:

* match with the 3 BTCUSDT at 30,500, and buy 3 BTC for 91,500 USDT
* then match with the 3 BTCUSDT at 30,800, and buy 2 BTC for 61,600 USDT

In total, you spend 153,100 USDT and receive 5 BTC.

If you send the same `LIMIT GTC BUY` order _using SOR_ for `BTCUSDT` with `quantity=5` and `price=31000`, you would:

* match with 1 BTCUSDC at 28,000, and buy 1 BTC for 28,000 USDT
* match with 1 BTCUSDP at 29,000, and buy 1 BTC for 29,000 USDT
* match with 1 BTCUSDC at 30,000, and buy 1 BTC for 30,000 USDT
* match with 3 BTCUSDT at 30,500, and buy 2 BTC for 61,000 USDT

In total, you spend 148,000 USDT and receive 5 BTC.

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

### Example 3

Using the same order book as Example 1 and 2:

```
BTCUSDT quantity 3 price 30,800
BTCUSDT quantity 3 price 30,500

BTCUSDC quantity 1 price 30,000
BTCUSDC quantity 1 price 28,000

BTCUSDP quantity 1 price 35,000
BTCUSDP quantity 1 price 29,000
```

If you send a `MARKET BUY` order for `BTCUSDT` _using SOR_ with `quantity=11`, there is only 10 BTC in total available across all eligible order books. Once all the order books in SOR configuration have been exhausted, the remaining quantity of 1 expires.

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

### Example 4

Let's consider a SOR configuration containing the symbols `BTCUSDT`, `BTCUSDC` and `BTCUSDP` and the following `BID` (`BUY` side) order book for those symbols:

```
BTCUSDT quantity 5 price 29,500

BTCUSDC quantity 5 price 35,000
BTCUSDC quantity 5 price 30,000

BTCUSDP quantity 5 price 28,000
```

If you send a `LIMIT GTC SELL` order for `BTCUSDT` with `price=29000` and `quantity=10`, you would sell 5 BTC and receive 147,500 USDT. Since there is no better price available on the BTCUSDT book, the remaining (unfilled) quantity of the order will rest there at the price of 29,000.

If you send a `LIMIT GTC SELL` order _using SOR_ for `BTCUSDT`, you would:

* match with 5 BTCUSDC at 35,000 and sell 5 BTC for 175,000 USDT
* match with 5 BTCUSDC at 30,000 and sell 5 BTC for 150,000 USDT

In total, you sell 10 BTC and receive 325,000 USDT.

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

**Summary: The goal of SOR is to potentially access better liquidity across order books with interchangeable quote assets. Better liquidity access can fill orders more fully and at better prices during an order's taker phase.**

## What symbols support SOR?

You can find the current SOR configuration in Exchange Information (`GET /api/v3/exchangeInfo` for Rest, and `exchangeInfo` on Websocket API).

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

## How do I place an order using SOR?

On the Rest API, the request is `POST /api/v3/sor/order`.

On the WebSocket API, the request is `sor.order.place`.

## In the API response, there's a field called `workingFloor`. What does that field mean?

This is a term used to determine where the order's last activity occurred (filling, expiring, or being placed as new, etc.).

If the `workingFloor` is `SOR`, this means your order interacted with other eligible order books in the SOR configuration.

If the `workingFloor` is `EXCHANGE`, this means your order interacted on the order book that you sent that order to.

## In the API response, `fills` contain fields `matchType` and `allocId`. What do they mean?

`matchType` field indicates a non-standard order fill.

When your order is filled by SOR, you will see `matchType: ONE_PARTY_TRADE_REPORT`, indicating that you did not trade directly on the exchange (`tradeId: -1`). Instead your order is filled by _allocations_.

`allocId` field identifies the allocation so that you can query it later.

## What are allocations?

**An allocation** is a transfer of an asset from the exchange to your account. For example, when SOR takes liquidity from eligible order books, your order is filled by allocations. In this case you don't trade directly, but rather receive allocations from SOR corresponding to the trades made by SOR on your behalf.

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
```

## How do I query orders that used SOR?

You can find them the same way you query any other order. The main difference is that in the response for an order that used SOR there are two extra fields: `usedSor` and `workingFloor`.


## How do I get details of my fills for orders that used SOR?

When SOR orders trade against order books other than the symbol submitted with the order, the order is filled with an **allocation** and not a trade. Orders placed with SOR can potentially have both allocations and trades.

In the API response, you can review the `fills` fields. Allocations have an `allocId` and `"matchType": "ONE_PARTY_TRADE_REPORT"`, while trades will have a non-negative `tradeId`.

Allocations can be queried using `GET /api/v3/myAllocations` (Rest API) or `myAllocations` (WebSocket API).

Trades can be queried using `GET /api/v3/myTrades` (Rest API) or `myTrades` (WebSocket API).
