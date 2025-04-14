# WebSocket 账户接口

**最近更新： 2025-04-09**

## 基本信息
* 目前有两种方法可以订阅 User Data Stream:
  * **[首选]** 直接通过 [WebSocket API](web-socket-api_CN.md#user_data_stream_susbcribe) 使用 API 密钥.
  * **[已弃用]** 通过使用 [REST API](rest-api_CN.md#user-data-stream-requests)或 [WebSocket API](web-socket-api_CN.md#user-data-stream-requests)生成一个 **listen key** 并使用它来监听 **stream.binance.com**。
* 两个源都将**实时** 推送与您的帐户相关的所有事件.
* 如何在 **stream.binance.com** 使用 `listen key`:
  * 基本端点是**wss://stream.binance.com:9443** 或 **wss://stream.binance.com:443**。
  * 连接到 **stream.binance.com** 的连接仅在 24 小时内有效;将会在到达 24 小时时断开连接。
  * 用户数据流可通过 **/WS/\<listenKey\>** 或 **/stream？streams=\<listenKey\>** 访问。
  * JSON payload 中所有与时间和时间戳相关的字段默认为 **毫秒**。以微秒为单位接收信息,请在连接 URL 中添加参数 `timeUnit=MICROSECOND` 或 `timeUnit=microsecond`。
    * 例如 `/WS/<listenKey>？timeUnit=MICROSECOND`

## 用户数据流事件

### 账户更新

每当帐户余额发生更改时，都会发送一个事件`outboundAccountPosition`，其中包含可能由生成余额变动的事件而变动的资产。

**Payload**

```javascript
{
  "e": "outboundAccountPosition", // 事件类型
  "E": 1564034571105,             // 事件时间
  "u": 1564034571073,             // 账户末次更新时间戳
  "B": [                          // 余额
    {
      "a": "ETH",                 // 资产名称
      "f": "10000.000000",        // 可用余额
      "l": "0.000000"             // 冻结余额
    }
  ]
}
```

### 余额更新

当下列情形发生时更新:

* 账户发生充值或提取
* 交易账户之间发生划转(例如 现货向杠杆账户划转)

**Payload**

```javascript
{
  "e": "balanceUpdate",         // Event Type
  "E": 1573200697110,           // Event Time
  "a": "ABC",                   // Asset
  "d": "100.00000000",          // Balance Delta
  "T": 1573200697068            // Clear Time
}
```

<a id="executionReport"></a>
### 订单更新
订单通过`executionReport`事件进行更新。

与使用用户数据流相比，我们建议使用 [FIX API](fix-api_CN.md) 以获得更好的性能。


**Payload:**

```javascript
{
  "e": "executionReport",        // 事件类型
  "E": 1499405658658,            // 事件时间
  "s": "ETHBTC",                 // 交易对
  "c": "mUvoqJxFIILMdfAW5iGSOW", // clientOrderId
  "S": "BUY",                    // 订单方向
  "o": "LIMIT",                  // 订单类型
  "f": "GTC",                    // 有效方式
  "q": "1.00000000",             // 订单原始数量
  "p": "0.10264410",             // 订单原始价格
  "P": "0.00000000",             // 止盈止损单触发价格
  "F": "0.00000000",             // 冰山订单数量
  "g": -1,                       // OCO订单 OrderListId
  "C": "",                       // 原始订单自定义ID(原始订单，指撤单操作的对象。撤单本身被视为另一个订单)
  "x": "NEW",                    // 本次事件的具体执行类型
  "X": "NEW",                    // 订单的当前状态
  "r": "NONE",                   // 订单被拒绝的原因
  "i": 4293153,                  // orderId
  "l": "0.00000000",             // 订单末次成交量
  "z": "0.00000000",             // 订单累计已成交量
  "L": "0.00000000",             // 订单末次成交价格
  "n": "0",                      // 手续费数量
  "N": null,                     // 手续费资产类别
  "T": 1499405658657,            // 成交时间
  "I": 8641984,                  // Execution ID
  "w": true,                     // 订单是否在订单簿上？
  "m": false,                    // 该成交是作为挂单成交吗？
  "M": false,                    // 请忽略
  "O": 1499405658657,            // 订单创建时间
  "Z": "0.00000000",             // 订单累计已成交金额
  "Y": "0.00000000",             // 订单末次成交金额
  "Q": "0.00000000",             // Quote Order Quantity
  "D": 1668680518494,            // 追踪时间; 这仅在追踪止损订单已被激活时可见
  "W": 1499405658657,            // Working Time; 订单被添加到 order book 的时间
  "V": "NONE"                    // SelfTradePreventionMode
}
```

**备注:** 通过将`Z`除以`z`可以找到平均价格。

#### `executionReport` 中的特定条件时才会出现的字段

这些字段仅在满足特定条件时才会出现。有关这些参数的更多信息，请参阅 [现货交易API术语表](./faqs/spot_glossary_CN.md)。

<table>
  <tr>
    <th>字段</th>
    <th>名称</th>
    <th>描述</th>
    <th>示例</th>
  </tr>
  <tr>
    <td><code>d</code></td>
    <td>Trailing Delta</td>
    <td rowspan="2">出现在追踪止损订单中。</td>
    <td><code>"d": 4</code></td>
  </tr>
  <tr>
    <td><code>D</code></td>
    <td>Trailing Time</td>
    <td><code>"D": 1668680518494</code></td>
  </tr>
  <tr>
    <td><code>j</code></td>
    <td>Strategy Id</td>
    <td>如果在请求中添加了<code>strategyId</code>参数，则会出现。</td>
    <td><code>"j": 1</code></td>
  </tr>
  <tr>
    <td><code>J</code></td>
    <td>Strategy Type</td>
    <td>如果在请求中添加了<code>strategyType</code>参数，则会出现。</td>
    <td><code>"J": 1000000</code></td>
  </tr>
  <tr>
    <td><code>v</code></td>
    <td>Prevented Match Id</td>
    <td rowspan="9">只有在因为 STP 导致订单失效时可见。</td>
    <td><code>"v": 3</code></td>
  </tr>
  <tr>
    <td><code>A</code>
    <td>Prevented Quantity</td>
    <td><code>"A":"3.000000"</code></td>
  </tr>
  <tr>
    <td><code>B</code></td>
    <td>Last Prevented Quantity</td>
    <td><code>"B":"3.000000"</code></td>
  </tr>
  <tr>
    <td><code>u</code></td>
    <td>Trade Group Id</td>
    <td><code>"u":1</code></td>
  </tr>
  <tr>
    <td><code>U</code></td>
    <td>Counter Order Id</td>
    <td><code>"U":37</code></td>
  </tr>
  <tr>
    <td><code>Cs</code></td>
    <td>Counter Symbol</td>
    <td><code>"Cs": "BTCUSDT"</code></td>
  </tr>
  <tr>
    <td><code>pl</code></td>
    <td>Prevented Execution Quantity</td>
    <td><code>"pl":"2.123456"</code></td>
  </tr>
  <tr>
    <td><code>pL</code></td>
    <td>Prevented Execution Price</td>
    <td><code>"pL":"0.10000001"</code></td>
  </tr>
  <tr>
    <td><code>pY</code></td>
    <td>Prevented Execution Quote Qty</td>
    <td><code>"pY":"0.21234562"</code></td>
  </tr>
  <tr>
    <td><code>W</code></td>
    <td>Working Time</td>
    <td>只有在订单在订单簿上时可见</td>
    <td><code>"W": 1668683798379</code></td>
  </tr>
  <tr>
    <td><code>b</code></td>
    <td>Match Type</td>
    <td rowspan="2">只有在订单有分配时可见</td>
    <td><code>"b":"ONE_PARTY_TRADE_REPORT"</code></td>
  </tr>
  <tr>
    <td><code>a</code></td>
    <td>Allocation ID</td>
    <td><code>"a":1234</code></td>
  </tr>
  <tr>
    <td><code>k</code></td>
    <td>Working Floor</td>
    <td>只有在订单可能有分配时可见</td>
    <td><code>"k":"SOR"</code></td>
  </tr>
  <tr>
    <td><code>uS</code></td>
    <td>UsedSor</td>
    <td>只有在订单使用 SOR 时可见</td>
    <td><code>"uS":true</code></td>
  </tr>
</table>

如果是一个订单组，则除了显示`executionReport`事件外，还将显示一个名为`ListStatus`的事件。

**Payload**

```javascript
{
  "e": "listStatus",                // 事件类型
  "E": 1564035303637,               // 事件时间
  "s": "ETHBTC",                    // 交易对
  "g": 2,                           // OrderListId
  "c": "OCO",                       // Contingency Type
  "l": "EXEC_STARTED",              // List Status Type
  "L": "EXECUTING",                 // List Order Status
  "r": "NONE",                      // List 被拒绝的原因
  "C": "F4QN4G8DlFATFlIUQ0cjdD",    // List Client Order ID
  "T": 1564035303625,               // 成交时间
  "O": [
    {
      "s": "ETHBTC",                // 交易对
      "i": 17,                      // orderId
      "c": "AJYsMjErWJesZvqlJCTUgL" // clientOrderId
    },
    {
      "s": "ETHBTC",
      "i": 18,
      "c": "bfYPSQdLoqAJeNrOr9adzq"
    }
  ]
}
```

**可能的执行类型:**

* `NEW` - 新订单已被引擎接受。
* `CANCELED` - 订单被用户取消。
* `REPLACED` - 订单已被修改。
* `REJECTED` - 新订单被拒绝 （e.g. 在撤消挂单再下单时，其中新订单被拒绝但撤消挂单请求成功）。
* `TRADE` - 订单有新成交。
* `EXPIRED` - 订单已根据 Time In Force 参数的规则取消（e.g. 没有成交的 LIMIT FOK 订单或部分成交的 LIMIT IOC 订单）或者被交易所取消（e.g. 强平或维护期间取消的订单）。
* `TRADE_PREVENTION` - 订单因 STP 触发而过期。

请查阅 [枚举定义](./enums_CN.md) 文档获取更多枚举定义。

### Listen Key 已过期

当监听 listen key 过期时会发送此事件。此后不会再发送任何事件，直到创建新的 `listenKey`。

正常关闭流时不会推送该事件。

**Payload:**

```javascript
{
  "e": "listenKeyExpired",  // 事件类型
  "E": 1699596037418,      // 事件时间
  "listenKey": "OfYGbUzi3PraNagEkdKuFwUHn48brFsItTdsuiIXrucEvD0rhRXZ7I6URWfE8YE8"
}
```


## 事件流已终止

此事件仅在订阅 WebSocket API 时显示。

当账户数据流被终止时，`eventStreamTerminated` 会被发送。例如，在您发送 `userDataStream.unsubscribe` 请求或 `session.logout` 请求之后。

**Payload:**

```javascript
{
  "event": {
    "e": "eventStreamTerminated", // Event Type
    "E": 1728973001334            // Event Time
  }
}
```

## 外部锁定更新

当您的现货钱包余额被外部系统锁定/解锁时 （例如，当用作保证金抵押品时），新事件 `externalLockUpdate` 将会被发送。

**Payload:**

```javascript
{
  "e": "externalLockUpdate",  // Event Type
  "E": 1581557507324,         // Event Time
  "a": "NEO",                 // Asset
  "d": "10.00000000",         // Delta
  "T": 1581557507268          // Transaction Time
}
```
