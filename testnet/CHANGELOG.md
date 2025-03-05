# CHANGELOG for Binance SPOT Testnet

**Last Updated: 2025-03-05**

**Note:** All features here will only apply to the [SPOT Testnet](https://testnet.binance.vision/).
This is not always synced with the live exchange.

## 2025-03-05

* **Notice: This is in the process of being deployed. Please consult the Spot Test Network's [homepage](https://testnet.binance.vision/) to be informed of the release completion.**
  The following request weights will be increased from 2 to 4:
  * REST API: `GET /api/v3/aggTrade`
  * WebSocket API: `trades.aggregate`
* The documentation for both REST and WebSocket API has been updated to reflect the upcoming changes.

---

## 2025-02-28

* **SBE Market Data Streams** are now available. These streams offer a smaller payload and should offer better latency than the equivalent JSON streams for a subset of latency-sensitive market data streams.
* Streams available in SBE format:
  * Real-time: trade stream
  * Real-time: best bid/ask
  * Every 100 ms: diff. depth
  * Every 100 ms: partial book depth
* For more information please refer to the [SBE Market Data Streams](sbe-market-data-streams.md).

---

### 2025-02-05

* **Notice: These changes will be deployed starting at 7:00 UTC, and may take several hours to complete.**
  **The following changes will apply to WebSocket Market Data Streams, User Data Streams, and the WebSocket API:**
    * Our WebSocket services will send a ping frame **every 20 seconds** instead of 3 minutes.
    * The allowed pong delay will be **1 minute** instead of 10 minutes.
    * The documentation for these services have been updated to reflect the change.
* `AggressorSide (2446)` is now rendered in the [FIX Market Data Trade Stream](fix-api.md#tradestream). The QuickFIX schema [file](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml) has also been updated. Please download the latest schema before the Spot Testnet upgrade is completed.

---

### 2024-12-17

* FIX Market Data is now available. The [FIX API](fix-api.md) documentation for SPOT Testnet has been updated regarding this feature.
* Please refer to this [link](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml) for the QuickFIX Schema for FIX Market Data.

---


### 2024-11-27

**Note:** These changes will be deployed live **starting 2024-11-28** and may take several hours for all features to work as intended.

**New Feature: Microsecond support:**

The system now supports microseconds in all related time and/or timestamp fields. Microsecond support is **opt-in**, by default the requests and responses still use milliseconds.<br></br>
Examples in documentation are also using milliseconds for the foreseeable future.

WebSocket Streams

* A new optional parameter `timeUnit` can be used in the connection URL to select the time unit.
  * For example: `/stream?streams=btcusdt@trade&timeUnit=millisecond`
  * Supported values are:
    * `MILLISECOND`
    * `millisecond`
    * `MICROSECOND`
    * `microsecond`
  * If the time unit is not selected, milliseconds will be used by default.

REST API

* A new optional header `X-MBX-TIME-UNIT` can be sent in the request to select the time unit.
  * Supported values:
    * `MILLISECOND`
    * `millisecond`
    * `MICROSECOND`
    * `microsecond`
  * The time unit affects time-related parameters in requests (e.g, `startTime`, `endTime`, `timestamp`).
  * The time unit affects timestamp fields in responses (e.g., `time`, `transactTime`).
  * If the time unit is not selected, milliseconds will be used by default.

WebSocket API

* A new optional parameter `timeUnit` can be used in the connection URL to select the time unit.
  * Supported values:
    * `MILLISECOND`
    * `millisecond`
    * `MICROSECOND`
    * `microsecond`
  * The time unit affects time-related parameters in requests (e.g, `startTime`, `endTime`, `timestamp`).
  * The time unit affects timestamp fields in responses (e.g., `time`, `transactTime`).
  * If the time unit is not selected, milliseconds will be used by default.

User Data Streams

* A new optional parameter `timeUnit` can be used in the connection URL to select the time unit.
  * Supported values
    * `MILLISECOND`
    * `MICROSECOND`.
    * `microsecond`
    * `millisecond`

General Changes:

* Fixed a bug that prevented orders from being placed when submitting OCOs on the `BUY` side without providing a `stopPrice`.
* `TAKE_PROFIT` and `TAKE_PROFIT_LIMIT` support has been added for OCOs.
  * Previously OCOs could only be composed by the following order types:
    * `LIMIT_MAKER` \+ `STOP_LOSS`
    * `LIMIT_MAKER` \+ `STOP_LOSS_LIMIT`
  * Now OCOs can be composed of the following order types:
    * `LIMIT_MAKER` \+ `STOP_LOSS`
    * `LIMIT_MAKER` \+ `STOP_LOSS_LIMIT`
    * `TAKE_PROFIT` \+ `STOP_LOSS`
    * `TAKE_PROFIT` \+ `STOP_LOSS_LIMIT`
    * `TAKE_PROFIT_LIMIT` \+ `STOP_LOSS`
    * `TAKE_PROFIT_LIMIT` \+ `STOP_LOSS_LIMIT`
  * This is supported by the following requests:
    * `POST /api/v3/orderList/oco`
    * `POST /api/v3/orderList/otoco`
    * `orderList.place.oco`
    * `orderList.place.otoco`
    * `NewOrderList<E>`
  * Error code `-1167` will be obsolete after this update and will be removed from the documentation in a later update.
* Timestamp parameters now reject values too far into the past or the future. To be specific, the parameter will be rejected if:
  * `timestamp` before 2017-01-01 (less than 1483228800000\)
  * `timestamp` is more than 10 seconds after the current time (e.g., if current time is 1729745280000 then it is an error to use 1729745291000 or greater)
* If `startTime` and/or `endTime` values are outside of range, the values will be adjusted to fit the correct range.
* The field for quote order quantity (`origQuoteOrderQty`) has been added to responses that previously did not have it. Note that for order placement endpoints the field will only appear for requests with `newOrderRespType` set to `RESULT` or `FULL`.

  * Please refer to the table for requests with `origQuoteOrderQty`:

  | Service | Request |
  | :---- | :---- |
  | REST | `POST /api/v3/order`  |
  |  | `POST /api/v3/sor/order`  |
  |  | `POST /api/v3/order/oco`  |
  |  | `POST /api/v3/orderList/oco`  |
  |  | `POST /api/v3/orderList/oto`  |
  |  | `POST /api/v3/orderList/otoco`  |
  |  | `DELETE /api/v3/order`  |
  |  | `DELETE /api/v3/orderList`  |
  |  | `POST /api/v3/order/cancelReplace` |
  | WebSocket API | `order.place`  |
  |  | `sor.order.place`  |
  |  | `orderList.place`  |
  |  | `orderList.place.oco`  |
  |  | `orderList.place.oto`  |
  |  | `orderList.place.otoco`  |
  |  | `order.cancel`  |
  |  | `orderList.cancel`  |
  |  | `order.cancelReplace` |

SBE

* A new schema 2:1 [spot_2_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_1.xml) has been released. The current schema 2:0 [spot_2_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_0.xml) will thus be deprecated, and retired from the API in 6 months as per our schema deprecation policy.
* Schema 2:1 is a backward compatible update of schema 2:0. You will always receive payloads in 2:1 format when you request either schema 2:0 or 2:1.
* Changes in SBE schema 2:1:
  * New field `origQuoteOrderQty` in order placement/cancellation responses (Note: Decoders generated using the 2:0 schema will skip this field.):
    * `NewOrderResultResponse`
    * `NewOrderFullResponse`
    * `CancelOrderResponse`
    * `NewOrderListResultResponse`
    * `NewOrderListFullResponse`
    * `CancelOrderListResponse`
  * WebSocket API only: New field `userDataStream` in session status responses:
    * `WebSocketSessionLogonResponse`
    * `WebSocketSessionStatusResponse`
    * `WebSocketSessionLogoutResponse`
  * WebSocket API only: New messages for User Data Stream support:
    * `UserDataStreamSubscribeResponse`
    * `UserDataStreamUnsubscribeResponse`
    * `BalanceUpdateEvent`
    * `EventStreamTerminatedEvent`
    * `ExecutionReportEvent`
    * `ExternalLockUpdateEvent`
    * `ListStatusEvent`
    * `OutboundAccountPositionEvent`

WebSocket API

* You can now subscribe to User Data Stream events through your WebSocket API connection.
  * Note: This feature is only available for users of Ed25519 API keys.
  * Note: New SBE schema 2:1 is required for User Data Stream subscriptions in SBE format.
* New requests:
  * `userDataStream.subscribe`
  * `userDataStream.unsubscribe`
* Changes to `session.logon`, `session.status`, and `session.logout`
  * Added a new field `userDataStream` indicating if the user data stream subscription is active.
* Fixed a bug where you wouldn't receive a new listenKey using `userDataStream.start` after `session.logon`

User Data Stream

* WebSocket API only: New event `eventStreamTerminated` is emitted when you either logout from your websocket session or you have unsubscribed from the user data stream.
* New event `externalLockUpdate` is sent when your spot wallet balance is locked/unlocked by an external system.

FIX API

* The [schema](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml) has been updated with a new Administrative message News &lt;B&gt;, which can be used for all FIX services. Receiving this message indicates that your connection is about to be closed.
* **Note:** This message will be available in the live exchange at a later date.


---


### 2024-11-05

**Note:** This is in the process of being deployed. Please consult the Spot Test Network's [homepage](https://testnet.binance.vision/) to be informed of the release completion.

Changes to Exchange Information (i.e. [`GET /api/v3/exchangeInfo`](rest-api.md#exchangeInfo) from REST and [`exchangeInfo`](web-socket-api.md#exchangeInfo) for WebSocket API).

* A new optional parameter `showPermissionSets` can be used to hide the permissions from `permissionsSets`; This can be used for a reduced payload size.
* A new optional parameter `symbolStatus` can now be used to only show symbols with the specified status. (e.g. `TRADING`, `HALT`, `BREAK`)

---

### 2024-10-02

REST and WebSocket API:

* Reminder that SBE 1:0 schema will be disabled on 2024-10-04, [6 months after being deprecated](../faqs/sbe_faq.md), as per our SBE policy.
* The [SBE lifecycle for Testnet](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/sbe_schema_lifecycle_testnet.json) has been updated to reflect this change.

---

### 2024-09-04

* Spot Testnet now supports Unfilled Order Count. Please refer to this [page](../faqs/order_count_decrement.md) on how you can decrement your unfilled order count when placing orders.
* The documentation has been updated to reflect the wording.

---

### 2024-08-16

General Changes:

* New error messages have been added when quote quantity market orders (aka reverse market orders) are rejected in low-liquidity situations.

---

### 2024-08-07

* The [QuickFIX schema](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml) has been modified.

---

### 2024-07-23

**Note:** This will be deployed starting around 7am UTC. Please consult the Spot Test Network's [homepage](https://testnet.binance.vision/) to be informed of the release completion.

* FIX Drop Copy sessions are now supported on the Spot Test Network.
* New API Key permission `FIX_API_READ_ONLY` has been introduced.

---

### 2024-07-17

General changes:

* Fixed a bug where klines had incorrect timestamps.
  * REST API: `GET /api/v3/klines` and `GET /api/v3/uiKlines` with `timeZone` parameter
  * WebSocket API: `klines` and `uiKlines` with `timeZone` parameter
  * WebSocket Streams: `<symbol>@kline_<interval>@+08:00` streams

---

### 2024-06-21

* [FIX API](fix-api.md) will be available today (2024-06-21) on the Spot Test Network. Please consult the Spot Test Network's [homepage](https://testnet.binance.vision/) to be informed of the release completion.
  * Using the FIX API requires an Ed25519 API Key with the `FIX_API` permission.
  * The release date on the live exchange has not been determined.

---

### 2024-06-05

WebSocket Streams

-  Buyer order ID (`b`) and Seller order ID (`a`) have been removed from the Trade streams. (i.e. `<symbol>@trade`)
-  To monitor if your order was part of a trade, please listen to the [User Data Streams](user-data-stream.md).

---

### 2024-05-30

WebSocket API

* `wss://ws-api.testnet.binance.vision/ws-api/v3` is now the primary URL for the Spot Testnet WebSocket API. Other URLs will be phased out over time.

WebSocket Streams

* `wss://stream.testnet.binance.vision/ws` and `wss://stream.testnet.binance.vision/stream` are now the primary URLs for the Spot Testnet WebSocket Streams. Other URLs will be phased out over time.

---

### 2024-05-23

REST API

* `orderRateLimitExceededMode` has been added to `POST /api/v3/order/cancelReplace`

WebSocket API

* `orderRateLimitExceededMode` has been added to `order.cancelReplace`

WebSocket Streams

* Kline/Candlestick streams can now support a UTC+8:00 timezone offset. (e.g. `btcusdt@kline_1d@+08:00`)

---

### 2024-05-02

* One-Triggers-the-Other (OTO) orders and One-Triggers-a-One-Cancels-The-Other (OTOCO) orders are now enabled.
* New requests have been added:
    * REST API:
        * `POST /api/v3/orderList/oto`
        * `POST /api/v3/orderList/otoco`
    * WebSocket API:
        * `orderList.place.oto`
        * `orderList.place.otoco`

---

### 2024-04-04

General changes:

* Symbol permission information in Exchange Information responses has moved from field `permissions` to field `permissionSets`.
* Field `permissions` will be empty and will be removed in a future release.
* Previously, `"permissions":["SPOT","MARGIN"]` meant that you could place an order on the symbol if your account had `SPOT` or `MARGIN` permissions. The equivalent is `"permissionSets":[["SPOT","MARGIN"]]`. (Note the extra set of square brackets.) Each array of permissions inside the `permissionSets` array is called a "permission set".
* Symbol permissions can now be more complex. `"permissionSets":[["SPOT","MARGIN"],["TRD_GRP_004","TRD_GRP_005"]]` means that you may place an order on the symbol if your account has SPOT or MARGIN permissions **and** `TRD_GRP_004` or `TRD_GRP_005` permissions. There may be an arbitrary number of permission sets in a symbol's `permissionSets`.
* **The weight of the following requests has increased from 10 to 25**:
	* `GET /api/v3/trades`
	* `GET /api/v3/historicalTrades`
	* `trades.recent`
	* `trades.historical`

REST API

* The `POST /api/v3/order/oco` endpoint is now deprecated on the REST API. You should use the new `POST /api/v3/orderList/oco` endpoint instead. Note that this new endpoint uses different parameters.
* `POST /api/v3/order/oco` has been removed from the Rest API documentation for SPOT Testnet.
* `otoAllowed` will now appear on `GET /api/v3/exchangeInfo`, that indicates if One-Triggers-the-Other (OTO) orders are supported on that symbol.

WebSocket API

* The `orderList.place` request is now deprecated on the WebSocket API. You should now use the new `orderList.place.oco` request instead. Note that this new request uses different parameters.
* `orderList.place` has been removed from the WebSocket API documentation for SPOT Testnet.
* `otoAllowed` will now appear on `exchangeInfo`, that indicates if One-Triggers-the-Other (OTO) orders are supported on that symbol.

SBE

* A new schema 2:0 [spot_2_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_0.xml) has been released for SPOT Testnet. The current schema, 1:0 [spot_1_0.xml](https://github.com/binance/binance-spot-api-docs/blob/becd4d44a09d94821d2dc761ba9197aae8b495c3/sbe/schemas/spot_1_0.xml), will thus be deprecated and retired from the Testnet APIs in 6 months as per our schema deprecation policy.
* When using schema 1:0 on REST API or WebSocket API, group "permissions" in message "ExchangeInfoResponse" will always be empty. Upgrade to schema 2:0 to find permission information in group "permissionSets". See General changes above for more details.
* Responses for deprecated OCO requests are supported by both schema 1:0 and 2:0



---

### 2024-03-13

General changes:

* `GET /api/v3/account` has a new optional parameter `omitZeroBalances`, allowing to hide all zero balances
* `account.status` has a new optional parameter `omitZeroBalances` allowing to hide all zero balances.


User Data Stream:

* New event `listenKeyExpired` is now emitted when a `listenKey` expires.
