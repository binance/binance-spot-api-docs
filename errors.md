# Error codes for Binance

Errors consist of two parts: an error code and a message. Codes are universal,
 but messages can vary. Here is the error JSON payload:
```javascript
{
  "code":-1121,
  "msg":"Invalid symbol."
}
```

## 10xx - General Server or Network issues
### -1000 UNKNOWN
 * An unknown error occurred while processing the request.

### -1001 DISCONNECTED
 * Internal error; unable to process your request. Please try again.

### -1002 UNAUTHORIZED
 * You are not authorized to execute this request.

### -1003 TOO_MANY_REQUESTS
 * Too many requests queued.
 * Too much request weight used; current limit is %s request weight per %s. Please use WebSocket Streams for live updates to avoid polling the API.
 * Way too much request weight used; IP banned until %s. Please use WebSocket Streams for live updates to avoid bans.

### -1006 UNEXPECTED_RESP
 * An unexpected response was received from the message bus. Execution status unknown.

### -1007 TIMEOUT
 * Timeout waiting for response from backend server. Send status unknown; execution status unknown.

### -1008 SERVER_BUSY
  * Server is currently overloaded with other requests. Please try again in a few minutes.

### -1013 INVALID_MESSAGE
  * The request is rejected by the API. (i.e. The request didn't reach the Matching Engine.)
  * Potential error messages can be found in [Filter Failures](#filter-failures) or [Failures during order placement](#other-errors).

### -1014 UNKNOWN_ORDER_COMPOSITION
 * Unsupported order combination.

### -1015 TOO_MANY_ORDERS
 * Too many new orders.
 * Too many new orders; current limit is %s orders per %s.

### -1016 SERVICE_SHUTTING_DOWN
 * This service is no longer available.

### -1020 UNSUPPORTED_OPERATION
 * This operation is not supported.

### -1021 INVALID_TIMESTAMP
 * Timestamp for this request is outside of the recvWindow.
 * Timestamp for this request was 1000ms ahead of the server's time.

### -1022 INVALID_SIGNATURE
 * Signature for this request is not valid.

### -1033 COMP_ID_IN_USE
 * `SenderCompId(49)` is currently in use. Concurrent use of the same SenderCompId within one account is not allowed.

### -1034 TOO_MANY_CONNECTIONS
 * Too many concurrent connections; current limit is '%s'.
 * Too many connection attempts for account; current limit is %s per '%s'.
 * Too many connection attempts from IP; current limit is %s per '%s'.

### -1035 LOGGED_OUT
 * Please send [Logout`<5>`](fix-api.md#logout) message to close the session.

## 11xx - Request issues
### -1100 ILLEGAL_CHARS
 * Illegal characters found in a parameter.
 * Illegal characters found in parameter '%s'; legal range is '%s'.

### -1101 TOO_MANY_PARAMETERS
 * Too many parameters sent for this endpoint.
 * Too many parameters; expected '%s' and received '%s'.
 * Duplicate values for a parameter detected.

### -1102 MANDATORY_PARAM_EMPTY_OR_MALFORMED
 * A mandatory parameter was not sent, was empty/null, or malformed.
 * Mandatory parameter '%s' was not sent, was empty/null, or malformed.
 * Param '%s' or '%s' must be sent, but both were empty/null!
 * Required tag '%s' missing.
 * Field value was empty or malformed.
 * '%s' contains unexpected value. Cannot be greater than %s.

### -1103 UNKNOWN_PARAM
 * An unknown parameter was sent.
 * Undefined Tag.

### -1104 UNREAD_PARAMETERS
 * Not all sent parameters were read.
 * Not all sent parameters were read; read '%s' parameter(s) but was sent '%s'.

### -1105 PARAM_EMPTY
 * A parameter was empty.
 * Parameter '%s' was empty.

### -1106 PARAM_NOT_REQUIRED
 * A parameter was sent when not required.
 * Parameter '%s' sent when not required.
 * A tag '%s' was sent when not required.

### -1108 PARAM_OVERFLOW
 * Parameter '%s' overflowed.

### -1111 BAD_PRECISION
 * Parameter '%s' has too much precision.

### -1112 NO_DEPTH
 * No orders on book for symbol.

### -1114 TIF_NOT_REQUIRED
 * TimeInForce parameter sent when not required.

### -1115 INVALID_TIF
 * Invalid timeInForce.

### -1116 INVALID_ORDER_TYPE
 * Invalid orderType.

### -1117 INVALID_SIDE
 * Invalid side.

### -1118 EMPTY_NEW_CL_ORD_ID
 * New client order ID was empty.

### -1119 EMPTY_ORG_CL_ORD_ID
 * Original client order ID was empty.

### -1120 BAD_INTERVAL
 * Invalid interval.

### -1121 BAD_SYMBOL
 * Invalid symbol.

### -1122 INVALID_SYMBOLSTATUS
  * Invalid symbolStatus.

### -1125 INVALID_LISTEN_KEY
 * This listenKey does not exist.

### -1127 MORE_THAN_XX_HOURS
 * Lookup interval is too big.
 * More than %s hours between startTime and endTime.

### -1128 OPTIONAL_PARAMS_BAD_COMBO
 * Combination of optional parameters invalid.
 * Combination of optional fields invalid. Recommendation: '%s' and '%s' must both be sent.
 * Fields [%s] must be sent together or omitted entirely.
 * Invalid `MDEntryType (269)` combination. BID and OFFER must be requested together.
 * Conflicting fields: ['%s'...]

### -1130 INVALID_PARAMETER
 * Invalid data sent for a parameter.
 * Data sent for parameter '%s' is not valid.

### -1134 BAD_STRATEGY_TYPE
 * `strategyType` was less than 1000000.
 * `TargetStrategy (847)` was less than 1000000.

### -1135 INVALID_JSON
 * Invalid JSON Request
 * JSON sent for parameter '%s' is not valid

### -1139 INVALID_TICKER_TYPE
 * Invalid ticker type.

### -1145 INVALID_CANCEL_RESTRICTIONS
 * `cancelRestrictions` has to be either `ONLY_NEW` or `ONLY_PARTIALLY_FILLED`.

### -1151 DUPLICATE_SYMBOLS
 * Symbol is present multiple times in the list.

### -1152 INVALID_SBE_HEADER
* Invalid `X-MBX-SBE` header; expected `<SCHEMA_ID>:<VERSION>`.
* Invalid SBE message header.

### -1153 UNSUPPORTED_SCHEMA_ID
* Unsupported SBE schema ID or version specified in the `X-MBX-SBE` header.
* Invalid SBE schema ID or version specified.

### -1155 SBE_DISABLED
* SBE is not enabled.

### -1158 OCO_ORDER_TYPE_REJECTED
* Order type not supported in OCO.
* If the order type provided in the `aboveType` and/or `belowType` is not supported.

### -1160 OCO_ICEBERGQTY_TIMEINFORCE
* Parameter '%s' is not supported if `aboveTimeInForce`/`belowTimeInForce` is not GTC.
* If the order type for the above or below leg is `STOP_LOSS_LIMIT`, and `icebergQty` is provided for that leg, the `timeInForce` has to be `GTC` else it will throw an error.
* `TimeInForce (59)` must be `GTC (1)` when `MaxFloor (111)` is used.

### -1161 DEPRECATED_SCHEMA
* Unable to encode the response in SBE schema 'x'. Please use schema 'y' or higher.

### -1165 BUY_OCO_LIMIT_MUST_BE_BELOW
* A limit order in a buy OCO must be below.

### -1166 SELL_OCO_LIMIT_MUST_BE_ABOVE
* A limit order in a sell OCO must be above.

### -1168 BOTH_OCO_ORDERS_CANNOT_BE_LIMIT
* At least one OCO order must be contingent.

### -1169 INVALID_TAG_NUMBER
 * Invalid tag number.

### -1170 TAG_NOT_DEFINED_IN_MESSAGE
 * Tag '%s' not defined for this message type.

### -1171 TAG_APPEARS_MORE_THAN_ONCE
 * Tag '%s' appears more than once.

### -1172 TAG_OUT_OF_ORDER
 * Tag '%s' specified out of required order.

### -1173 GROUP_FIELDS_OUT_OF_ORDER
 * Repeating group '%s' fields out of order.

### -1174 INVALID_COMPONENT
 * Component '%s' is incorrectly populated on '%s' order. Recommendation: '%s'

### -1175 RESET_SEQ_NUM_SUPPORT
 * Continuation of sequence numbers to new session is currently unsupported. Sequence numbers must be reset for each new session.

### -1176 ALREADY_LOGGED_IN
 * [Logon`<A>`](fix-api.md#logon-main) should only be sent once.

### -1177 GARBLED_MESSAGE
 * `CheckSum(10)` contains an incorrect value.
 * `BeginString (8)` is not the first tag in a message.
 * `MsgType (35)` is not the third tag in a message.
 * `BodyLength (9)` does not contain the correct byte count.
 * Only printable ASCII characters and SOH (Start of Header) are allowed.
 * Tag specified without a value.
* Invalid encodingType.

### -1178 BAD_SENDER_COMPID
 * `SenderCompId(49)` contains an incorrect value. The SenderCompID value should not change throughout the lifetime of a session.

### -1179 BAD_SEQ_NUM
 * `MsgSeqNum(34)` contains an unexpected value. Expected: '%d'.

### -1180 EXPECTED_LOGON
 * [Logon`<A>`](fix-api.md#logon-main) must be the first message in the session.

### -1181 TOO_MANY_MESSAGES
 * Too many messages; current limit is '%d' messages per '%s'.

### -1182 PARAMS_BAD_COMBO
 * Conflicting fields: [%s]

### -1183 NOT_ALLOWED_IN_DROP_COPY_SESSIONS
 * Requested operation is not allowed in DropCopy sessions.

### -1184 DROP_COPY_SESSION_NOT_ALLOWED
 * DropCopy sessions are not supported on this server. Please reconnect to a drop copy server.

### -1185 DROP_COPY_SESSION_REQUIRED
 * Only DropCopy sessions are supported on this server. Either reconnect to order entry server or send `DropCopyFlag (9406)` field.

### -1186 NOT_ALLOWED_IN_ORDER_ENTRY_SESSIONS
* Requested operation is not allowed in order entry sessions.

### -1187 NOT_ALLOWED_IN_MARKET_DATA_SESSIONS
* Requested operation is not allowed in market data sessions.

### -1188 INCORRECT_NUM_IN_GROUP_COUNT
* Incorrect NumInGroup count for repeating group '%s'.

### -1189 DUPLICATE_ENTRIES_IN_A_GROUP
* Group '%s' contains duplicate entries.

### -1190 INVALID_REQUEST_ID
* `MDReqID (262)` contains a subscription request id that is already in use on this connection.
* `MDReqID (262)` contains an unsubscription request id that does not match any active subscription.

### -1191 TOO_MANY_SUBSCRIPTIONS
* Too many subscriptions. Connection may create up to '%s' subscriptions at a time.
* Similar subscription is already active on this connection. Symbol='%s', active subscription id: '%s'.

### -1194 INVALID_TIME_UNIT
* Invalid value for time unit; expected either MICROSECOND or MILLISECOND.

### -1196 BUY_OCO_STOP_LOSS_MUST_BE_ABOVE
* A stop loss order in a buy OCO must be above.

### -1197 SELL_OCO_STOP_LOSS_MUST_BE_BELOW
* A stop loss order in a sell OCO must be below.

### -1198 BUY_OCO_TAKE_PROFIT_MUST_BE_BELOW
* A take profit order in a buy OCO must be below.

### -1199 SELL_OCO_TAKE_PROFIT_MUST_BE_ABOVE
* A take profit order in a sell OCO must be above.

### -1210 INVALID_PEG_PRICE_TYPE
* Invalid pegPriceType.

### -1211 INVALID_PEG_OFFSET_TYPE
* Invalid pegOffsetType.

### -1220 SYMBOL_DOES_NOT_MATCH_STATUS
* The symbol's status does not match the requested symbolStatus.

### -1221 INVALID_SBE_MESSAGE_FIELD
* Invalid/missing field(s) in SBE message.

### -1222 OPO_WORKING_MUST_BE_BUY

* Working order in an OPO list must be a bid.

### -1223 OPO_PENDING_MUST_BE_SELL

* Pending orders in an OPO list must be asks.

### -1224 WORKING_PARAM_REQUIRED

* Working order must include the '{param}' tag.

### -1225 PENDING_PARAM_NOT_REQUIRED

* Pending orders should not include the '%s' tag.

### -2010 NEW_ORDER_REJECTED
 * NEW_ORDER_REJECTED

### -2011 CANCEL_REJECTED
 * CANCEL_REJECTED

### -2013 NO_SUCH_ORDER
 * Order does not exist.

### -2014 BAD_API_KEY_FMT
 * API-key format invalid.

### -2015 REJECTED_MBX_KEY
 * Invalid API-key, IP, or permissions for action.

### -2016 NO_TRADING_WINDOW
 * No trading window could be found for the symbol. Try ticker/24hrs instead.

### -2026 ORDER_ARCHIVED
  * Order was canceled or expired with no executed qty over 90 days ago and has been archived.

### -2035 SUBSCRIPTION_ACTIVE
  * User Data Stream subscription already active.

### -2036 SUBSCRIPTION_INACTIVE
  * User Data Stream subscription not active.

### -2039 CLIENT_ORDER_ID_INVALID
  * Client order ID is not correct for this order ID.

### -2042 MAXIMUM_SUBSCRIPTION_IDS
* Maximum subscription ID reached for this connection.

<a id="other-errors"></a>

## Messages for -1010 ERROR_MSG_RECEIVED, -2010 NEW_ORDER_REJECTED, -2011 CANCEL_REJECTED, and -2038 ORDER_AMEND_REJECTED
This code is sent when an error has been returned by the matching engine.
The following messages which will indicate the specific error:


Error message                                                   | Description
------------                                                    | ------------
"Unknown order sent."                                           | The order (by either `orderId`, `clOrdId`, `origClOrdId`) could not be found.
"Duplicate order sent."                                         | The `clOrdId` is already in use.
"Market is closed."                                             | The symbol is not trading.
"Account has insufficient balance for requested action."        | Not enough funds to complete the action.
"Market orders are not supported for this symbol."              | `MARKET` is not enabled on the symbol.
"Iceberg orders are not supported for this symbol."             | `icebergQty` is not enabled on the symbol.
"Stop loss orders are not supported for this symbol."           | `STOP_LOSS` is not enabled on the symbol.
"Stop loss limit orders are not supported for this symbol."     | `STOP_LOSS_LIMIT` is not enabled on the symbol.
"Take profit orders are not supported for this symbol."         | `TAKE_PROFIT` is not enabled on the symbol.
"Take profit limit orders are not supported for this symbol."   | `TAKE_PROFIT_LIMIT` is not enabled on the symbol.
"Order amend is not supported for this symbol."                 | Order amend keep priority is not enabled on the symbol.
"Price * QTY is zero or less."                                  | `price` * `quantity` is too low.
"IcebergQty exceeds QTY."                                       | `icebergQty` must be less than the order quantity.
"This action is disabled on this account."                      | Contact customer support; some actions have been disabled on the account.
"This account may not place or cancel orders."                  | Contact customer support; the account has trading ability disabled.
"Unsupported order combination"                                 | The `orderType`, `timeInForce`, `stopPrice`, and/or `icebergQty` combination isn't allowed.
"Order would trigger immediately."                              | The order's stop price is not valid when compared to the last traded price.
"Cancel order is invalid. Check origClOrdId and orderId."       | No `origClOrdId` or `orderId` was sent in.
"Order would immediately match and take."                       | `LIMIT_MAKER` order type would immediately match and trade, and not be a pure maker order.
"The relationship of the prices for the orders is not correct." | The prices set in the `OCO` is breaking the Price restrictions. <br/> For reference: <br/> `BUY` : `LIMIT_MAKER` `price` < Last Traded Price < `stopPrice` <br>`SELL` : `LIMIT_MAKER` `price` > Last Traded Price > `stopPrice`
"OCO orders are not supported for this symbol"                  | `OCO` is not enabled on the symbol.
"Quote order qty market orders are not support for this symbol."| `MARKET` orders using the parameter `quoteOrderQty` are not enabled on the symbol.
"Trailing stop orders are not supported for this symbol."       | Orders using `trailingDelta` are not enabled on the symbol.
"Order cancel-replace is not supported for this symbol."        | `POST /api/v3/order/cancelReplace` (REST API) or `order.cancelReplace` (WebSocket API) is not enabled on the symbol.
"This symbol is not permitted for this account."                | Account and symbol do not have the same permissions. (e.g. `SPOT`, `MARGIN`, etc)
"This symbol is restricted for this account."                   | Account is unable to trade on that symbol. (e.g. An `ISOLATED_MARGIN` account cannot place `SPOT` orders.)
"Order was not canceled due to cancel restrictions."            | Either `cancelRestrictions` was set to `ONLY_NEW` but the order status was not `NEW` <br/> or <br/> `cancelRestrictions` was set to `ONLY_PARTIALLY_FILLED` but the order status was not `PARTIALLY_FILLED`.
"Rest API trading is not enabled." / "WebSocket API trading is not enabled." | Order is being placed or a server that is not configured to allow access to `TRADE` endpoints.
"FIX API trading is not enabled.                                | Order is placed on a FIX server that is not TRADE enabled.
"Order book liquidity is less than `LOT_SIZE` filter minimum quantity." |Quote quantity market orders cannot be placed when the order book liquidity is less than minimum quantity configured for the `LOT_SIZE` filter.
"Order book liquidity is less than `MARKET_LOT_SIZE` filter minimum quantity."|Quote quantity market orders cannot be placed when the order book liquidity is less than the minimum quantity for `MARKET_LOT_SIZE` filter.
"Order book liquidity is less than symbol minimum quantity." | Quote quantity market orders cannot be placed when there are no orders on the book.
"Order amend (quantity increase) is not supported." | `newQty` must be less than the order quantity.
"The requested action would change no state; rejecting". | The request sent would not have changed the status quo.<br></br>(e.g. `newQty` cannot equal the order quantity.)
"Pegged orders are not supported for this symbol." | `pegInstructionsAllowed` has not been enabled. |
"This order type may not use pegged price." | You are using parameter `pegPriceType` with an unsupported order type. (e.g. `MARKET`) |
"This price peg cannot be used with this order type." | You are using `pegPriceType`=`MARKET_PEG` for a `LIMIT_MAKER` order.|
"Order book liquidity is too low for this pegged order." | The order book doesn’t have the best price level to peg the price to. |
| OPO orders are not supported for this symbol. |  |
| Order amend (pending OPO order) is not supported. | You cannot amend the pending quantity of an OPO order |

## Errors regarding placing orders via cancelReplace

### -2021 Order cancel-replace partially failed
* This code is sent when either the cancellation of the order failed or the new order placement failed but not both.

### -2022 Order cancel-replace failed.
* This code is sent when both the cancellation of the order failed and the new order placement failed.

<a id="filter_failures"></a>

## Filter failures
Error message | Description
------------ | ------------
"Filter failure: PRICE_FILTER" | `price` is too high, too low, and/or not following the tick size rule for the symbol.
"Filter failure: PERCENT_PRICE" | `price` is X% too high or X% too low from the average weighted price over the last Y minutes.
"Filter failure: LOT_SIZE" | `quantity` is too high, too low, and/or not following the step size rule for the symbol.
"Filter failure: MIN_NOTIONAL" | `price` * `quantity` is too low to be a valid order for the symbol.
"Filter failure: NOTIONAL"    | `price` * `quantity` is not within range of the `minNotional` and `maxNotional`
"Filter failure: ICEBERG_PARTS" | `ICEBERG` order would break into too many parts; icebergQty is too small.
"Filter failure: MARKET_LOT_SIZE" | `MARKET` order's `quantity` is too high, too low, and/or not following the step size rule for the symbol.
"Filter failure: MAX_POSITION" | The account's position has reached the maximum defined limit. <br/> This is composed of the sum of the balance of the base asset, and the sum of the quantity of all open `BUY` orders.
"Filter failure: MAX_NUM_ORDERS" | Account has too many open orders on the symbol.
"Filter failure: MAX_NUM_ALGO_ORDERS" | Account has too many open stop loss and/or take profit orders on the symbol.
"Filter failure: MAX_NUM_ICEBERG_ORDERS" | Account has too many open iceberg orders on the symbol.
"Filter failure: MAX_NUM_ORDER_AMENDS" | Account has made too many amendments to a single order on the symbol.
"Filter failure: MAX_NUM_ORDER_LISTS" | Account has too many open order lists on the symbol. |
"Filter failure: TRAILING_DELTA" | `trailingDelta` is not within the defined range of the filter for that order type.
"Filter failure: EXCHANGE_MAX_NUM_ORDERS" | Account has too many open orders on the exchange.
"Filter failure: EXCHANGE_MAX_NUM_ALGO_ORDERS" | Account has too many open stop loss and/or take profit orders on the exchange.
"Filter failure: EXCHANGE_MAX_NUM_ICEBERG_ORDERS" | Account has too many open iceberg orders on the exchange.
"Filter failure: EXCHANGE_MAX_NUM_ORDER_LISTS" | Account has too many open order lists on the exchange.
