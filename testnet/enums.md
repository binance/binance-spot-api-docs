# ENUM Definitions

This will apply for both REST API and WebSocket API.

## Symbol status (status):

- `TRADING`
- `END_OF_DAY`
- `HALT`
- `BREAK`
- `CANCEL_ONLY`

<a id="account-and-symbol-permissions"></a>

## Account and Symbol Permissions (permissions)

- `SPOT`

## Order status (status)

| Status             | Description                                                                                                                                                                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `NEW`              | The order has been accepted by the engine.                                                                                                                                                                                                              |
| `PENDING_NEW`      | The order is in a pending phase until the working order of an order list has been fully filled.                                                                                                                                                         |
| `PARTIALLY_FILLED` | A part of the order has been filled.                                                                                                                                                                                                                    |
| `FILLED`           | The order has been completed.                                                                                                                                                                                                                           |
| `CANCELED`         | The order has been canceled by the user.                                                                                                                                                                                                                |
| `PENDING_CANCEL`   | Currently unused                                                                                                                                                                                                                                        |
| `REJECTED`         | The order was not accepted by the engine and not processed.                                                                                                                                                                                             |
| `EXPIRED`          | The order was canceled according to the order type's rules (e.g. LIMIT FOK orders with no fill, LIMIT IOC or MARKET orders that partially fill) <br/> or by the exchange, (e.g. orders canceled during liquidation, orders canceled during maintenance) |
| `EXPIRED_IN_MATCH` | The order was expired by the exchange due to STP. (e.g. an order with `EXPIRE_TAKER` will match with existing orders on the book with the same account or same `tradeGroupId`)                                                                          |

## Order List Status (listStatusType)

| Status         | Description                                                                                                    |
| -------------- | -------------------------------------------------------------------------------------------------------------- |
| `RESPONSE`     | This is used when the ListStatus is responding to a failed action. (E.g. order list placement or cancellation) |
| `EXEC_STARTED` | The order list has been placed or there is an update to the order list status.                                 |
| `UPDATED`      | The clientOrderId of an order in the order list has been changed.                                              |
| `ALL_DONE`     | The order list has finished executing and thus is no longer active.                                            |

## Order List Order Status (listOrderStatus)

| Status      | Description                                                                                       |
| ----------- | ------------------------------------------------------------------------------------------------- |
| `EXECUTING` | Either an order list has been placed or there is an update to the status of the list.             |
| `ALL_DONE`  | An order list has completed execution and thus no longer active.                                  |
| `REJECT`    | The List Status is responding to a failed action either during order placement or order canceled. |

## ContingencyType

- `OCO`
- `OTO`

<a id="allocationtype"></a>

## AllocationType

- `SOR`

<a id="ordertypes"></a>

## Order types (orderTypes, type)

- `LIMIT`
- `MARKET`
- `STOP_LOSS`
- `STOP_LOSS_LIMIT`
- `TAKE_PROFIT`
- `TAKE_PROFIT_LIMIT`
- `LIMIT_MAKER`

<a id="orderresponsetype"></a>

## Order Response Type (newOrderRespType)

- `ACK`
- `RESULT`
- `FULL`

## Working Floor

- `EXCHANGE`
- `SOR`

<a id="side"></a>

## Order side (side)

- `BUY`
- `SELL`

<a id="timeinforce"></a>

## Time in force (timeInForce)

This sets how long an order will be active before expiration.

| Status | Description                                                                                               |
| ------ | --------------------------------------------------------------------------------------------------------- |
| `GTC`  | Good Til Canceled <br/> An order will be on the book unless the order is canceled.                        |
| `IOC`  | Immediate Or Cancel <br/> An order will try to fill the order as much as it can before the order expires. |
| `FOK`  | Fill or Kill <br/> An order will expire if the full order cannot be filled upon execution.                |

## Rate limiters (rateLimitType)

- REQUEST_WEIGHT

```json
{
  "rateLimitType": "REQUEST_WEIGHT",
  "interval": "MINUTE",
  "intervalNum": 1,
  "limit": 6000
}
```

- ORDERS

```json
{
  "rateLimitType": "ORDERS",
  "interval": "SECOND",
  "intervalNum": 1,
  "limit": 10
}
```

- RAW_REQUESTS

```json
{
  "rateLimitType": "RAW_REQUESTS",
  "interval": "MINUTE",
  "intervalNum": 5,
  "limit": 61000
}
```

## Rate limit intervals (interval)

- SECOND
- MINUTE
- DAY

<a id="stpmodes"></a>

## STP Modes

Read [Self Trade Prevention (STP) FAQ](../faqs/stp_faq.md) to learn more.

- `NONE`
- `EXPIRE_MAKER`
- `EXPIRE_TAKER`
- `EXPIRE_BOTH`
- `DECREMENT`
- `TRANSFER`

<a id="execution-types"></a>

## Execution Types

| Status             | Description                                                                                                                                                                                                                                        |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `NEW`              | The order has been accepted into the engine.                                                                                                                                                                                                       |
| `CANCELED`         | The order has been canceled by the user.                                                                                                                                                                                                           |
| `REPLACED`         | The order has been amended.                                                                                                                                                                                                                        |
| `REJECTED`         | The order has been rejected and was not processed. (e.g. Cancel Replace orders where the new order placement was rejected even if the cancel request succeeded.)                                                                                   |
| `TRADE`            | Part of the order or all of the order's quantity has filled.                                                                                                                                                                                       |
| `EXPIRED`          | The order was canceled according to the order type's rules (e.g. LIMIT FOK orders with no fill, LIMIT IOC or MARKET orders that partially fill) or by the exchange, (e.g. orders canceled during liquidation, orders canceled during maintenance). |
| `TRADE_PREVENTION` | The order has expired due to STP.                                                                                                                                                                                                                  |

<a id="execution-rules"></a>

## Execution Rules

- `PRICE_RANGE`

<!-- prettier-ignore -->
<a id="expiryreasons"></a> 
<a id="expiry-reasons"></a>

## Expiry Reasons

| Expiry Reason (`eR`)                  | Explanation                                                                                                                                                                                      |
| :------------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `NONE`                                | The order has not expired.                                                                                                                                                                       |
| `REJECTED`                            | A contingent order or an order that was part of an OTO was rejected by the matching engine when trying to place it on the order book. Common reasons are lack of funds and rejection by filters. |
| `EXCHANGE_CANCELED`                   | The order was canceled by Binance.                                                                                                                                                               |
| `OCO_TRIGGER`                         | An order that was part of an OCO pair was canceled because the other order of the pair started working or the entire OCO expired.                                                                |
| `OTO_PHASE_ONE_EXPIRED`               | The working order of the order list expired, thus expiring the entire order list.                                                                                                                |
| `UNFILLED_IOC_QUANTITY_EXPIRED`       | The IOC order was not fully filled and thus expired.                                                                                                                                             |
| `UNFILLED_FOK_ORDER_EXPIRED`          | The FOK order was not fully filled and thus expired.                                                                                                                                             |
| `INSUFFICIENT_LIQUIDITY`              | There were not enough orders in the order book to match with this order.                                                                                                                         |
| `EXECUTION_RULE_PRICE_RANGE_EXCEEDED` | The order attempted to trade at a price that would not meet the Price Range Execution Rule.                                                                                                      |
| `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT`   | The order's `quoteNotionalLimit` prevented the remaining quantity from executing or resting, so it expired. See [Quote Notional Limit](rest-api.md#quote-notional-limit).                        |
| `EXHAUSTED`                           | The order was amended down to its filled quantity or less, so it expired.                                                                                                                        |
