# WebSocket 账户接口

## 一般信息

* 通过使用 API Key 订阅 [WebSocket API](web-socket-api_CN.md#user-data-stream-subscribe)。
* 支持 [SBE](./faqs/sbe_faq_CN.md) 和 JSON 输出格式。
* 账户事件以 **实时** 方式推送。
* JSON 数据中的所有时间戳默认均为 **毫秒**。
* 如果您持有或交易任何名称包含非 ASCII 字符的资产或交易对，那么事件中可能包含以 UTF-8 编码的非 ASCII 字符。

## 用户数据流事件

### 账户更新

每当帐户余额发生更改时，都会发送一个事件`outboundAccountPosition`，其中包含可能由生成余额变动的事件而变动的资产。

**Payload**

```javascript
{
  "subscriptionId": 0,
  "event": {
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
}
```

### 余额更新

当下列情形发生时更新:

* 账户发生充值或提取
* 交易账户之间发生划转(例如 现货向杠杆账户划转)

**Payload**

```javascript
{
  "subscriptionId": 0,
  "event": {
    "e": "balanceUpdate",         // 事件类型
    "E": 1573200697110,           // 事件时间
    "a": "BTC",                   // 资产名称
    "d": "100.00000000",          // 余额增量
    "T": 1573200697068            // 清算时间
  }
}
```

<a id="executionReport"></a>
### 订单更新
订单通过`executionReport`事件进行更新。


**Payload:**

```javascript
{
  "subscriptionId": 0,
  "event": {
    "e": "executionReport",         // 事件类型
    "E": 1499405658658,             // 事件时间
    "s": "ETHBTC",                  // 交易对
    "c": "mUvoqJxFIILMdfAW5iGSOW",  // clientOrderId
    "S": "BUY",                     // 订单方向
    "o": "LIMIT",                   // 订单类型
    "f": "GTC",                     // 有效方式
    "q": "1.00000000",              // 订单原始数量
    "p": "0.10264410",              // 订单原始价格
    "P": "0.00000000",              // 止盈止损单触发价格
    "F": "0.00000000",              // 冰山订单数量
    "g": -1,                        // OCO订单 OrderListId
    "C": "",                        // 原始订单自定义ID（原始订单，指撤单操作的对象。撤单本身被视为另一个订单）
    "x": "NEW",                     // 本次事件的具体执行类型
    "X": "NEW",                     // 订单的当前状态
    "r": "NONE",                    // 订单被拒绝的原因；请参阅订单被拒绝的原因（下文）了解更多信息
    "i": 4293153,                   // orderId
    "l": "0.00000000",              // 订单末次成交量
    "z": "0.00000000",              // 订单累计已成交量
    "L": "0.00000000",              // 订单末次成交价格
    "n": "0",                       // 手续费数量
    "N": null,                      // 手续费资产类别
    "T": 1499405658657,             // 成交时间
    "t": -1,                        // Trade ID
    "v": 3,                         // 被阻止的交易Id；仅在订单因为STP被阻止时显示
    "I": 8641984,                   // Execution ID
    "w": true,                      // 订单是否在订单簿上？
    "m": false,                     // 该成交是作为挂单成交吗？
    "M": false,                     // 请忽略
    "O": 1499405658657,             // 订单创建时间
    "Z": "0.00000000",              // 订单累计已成交金额
    "Y": "0.00000000",              // 订单末次成交金额
    "Q": "0.00000000",              // Quote Order Quantity
    "W": 1499405658657,             // Working Time; 订单被添加到 order book 的时间
    "V": "NONE"                     // SelfTradePreventionMode
  }
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
   <tr>
      <td><code>gP</code></td>
      <td>Pegged Price Type</td>
      <td rowspan="4">仅出现在挂钩订单中</td>
      <td><code>"gP": "PRIMARY_PEG"</code></td>
   </tr>
   <tr>
      <td><code>gOT</code></td>
      <td>Pegged offset Type</td>
      <td><code>"gOT": "PRICE_LEVEL"</code></td>
   </tr>
   <tr>
      <td><code>gOV</code></td>
      <td>Pegged Offset Value</td>
      <td><code>"gOV": 5</code></td>
   </tr>
   <tr>
      <td><code>gp</code></td>
      <td>Pegged Price</td>
      <td><code>"gp": "1.00000000"</code></td>
   </tr>
</table>

#### 订单拒绝原因

有关更多详细信息，请查阅 [错误代码汇总](errors_CN.md#other-errors) 文档中的错误消息。

|拒绝原因 (`r`)| 错误信息|
|---             | ---          |
|`NONE`           | N/A (i.e. The order was not rejected.)|
|`INSUFFICIENT_BALANCES`|"Account has insufficient balance for requested action."|
|`STOP_PRICE_WOULD_TRIGGER_IMMEDIATELY`|"Order would trigger immediately."|
|`WOULD_MATCH_IMMEDIATELY`|"Order would immediately match and take."|
|`OCO_BAD_PRICES`|"The relationship of the prices for the orders is not correct."|

如果是一个订单组，则除了显示 `executionReport` 事件外，还将显示一个名为 `ListStatus` 的事件。

**Payload**

```javascript
{
  "subscriptionId": 0,
  "event": {
    "e": "listStatus",                // 事件类型
    "E": 1564035303637,               // 事件时间
    "s": "ETHBTC",                    // 交易对
    "g": 2,                           // OrderListId
    "c": "OCO",                       // Contingency 类型
    "l": "EXEC_STARTED",              // List 状态类型
    "L": "EXECUTING",                 // List 订单类型
    "r": "NONE",                      // List 被拒绝的原因
    "C": "F4QN4G8DlFATFlIUQ0cjdD",    // List Client Order ID
    "T": 1564035303625,               // 成交时间
    "O": [                            // 对象数组
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

<a id="event-stream-terminated"></a>
## 事件流已终止

此事件仅在订阅 WebSocket API 时显示。

`eventStreamTerminated` 会在以下情况下发送：

* 当 [Listen Token 订阅](https://developers.binance.com/docs/zh-CN/margin_trading/trade-data-stream/Listen-Token-Websocket-API) 因 Token 过期而失效时。
* 在发送 [`session.logout`](https://developers.binance.com/docs/zh-CN/binance-spot-api-docs/websocket-api/authentication-requests#%E9%80%80%E5%87%BA%E4%BC%9A%E8%AF%9D) 方法后，[登录订阅](https://developers.binance.com/docs/zh-CN/binance-spot-api-docs/websocket-api/authentication-requests#%E7%94%A8api-key%E7%99%BB%E5%BD%95-signed) 结束时。
* 通过 [`userDataStream.unsubscribe`](https://developers.binance.com/docs/zh-CN/binance-spot-api-docs/websocket-api/user-data-stream-requests#%E5%8F%96%E6%B6%88%E8%AE%A2%E9%98%85%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E6%B5%81) 方法终止订阅时。

**Payload:**

```javascript
{
  "subscriptionId": 0,
  "event": {
    "e": "eventStreamTerminated", // 事件类型
    "E": 1728973001334            // 事件时间
  }
}
```

## 外部锁定更新

当您的现货钱包余额被外部系统锁定/解锁时 （例如，当用作保证金抵押品时），新事件 `externalLockUpdate` 将会被发送。

**Payload:**

```javascript
{
  "subscriptionId": 0,
  "event": {
    "e": "externalLockUpdate",  // 事件类型
    "E": 1581557507324,         // 事件时间
    "a": "NEO",                 // 资产
    "d": "10.00000000",         // 余额变动量
    "T": 1581557507268          // 交易时间
  }
}
```
