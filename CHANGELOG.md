# CHANGELOG for Binance's API

**Last Updated: 2026-02-02**

### 2026-02-02

* Documented that [FIX Drop Copy session](fix-api.md#fix-api-drop-copy-sessions) data is delayed by 1 second. This has been the delay since the inception of the FIX API.

---

### 2026-01-29

* [Demo Mode](./demo-mode/general-info.md) is now available.
* For information on when the Spot Demo Mode environment will be unavailable due to maintenance, please refer to the [Demo Mode Changelog](./demo-mode/CHANGELOG.md)

---

### 2026-01-27

**Notice: The following changes will occur at 2026-02-11 7:00 UTC**:

* [ICEBERG_PARTS](https://developers.binance.com/docs/binance-spot-api-docs/filters#iceberg_parts) will be increased to 50 for all symbols.

---

### 2026-01-26

* Added undocumented `recvWindow` to `userDataStream.subscribe.signature`.

---

### 2026-01-21

Following the announcement from [2025-10-24](#2025-10-24), the following endpoints/methods will no longer be available starting from **2026-02-20,07:00 UTC**

REST API
* `POST /api/v3/userDataStream`
* `PUT /api/v3/userDataStream`
* `DELETE /api/v3/userDataStream`

WebSocket API

* `userDataStream.start`
* `userDataStream.ping`
* `userDataStream.stop`

---

### 2025-12-18

* Updated [FIX SBE documentation](fix-api.md#fix-sbe)
* Clarified User Data Stream documentation regarding [`eventStreamTerminated`](user-data-stream.md#event-stream-terminated).
* Assets `这是测试币` and `456` and symbol `这是测试币456` have been added to [SPOT Testnet](http://testnet.binance.vision) for testing endpoints/methods with a Unicode symbol. See the [Testnet CHANGELOG](https://developers.binance.com/docs/binance-spot-api-docs/testnet) for more information.

---

### 2025-12-17

#### Time-sensitive Notice

* **The following change to REST API will occur at approximately 2026-01-15 07:00 UTC:** <br>When calling endpoints that require signatures, percent-encode payloads before computing signatures. Requests that do not follow this order will be rejected with [`-1022 INVALID_SIGNATURE`](errors.md#-1022-invalid_signature). Please review and update your signing logic accordingly. This has now been enabled on [SPOT Testnet](http://testnet.binance.vision)

#### REST API

* Updated documentation for REST API regarding [Signed Endpoints examples for placing an order](https://developers.binance.com/docs/binance-spot-api-docs/rest-api/request-security#signed-endpoint-examples-for-post-apiv3order).

#### WebSocket API

* Updated documentation for WebSocket API regarding [SIGNED request security](https://developers.binance.com/docs/binance-spot-api-docs/websocket-api/request-security#signed-request-security).

---

### 2025-12-15

**Clarification Regarding UTF-8 Encoding:**

* In [FIX](fix-api.md), [REST](https://developers.binance.com/docs/binance-spot-api-docs/rest-api/general-api-information), and [WebSocket APIs](https://developers.binance.com/docs/binance-spot-api-docs/websocket-api/general-api-information), if your request contains a symbol name containing non-ASCII characters, then the response may contain non-ASCII characters encoded in UTF-8.
* In REST and WebSocket APIs, some endpoints/methods may return asset and/or symbol names containing non-ASCII characters encoded in UTF-8 even if the request did not contain non-ASCII characters.
* In [WebSocket Streams](web-socket-streams.md), if your request contains a symbol name containing non-ASCII characters, then the stream events may contain non-ASCII characters encoded in UTF-8.
* In WebSocket Streams, [All Market Mini Tickers Stream](web-socket-streams.md#all-market-mini-tickers-stream) and [All Market Rolling Window Statistics Streams](web-socket-streams.md#all-market-rolling-window-statistics-streams) events may contain non-ASCII characters encoded in UTF-8.
* In [SBE Market Data Streams](sbe-market-data-streams.md), if your request contains a symbol name containing non-ASCII characters, then the stream events may contain non-ASCII characters encoded in UTF-8.
* [UserDataStream events](user-data-stream.md) may contain non-ASCII characters encoded in UTF-8 if you own or trade any assets or symbols whose names contain non-ASCII characters.
* For full compatibility with Binance APIs, please ensure your code is designed to handle UTF-8-encoded strings.

---

### 2025-12-09

* [Schema for FIX SBE](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot-fixsbe-1_0.xml) has been updated to use `smallGroupSize16Encoding` in `MarketDataSnapshot` and use `presence="optional"` for incremental book ticker/depth `MDEntrySize` fields.
* Updated documentation re: [FIX vs FIX SBE](fix-api.md#fix-vs-fix-sbe-schema)
* Added documentation in REST, and WebSocket APIs stating: <br>**Please avoid SQL keywords in requests** as they may trigger a security block by a WAF (Web Application Firewall) rule. <br> See https://www.binance.com/en/support/faq/detail/360004492232 for more details.

---

### 2025-12-02

**Notice:** The changes in this section will be gradually rolled out, and will take approximately up to two weeks to complete.

#### General Changes

* Parameter `symbol` and `symbols` can now support Unicode values encoded in UTF-8.
* Following the announcement from [2025-11-14](#2025-11-14), all documentation related to `!ticker@arr` has been removed.
  * The feature will remain available until a future retirement announcement is made.
  * Please use `<symbol>@ticker` or `!miniTicker@arr` instead.

#### FIX API

* Unicode values encoded in UTF-8 can now be accepted in FIX messages. This is allowed for the following tags only:
  * `Currency (15)`
  * `MiscFeeCurr (138)`
  * `Symbol (55)`
  * `SecondarySymbol (25019)`
  * `CounterSymbol (25028)`
  * `SecurityDesc (107)`
* When Unicode is put in a tag value that is not one of the tags above, FIX API will now send back a `RefTagID (371)` tag in the Reject `<3>`, pointing to exactly which tag is not allowed to contain Unicode.
* NewOrderList `<E>` accepts `TriggerPriceDirection (1109)` without `TriggerPrice (1102)`.

#### WebSocket Streams

* WebSocket Market Streams supports URL-encoded urls.
<br>
<br>

**Notice: The following changes will occur at approximately 2025-12-18 7:00 UTC**:
* [ICEBERG_PARTS](https://developers.binance.com/docs/binance-spot-api-docs/filters#iceberg_parts) will be increased to 25 for all symbols.
* [FIX SBE support](fix-api.md) becomes available.
* [One Pays the Other (OPO)](https://github.com/binance/binance-spot-api-docs/blob/master/faqs/opo.md) becomes available on all symbols.
  * `opoAllowed` begins to appear in Exchange Information requests, indicating if One-Pays-the-Other (OPO) orders are supported on each symbol.
    * REST API: `GET /api/v3/exchangeInfo`
    * WebSocket API: `exchangeInfo`
  * New requests for OPO:
    * REST API:
      * `POST /api/v3/orderList/opo`
      * `POST /api/v3/orderList/opoco`
    * WebSocket API
      * `orderList.place.opo`
      * `orderList.place.opoco`
    * FIX API
      * NewOrderList `<E>` has field `OPO (25046)`. Please update to the latest QuickFIX Schema for OPO support.
* STP mode [`TRANSFER`](./faqs/stp_faq.md) has been added. The exact date that STP `TRANSFER` will be enabled has not yet been determined.
* **SBE: A new schema 3:2 ([spot_3_2.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_2.xml)) is available.**
  * The current schema 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml)) is deprecated and will be retired in 6 months as per our schema deprecation policy.
  * Changes in 3:2:
    * New enum variant `TRANSFER` for `selfTradePreventionMode` and `allowedSelfTradePreventionModes`.
    * All schemas below 3:1 are unable to represent any response that could contain the STP mode `TRANSFER` (e.g. Exchange Information, order placement, order cancellation, or querying the status of an order). <br> When a response cannot be represented in the requested schema, an error is returned.
* FIX API changes:
    * `LastFragment (893)` becomes deprecated.
      * This means that the MarketIncrementalRefresh `<X>` messages will no longer be fragmented and may contain more than 10,000 entries.
      * The documentation has been updated to reflect this change.
    * ListStatus `<N>` will no longer emit the optional `symbol` field.
      * This applies to FIX Order Entry and FIX Drop Copy.
      * The documentation has been updated to reflect this change.
---

### 2025-11-14

* All Market Tickers Stream (`!ticker@arr`) has been deprecated; This means this will be removed both from the documentation and from our systems at a later date. More details to follow.
* Please use [`<symbol>@ticker`](https://developers.binance.com/docs/binance-spot-api-docs/web-socket-streams#individual-symbol-ticker-streams) or [`!miniTicker@arr`](https://developers.binance.com/docs/binance-spot-api-docs/web-socket-streams#all-market-mini-tickers-stream) instead.

---

### 2025-11-12

* The steps on [how to manage a local order book correctly](https://developers.binance.com/docs/binance-spot-api-docs/web-socket-streams#how-to-manage-a-local-order-book-correctly) has been corrected.

---

### 2025-11-11

#### SBE Market Data

* **At 2025-11-26 07:00 UTC, the update speed of `<symbol>@depth` and `<symbol>@depth20` streams will be changed to 50ms**.
  * This change will apply automatically to all users of SBE Market Data and doesn't require any action.
  * The total amount of data received per second will be increased (up to 2x).
  * [SPOT Testnet](https://testnet.binance.vision/) will have these changes at **2025-11-11 07:00 UTC**.
  * [SBE Market Data](sbe-market-data-streams.md) has been updated to reflect these changes.

---

### 2025-11-10

* "Last Updated" dates will be removed from all documents except for CHANGELOG.
* Moving forward, CHANGELOG will be the source of reference for when changes were made to any document.

---

### 2025-10-28

**Notice: The following changes will be deployed on 2025-10-28, starting at 04:00 UTC and may take several hours to complete:**

* An optional parameter, `symbolStatus`, has been added to the following endpoints:
    * **REST API**
      * `GET /api/v3/depth`
      * `GET /api/v3/ticker/price`
      * `GET /api/v3/ticker/bookTicker`
      * `GET /api/v3/ticker/24hr`
      * `GET /api/v3/ticker/tradingDay`
      * `GET /api/v3/ticker`
    * **WebSocket API**
      * `depth`
      * `ticker.price`
      * `ticker.book`
      * `ticker.24hr`
      * `ticker.tradingDay`
      * `ticker`
* When the parameter `symbolStatus=<STATUS>` is provided, only symbols whose trading status matches the specified `STATUS` will be included in the response:_
    * If a single symbol is specified using the `symbol=<SYMBOL>` parameter and its trading status does not match the given `STATUS`, the endpoint will return error code [`-1220 SYMBOL_DOES_NOT_MATCH_STATUS`](./errors.md#-1220-symbol_does_not_match_status).
    * If multiple symbols are specified using the `symbols=[...]` parameter, the response will be an array that excludes any symbols whose trading status does not match `STATUS`. If no symbols from the symbols parameter have a trading status that matches `STATUS`, the response is an empty array.
    * For endpoints where the `symbol` and `symbols` parameters are optional, omitting these parameters is treated as if all symbols had been specified in the `symbols=[...]` parameter. See the previous line for the behavior of `symbolStatus=<STATUS>`.

---

### 2025-10-24

#### SBE

* SBE: schema 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml)) has been updated to support [listenToken Subscription Methods](https://developers.binance.com/docs/margin_trading/trade-data-stream/Listen-Token-Websocket-API) for Margin Trading.

#### REST and WebSocket API

Following the announcement from [2025-04-07](#2025-04-07), all documentation related with `listenKey` for use on `wss://stream.binance.com` has been removed.

**We remind you that you should instead get user data updates by subscribing to the [User Data Stream on the WebSocket API](https://developers.binance.com/docs/binance-spot-api-docs/websocket-api/user-data-stream-requests). This will offer better performance (lower latency).**

Please refer to the list of requests and methods below for more information.

The features will remain available until a future retirement announcement is made.

* REST API
  * `POST /api/v3/userDataStream`
  * `PUT /api/v3/userDataStream`
  * `DELETE /api/v3/userDataStream`

* WebSocket API
  * `userDataStream.start`
  * `userDataStream.ping`
  * `userDataStream.stop`

---

### 2025-10-21

REST and WebSocket API:

* Reminder that SBE 2:1 schema will be retired on 2025-10-24, [6 months after being deprecated](faqs/sbe_faq.md#sbe-schema).
* The [SBE lifecycle for Production](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/sbe_schema_lifecycle_prod.json) has been updated to reflect this change.

---

### 2025-10-08

#### FIX API

* Updated [QuickFIX Schema](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml) for FIX Market Data:
  * Updated `RecvWindow(25000)` to reflect microsecond support announced on [2025-08-12](#2025-08-12).
  * Updated [InstrumentList `<y>`](fix-api.md#instrumentlist) message:
    * Added fields: `StartPriceRange`, `EndPriceRange`.
    * Made the following fields optional: `MinTradeVol(562)`, `MaxTradeVol(1140)`, `MinQtyIncrement(25039)`, `MarketMinTradeVol(25040)`, `MarketMaxTradeVol(25041)`, `MarketMinQtyIncrement(25042)`, `MinPriceIncrement(969)`.
  * **The changes to InstrumentList <y> are breaking changes, and will roll out at around 2025-10-23 07:00 UTC. Please update to the new schema before then.**
  * [SPOT Testnet](https://testnet.binance.vision/) has the breaking changes already enabled.

---

### 2025-09-29

**Notice: The following changes will be deployed on 2025-09-29, starting at 10:00 UTC and may take several hours to complete.**

* Added an endpoint to retrieve the list of filters relevant to an account on a given symbol. This is the only endpoint that shows if an account has `MAX_ASSET` filters applied to it.
  * REST API: [`GET /api/v3/myFilters`](rest-api.md#myFilters)
  * WebSocket API: [`myFilters`](web-socket-api.md#myFilters)
* Comments in **SBE: schema 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml))** have been added, modified, and removed. Although there is no need for users of `3:1` to update to this version of the file, we advise updating to maintain consistency.
* Added documentation for filter [`MAX_ASSET`](filters.md#max_asset).

---

### 2025-09-18

* Updated documentation for `recvWindow` to reflect microsecond support announced on [2025-08-12](#2025-08-12).
  * REST API: [Timing Security](rest-api.md#timingsecurity)
  * WebSocket API: [Timing Security](web-socket-api.md#timingsecurity)

---

### 2025-09-12

* The [QuickFix schema for FIX Order Entry](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml) has been updated to support Pegged Orders.
* Updated FIX API Documentation for `RecvWindow` in
  * [Message Components](fix-api.md#header)
  * [Timing Security](fix-api.md#timing-security)

---

### 2025-08-28

* Updated SBE FAQ section [regarding legacy support](faqs/sbe_faq.md#regarding-legacy-support) to include more details on schema compatibility and explain `NonRepresentable` and `NonRepresentableMessage`.

---

### 2025-08-26

* Updated "Request Security" documentation for [REST API](rest-api.md#request-security) and [WebSocket API](web-socket-api.md#request-security) with no functional changes.

---

### 2025-08-25

* **SBE: schema 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml))** will be updated on **2025-08-25 at 05:00 UTC**
  * The following fields have been renamed because the [SbeTool](faqs/sbe_faq.md#generate-sbe-decoders) code generator has been found to generate Java code that does not compile.
    * Although only users impacted by this issue need to update the schema, we advise all users to upgrade to the latest version to maintain consistency.
    * Message `MaxAssetFilter`
      * field `limitExponent` renamed to `qtyExponent`
      * field `limit` renamed to `maxQty`

---

### 2025-08-19

* `userDataStream.subscribe` returns `subscriptionId` in the responses. <br> This was missed in the [previous](#2025-08-12) changelog entry.

---

### 2025-08-12

**Notice:** The changes in this section will be gradually rolled out, and will take approximately up to two weeks to complete.

#### General Changes

* New error codes `-1120` and `1211`. See [Errors](errors.md) for more information.
* The following requests have a new structure called `specialCommission`. See [Commission Rates](faqs/commission_faq.md).
  * REST API
    * `GET /api/v3/account/commission`
    * `POST /api/v3/order/test` with `computeCommissionRates=true`
    * `POST /api/v3/sor/order/test` with `computeCommissionRates=true`
  * WebSocket API
    * `account.commission`
    * `order.test` with `computeCommissionRates=true`
    * `sor.order.test` with `computeCommissionRates=true`
* **SBE: A new schema 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml)) is available.**
  * The current schema 3:0 ([spot_3_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_0.xml)) is deprecated and will retire in 6 months as per our schema deprecation policy.
  * Changes in schema 3:1:
    * `ExchangeInfoResponse`: new field `pegInstructionsAllowed`
    * `ExecutionReportEvent`: new fields `pricePeg`, `pricePegOffsetLevel`, `peggedPrice`
    * `UserDataStreamSubscribeResponse`: new field `subscriptionId`
    * New field `subscriptionId` for all user data stream events.
    * Field `apiKey` renamed to `loggedOnApiKey` for `WebSocketSessionLogonResponse`, `WebSocketSessionStatusResponse` and `WebSocketSessionLogoutResponse`
    * `OrderTestWithCommissionsResponse`: 2 new fields `specialCommissionForOrderMaker` and `specialCommissionForOrderTaker`
    * `AccountCommissionResponse`: 4 new fields `specialCommissionMaker`, `specialCommissionTaker`, `specialCommissionBuyer` and `specialCommissionSeller`
    * Support for `EXCHANGE_MAX_NUM_ORDER_LISTS`, `MAX_NUM_ORDER_LISTS`, and `MAX_NUM_ORDER_AMENDS` filters.
    * `ExecutionReportEvent`: fields `rejectReason` and `origClientOrderId` now show their default values in SBE format to match the JSON format.
    * `NonRepresentableMessage`: New message added to represent a message that cannot be represented in this schema ID and version. Receipt of this message indicates that something should be available, but it is not representable using the SBE schema currently in use.
* Orders with cumulative quantity of 0 in the final state `EXPIRED_IN_MATCH` (i.e., the order expired due to STP) will be archived after 90 days.
* Query order lists requests will first query the data in the cache, and if it cannot be found, will query the database.
  * REST API: `GET /api/v3/openOrderLists`
  * WebSocket API: `openOrderLists.status`

#### WebSocket API

* A single WebSocket connection can subscribe to multiple User Data Streams at once.
  * Only one subscription per account is allowed on a single connection.
* Method `userDataStream.subscribe.signature` has been added that allows you to subscribe to the User Data Stream without needing to login first.
  * This also doesn’t require an Ed25519 API Key, and can work with any [API Key type](faqs/api_key_types.md).
  * For [SBE support](faqs/sbe_faq.md), you need to use at least schema 3:1.
* Method `session.subscriptions` has been added to list all active subscriptions for the current session.
* The meaning of the field `userDataStream` in the session requests has changed slightly.
  * Previously, this returned `true` if you were subscribed to the user data stream of your logged-on account.
  * Now returns `true` if you have at least one active user data stream subscription, otherwise `false`.
* `userDataStream.unsubscribe` supports closing multiple subscriptions.
  * When called with no parameter, this will close all subscriptions.
  * When called with `subscriptionId`, this will attempt to close the subscription matching that Id, if it exists.
  * The authorization for this request has been changed to `NONE`.
* Field `subscriptionId` has been added to the User Data Stream events payload when listening through the [WebSocket API](web-socket-api.md#user_data_stream_subscribe). This will identify which subscription the event is coming from.

#### FIX API

* When a client sends a reject message, the FIX API will no longer send the client back a Reject `<3>` message.
* Error messages are now clearer when a tag is invalid, missing a value, or when the field value is empty or malformed.
  * ```json
    { "code": -1169, "msg": "Invalid tag number." }
    ```
  * ```json
    { "code": -1177, "msg": "Tag specified without a value." }
    ```
  * ```json
    { "code": -1102, "msg": "Field value was empty or malformed." }
    ```

#### Future Changes

The following changes will be available on **2025-08-27 starting at 07:00 UTC**:
* Exchange Information requests will emit a new field, `pegInstructionsAllowed`.
* Bug fix: The Matching Engine will no longer accept order lists that exceed the order count filter limits. Affected filters:
  * `MAX_NUM_ORDERS`
  * `MAX_ALGO_ORDERS`
  * `MAX_ICEBERG_ORDERS`
  * `EXCHANGE_MAX_NUM_ORDERS`
  * `EXCHANGE_MAX_ALGO_ORDERS`
  * `EXCHANGE_MAX_ICEBERG_ORDERS`

The following changes will be available on **2025-08-28 starting at 07:00 UTC**:
* The [pegged orders](faqs/pegged_orders.md) functionality will be available.
  * `pegInstructionsAllowed` will be set to `true` for all symbols, enabling the use of pegged orders for all APIs.
  * The following conditional fields `pegPriceType`, `pegOffSetType`, `pegOffsetValues`, and `peggedPrice` will appear in responses of the following requests if the order is a pegged order:
    * REST API
      * `GET /api/v3/order`
      * `GET /api/v3/orderList`
      * `GET /api/v3/openOrderList`
      * `GET /api/v3/allOrders`
      * `DELETE /api/v3/order`
      * `DELETE /api/v3/orderList`
      * `DELETE /api/v3/openOrders`
      * `PUT /api/v3/order/amend/keepPriority`
    * WebSocket API
      * `order.status`
      * `orderList.status`
      * `allOrders`
      * `order.cancel`
      * `orderList.cancel`
      * `openOrders.cancelAll`
      * `order.amend.keepPriority`
  * FIX API
    * `OrdType(40)` supports new value `P(PEGGED)`
    * Tags `PegOffsetValue(211)`, `PegPriceType(1094)`, `PegMoveType(835)`, and `PegOffsetType(836)` have been added to the following messages:
      * NewOrderSingle `<D>`
      * NewOrderList `<E>`
      * OrderCancelRequestAndNewOrderSingle `<XCN>`
    * When placing an order, the `ExecutionReport` `<8>` message will echo back `PegInstructions`, with an extra optional field `PeggedPrice (839)`.
  * New error messages for pegged orders are added. Please see the [Errors](errors.md) document for more information.
* Changes with `recvWindow` will be enabled.
  * A third check is performed after your message leaves our message broker, just before it is sent to the Matching Engine.
    * This does not cover potential delays inside the Matching Engine itself.
  * `recvWindow` supports microseconds.
    * The value is still specified in milliseconds, but it can now include a decimal component for higher precision.
    * This means the parameter can now support up to **three decimal places** (e.g., 6000.346).
    * APIs affected:
      * FIX API
      * REST API
      * WebSocket API
* New [`MAX_NUM_ORDER_LISTS`](filters.md#max-num-order-lists) filter will be enabled, limiting the number of order lists to 20 per symbol.
* New [`MAX_NUM_ORDER_AMENDS`](filters.md#max_num_order_amends) filter will be enabled, limiting each order to a maximum of 10 amendments.


---

### 2025-08-07

* Updated FIX API documentation
  * [FIX Market Data limits](fix-api.md#connection-limits): The subscription limit has always been present but was undocumented.
  * [On message processing order](fix-api.md#on-message-processing-order): Reworded and reformatted.

---

### 2025-07-03

* Beginning at **2025-07-08 07:00 UTC**, [WebSocket Streams](web-socket-streams.md) will be upgraded.
* During the upgrade, **existing and new connections may be disconnected in less than 24 hours**.
* The upgrade may take up to 2 hours; We apologize for the inconvenience.

---

### 2025-06-04

REST and WebSocket API:

* Reminder that SBE 2:0 schema will be retired on 2025-06-12, [6 months after being deprecated](faqs/sbe_faq.md#sbe-schema).
* The [SBE lifecycle for Production](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/sbe_schema_lifecycle_prod.json) has been updated to reflect this change.

---

### 2025-05-28

* Documented API timeout value and error under General API Information for each API:
    * [FIX](fix-api.md#general-api-information)
    * [REST](rest-api.md#general-api-information)
    * [WebSocket](web-socket-api.md#general-api-information)

---

### 2025-05-22

**Notice: The following changes will happen at 2025-06-06 7:00 UTC.**

* The previous behavior of `recvWindow` on FIX, REST, and WebSocket APIs will be augmented by an additional check.
  * To review, the existing behavior is:
    * If `timestamp` is greater than `serverTime` + 1 second at receipt of the request, the request is rejected. Rejection by this check increments message limits (FIX API) and IP limits (REST and WebSocket APIs), but not Unfilled Order Count (order placement endpoints of all APIs).
    * If the difference between `timestamp` and `serverTime` at receipt of the request is greater than `recvWindow`, the request is rejected. Rejection by this check increments message limits (FIX API) and IP limits (REST and WebSocket APIs) but not Unfilled Order Count (order placement endpoints of all APIs).
  * The additional check is:
    * Just before a request is forwarded to the Matching Engine, if the difference between `timestamp` and the current `serverTime` is greater than `recvWindow`, the request is rejected. Rejection by this check increments message limits (FIX API), IP limits (REST and WebSocket APIs), and Unfilled Order Count (order placement endpoints of all APIs).
  * The documentation for Timing security has been updated to reflect the additional check.
    * [REST API](rest-api.md#timing-security)
    * [WebSocket API](web-socket-api.md#timing-security)
    * [FIX API](fix-api.md#timing-security)
* Fixed a bug in FIX Market Data message InstrumentList `<y>`. Previously, the value of `NoRelatedSym(146)` could have been incorrect.

---

### 2025-04-29

* Features that currently require an Ed25519 API key will soon be opened up to HMAC and RSA keys.
  * For example, subscribing to User Data Stream in WebSocket API will be possible with any API key type before listenKeys are removed.
  * Users are still encouraged to migrate to Ed25519 API keys as they are more secure and performant on Binance Spot Trading.
  * More details to come.

---

### 2025-04-25

* The following request weights have been increased from 1 to 4:
  * REST API: `PUT /api/v3/order/amend/keepPriority`
  * WebSocket API: `order.amend.keepPriority`
  * The documentation for both REST and WebSocket API has been updated to reflect these changes.
* Clarified that `SEQNUM` in the FIX-API is a 32-bit unsigned integer that rolls over. This has been the `SEQNUM` data type since the inception of the FIX-API.

---

### 2025-04-21

**Clarification on the release of [Order Amend Keep Priority](./faqs/order_amend_keep_priority.md) and [STP Decrement](./faqs/stp_faq.md):**
* At **2025-05-07 07:00 UTC**
  * Order Amend Keep Priority will be enabled on all symbols.
  * STP Decrement will be allowed on all symbols.
* At **2025-04-24, 07:00 UTC**, the field `amendAllowed` will become visible on Exchange Information requests, but the feature will not be enabled yet.
* [SPOT Testnet](https://testnet.binance.vision/) has both features enabled/allowed on all symbols.

---

### 2025-04-08

**Notice:** The changes in this section will be gradually rolled out, and will take a week to complete.

* New Error code `-2039` where if querying an order with both `orderId` and `origClientOrderId` and no order is found with this combination.
  * Affected requests:
    * REST API: `GET /api/v3/order`
    * WebSocket API: `order.status`
* The [Errors Documentation](errors.md) has also been updated with the new error messages for code `-1034` when the FIX connection rate limits have exceeded. (More details can be found in yesterday's [update](#2025-04-07))

---

### 2025-04-07

#### General Changes

**Notice:** The changes in this section will be gradually rolled out, and will take a week to complete.

* FIX Market Data connection limits were increased from 5 to 100 on January 16, 2025. This was not previously highlighted in changelog.
* New Error code `-2038` for order amend keep priority requests that fail.
* New messages for error code `-1034`.
* If the unfilled order count for `intervalNum:DAY` is exceeded, the unfilled order count for `intervalNum:SECOND` is no longer incremented.
* Previously, the request weight for myTrades was 20 regardless of the parameters provided. Now, if you provide `orderId`, the request weight is 5.
  * REST API: `GET /api/v3/myTrades`
  * WebSocket API: `myTrades`
* Change when querying and deleting orders:
  * When neither `orderId` nor `origClientOrderId` are present, the request is now rejected with `-1102` instead of `-1128`.
  * Affected requests:
    * REST API:
      * `GET /api/v3/order`
      * `DELETE /api/v3/order`
    * WebSocket API
      * `order.status`
      * `order.cancel`
    * FIX API
      * OrderCancelRequest `<F>`

#### FIX API

**Notice:** The following changes will occur during April 21, 2025.

* FIX API verifies that `EncryptMethod(98)` is 0 at Logon `<A>`.
* FIX Order Entry connection limits will be a maximum of 10 concurrent connections per account.
* The connection rate limits are now enforced. Note that these limits are checked independently for both the account and the IP address.
    * FIX Order Entry: 15 connection attempts within 30 seconds
    * FIX Drop Copy: 15 connection attempts within 30 seconds
    * FIX Market Data: 300 connection attempts within 300 seconds
* News `<B>` contains a countdown until disconnection in the Headline field.
    * Following the completion of this update, when the server enters maintenance, a `News` message will be sent to clients **every 10 seconds for 10 minutes**. After this period, clients will be logged out and their sessions will be closed.
* OrderCancelRequest `<F>` and OrderCancelRequestAndNewOrderSingle `<XCN>` now allow both `orderId` and `clientOrderId`.
* The [QuickFix schema for FIX OE](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml) is updated to support the Order Amend Keep Priority feature and new STP mode, `DECREMENT`.

#### User Data Streams

* **Receiving user data streams on wss://stream.binance.com:9443 using a `listenKey` is now deprecated.**
    * This feature will be removed from our systems at a later date.
* **Instead, you should get user data updates by subscribing to the [User Data Stream on the WebSocket API](https://developers.binance.com/docs/binance-spot-api-docs/websocket-api/user-data-stream-requests)**.
    * This should offer slightly better performance **(lower latency)**.
    * This requires the use of an Ed25519 API Key.
* In a future update, information about the base WebSocket endpoint for the User Data Streams will be removed.
* In a future update, the following requests will be removed from the documentation:
    * `POST /api/v3/userDataStream`
    * `PUT /api/v3/userDataStream`
    * `DELETE /api/v3/userDataStream`
    * `userDataStream.start`
    * `userDataStream.ping`
    * `userDataStream.stop`
* The [User Data Stream documentation](user-data-stream.md) will remain as reference for the payloads you can receive.

#### Future Changes

The following changes will occur at **April 24, 2025, 07:00 UTC**:

* ~~[Order Amend Keep Priority](https://github.com/binance/binance-spot-api-docs/blob/master/faqs/order_amend_keep_priority.md) becomes available. (Note that the symbol has to have the feature enabled to be used.)~~
  * **UPDATE 2025-04-21: The exact date "Order Amend Keep Priority" will be enabled has not yet been determined.**
  * New field `amendAllowed` becomes visible in Exchange Information responses.
    * REST API: `GET /api/v3/exchangeInfo`
    * WebSocket API: `exchangeInfo`
  * FIX API: New Order Entry Messages **OrderAmendKeepPriorityRequest** and **OrderAmendReject**
  * REST API: `PUT /api/v3/order/amend/keepPriority`
  * WebSocket API: `order.amend.keepPriority`
* ~~STP mode `DECREMENT` becomes visible in Exchange Information if the symbol has it configured.~~
  * **UPDATE 2025-04-21: The exact date `DECREMENT` STP will be enabled has not yet been determined.**
  * Instead of expiring only the maker, only the taker, or unconditionally both orders, STP decrement decreases the available quantity of **both** orders and increases the `prevented quantity` of **both** orders by the amount of the prevented match.
  * This expires the order with less available quantity as (`filled quantity` \+ `prevented quantity`) equals `order quantity`. Both orders expire if their available quantities are equal. It is called a "decrement" because it reduces available quantity.
* Behavior when querying and/or canceling with `orderId` and `origClientOrderId/cancelOrigClientOrderId`:
  * The behavior when both parameters were provided was not consistent across all endpoints.
  * Moving forward, when both parameters are provided, the order is first searched for using its `orderId`, and if found, `origClientOrderId`/`cancelOrigClientOrderId` is checked against that order. If both conditions pass, the request succeeds. If both conditions are not met the request is rejected.
  * Affected requests:
    * REST API:
      * `GET /api/v3/order`
      * `DELETE /api/v3/order`
      * `POST /api/v3/order/cancelReplace`
    * WebSocket API:
      * `order.status`
      * `order.cancel`
      * `order.cancelReplace`
    * FIX API
      * OrderCancelRequest `<F>`
      * OrderCancelRequestAndNewOrderSingle `<XCN>`
* Behavior when canceling with `listOrderId` and `listClientOrderId`:
  * The behavior when both parameters were provided was not consistent across all endpoints.
  * Moving forward, when both parameters are passed, the order list is first searched for using its `listOrderId`, and if found, `listClientOrderId` is checked against that order list. If both conditions are not met the request is rejected.
  * Affected requests:
    * REST API
      * `DELETE /api/v3/orderList`
    * WebSocket API
      * `orderList.cancel`
* **SBE: A new schema 3:0 ([spot_3_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_0.xml)) is now available.**
  * The current schema 2:1 ([spot_2_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_1.xml)) is now deprecated and will be retired in 6 months as per our schema deprecation policy.
  * Note that trying to use schema 3:0 before it is released will result in an error.
  * Changes in schema 3:0:
      * Support for Order Amend Keep Priority:
          * Added field `amendAllowed` to ExchangeInfoResponse.
          * New Messages `OrderAmendmentsResponse` and `OrderAmendKeepPriorityResponse`
      * All enums now have a `NON_REPRESENTABLE` variant. This will be used to encode new enum values in the future, which would be incompatible with 3:0.
      * New enum variant `DECREMENT` for `selfTradePreventionMode` and `allowedSelfTradePreventionModes`
      * `symbolStatus` enum values `AUCTION_MATCH`, `PRE_TRADING` and `POST_TRADING` have been removed.
      * Fields `usedSor`, `orderCapacity`, `workingFloor`, `preventedQuantity`, and `matchType` are no longer optional.
      * Field `orderCreationTime` in `ExecutionReportEvent` is now optional.
   * When using deprecated schema 2:1 on the WebSocket API to listen to the User Data Stream:
      * `ListStatusEvent` field `listStatusType` will be rendered as `ExecStarted` when it should have been `Updated`. Upgrade to schema 3:0 to get the correct value.
      * `ExecutionReportEvent` field `selfTradePreventionMode` will be rendered as `None` when it should have been `Decrement`. This only happens when `executionType` is `TradePrevention`.
      * `ExecutionReportEvent`  field `orderCreationTime` will be rendered as -1 when it has no value.
    * All schemas below 3:0 are unable to represent responses for Order Amend Keep Priority requests and any response that could contain the STP mode `DECREMENT` (e.g. Exchange Information, order placement, order cancelation, or querying the status of your order). When a response cannot be represented in the requested schema, an error is returned.

---

### 2025-04-03

* Following SPOT Testnet's latest announcement, updating the URL in the WebSocket API to the latest URL for [SPOT Testnet](https://testnet.binance.vision/).

---

### 2025-03-31

* Added a clarification on the performance of canceling an order.

---

### 2025-03-10

* **Notice: The following changes will happen on 2025-03-13 09:00 UTC**
  * FIX Drop Copy sessions will have a limit of **60 messages per minute**.
  * FIX Market Data sessions will have a limit of **2000 messages per minute**.
  * The FIX API documentation has been updated to reflect the upcoming changes.
* **SBE Market Data Streams will be available on March 18 2025, 07:00 UTC.** These streams offer a smaller payload and should offer better latency than the equivalent JSON streams for a subset of latency-sensitive market data streams.
  * Streams available in SBE format:
    * Real-time: trade stream
    * Real-time: best bid/ask
    * Every 100 ms: diff. depth
    * Every 100 ms: partial book depth
  * For more information please refer to the [SBE Market Data Streams](sbe-market-data-streams.md).

---

### 2025-03-05

* **Notice: The following changes will happen on March 10, 2025 12:00 UTC.** <br>
  The following request weights will be increased from 2 to 4:
  * REST API: `GET /api/v3/aggTrade`
  * WebSocket API: `trades.aggregate`
* The documentation for both REST and WebSocket API has been updated to reflect the upcoming changes.

---

### 2025-02-12

* **Notice: These changes will take effect on February 26, 2025 05:00 UTC.** Please ensure you have downloaded the latest schema before then.
* `AggressorSide (2446)` will be rendered in the [FIX Market Data Trade Stream](fix-api.md#tradestream). The QuickFix schema [file](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml) has also been updated.

---

### 2025-01-28

* **Notice: These changes will be gradually rolled out between February 3, 2025 and February 14, 2025.**
  **The following changes will apply to WebSocket Market Data Streams, User Data Streams, and the WebSocket API:**
    * Our WebSocket services will send a ping frame **every 20 seconds** instead of 3 minutes.
    * The allowed pong delay will be **every 1 minute** instead of 10 minutes.
    * The documentation for these services have been updated to reflect the change.

---

### 2025-01-09

* FIX Market Data will be available at **January 16, 05:00 UTC**. The FIX API documentation has been updated regarding this feature.
* Please refer to this [link](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml) for the QuickFIX Schema for FIX Market Data.

---

### 2024-12-17

General Changes:

The system now supports microseconds in all related time and/or timestamp fields. Microsecond support is **opt-in**, by default the requests and responses still use milliseconds. Examples in documentation are also using milliseconds for the foreseeable future.

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
  * The time unit affects timestamp fields in JSON responses (e.g., `time`, `transactTime`).
    * SBE responses continue to be in microseconds regardless of time unit.
  * If the time unit is not selected, milliseconds will be used by default.
* Timestamp parameters (e.g. `startTime`, `endTime`, `timestamp)` can now be passed in milliseconds or microseconds.

WebSocket API

* A new optional parameter `timeUnit` can be used in the connection URL to select the time unit.
  * Supported values:
    * `MILLISECOND`
    * `millisecond`
    * `MICROSECOND`
    * `microsecond`
  * The time unit affects timestamp fields in JSON responses (e.g., `time`, `transactTime`).
    * SBE responses continue to be in microseconds regardless of time unit.
  * If the time unit is not selected, milliseconds will be used by default.
* Timestamp parameters (e.g. `startTime`, `endTime`, `timestamp)` can now be passed in milliseconds or microseconds.

User Data Streams

* A new optional parameter `timeUnit` can be used in the connection URL to select the time unit.
  * Supported values
    * `MILLISECOND`
    * `MICROSECOND`.
    * `microsecond`
    * `millisecond`

---

### 2024-12-09

**Notice:** The changes below will be rolled out starting at **2024-12-12** and may take approximately a week to complete.

General Changes

* Timestamp parameters now reject values too far into the past or the future. To be specific, the parameter will be rejected if:
  * `timestamp` before 2017-01-01 (less than 1483228800000)
  * `timestamp` is more than 10 seconds after the current time (e.g., if current time is 1729745280000 then it is an error to use 1729745291000 or greater)
* If `startTime` and/or `endTime` values are outside of range, the values will be adjusted to fit the correct range.
* The field for quote order quantity (`origQuoteOrderQty`) has been added to responses that previously did not have it. Note that for order placement endpoints the field will only appear for requests with `newOrderRespType` set to `RESULT` or `FULL`.
  * Please refer to the table below for affected requests with: `origQuoteOrderQty`:

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
  * Note: This feature is only available for users of the Ed25519 API keys.
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

* The [schema](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml) has been updated with a new Administrative message News \<B\>, which can be used for all FIX services. Receiving this message indicates that your connection is about to be closed.

The following changes will occur **between 2024-12-16 to 2024-12-20**:

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
  * Error code \-1167 will be obsolete after this update and will be removed from the documentation in a later update.

---

### 2024-10-18

REST and WebSocket API:

* Reminder that SBE 1:0 schema will be disabled on 2024-10-25, [6 months after being deprecated](./faqs/sbe_faq.md), as per our SBE policy.
* The [SBE lifecycle for Prod](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/sbe_schema_lifecycle_prod.json) has been updated to reflect this change.

---

### 2024-10-17


Changes to Exchange Information (i.e. [`GET /api/v3/exchangeInfo`](rest-api.md#exchangeInfo) from REST and [`exchangeInfo`](web-socket-api.md#exchangeInfo) for WebSocket API).

* A new optional parameter `showPermissionSets` can be used to hide the permissions from `permissionsSets`; This can be used for a reduced payload size.
* A new optional parameter `symbolStatus` can now be used to only show symbols with the specified status. (e.g. `TRADING`, `HALT`, `BREAK`)

---

### 2024-08-26

* [Spot Unfilled Order Count Rules](./faqs/order_count_decrement.md) have been updated to explain how to decrease your unfilled order count when placing orders.

---

### 2024-08-16

**Notice:** The changes below are being rolled out gradually, and may take approximately a week to complete.

General Changes:
* New error messages have been added when quote quantity market orders (aka reverse market orders) are rejected in low-liquidity situations.

---

### 2024-08-01

* [FIX API and Drop Copy Sessions](fix-api.md) will be available on **August 8, 05:00 UTC**.

---

### 2024-07-26

* [FIX API and Drop Copy Sessions](fix-api.md) has been added to the documentation.
* The release date to the live exchange has not been determined.


---

### 2024-07-22

General changes:

* Fixed a bug where klines had incorrect timestamps.
  * REST API: `GET /api/v3/klines` and `GET /api/v3/uiKlines` with `timeZone` parameter
  * WebSocket API: `klines` and `uiKlines` with `timeZone` parameter
  * WebSocket Streams: `<symbol>@kline_<interval>@+08:00` streams

---

### 2024-06-11

* On **June 11, 05:00 UTC**, One-Triggers-the-Other (OTO) orders and One-Triggers-a-One-Cancels-The-Other (OTOCO) orders will be enabled. (Note this may take a few hours to be rolled out to all servers.)
    * New requests have been added:
        * REST API:
            * `POST /api/v3/orderList/oto`
            * `POST /api/v3/orderList/otoco`
        * WebSocket API:
            * `orderList.place.oto`
            * `orderList.place.otoco`
* On **June 18, 05:00 UTC**, Buyer order ID `b` and Seller order ID `a` will be removed from the Trade Streams (i.e. `<symbol>@trade`).  (Note that this may take a few hours to be rolled out to all servers.)
    * [WebSocket Streams](web-socket-streams.md) has been updated regarding this change.
    * To monitor if your order was part of a trade, please listen to the [User Data Streams](user-data-stream.md)

---

### 2024-06-06

This will be available by **June 6, 11:59 UTC**.

REST API

* `orderRateLimitExceededMode` has been added to `POST /api/v3/order/cancelReplace`.

WebSocket API

* `orderRateLimitExceededMode` has been added to `order.cancelReplace`.

---

### 2024-05-30

WebSocket Streams:

* Kline/Candlestick streams can now support a UTC+8 timezone offset. (e.g. `btcusdt@kline_1d@+08:00`)

---

### 2024-04-10

The following changes have been postponed to take effect on **April 25, 05:00 UTC**

General changes:

* Symbol permission information in Exchange Information responses has moved from field `permissions` to field `permissionSets`.
* Field `permissions` will be empty and will be removed in a future release.
* Previously, `"permissions":["SPOT","MARGIN"]` meant that you could place an order on the symbol if your account had `SPOT` or `MARGIN` permissions. The equivalent is `"permissionSets":[["SPOT","MARGIN"]]`. (Note the extra set of square brackets.) Each array of permissions inside the `permissionSets` array is called a "permission set".
* Symbol permissions can now be more complex. `"permissionSets":[["SPOT","MARGIN"],["TRD_GRP_004","TRD_GRP_005"]]` means that you may place an order on the symbol if your account has SPOT or MARGIN permissions **and** `TRD_GRP_004` or `TRD_GRP_005` permissions. There may be an arbitrary number of permission sets in a symbol's `permissionSets`.

REST API

* `otoAllowed` will now appear on `GET /api/v3/exchangeInfo`, that indicates if One-Triggers-the-Other (OTO) orders are supported on that symbol.

WebSocket API

* `otoAllowed` will now appear on `exchangeInfo`, that indicates if One-Triggers-the-Other (OTO) orders are supported on that symbol.

SBE

* A new schema 2:0 [spot_2_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_0.xml) has been released. The current schema 1:0 [spot_1_0.xml](https://github.com/binance/binance-spot-api-docs/blob/becd4d44a09d94821d2dc761ba9197aae8b495c3/sbe/schemas/spot_1_0.xml) will thus be deprecated, and retired from the API in 6 months as per our schema deprecation policy.
* When using schema 1:0 on REST API or WebSocket API, group "permissions" in message "ExchangeInfoResponse" will always be empty. Upgrade to schema 2:0 to find permission information in group "permissionSets". See General changes above for more details.
* Deprecated OCO requests will still be supported by the latest schema.
* Note that trying to use schema 2:0 before it is actually released will result in an error.


---

### 2024-04-02

**Notice:** The changes below are being rolled out gradually, and will take approximately a week to complete.

General changes:

* `GET /api/v3/account` has a new optional parameter `omitZeroBalances`, which if enabled hides all zero balances.
* `account.status` has a new optional parameter `omitZeroBalances` which if enabled hides all zero balances.
* **The weight of the following requests has been increased from 10 to 25 (This will take effect on April 4, 2024)**:
    * `GET /api/v3/trades`
    * `GET /api/v3/historicalTrades`
    * `trades.recent`
    * `trades.historical`

User Data Stream:

* New event `listenKeyExpired` that will be emitted in the streams if the `listenKey` expired.

REST API

* The `POST /api/v3/order/oco` endpoint is now deprecated on the REST API. You should use the new `POST /api/v3/orderList/oco` endpoint instead. Note that this new endpoint uses different parameters.

WebSocket API

* The `orderList.place` request is now deprecated on the WebSocket API. You should now use the new `orderList.place.oco` request instead. Note that this new request uses different parameters.


**The following will take effect _approximately_ a week after the release date:**

General changes:

* Symbol permission information in Exchange Information responses has moved from field `permissions` to field `permissionSets`.
* Field `permissions` will be empty and will be removed in a future release.
* Previously, `"permissions":["SPOT","MARGIN"]` meant that you could place an order on the symbol if your account had `SPOT` or `MARGIN` permissions. The equivalent is `"permissionSets":[["SPOT","MARGIN"]]`. (Note the extra set of square brackets.) Each array of permissions inside the `permissionSets` array is called a "permission set".
* Symbol permissions can now be more complex. `"permissionSets":[["SPOT","MARGIN"],["TRD_GRP_004","TRD_GRP_005"]]` means that you may place an order on the symbol if your account has SPOT or MARGIN permissions **and** `TRD_GRP_004` or `TRD_GRP_005` permissions. There may be an arbitrary number of permission sets in a symbol's `permissionSets`.

REST API

* `otoAllowed` will now appear on `GET /api/v3/exchangeInfo`, that indicates if One-Triggers-the-Other (OTO) orders are supported on that symbol.

WebSocket API

* `otoAllowed` will now appear on `exchangeInfo`, that indicates if One-Triggers-the-Other (OTO) orders are supported on that symbol.


SBE

* A new schema 2:0 [spot_2_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_0.xml) has been released. The current schema 1:0 [spot_1_0.xml](https://github.com/binance/binance-spot-api-docs/blob/becd4d44a09d94821d2dc761ba9197aae8b495c3/sbe/schemas/spot_1_0.xml) will thus be deprecated, and retired from the API in 6 months as per our schema deprecation policy.
* When using schema 1:0 on REST API or WebSocket API, group "permissions" in message "ExchangeInfoResponse" will always be empty. Upgrade to schema 2:0 to find permission information in group "permissionSets". See General changes above for more details.
* Deprecated OCO requests will still be supported by the latest schema.
* Note that trying to use schema 2:0 before it is actually released will result in an error.

---


### 2024-02-28

**This will take effect on March 5, 2024.**

Simple Binary Encoding (SBE) will be added to the live exchange, both for the Rest API and WebSocket API.

For more information on SBE, please refer to the [FAQ](./faqs/sbe_faq.md)


---

### 2024-02-08

The SPOT WebSocket API can now support SBE on [SPOT Testnet](https://testnet.binance.vision).

The SBE schema has been updated with WebSocket API metadata without incrementing either `schemaId` or `version`.

Users using SBE only on the REST API may continue to use the SBE schema with git commit hash [`128b94b2591944a536ae427626b795000100cf1d`](https://github.com/binance/binance-spot-api-docs/blob/128b94b2591944a536ae427626b795000100cf1d/sbe/schemas/spot_1_0.xml) or update to the newly-published SBE schema.

Users who want to use SBE on the WebSocket API must use the [newly-published SBE schema](https://github.com/binance/binance-spot-api-docs/blob/becd4d44a09d94821d2dc761ba9197aae8b495c3/sbe/schemas/spot_1_0.xml).

The [FAQ](./faqs/sbe_faq.md) for SBE has been updated.

---

### 2023-12-08

Simple Binary Encoding (SBE) has been added to [SPOT Testnet](https://testnet.binance.vision).

This will be added to the live exchange at a later date.

For more information on what SBE is, please refer to the [FAQ](./faqs/sbe_faq.md)

---

### 2023-12-04

**Notice:** The changes below are being rolled out gradually, and will take approximately a week to complete.

General Changes:

* Error message `Precision is over the maximum defined for this asset.` has been changed to `Parameter '%s' has too much precision.`
    * This error message is returned when a parameter has more precision than allowed:
      e.g. if `base asset` precision is 6 and `quantity=0.1234567` then this error message will appear.
    * This affects all requests with the following parameters:
        * `quantity`
        * `quoteOrderQty`
        * `icebergQty`
        * `limitIcebergQty`
        * `stopIcebergQty`
        * `price`
        * `stopPrice`
        * `stopLimitPrice`
* Requests for open OCO now correctly return results in **ascending order**. This affects the following requests:
    * REST API: `GET /api/v3/openOrderList`
    * WebSocket API: `openOrderList.status`
* Requests for all OCO now correctly return results in **ascending order** when `startTime` or `fromId` are specified. This affects the following requests:
    * REST API: `GET /api/v3/allOrderList`
    * WebSocket API: `allOrderLists`
* Fixed a bug where order query requests would incorrectly return [`-2026 ORDER_ARCHIVED`](./errors.md#-2026-order_archived) error for newly placed orders.
    * REST API: `GET /api/v3/order`
    * WebSocket API: `order.status`

REST API

* New endpoint `GET /api/v3/account/commission`
* New endpoint `GET /api/v3/ticker/tradingDay`
* `GET /api/v3/avgPrice` response has a new field `closeTime`, indicating the last trade time.
* `GET /api/v3/klines` and `/api/v3/uiKlines` have a new optional parameter `timeZone`.
* `POST /api/v3/order/test` and `POST /api/v3/sor/order/test` have a new optional parameter `computeCommissionRates`.
* Changes regarding invalid endpoints being sent:
    * Previously, if you query an non-existing endpoint (e.g. `curl -X GET "https://api.binance.com/api/v3/exchangie`) you would get a HTTP 404 code with the response `<html><body><h2>404 Not found</h2></body></html>`
    * From now on the HTML response will only appear if the Accept request header has `text/html` for this situation. The HTTP code will remain the same.

WebSocket API

* New request `account.commission`
* New requests to allow session authentication: **(Note that these requests can only be used with Ed25519 keys.)**
    * `session.logon`
    * `session.logout`
    * `session.status`
* New request `ticker.tradingDay`
* `avgPrice` response has a new field `closeTime`, indicating the last trade time.
* `klines` and `uiKlines` have a new optional parameter `timeZone`.
* `order.test` and `sor.order.test` have a new optional parameter `computeCommissionRates`.
* Fixed a bug where unsolicited pongs sent before the ping would cause disconnection.

WebSocket Streams

* New stream `<symbol>@avgPrice`
* `id` now supports the same values as used for `id` in the WebSocket API:
    * 64-bit signed integers (previously this was unsigned)
    * Alphanumeric strings, max of 36 in length
    * `null`
* Fixed a bug where unsolicited pongs sent before the ping would cause disconnection.

User Data Streams

* When an event of type `executionReport` has an execution type (`x`) of `TRADE_PREVENTION`, fields `l`, `L` and `Y` will now always be 0. New fields `pl`, `pL` and `pY` will describe the prevented execution quantity, prevented execution price, and prevented execution notional instead. These new fields show the values of what would `l`, `L` and `Y` have been if the taker order didn't have self-trade prevention enabled.


**The following will take effect _approximately_ a week after the release date:**

* Symbol Permissions will only affect order placement, not cancellation.
    * `permissions` still apply to Cancel-Replace orders (i.e. The cancellation won't be allowed if your account does have the permission to place an order using this request.)


---

### 2023-10-19

**Effective on 2023-10-19 00:00 UTC**

* The request weights of the following requests have been increased:

<table>
    <tr>
        <th>REST API</th>
        <th>WebSocket API</th>
        <th>Condition </th>
        <th>Previous Request Weight</th>
        <th>New Request Weight</th>
    </tr>
    <tr>
        <td width="200px"><code>GET /api/v3/trades</code></td>
        <td><code>trades.recent</code></td>
        <td>N/A</td>
        <td>2</td>
        <td>10</td>
    </tr>
    <tr >
       <td rowspan="4"><code>GET /api/v3/depth</code></td>
       <td rowspan="4"><code>depth</code></td>
       <td><b>Limit 1-100</b></td>
       <td>2</td>
       <td>5</td>
    </tr>
    <tr>
        <td width="100px"><b>Limit 101-500</b></td>
        <td>10</td>
        <td>25</td>
    </tr>
    <tr>
        <td><b>Limit 501-1000</b></td>
        <td>20</td>
        <td>50</td>
    </tr>
    <tr>
        <td><b>Limit 1001-5000</b></td>
        <td>100</td>
        <td>250</td>
    </tr>
</table>

---

### 2023-10-03

* **Order decrement feature went live at 06:15 UTC**.
* For more information on this feature, please refer to our [FAQ](./faqs/order_count_decrement.md)

---

### 2023-08-25

* For WebSocket API, removed `RAW REQUESTS` rate limit in `exchangeInfo`, replaced it with `CONNECTIONS` rate limit, which is the limit for new Websocket connections.

**The following changes will be effective from 2023-08-25 at UTC 00:00.**
* The `CONNECTIONS` rate limit for WebSocket API has been adjusted to 300 every 5 minutes.
* The `REQUEST_WEIGHT` rate limit for both REST and WebSocket API has been adjusted to 6,000 every minute.
* The `RAW_REQUESTS` rate limit for REST API has been adjusted to 61,000 every 5 minutes.
* Previously, connecting to WebSocket API used to cost 1 weight. **The cost is now 2**.
* The weights to the following requests for both REST API and WebSocket API have been adjusted.

Please refer to the table for more details:

|Request|Previous Request Weight| New Request Weight|
|----   |----                   | ----                   |
|`GET /api/v3/order` <br> `order.status` |2 | 4|
|`GET /api/v3/orderList` <br> orderList.status| 2|4|
|`GET /api/v3/openOrders` <br> `openOrders.status` - **With `symbol`**|3|6|
|`GET /api/v3/openOrders` <br> `openOrders.status` - **Without `symbol`**|40|80|
|`GET /api/v3/openOrderList` <br>`openOrderLists.status`|3|6|
|`GET /api/v3/allOrders` <br>`allOrders`  |10|20
|`GET /api/v3/allOrderList` <br> `allOrderLists` |10|20
|`GET /api/v3/myTrades`  <br> `myTrades`|10|20|
|`GET /api/v3/myAllocations`  <br> `myAllocations` |10|20|
|`GET /api/v3/myPreventedMatches`  <br> `myPreventedMatches`  - **Using `preventedMatchId`** | 1 | 2
|`GET /api/v3/myPreventedMatches`  <br> `myPreventedMatches`  - **Using `orderId`**|10|20|
|`GET /api/v3/account` <br> `account.status` |10 |20|
|`GET /api/v3/rateLimit/order` <br> `account.rateLimits.orders`|20|40|
|`GET /api/v3/exchangeInfo` <br> `exchangeInfo`|10|20|
|`GET /api/v3/depth`<br> `depth`  - **Limit 1-100**|1|2|
|`GET /api/v3/depth` <br> `depth` - **Limit 101-500**|5|10|
|`GET /api/v3/depth` <br>`depth`  - **Limit 501-1000**|10|20|
|`GET /api/v3/depth` <br> `depth`  - **Limit 1001-5000**|50|100|
|`GET /api/v3/aggTrades`  <br> `trades.aggregate` |1|2|
|`GET /api/v3/trades` <br> `trades.recent`  |1|2|
|`GET /api/v3/historicalTrades`  <br> `trades.historical` |5|10|
|`GET /api/v3/klines` <br> `klines`  |1|2|
|`GET /api/v3/uiKlines` <br> `uiKlines` |1|2|
|`GET /api/v3/ticker/bookTicker` <br> `ticker.book` - **With `symbol`**|1|2|
|`GET /api/v3/ticker/bookTicker` <br> `ticker.book` - **Without `symbol`** or **With `symbols`**|2|4|
|`GET /api/v3/ticker/price`<br> `ticker.price` - **With `symbol`**|1|2|
|`GET /api/v3/ticker/price`<br> `ticker.price` - **Without `symbol`** or **With `symbols`**|2|4|
|`GET /api/v3/ticker/24hr` <br> `ticker.24hr` - **With `symbol`** or **With `symbols` using 1-20 symbols** |1|2|
|`GET /api/v3/ticker/24hr` <br> `ticker.24hr` - **With `symbols using 21-100 symbols`**|20|40|
|`GET /api/v3/ticker/24hr` <br> `ticker.24hr` - **Without `symbol` or `symbols using 101 or more symbols`**|40|80|
|`GET /api/v3/avgPrice` <br>`avgPrice`|1|2|
|`GET /api/v3/ticker` <br> `ticker`|2|4|
|`GET /api/v3/ticker` <br> `ticker` - Maximum weight for this request|100|200|
|`POST /api/v3/userDataStream` <br> `userDataStream.start`|1|2|
|`PUT /api/v3/userDataStream` <br> `userDataStream.ping`|1|2|
|`DELETE /api/v3/userDataStream`<br> `userDataStream.stop`|1|2|


---

### 2023-08-08

Smart Order Routing (SOR) has been added to the APIs. For more information please refer to our [FAQ](./faqs/sor_faq.md). Please wait for future announcements on when the feature will be enabled.

REST API

* Changes to `GET /api/v3/exchangeInfo`:
    * New field in response: `sors`, describing SORs enabled on the exchange.
* Changes to `GET /api/v3/myPreventedMatches`
    * New field `makerSymbol` will appear in the response for all prevented matches.
* New endpoints for order placement using SOR:
    * `POST /api/v3/sor/order`
    * `POST /api/v3/sor/order/test`
* New endpoint `GET /api/v3/myAllocations`

WEBSOCKET API

* Changes to `exchangeInfo`:
    * New field in response: `sors`, describing SORs enabled on the exchange.
* Changes to `myPreventedMatches`:
    * New field `makerSymbol` will appear in the response for all prevented matches.
* New requests for order placement using SOR:
    * `sor.order.place`
    * `sor.order.test`
* New request `myAllocations`

USER DATA STREAM

* Changes to `executionReport`:
    * These fields are only relevant for orders placed using SOR:
        * New field `b` for `matchType`
        * New field `a` for `allocId`
        * New field `k` for `workingFloor`
    * This field is only relevant for orders expiring due to STP:
        * New field `Cs` for `counterSymbol`

---

### 2023-07-18

* New API key type – Ed25519 – is now supported. (UI support will be released this week.)
  * Ed25519 API keys are an alternative to RSA API keys, using asymmetric cryptography to authenticate your requests on the API.
  * **We recommend switching to Ed25519** for improved performance and security. <br>
    For more information, please refer to the [API Key Types](./faqs/api_key_types.md).
* Documentation has been updated with how to sign a payload with Ed25519 keys.

---

### 2023-07-11

**Notice:** The change below are being rolled out, and will take approximately a week to complete.

General Changes:

* Changes to error messages:
    * Previously, when duplicate symbols were passed to requests that do not allow it, the error would be "Mandatory parameter symbols was not sent, was empty/null, or malformed."
    * Now, the error message is "Symbol is present multiple times in the list", with a new error code `-1151`
    * This affects the following requests:
        * `GET /api/v3/exchangeInfo`
        * `GET /api/v3/ticker/24hr`
        * `GET /api/v3/ticker/price`
        * `GET/api/v3/ticker/bookTicker`
        * `exchangeInfo`
        * `ticker.24hr`
        * `ticker.price`
        * `ticker.book`
* Fixed a bug where some non-archived orders being queried would receive the error code that their order was archived.

Rest API

* Changes to `GET /api/v3/account`:
    * New field `preventSor` will appear in the response.
    * New field `uid` that shows the User Id/Account will appear in the response.
* Changes to `GET /api/v3/historicalTrades`:
    * Changed security type from `MARKET_DATA` to `NONE`.
    * This means that the `X-MBX-APIKEY` header is no longer necessary and is now ignored.

Websocket API

* Changes to `account.status`:
    * New field `preventSor` will appear in the response.
    * New field `uid` that shows the User Id/Account will appear in the response.
* Changes to `trades.historical`:
    * Changed security type from `MARKET_DATA` to `NONE`.
    * This means that the `apiKey` parameter is no longer necessary and is now ignored.

**The following changes will take effect _approximately a week from the release date_:**:

* Fixed multiple bugs with orders that use `type=MARKET` and `quoteOrderQty`, also known as “reverse market orders”:
    * Reverse market orders are no longer partially filled, or filled for zero or negative quantity under extreme market conditions.
    * `MARKET_LOT_SIZE` filter now correctly rejects reverse market orders that go over the symbol's `maxQty`.
* Fixed a bug where OCO orders using `trailingDelta` could have an incorrect `trailingTime` value after either leg of the OCO is touched.
* New field `transactTime` will appear in order cancellation responses. This affects the following requests:
    * `DELETE /api/v3/order`
    * `POST /api/v3/order/cancelReplace`
    * `DELETE /api/v3/openOrders`
    * `DELETE /api/v3/orderList`
    * `order.cancel`
    * `order.cancelReplace`
    * `openOrders.cancelAll`
    * `orderList.cancel`

---

### 2023-06-06

* A new endpoint is now available for redundancy: **https://api-gcp.binance.com/**
    * This is using the GCP (Google Cloud Platform) CDN and may have slower performance compared to `api1`-`api4` endpoints.

---

### 2023-05-26

**Notice:** The change below are being rolled out, and will take approximately a week to complete.

* The following base endpoints may give better performance but have less stability than **https://api.binance.com**:
  * **https://api1.binance.com**
  * **https://api2.binance.com**
  * **https://api3.binance.com**
  * **https://api4.binance.com**

---

### 2023-05-24

* **The previous market data URLs have been deprecated. Please update your code immediately to prevent interruption of our services.**
    * API Market data from `data.binance.com` can now be accessed from `data-api.binance.vision`.
    * Websocket Market Data from `data-stream.binance.com` can now be accessed from `data-stream.binance.vision`.

---

### 2023-03-13

**Notice:** All changes are being rolled out gradually to all our servers, and may take a week to complete.

GENERAL CHANGES

* The error messages for certain issues have been improved for easier troubleshooting.

<table>
    <tr>
        <th>Situation</th>
        <th>Old Error Message</th>
        <th>New Error Message</th>
    </tr>
    <tr>
        <td>An account cannot place or cancel an order, due to trading ability disabled.</td>
        <td rowspan="3">This action is disabled on this account.</td>
        <td>This account may not place or cancel orders.</td>
    </tr>
    <tr>
        <td>When the permissions configured on the symbol do not match the permissions on the account.</td>
        <td>This symbol is not permitted for this account.</td>
    </tr>
    <tr>
        <td>When the account tries to place an order on a symbol it has no permissions for.</td>
        <td>This symbol is restricted for this account.</td>
    </tr>
    <tr>
        <td>Placing an order when <tt>symbol</tt> is not <tt>TRADING</tt>. </td>
        <td rowspan="2">Unsupported order combination.</td>
        <td>This order type is not possible in this trading phase.</td>
    </tr>
    <tr>
        <td>Placing an order with <tt>timeinForce</tt>=<tt>IOC</tt> or <tt>FOK</tt> on a trading phase that does not support it.</td>
        <td>Limit orders require GTC for this phase.</td>
    </tr>
</table>

* Fixed error message for querying archived orders:
    * Previously, if an archived order (i.e. order with status `CANCELED` or `EXPIRED` where `executedQty` == 0 that occurred more than 90 days in the past.) is queried, the error message would be:
    ```json
    {
        "code": -2013,
        "msg": "Order does not exist."
    }
    ```
    * Now, the error message is:
    ```json
    {
        "code": -2026,
        "msg": "Order was canceled or expired with no executed qty over 90 days ago and has been archived."
    }
    ```
* Behavior for API requests with `startTime` and `endTime`:
    * Previously some requests failed if the `startTime` == `endTime`.
    * Now, all API requests that accept `startTime` and `endTime` allow the parameters to be equal. This applies to the following requests:
        * Rest API
            * `GET /api/v3/aggTrades`
            * `GET /api/v3/klines`
            * `GET /api/v3/allOrderList`
            * `GET /api/v3/allOrders`
            * `GET /api/v3/myTrades`
        * Websocket API
            * `trades.aggregate`
            * `klines`
            * `allOrderList`
            * `allOrders`
            * `myTrades`
* Users connected to the websocket API will now be disconnected if their IP is banned due to violation of the IP rate limits (status `418`).

The following changes will take effect **approximately a week from the release date**, but the rest of the documentation has been updated to reflect the future changes:

* Changes to Filter Evaluation:
    * Previous behavior: `LOT_SIZE` and `MARKET_LOT_SIZE` required that (`quantity` - `minQty`) % `stepSize` == 0.
    * New behavior: This has now been changed to (`quantity` % `stepSize`) == 0.
* Bug fix with reverse `MARKET` orders (i.e., `MARKET` using `quoteOrderQty`):
    * Previous behavior: Reverse market orders would always have the status `FILLED` even if the order did not fully fill due to low liquidity.
    * New behavior: If the reverse market order did not fully fill due to low liquidity the order status will be `EXPIRED`, and `FILLED` only if the order was completely filled.

REST API

* Changes to `DELETE /api/v3/order` and `POST /api/v3/order/cancelReplace`:
    * A new optional parameter `cancelRestrictions` that determines whether the cancel will succeed if the order status is `NEW` or `PARTIALLY_FILLED`.
    * If the order cancellation fails due to `cancelRestrictions`, the error will be:
    ```json
    {
        "code": -2011,
        "msg": "Order was not canceled due to cancel restrictions."
    }
    ```

WEBSOCKET API

* Changes to `order.cancel` and `order.cancelReplace`:
    * A new optional parameter `cancelRestrictions` that determines whether the cancel will succeed if the order status is `NEW` or `PARTIALLY_FILLED`.
    * If the order cancellation fails due to `cancelRestrictions`, the error will be:
    ```json
    {
        "code": -2011,
        "msg": "Order was not canceled due to cancel restrictions."
    }
    ```
---

### 2023-02-17

**Changes to Websocket Limits**

The WS-API and Websocket Stream now only allows 300 connections requests every 5 minutes.

This limit is **per IP address**.

Please be careful when trying to open multiple connections or reconnecting to the Websocket API.

---

### 2023-01-26

As per the [announcement](https://www.binance.com/en/support/announcement/binance-spot-launches-self-trade-prevention-stp-function-on-api-312fd0112fb44635b397c116e56d8f84), Self Trade Prevention will be enabled at **2023-01-26 08:00 UTC**.

Please refer to `GET /api/v3/exchangeInfo` from the Rest API or `exchangeInfo` from the Websocket API on the default and allowed modes.

---

### 2023-01-23

New API cluster has been added. Note that all endpoints are functionally equal, but may vary in performance.

* https://api4.binance.com

---

### 2023-01-19

**ACTUAL RELEASE DATE TBD**

**New Feature**: Self-Trade Prevention (aka STP) will be added to the system at a later date. This will prevent orders from matching with orders from the same account, or accounts under the same `tradeGroupId`.

Please refer to `GET /api/v3/exchangeInfo` from the Rest API or `exchangeInfo` from the Websocket API on the status.

```javascript
{
    "defaultSelfTradePreventionMode": "NONE",     // If selfTradePreventionMode not provided, this will be the value passed to the engine
    "allowedSelfTradePreventionModes": [          // What the allowed modes of selfTradePrevention are
        "NONE",
        "EXPIRE_TAKER",
        "EXPIRE_BOTH",
        "EXPIRE_MAKER"
    ]
}
```

Additional details on the functionality of STP is explained in the [STP FAQ](./faqs/stp_faq.md) document.

REST API

* New order status: `EXPIRED_IN_MATCH` - This means that the order expired due to STP being triggered.
* New endpoint:
   * `GET /api/v3/myPreventedMatches` - This queries the orders that expired due to STP being triggered.
* New optional parameter `selfTradePreventionMode` has been added to the following endpoints:
    * `POST /api/v3/order`
    * `POST /api/v3/order/oco`
    * `POST /api/v3/cancelReplace`
* New responses that will appear for all order placement endpoints if there was a prevented match (i.e. if an order could have matched with an order of the same account, or the accounts are in the same `tradeGroupId`):
    * `tradeGroupId`      - This will only appear if account is configured to a `tradeGroupId` and if there was a prevented match.
    * `preventedQuantity` - Only appears if there was a prevented match.
    * An array `preventedMatches` with the following fields:
        * `preventedMatchId`
        * `makerOrderId`
        * `price`
        * `takerPreventedQuantity` - This will only appear if `selfTradePreventionMode` set is `EXPIRE_TAKER` or `EXPIRE_BOTH`.
        * `makerPreventedQuantity` - This will only appear if `selfTradePreventionMode` set is `EXPIRE_MAKER` or `EXPIRE_BOTH`.
* New fields `preventedMatchId` and `preventedQuantity` that can appear in the order query endpoints if the order had expired due to an STP trigger:
    * `GET /api/v3/order`
    * `GET /api/v3/openOrders`
    * `GET /api/v3/allOrders`

WEBSOCKET API

* New order status: `EXPIRED_IN_MATCH` - This means that the order expired due to STP being triggered.
* New optional parameter `selfTradePreventionMode` has been added to the following requests:
    * `order.place`
    * `orderList.place`
    * `order.cancelReplace`
* New request: `myPreventedMatches` - This queries the orders that expired due to STP being triggered.
* New responses that will appear for all order placement endpoints if there was a prevented match (i.e. if an order could have matched with an order of the same account, or the accounts are in the same `tradeGroupId`):
    * `tradeGroupId`      - This will only appear if account is configured to a `tradeGroupId` and if there was a prevented match.
    * `preventedQuantity` - Only appears if there was a prevented match.
    * An array `preventedMatches` with the following fields:
        * `preventedMatchId`
        * `makerOrderId`
        * `price`
        * `takerPreventedQuantity` - This will only appear if `selfTradePreventionMode` set is `EXPIRE_TAKER` or `EXPIRE_BOTH`.
        * `makerPreventedQuantity` - This will only appear if `selfTradePreventionMode` set is `EXPIRE_MAKER` or `EXPIRE_BOTH`.
* New fields `preventedMatchId` and `preventedQuantity` that can appear in the order query requests if the order had expired due to STP trigger:
    * `order.status`
    * `openOrders.status`
    * `allOrders`


USER DATA STREAM

* New execution Type: `TRADE_PREVENTION`
* New fields for `executionReport` (These fields will only appear if the order has expired due to STP trigger)
    * `u` - `tradeGroupId`
    * `v` - `preventedMatchId`
    * `U` - `counterOrderId`
    * `A` - `preventedQuantity`
    * `B` - `lastPreventedQuantity`


---

### 2022-12-28

* SPOT WebSocket API documentation has been updated to show how to sign a request using an RSA key.

---

### 2022-12-26

* Spot WebSocket API is now available on the live exchange.
* Spot Websocket API can be accessed through this URL: `wss://ws-api.binance.com/ws-api/v3`

---

### 2022-12-15

* New RSA signature
    * Documentation has been updated to show how to create RSA keys.
    * For security reasons, we recommend to use RSA keys instead of HMAC keys when generating an API key.
    * We accept `PKCS#8` (BEGIN PUBLIC KEY).
    * More details on how to upload your RSA public key will be added at a later date.
* SPOT WebSocket API is now available on **SPOT Testnet**.
    * WebSocket API allows placing orders, canceling orders, etc. through a WebSocket connection.
    * WebSocket API is a **separate** service from WebSocket Market Data streams. I.e., placing orders and listening to market data requires two separate WebSocket connections.
    * WebSocket API is subject to the same Filter and Rate Limit rules as REST API.
    * WebSocket API and REST API are functionally equivalent: they provide the same features, accept the same parameters, return the same status and error codes.

**WEBSOCKET API WILL BE AVAILABLE ON THE LIVE EXCHANGE AT A LATER DATE.**

---

### 2022-12-13

REST API

Some error messages on error code `-1003` have changed.
* Previous error message: `Too much request weight used; current limit is %s request weight per %s %s. Please use the websocket for live updates to avoid polling the API.` has been updated to:
```
Too much request weight used; current limit is %s request weight per %s. Please use WebSocket Streams for live updates to avoid polling the API.
```
* Previous error message `Way too much request weight used; IP banned until %s. Please use the websocket for live updates to avoid bans.` has been updated to:
```
Way too much request weight used; IP banned until %s. Please use WebSocket Streams for live updates to avoid bans.
```

---

### 2022-12-05

**Notice:** These changes are being rolled out gradually to all our servers, and will take approximately a week to complete.

WEBSOCKET

* `!bookTicker` will be removed by **December 7, 2022**. Please use the Individual Book Ticker Streams instead (`<symbol>@bookTicker`).
    * Multiple `<symbol>@bookTicker` streams can be subscribed to over one connection. (E.g. `wss://stream.binance.com:9443/stream?streams=btcusdt@bookTicker/bnbbtc@bookTicker`)

REST API

* New error code `-1135`
    * This error code will occur if a parameter requiring a JSON object is invalid.
* New error code `-1108`
    * This error will occur if a value to a parameter being sent was too large, potentially causing overflow.
    * This error code can occur in the following endpoints:
        * `POST /api/v3/order`
        * `POST /api/v3/order/cancelReplace`
        * `POST /api/v3/order/oco`
* Changes to `GET /api/v3/aggTrades`
    * Previous behavior: `startTime` and `endTime` had to be used in combination and could only be an hour apart.
    * New behavior: `startTime` and `endTime` can be used individually and the 1 hour limit has been removed.
        * When using `startTime` only, this will return trades from that time, up to the `limit` provided.
        * When using `endTime` only, this will return trades before that time, up to the `limit` provided.
        * If `limit` not provided, regardless of used in combination or sent individually, the endpoint will use the default limit.
* Changes to `GET /api/v3/myTrades`
    * Fixed a bug where `symbol` + `orderId` combination would return all trades even if the number of trades went beyond the `500` default limit.
    * Previous behavior: The API would send specific error messages depending on the combination of parameters sent. E.g:

        ```json
        {
            "code": -1106,
            "msg": "Parameter X was sent when not required."
        }
        ```
    * New behavior: If the combinations of optional parameters to the endpoint were not supported, then the endpoint will respond with the generic error:

        ```json
        {
            "code": -1128,
            "msg": "Combination of optional parameters invalid."
        }
        ```
    * Added a new combination of supported parameters: `symbol` + `orderId` + `fromId`.
    * The following combinations of parameters were previously supported but no longer accepted, as these combinations were only taking `fromId` into consideration, ignoring `startTime` and `endTime`:
        * `symbol` + `fromId` + `startTime`
        * `symbol` + `fromId` + `endTime`
        * `symbol` + `fromId` + `startTime` + `endTime`
    * Thus, these are the supported combinations of parameters:
        * `symbol`
        * `symbol` + `orderId`
        * `symbol` + `startTime`
        * `symbol` + `endTime`
        * `symbol` + `fromId`
        * `symbol` + `startTime` + `endTime`
        * `symbol`+ `orderId` + `fromId`

**Note:** These new fields will appear approximately a week from the release date.

* Changes to `GET /api/v3/exchangeInfo`
    * New fields `defaultSelfTradePreventionMode` and `allowedSelfTradePreventionModes`
* Changes to the Order Placement Endpoints/Order Query/Order Cancellation Endpoints:
    * New field `selfTradePreventionMode` will appear in the response.
    * Affects the following endpoints:
        * `POST /api/v3/order`
        * `POST /api/v3/order/oco`
        * `POST /api/v3/order/cancelReplace`
        * `GET /api/v3/order`
        * `DELETE /api/v3/order`
        * `DELETE /api/v3/orderList`
* Changes to  `GET /api/v3/account`
    * New field `requireSelfTradePrevention` will appear in the response.
* New field `workingTime`, indicating when the order started working on the order book, will appear in the following endpoints:
    * `POST /api/v3/order`
    * `GET /api/v3/order`
    * `POST /api/v3/order/cancelReplace`
    * `POST /api/v3/order/oco`
    * `GET /api/v3/order`
    * `GET /api/v3/openOrders`
    * `GET /api/v3/allOrders`
* Field `trailingTime`, indicating the time when the trailing order is active and tracking price changes, will appear for the following order types  (`TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`, `STOP_LOSS`, `STOP_LOSS_LIMIT` if `trailingDelta` parameter was provided) for the following endpoints:
    * `POST /api/v3/order`
    * `GET /api/v3/order`
    * `GET /api/v3/openOrders`
    * `GET /api/v3/allOrders`
    * `POST /api/v3/order/cancelReplace`
    * `DELETE /api/v3/order`
* Field `commissionRates` will appear in the `GET /api/v3/acccount` response


USER DATA STREAM

* eventType `executionReport` has new fields
    * `V` - `selfTradePreventionMode`
    * `D` - `trailing_time`  (Appears if the trailing stop order is active)
    * `W` - `workingTime`   (Appears if `isWorking`=`true`)


---

### 2022-12-02

* Added a new market data base URL `https://data.binance.com`.
* Added a new WebSocket URL `wss://data-stream.binance.com`.

---

### 2022-09-30

Scheduled changes to the removal of `!bookTicker` around November 2022.

* The All Book Tickers stream (`!bookTicker`) is set to be removed in **November 2022**.
* More details of the actual removal date will be announced at a later time.
* Please use the Individual Book Ticker Streams instead. (`<symbol>@bookTicker`).
* Multiple `<symbol>@bookTicker` streams can be subscribed to over one connection.
    * Example: wss://stream.binance.com:9443/stream?streams=btcusdt@bookTicker/bnbbtc@bookTicker

---

### 2022-09-15

Note that these are rolling changes, so it may take a few days for it to rollout to all our servers.

* Changes to `GET /api/v3/exchangeInfo`
    * New optional parameter `permissions` added to display all symbols with the permissions matching the parameter provided. (eg.`SPOT`,`MARGIN`)
    * If not provided, the default value will be `["SPOT","MARGIN","LEVERAGED"]`.
        * This means the request `GET /api/v3/exchangeInfo` without any parameters will show all symbols that can be used for `SPOT`, `MARGIN`, and/or `LEVERAGED` trading.
        * To search for symbols that can be traded on other permissions (e.g. `TRD_GRP_004`, etc), then this needs to be searched for explicitly. (e.g.`permissions`=`TRD_GRP_004`)
    * Cannot be combined with `symbol` or `symbols`

---

### 2022-08-23

Note that these are rolling changes, so it may take a few days for it to rollout to all our servers.

* Changes to `GET /api/v3/ticker` and `GET /api/v3/ticker/24hr`
    * New optional parameter `type` added
    * Supported values for parameter `type` are `FULL` and `MINI`
        * `FULL` is the default value and the response that is currently being returned from the endpoint
        * `MINI` omits the following fields from the response: `priceChangePercent`, `weightedAvgPrice`, `bidPrice`, `bidQty`, `askPrice`, `askQty`, and `lastQty`
* New error code `-1008`
    * This is sent whenever the servers are overloaded with requests.
* New field `brokered` has been added to `GET /api/v3/account`
* New kline interval: `1s`
* New endpoint added: `GET /api/v3/uiKlines`

---

### 2022-08-08

REST API

* Changes to `POST /api/v3/order` and `POST /api/v3/order/cancelReplace`
    * New optional fields `strategyId` and `strategyType`
        * `strategyId` is a parameter used to identify an order as part of a strategy.
        * `strategyType` is a parameter used to identify what strategy was running. (E.g. If all the orders are part of spot grid strategy, it can be set to `strategyType=1000000`)
            * Note that the minimum value allowed for `strategyType` is `1000000`.
* Changes to `POST /api/v3/order/oco`
    * New optional fields `limitStrategyId`, `limitStrategyType`, `stopStrategyId`, `stopStrategyType`
    * These are the strategy metadata parameters for both legs of the OCO orders.
    * `limitStrategyType` and `stopStrategyType` both cannot be less than `1000000`.
* Changes to `GET /api/v3/order`, `GET /api/v3/openOrders`, and `GET /api/v3/allOrders`
    * New fields `strategyId` and `strategyType` will appear in the response JSON for orders that had these fields populated upon order placement.
* Changes to `DELETE /api/v3/order` and `DELETE /api/v3/openOrders`
    * New fields `strategyId` and `strategyType` will appear in the response JSON for cancelled orders that had these fields populated upon order placement.


USER DATA STREAM

* New fields to eventType `executionReport`
    * `j` for `strategyId`
    * `J` for `strategyType`
    * Note that these fields only appear if these were populated upon order placement.

---

### 2022-06-20

Changes to `GET /api/v3/ticker`

* Weight has been reduced from 5 to 2 per symbol, regardless of `windowSize`.
* The max number of symbols that can be processed in a request is 100.
    * If the number of `symbols` sent is more than 100, the error will be as follows:
    ```json
    {
        "code": -1101,
        "msg": "Too many values sent for parameter 'symbols', maximum allowed up to 100."
    }
    ```
* The max Weight for this endpoint will cap at 100.
    * I.e. If the request has more than 50 symbols, the Weight will still be 100, regardless of `windowSize`.

---

### 2022-06-15

**Note:** The update is being rolled out over the next few days, so these changes may not be visible right away.

SPOT API

* `GET /api/v3/ticker` added
    * Rolling window price change statistics based on `windowSize` provided.
    * Contrary to `GET /api/v3/ticker/24hr` the list of symbols cannot be omitted.
    * If `windowSize` not specified, the value will default to `1d`.
    * Response is similar to `GET /api/v3/ticker/24hr`, minus the following fields: `prevClosePrice`, `lastQty`, `bidPrice`, `bidQty`, `askPrice`, `askQty`
* `GET /api/v3/exchangeInfo` returns new field `cancelReplaceAllowed` in `symbols` list.
* `POST /api/v3/order/cancelReplace` added
    * Cancels an existing order and places a new order on the same symbol.
    * The filters are evaluated **before** the cancel order is placed.
        * e.g. If the `MAX_NUM_ORDERS` filter is 10, and the total number of open orders on the account is also 10, when using `POST /api/v3/order/cancelReplace` both the cancel order placement and new order will fail because of the filter.
    * The change is being rolled out in the next few days, thus this feature will be enabled once the upgrade is completed.
* New filter `NOTIONAL` has been added.
    * Defines the allowed notional value (`price * quantity`) based on a configured `minNotional` and `maxNotional`
* New exchange filter `EXCHANGE_MAX_NUM_ICEBERG_ORDERS` has been added.
    * Defines the limit of open iceberg orders on an account

---

### 2022-05-23
* Changes to Order Book Depth Levels
    * Quantities in the Depth levels were returning negative values in situations where they were exceeding the max value, resulting in an overflow.
    * Going forward depth levels will not overflow, but will be capped at the max value based on the precision of the base asset. This means that the depth level is at max value *or more*.
        * E.g. If the precision is 8, then the max value for quantity will be at 92,233,720,368.54775807.
    * When the fix has been applied, a change in the order book at the affected price level is required for the changes to be visible.
* What does this affect?
    * SPOT API
        * `GET /api/v3/depth`
    * Websocket Streams
        * `<symbol>@depth`
        * `<symbol>@depth@100ms`
        * `<symbol>@depth<levels>`
        * `<symbol>@depth<levels>@100ms`

* Updates to `MAX_POSITION`
    * If an order's `quantity` can cause the position to overflow, this will now fail the `MAX_POSITION` filter.
---

### 2022-05-17

* Changes to GET `api/v3/aggTrades`
    * When providing `startTime` and `endTime`, the oldest items are returned.
* Changed error messaging on `GET /api/v3/myTrades` where parameter `symbol` is not provided:
```json
{
    "code": -1102,
    "msg": "Mandatory parameter 'symbol' was not sent, was empty/null, or malformed."
}
```
*  The following endpoints now support multi-symbol querying using the parameter `symbols`.
    * `GET /api/v3/ticker/24hr`
    * `GET /api/v3/ticker/price`
    * `GET /api/v3/ticker/bookTicker`
* In the above, the request weight will depend on the number of symbols provided in `symbols`. <br/> Please refer to the table below:

|Endpoint|Number of Symbols|Weight|
|-----|-----|----|
| `GET /api/v3/ticker/price`|Any| 2|
|`GET /api/v3/ticker/bookTicker`|Any|2|
|`GET /api/v3/ticker/24hr`|1-20|1|
|`GET /api/v3/ticker/24hr`|21-100|20|
|`GET /api/v3/ticker/24hr`|101 or more|40|


---

### 2022-04-13

REST API

* Trailing Stops have been enabled.
    * This is a type of algo order where the activation is based on a percentage of a price change in the market using the new parameter `trailingDelta`.
    * This can only used with any of the following order types: `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`.
    * The `trailingDelta` parameter will be done in Basis Points or BIPS.
        * For example: a STOP_LOSS SELL order with a `trailingDelta` of 100 will trigger after a price decrease of 1% from the highest price after the order is placed. (100 / 10,000 => 0.01 => 1%)
    * When used in combination with OCO Orders, the `trailingDelta` will determine when the contingent leg of the OCO will trigger.
    * When `trailingDelta` is used in combination with `stopPrice`, once the `stopPrice` condition is met, the trailing stop starts tracking the price change from the `stopPrice` based on the `trailingDelta` provided.
    * When no `stopPrice` is sent, the trailing stop starts tracking the price changes from the last price based on the `trailingDelta` provided.
* Changes to POST `/api/v3/order`
    * New optional field `trailingDelta`
* Changes to POST `/api/v3/order/test`
    * New optional field `trailingDelta`
* Changes to POST `/api/v3/order/oco`
    * New optional field `trailingDelta`
* A new filter `TRAILING_DELTA` has been added.
    * This filter is defined by the minimum and maximum values for the `trailingDelta` value.

USER DATA STREAM

* New field in `executionReport`
    * "d" for `trailingDelta`

---

### 2022-04-12

**Note:** The changes are being rolled out during the next few days, so these will not appear right away.

* Error message changed on `GET api/v3/allOrders` where `symbol` is not provided:
    ```json
    {
        "code": -1102,
        "msg": "Mandatory parameter 'symbol' was not sent, was empty/null, or malformed."
    }
    ```
* Fixed a typo with an error message when an account has disabled permissions (e.g. to withdraw, to trade, etc)
    ```json
    "This action is disabled on this account."
    ```
* During a market data audit, we detected some issues with the Spot aggregate trade data.
    * Missing aggregate trades were recovered.
    * Duplicated records were marked invalid with the following values:
        * p = '0' // price
        * q = '0' // qty
        * f = -1 // ﬁrst_trade_id
        * l = -1 // last_trade_id

---

### 2022-02-28

* New field `allowTrailingStop` has been added to `GET /api/v3/exchangeInfo`

---

### 2022-02-24

* `(price-minPrice) % tickSize == 0` rule in `PRICE_FILTER` has been changed to `price % tickSize == 0`.
* A new filter `PERCENT_PRICE_BY_SIDE` has been added.
* Changes to GET `api/v3/depth`
    * The `limit` value can be outside of the previous values (i.e. 5, 10, 20, 50, 100, 500, 1000,5000) and will return the correct limit. (i.e. if limit=3 then the response will be the top 3 bids and asks)
    * The limit still cannot exceed 5000. If the limit provided is greater than 5000, then the response will be truncated to 5000.
    * Due to the changes, these are the updated request weights based on the limit value provided:

|Limit|Request Weight
------|-------
1-100|  1
101-500| 5
501-1000| 10
1001-5000| 50

* Changes to GET `api/v3/aggTrades`
    * When providing `startTime` and `endTime`, the oldest items are returned.
---

### 2021-12-29
* Removed out dated "Symbol Type" enum; added "Permissions" enum.

### 2021-11-01
* `GET /api/v3/rateLimit/order` added
    * The endpoint will display the user's current order count usage for all intervals.
    * This endpoint will have a request weight of 20.

### 2021-09-14
* Add a [YAML file](https://github.com/binance/binance-api-swagger) with OpenApi specification on the RESTful API.

### 2021-08-12
* GET `api/v3/myTrades` has a new optional field `orderId`

---

### 2021-05-12
* Added `Data Source` in the documentation to explain where each endpoint is retrieving its data.
* Added field `Data Source` to each API endpoint in the documentation
* GET `api/v3/exchangeInfo` now supports single or multi-symbol query

---

### 2021-04-26

On **April 28, 2021 00:00 UTC** the weights to the following endpoints will be adjusted:

* `GET /api/v3/order` weight increased to 2
* `GET /api/v3/openOrders` weight increased to 3
* `GET /api/v3/allOrders` weight increased to 10
* `GET /api/v3/orderList` weight increased to 2
* `GET /api/v3/openOrderList` weight increased to 3
* `GET /api/v3/account` weight increased to 10
* `GET /api/v3/myTrades` weight increased to 10
* `GET /api/v3/exchangeInfo` weight increased to 10

---

### 2021-01-01

**USER DATA STREAM**

* `outboundAccountInfo` has been removed.

---

### 2020-11-27

New API clusters have been added in order to improve performance.

Users can access any of the following API clusters, in addition to `api.binance.com`

If there are any performance issues with accessing `api.binance.com` please try any of the following instead:

* https://api1.binance.com/api/v3/*
* https://api2.binance.com/api/v3/*
* https://api3.binance.com/api/v3/*

### 2020-09-09

USER DATA STREAM

* `outboundAccountInfo` has been deprecated.
* `outboundAccountInfo` will be removed in the future. (Exact date unknown) **Please use `outboundAccountPosition` instead.**
* `outboundAccountInfo` will now only show the balance of non-zero assets and assets that have been reduced to 0.

---

### 2020-05-01
* From 2020-05-01 UTC 00:00, all symbols will have a limit of 200 open orders using the [MAX_NUM_ORDERS](./rest-api.md#max_num_orders) filter.
    * No existing orders will be removed or canceled.
    * Accounts that have 200 or more open orders on a symbol will not be able to place new orders on that symbol until the open order count is below 200.
    * OCO orders count as 2 open orders before the `LIMIT` order is touched or the `STOP_LOSS` (or `STOP_LOSS_LIMIT`) order is triggered; once this happens the other order is canceled and will no longer count as an open order.

---

### 2020-04-25

REST API

* New field `permissions`
    * Defines the trading permissions that are allowed on accounts and symbols.
    * `permissions` is an enum array; values:
        * `SPOT`
        * `MARGIN`
    * `permissions` will replace `isSpotTradingAllowed` and `isMarginTradingAllowed` on `GET api/v3/exchangeInfo` in future API versions (v4+).
    * For an account to trade on a symbol, the account and symbol must share at least 1 permission in common.
* Updates to `GET api/v3/exchangeInfo`
    *  New field `permissions` added.
    *  New field `quoteAssetPrecision` added; a duplicate of the `quotePrecision` field. `quotePrecision` will be removed in future API versions (v4+).
* Updates to `GET api/v3/account`
    * New field `permissions` added.
* New endpoint `DELETE api/v3/openOrders`
    * This will allow a user to cancel all open orders on a single symbol.
    * This endpoint will cancel all open orders including OCO orders.
* Orders can be canceled via the API on symbols in the `BREAK` or `HALT` status.

USER DATA STREAM

* `OutboundAccountInfo` has new field `P` which shows the trading permissions of the account.

---

### 2020-04-23

WEB SOCKET STREAM

* WebSocket connections have a limit of 5 incoming messages per second. A message is considered:
    * A PING frame
    * A PONG frame
    * A JSON control message (e.g. subscribe, unsubscribe)
* A connection that goes beyond the limit will be disconnected; IPs that are repeatedly disconnected may be banned.
* A single connection can listen to a maximum of 1024 streams.


---
### 2020-03-24

* `MAX_POSITION` filter added.
    * This filter defines the allowed maximum position an account can have on the base asset of a symbol. An account's position defined as the sum of the account's:
        * free balance of the base asset
        * locked balance of the base asset
        * sum of the qty of all open BUY orders

    * `BUY` orders will be rejected if the account's position is greater than the maximum position allowed.

---
### 2019-11-22

* Quote Order Qty Market orders have been enabled on all symbols.
    * Quote Order Qty `MARKET` orders allow a user to specify the total `quoteOrderQty` spent or received in the `MARKET` order.
    * Quote Order Qty `MARKET` orders will not break `LOT_SIZE` filter rules; the order will execute a quantity that will have the notional value as close as possible to `quoteOrderQty`.
    * Using `BNBBTC` as an example:
        * On the `BUY` side, the order will buy as many BNB as `quoteOrderQty` BTC can.
        * On the `SELL` side, the order will sell as much BNB as needed to receive `quoteOrderQty` BTC.

---
### 2019-11-13

REST API

* api/v3/exchangeInfo has new fields:
    * `quoteOrderQtyMarketAllowed`
    * `baseCommissionDecimalPlaces`
    * `quoteCommissionDecimalPlaces`
* `MARKET` orders have a new optional field: `quoteOrderQty` used to specify the quote quantity to BUY or SELL. This cannot be used in combination with `quantity`.
    * The exact timing that `quoteOrderQty` MARKET orders will be enabled is TBD. There will be a separate announcement and further details at that time.
* All order query endpoints will return a new field `origQuoteOrderQty` in the JSON payload. (e.g. GET api/v3/allOrders)
* Updated error messages for  -1128
    * Sending an `OCO` with a `stopLimitPrice` but without a `stopLimitTimeInForce` will return the error:
    ```json
    {
        "code": -1128,
        "msg": "Combination of optional parameters invalid. Recommendation: 'stopLimitTimeInForce' should also be sent."
    }
    ```
* Updated error messages for -1003 to specify the limit is referring to the request weight, not to the number of requests.

**Deprecation of v1 endpoints**:

By end of Q1 2020, the following endpoints will be removed from the API. The documentation has been updated to use the v3 versions of these endpoints.

* GET api/v1/depth
* GET api/v1/historicalTrades
* GET api/v1/aggTrades
* GET api/v1/klines
* GET api/v1/ticker/24hr
* GET api/v1/ticker/price
* GET api/v1/exchangeInfo
* POST api/v1/userDataStream
* PUT api/v1/userDataStream
* GET api/v1/ping
* GET api/v1/time
* GET api/v1/ticker/bookTicker

**These endpoints however, will NOT be migrated to v3. Please use the following endpoints instead moving forward.**

<table>
<tr>
<th>Old V1 Endpoints</th>
<th>New V3 Endpoints</th>
</tr>
<tr>
<td>GET api/v1/ticker/allPrices</td>
<td>GET api/v3/ticker/price</td>
</tr>
<tr>
<td>GET api/v1/ticker/allBookTickers</td>
<td>GET api/v3/ticker/bookTicker</td>
</tr>
</table>

USER DATA STREAM

* Changes to`executionReport` event
    * If the C field is empty, it will now properly return `null`, instead of `"null"`.
    * New field Q which represents the `quoteOrderQty`.

* `balanceUpdate` event type added
    * This event occurs when funds are deposited or withdrawn from your account.

WEB SOCKET STREAMS

* WSS now supports live subscribing/unsubscribing to streams.

---
### 2019-09-09
* New WebSocket streams for bookTickers added: `<symbol>@bookTicker` and `!bookTicker`. See `web-socket-streams.md` for details.

---
### 2019-09-03
* Faster order book data with 100ms updates: `<symbol>@depth@100ms` and `<symbol>@depth#@100ms`
* Added "Update Speed:" to `web-socket-streams.md`
* Removed deprecated v1 endpoints as per previous announcement:
    * GET api/v1/order
    * GET api/v1/openOrders
    * POST api/v1/order
    * DELETE api/v1/order
    * GET api/v1/allOrders
    * GET api/v1/account
    * GET api/v1/myTrades

---
### 2019-08-16 (Update 2)
* GET api/v1/depth `limit` of 10000 has been temporarily removed

---
### 2019-08-16
* In Q4 2017, the following endpoints were deprecated and removed from the API documentation. They have been permanently removed from the API as of this version. We apologize for the omission from the original changelog:
    * GET api/v1/order
    * GET api/v1/openOrders
    * POST api/v1/order
    * DELETE api/v1/order
    * GET api/v1/allOrders
    * GET api/v1/account
    * GET api/v1/myTrades

* Streams, endpoints, parameters, payloads, etc. described in the documents in this repository are **considered official** and **supported**. The use of any other streams, endpoints, parameters, or payloads, etc. is **not supported; use them at your own risk and with no guarantees.**

---
### 2019-08-15

REST API
* New order type: OCO ("One Cancels the Other")
    * An OCO has 2 orders: (also known as legs in financial terms)
        * ```STOP_LOSS``` or ```STOP_LOSS_LIMIT``` leg
        * ```LIMIT_MAKER``` leg

    * Price Restrictions:
        * ```SELL Orders``` : Limit Price > Last Price > Stop Price
        * ```BUY Orders``` : Limit Price < Last Price < Stop Price
        * As stated, the prices must "straddle" the last traded price on the symbol. EX: If the last price is 10:
            * A SELL OCO must have the limit price greater than 10, and the stop price less than 10.
            * A BUY OCO must have a limit price less than 10, and the stop price greater than 10.

    * Quantity Restrictions:
        * Both legs must have the **same quantity**.
        * ```ICEBERG``` quantities however, do not have to be the same.

    * Execution Order:
        * If the ```LIMIT_MAKER``` is touched, the limit maker leg will be executed first BEFORE canceling the Stop Loss Leg.
        * if the Market Price moves such that the ```STOP_LOSS``` or ```STOP_LOSS_LIMIT``` will trigger, the Limit Maker leg will be canceled BEFORE executing the ```STOP_LOSS``` Leg.

    * Canceling an OCO
        * Canceling either order leg will cancel the entire OCO.
        * The entire OCO can be canceled via the ```orderListId``` or the ```listClientOrderId```.

    * New Enums for OCO:
        1. ```ListStatusType```
            * ```RESPONSE``` - used when ListStatus is responding to a failed action. (either order list placement or cancellation)
            * ```EXEC_STARTED``` - used when an order list has been placed or there is an update to a list's status.
            * ```ALL_DONE``` - used when an order list has finished executing and is no longer active.
        1. ```ListOrderStatus```
            * ```EXECUTING``` - used when an order list has been placed or there is an update to a list's status.
            * ```ALL_DONE``` - used when an order list has finished executing and is no longer active.
            * ```REJECT``` - used when ListStatus is responding to a failed action. (either order list placement or cancellation)
        1. ```ContingencyType```
            * ```OCO``` - specifies the type of order list.

    * New Endpoints:
        * POST api/v3/order/oco
        * DELETE api/v3/orderList
        * GET api/v3/orderList

* ```recvWindow``` cannot exceed 60000.
* New `intervalLetter` values for headers:
    * SECOND => S
    * MINUTE => M
    * HOUR => H
    * DAY => D
* New Headers `X-MBX-USED-WEIGHT-(intervalNum)(intervalLetter)` will give your current used request weight for the (intervalNum)(intervalLetter) rate limiter. For example, if there is a one minute request rate weight limiter set, you will get a `X-MBX-USED-WEIGHT-1M` header in the response. The legacy header `X-MBX-USED-WEIGHT` will still be returned and will represent the current used weight for the one minute request rate weight limit.
* New Header `X-MBX-ORDER-COUNT-(intervalNum)(intervalLetter)`that is updated on any valid order placement and tracks your current order count for the interval; rejected/unsuccessful orders are not guaranteed to have `X-MBX-ORDER-COUNT-**` headers in the response.
    * Eg. `X-MBX-ORDER-COUNT-1S` for "orders per 1 second" and `X-MBX-ORDER-COUNT-1D` for orders per "one day"
* GET api/v1/depth now supports `limit` 5000 and 10000; weights are 50 and 100 respectively.
* GET api/v1/exchangeInfo has a new parameter `ocoAllowed`.

USER DATA STREAM
* ```executionReport``` event now contains "g" which has the ```orderListId```; it will be set to -1 for non-OCO orders.
* New Event Type ```listStatus```; ```listStatus``` is sent on an update to any OCO order.
* New Event Type ```outboundAccountPosition```; ```outboundAccountPosition``` is sent any time an account's balance changes and contains the assets that could have changed by the event that generated the balance change (a deposit, withdrawal, trade, order placement, or cancellation).

NEW ERRORS
* **-1131 BAD_RECV_WINDOW**
    * ```recvWindow``` must be less than 60000
* **-1099 Not found, authenticated, or authorized**
    * This replaces error code -1999

NEW -2011 ERRORS
* **OCO_BAD_ORDER_PARAMS**
    * A parameter for one of the orders is incorrect.
* **OCO_BAD_PRICES**
    * The relationship of the prices for the orders is not correct.
* **UNSUPPORTED_ORD_OCO**
    * OCO orders are not supported for this symbol.

---
### 2019-03-12

REST API
* X-MBX-USED-WEIGHT header added to Rest API responses.
* Retry-After header added to Rest API 418 and 429 responses.
* When canceling the Rest API can now return `errorCode` -1013 OR -2011 if the symbol's `status` isn't `TRADING`.
* `api/v1/depth` no longer has the ignored and empty `[]`.
* `api/v3/myTrades` now returns `quoteQty`; the price * qty of for the trade.

WEBSOCKET STREAMS
* `<symbol>@depth` and `<symbol>@depthX` streams no longer have the ignored and empty `[]`.

SYSTEM IMPROVEMENTS
* Matching Engine stability/reliability improvements.
* Rest API performance improvements.

---
### 2018-11-13
REST API
* Can now cancel orders through the Rest API during a trading ban.
* New filters: `PERCENT_PRICE`, `MARKET_LOT_SIZE`, `MAX_NUM_ICEBERG_ORDERS`.
* Added `RAW_REQUESTS` rate limit. Limits based on the number of requests over X minutes regardless of weight.
* /api/v3/ticker/price increased to weight of 2 for a no symbol query.
* /api/v3/ticker/bookTicker increased weight of 2 for a no symbol query.
* DELETE /api/v3/order will now return an execution report of the final state of the order.
* `MIN_NOTIONAL` filter has two new parameters: `applyToMarket` (whether or not the filter is applied to MARKET orders) and `avgPriceMins` (the number of minutes over which the price averaged for the notional estimation).
* `intervalNum` added to /api/v1/exchangeInfo limits. `intervalNum` describes the amount of the interval. For example: `intervalNum` 5, with `interval` minute, means "every 5 minutes".

#### Explanation for the average price calculation:
1. (qty * price) of all trades / sum of qty of all trades over previous 5 minutes.

2. If there is no trade in the last 5 minutes, it takes the first trade that happened outside of the 5min window.
   For example if the last trade was 20 minutes ago, that trade's price is the 5 min average.

3. If there is no trade on the symbol, there is no average price and market orders cannot be placed.
   On a new symbol with `applyToMarket` enabled on the `MIN_NOTIONAL` filter, market orders cannot be placed until there is at least 1 trade.

4. The current average price can be checked here: `https://api.binance.com/api/v3/avgPrice?symbol=<symbol>`
   For example:
   https://api.binance.com/api/v3/avgPrice?symbol=BNBUSDT

USER DATA STREAM
* `Last quote asset transacted quantity` (as variable `Y`) added to execution reports. Represents the `lastPrice` * `lastQty` (`L` * `l`).

---
### 2018-07-18
REST API
*  New filter: `ICEBERG_PARTS`
*  `POST api/v3/order` new defaults for `newOrderRespType`. `ACK`, `RESULT`, or `FULL`; `MARKET` and `LIMIT` order types default to `FULL`, all other orders default to `ACK`.
*  POST api/v3/order `RESULT` and `FULL` responses now have `cummulativeQuoteQty`
*  GET api/v3/openOrders with no symbol weight reduced to 40.
*  GET api/v3/ticker/24hr with no symbol weight reduced to 40.
*  Max amount of trades from GET /api/v1/trades increased to 1000.
*  Max amount of trades from GET /api/v1/historicalTrades increased to 1000.
*  Max amount of aggregate trades from GET /api/v1/aggTrades increased to 1000.
*  Max amount of aggregate trades from GET /api/v1/klines increased to 1000.
*  Rest API Order lookups now return `updateTime` which represents the last time the order was updated; `time` is the order creation time.
*  Order lookup endpoints will now return `cummulativeQuoteQty`. If `cummulativeQuoteQty` is < 0, it means the data isn't available for this order at this time.
*  `REQUESTS` rate limit type changed to `REQUEST_WEIGHT`. This limit was always logically request weight and the previous name for it caused confusion.

USER DATA STREAM
*  `cummulativeQuoteQty` field added to order responses and execution reports (as variable `Z`). Represents the cummulative amount of the `quote` that has been spent (with a `BUY` order) or received (with a `SELL` order). Historical orders will have a value < 0 in this field indicating the data is not available at this time. `cummulativeQuoteQty` divided by `cummulativeQty` will give the average price for an order.
*  `O` (order creation time) added to execution reports

---
### 2018-01-23
* GET /api/v1/historicalTrades weight decreased to 5
* GET /api/v1/aggTrades weight decreased to 1
* GET /api/v1/klines weight decreased to 1
* GET /api/v1/ticker/24hr all symbols weight decreased to number of trading symbols / 2
* GET /api/v3/allOrders weight decreased to 5
* GET /api/v3/myTrades weight decreased to 5
* GET /api/v3/account weight decreased to 5
* GET /api/v1/depth limit=500 weight decreased to 5
* GET /api/v1/depth limit=1000 weight decreased to 10
* -1003 error message updated to direct users to websockets

---
### 2018-01-20
* GET /api/v1/ticker/24hr single symbol weight decreased to 1
* GET /api/v3/openOrders all symbols weight decreased to number of trading symbols / 2
* GET /api/v3/allOrders weight decreased to 15
* GET /api/v3/myTrades weight decreased to 15
* GET /api/v3/order weight decreased to 1
* myTrades will now return both sides of a self-trade/wash-trade

---
### 2018-01-14
* GET /api/v1/aggTrades weight changed to 2
* GET /api/v1/klines weight changed to 2
* GET /api/v3/order weight changed to 2
* GET /api/v3/allOrders weight changed to 20
* GET /api/v3/account weight changed to 20
* GET /api/v3/myTrades weight changed to 20
* GET /api/v3/historicalTrades weight changed to 20
