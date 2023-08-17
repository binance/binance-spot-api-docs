# SPOT API Glossary

**Disclaimer:** This glossary refers only to the SPOT API Implementation. The definition for these terms may differ with regards to Futures, Options, and other APIs by Binance.

## A

`ACK`
* `newOrderRespType` enum. Stands for a type of order response in JSON where only the following fields are emitted: `symbol`, `orderId`, `orderListId`, `clientOrderId`, and `transactTime`.

`aggTrade`/Aggregate trade
* Aggregation of one or more individual trades from the same taker order that got filled at the same time and price.

allocation
* Transfer of asset from the exchange to your account (e.g., when an order is filled by SOR instead of trading directly).

`allocationId`
* Unique identifier of an allocation on a symbol.

`allocationType`
* See [AllocationType](../rest-api.md#allocationtype) 

`askPrice`
* In ticker responses: the lowest price on the `SELL` side.

`askQty`
* In ticker responses: total quantity offered at the lowest price on the `SELL` side.

`asks`
* Orders on the `SELL` side.

`avgPrice`
* Represents the volume-weighted average price for a set interval of minutes.

---

## B

`baseAsset`
* The first asset in the symbol (e.g. `BTC` is the `base asset` of symbol `BTCUSDT`), which represents the asset being bought and sold (the `quantity`).

`baseAssetPrecision`
* A field found in `GET /api/v3/exchangeInfo` that shows the number of decimals allowed on the `baseAsset`.

`baseCommissionPrecision`
* A field found in `GET /api/v3/exchangeInfo` that represents the number of decimals base asset commission will be calculated to.

`bidPrice`
* In ticker responses: the highest price on the `BUY` side.

`bidQty`
* In ticker responses: total quantity offered at the highest price on the `BUY` side.

`bids`
* Orders on the `BUY` side.

`BREAK`
* Symbol's trading status that represents the symbol is not available for trading, which can happen during expected downtime. Market data is not generated during `BREAK`.

`BUY`
* An enum in the `side` parameter when a user wants to purchase an asset (e.g. `BTC`).

---

## C

`CANCELED`
* Order `status` indicating the order has been canceled by the user.

`clientOrderId`
* A field, which can be set by the user, in the JSON response for `POST /api/v3/order` to identify the newly placed order.

`commission`
* The fee that was paid on a trade.

`commissionAsset`
* The asset the commission fees were deducted from.

`cancelReplaceMode`
* Parameter used in Cancel Replace orders (i.e. `POST /api/v3/order/cancelReplace`) that define whether the New Order Placement should proceed if the Cancel Request fails.

`cummulativeQuoteQty`
* The accumulation of the `price` * `qty` for each fill of an order.

---

## D

Data Source
* Specifies where the endpoint or request is retrieving their data.

---

## E

`executedQty`
* The field that shows how much of the quantity was filled in an order.

`EXPIRED`
* Order `status` indicating the order was canceled according to the order type's rules or by the exchange.

`EXPIRED_IN_MATCH`
* Order `status` indicating the order was canceled by the exchange due to STP. (e.g. an order with `EXPIRE_TAKER` will match with existing orders on the book with the same account or same `tradeGroupId`)

---

## F

`filters`
* Defines the trading rules on the exchange.

`FOK`/ Fill or Kill
* `timeInForce` enum where the order will not fill and expire if the order cannot be fully filled.

`free`
* The amount of an asset in a user's balances that can be used to trade or withdraw.

`FULL`
* `newOrderRespType` enum. Stands for a type of order response in JSON, where all the order information is emitted, including orders `fills` field.

---

## G

`GTC`/ Good Til Canceled
* `timeInForce` enum where the order will remain active until it is canceled or fully filled.

---

## H

`HALT`
* Symbol's trading status that represents the symbol is not available for trading, which can happen during emergency downtime. Market data is still generated during `HALT`.

---

## I

`intervalNum`
* Describes the amount of time in the interval (e.g. if `interval` is `SECOND` and `intervalNum` is 5, then this will be interpreted as every 5 seconds).

`IOC` / Immediate or Canceled
* `timeInForce` enum where the order tries to fill as much as possible, and the remaining unfilled quantity will expire.

`isBestMatch`
* Field in the Response JSON that determines if the price of the trade was the best available on the exchange.

`isBuyerMaker`
* Field in the Response JSON that indicates if the Buy side (the Buyer) was also the market maker (the Maker).

`isWorking`
* Field in the JSON that shows if the order has started working on the order book.

---

## K

`kline`
* Identifies the open, close, high, low price, trading volume, and other market data, of a symbol at a specified time for a specific duration. Also known as a Candlestick.

---

## L

Last Prevented Quantity
* Order quantity that expired due to STP.

`lastPrice`
* Price of the latest trade.

`lastQty`
* Quantity filled by the last trade.

`LIMIT`
* a `type` of order where the execution price will be no worse than the order's set price. The execution price is limited to be the set price or better.

`LIMIT_MAKER`
* A `type` of order where the order can only be a maker order (i.e. The order cannot immediately match and take).

`limitClientOrderId`
* A parameter used in placing OCO orders (i.e. `POST /api/v3/order/oco`) that identifies the `LIMIT_MAKER` pair of the OCO Order.

`listClientOrderId`
* A parameter used in placing OCO Orders (i.e. `POST /api/v3/order/oco`) that identifies the pair of orders.

`listenKey`
* Individual key used on User Data Stream to get live updates on the associated account.

`locked`
* The amount of an asset in a user's balances that are currently locked in open orders and other services by the platform.

---

## M

`MARKET`
* A `type` of order where the user buys or sells an asset at the best available prices and liquidity until the order is fully filled or the order book's liquidity is exhausted.

Matching Engine
* This can either refer to a Data Source in the documentation which means the response is coming from the engine.
* Or this is referred to as the system that handles all the requests and matches orders.

Memory
* Data Source where the response is coming from the API's internal memory or cache.

---

## N

`NEW`
* Order `status` where a order has been successfully sent to the Matching Engine.

`newClientOrderId`
* Parameter used in the SPOT API to assign the `clientOrderId` for the order being placed or the cancel message.

Notional value
* The `price` * `qty` value.

---

## O

`OCO`
* One-Cancels-the-Other type of order that is composed by a pair of orders (i.e. `STOP_LOSS` or `STOP_LOSS_LIMIT` paired with a `LIMIT_MAKER` order) with the condition that if one of the orders execute, the other is automatically expired.

Order Book
* List of the open bids and asks for a symbol.

`orderId`
* A field in the order response that uniquely identifies the order on a symbol.

`origQty`
* The original `quantity` that was sent during order placement.

`origClientOrderId`
* Field used when canceling or querying an order by providing the `clientOrderId`.

---

## P

`PARTIALLY_FILLED`
* Order `status` indicating that part of the order has been partially filled.

`preventedQuantity`
* Order quantity expired due to STP events.

Prevented Match
* When order(s) expire due to the STP, a "prevented match" records the event.

`preventedMatchId`
* When used in combination with `symbol`, can be used to query a prevented match of the expired order.

---

## Q

`quantity`
* Parameter used to specify the amount of the `base asset` to buy or sell.

`quoteAsset`
* The second asset in the symbol (e.g. `USDT` is the `quote asset` of symbol `BTCUSDT`) which represents the asset being used to quote prices (the `price`).

`quoteAssetPrecision`
* A field found in `GET /api/v3/exchangeInfo` that shows the number of decimals allowed on the `quoteAsset`.

`quoteCommissionPrecision`
* A field found in `GET /api/v3/exchangeInfo` that represents the number of decimals quote asset commission will be calculated to.

`quoteOrderQty`
* `MARKET` order parameter that specifies the amount of the quote asset one wants to spend/receive in a "Reverse MARKET order".

`quoteQty`
* `price` * `qty`; the notional value.

---

## R

`recvWindow`
* Parameter in the APIs that can be used to specify the number of milliseconds after the `timestamp` the request is valid for.

`RESULT`
* `newOrderRespType` enum. Stands for a type of order response in JSON, where all the order information is emitted, except order's `fills` field.

Reverse `MARKET` order
* A `MARKET` order that is specified using the `quoteOrderQty` instead of the `quantity`.

---

## S

Self Trade Prevention (STP)
* Self Trade Prevention is a feature that prevents orders of users, or the user's `tradeGroupId` from matching against their own.

`selfTradePreventionMode`
* A parameter used to specify what the system will do if an order could cause a self-trade.

`SELL`
* An enum in the `side` used when a user wants to sell an asset (e.g. BTC).

Smart Order Routing (SOR)
* Smart Order Routing uses interchangeable quote assets to improve liquidity. [Read SOR FAQ](./sor_faq.md) to learn more.

`SPOT`
* This is to distinguish a type of trading, where the purchase and delivery of a asset is made immediately.

`stopClientOrderId`
* A parameter used in placing OCO orders (i.e. `POST /api/v3/order/oco`) that identifies the `STOP_LOSS` or `STOP_LOSS_LIMIT` pair of the OCO Order.

`stopPrice`
* The price used in algorithmic orders (e.g. `STOP_LOSS`, `TAKE_PROFIT`) that determines when an order will be triggered to be placed on the order book.
* The price used in trailing algorithmic orders (e.g. `STOP_LOSS`, `TAKE_PROFIT`) to determine when trailing price tracking begins.

`STOP_LOSS`
* A `type` of algorithmic order where once the market price hits the `stopPrice`, a `MARKET` order is placed on the order book.

`STOP_LOSS_LIMIT`
* A `type` of algorithmic order where once the market price hits the `stopPrice`, a `LIMIT` order is placed on the order book.

`strategyId`
* Arbitrary numeric value identifying the order within an order strategy.

`strategyType`
* Arbitrary numeric value identifying the order strategy.

`symbol`
* A trading pair, composed of a `base asset` and a `quote asset`. (e.g. BTCUSDT and BNBBTC)

---

## T

`TAKE_PROFIT`
* A `type` of algorithmic order where once the market price hits the `stopPrice`, a `MARKET` order is placed on the order book.

`TAKE_PROFIT_LIMIT`
* A `type` of algorithmic order where once the market price hits the `stopPrice`, a `LIMIT` order is placed on the order book.

`ticker`
* Reports the price change, and other maker data, of a symbol within a certain rolling interval.

`time`
* For trade/allocation queries: the time when trades/allocations were executed.
* For order queries: the time when orders were created.

`timeInForce`
* Determines the taker behavior of an order, if an order can be a maker order, and how long the order will stay on the order book before it expires.
* Supported enums are `GTC`, `IOC`, and `FOK`.

`tradeGroupId`
* Group of accounts that belong to the same "trade group".

`TRADING`
* Trading status where orders can be placed.

`trailingDelta`
* Trailing Stop Order parameter that specifies the delta price change required before order activation.

`trailingTime`
* The time when the trailing order is now active and tracking price changes.

`transactTime`
* The time when the order was updated: placed, filled, or canceled. This field (as well as all timestamp related fields) will be in milliseconds.

---

## U

`uiKlines`
* Modified candlestick data that is optimized for presentation of candlestick charts.

`updateTime`
* Last update to the order. This field (as well as all timestamp related fields) will be in milliseconds.

User Data Stream
* Websocket stream used to get real-time information of a user's account. (e.g. Changes to Balances, Order Updates, etc.)

---

## W

`weightedAveragePrice`
* The volume weighted average price in the last x minutes.

`workingFloor`
* A field that determines whether the order is being filled by the SOR or by the order book the order was submitted to.

`workingTime`
* The time when the order started working on the order book.

---

## X

`X-MBX-ORDER-COUNT-XX`
* Response header that is emitted when a user places an order, indicating the current order count for the interval XX for that account.

`X-MBX-USED-WEIGHT-XX`
* Response header that is emitted when a user sends any request to the API, indicating the current used request weight for the XX interval by the user's IP.

