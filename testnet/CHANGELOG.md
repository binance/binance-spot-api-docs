# CHANGELOG for Binance SPOT Testnet

**Last Updated: 2026-02-04**

**Note:** All features here will only apply to the [SPOT Testnet](https://testnet.binance.vision/).
This is not always synced with the live exchange.

### 2026-02-04

**Data reset**

All data on the Spot Test Network will be deleted today according to the periodic reset procedure. See [F.A.Q.](../faqs/testnet.md#faq-periodic-reset) for more details.

REST and WebSocket API:

* Reminder that SBE 3:0 schema will be retired on 2026-02-06, [6 months after being deprecated](../faqs/sbe_faq.md#regarding-legacy-support).

---

### 2026-02-02

* Documented that [FIX Drop Copy session](fix-api.md#fix-api-drop-copy-sessions) data is delayed by 1 second. This has been the delay since the inception of the FIX API.

---

### 2026-01-27

* [ICEBERG_PARTS](https://developers.binance.com/docs/binance-spot-api-docs/testnet/filters#iceberg_parts) will be increased to 50 for all symbols today.

---

### 2026-01-26

* Added undocumented `recvWindow` to `userDataStream.subscribe.signature`.

---

### 2026-01-21

#### REST and WebSocket API

Following the announcement from [2025-10-24](#2025-10-24), the following endpoints/methods will no longer be available starting from **2026-02-04, 07:00 UTC**

REST API
* `POST /api/v3/userDataStream`
* `PUT /api/v3/userDataStream`
* `DELETE /api/v3/userDataStream`

WebSocket API

* `userDataStream.start`
* `userDataStream.ping`
* `userDataStream.stop`

---

### 2026-01-07

**Data reset**

All data on the Spot Test Network will be deleted today according to the periodic reset procedure. See [F.A.Q.](../faqs/testnet.md#faq-periodic-reset) for more details.

---

### 2025-12-18

* Updated [FIX SBE documentation](fix-api.md#fix-sbe)
* Clarified User Data Stream documentation regarding [`eventStreamTerminated`](user-data-stream.md#event-stream-terminated).
* Assets `这是测试币` and `456` and symbol `这是测试币456` have been added for testing endpoints/methods with a Unicode symbol. Balances for both assets have been distributed to all accounts.

---

### 2025-12-17

#### REST API

* When calling endpoints that require signatures, percent-encode payloads before computing signatures. Requests that do not follow this order will be rejected with [`-1022 INVALID_SIGNATURE`](errors.md#-1022-invalid_signature). Please review and update your signing logic accordingly.
* Updated documentation for REST API regarding [Signed Endpoints examples for placing an order](https://developers.binance.com/docs/binance-spot-api-docs/testnet/rest-api/request-security#signed-endpoint-examples-for-post-apiv3order)

#### WebSocket API

* Updated documentation for WebSocket API regarding [SIGNED request security](https://developers.binance.com/docs/binance-spot-api-docs/testnet/websocket-api/request-security#signed-request-security)

---

### 2025-12-15

**Clarification Regarding UTF-8 Encoding:**

* In [FIX](fix-api.md), [REST](https://developers.binance.com/docs/binance-spot-api-docs/testnet/rest-api/general-api-information), and [WebSocket APIs](https://developers.binance.com/docs/binance-spot-api-docs/testnet/websocket-api/general-api-information), if your request contains a symbol name containing non-ASCII characters, then the response may contain non-ASCII characters encoded in UTF-8.
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
* Added documentation in REST, and WebSocket APIs stating:
<br> **Please avoid SQL keywords in requests** as they may trigger a security block by a WAF (Web Application Firewall) rule. <br> See https://www.binance.com/en/support/faq/detail/360004492232 for more details.

---

### 2025-12-04

* [QuickFix Schema for FIX OE](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml) has been updated to add `ExecutionReportType` and `SBESchemaVersionDeprecated` for FIX SBE support.
* [QuickFix Schema for FIX MD](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml) has been updated to add `SBESchemaVersionDeprecated` for FIX SBE support.

---

### 2025-11-28

**Notice: The following changes will be deployed starting from 2025-12-01 2:00 UTC and may take several hours to complete**

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

**Notice: The following changes will occur at approximately 2025-12-02 11:00 UTC**:
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
* STP mode [`TRANSFER`](../faqs/stp_faq.md) has been added. The exact date that STP `TRANSFER` will be enabled has not yet been determined.
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

### 2025-11-25

* [`ICEBERG_PARTS`](https://developers.binance.com/docs/binance-spot-api-docs/testnet/filters#iceberg_parts) will be increased to 25 for all symbols.

---

### 2025-11-14

* All Market Tickers Stream (`!ticker@arr`) has been deprecated; This means this will be removed both from the documentation and from our systems at a later date. More details to follow.
* Please use [`<symbol>@ticker`](https://developers.binance.com/docs/binance-spot-api-docs/web-socket-streams#individual-symbol-ticker-streams) or [`!miniTicker@arr`](https://developers.binance.com/docs/binance-spot-api-docs/web-socket-streams#all-market-mini-tickers-stream) instead.

---

### 2025-11-12

* The steps on [how to manage a local order book correctly](https://developers.binance.com/docs/binance-spot-api-docs/testnet/web-socket-streams#how-to-manage-a-local-order-book-correctly) has been corrected.

---

### 2025-11-11

#### SBE Market Data

* **At 2025-11-11 07:00 UTC, the update speed of `<symbol>@depth` and `<symbol>@depth20` streams will be changed to 50ms**.
  * This change will apply automatically to all users of SBE Market Data and doesn't require any action.
  * The total amount of data received per second will be increased (up to 2x).
  * These new update speeds will take effect on the live exchange at **2025-11-26 07:00 UTC**.
  * [SBE Market Data](sbe-market-data-streams.md) has been updated to reflect these changes.

---

### 2025-11-10

* "Last Updated" dates will be removed from all documents except for CHANGELOG.
* Moving forward, CHANGELOG will be the source of reference for when changes were made to any document.

---

### 2025-11-05

**Data reset**

All data on the Spot Test Network will be deleted today according to the periodic reset procedure. See [F.A.Q.](../faqs/testnet.md#faq-periodic-reset) for more details.

---

### 2025-10-24

#### SBE

* SBE: schema 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml)) has been updated to support [listenToken Subscription Methods](https://developers.binance.com/docs/margin_trading/trade-data-stream/Listen-Token-Websocket-API) for Margin Trading.

#### REST and WebSocket API

Following the announcement from [2025-04-01](#2025-04-01), all documentation related with `listenKey` for use on `wss://stream.binance.com` has been removed.

Please refer to the list of requests and methods below for more information.

The features will remain available until a future retirement announcement is made.

REST API
* `POST /api/v3/userDataStream`
* `PUT /api/v3/userDataStream`
* `DELETE /api/v3/userDataStream`

WebSocket API

* `userDataStream.start`
* `userDataStream.ping`
* `userDataStream.stop`

---

### 2025-10-17

**Notice: The following changes will be enabled at 2025-10-17 07:00 UTC**

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
* When the parameter `symbolStatus=<STATUS>` is provided, only symbols whose trading status matches the specified `STATUS` will be included in the response:
    * If a single symbol is specified using the `symbol=<SYMBOL>` parameter and its trading status does not match the given `STATUS`, the endpoint will return error code [`-1220 SYMBOL_DOES_NOT_MATCH_STATUS`](./errors.md#-1220-symbol_does_not_match_status).
    * If multiple symbols are specified using the `symbols=[...]` parameter, the response will be an array that excludes any symbols whose trading status does not match `STATUS`. If no symbols from the symbols parameter have a trading status that matches `STATUS`, the response is an empty array.
    * For endpoints where the `symbol` and `symbols` parameters are optional, omitting these parameters is treated as if all symbols had been specified in the `symbols=[...]` parameter. See the previous line for the behavior of `symbolStatus=<STATUS>`.

---

### 2025-10-08

#### FIX API

**Notice: The following changes will be enabled at 2025-10-08 07:00 UTC**

* Updated [QuickFIX Schema](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml) for FIX Market Data:
  * Updated RecvWindow (25000) to reflect microsecond support announced on [2025-08-05](#2025-08-05).
  * Updated [InstrumentList `<y>`](fix-api.md#instrumentlist) message:
    * Added fields: `StartPriceRange`, `EndPriceRange`.
    * Made the following fields optional: `MinTradeVol`, `MaxTradeVol`, `MinQtyIncrement`, `MarketMinTradeVol`, `MarketMaxTradeVol`, `MarketMinQtyIncrement`, `MinPriceIncrement`.
  * The changes to InstrumentList `<y>` are breaking changes. Please update to the new schema.

---

### 2025-10-01

**Data reset**

All data on the Spot Test Network will be deleted today according to the periodic reset procedure. See [F.A.Q.](../faqs/testnet.md#faq-periodic-reset) for more details.

REST and WebSocket API:

* Reminder that SBE 2:1 schema will be retired on 2025-10-02, [6 months after being deprecated](../faqs/sbe_faq.md#regarding-legacy-support).
* The [SBE lifecycle for Testnet](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/sbe_schema_lifecycle_testnet.json) has been updated to reflect this change.

---

### 2025-09-24

**Notice: The following changes will be deployed on 2025-09-24, starting at 7:00 UTC and may take several hours to complete.**

* Added an endpoint to retrieve the list of filters relevant to an account on a given symbol. This is the only endpoint that shows if an account has `MAX_ASSET` filters applied to it.
  * REST API: [`GET /api/v3/myFilters`](rest-api.md#myFilters)
  * WebSocket API: [`myFilters`](web-socket-api.md#myFilters)
* Comments in **SBE: schema 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml))** have been added, modified, and removed. Although there is no need for users of `3:1` to update to this version of the file, we advise updating to maintain consistency.
* Added documentation for filter [`MAX_ASSET`](filters.md#max_asset).
  * In `Testnet` only: all accounts have a `MAX_ASSET` filter for asset `JPY` with value set to `1000000`.

---

### 2025-09-18

* Updated documentation for `recvWindow` to reflect microsecond support announced on [2025-08-05](#2025-08-05).
  * REST API: [Timing Security](rest-api.md#timingsecurity)
  * WebSocket API: [Timing Security](web-socket-api.md#timingsecurity)

---

### 2025-09-12

* The [QuickFix schema for FIX Order Entry](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml) has been updated to support Pegged Orders.
* Updated FIX API Documentation for `RecvWindow` in
  * [Message Components](fix-api.md#header)
  * [Timing Security](fix-api.md#timing-security)

---

### 2025-09-05

**Data reset**

All data on the Spot Test Network will be deleted today according to the periodic reset procedure. See [F.A.Q.](../faqs/testnet.md#faq-periodic-reset) for more details.

---

### 2025-08-28

* Updated SBE FAQ section [regarding legacy support](../faqs/sbe_faq.md#regarding-legacy-support) to include more details on schema compatibility and explain `NonRepresentable` and `NonRepresentableMessage`.

---

### 2025-08-26

* Updated "Request Security" documentation for [REST API](rest-api.md#request-security) and [WebSocket API](web-socket-api.md#request-security) with no functional changes.

---

### 2025-08-25

* **SBE: schema 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml))** will be updated on **2025-08-25 at 05:00 UTC**
  * The following fields have been renamed because the [SbeTool](../faqs/sbe_faq.md#generate-sbe-decoders) code generator has been found to generate Java code that does not compile.
    * Although only users impacted by this issue need to update the schema, we advise all users to upgrade to the latest version to maintain consistency.
    * Message `MaxAssetFilter`
      * field `limitExponent` renamed to `qtyExponent`
      * field `limit` renamed to `maxQty`

---

### 2025-08-19

* `userDataStream.subscribe` returns `subscriptionId` in the responses. <br> This was missed in a [previous](#2025-08-05) changelog entry.

---

### 2025-08-07

* Updated FIX API documentation
  * [FIX Market Data limits](fix-api.md#connection-limits): The subscription limit has always been present but was undocumented.
  * [On message processing order](fix-api.md#on-message-processing-order): Reworded and reformatted.

**Notice: The following will be enabled on 2025-08-08, 07:00 UTC**

* Filter [`MAX_NUM_ORDER_LISTS`](filters.md#max-num-order-lists), is enabled with the limit of 20 per symbol.

---

### 2025-08-05

**Notice: The following changes will be deployed on 2025-08-06, starting 7:00 UTC and may take several hours to complete.** <br>
Please consult the Spot Test Network's [homepage](https://testnet.binance.vision/) to be informed of the release completion.

#### General Changes

* The [pegged order](../faqs/pegged_orders.md) functionality is now available.
  * Exchange Information requests emit the field `pegInstructionsAllowed`.
  * The following conditional fields `pegPriceType`, `pegOffSetType`, `pegOffsetValues`, and `peggedPrice` appear in responses of the following requests if the order was a pegged order:
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
    * `OrdType(4)` supports new value `P(PEGGED)`
    * Tags `PegOffsetValue(211)`, `PegPriceType(1094)`, `PegMoveType(835)`, and `PegOffsetType(836)` have been added to the following messages:
      * NewOrderSingle `<D>`
      * NewOrderList `<E>`
      * OrderCancelRequestAndNewOrderSingle `<XCN>`
      * When placing an order, the `ExecutionReport` `<8>` message will echo back `PegInstructions`, with an extra optional field `PeggedPrice (839)`.
  * New error messages for pegged orders are added. Please see the [Errors](errors.md) document for more information.
* Changes with `recvWindow`:
  * A third check is made after your message leaves the message broker just before it is sent to the Matching Engine.
    * This does not cover potential delays inside the Matching Engine itself.
  * `recvWindow` supports microseconds.
    * The value is still specified in milliseconds, but can now take a decimal component to specify it with higher precision.
    * This means that the parameter supports a **maximum precision of 3 decimal places**. (e.g. 6000.346)
    * APIs affected:
        * FIX API
        * REST API
        * WebSocket API
* The following requests have a new structure called `specialCommission`. See [Commission Rates](../faqs/commission_faq.md).
  * REST API
    * `GET /api/v3/account/commission`
    * `POST /api/v3/order/test` with `computeCommissionRates=true`
    * `POST /api/v3/sor/order/test` with `computeCommissionRates=true`
  * WebSocket API
    * `account.commission`
    * `order.test` with `computeCommissionRates=true`
    * `sor.order.test` with `computeCommissionRates=true`
* The new [`MAX_NUM_ORDER_AMENDS`](https://github.com/binance/binance-spot-api-docs/blob/master/testnet/filters.md#max_num_order_amends) filter is enabled with a limit of 10 amendments per order.
* New error codes `-1120` and `1211`. See [Errors](errors.md) for more information.
* **SBE: A new schema 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml)) is available.**
  * The current schema 3:0 ([spot_3_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_0.xml)) is deprecated and will be retired in 6 months as per our schema deprecation policy.
  * Changes in schema 3:1:
    * `ExchangeInfoResponse`: new field `pegInstructionsAllowed`
    * `ExecutionReportEvent`: new fields `pricePeg`, `pricePegOffsetLevel`, `peggedPrice`
    * `UserDataStreamSubscribeResponse`: new field `subscriptionId`
    * New field `subscriptionId` for all user data stream events.
    * Field `apiKey` renamed to `loggedOnApiKey` for `WebSocketSessionLogonResponse`, `WebSocketSessionStatusResponse` and WebSocketSessionLogoutResponse
    * `OrderTestWithCommissionsResponse`: 2 new fields `specialCommissionForOrderMaker` and `specialCommissionForOrderTaker`
    * `AccountCommissionResponse`: 4 new fields `specialCommissionMaker`, `specialCommissionTaker`, `specialCommissionBuyer` and `specialCommissionSeller`
    * Support for `EXCHANGE_MAX_NUM_ORDER_LISTS`, `MAX_NUM_ORDER_LISTS`, and `MAX_NUM_ORDER_AMENDS` filters.
    * `ExecutionReportEvent`: fields `rejectReason` and `origClientOrderId` now show their default values in SBE format to match the JSON format.
    * `NonRepresentableMessage`: New message added to represent a message that cannot be represented in this schema ID and version. Receipt of this message indicates that something should be available, but it is not representable using the SBE schema currently in use.
* Query order lists requests will first query the data in the cache, and if it cannot be found will query the database.
  * REST API: `GET /api/v3/openOrderLists`
  * WebSocket API: `openOrderLists.status`
* Orders with cumulative quantity of 0 in the final state `EXPIRED_IN_MATCH` (i.e. the order expired due to STP) will be archived after 90 days.
* Bug fix: The Matching Engine no longer accepts order lists that exceed the order count filter limits. Affected filters:
  * `MAX_NUM_ORDERS`
  * `MAX_ALGO_ORDERS`
  * `MAX_ICEBERG_ORDERS`
  * `EXCHANGE_MAX_NUM_ORDERS`
  * `EXCHANGE_MAX_ALGO_ORDERS`
  * `EXCHANGE_MAX_ICEBERG_ORDERS`

#### WebSocket API

* A single WebSocket connection can subscribe to multiple User Data Streams at once.
  * Only one subscription per account is allowed on a single connection.
* Method `userDataStream.subscribe.signature` has been added that allows you to subscribe to the User Data Stream without needing to login first.
  * This also doesn’t require an Ed25519 API Key, and can work with any [API Key type](../faqs/api_key_types.md).
  * For [SBE support](../faqs/sbe_faq.md) you need to use schema 3:1 at least.
* Method `session.subscriptions` has been added that lists all the subscriptions active for the current session.
* The meaning of the field `userDataStream` in the session requests has changed slightly.
  * Previously, this returned `true` if you were subscribed to the user data stream of your logged-on account.
  * Now it returns `true` if you have at least one active user data stream subscription
    * `true` - If there is at least one subscription active
    * `false` - If there are no active subscriptions
* `userDataStream.unsubscribe` supports closing multiple subscriptions.
  * When called with no parameter, this will close all subscriptions.
  * When called with `subscriptionId`, this will attempt to close the subscription matching that Id, if it exists.
  * The authorization for this request has been changed to `NONE`.

#### User Data Stream

* Field `subscriptionId` has been added to the User Data Stream events payload when listening through the [WebSocket API](web-socket-api.md#user_data_stream_subscribe). This will identify which subscription the event is coming from.

#### FIX API

* When a client sends a reject message, the FIX API will no longer send the client back a Reject `<3>` message.
Error messages are clearer when a tag is invalid, missing a value, or when the field value is empty or malformed
  * If the tag number was invalid, you will receive the error:
    ```json
    { "code": -1169, "msg": "Invalid tag number." }
    ```
  * If a valid tag was specified without a value, you will receive the error:
    ```json
    { "code": -1177, "msg": "Tag specified without a value." }
    ```
  * If the field value was empty or malformed, you will still receive the error:
    ```json
    { "code": -1102, "msg": "Field value was empty or malformed." }
    ```
---

### 2025-07-02

**Data reset**

All data on the Spot Test Network will be deleted today according to the periodic reset procedure. (see [F.A.Q.](../faqs/testnet.md#faq-periodic-reset) for more details)

---

### 2025-06-04

**Data reset**

All data on the Spot Test Network will be deleted today according to the periodic reset procedure. (see [F.A.Q.](../faqs/testnet.md#faq-periodic-reset) for more details)

---

### 2025-05-28

* Documented API timeout value and error under General API Information for each API:
    * [FIX](fix-api.md#general-api-information)
    * [REST](rest-api.md#general-api-information)
    * [WebSocket](web-socket-api.md#general-api-information)

---

### 2025-05-22

REST and WebSocket API:

* Reminder that SBE 2:0 schema will be retired on 2025-05-28, [6 months after being deprecated](../faqs/sbe_faq.md).
* The [SBE lifecycle for Testnet](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/sbe_schema_lifecycle_testnet.json) has been updated to reflect this change.

---

### 2025-05-21
**Notice: The following changes will happen at 2025-05-21 7:00 UTC.**

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

* **Notice: The following changes will happen at 2025-04-25 09:00 UTC.**
* The following request weights will be increased from 1 to 4:
  * REST API: `PUT /api/v3/order/amend/keepPriority`
  * WebSocket API: `order.amend.keepPriority`
  * The documentation for both REST and WebSocket API has been updated to reflect the upcoming changes.
* Clarified that `SEQNUM` in the FIX-API is a 32-bit unsigned integer that rolls over. This has been the `SEQNUM` data type since the inception of the FIX-API.

---

### 2025-04-21

* **[Order Amend Keep Priority](https://github.com/binance/binance-spot-api-docs/blob/master/faqs/order_amend_keep_priority.md) is now enabled on all symbols.**
* **[Self-trade prevention mode `DECREMENT`](https://github.com/binance/binance-spot-api-docs/blob/master/faqs/stp_faq.md) is now enabled on all symbols.**

---

### 2025-04-01

**Notice:** The following changes will be deployed tomorrow **April 2, 2025 starting at 7:00 UTC** and may take several hours to complete. <br>
Please consult the Spot Test Network's [homepage](https://testnet.binance.vision/) to be informed of the release completion.

#### New Features

* **[Order Amend Keep Priority](https://github.com/binance/binance-spot-api-docs/blob/master/faqs/order_amend_keep_priority.md) is now available.**
    * FIX API: New Order Entry Messages **OrderAmendKeepPriorityRequest** and **OrderAmendReject**
    * REST API: `PUT /api/v3/order/amend/keepPriority`
    * WebSocket API: `order.amend.keepPriority`
    * You can check the new `allowAmend` field in Exchange Information Requests to see if it's enabled on a given symbol:
        * REST API: `GET /api/v3/exchangeInfo`
        * WebSocket API: `exchangeInfo`
* **Self-trade prevention mode `DECREMENT` is now available.**
    * Instead of expiring one or both orders, `DECREMENT` mode decreases the available quantity of both orders by increasing the `prevented quantity` of both orders by the amount of the prevented match.
    * This can expire the orders if their `filled quantity` \+ `prevented quantity` >= `order quantity`.
    * You can check the `allowedSelfTradePreventionModes` field in Exchange Information Requests to see if this mode is enabled on a given symbol.


#### General Changes

* **Important:** The following legacy URLs will be **removed in May 2025**. Please change to the new URLs as soon as possible:

|Legacy URL | Latest URL|
|---        | ---|
|`wss://testnet.binance.vision/ws-api/v3`|`wss://ws-api.testnet.binance.vision/ws-api/v3`|
|`wss://testnet.binance.vision/ws` |`wss://stream.testnet.binance.vision/ws`|

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
* Previously, the request weight for myTrades was 20 regardless of the parameters provided. Now, if you provide `orderId`, the request weight is 5.
    * REST API: `GET /api/v3/myTrades`
    * WebSocket API: `myTrades`
* If the unfilled order count for `intervalNum:DAY` is exceeded, the unfilled order count for `intervalNum:SECOND` is no longer incremented.
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
* New Error code `-2038` for order amend keep priority requests that fail.
* New messages for error code `-1034`.


#### FIX API

* The [QuickFix schema for FIX OE](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml) is updated to support the Order Amend Keep Priority feature and new STP mode, `DECREMENT`.
* FIX Order Entry connection limits will be a maximum of 10 concurrent connections per account.
* The connection rate limits are now enforced. Note that these limits are checked independently for both the account and the IP address.
    * FIX Order Entry: 15 connection attempts within 30 seconds
    * FIX Drop Copy: 15 connection attempts within 30 seconds
    * FIX Market Data: 300 connection attempts within 300 seconds
* News `<B>` contains a countdown until disconnection in the Headline field.
    * Following the completion of this update, when the server enters maintenance, a `News` message will be sent to clients **every 10 seconds for 10 minutes**. After this period, clients will be logged out and their sessions will be closed.
* OrderCancelRequest `<F>` and OrderCancelRequestAndNewOrderSingle `<XCN>` will now allow both `orderId` and `clientOrderId`.
* FIX API verifies that `EncryptMethod(98)` is 0 at Logon `<A>`.


#### User Data Streams

* **Receiving user data streams on stream.testnet.binance.vision using a `listenKey` is now deprecated.**
    * This feature will be removed from our systems at a later date.
* **Instead, you should get user data updates by subscribing to the [User Data Stream on the WebSocket API](https://developers.binance.com/docs/binance-spot-api-docs/testnet/websocket-api/user-data-stream-requests).**
    * This should offer slightly better performance (lower latency).
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

#### SBE

* **A new schema 3:0 ([spot_3_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_0.xml)) is now available.**
    * The current schema 2:1 ([spot_2_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_1.xml)) is now deprecated and will be retired in 6 months as per our schema deprecation policy.
    * Note that trying to use schema 3:0 before it is released will result in an error.
* Changes in schema 3:0:
    * Support for Order Amend Keep Priority:
        * Added field `amendAllowed` to ExchangeInfoResponse.
        * New Messages `OrderAmendmentsResponse` and `OrderAmendKeepPriorityResponse`
    * Breaking changes:
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

### 2025-03-31

* Added a clarification on the performance of canceling an order.

---

### 2025-03-13

* **Notice: The following changes will happen on March 13,2025 at 05:00 UTC:**
  * FIX Drop Copy sessions will have a limit of **60 messages per minute**.
  * FIX Market Data sessions will have a limit of **2000 messages per minute**.
  * The FIX API documentation has been updated to reflect the upcoming changes.

---

### 2025-03-05

* **Notice: This is in the process of being deployed. Please consult the Spot Test Network's [homepage](https://testnet.binance.vision/) to be informed of the release completion.** <br>
  The following request weights will be increased from 2 to 4:
  * REST API: `GET /api/v3/aggTrade`
  * WebSocket API: `trades.aggregate`
* The documentation for both REST and WebSocket API has been updated to reflect the upcoming changes.

---

### 2025-02-28

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
