# Using quoteNotionalLimit

**Disclaimer**

- This explanation only applies to the SPOT Exchange.
- The symbols and values used here are fictional and do not imply anything about the actual setup on
  the live exchange.
- For simplicity, the examples in this document do not include commission.

## Why use `quoteNotionalLimit`?

Current order types have some limitations.

BUY `MARKET` orders mean "get me `quantity` X at whatever price the book offers." They can walk the
book and spend more quote asset than you expected if liquidity is thin.

BUY `LIMIT` orders are more predictable, but they are balance-inefficient: the account must lock
`price * quantity` for the worst case where the whole order rests on the book. If you have a good
read on the book and can estimate slippage, that locked notional can be larger than what you
actually intend to spend.

`quoteNotionalLimit` is an optional parameter on BUY orders that limits the quote asset the order
will consume. The order locks the notional limit at entry rather than walking the book or using
`price * quantity`, and execution stops as soon as the notional limit is spent, even if there is
`quantity` left over.

Unlike `quoteOrderQty`, where the engine computes the quantity for you, `quoteNotionalLimit` keeps
the requested `quantity` and treats the notional limit as a hard spending limit. `quoteOrderQty` and
`quoteNotionalLimit` cannot be specified on the same order.

## How do I send an order with a notional limit?

The notional limit is a `DECIMAL` amount in the quote asset. It must be greater than zero when
supplied, and it is accepted only on BUY orders.

For example, supplying `quoteNotionalLimit=0` through the REST API returns:

```json
{
  "code": -1102,
  "msg": "Mandatory parameter 'quoteNotionalLimit' was not sent, was empty/null, or malformed."
}
```

A notional limit that is too large for the Matching Engine is rejected with `-1013`:

```json
{
  "code": -1013,
  "msg": "Quote notional limit is over the symbol's maximum notional."
}
```

This is not the same as a notional limit that the account cannot fund. The order locks the notional
limit at entry, so if the account does not have enough free quote asset, the order is rejected with
the usual insufficient balance error instead. For example, an account with `1,000 USDT` free that
sends a `BTCUSDT` order with `quoteNotionalLimit=5000` receives:

```json
{
  "code": -2010,
  "msg": "Account has insufficient balance for requested action."
}
```

Order-list endpoints prefix the parameter name to identify which order it applies to:

- OCO: `aboveQuoteNotionalLimit`, `belowQuoteNotionalLimit`
- OTO: `workingQuoteNotionalLimit`, `pendingQuoteNotionalLimit`
- OTOCO: `workingQuoteNotionalLimit`, `pendingAboveQuoteNotionalLimit`,
  `pendingBelowQuoteNotionalLimit`
- OPO, OPOCO: `workingQuoteNotionalLimit` (pending orders are SELL-only and cannot have a notional
  limit)

Please refer to the following table:

<table border="1" cellpadding="5" cellspacing="0">
  <thead>
    <tr>
      <th>API</th>
      <th>Request</th>
      <th>Parameters</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3">REST API</td>
      <td><code>POST /api/v3/order</code>, <code>POST /api/v3/order/test</code></td>
      <td rowspan="6"><code>quoteNotionalLimit</code> on single-order endpoints; <code>above*</code>/<code>below*</code>, <code>working*</code>/<code>pending*</code> prefixed variants on order lists (see the list above).</td>
    </tr>
    <tr>
      <td><code>POST /api/v3/orderList/oco</code>, <code>/oto</code>, <code>/otoco</code>, <code>/opo</code>, <code>/opoco</code></td>
    </tr>
    <tr>
      <td><code>POST /api/v3/order/cancelReplace</code></td>
    </tr>
    <tr>
      <td rowspan="3">WebSocket API</td>
      <td><code>order.place</code>, <code>order.test</code></td>
    </tr>
    <tr>
      <td><code>orderList.place.oco</code>, <code>.oto</code>, <code>.otoco</code>, <code>.opo</code>, <code>.opoco</code></td>
    </tr>
    <tr>
      <td><code>order.cancelReplace</code></td>
    </tr>
    <tr>
      <td rowspan="3">FIX API</td>
      <td>NewOrderSingle <code>&lt;D&gt;</code></td>
      <td rowspan="3">Custom tag <code>25057</code> <code>QuoteNotionalLimit</code>. The same order type rules apply as on the REST API and WebSocket API.</td>
    </tr>
    <tr>
      <td>NewOrderList <code>&lt;E&gt;</code></td>
    </tr>
    <tr>
      <td>OrderCancelRequestAndNewOrderSingle <code>&lt;XCN&gt;</code></td>
    </tr>
  </tbody>
</table>

> [!NOTE]
>
> The legacy OCO endpoint (`POST /api/v3/order/oco`, WebSocket `orderList.place`) does not accept the
> new parameters. `quoteNotionalLimit` is also not shown in `POST /api/v3/order/test` responses or in
> `newOrderRespType=ACK` responses since they carry no order attributes.
>

[Smart Order Routing (SOR)](sor_faq.md) does not support `quoteNotionalLimit`: SOR endpoints reject
requests with this parameter, returning [`-1106`](../testnet/errors.md#-1106-param_not_required).

This sample REST API response shows `quoteNotionalLimit` on a BUY `LIMIT` order for `10` at `30`
with `quoteNotionalLimit=250`. Resting all `10` on the book would need `10 * 30 = 300`, which
exceeds the notional limit, so if nothing filled on entry the order would expire instead of resting.
Here the order filled `5` at `20` on entry, spending `100`. Resting the remaining `5` needs
`5 * 30 = 150`, and `100 + 150 = 250` fits within the notional limit, so the remaining `5` rest on
the book:

```json
{
  "symbol": "BTCUSDT",
  "orderId": 42,
  "orderListId": -1,
  "clientOrderId": "P8hJx3kN9rT2vL6qY4wB1z",
  "transactTime": 1750313780050,
  "price": "30.00000000",
  "origQty": "10.00000000",
  "executedQty": "5.00000000",
  "origQuoteOrderQty": "0.00000000",
  "quoteNotionalLimit": "250.00000000",
  "cummulativeQuoteQty": "100.00000000",
  "status": "PARTIALLY_FILLED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "workingTime": 1750313780050,
  "fills": [
    {
      "price": "20.00000000",
      "qty": "5.00000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": 7
    }
  ],
  "selfTradePreventionMode": "NONE"
}
```

Note that `quoteNotionalLimit` is only present in a response when the order was placed with the
parameter.

It appears on order placement (`RESULT`/`FULL`), order status, open orders, all orders, cancel,
cancel-replace and amend-keep-priority responses, as well as in `orderReports` entries for order
lists.

### Example market buy

Send a signed REST API `POST /api/v3/order` request with these parameters, plus the usual
`timestamp` and `signature`. Use `X-MBX-TIME-UNIT: MICROSECOND` for the timestamp units shown below.

| Parameter            | Value              |
| -------------------- | ------------------ |
| `symbol`             | `BTCUSDT`          |
| `side`               | `BUY`              |
| `type`               | `MARKET`           |
| `quantity`           | `1`                |
| `quoteNotionalLimit` | `60000`            |
| `newOrderRespType`   | `FULL`             |
| `newClientOrderId`   | `notional_example` |

With an ask for `1 BTC` at `100,000 USDT`, the order buys `0.6 BTC`, spends `60,000 USDT`, and
expires because its remaining `0.4 BTC` would exceed the notional limit.

Response:

```json
{
  "symbol": "BTCUSDT",
  "orderId": 1,
  "orderListId": -1,
  "clientOrderId": "notional_example",
  "transactTime": 1792059013310530,
  "price": "0.0000",
  "origQty": "1.0000",
  "executedQty": "0.6000",
  "origQuoteOrderQty": "0.0000",
  "quoteNotionalLimit": "60000.0000",
  "cummulativeQuoteQty": "60000.0000",
  "status": "EXPIRED",
  "expiryReason": "INSUFFICIENT_QUOTE_NOTIONAL_LIMIT",
  "timeInForce": "GTC",
  "type": "MARKET",
  "side": "BUY",
  "workingTime": 1792059013310530,
  "fills": [
    {
      "price": "100000.0000",
      "qty": "0.6000",
      "commission": "0.0000",
      "commissionAsset": "BTC",
      "tradeId": 0
    }
  ],
  "selfTradePreventionMode": "NONE"
}
```

`origQty` remains the requested quantity. `executedQty` and `cummulativeQuoteQty` show what the
order actually bought and spent; `quoteNotionalLimit` retains the supplied limit.

## What order types support orders with a notional limit?

Only BUY orders accept `quoteNotionalLimit`. On orders on the SELL side, the parameter is rejected
with `-1013` "Quote notional limit is not allowed for this order."

### Market orders

- `MARKET`, `STOP_LOSS`, `TAKE_PROFIT`
- Supported when `marketQuoteNotionalLimitAllowed` is `true` for the symbol. This field is shown in
  `exchangeInfo`.
- The order locks the supplied notional limit at entry instead of estimating a lock from the book.
- `STOP_LOSS` and `TAKE_PROFIT` keep the notional limit locked through the waiting phase so when the
  order triggers, it never fails for lack of funds.
- `quoteNotionalLimit` and `quoteOrderQty` cannot both be specified on the same order. This is
  rejected with [`-1106`](../testnet/errors.md#-1106-param_not_required).

### Limit orders

- `LIMIT`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT_LIMIT`
- Supported when `limitQuoteNotionalLimitAllowed` is `true` for the symbol. This field is shown in
  `exchangeInfo`.
- The order locks the supplied notional limit instead of `price * quantity`. The notional limit must
  be strictly less than `price * quantity`, otherwise, the request is rejected with
  [`-1013`](../testnet/errors.md#-1013-invalid_message).
- `LIMIT_MAKER` orders cannot have a notional limit, because it would conflict with their resting
  notional. The same applies anywhere a `LIMIT_MAKER` can appear, including order lists.
- Pegged `LIMIT` orders lock the smaller of the notional limit and `quantity * peg price`.

### Order lists

- `OCO`: each contingent order has its own optional notional limit, with the order-specific prefixes
  above. A `LIMIT_MAKER` order still cannot have one.
- `OTO`, `OTOCO`: the working `LIMIT` order and any eligible pending BUY order (single or OCO) can
  have a notional limit. If the working order expires with `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT`,
  pending orders expire with `OTO_PHASE_ONE_EXPIRED` and are never placed.
- `OPO`, `OPOCO`: only the working `LIMIT` order can have a notional limit. Pending orders are
  SELL-only and cannot have one.

## What happens when the notional limit is reached?

Before finalizing a fill, the engine compares the notional limit against the notional already spent
by the taker order plus the candidate fill. If the notional limit would be exceeded, the candidate
fill is reduced until the notional limit would not be exceeded and then further reduced so the
candidate fill is a quantity in whole `LOT_SIZE` steps. If the taker order has any unfilled
quantity, it expires with the new expiry reason `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT` (see
[enums](../testnet/enums.md#expiry-reasons)).

The taker order reaching its notional limit does not affect the maker order's quantity or priority.

Note that a market order expires with the expiry reason `INSUFFICIENT_LIQUIDITY` when no asks remain
on the book after it takes, even if the last fill spent the whole notional limit.

For a BUY `GTC` `LIMIT`, the remainder rests only if `spent + remaining quantity * price` still fits
within the notional limit. Otherwise the remainder expires with `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT`,
even if the book (not the notional limit) ended the take.

For `IOC`, any remaining quantity expires after the take. The expiry reason is
`INSUFFICIENT_QUOTE_NOTIONAL_LIMIT` when the notional limit stopped the take and asks at or below
the limit price remain on the book. Otherwise it is `UNFILLED_IOC_QUANTITY_EXPIRED`, even if the
last fill spent the whole notional limit.

For `FOK`, the engine first checks that the entire quantity fits within the notional limit; if not,
the order expires with `UNFILLED_FOK_ORDER_EXPIRED` and no fills.

[Self-Trade Prevention (STP)](stp_faq.md) affects the notional limit depending on the STP mode
applied to the prevented match:

- `TRANSFER`: the notional of the prevented quantity is transferred and counts toward the notional
  limit like a trade. It is not included in `cummulativeQuoteQty`, which only reflects trades. In
  the `GTC` rest check above, transferred notional counts as spent.
- `DECREMENT`: the prevented quantity reduces the order's remaining quantity but does not count
  toward the notional limit. This also applies when an order specifies `TRANSFER` but `DECREMENT` is
  applied, for example when both orders are from the same account.
- `EXPIRE_TAKER`, `EXPIRE_MAKER`, `EXPIRE_BOTH`: behave the same as for orders without a notional
  limit.

### Example: BUY `10` MARKET with `quoteNotionalLimit=90`, book ask `3` at `30` then `5` at `40`

User data stream `executionReport` events:

| Event | `x`       | `X`                | `l` | `L` | `z` | `Z` | `eR`                                |
| ----- | --------- | ------------------ | --- | --- | --- | --- | ----------------------------------- |
| 1     | `NEW`     | `NEW`              | 0   | 0   | 0   | 0   | absent                              |
| 2     | `TRADE`   | `PARTIALLY_FILLED` | 3   | 30  | 3   | 90  | absent                              |
| 3     | `EXPIRED` | `EXPIRED`          | 0   | 0   | 3   | 90  | `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT` |

Every event retains `q=10`, `Qq=90` and `Q=0`. The quote asset lock is `90` at entry and `0` after
the expiry; the full notional limit was spent. The maker trade event comes before the expiry event.
If an order fills completely exactly at the notional limit, there is a single `FILLED` event and no
expiry event.

### Example: BUY `10` at `30` `GTC` with `quoteNotionalLimit=250`

| Entry fills  | Spent | Remaining | Spent + remaining \* limit price | Outcome                                                                      |
| ------------ | ----- | --------- | -------------------------------- | ---------------------------------------------------------------------------- |
| None         | 0     | 10        | 300                              | Expire with `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT`; release `250`               |
| `2` at `20`  | 40    | 8         | 280                              | Expire remaining `8` with `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT`; release `210` |
| `5` at `20`  | 100   | 5         | 250                              | Rest `5` with lock `150`; nothing to release                                 |
| `10` at `20` | 200   | 0         | 200                              | `FILLED`; release `50`                                                       |

In the "`2` at `20`" case, the events are `NEW`, `TRADE` with `z=2`/`Z=40`, then `EXPIRED` with the
same totals and `eR=INSUFFICIENT_QUOTE_NOTIONAL_LIMIT`. The remainder never rests and does not
appear in `openOrders`.

## Which symbols allow orders with a notional limit?

Please refer to Exchange Information requests and look for two new per-symbol fields:

- `marketQuoteNotionalLimitAllowed` — if `true`, `MARKET`, `STOP_LOSS` and `TAKE_PROFIT` BUY orders
  can have a notional limit.
- `limitQuoteNotionalLimitAllowed` — if `true`, `LIMIT`, `STOP_LOSS_LIMIT` and `TAKE_PROFIT_LIMIT`
  BUY orders can have a notional limit.

The two flags are independent of each other and independent of `quoteOrderQtyMarketAllowed`. When
the applicable flag is `false`, sending the parameter is rejected with `-1013` "Quote notional limit
is not supported for this order type on this symbol."

For example, these `exchangeInfo` fields allow both groups. This excerpt omits unrelated symbol
fields.

```json
{
  "symbol": "BTCUSDT",
  "orderTypes": [
    "LIMIT",
    "LIMIT_MAKER",
    "MARKET",
    "STOP_LOSS",
    "STOP_LOSS_LIMIT",
    "TAKE_PROFIT",
    "TAKE_PROFIT_LIMIT"
  ],
  "marketQuoteNotionalLimitAllowed": true,
  "limitQuoteNotionalLimitAllowed": true
}
```

## Which filters are applicable to orders with a notional limit?

Orders with a notional limit are checked against the same filters as orders without one, with the
following adjustments:

- `MIN_NOTIONAL`, `NOTIONAL` (including iceberg slice checks) and quote asset `MAX_ASSET` filters
  use the smaller of the notional limit and the computed notional.
- `MARKET_LOT_SIZE`, `LOT_SIZE`, `PRICE_FILTER` and max position checks are unchanged. Max position
  is checked against the supplied `quantity` at entry (unlike quote-asset quantity market orders,
  which defer the check).
- The notional limit itself is not checked against any step size or precision filter beyond the
  quote asset's own decimals.
- For a BUY `MARKET` order with a notional limit on a symbol with no last price, `MIN_NOTIONAL` with
  `applyToMarket` and `NOTIONAL` with `applyMinToMarket` still reject the order, because the
  notional limit is only an upper bound and the trade could still fall below the minimum.
  Maximum-side market filters (`NOTIONAL` with `applyMaxToMarket` and quote asset `MAX_ASSET`) check
  the notional limit against the maximum.
