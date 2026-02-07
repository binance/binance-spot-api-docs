# 更新日志

**最近更新： 2026-02-02**

### 2026-02-02

* 记录了 [FIX Drop Copy 会话](fix-api_CN.md#fix-api-drop-copy-sessions) 数据存在 1 秒的延迟。自 FIX API 推出以来，该延迟一直存在。

---

### 2026-01-29

* [模拟交易](./demo-mode/general-info_CN.md) 现已开放。
* 有关现货模拟交易环境因维护而不可用的时间信息，请参阅 [模拟交易更新日志](./demo-mode/CHANGELOG_CN.md)。

---

### 2026-01-27

**通知：以下变更将于 2026-02-11 07:00 UTC 生效**：

* 所有交易对的 [ICEBERG_PARTS](https://developers.binance.com/docs/zh-CN/binance-spot-api-docs/filters#iceberg_parts-%E5%86%B0%E5%B1%B1%E8%AE%A2%E5%8D%95%E6%8B%86%E5%88%86%E6%95%B0) 将增加到 50。

---

### 2026-01-26

* 在 `userDataStream.subscribe.signature` 中添加了字段 `recvWindow`的说明。

---

### 2026-01-21

根据 **2025-10-24** 的公告，以下端点和方法将从 **2026-02-20 07:00 UTC** 起不再可用。

## REST API

- `POST /api/v3/userDataStream`
- `PUT /api/v3/userDataStream`
- `DELETE /api/v3/userDataStream`

## WebSocket API

- `userDataStream.start`
- `userDataStream.ping`
- `userDataStream.stop`

---

### 2025-12-18

* [FIX SBE 文档](fix-api_CN.md#fix-sbe) 已更新。
* 澄清了用户数据流文档中关于 [`eventStreamTerminated`](user-data-stream_CN.md#event-stream-terminated) 的说明
* 资产 `这是测试币` 和 `456` 以及交易对 `这是测试币456` 已被添加到 [现货测试网](http://testnet.binance.vision) 以供用户测试含有 Unicode 交易对的端点/方法。 预知详情，请参考 [现货测试网的相关更新日志](https://developers.binance.com/docs/binance-spot-api-docs/testnet)。

---

### 2025-12-17

#### 时效性通知

* **以下有关于REST API变更将在 2026-01-15 07:OO UTC 发生:** <br> 调用需要签名的接口时，请在计算签名之前对 payload 进行百分比编码（percent-encode）。不符合此顺序的请求将被拒绝，并返回错误代码 [`-1022 签名不正确`](errors_CN.md#-1022-invalid_signature)。请检查并相应地更新您代码中的签名逻辑部分。这项功能现已在 [SPOT Testnet](http://testnet.binance.vision) 上启用。

#### REST API

* 更新了 REST API 文档中有关于 [签名请求示例](rest-api_CN.md#hmac-keys) 的部分。

#### WebSocket API

* 更新了 WebSocket API 文档中有关于 [签名请求示例](web-socket-api_CN.md#hmac) 的部分。

---

### 2025-12-15

**关于 UTF-8 编码的说明：**

* 在 [FIX API](fix-api_CN.md)、[REST API](rest-api_CN.md) 和 [WebSocket API](web-socket-api_CN.md) 中，如果您的请求包含非 ASCII 字符的交易对名称，那么响应中可能包含以 UTF-8 编码的非 ASCII 字符。
* 在 REST 和 WebSocket API 中，即使请求本身不包含非 ASCII 字符，某些端点/方法也可能返回包含以 UTF-8 编码的非 ASCII 字符的资产和/或交易对名称。
* 在 [WebSocket 数据流](web-socket-streams_CN.md) 中，如果您的请求包含非 ASCII 字符的交易对名称，那么数据流事件中可能包含以 UTF-8 编码的非 ASCII 字符。
* 在 WebSocket 数据流中，[全市场所有 Symbol 的精简 Ticker](web-socket-streams_CN.md#all-markets-mini-ticker) 和 [全市场滚动窗口统计](web-socket-streams_CN.md#all-market-rolling-window-ticker) 事件可能包含以 UTF-8 编码的非 ASCII 字符。
* 在 [SBE 市场数据流](sbe-market-data-streams_CN.md) 中，如果您的请求包含包含非 ASCII 字符的交易对名称，那么数据流事件中可能包含以 UTF-8 编码的非 ASCII 字符。
* 如果您持有或交易任何名称包含非 ASCII 字符的资产或交易对，那么 [WebSocket 账户接口](user-data-stream_CN.md) 中可能包含以 UTF-8 编码的非 ASCII 字符。
* 为了与币安 API 完全兼容，请务必确保您的代码能够处理 UTF-8 编码的字符串。

---

### 2025-12-09

* [FIX SBE Schema](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot-fixsbe-1_0.xml) 已更新，会在 `MarketDataSnapshot` 中使用 `smallGroupSize16Encoding`。并在增量单个交易对订单簿数据流/深度数据流的 `MDEntrySize` 的字段中使用 `presence="optional"`。
* 更新了关于 [FIX 与 FIX SBE](fix-api.md#fix-vs-fix-sbe-schema) 的文档说明
* 在 REST 和 WebSocket API 文档中新增说明：**请避免在请求中使用 SQL 关键字**，因为这可能会触发 Web 应用防火墙（WAF）规则导致安全拦截。详情请参见 https://www.binance.com/zh-CN/support/faq/detail/360004492232 。

---

### 2025-12-02

**注意： 本节中的变更将逐步推出，预计大约需要两周时间完成。**

#### 常规更改

* 参数 `symbol` 和 `symbols` 现在支持以 UTF-8 编码的 Unicode 值。
* 根据 [2025-11-14](#2025-11-14) 的公告，所有与 `!ticker@arr` 相关的内容已经在文档被移除。
  * 该功能将在未来弃用公告发布前继续可用。
  * 请改用 `<symbol>@ticker` 或 `!miniTicker@arr`。

#### FIX API

* 现在 FIX 消息中可以接受以 UTF-8 编码的 Unicode 值。仅允许以下标签使用：
  * `Currency (15)`
  * `MiscFeeCurr (138)`
  * `Symbol (55)`
  * `SecondarySymbol (25019)`
  * `CounterSymbol (25028)`
  * `SecurityDesc (107)`
* 当 Unicode 出现在非上述标签的值中时，FIX API 会在拒绝消息 `<3>` 中返回 `RefTagID (371)` 标签，指明具体哪个标签不允许包含 Unicode。
* NewOrderList `<E>` 可以接受没有 `TriggerPrice (1102)` 的 `TriggerPriceDirection (1109)`。

#### WebSocket 数据流

* WebSocket 市场数据流支持 URL 编码的 URL。
<br>
<br>

**注意：以下变更预计于 2025-12-18 07:00 UTC 左右生效**：
* [ICEBERG_PARTS](https://developers.binance.com/docs/zh-CN/binance-spot-api-docs/filters#iceberg_parts) 将对所有交易对提升至 25。
* [FIX SBE 支持](fix-api_CN.md) 将上线。
* [一个订单支付另一个订单（OPO）](https://github.com/binance/binance-spot-api-docs/blob/master/faqs/opo_CN.md) 将在所有交易对上线。
  * `opoAllowed` 将开始出现在 `Exchange Information` 请求中，指示每个交易对是否支持一个订单支付另一个订单（OPO）。
    * REST API: `GET /api/v3/exchangeInfo`
    * WebSocket API: `exchangeInfo`
  * OPO 新请求：
    * REST API：
      * `POST /api/v3/orderList/opo`
      * `POST /api/v3/orderList/opoco`
    * WebSocket API：
      * `orderList.place.opo`
      * `orderList.place.opoco`
    * FIX API：
      * NewOrderList `<E>` 新增字段 `OPO (25046)`。请更新至最新 QuickFIX 模式以支持 OPO。
* 新增 STP 模式 [`TRANSFER`](./faqs/stp_faq_CN.md)。STP `TRANSFER` 启用的具体日期尚未确定。
* **SBE：新增 schema 3:2 版本 ([spot_3_2.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_2.xml))。**
  * 当前 schema 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml)) 已被弃用，将根据我们的 schema 弃用政策在 6 个月后停用。
  * 3:2 版本变更：
    * 为 `selfTradePreventionMode` 和 `allowedSelfTradePreventionModes` 新增枚举值 `TRANSFER`。
    * 所有低于 3:1 的 schema 无法返回包含 STP 模式 `TRANSFER` 的响应（例如 Exchange Information、下单、撤单或订单状态查询）。<br>当响应无法用请求的 schema 表示时，将返回错误。
* FIX API 变更：
    * `LastFragment (893)` 被弃用。
      * 这意味着 MarketIncrementalRefresh `<X>` 消息将不再分片，可能包含超过 10,000 条目。
      * 文档已更新以反映此变更。
    * ListStatus `<N>` 将不再发送可选的 `symbol` 字段。
      * 适用于 FIX 订单接入和 FIX Drop Copy。
      * 文档已更新以反映此变更。

---

### 2025-11-14

* 全市场所有交易对的完整Ticker（`!ticker@arr`）已被弃用；这意味着该接口将在后续从文档和系统中移除，具体时间另行通知。
* 请改用 [`<symbol>@ticker`](https://developers.binance.com/docs/zh-CN/binance-spot-api-docs/web-socket-streams#twentyfourhourticker) 或 [`!miniTicker@arr`](https://developers.binance.com/docs/zh-CN/binance-spot-api-docs/web-socket-streams#all-markets-mini-ticker)。

---

### 2025-11-12

* 关于[如何正确管理本地订单簿](https://developers.binance.com/docs/zh-CN/binance-spot-api-docs/web-socket-streams#how-to-maintain-orderbook)的步骤已被修正。

---

### 2025-11-11

#### SBE 市场数据流

* **从 2025 年 11 月 26 日 07:00 UTC 起，`<symbol>@depth` 和 `<symbol>@depth20` 数据流的更新速度将变为 50 毫秒**。
  * 此变更将自动部署到 SBE 市场数据的所有用户端，因此客户无需进行任何操作。
  * 每秒可接收的总数据量将增加（最高可达 2 倍速）。
  * [SPOT 测试网](https://testnet.binance.vision/) 将会从 **2025 年 11 月 11 日 07:00 UTC** 起应用这些变更。
  * [SBE 市场数据流](sbe-market-data-streams_CN.md) 已更新以反映这些变更。

---

### 2025-11-10

* 除了更新日志之外，所有文档中的“最近更新”日期将被移除。
* 今后，更新日志将作为所有文档变更时间的唯一参考来源。

---

### 2025-10-28

**注意：以下变更将于 2025 年 10 月 28 日 04:00 (UTC) 推出，可能需要几个小时才能完成部署。**

* 可选参数 `symbolStatus` 已添加到以下端点：
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
* 当提供参数 `symbolStatus=<STATUS>` 时，只有交易状态与指定的 `STATUS` 相匹配的交易对才会包含在响应中：
    * 如果使用 `symbol=<SYMBOL>` 参数指定单个交易对，但是这个交易对的状态与指定的 `STATUS` 不匹配时，端点将返回错误代码 [`-1220 SYMBOL_DOES_NOT_MATCH_STATUS`](./errors_CN.md#-1220-symbol_does_not_match_status)。
    * 如果使用 `symbols=[...]` 参数指定了多个交易对，那么响应将是一个数组。该数组中将不会包括交易状态与 `STATUS` 不匹配的交易对。当 symbols 参数中所指定的交易对中没有任何一个交易对的状态与 `STATUS` 相匹配时，响应将为一个空数组。
    * 对于 `symbol` 和 `symbols` 参数为可选参数的端点，省略这些参数将被视为在 `symbols=[...]` 参数中指定了所有交易对。有关 `symbolStatus=<STATUS>` 的行为，请参阅上一行。

---

### 2025-10-24

#### SBE

* SBE：schema 3:1（[spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml)）已更新，支持保证金交易的[listenToken 订阅方法](https://developers.binance.com/docs/zh-CN/margin_trading/trade-data-stream/Listen-Token-Websocket-API)。

#### REST 和 WebSocket API

根据 [2025-04-07](#2025-04-07) 的公告，所有关于在 `wss://stream.binance.com` 上使用 `listenKey` 的文档已被移除。请参阅以下请求和方法列表以获取详细信息。
该功能将在未来发布停用公告之前继续可用。

**特此提醒您： 您应该通过订阅 [在 WebSocket API 内的账户信息流](https://developers.binance.com/docs/zh-CN/binance-spot-api-docs/websocket-api/user-data-stream-requests) 来获得用户账户更新。这样可以提供更好的性能（更低的延迟）。**

请参阅下方请求和方法列表来了解更多详情。

这些功能将持续可用，直至另行发布停用公告。

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

REST 和 WebSocket API：

* 注意：根据我们的 SBE 政策，[在被废止 6 个月后](faqs/sbe_faq_CN.md#sbe-schema)， SBE 2:1 模式将于 2025 年 10 月 24 日被禁用。
* [面向生产的 SBE 生命周期](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/sbe_schema_lifecycle_prod.json) 已基于本次更改进行了更新。

---

### 2025-10-08

* 更新了用于 FIX 市场数据的 [QuickFIX Schema](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml)：
  * 更新了 `RecvWindow(25000)`，以便对在 [2025-08-12](#2025-08-12) 公布的微秒级进行支持。
  * 更新了 [`InstrumentList`](fix-api_CN.md#instrumentlist) 消息：
    * 新增字段：`StartPriceRange`、`EndPriceRange`。
    * 以下字段改为可选：`MinTradeVol`、`MaxTradeVol`、`MinQtyIncrement`、`MarketMinTradeVol`、`MarketMaxTradeVol`、`MarketMinQtyIncrement`、`MinPriceIncrement`。
  * **InstrumentList `<y>` 的变更属于破坏性变更，预计将于 2025-10-23 07:00 UTC 左右发布。请在此之前更新到新的模式。**
  * 在[SPOT 测试网](https://testnet.binance.vision/) 上已经启用了这个重大更改。

---

### 2025-09-29

**注意：以下变更将于 2025 年 9 月 29 日 10:00 (UTC) 推出，可能需要几个小时才能完成部署。**

* 新增一个端点，用于检索与指定交易品种的账户相关的过滤器列表。这是唯一一个目前会显示账户是否应用了 `MAX_ASSET` 过滤器的端点。
  * REST API： [`GET /api/v3/myFilters`](rest-api_CN.md#myFilters)
  * WebSocket API： [`myFilters`](web-socket-api_CN.md#myFilters)
* 对 **SBE: schema 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml))** 中的部分注释进行了添加、修改和删除。虽然 `3:1` 用户无需更新到此版本的模式，但我们建议您进行更新以保持一致性。
* 新增了有关过滤器 [`MAX_ASSET`](filters_CN.md#max_asset) 的文档。

---

### 2025-09-18

* 更新了文档中有关 `recvWindow` 的部分， 用以体现在 [2025-08-12](#2025-08-12) 发表的微秒级支持。
  * REST API: [时间同步安全](rest-api_CN.md#timingsecurity)
  * WebSocket API: [时间同步安全](web-socket-api_CN.md#timingsecurity)

---

### 2025-09-12

* 更新了 FIX 订单接入 API 的 [QuickFix 模式](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml)，用以支持挂钩订单。
* 在 FIX API 文档中， 对以下部分的 `RecvWindow` 进行了更新：
  * [消息组件](fix-api_CN.md#header)
  * [时间同步安全](fix-api_CN.md#timingsecurity)

---

### 2025-08-28

* 更新了 “简单二进制编码（SBE）常见问题” 文档中 [关于对旧版本的支持](faqs/sbe_faq_CN.md#regarding-legacy-support) 部分，增加了关于模式兼容性的更多细节，并解释了 `NonRepresentable` 和 `NonRepresentableMessage` 的用法。

---

### 2025-08-26

* 更新了 [REST API](rest-api_CN.md#request-security) 和 [WebSocket API](web-socket-api_CN.md#request-security) 文档的 “请求鉴权类型” 部分，不过相关的功能没有改动。

---

### 2025-08-25

* **SBE: 模式 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml))**将于**2025-08-25 05:00 UTC** 进行更新。
  * 以下字段已被重命名，用于解决通过 [SbeTool](faqs/sbe_faq_CN.md#generate-sbe-decoders) 代码生成器所生成的 Java 代码被发现无法进行编译的问题。
    * 虽然只有受此问题影响的用户才需要更新模式文档，但我们建议所有用户都升级到最新版本以保持一致性。
    * 消息 `MaxAssetFilter`
      * 字段 `limitExponent` 重命名为 `qtyExponent`
      * 字段 `limit` 重命名为 `maxQty`

---

### 2025-08-19

* `userDataStream.subscribe` 在响应中返回 `subscriptionId`。<br> 这在 [previous](#2025-08-12) 变更日志条目中被遗漏了

---

### 2025-08-12

**通知:** 本节中的更改将逐步推出,需要大约两周时间才能完成部署。

#### 常规更改

* 新的错误代码 `-1120` 和 `1211`。有关详情，请参阅 [错误代码汇总](errors_CN.md)。
* 以下请求包含一个名为 `specialCommission` 的新结构。请参阅 [佣金率](faqs/commission_faq_CN.md)。
  * REST API
    * `GET /api/v3/account/commission`
    * `POST /api/v3/order/test` 并设置 `computeCommissionRates=true`
    * `POST /api/v3/sor/order/test` 并设置 `computeCommissionRates=true`
  * WebSocket API
    * `account.commission`
    * `order.test` 并设置 `computeCommissionRates=true`
    * `sor.order.test` 并设置 `computeCommissionRates=true`
* **SBE: 新模式 3:1 ([spot_3_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_1.xml)) 将可以使用。**
  * 现行模式 3:0 ([spot_3_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_0.xml)) 将会被废止并根据我们的废止政策，在6个月内停止使用。
  * 模式 3:1 中的更改：
    * `ExchangeInfoResponse`：增加新字段 `pegInstructionsAllowed`
    * `ExecutionReportEvent`：增加新字段 `pricePeg`，`pricePegOffsetLevel` 和 `peggedPrice`
    * `UserDataStreamSubscribeResponse`：新字段 `subscriptionId`
    * 所有的用户数据流事件都增加了新字段 `subscriptionId`。
    * 对于 `WebSocketSessionLogonResponse`，`WebSocketSessionStatusResponse` 和 `WebSocketSessionLogoutResponse`，字段 `apiKey` 被改名为 `loggedOnApiKey`
    * `OrderTestWithCommissionsResponse`：增加2个新字段 `specialCommissionForOrderMaker` 和 `specialCommissionForOrderTaker`
    * `AccountCommissionResponse`：增加4个新字段 `specialCommissionMaker`，`specialCommissionTaker`，`specialCommissionBuyer` 和 `specialCommissionSeller`
    * 支持 `EXCHANGE_MAX_NUM_ORDER_LISTS`，`MAX_NUM_ORDER_LISTS` 和 `MAX_NUM_ORDER_AMENDS` 过滤器。
    * `ExecutionReportEvent`：字段 `rejectReason` 和 `origClientOrderId` 在 SBE 格式也会显示它们的默认值，以和 JSON 格式保持一致，
    * `NonRepresentableMessage`：添加了一条新消息，用于表示无法使用现有 `schema ID` 和 `version` 来显示的信息。收到此消息则表示应该有可用内容，但不适用于当前使用的 SBE 模式。
* 最终状态为 `EXPIRED_IN_MATCH`（即因 STP 而过期的订单）且累计数量为 0 的订单将在 90 天后做归档处理。
* 查询订单列表请求将首先查询缓存中的数据，如果找不到，再去查询数据库。
  * REST API：`GET /api/v3/openOrderLists`
  * WebSocket API：`openOrderLists.status`

#### WebSocket API

* 单个 WebSocket 连接可以同时订阅多个用户数据流。
  * 单个连接上，每个帐户只能有一个订阅。
* 新增了 `userDataStream.subscribe.signature` 方法，允许您无需事先登录即可订阅用户数据流。
  * 此方法也不需要 Ed25519 API 密钥，并且可以与任何 [API Key 类型](faqs/api_key_types_CN.md) 配合使用。
  * 为了获得 [SBE 支持](faqs/sbe_faq_CN.md)，您至少需要使用架构 3:1。
* 新增了 `session.subscriptions` 方法，用于列出当前会话中所有有效的订阅。
* 会话请求中字段 `userDataStream` 的含义略有变化。
  * 之前，如果您订阅了已登录帐户的用户数据流，则此方法返回 `true`。
  * 现在，如果您至少有一个有效的用户数据流订阅，它将返回 `true`，否则为 `false`。
* `userDataStream.unsubscribe` 支持关闭多个订阅。
  * 调用时不带参数，将关闭所有订阅。
  * 调用时传入 `subscriptionId`，如果该 ID 存在的话，将尝试关闭与该 ID 匹配的订阅。
  * 此请求的授权已更改为 `NONE`。
* 通过 [WebSocket API](web-socket-api_CN.md#user-data-stream-subscribe) 进行监听时，用户数据流事件负载中新增了字段 `subscriptionId`。这个字段会标识事件来自哪个订阅。

#### FIX API

* 当客户端发送拒绝消息时，FIX API 将不再向客户端返回 `<3>` 拒绝消息。
* 现在，当标签无效、缺少值、字段值为空或格式错误时，错误消息将更加清晰明了。
  * ```json
    { "code": -1169, "msg": "Invalid tag number." }
    ```
  * ```json
    { "code": -1177, "msg": "Tag specified without a value." }
    ```
  * ```json
    { "code": -1102, "msg": "Field value was empty or malformed." }
    ```

#### 将会发生的更改

以下变更将于**2025-08-27 07:00 UTC 开始发生**：
* 交易所信息请求将发送字段 `pegInstructionsAllowed`。
* 已修复的问题：撮合引擎不会再接受数量超出订单数量过滤器上限的订单列表。受到影响的过滤器包括：
  * `MAX_NUM_ORDERS`
  * `MAX_ALGO_ORDERS`
  * `MAX_ICEBERG_ORDERS`
  * `EXCHANGE_MAX_NUM_ORDERS`
  * `EXCHANGE_MAX_ALGO_ORDERS`
  * `EXCHANGE_MAX_ICEBERG_ORDERS`

以下变更将于**2025-08-28 07:00 UTC 开始发生**：
* [挂钩订单](faqs/pegged_orders_CN.md) 功能将可以使用。
  * `pegInstructionsAllowed` 将在所有的交易对上被设为 `true`, 在所有的 API 上正式启用挂钩订单相关功能。
  * 如果订单是挂钩订单，那么以下条件字段 `pegPriceType`， `pegOffSetType`， `pegOffsetValues` 和 `peggedPrice` 将出现在以下请求的响应中：
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
    * `OrdType(40)` 支持新的数值 `P(PEGGED)`
    * 字段 `PegOffsetValue(211)`，`PegPriceType(1094)`，`PegMoveType(835)` 和 `PegOffsetType(836)` 被添加到以下消息中：
      * NewOrderSingle `<D>`
      * NewOrderList `<E>`
      * OrderCancelRequestAndNewOrderSingle `<XCN>`
    * 下单时，`ExecutionReport` `<8>` 消息将回显 PegInstructions，并带有一个额外的可选字段 `PeggedPrice (839)`
  * 添加了新的挂钩订单错误消息。欲知详情，请参阅 [错误代码汇总](errors_CN.md) 文档。
* `recvWindow` 上的变更将被启用。
  * 当您的消息离开我方的消息代理后，在即将发送到撮合引擎之前，将进行第三次检查。
    * 这不包括撮合引擎本身内部的潜在延迟。
  * `recvWindow` 支持微秒级。
    * 该值仍然以毫秒为单位指定，但现在可以采用小数部分来指定更高的精度。
    * 这意味着该参数现在支持**最大精度为 3 位小数**。（例如， 6000.346）
    * 受影响的 API：
      * FIX API
      * REST API
      * WebSocket API
* 新的 [`MAX_NUM_ORDER_LISTS`](filters.md#max-num-order-lists) 过滤器将被启用：会允许每个交易对上最多有 20 个订单列表挂单。
* 新的 [`MAX_NUM_ORDER_AMENDS`](filters_CN.md#max_num_order_amends) 过滤器将被启用：会以 10 次为上限来限制订单修改次数。

---

### 2025-08-07

* 更新了 FIX API 文档
  * [FIX 市场数据限制](fix-api_CN.md#connection-limits)：订阅限制一直存在，但之前未有文档说明。
  * [消息处理顺序](fix-api_CN.md#orderedmode)：重新措辞并调整了格式。

---

### 2025-07-03

* 自 **2025-07-08 07:00 UTC** 起，[WebSocket 账户接口](web-socket-streams_CN.md) 将会进行升级。
* 升级期间，**现有连接和新连接可能会在 24 小时内断开**。
* 升级过程最多可能需要 2 小时。若有不便之处，敬请谅解。

---

### 2025-06-04

REST 和 WebSocket API：

* 注意：根据我们的 SBE 政策，[在被废止 6 个月后](faqs/sbe_faq_CN.md#sbe-schema)， SBE 2：0 模式将于 2025 年 06 月 12 日被禁用。
* [面向生产的 SBE 生命周期](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/sbe_schema_lifecycle_prod.json) 已基于本次更改进行了更新。

---

### 2025-05-28

* 在 “API 基本信息” 下，记录每个 API 的超时值和错误：
  * [FIX](fix-api_CN.md#general-api-information)
  * [REST](rest-api_CN.md#general-api-information)
  * [WebSocket](web-socket-api_CN.md#general-api-information)

---

### 2025-05-22

**注意：以下变更将于 2025 年 6 月 6 日 7:00 (UTC) 生效。**

* 将通过一项额外的检查，对 FIX、REST 和 WebSocket API 上先前关于`recvWindow` 的行为进行强化。
  * 让我们回顾一下现有行为：
    * 如果在接收请求时， `timestamp` 大于 `serverTime` + 1 秒，则请求会被拒绝。此检查被拒绝的话，将会增加消息限制（FIX API）和 IP 限制（REST 和 WebSocket API），但不会增加未成交订单计数（所有 API 的下单端点）。
    * 如果在接收请求时， `timestamp` 和 `serverTime` 之间的差值大于 `recvWindow`，则请求被拒绝。此检查被拒绝的话，将会增加消息限制（FIX API）和 IP 限制（REST 和 WebSocket API），但不会增加未成交订单计数（所有 API 的下单端点）。
  * 附加检查：
    * 在请求转发到撮合引擎之前，如果 `timestamp` 与当前 `serverTime` 之间的差值大于 `recvWindow`，则拒绝该请求。此检查被拒绝的话，将会增加消息限制（FIX API）、IP 限制（REST 和 WebSocket API）以及未成交订单计数（所有 API 的下单端点）。
  * 已更新 Timing 安全性文档，以反映新增的附加检查。
    * [REST API](rest-api_CN.md#timingsecurity)
    * [WebSocket API](web-socket-api_CN.md#timingsecurity)
    * [FIX API](fix-api_CN.md#timingsecurity)
* 修复了 FIX Market Data 消息 InstrumentList `<y>` 中的一个错误。之前，`NoRelatedSym(146)` 的值可能会不正确。

---

### 2025-04-29

* 目前需要 Ed25519 API 密钥才能使用的功能将很快会对 HMAC 和 RSA 密钥开放。
  * 例如，在 listenKeys 被移除前，您可以使用任何 API 密钥类型在 WebSocket API 中订阅账户数据流。
  * 我们仍然鼓励用户迁移到 Ed25519 API 密钥，因为它们在币安现货交易中更安全、性能更佳。
  * 更多详情即将公布，敬请关注，不要错过。

---

### 2025-04-25

* 以下请求权重将从 1 增加到 4：
  * REST API: `PUT /api/v3/order/amend/keepPriority`
  * WebSocket API: `order.amend.keepPriority`
  * REST 和 WebSocket API 的文档均已更新，以反映即将发生的更改。
* 澄清了 FIX-API 中的 `SEQNUM` 是一个累计到最大值后将会归 0，然后重新开始计数的 32 位无符号整数。自 FIX-API 诞生以来，它一直是 `SEQNUM` 数据类型。

---

### 2025-04-21

**关于 [保留优先级的修改订单请求 (Order Amend Keep Priority)](./faqs/order_amend_keep_priority_CN.md) 和 [STP 方式 `DECREMENT`（递减）](./faqs/stp_faq_CN.md) 发布的说明：**
* 于 **2025-05-07 07:00 UTC**
  * 所有交易对将启用 ”保留优先级的修改订单请求“ 功能。
  * 所有交易对将允许 “STP 递减”。
* 自**2025年4月24日 07:00 UTC**起，`amendAllowed` 字段将在交易所信息请求中可见，但该功能尚未启用。
* [SPOT 测试网](https://testnet.binance.vision/) 在所有交易对上都启用了这两项功能。

---

### 2025-04-08

**通知:** 本节中的更改将逐步推出,需要一周时间才能完成.

* 新的错误代码 `-2039`，如果查询同时具有 `orderId` 和 `origClientOrderId` 的订单，并且未找到具有此组合的订单.
  * 受影响的请求:
    * REST API:`GET /api/v3/order`
    * WebSocket API:`order.status`
* [错误代码文档](errors_CN.md) 在错误代码 `-1034` 中更新了新的错误消息, 当超过 FIX 连接速率限制时会出现。(更多细节参看昨天的 [更新](#2025-04-07))

---

### 2025-04-07

#### 常规更改

**通知:** 本节中的更改将逐步推出,需要一周时间才能完成.

* 在2025 年 1 月 16 日，FIX Market Data 会话的连接限制从 5 个增加到了 100 个。这个改变没有在之前的更改日志中被明示。
* 新的错误代码 `-2038`，将在保留优先权的修改订单请求失败时出现。
* 错误代码 `-1034` 有了新消息。
* 如果未成交订单计数超过了在 `intervalNum:DAY` 里定义的限制, 那么 `intervalNum:SECOND` 下的未成交订单计数将不再递增。
* 以前，无论提供的参数如何，myTrades 请求的权重都是 20。现在，如果您提供 `orderId` ，请求的权重将为 5。
  * REST API：`GET /api/v3/myTrades`
  * WebSocket API：`myTrades`
* 查询和删除订单的变化：
  * 现在，当 `orderId` 和 `origClientOrderId` 都不存在时，请求将被拒绝，并显示 `-1102` 而不是 `-1128`。
  * 受影响的请求：
    * REST API：
      * `GET /api/v3/order`
      * `DELETE /api/v3/order`
    * WebSocket API：
      * `order.status`
      * `order.cancel`
    * FIX API：
      * OrderCancelRequest `<F>`

#### FIX API

**通知:** 以下变更将于 2025 年 4 月 21 日期间发生。

* FIX API 会验证 `EncryptMethod(98)` 在登录 Logon `<A>` 时是否为 0。
* FIX 订单接入会话的每个账户有 10 个并发 TCP 连接的限制。
* 现在强制实施连接速率限制.请注意，这些限制是针对账户和 IP 地址独立检查的。
  * FIX 订单接入会话：在 30 秒内 15 次连接尝试的限制
  * FIX Drop Copy 会话：在 30 秒内 15 次连接尝试的限制。
  * FIX Market Data 会话：在 300 秒内 300 次连接尝试的限制。
* News `<B>` 在 Headline 字段中包括了一个倒计时消息。
  * 在本次更新完成后： 当服务器进入维护状态时，将向客户端**每隔 10 秒发送一条** `News` 消息，并**持续 10 分钟**。在10分钟过后，客户端将被注销，其会话将被关闭。
* OrderCancelRequest `<F>` 和 OrderCancelRequestAndNewOrderSingle `<XCN>` 现在允许使用 `orderId` 和 `clientOrderId`。
* [FIX 订单接入会话的 QuickFix 模式](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml) 已被更新，将支持保留优先级的修改订单请求（Order Amend Keep Priority）和新的 STP 方式 `DECREMENT`。

#### User Data Streams

* **我们将弃用此功能： 通过使用 `listenKey` 来访问 wss://stream.binance.com:9443 以监听账户信息的。**
    * 以后但不是当前，此功将被能从我们的系统中删除。
* **您应该通过订阅 [在 WebSocket API 内的账户信息流](https://developers.binance.com/docs/zh-CN/binance-spot-api-docs/websocket-api/user-data-stream-requests) 来获得用户账户更新。**
  * 这个方式会提供稍好的性能 **（较低的延迟）**。
  * 必须使用 Ed25519 API 密钥
* 在未来的更新中，将删除有关账户数据流的 WebSocket 基本访问地址的信息。
* 在未来的更新中，以下请求将被从文档中删除：
    * `POST /api/v3/userDataStream`
    * `PUT /api/v3/userDataStream`
    * `DELETE /api/v3/userDataStream`
    * `userDataStream.start`
    * `userDataStream.ping`
    * `userDataStream.stop`
* [WebSocket 账户接口文档](user-data-stream_CN.md) 将保留可以接收的有效负载，以供您参考。

#### 将会发生的更改

以下变更将于**2025 年 4 月 24 日 07:00 UTC**发生：

* ~~[保留优先级的修改订单请求（Order Amend Keep Priority）](https://github.com/binance/binance-spot-api-docs/blob/master/faqs/order_amend_keep_priority_CN.md) 将可以使用。 (请注意，必须在相应交易对上启用该功能后才能使用。)~~
  * 新字段 `amendAllowed` 会出现在 Exchange Information 响应中。
    * **2025-04-21 更新 ："保留优先级的修改订单请求" 的具体启用日期尚未确定**
    * REST API： `GET /api/v3/exchangeInfo`
    * WebSocket API： `exchangeInfo`
  * FIX API：新的 Order Entry 消息 **OrderAmendKeepPriorityRequest** 和 **OrderAmendReject**
  * REST API：`PUT /api/v3/order/amend/keepPriority`
  * WebSocket API： `order.amend.keepPriority`
* ~~如果已在交易对上作了相应配置，那么 STP 方式 `DECREMENT` （递减） 将在 Exchange Information 中可见。~~
  * **2025-04-21 更新 ："STP 方式 DECREMENT" 的具体启用日期尚未确定。**
  * 通过不仅仅使挂单或吃单过期,或无条件地让两种订单都过期，STP 递减会减少**两种**订单的可用数量，并将通过阻止匹配的数量来增加**两种**订单的 `prevented quantity` 值。
  * 这将使可用数量较少的订单过期，因为(`filled quantity` \+ `prevented quantity`)等于 `order quantity`。如果两个订单的可用数量相等，那么两个订单都将过期。这种情况被称为“递减”，因为可用数量减少了。
* 使用 `orderId` 和 `origClientOrderId/cancelOrigClientOrderId` 来查询和/或取消订单：
  * 以前，当两个参数都提供时，在各个端点的上行为并不一致。
  * 以后，当同时提供两个参数时，系统首先将会使用订单的 `orderId` 来搜索订单。如果订单被找到， `origClientOrderId`/`cancelOrigClientOrderId` 的值将会被用来验证被找到的订单。如果两个检测条件都通过，那么请求成功。如果两个条件都不满足，则请求将被拒绝。
  * 受影响的请求：
    * REST API：
      * `GET /api/v3/order`
      * `DELETE /api/v3/order`
      * `POST /api/v3/order/cancelReplace`
    * WebSocket API：
      * `order.status`
      * `order.cancel`
      * `order.cancelReplace`
    * FIX API：
      * OrderCancelRequest `<F>`
      * OrderCancelRequestAndNewOrderSingle `<XCN>`
* 使用 `listOrderId` 和 `listClientOrderId` 来取消订单:
  * 以前，当两个参数都提供时，在各个端点的上行为并不一致。
  * 以后，当同时提供两个参数时，系统首先将会使用 `listOrderId` 来搜索订单列表。如果找到相应的订单列表，`listClientOrderId` 将会被用来验证被找到的订单列表。如果两个条件都不满足，请求将被拒绝。
  * 受影响的请求
    * REST API
      * `DELETE /api/v3/orderList`
    * WebSocket API
      * `orderList.cancel`
* **SBE： 新模式 3:0 ([spot_3_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_3_0.xml)) 将可以使用。**
  * 现行模式 2:1 ([spot_2_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_1.xml)) 将会被废止并根据我们的废止政策，在6个月内停止使用。
  * 请注意：在新模式被发布前，尝试使用模式 3:0 将导致错误。
  * 3:0 中的更改：
    * 将会支持保留优先级的修改订单（Order Amend Keep Priority）请求:
      * 在 ExchangeInfoResponse 中添加了 `amendAllowed` 字段。
      * 新消息 `OrderAmendmentsResponse` 和 `OrderAmendKeepPriorityResponse`。
    * 所有的枚举类型都有了新变量：`NON_REPRESENTABLE`。这将用于将来，对新的枚举值进行编码。新模式的枚举类型可能会与 3:0 不兼容。
    * 新增针对 `selfTradePreventionMode` 和 `allowedSelfTradePreventionModes` 的新枚举变量 `DECREMENT`。
    * `symbolStatus` 枚举下的值 `AUCTION_MATCH`， `PRE_TRADING` 以及 `POST_TRADING` 已被删除。
    * 字段 `usedSor`, `orderCapacity`， `workingFloor`，`preventedQuantity` 以及 `matchType` 将不再是可选的参数。
    * 更改了 `ExecutionReportEvent`：现在，字段 `orderCreationTime` 是可选的。
  * 通过 WebSocket API 来使用被废止的 2:1 版本的模式来侦听账户数据流：
    * `ListStatusEvent` 的字段 `listStatusType` 在 `Updated` 时，会呈现为 `ExecStarted`。升级到模式 3:0 来获取正确的值。
    * `ExecutionReportEvent` 的字段 `selfTradePreventionMode` 在 `Decrement` 时，会呈现为 `None`。 这将只在 `executionType` 为 `TradePrevention` 时发生。
    * `ExecutionReportEvent` 的字段 `orderCreationTime` 在没有值时将呈现为 -1。
  * 所有低于 3:0 版本的模式不会支持对保留优先级修改订单请求（Order Amend Keep Priority）的响应，以及任何可能包含 STP 方式 `DECREMENT` 的响应（比如，Exchange Information，下单，取消订单或查询您的订单状态）。当响应无法用被指定的模式来表达时，系统将返回错误。

---

### 2025-04-03

关注 SPOT 测试网的最新更新,将 WebSocket API 中的 URL 更新为 [SPOT Testnet] 的最新 URL(https://testnet.binance.vision/).

---

### 2025-03-31

* 添加了对取消订单性能的说明.

---

### 2025-03-10

* **注意：以下变更将于 2025 年 3 月 13 日 09:00 UTC 发生。**
  * FIX Drop Copy 会话的限制将会为**每分钟 60 条消息**。
  * FIX Market Data 会话的限制将会为**每分钟 2000 条消息**。
  * FIX API 文档已更新，以反映即将发生的更改。
* **SBE 市场数据流将于 2025 年 3 月 18 日 07:00 UTC 上线。** 这些流会提供较小的有效负载，将为部分对延迟比较敏感的市场数据连接提供更快的响应时间。
  * 以 SBE 格式提供的数据流：
  * 实时：逐笔交易
  * 实时：最优挂单信息
    * 每 100 毫秒：增量深度信息
  * 每 100 毫秒：有限档深度信息
  * 有关更多信息，请参阅 [SBE 市场数据连流](sbe-market-data-streams_CN.md)。

---

### 2025-03-05

* **注意：以下变更将于 2025 年 3 月 10 日 12:00 UTC 发生。** <br>
  以下请求权重将从 2 增加到 4：
  * REST API： `GET /api/v3/aggTrade`
  * WebSocket API： `trades.aggregate`
* REST 和 WebSocket API 的文档均已更新，以反映即将发生的更改。

---

### 2025-02-12

* **注意：这些更改将于 2025 年 2 月 26 日 05：00 UTC 生效。请确保在此之前您已下载最新的模式。
* `AggressorSide （2446)` 将在 FIX 市场数据流中提供。QuickFIX 模式 [file](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml) 也已更新。

---

### 2024-01-28

**注意: 该变更会在2025年2月3日到2025年2月14日之间逐步推出。**

* **以下变更将会同时应用于WebSocket 市场数据流，WebSocket 账户接口 和 WebSocket API:**
    * WebSocket 服务将会**每20秒**发送发送 PING 消息而不是每3分钟。
    * PONG 消息的延迟将会是每1分钟而不是每10分钟。
    * 相关服务的更改所对应的文档都已经更新。

---

### 2025-01-09

* FIX 市场数据将在 **January 16, 05:00 UTC** 提供。FIX API 文档已更新有关此功能。
* 请参阅此 [链接]（https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml） 以获取 FIX市场数据的 QuickFIX 模式。

---

### 2024-12-17

常规更改：

系统现在在所有相关时间和/或时间戳的字段中支持微秒。微秒支持是 **opt-in**，默认情况下，请求和响应仍然使用毫秒。文档中的示例也在可预见的将来使用毫秒。

WebSocket Streams

* 可以在连接 URL 中使用新的可选参数 `timeUnit` 来选择时间单位。
  * 例如：`/stream？streams=btcusdt@trade&timeUnit=millisecond`
  * 支持的值为：
    * `MILLISECOND`
    * `millisecond`
    * `MICROSECOND`
    * `microsecond`
  * 如果未选择时间单位，则默认使用毫秒。

REST API

* 可以在请求中发送新的可选报文头 `X-MBX-TIME-UNIT` 来选择时间单位。
  * 支持的值：
    * `MILLISECOND`
    * `millisecond`
    * `MICROSECOND`
    * `microsecond`
  * 时间单位会影响 JSON 响应中的时间戳字段（例如，`time`、`transactTime`）。
    * 无论时间单位如何，SBE 响应都将继续以微秒为单位。
  * 如果未选择时间单位，则默认使用毫秒。
* 时间戳参数（例如 'startTime'、'endTime'、'timestamp）' 现在可以以毫秒或微秒为单位传递。

WebSocket API

* 可以在连接 URL 中使用新的可选参数 `timeUnit` 来选择时间单位。
  * 支持的值：
    * `MILLISECOND`
    * `millisecond`
    * `MICROSECOND`
    * `microsecond`
  * 时间单位会影响 JSON 响应中的时间戳字段（例如，`time`、`transactTime`）。
    * 无论时间单位如何，SBE 响应都将继续以微秒为单位。
  * 如果未选择时间单位，则默认使用毫秒。
* 时间戳参数（例如 `startTime`、`endTime`、`timestamp`） 现在可以以毫秒或微秒为单位传递。

User Data Streams

* 可以在连接 URL 中使用新的可选参数 `timeUnit` 来选择时间单位。
  * 支持的值
    * `MILLISECOND`
    * `MICROSECOND`
    * `microsecond`
    * `millisecond`

---

### 2024-12-09

**注意：** 以下的变更会从**2024 年 12 月 12日**开始推出，可能需要大约一周的时间才能完成。

常规更改：

 * 现在会拒绝距离过去或未来太远的时间戳参数值。
  * 时间戳数值在 2017 年 1 月 1 日之前（小于 1483228800000）
  * 时间戳数值超过当前时间 10 秒以后（例如，如果当前时间为 1729745280000, 那么使用 1729745290000 或更大是错误的）
* 如果 `startTime` 和/或 `endTime` 的值超出范围，数值会被调整至正确的范围。
* 已将 `quote order quantity` （`origQuoteOrderQty`） 字段添加到原先没有该字段的响应中。请注意，对于下单相关的接口，该字段将仅针对 `newOrderRespType` 设置为 `RESULT` 或 `FULL` 的请求显示。

请参阅以下列表，以便了解因带有： `origQuoteOrderQty` 而受影响的请求：


| 服务 | 请求 |
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

* 已发布新模式 2:1 [spot_2_1.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_1.xml)。 当前模式 2:0 [spot_2_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_0.xml) 将被弃用。根据我们的模式弃用政策，当前模式 2:0 会将在 6个月内从 API 中停用。
* 模式 2：1 是 模式 2：0 的向后兼容更新版本。当您请求模式 2：0 或 2：1 时，您将始终收到 2：1 格式的有效载荷。
* SBE 模式 2：1 中的更改：
  * 下单/取消订单响应中的新字段 `origQuoteOrderQty` （注意：使用 2：0 模式生成的解码器将忽略此字段）：
    * `NewOrderResultResponse`
    * `NewOrderFullResponse`
    * `CancelOrderResponse`
    * `NewOrderListResultResponse`
    * `NewOrderListFullResponse`
    * `CancelOrderListResponse`
  * 仅限 WebSocket API：会话状态响应中的新字段 `userDataStream`：
    * `WebSocketSessionLogonResponse`
    * `WebSocketSessionStatusResponse`
    * `WebSocketSessionLogoutResponse`
  * 仅限 WebSocket API：在 User Data Stream 中会支持的新消息：
    * `UserDataStreamSubscribeResponse`
    * `UserDataStreamUnsubscribeResponse`
    * `BalanceUpdateEvent`
    * `EventStreamTerminatedEvent`
    * `ExecutionReportEvent`
    * `ExternalLockUpdateEvent`
    * `ListStatusEvent`
    * `OutboundAccountPositionEvent`

WebSocket API

* 您现在可以通过 WebSocket API 连接订阅账户数据流事件。
  * 请注意：此功能仅适用于使用 Ed25519 API 密钥的用户。
  * 请注意：如果您要订阅使用 SBE 格式的账户数据流，则需使用新的 SBE 模式 2:1。
* 新请求：
  * `userDataStream.subscribe`
  * `userDataStream.unsubscribe`
* 对于 `session.logon`、 `session.status` 和 `session.logout` 的更改。
  * 添加了一个新字段 `userDataStream`，用于显示账户数据流订阅是否处于活跃状态。
* 修复了在 `session.logon` 之后使用 `userDataStream.start` 不会收到新 listenKey 的错误。

User Data Stream

* 仅限于 WebSocket API：当您从 websocket 会话注销或取消订阅账户数据流时，新事件 `eventStreamTerminated` 将会被发出。
* 当您的现货钱包余额被外部系统锁定/解锁时，新事件 `externalLockUpdate` 将会被发送。

FIX API

* [模式](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml) 中已添加了新的管理消息 News \<B\>，该消息可用于所有 FIX 服务。收到此消息意味着您的连接即将关闭。

以下更改将在**2024 年 12 月 16 日到 2024 年 12 月 20日之间**发生：

* 修复一个错误： `BUY` 方 OCO 单如果不提供 `stopPrice` 就会被阻止下单。
* 在 OCO 中添加了对 `TAKE_PROFIT` 和 `TAKE_PROFIT_LIMIT` 的支持。
  * 以前，OCO 只能由以下订单类型组成：
    * `LIMIT_MAKER` + '`STOP_LOSS`
    * `LIMIT_MAKER` + `STOP_LOSS_LIMIT`
  * 现在，OCO 可以由以下订单类型组成：
    * `LIMIT_MAKER` + `STOP_LOSS`
    * `LIMIT_MAKER` + `STOP_LOSS_LIMIT`
    * `TAKE_PROFIT` + `STOP_LOSS`
    * `TAKE_PROFIT` + `STOP_LOSS_LIMIT`
    * `TAKE_PROFIT_LIMIT` + `STOP_LOSS`
    * `TAKE_PROFIT_LIMIT` + `STOP_LOSS_LIMIT`
  * 以下请求支持此功能：
    * `POST /api/v3/orderList/oco`
    * `POST /api/v3/orderList/otoco`
    * `orderList.place.oco`
    * `orderList.place.otoco`
    * `NewOrderList<E>`
  * 错误代码 -1167 将在此次更新后过时，并将在以后的更新中从文档中删除。

---

### 2024-10-18

Rest 和 WebSocket API:

* 注意：根据我们的 SBE 政策，SBE 1：0 模式将于 2024 年 10 月 25 日被禁用 [废止后 6 个月](./faqs/sbe_faq_CN.md)。
* [面向生产的 SBE 生命周期](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/sbe_schema_lifecycle_prod.json) 已基于本次更改进行了更新。

---

### 2024-10-17

Exchange Information 的更改 （即 REST API 的 [`GET /api/v3/exchangeInfo`](rest-api_CN.md#exchangeInfo) 和 WebSocket API 的 [`exchangeInfo`](web-socket-api_CN.md#exchangeInfo)）。

* 新的可选参数 `showPermissionSets` 可用于隐藏 `permissionsSets` 的权限信息;这可用于减小消息大小。
* 新的可选参数 `symbolStatus` 用于仅显示具有指定状态的交易对。（例如：`TRADING`， `HALT`， `BREAK`）


---

### 2024-08-26

* [现货未成交订单计数规则](./faqs/order_count_decrement_CN.md)  已更新，解释了如何在下订单时减少未成交的订单数量。

---
### 2024-08-16

**注意：** 以下的变更正在逐步推出，可能需要大约一周的时间才能完成。

常规更改：
* 在市场流动性低的情况下，当提交包含 `quoteOrderQty` 的市价单（又名反向市价单）被拒绝时，添加了新的错误消息。


---

### 2024-08-01

* [FIX API 和 Drop Copy 会话](fix-api_CN.md) 将于 **8 月 8 日 05:00 UTC** 上线。

---

### 2024-07-26

* [FIX API 和 Drop Copy 会话](fix-api_CN.md) 已添加到文档中。
* 实时交换的发布日期尚未确定。

---

### 2024-07-22

常规更改：

* 修复了 klines 的时间戳不正确的 bug。
  * REST API： 带有 `timeZone` 参数的 `GET /api/v3/klines` 和 `GET /api/v3/uiKlines`
  * WebSocket API： 带有 `timeZone` 参数的 `klines` 和 `uiKlines`
  * WebSocket Streams: `<symbol>@kline_<interval>@+08：00`

---

### 2024-06-11

* 在 **6月11日 UTC 时间 05:00**，我们将开始推出新功能 `One-Triggers-the-Other` (OTO) 订单和 `One-Triggers-a-One-Cancels-The-Other` (OTOCO) 订单。（请注意，我们可能需要花几个小时来部署到所有服务器。）
    * 有关详细信息，请参阅以下页面：
        * REST API：
            * `POST /api/v3/orderList/oto`
            * `POST /api/v3/orderList/otoco`
        * WebSocket API：
            * `orderList.place.oto`
            * `orderList.place.otoco`
* 在 **6月18日 UTC 时间 05:00**，我们将会把买方的订单 ID（`b`） 和卖方的订单 ID（`a`） 从交易流中删除（i.e. `<symbol>@trade`）。 （请注意，我们可能需要花几个小时来部署到所有服务器。）
    * [WebSocket 账户接口](web-socket-streams_CN.md) 与其相关的文档已经被更改了。
    * 要监控您的订单是否是交易的一部分，请订阅 [WebSocket 账户接口](user-data-stream_CN.md)。

---

### 2024-06-06

此功能将在**6月6日 UTC时间11:59**前上线。

REST API

* `POST /api/v3/order/cancelReplace` 新加可选参数 `orderRateLimitExceededMode`。

WebSocket API

* `order.cancelReplace` 新加可选参数 `orderRateLimitExceededMode`。

---

### 2024-05-30

WebSocket Streams

* Kline 新增加了对 UTC+8 时区的支持。（例如，`btcusdt@kline_1d@+08:00`）

---

### 2024-04-10

以下更新的生效时间已被推迟到 **4月25日 05：00 UTC**

* "交易规范信息"响应中的交易对权限信息已从字段 `permissions` 移至字段 `permissionSets`。
* 字段 `permissions` 将为空，并将在未来版本中删除。
* 以前，`"permissions":["SPOT","MARGIN"]` 代表如果您的账户具有 `SPOT` 或 `MARGIN` 权限，您就可以在该交易对上下订单。现在，等效项是 `"permissionSets":[["SPOT","MARGIN"]]`（请注意额外的方括号）。`permissionSets`数组中的每个权限数组称为 "permission set"。
* 交易对的权限现在可以有更多权限类型。例如，`"permissionSets":[["SPOT","MARGIN"],["TRD_GRP_004","TRD_GRP_005"]]` 指示除了支持以上提过的权限集，也接受 `TRD_GRP_004` 或 `TRD_GRP_005`。交易对的 `permissionSets` 中可以有任意排列组合的权限集。

REST API

* `otoAllowed` 现在将出现在 `GET /api/v3/exchangeInfo` 上，指示该交易品种是否支持 One-Triggers-the-Other (OTO) 订单。

WebSocket API

* `otoAllowed` 现在将出现在 `exchangeInfo` 上，指示该交易品种是否支持 One-Triggers-the-Other (OTO) 订单。

SBE

* 已发布新模式 2:0 [Spot_2_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_0.xml)。 当前模式 1:0 [spot_1_0.xml](https://github.com/binance/binance-spot-api-docs/blob/becd4d44a09d94821d2dc761ba9197aae8b495c3/sbe/schemas/spot_1_0.xml) 将被弃用，并从根据我们模式弃用政策，会将在 6 个月内下线。
* 在 REST API 或 WebSocket API 上使用模式 1:0 时，消息 `ExchangeInfoResponse` 中的组"权限"将始终为空。在升级到模式 2:0后， 您才可以在 `permissionSets` 组中查找权限信息。
* 最新模式仍将支持已弃用的 OCO 请求。
* 请注意，在模式 2:0 实际发布之前尝试使用它会导致错误。

---

### 2024-04-02

**注意：** 以下的变更将逐步推出，并预计需要大约一周的时间完成。

* `GET /api/v3/account` 新加可选参数 `omitZeroBalances`，如果启用，则会隐藏所有零余额。
* `account.status` 新加可选参数 `omitZeroBalances`，如果启用，则会隐藏所有零余额。
* **以下请求的权重已从 10 增加到 25 （该规定将于2024年4月4日生效）**：
    * `GET /api/v3/trades`
    * `GET /api/v3/historicalTrades`
    * `trades.recent`
    * `trades.historical`

User Data Stream

* 如果 `listenKey` 过期，将在流中发出新事件 `listenKeyExpired`。

REST API

* REST API 上现已弃用 `POST /api/v3/order/oco` 接口。从今开始，请使用新的 `POST /api/v3/orderList/oco` 接口（请注意，新接口使用不同的参数）。

WebSocket API

* WebSocket API 上现已弃用 `orderList.place` 请求。从今开始，请使用新的 `orderList.place.oco` 请求（请注意，新接口使用不同的参数）。


**以下内容将于发布日期 _大约_ 一周后生效：**

* "交易规范信息"响应中的交易对权限信息已从字段 `permissions` 移至字段 `permissionSets`。
* 字段 `permissions` 将为空，并将在未来版本中删除。
* 以前，`"permissions":["SPOT","MARGIN"]` 代表如果您的账户具有 `SPOT` 或 `MARGIN` 权限，您就可以在该交易对上下订单。现在，等效项是 `"permissionSets":[["SPOT","MARGIN"]]`（请注意额外的方括号）。`permissionSets`数组中的每个权限数组称为 "permission set"。
* 交易对的权限现在可以有更多权限类型。例如，`"permissionSets":[["SPOT","MARGIN"],["TRD_GRP_004","TRD_GRP_005"]]` 指示除了支持以上提过的权限集，也接受 `TRD_GRP_004` 或 `TRD_GRP_005`。交易对的 `permissionSets` 中可以有任意排列组合的权限集。

REST API

* `otoAllowed` 现在将出现在 `GET /api/v3/exchangeInfo` 上，指示该交易品种是否支持 One-Triggers-the-Other (OTO) 订单。

WebSocket API

* `otoAllowed` 现在将出现在 `exchangeInfo` 上，指示该交易品种是否支持 One-Triggers-the-Other (OTO) 订单。


SBE

* 已发布新模式 2:0 [Spot_2_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_0.xml)。 当前模式 1:0 [spot_1_0.xml](https://github.com/binance/binance-spot-api-docs/blob/becd4d44a09d94821d2dc761ba9197aae8b495c3/sbe/schemas/spot_1_0.xml) 将被弃用，并从根据我们模式弃用政策，会将在 6 个月内下线。
* 在 REST API 或 WebSocket API 上使用模式 1:0 时，消息 `ExchangeInfoResponse` 中的组"权限"将始终为空。在升级到模式 2:0后， 您才可以在 `permissionSets` 组中查找权限信息。
* 最新模式仍将支持已弃用的 OCO 请求。
* 请注意，在模式 2:0 实际发布之前尝试使用它会导致错误。

### 2024-02-28

**将于 2024 年 3 月 5 日生效。**

简单二进制编码 (SBE) 将部署到现货的 Rest API 和  WebSocket API 生产系统上。

更多关于SBE的信息, 请参考[常见问题解答 (FAQ)](./faqs/sbe_faq_CN.md)

---

### 2024-02-08

现货的 WebSocket API 现在在[测试网](https://testnet.binance.vision)上支持简单二进制编码(SBE)。

SBE 模式已经更新了 WebSocket API 元数据，但并没有增加 `schemaId` 或者 `version`。

* 仅在 REST API 上使用 SBE 的用户可以继续使用 SBE 模式 [`128b94b2591944a536ae427626b795000100cf1d`](https://github.com/binance/binance-spot-api-docs/blob/128b94b2591944a536ae427626b795000100cf1d/sbe/schemas/spot_1_0.xml)，或者更新到新提交的 SBE 模式。

* 希望在 WebSocket API 上使用 SBE 的用户，需要更新到[最新的 SBE 模式](https://github.com/binance/binance-spot-api-docs/blob/becd4d44a09d94821d2dc761ba9197aae8b495c3/sbe/schemas/spot_1_0.xml)。

SBE 的 [FAQ](./faqs/sbe_faq_CN.md) 已经更新。

---

### 2023-12-08

简单二进制编码 (SBE) 已经在[现货测试网](https://testnet.binance.vision)上线。
生产系统会在随后支持。
更多关于SBE的信息, 请参考[常见问题解答 (FAQ)](./faqs/sbe_faq_CN.md)

---

### 2023-12-04

**注意**： 以下的变更将逐步推出，并预计需要大约一周的时间完成。

* 错误消息 `Precision is over the maximum defined for this asset.` 被改为 `Parameter '%s' has too much precision.`
    * 当参数的精度超出允许范围时，将返回此错误消息。例如，如果基础资产（`base asset`）精度为6，但是设置`quantity=0.1234567`，则会出现此错误消息。
    * 这会影响所有具有以下参数的请求:
        * `quantity`
        * `quoteOrderQty`
        * `icebergQty`
        * `limitIcebergQty`
        * `stopIcebergQty`
        * `price`
        * `stopPrice`
        * `stopLimitPrice`
* 现在，请求查询OCO开单时会正确返回**升序**的结果。这会影响以下请求：
    * REST API: `GET /api/v3/openOrderList`
    * WebSocket API: `openOrderList.status`
* 现在，当指定`startTime`或`fromId`时，请求查询所有 OCO 订单会正确返回**升序**的结果。这会影响以下请求：
    * REST API: `GET /api/v3/allOrderList`
    * WebSocket API: `allOrderLists`
* 修复了一个错误。订单查询请求不再会对新下的订单错误返回[`-2026 ORDER_ARCHIVED`](./errors.md#-2026-order_archived)错误。
    * REST API: `GET /api/v3/order`
    * WebSocket API: `order.status`


REST API

* 新接口 `GET /api/v3/account/commission`
* 新接口 `GET /api/v3/ticker/tradingDay`
* `GET /api/v3/avgPrice` 新加字段 `closeTime`, 用于显示最后交易时间。
* `GET /api/v3/klines` 和 `/api/v3/uiKlines` 新加可选参数 `timeZone`。
* `POST /api/v3/order/test` 和 `POST /api/v3/sor/order/test` 新加可选参数 `computeCommissionRates`。
* 关于发送无效接口的变动：
    * 以前，如果查询一个不存在的端点（例如 `curl -X GET "https://api.binance.com/api/v3/exchangie"`），你会收到 HTTP 404 状态码，以及响应 "`<html><body><h2>404 Not found</h2></body></html>`"。
    * 从现在开始，只有当接受请求头中包含`text/html`时，HTML响应才会出现在这种情况下。HTTP状态码将保持不变。

WebSocket API

* 新请求 `account.commission`
* 新增请求以允许会话身份验证: **(请注意，这些请求只能使用Ed25519密钥。)**
    * `session.logon`
    * `session.logout`
    * `session.status`
* 新请求 `ticker.tradingDay`
* 方法 `avgPrice` 新加字段 `closeTime`, 用于显示最后交易时间。
* 方法 `klines` 和 `uiKlines` 新加可选参数 `timeZone`。
* 方法 `order.test` 和 `sor.order.test` 新加可选参数 `computeCommissionRates`.
* 修复了一个错误。之前在发送 ping 之前未经请求发送的 pongs 会导致断开连接。

WebSocket Streams

* 新数据流 `<symbol>@avgPrice`
* 请求中的`id`现在支持和 WebSocket API 里`id`一样的值:
    * 64位有符号整数 (之前是无符号整数)
    * 字母数字字符串；最大长度36
    * `null`
* 修复了一个错误，之前在发送 ping 之前未经请求发送的 pongs 会导致断开连接。

User Data Streams

* 当事件类型为 `executionReport`，而执行类型（x）为`TRADE_PREVENTION`时，字段`l`、`L`和`Y`现在将始终为0。新增字段`pl`、`pL`和`pY`将描述被阻止执行的数量、被阻止执行的价格和被阻止执行的名义金额。这些新字段显示了如果接收方订单没有启用自成交防止功能时，`l`、`L`和`Y`会是什么值。


**以下将在发布日期后_大约_一周后生效:**

* 交易对权限将仅影响下单，而不影响取消订单。
    * `permissions`仍然适用于撤消挂单再下单（Cancel-Replace orders）（比如，如果您的账户有使用此请求下单的权限，则将不允许取消操作）。


---

### 2023-10-19

**从 2023-10-19 00:00 UTC 开始生效**

* 调高如下接口的请求权重:

<table>
    <tr>
        <th>REST API</th>
        <th>WebSocket API</th>
        <th>条件 </th>
        <th>之前的权重</th>
        <th>调整后权重</th>
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

* **下单量的退回(`Order decrement`)功能在 06:15 UTC上线.**
* 此功能的更详细信息, 请参考 [FAQ](./faqs/order_count_decrement_CN.md)

---

### 2023-08-25

* Websocket API 的 `exchangeInfo` 中的 `RAW REQUESTS` 被移除，新增了用于表示WebSocket连接数限制的 `CONNECTIONS`。

**下面的变更会在UTC时间 2023-08-25 00:00 上线**
* WebSocket API 的 `CONNECTIONS` 被调整为每5分钟300。
* REST API 和 WebSocket API 的 `REQUEST_WEIGHT` 调整为每分钟6,000。
* REST API 中的 `RAW_REQUESTS` 调整为每5分钟61,000。
* 之前连接到 WebSocket API 的权重为1。**现权重调整到 2**。
* 下表的 REST API 和 WebSocket API 请求的权重被调整:

|请求接口|之前请求权重| 新请求权重|
|----   |----                   | ----                   |
|`GET /api/v3/order` <br> `order.status` |2 | 4|
|`GET /api/v3/orderList` <br> orderList.status| 2|4|
|`GET /api/v3/openOrders` <br> `openOrders.status` - **带 `symbol`**|3|6|
|`GET /api/v3/openOrders` <br> `openOrders.status` - **不带 `symbol`**|40|80|
|`GET /api/v3/openOrderList` <br>`openOrderLists.status`|3|6|
|`GET /api/v3/allOrders` <br>`allOrders`  |10|20
|`GET /api/v3/allOrderList` <br> `allOrderLists` |10|20
|`GET /api/v3/myTrades`  <br> `myTrades`|10|20|
|`GET /api/v3/myAllocations`  <br> `myAllocations` |10|20|
|`GET /api/v3/myPreventedMatches`  <br> `myPreventedMatches`  - **使用 `preventedMatchId`** | 1 | 2
|`GET /api/v3/myPreventedMatches`  <br> `myPreventedMatches`  - **使用 `orderId`**|10|20|
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
|`GET /api/v3/ticker/bookTicker` <br> `ticker.book` - **带 `symbol`**|1|2|
|`GET /api/v3/ticker/bookTicker` <br> `ticker.book` - **不带 `symbol`** 或者 **带 `symbols`**|2|4|
|`GET /api/v3/ticker/price`<br> `ticker.price` - **带 `symbol`**|1|2|
|`GET /api/v3/ticker/price`<br> `ticker.price` - **不带 `symbol`** 或者 **带 `symbols`**|2|4|
|`GET /api/v3/ticker/24hr` <br> `ticker.24hr` - **带 `symbol`** 或者 ** `symbols` 带 1-20 交易对** |1|2|
|`GET /api/v3/ticker/24hr` <br> `ticker.24hr` - **带 `symbols` 21-100 交易对**|20|40|
|`GET /api/v3/ticker/24hr` <br> `ticker.24hr` - **不带 `symbol` 或者 `symbols` 带 101 个或者更多交易对**|40|80|
|`GET /api/v3/avgPrice` <br>`avgPrice`|1|2|
|`GET /api/v3/ticker` <br> `ticker`|2|4|
|`GET /api/v3/ticker` <br> `ticker` - 请求的最大权重|100|200|
|`POST /api/v3/userDataStream` <br> `userDataStream.start`|1|2|
|`PUT /api/v3/userDataStream` <br> `userDataStream.ping`|1|2|
|`DELETE /api/v3/userDataStream`<br> `userDataStream.stop`|1|2|

---

### 2023-08-08

智能订单路由(Smart Order Routing：SOR）添加到 API 中。您可以在[SOR 常见问题](./faqs/sor_faq_CN.md) 文档中找到更多详细信息。具体上线时间请关注相关公告。

REST API

* `GET /api/v3/exchangeInfo` 变动：
    * 返回数据中添加新字段: `sors`, 用于描述交易中是否使用了 SOR。
* `GET /api/v3/myPreventedMatches` 变动：
    * 对于所有的 Prevented Matches， 返回数据中添加新字段 `makerSymbol` 。
* 为了在下单时使用 SOR 而添加的新接口：
    * `POST /api/v3/sor/order`
    * `POST /api/v3/sor/order/test`
* 添加新接口: `GET /api/v3/myAllocations`

WEBSOCKET API

* `exchangeInfo` 变动：
    *  返回数据中添加新字段: `sors` , 用于描述交易中是否使用了 SOR。
* `myPreventedMatches` 变动：
    * 对于所有的 Prevented Matches， 返回数据中添加新字段 `makerSymbol`。
* 为了在下单时使用 SOR 而添加的新请求：
    * `sor.order.place`
    * `sor.order.test`
* 添加新请求 `myAllocations`

USER DATA STREAM

* `executionReport` 变动：
    * 以下这些字段只适用于下单时使用 SOR 的情况：
        * 新字段 `b` 代表 `matchType`
        * 新字段 `a` 代表 `allocId`
        * 新字段 `k` 代表 `workingFloor`
    * 这个字段只适用于订单因为触发 STP 而将过期的情况：
        * 新字段 `Cs` 代表 `counterSymbol`

---

### 2023-07-18

* 现在支持使用 Ed25519 类型的 API key。(UI 会在本周发布更新支持)
    * Ed25519 API keys 是 RSA API keys 的替代品，使用非对称加密技术来验证您的 API 请求。
    * **我们建议切换到 Ed25519** 以提高性能和安全性。 <br>
        详情请参考[API Key 类型](./faqs/api_key_types_CN.md)。
* 文档已更新，包括了有关如何使用 Ed25519 key 对有效载荷进行签名的说明。

---

### 2023-07-11

**注意:** 所有更改都将逐步推出，可能需要一周时间才能完成。

* 错误消息的变动:
    * 之前当发送重复交易对时，会返回错误信息: "Mandatory parameter symbols was not sent, was empty/null, or malformed."
    * 现在则返回消息: "Symbol is present multiple times in the list", with a new error code `-1151`
    * 受影响的接口:
        * `GET /api/v3/exchangeInfo`
        * `GET /api/v3/ticker/24hr`
        * `GET /api/v3/ticker/price`
        * `GET/api/v3/ticker/bookTicker`
        * `exchangeInfo`
        * `ticker.24hr`
        * `ticker.price`
        * `ticker.book`
* 修复一个bug，当查询没有被存档的订单时候，可能返回错误消息称已经被存档。

Rest API

* `GET /api/v3/account` 变动：
    * 返回数据中添加新字段 `preventSor`。
    * 返回数据中添加用户ID的新字段 `uid`。
* `GET /api/v3/historicalTrades` 变动：
    * 鉴权类型从 `MARKET_DATA` 变更为 `NONE`。
    * 不需要设置 `X-MBX-APIKEY` 到请求的header中。

Websocket API

* `account.status` 变动：
    * 返回数据中添加新字段 `preventSor`。
    * 返回数据中添加用户ID的新字段 `uid`。
* `trades.historical` 变动：
    * 鉴权类型从 `MARKET_DATA` 变更为 `NONE`。
    * 请求中不需要设置 `apiKey`。

* 修改了几个bugs: 当下单时设置 `type=MARKET` 和 `quoteOrderQty`, 也被称为"反向市价单":
    * 当处于极端市场情况下, 订单不会返回部分成交，或者成交的数量为0甚至是负数。
    * 当这种反向市价单的成交数量超过交易对的 `maxQty`, 订单会因为违反`MARKET_LOT_SIZE` 过滤器而被拒绝。
* 修复一个OCO订单的bug: 当使用 `trailingDelta` 时候, 当任何leg被触发时, `trailingTime` 值可能不正确。
* 这些接口的返回数据中添加新字段 `transactTime` :
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

* 为了提供系统的冗余能力，新加一个API接入网址: **https://api-gcp.binance.com/**
    * 此网址利用了 GCP (Google Cloud Platform) 的CDN，可能在性能上比`api1`-`api4`要慢。

---

### 2023-05-26

**注意:** 所有更改都将逐步推出到我们的所有服务器，并可能需要一周时间才能完成。
* 以下基本接口可能会提供比 **https://api.binance.com** 更好的性能但其稳定性略为逊色:
  * **https://api1.binance.com**
  * **https://api2.binance.com**
  * **https://api3.binance.com**
  * **https://api4.binance.com**

---

### 2023-05-24

* **以前的市场数据 URL 已不建议使用。请立即更新您的代码，以防止来自我们的服务被中断**
    * 来自 `data.binance.com` 的 API 市场数据现在可以从 `data-api.binance.vision` 访问。
    * 来自 `data-stream.binance.com` 的 Websocket 市场数据现在可以从 `data-stream.binance.vision` 访问。

---

### 2023-03-13

**注意:** 所有更改都将逐步推出到我们的所有服务器，并可能需要一周时间才能完成。

* 某些问题的错误消息已经改进，以便更轻松地进行解决。

<table>
    <tr>
        <th>情况</th>
        <th>之前的错误消息</th>
        <th>新错误消息</th>
    </tr>
    <tr>
        <td>由于交易权限被禁用，账户无法下订单或取消订单。</td>
        <td rowspan="3">This action is disabled on this account.</td>
        <td>This account may not place or cancel orders.</td>
    </tr>
    <tr>
        <td>当配置在交易对上的权限与账户上的权限不匹配时。</td>
        <td>This symbol is not permitted for this account.</td>
    </tr>
    <tr>
        <td>当账户在其没有权限的交易对上下订单时。</td>
        <td>This symbol is restricted for this account.</td>
    </tr>
    <tr>
        <td>当 <tt>symbol</tt> 不在 <tt>TRADING</tt> 时下订单。</td>
        <td rowspan="2">Unsupported order combination.</td>
        <td>This order type is not possible in this trading phase.</td>
    </tr>
    <tr>
        <td>在不支持 <tt>IOC</tt> 或 <tt>FOK</tt> 的交易阶段上使用 <tt>timeinForce</tt> = <tt>IOC</tt> 或 <tt>FOK</tt> 下订单时。</td>
        <td>Limit orders require GTC for this phase.</td>
    </tr>
</table>

* 更正了查询归档订单的错误消息：
    * 之前，如果查询了一个归档订单（即状态为 `CANCELED` 或 `EXPIRED`，`executedQty` == 0 而且最后的更新在 90 天以前），错误消息将是：
    ```json
    {
        "code": -2013,
        "msg": "Order does not exist."
    }
    ```
    * 现在，错误消息为：
    ```json
    {
        "code": -2026,
        "msg": "Order was canceled or expired with no executed qty over 90 days ago and has been archived."
    }
    ```
* API 请求使用 `startTime` 和 `endTime` 的行为：
    * 之前，如果 `startTime` == `endTime`，一些请求会失败。
    * 现在，所有接受 `startTime` 和 `endTime` 的 API 请求会允许这些参数相等。这适用于以下接口：
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

* 如果用户的IP地址因违反 IP 速率限制（状态码为 `418`）而被禁止，那么连接到 WebSocket API 的用户将被断开连接。

虽然以下更改将在发布日期后 **大约一周内生效**，但是与其相关的文档已经被更改了：

* 过滤器评估的更改：
    * 之前的行为: `LOT_SIZE` 和 `MARKET_LOT_SIZE` 要求 (`quantity` - `minQty`) % `stepSize` == 0。
    * 新行为: 现在已更改为 (`quantity` % `stepSize`) == 0。
* 使用 `quoteOrderQty` 的 `MARKET`订单的错误修复：
    * 之前的行为: 订单的状态将始终为 `FILLED`，即使订单没有完全成交。
    * 新行为: 如果订单由于流动性不足而没有完全成交，则订单状态将为 `EXPIRED`，仅当订单完全成交时状态为 `FILLED`。

REST API

* `DELETE /api/v3/order` 和 `POST /api/v3/order/cancelReplace` 的更改:
    * 新的可选参数 `cancelRestrictions`，该参数用于决定是否能成功取消状态为 `NEW` 或 `PARTIALLY_FILLED` 的订单。
    * 如果由于 `cancelRestrictions` 而取消订单失败，错误将是：
    ```json
    {
        "code": -2011,
        "msg": "Order was not canceled due to cancel restrictions."
    }
    ```

WEBSOCKET API

* `order.cancel` 和 `order.cancelReplace` 的更改:
    * 新的可选参数 `cancelRestrictions`，该参数用于决定是否能成功取消状态为 `NEW` 或 `PARTIALLY_FILLED` 的订单。
    * 如果由于 `cancelRestrictions` 而取消订单失败，错误将是：
    ```json
    {
        "code": -2011,
        "msg": "Order was not canceled due to cancel restrictions."
    }
    ```

---

### 2023-02-17

**WebSocket频率限制变动**

`WS-API` 和 `Websocket Stream` 现在限制每个IP地址、每5分钟可以发送连接请求的上限是300次。


---

### 2023-01-26

根据此[公告](https://www.binance.com/zh-CN/support/announcement/%E5%B9%A3%E5%AE%89%E7%8F%BE%E8%B2%A8%E6%8E%A8%E5%87%BAapi%E8%87%AA%E6%88%90%E4%BA%A4%E9%A0%90%E9%98%B2-stp-%E5%8A%9F%E8%83%BD-312fd0112fb44635b397c116e56d8f84)，Self-Trade Prevention 将在 **2023-01-26 08:00 UTC** 发布。

---

### 2023-01-23

* 添加了新的 API 集群 https://api4.binance.com

---

### 2023-01-19

实际发布日期待定

**新功能**：Self-Trade Prevention（STP）会添加到系统中。此功能将阻止订单与来自同一账户或者同一 `tradeGroupId` 账户的订单交易。

请使用现货 REST API 的 `GET /api/v3/exchangeInfo` 或 Websocket API 的 `exchangeInfo` 看 STP 的状态。

```javascript
{
    "defaultSelfTradePreventionMode": "NONE",     // selfTradePreventionMode 的默认值
    "allowedSelfTradePreventionModes": [          // selfTradePrevention 的可用模式
        "NONE",
        "EXPIRE_TAKER",
        "EXPIRE_BOTH",
        "EXPIRE_MAKER"
    ]
}
```

在[STP 常见问题](./faqs/stp_faq_CN.md) 文档中可以找到更多其它关于 STP 功能的详细信息。

REST API

* 新的订单状态：`EXPIRED_IN_MATCH` - 订单由于 STP 触发而过期。
* 新的接口：
   * `GET /api/v3/myPreventedMatches` - 获取由于 STP 触发而过期的订单。
* 新的可选参数 `selfTradePreventionMode` 已添加到以下的接口：
    * `POST /api/v3/order`
    * `POST /api/v3/order/oco`
    * `POST /api/v3/cancelReplace`
* 如果有预防自我交易(Prevented Match)，所有下单相关的接口会出现新字段：
    * `tradeGroupId`      - 仅当账户配置为 `tradeGroupId` 时才会出现。
    * `preventedQuantity` - 被防止交易的订单数量。
    * `preventedMatches` 数组会有以下的字段：
        * `preventedMatchId`
        * `makerOrderId`
        * `price`
        * `takerPreventedQuantity` - 仅当 `selfTradePreventionMode` 设置为 `EXPIRE_TAKER` 或 `EXPIRE_BOTH` 时才会出现。
        * `makerPreventedQuantity` - 仅当 `selfTradePreventionMode` 设置为 `EXPIRE_MAKER` 或 `EXPIRE_BOTH` 时才会出现。
* 如果订单因 STP 触发而过期，以下查询订单接口的响应中可以出现新的字段 `preventedMatchId` 和 `preventedQuantity`：
    * `GET /api/v3/order`
    * `GET /api/v3/openOrders`
    * `GET /api/v3/allOrders`

WEBSOCKET API

* 新的订单状态：`EXPIRED_IN_MATCH` - 订单由于 STP 触发而过期。
* 新的请求：`myPreventedMatches` - 获取由于 STP 触发而过期的订单。
* 新的可选参数 `selfTradePreventionMode` 已添加到以下的请求：
    * `order.place`
    * `orderList.place`
    * `order.cancelReplace`
* 如果有防止自我交易，将为所有下订单的请求会出现的新响应：
    * `tradeGroupId`      - 仅当账户配置为 `tradeGroupId` 时才会出现。
    * `preventedQuantity` - 被防止交易的订单数量。
    * `preventedMatches` 数组会有以下的字段：
        * `preventedMatchId`
        * `makerOrderId`
        * `price`
        * `takerPreventedQuantity` - 仅当 `selfTradePreventionMode` 设置为 `EXPIRE_TAKER` 或 `EXPIRE_BOTH` 时才会出现。
        * `makerPreventedQuantity` - 仅当 `selfTradePreventionMode` 设置为 `EXPIRE_MAKER` 或 `EXPIRE_BOTH` 时才会出现。
* 如果订单因 STP 触发而过期，以下查询订单接口的响应中可以出现新的字段 `preventedMatchId` 和 `preventedQuantity`：
    * `order.status`
    * `openOrders.status`
    * `allOrders`


USER DATA STREAM

* 新的执行类型：`TRADE_PREVENTION`。
* `executionReport` 的新字段（这些字段只会在订单因 STP 触发而过期时出现）：
    * `u` - `tradeGroupId`
    * `v` - `preventedMatchId`
    * `U` - `counterOrderId`
    * `A` - `preventedQuantity`
    * `B` - `lastPreventedQuantity`

---

### 2022-12-28

* 现货 WebSocket API 文档已更新，添加了如何使用 RSA key 签署请求。

---

### 2022-12-26

* 现货的 Websocket API 发布到生产系统中。
* 现货的 Websocket API 可以通过URL: `wss://ws-api.binance.com/ws-api/v3` 来访问。

---

### 2022-12-15

* 添加新的RSA签名验证方式
    * 文档已更新以显示如何创建 RSA keys。
    * 建议在生成 API key 时使用 RSA keys。
    * 我们接受`PKCS#8`（BEGIN PUBLIC KEY）。
    * 稍后将添加有关如何上传 RSA public key 的更多详细信息。
* 现货 WebSocket API 现在可以在 SPOT 测试网上使用。
    * WebSocket API 允许通过 WebSocket 连接下订单、取消订单等。
    * WebSocket API 是一个 **独立** 于 WebSocket 市场数据流的服务。 即，下订单和收听市场数据需要两个独立的 WebSocket 连接。
    * WebSocket API 与 REST API 相同过滤器和速率限制规则。
    * WebSocket API 与 REST API 提供相同的功能，接受相同的参数，返回相同的状态和错误代码。

**WEBSOCKET API 会晚些时候在生产系统中可用。**

---

### 2022-12-13

REST API

错误代码 `-1003` 的一些错误消息已更改
* 之前的错误消息: `Too much request weight used; current limit is %s request weight per %s %s. Please use the websocket for live updates to avoid polling the API.` 改成了：
```
Too much request weight used; current limit is %s request weight per %s. Please use WebSocket Streams for live updates to avoid polling the API.
```
* 之前的错误消息: `Way too much request weight used; IP banned until %s. Please use the websocket for live updates to avoid bans.` 改成了：
```
Way too much request weight used; IP banned until %s Please use WebSocket Streams for live updates to avoid bans.
```

---

### 2022-12-05

**备注：** 这些更新正在逐步部署到我们所有的服务器，大约需要一周时间才能完成。

WEBSOCKET

* `!bookTicker` 在 **2022-12-07** 下线。 请改用按 symbol 的最优挂单信息的数据流（`<symbol>@bookTicker`）。
    * 可以通过一个连接订阅多个 `<symbol>@bookTicker` 数据流。 （例如`wss://stream.binance.com:9443/stream?streams=btcusdt@bookTicker/bnbbtc@bookTicker`）

REST API

* 新的错误代码 `-1135`
    * 如果参数是无效的 JSON 格式，则会出现此错误代码。
* 新的错误代码 `-1108`
    * 如果发送的参数的值太长，可能会导致溢出，则会发生此错误。
    * 此错误代码可能出现在以下接口：
        * `POST /api/v3/order`
        * `POST /api/v3/order/cancelReplace`
        * `POST /api/v3/order/oco`
* `GET /api/v3/aggTrades` 更新
    * 之前的规则: `startTime` 和 `endTime` 必须结合使用，并且只能相隔一个小时。
    * 新的规则: `startTime` 和 `endTime` 可以单独使用，一个小时的限制已被取消。
        * 仅使用 startTime 时，如果limit的值为N, 将返回从此时间开始的N条交易。
        * 仅使用 endTime 时，如果limit的值为N, 将返回到此时间的N条交易.
        * 如果不提供 `limit`，无论是组合使用还是单独发送，服务器端都将使用默认的 `limit`。
* `GET /api/v3/myTrades` 更新
    * 修复了 `symbol` + `orderId` 组合会返回所有交易，可能会超过`LIMIT`的默认值`500`。
    * 之前的行为： API 将根据发送的参数组合发送特定的错误消息。 例如：

        ```json
        {
            "code": -1106,
            "msg": "Parameter X was sent when not required."
        }
        ```

    * 新的行为: 如果接口不支持可选参数组合，那么服务器会返回一般性的错误:

        ```json
        {
            "code": -1128,
            "msg": "Combination of optional parameters invalid."
        }
        ```
    * 添加一个新的参数组合: `symbol` + `orderId` + `fromId`.
    * 下面的参数组合不再支持:
        * `symbol` + `fromId` + `startTime`
        * `symbol` + `fromId` + `endTime`
        * `symbol` + `fromId` + `startTime` + `endTime`
    * 当前支持的所有参数组合：
        * `symbol`
        * `symbol` + `orderId`
        * `symbol` + `startTime`
        * `symbol` + `endTime`
        * `symbol` + `fromId`
        * `symbol` + `startTime` + `endTime`
        * `symbol`+ `orderId` + `fromId`

**备注：** 这些新字段将在发布日期后大约一周出现。

* `GET /api/v3/exchangeInfo` 更新
    * 新字段 `defaultSelfTradePreventionMode` 和 `allowedSelfTradePreventionModes`
* 下单，查询订单和撤销订单接口的更新:
    * 响应中会出现新的字段 `selfTradePreventionMode`。
    * 以下接口会受到影响:
        * `POST /api/v3/order`
        * `POST /api/v3/order/oco`
        * `POST /api/v3/order/cancelReplace`
        * `GET /api/v3/order`
        * `DELETE /api/v3/order`
        * `DELETE /api/v3/orderList`
* `GET /api/v3/account` 更新
    * 响应中会出现新的字段 `requireSelfTradePrevention`.
* 以下接口的响应中会出现新字段 `workingTime`（指示订单何时添加到了订单薄）：
    * `POST /api/v3/order`
    * `GET /api/v3/order`
    * `POST /api/v3/order/cancelReplace`
    * `POST /api/v3/order/oco`
    * `GET /api/v3/order`
    * `GET /api/v3/openOrders`
    * `GET /api/v3/allOrders`
* 如果`trailingDelta`作为参数提供给了`TAKE_PROFIT`，`TAKE_PROFIT_LIMIT`，`STOP_LOSS`或 `STOP_LOSS_LIMIT`的订单，那么下面接口中会出现`trailingTime`, 用来表示追踪单被激活和跟踪价格变化的时间:
    * `POST /api/v3/order`
    * `GET /api/v3/order`
    * `GET /api/v3/openOrders`
    * `GET /api/v3/allOrders`
    * `POST /api/v3/order/cancelReplace`
    * `DELETE /api/v3/order`
* 字段 `commissionRates` 会在 `GET /api/v3/acccount` 的响应中出现。


USER DATA STREAM

* eventType `executionReport` 有新的字段
    * `V` - `selfTradePreventionMode`
    * `D` - `trailing_time`  (追踪单被激活时会出现)
    * `W` - `workingTime`   (如果 `isWorking`=`true` 会出现)


---

### 2022-12-02

* 新增一个用于访问市场信息的RESTful API URL: `https://data.binance.com`.
* 新增一个用于访问市场信息的WebSocket URL: `wss://data-stream.binance.com`.

---

### 2022-09-30


`!bookTicker`的WebSocket推送的变更.

* 全市场最优挂单信息推送(`!bookTicker`)计划在**2022年11月**下线, 具体下线的时间会在后面通告.
* 请使用按Symbol的最优挂单信息推送(`<symbol>@bookTicker`).
* 多个 `<symbol>@bookTicker` 可以订阅在一个WebSocket连接上.
    * 比如 `wss://stream.binance.com:9443/stream?streams=btcusdt@bookTicker/bnbbtc@bookTicker`

___


### 2022-09-15

这些变动会是滚动发布，可能需要几天才会部署到所有服务器.

* 接口 `GET /api/v3/exchangeInfo` 的变动
    * 添加一个新的参数 `permissions` , 用于查询适用于相应权限的所有交易对.
    * 如果查询时不提供此参数, 则默认值是 `["SPOT","MARGIN","LEVERAGED"]`.
        * 这表示如果请求 `GET /api/v3/exchangeInfo` 时候没有任何参数, 则会返回拥有权限是 `SPOT`, `MARGIN` , `LEVERAGED` 的交易对.
        * 如果要查询其他交易权限, 比如`TRD_GRP_004`等, 需要在查询参数里设置(比如`permissions`=`TRD_GRP_004`).
    * 此参数不可以同时和 `symbol` 或者 `symbols` 使用.


---

### 2022-08-23

此变动会滚动发布, 可能需要一段时间更新到所有服务器上。

* 接口 `GET /api/v3/ticker` 与 `GET /api/v3/ticker/24hr` 变动
    * 添加新可选参数 `type`
    * `type` 可接受的参数值有 `FULL` 与 `MINI`
        * `FULL` 是默认值， 也是原来接口所返回的响应
        * `MINI` 省略了以下字段: `priceChangePercent`, `weightedAvgPrice`, `bidPrice`, `bidQty`, `askPrice`, `askQty` 与 `lastQty`
* 添加新错误代码 `-1008`
    * 每当服务器的请求超载时都会发送此消息
    * 此错误代码只会在 SPOT API 里出现
* 接口 `GET /api/v3/account` 添加新参数 `brokered`
* 添加新接口: `GET /api/v3/uiKlines`
* 添加新k线间隔: `1s`


---

### 2022-08-08

REST API

* 接口 `POST /api/v3/order` 与 `POST /api/v3/order/cancelReplace` 变动
    * 添加新可选参数 `strategyId` 与 `strategyType`
        * `strategyId` 是用于将订单标识为某策略的参数。
        * `strategyType` 是用于标识在执行的策略。(例如：如果所有订单属于现货网格策略，订单可设置为`strategyType=1000000`)
* 接口 `POST /api/v3/order/oco` 变动
    * 添加新可选参数  `limitStrategyId`, `limitStrategyType`, `stopStrategyId`, `stopStrategyType`
    * 这些是OCO订单里两个leg的策略元数据
    * `limitStrategyType` 和 `stopStrategyType` 都不能低于 `1000000`
* 接口 `GET /api/v3/order`, `GET /api/v3/openOrders` 与 `GET /api/v3/allOrders` 变动
    * 新增参数 `strategyId` 与 `strategyType` 必须在下单时填上字段才会在回应JSON里返回
* 接口 `DELETE /api/v3/order` 与 `DELETE /api/v3/openOrders` 变动
    * 新增参数 `strategyId` 与 `strategyType` 必须在下单时填上字段才会在回应JSON里返回


USER DATA STREAM

* eventType `executionReport` 新增参数
    * `j` 代表 `strategyId`
    * `J` 代表 `strategyType`
    * 必须在下单时填上字段才会在回应里返回

---

### 2022-06-20

接口 `GET /api/v3/ticker` 变动

* 权重从每`symbol` 5 降低到 2.
* 每次请求最多可以有100个交易对.
    * 如果`symbols`请求超过100个交易对, 会收到如下错误信息:
    ```json
    {
        "code": -1101,
        "msg": "Too many values sent for parameter 'symbols', maximum allowed up to 100."
    }
    ```
* 单请求的权重上限为100.
    * 比如，如果请求的交易对超过50个，请求的权重是100.

---

### 2022-06-15

**注意:** 此变动不会立刻可用, 会在后面几天上线。


SPOT API

* 添加新接口 `GET /api/v3/ticker`
    * 基于 `windowSize` 返回最近的价格变动。
    * 无需像 `GET /api/v3/ticker/24hr` 提供symbols参数。
    * 如果不提供 `windowSize` 参数，默认值是`1d`。
    * 响应和 `GET /api/v3/ticker/24hr` 相似，但不包括以下数据：`prevClosePrice`, `lastQty`, `bidPrice`, `bidQty`, `askPrice`, `askQty`
* 添加新接口 `POST /api/v3/order/cancelReplace`
    * 撤消当前的挂单并在同样的交易对上下新订单。
    * 过滤器会在**撤单前**做判断。
        * 例如，`MAX_NUM_ORDERS` 是 10，如果目前挂单也是10，调用 `POST /api/v3/order/cancelReplace`会失败。撤单与下单的操作都不会被执行。
    * 更新将在几天后上线，升级完毕后才会开启此功能。
* `GET /api/v3/exchangeInfo` 在`symbols`列表里返回新数据`cancelReplaceAllowed`。
* 添加新的过滤器 `NOTIONAL`
    * 基于`minNotional` 与 `maxNotional` 值来限制名义价值 (`price * quantity`)
* 添加新的过滤器 `EXCHANGE_MAX_NUM_ICEBERG_ORDERS`
    * 账号最大冰山挂单数

WEBSOCKETS

* 新的symbol ticker流, 可以选择 `1h` 或者 `4h`时间窗口：
    * 单个交易对: `<symbol>@ticker_<window-size>`
    * 市场所有交易对: `!ticker_<window-size>@arr`


---


### 2022-05-23
* Order Book 深度的变动
    * 之前深度的数量在一些极端情况下会出现负数.
    * 之后深度数量不会溢出, 而是限制在64位的最大值, 这表示深度的数量达到，或者超过了最大值. 最大值和交易对的`base asset`的精度有关. 比如如果精度是8位小数，最大值则为92,233,720,368.54775807.
    * 原有的深度价位, 在修复上线后, 需要价位上有变动, 才能体现新的修复.
* 哪里有影响?
    * 现货深度接口
        * `GET /api/v3/depth`
    * Websocket Streams
        * `<symbol>@depth`
        * `<symbol>@depth@100ms`
        * `<symbol>@depth<levels>`
        * `<symbol>@depth<levels>@100ms`

* `MAX_POSITION` 的更新
    * 如果一个订单的数量(`quantity`) 可能导致持有仓位溢出, 会触发过滤器 `MAX_POSITION`.


* `GET api/v3/aggTrades` 更新
    * 如果同时提供 `startTime` 和 `endTime`, 旧的记录会返回.
* 如果接口 `GET /api/v3/myTrades` 中没有提供参数 `symbol`, 错误消息变为:

```json
{
    "code": -1102,
    "msg": "Mandatory parameter 'symbol' was not sent, was empty/null, or malformed."
}
```
* 下面的接口提供参数 `symbols` 用于查询多个symbol.
    * `GET /api/v3/ticker/24hr`
    * `GET /api/v3/ticker/price`
    * `GET /api/v3/ticker/bookTicker`

* 上面接口的权重取决于请求 `symbols` 的数量, 具体请看下面的列表:

|接口|Symbols的数量|权重|
|-----|-----|----|
| `GET /api/v3/ticker/price`|Any| 2|
|`GET /api/v3/ticker/bookTicker`|Any|2|
|`GET /api/v3/ticker/24hr`|1-20|1|
|`GET /api/v3/ticker/24hr`|21-100|20|
|`GET /api/v3/ticker/24hr`| >= 101|40|



---

### 2022-04-13

REST API

* 现货交易支持追踪止损(Trailing Stop)订单.
    * 追踪止损通过一个新的参数`trailingDelta`来设置基于市场价的一个自动触发价格.
    * 只适用于订单类型: `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`.
    * 参数`trailingDelta`的单位为基点(BIPS).
        * 比如一个`STOP_LOSS`卖单设置`trailingDelta`为100, 那么订单会在当前市场价格从下单后的最高点下降1%的时候被触发。(100 / 10,000 => 0.01 => 1%)
    * 用于OCO订单的时候, 如果市场变动触发了`STOP_LOSS`订单, 那么此止损订单变成追踪止损订单.
    * 当参数`trailingDelta`和`stopPrice`一起使用时, 一旦`stopPrice`条件被触发，系统会开始追踪当前的价格变动. 从`stopPrice`价格开始，到基于`trailingDelta`值之间变动.
    * 如果没有提供`stopPrice`, 系统开始追踪价格从最新价到基于`trailingDelta`值之间变动.
* `POST /api/v3/order` 变动
    * 添加新可选参数 `trailingDelta`
* `POST /api/v3/order/test` 变动
    * 添加新可选参数 `trailingDelta`
* `POST /api/v3/order/oco` 变动
    * 添加新可选参数 `trailingDelta`
* 添加新的过滤器 `TRAILING_DELTA`
    * 用于限定 `trailingDelta` 的最大和最小值.

USER DATA STREAM

* User Data Stream 的`executionReport`添加新参数
  * "d" 代表`trailingDelta`

---

### 2022-04-12

**Note:** 下面的变更会在后面几天上线.


* `GET api/v3/allOrders` 如果没有提供 `symbol`, 则返回错误信息:
    ```json
    {
        "code": -1102,
        "msg": "Mandatory parameter 'symbol' was not sent, was empty/null, or malformed."
    }
    ```
* 修复一个错误信息中的拼写错误。 如果账号被禁用了相应的权限(比如提款，交易等), 则服务器返回错误:
    ```json
    "This action is disabled on this account."
    ```
* 在市场数据(market data)审计中，发现了一些现货的聚合交易数据(aggTrades)中的问题.
    * 丢失的记录已经被补回.
    * 重复的记录被标记成无效，具体的值设置成如下:
        * p = '0' // price
        * q = '0' // qty
        * f = -1 // ﬁrst_trade_id
        * l = -1 // last_trade_id

---

### 2022-02-28

* 在接口`GET /api/v3/exchangeInfo`中添加新字段`allowTrailingStop`.

---

### 2022-02-24

* 现货规则`PRICE_FILTER`里面的 `(price-minPrice) % tickSize == 0` 改成 `price % tickSize == 0`
* 新添加了一个规则 `PERCENT_PRICE_BY_SIDE`.
* 接口 `GET api/v3/depth` 的变动:
    * `limit` 原先必须是固定值(比如 5, 10, 20, 50, 100, 500, 1000, 5000), 现在可以是在1-5000之间的任意的正整数, 服务器会返回指定的limit数量。(比如如果设置limit=3, 会返回前3个最好的卖价和买价)
    * 如果`limit`超过5000, 服务器也最多返回5000条记录.
    * 相应的, 此接口的权重变成:

|Limit|Request Weight
------|-------
1-100|  1
101-500| 5
501-1000| 10
1001-5000| 50

* GET `api/v3/aggTrades` 接口的变动:
    * 当同时提供参数 `startTime` 和 `endTime`, 最旧的订单会优先返回.

---


### 2021-12-29
* 移除交易对类型枚举
* 新增权限枚举

---

### 2021-11-01
* 新增接口 `GET /api/v3/rateLimit/order`
    * 回传用户在当前时间区间内的下单总数
    * 此接口的权重为20

---

### 2021-09-14
* 添加一个基于OpenAPI规范的RESTful API接口定义的[YAML文件](https://github.com/binance/binance-api-swagger)

---

### 2021-08-12
* GET `api/v3/myTrades` 添加新的参数 `orderId`

---

### 2021-05-12
* 在文档中添加接口的数据来源说明
* 在每个接口中添加相应的数据源
* GET `api/v3/exchangeInfo` 现在支持单个或者多个交易对查询

---

### 2021-04-26

从 **April 28, 2021 00:00 UTC** 开始,下面接口的权重有如下变动:

* `GET /api/v3/order` 权重改为 2
* `GET /api/v3/openOrders` 权重改为 3
* `GET /api/v3/allOrders` 权重改为 10
* `GET /api/v3/orderList` 权重改为 2
* `GET /api/v3/openOrderList` 权重改为 3
* `GET /api/v3/account` 权重改为 10
* `GET /api/v3/myTrades` 权重改为 10
* `GET /api/v3/exchangeInfo` 权重改为 10

---

### 2021-01-01

**USER DATA STREAM**

* 移除`outboundAccountInfo`事件.

---

### 2020-11-27

为了优化性能，除了当前的`api.binance.com`，新加了一些API的集群。如果访问`api.binance.com`有性能问题，也可以尝试访问:

* https://api1.binance.com/api/v3/*
* https://api2.binance.com/api/v3/*
* https://api3.binance.com/api/v3/*

### 2020-09-09

用户数据 STREAM

* `outboundAccountInfo`事件不再推荐使用。
* `outboundAccountInfo`事件以后会被删除(具体时间未定) **请使用 `outboundAccountPosition` 事件.**
* `outboundAccountInfo`只推送余额不为0，以及余额刚变成0的资产。

---

### 2020-05-01
* 从2020-05-01 UTC 00:00开始, 所有交易对都会有最多200个挂单的限制, 体现在过滤器[MAX_NUM_ORDERS](./rest-api_CN.md#max_num_orders-%E6%9C%80%E5%A4%9A%E8%AE%A2%E5%8D%95%E6%95%B0)上.
  * 已经存在的挂单不会被移除或者撤销。
  * 单交易对(`symbol`)的挂单数量达到或超过200的账号, 无法在此交易对上下新的订单, 除非挂单数量低于200。
  * OCO订单在被触发成`LIMIT`订单, 或者被触发成`STOP_LOSS`(或者`STOP_LOSS_LIMIT`)前, 被认为是2个挂单量. 一旦OCO订单被触发, 就只被算作一个挂单。

---

### 2020-04-23

WEB SOCKET 连接限制

* Websocket服务器每秒最多接受5个消息。消息包括:
	* PING帧
	* PONG帧
	* JSON格式的消息, 比如订阅, 断开订阅.
* 如果用户发送的消息超过限制，连接会被断开连接。反复被断开连接的IP有可能被服务器屏蔽。
* 单个连接最多可以订阅 **1024** 个Streams。

---
### 2020-03-24

* 添加过滤器 `MAX_POSITION`.
    * 这个过滤器定义账户允许的基于`base asset`的最大仓位。一个用户的仓位可以定义为如下资产的总和:
        * `base asset`的可用余额
        * `base asset`的锁定余额
        * 所有处于open的买单的数量总和

    * 如果用户的仓位大于最大的允许仓位，买单会被拒绝。

---

### 2018-11-13
REST API
  * 账户交易权限被禁时允许进行撤单操作。
  * 增加了新的过滤器: `PERCENT_PRICE`, `MARKET_LOT_SIZE`, `MAX_NUM_ICEBERG_ORDERS`。
  * 增加了 `RAW_REQUESTS` 频率限制. 该限制仅统计请求次数，不统计请求权重。
  * /api/v3/ticker/price 无symbol参数时，权重增加到2。
  * /api/v3/ticker/bookTicker 无symbol参数时，权重增加到2。
  * DELETE /api/v3/order 现在会返回订单撤销前所处的末次状态。
  * `MIN_NOTIONAL` 新增两个参数: `applyToMarket` (是否对市价单生效) and `avgPriceMins` (对市价单生效时，估算金额时使用过去几分钟的平均价格?).
  *  /api/v1/exchangeInfo 中的限制增加了`intervalNum`. `intervalNum`表示该限制针对多少时间间隔进行统计. 例如: `intervalNum`= 5, `interval` = minute, 表示该限制对每5分钟内的行为进行统计。

#### 如何计算过去n分钟平均价格:
  1. [对过去n分钟所有订单的数量\*价格求和] / 过去n分钟所有订单的数量
  2. 如果过去n分钟没有交易发生，则继续向前追溯，直到找到第一个交易，以此价格作为过去n分钟平均价格。
  3. 如果该交易对之前从未发生过交易，则无平均价格，亦即无法在该交易对下市价单，必须至少有一个（双方均未限价单）的交易成交后才可以下市价单。
  4. 当前系统使用的平均价格可以通过接口 `https://api.binance.com/api/v3/avgPrice?symbol=<symbol>`查询
     例如
     https://api.binance.com/api/v3/avgPrice?symbol=BNBUSDT

USER DATA STREAM
  * 成交报告中增加了 `末次成交金额` (`Y`)，等于 `末次成交量` * `末次成交价格` (`L` * `l`).

---

### 2018-07-18
REST API
  *  新过滤器: `ICEBERG_PARTS`
  *  `POST api/v3/order` 中 `newOrderRespType` 参数的缺省值更改; `MARKET`  `LIMIT` 默认为 `FULL`, 其他默认为 `ACK`.
  *  POST api/v3/order `RESULT` 与 `FULL` 响应中增加 `cummulativeQuoteQty`
  *  GET api/v3/openOrders 无symbol调用权重下降为 40.
  *  GET api/v3/ticker/24hr 无symbol调用权重下降为 40.
  *  GET /api/v1/trades amount最大可取到1000.
  *  GET /api/v1/historicalTrades amount最大可取到1000.
  *  GET /api/v1/aggTrades amount最大可取到1000.
  *  GET /api/v1/klines amount最大可取到1000.
  *  订单查询结果返回中增加 `updateTime` 字段，代表该订单末次更新(创建、成交、过期、取消、拒绝等等)时间; `time` 仅表示创建时间.
  *  订单查询结果中增加 `cummulativeQuoteQty`字段. 负值表示尚无任何成交，该字段不可用.
  *  `REQUESTS` 限制更名为 `REQUEST_WEIGHT`. 避免名字造成的误解。

USER DATA STREAM
  *  订单报告与成交报告中增加`cummulativeQuoteQty` 字段 (`Z`). 表示已经成交的金额， 即已经花费的金额(买入订单)或已经收到的金额(卖出订单)，均未计算手续费. 此功能增加之前成交的订单在历史订单接口中查询到的该字段可能小于零.
  *  `cummulativeQuoteQty`/`cummulativeQty` 可以用来计算该订单已经成交部分的平均价格。
  *  成交报告中增加了 `O`字段 (订单创建时间)

---

### 2018-01-23
  * GET /api/v1/historicalTrades权重降为 5
  * GET /api/v1/aggTrades 权重降为 1
  * GET /api/v1/klines 权重降为 1
  * GET /api/v1/ticker/24hr 不带symbol参数的权重降为 symbols总数 / 2
  * GET /api/v3/allOrders 权重降为 5
  * GET /api/v3/myTrades 权重降为 5
  * GET /api/v3/account 权重降为 5
  * GET /api/v1/depth limit=500 权重降为 5
  * GET /api/v1/depth limit=1000 权重降为 10
  * websocket 用户增加 -1003 error code

---

### 2018-01-20
  * GET /api/v1/ticker/24hr 单symbol参数调用权重降为 1
  * GET /api/v3/openOrders 不带symbol参数的权重降为 symbols总数 / 2
  * GET /api/v3/allOrders  权重降为  15
  * GET /api/v3/myTrades  权重降为  15
  * GET /api/v3/order  权重降为  1
  * 自成交现在会在myTrades结果中有两条记录。

---

### 2018-01-14
  * GET /api/v1/aggTrades 权重改为 2
  * GET /api/v1/klines 权重改为 2
  * GET /api/v3/order 权重改为 2
  * GET /api/v3/allOrders 权重改为 20
  * GET /api/v3/account 权重改为 20
  * GET /api/v3/myTrades 权重改为 20
  * GET /api/v3/historicalTrades 权重改为 20
