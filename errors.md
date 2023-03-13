# Error codes for Binance (2023-03-13)
Errors consist of two parts: an error code and a message. Codes are universal,
 but messages can vary. Here is the error JSON payload:
```javascript
{
  "code":-1121,
  "msg":"Invalid symbol."
}
```


## 10xx - General Server or Network issues
#### -1000 UNKNOWN
 * An unknown error occurred while processing the request.

#### -1001 DISCONNECTED
 * Internal error; unable to process your request. Please try again.

#### -1002 UNAUTHORIZED
 * You are not authorized to execute this request.

#### -1003 TOO_MANY_REQUESTS
 * Too many requests queued.
 * Too much request weight used; current limit is %s request weight per %s. Please use WebSocket Streams for live updates to avoid polling the API.
 * Way too much request weight used; IP banned until %s. Please use WebSocket Streams for live updates to avoid bans.

#### -1006 UNEXPECTED_RESP
 * An unexpected response was received from the message bus. Execution status unknown.

#### -1007 TIMEOUT
 * Timeout waiting for response from backend server. Send status unknown; execution status unknown.

#### -1008 SERVER_BUSY
  * Server is currently overloaded with other requests. Please try again in a few minutes. 

#### -1014 UNKNOWN_ORDER_COMPOSITION
 * Unsupported order combination.

#### -1015 TOO_MANY_ORDERS
 * Too many new orders.
 * Too many new orders; current limit is %s orders per %s.

#### -1016 SERVICE_SHUTTING_DOWN
 * This service is no longer available.

#### -1020 UNSUPPORTED_OPERATION
 * This operation is not supported.

#### -1021 INVALID_TIMESTAMP
 * Timestamp for this request is outside of the recvWindow.
 * Timestamp for this request was 1000ms ahead of the server's time.

#### -1022 INVALID_SIGNATURE
 * Signature for this request is not valid.


## 11xx - Request issues
#### -1100 ILLEGAL_CHARS
 * Illegal characters found in a parameter.
 * Illegal characters found in parameter '%s'; legal range is '%s'.

#### -1101 TOO_MANY_PARAMETERS
 * Too many parameters sent for this endpoint.
 * Too many parameters; expected '%s' and received '%s'.
 * Duplicate values for a parameter detected.

#### -1102 MANDATORY_PARAM_EMPTY_OR_MALFORMED
 * A mandatory parameter was not sent, was empty/null, or malformed.
 * Mandatory parameter '%s' was not sent, was empty/null, or malformed.
 * Param '%s' or '%s' must be sent, but both were empty/null!

#### -1103 UNKNOWN_PARAM
 * An unknown parameter was sent.

#### -1104 UNREAD_PARAMETERS
 * Not all sent parameters were read.
 * Not all sent parameters were read; read '%s' parameter(s) but was sent '%s'.

#### -1105 PARAM_EMPTY
 * A parameter was empty.
 * Parameter '%s' was empty.

#### -1106 PARAM_NOT_REQUIRED
 * A parameter was sent when not required.
 * Parameter '%s' sent when not required.

#### -1108 PARAM_OVERFLOW
 * Parameter '%s' overflowed.

#### -1111 BAD_PRECISION
 * Precision is over the maximum defined for this asset.

#### -1112 NO_DEPTH
 * No orders on book for symbol.

#### -1114 TIF_NOT_REQUIRED
 * TimeInForce parameter sent when not required.

#### -1115 INVALID_TIF
 * Invalid timeInForce.

#### -1116 INVALID_ORDER_TYPE
 * Invalid orderType.

#### -1117 INVALID_SIDE
 * Invalid side.

#### -1118 EMPTY_NEW_CL_ORD_ID
 * New client order ID was empty.

#### -1119 EMPTY_ORG_CL_ORD_ID
 * Original client order ID was empty.

#### -1120 BAD_INTERVAL
 * Invalid interval.

#### -1121 BAD_SYMBOL
 * Invalid symbol.

#### -1125 INVALID_LISTEN_KEY
 * This listenKey does not exist.

#### -1127 MORE_THAN_XX_HOURS
 * Lookup interval is too big.
 * More than %s hours between startTime and endTime.

#### -1128 OPTIONAL_PARAMS_BAD_COMBO
 * Combination of optional parameters invalid.

#### -1130 INVALID_PARAMETER
 * Invalid data sent for a parameter.
 * Data sent for parameter '%s' is not valid.

#### -1134 BAD_STRATEGY_TYPE
 * `strategyType` was less than 1000000. 

#### -1135 INVALID_JSON
 * Invalid JSON Request
 * JSON sent for parameter '%s' is not valid

#### -1145 INVALID_CANCEL_RESTRICTIONS
 * `cancelRestrictions` has to be either `ONLY_NEW` or `ONLY_PARTIALLY_FILLED`.

#### -2010 NEW_ORDER_REJECTED
 * NEW_ORDER_REJECTED

#### -2011 CANCEL_REJECTED
 * CANCEL_REJECTED

#### -2013 NO_SUCH_ORDER
 * Order does not exist.

#### -2014 BAD_API_KEY_FMT
 * API-key format invalid.

#### -2015 REJECTED_MBX_KEY
 * Invalid API-key, IP, or permissions for action.

#### -2016 NO_TRADING_WINDOW
 * No trading window could be found for the symbol. Try ticker/24hrs instead.

#### -2026 ORDER_ARCHIVED
  * Order was canceled or expired with no executed qty over 90 days ago and has been archived.


## Messages for -1010 ERROR_MSG_RECEIVED, -2010 NEW_ORDER_REJECTED, and -2011 CANCEL_REJECTED
This code is sent when an error has been returned by the matching engine.
The following messages which will indicate the specific error:


Error message                                                   | Description
------------                                                    | ------------
"Unknown order sent."                                           | The order (by either `orderId`, `clOrdId`, `origClOrdId`) could not be found
"Duplicate order sent."                                         | The `clOrdId` is already in use.
"Market is closed."                                             | The symbol is not trading.
"Account has insufficient balance for requested action."        | Not enough funds to complete the action.
"Market orders are not supported for this symbol."              | `MARKET` is not enabled on the symbol.
"Iceberg orders are not supported for this symbol."             | `icebergQty` is not enabled on the symbol.
"Stop loss orders are not supported for this symbol."           | `STOP_LOSS` is not enabled on the symbol.
"Stop loss limit orders are not supported for this symbol."     | `STOP_LOSS_LIMIT` is not enabled on the symbol.
"Take profit orders are not supported for this symbol."         | `TAKE_PROFIT` is not enabled on the symbol.
"Take profit limit orders are not supported for this symbol."   | `TAKE_PROFIT_LIMIT` is not enabled on the symbol.
"Price * QTY is zero or less."                                  | `price` * `quantity` is too low.
"IcebergQty exceeds QTY."                                       | `icebergQty` must be less than the order quantity.
"This action is disabled on this account."                      | Contact customer support; some actions have been disabled on the account.
"This account may not place or cancel orders."                  | Contact customer support; the account has trading ability disabled.
"Unsupported order combination"                                 | The `orderType`, `timeInForce`, `stopPrice`, and/or `icebergQty` combination isn't allowed.
"Order would trigger immediately."                              | The order's stop price is not valid when compared to the last traded price.
"Cancel order is invalid. Check origClOrdId and orderId."       | No `origClOrdId` or `orderId` was sent in.
"Order would immediately match and take."                       | `LIMIT_MAKER` order type would immediately match and trade, and not be a pure maker order.
"The relationship of the prices for the orders is not correct." | The prices set in the `OCO` is breaking the Price rules. <br/> The rules are: <br/> `SELL Orders`: Limit Price > Last Price > Stop Price <br/>`BUY Orders`: Limit Price < Last Price < Stop Price
"OCO orders are not supported for this symbol"                  | `OCO` is not enabled on the symbol.
"Quote order qty market orders are not support for this symbol."| `MARKET` orders using the parameter `quoteOrderQty` are not enabled on the symbol.
"Trailing stop orders are not supported for this symbol."       | Orders using `trailingDelta` are not enabled on the symbol.
"Order cancel-replace is not supported for this symbol."        | `POST /api/v3/order/cancelReplace` is not enabled for the symbol.
"This symbol is not permitted for this account."                | Account does not have permission to trade on this symbol.
"This symbol is restricted for this account."                   | Account does not have permission to trade on this symbol.
"Order was not canceled due to cancel restrictions."            | Either `cancelRestrictions` was set to `ONLY_NEW` but the order status was not `NEW` <br/> or <br/> `cancelRestrictions` was set to `ONLY_PARTIALLY_FILLED` but the order status was not `PARTIALLY_FILLED`. 

## Errors regarding POST /api/v3/order/cancelReplace

### -2021 Order cancel-replace partially failed

This code is sent when either the cancellation of the order failed or the new order placement failed but not both.

### -2022 Order cancel-replace failed.

This code is sent when both the cancellation of the order failed and the new order placement failed.

## Filter failures
Error message | Description
------------ | ------------
"Filter failure: PRICE_FILTER" | `price` is too high, too low, and/or not following the tick size rule for the symbol.
"Filter failure: PERCENT_PRICE" | `price` is X% too high or X% too low from the average weighted price over the last Y minutes.
"Filter failure: LOT_SIZE" | `quantity` is too high, too low, and/or not following the step size rule for the symbol.
"Filter failure: MIN_NOTIONAL" | `price` * `quantity` is too low to be a valid order for the symbol.
"Filter failure: ICEBERG_PARTS" | `ICEBERG` order would break into too many parts; icebergQty is too small.
"Filter failure: MARKET_LOT_SIZE" | `MARKET` order's `quantity` is too high, too low, and/or not following the step size rule for the symbol.
"Filter failure: MAX_POSITION" | The account's position has reached the maximum defined limit. <br/> This is composed of the sum of the balance of the base asset, and the sum of the quantity of all open `BUY` orders.
"Filter failure: MAX_NUM_ORDERS" | Account has too many open orders on the symbol.
"Filter failure: MAX_NUM_ALGO_ORDERS" | Account has too many open stop loss and/or take profit orders on the symbol.
"Filter failure: MAX_NUM_ICEBERG_ORDERS" | Account has too many open iceberg orders on the symbol.
"Filter failure: TRAILING_DELTA" | `trailingDelta` is not within the defined range of the filter for that order type.
"Filter failure: EXCHANGE_MAX_NUM_ORDERS" | Account has too many open orders on the exchange.
"Filter failure: EXCHANGE_MAX_NUM_ALGO_ORDERS" | Account has too many open stop loss and/or take profit orders on the exchange.
"Filter failure: EXCHANGE_MAX_NUM_ICEBERG_ORDERS" | Account has too many open iceberg orders on the exchange.
