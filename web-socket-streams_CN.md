
# Web Socket 行情接口

**最近更新： 2024-12-17**

## 基本信息
* 本篇所列出的所有wss接口的baseurl为: **wss://stream.binance.com:9443** 或者 **wss://stream.binance.com:443**
* 所有stream均可以直接访问，或者作为组合streams的一部分。
* 直接访问时URL格式为 **/ws/\<streamName\>**
* 组合streams的URL格式为 **/stream?streams=\<streamName1\>/\<streamName2\>/\<streamName3\>**
* 订阅组合streams时，事件payload会以这样的格式封装 **{"stream":"\<streamName\>","data":\<rawPayload\>}**
* stream名称中所有交易对均为**小写**
* 每个到**stream.binance.com**的链接有效期不超过24小时，请妥善处理断线重连。
* Websocket 服务器每3分钟发送Ping消息。
    * 如果Websocket服务器在10分钟之内没有收到Pong消息应答，连接会被断开。
    * 当客户收到ping消息，必需尽快回复pong消息，同时payload需要和ping消息一致。
    * 未经请求的pong消息是被允许的，但是不会保证连接不断开。**对于这些pong消息，建议payload为空**
* **wss://data-stream.binance.vision** 可以用来订阅仅有市场信息的数据流。账户信息**无法**从此URL获得。
* 所有时间和时间戳相关字段均以**毫秒为默认单位**。 要以微秒为单位接收信息，请在 URL 中添加参数 `timeUnit=MICROSECOND` 或 `timeUnit=microsecond`。 
  * 例如： `/stream?streams=btcusdt@trade&timeUnit=MICROSECOND`

## WebSocket 连接限制

* WebSocket服务器每秒最多接受5个消息。消息包括:
    * PING帧
    * PONG帧
    * JSON格式的消息, 比如订阅, 断开订阅.
* 如果用户发送的消息超过限制，连接会被断开连接。反复被断开连接的IP有可能被服务器屏蔽。
* 单个连接最多可以订阅1024个Streams。
* 每IP地址、每5分钟最多可以发送300次连接请求。



## 实时订阅/取消数据流

* 以下数据可以通过websocket发送以实现订阅或取消订阅数据流。示例如下.
* 请求中的`id`被用作唯一标识来区分来回传递的消息。以下格式被接受:
  * 64位有符号整数
  * 字母数字字符串；最大长度36
  * `null`
* 如果相应内容中的`result` 为 `null`，表示请求发送成功。

### 订阅一个信息流

* 请求

  ```javascript
  {
    "method": "SUBSCRIBE",
    "params": [
      "btcusdt@aggTrade",
      "btcusdt@depth"
    ],
    "id": 1
  }
  ```

* 响应

  ```javascript
  {
    "result": null,
    "id": 1
  }
  ```

### 取消订阅一个信息流

* 请求

  ```javascript
  {
    "method": "UNSUBSCRIBE",
    "params": [
      "btcusdt@depth"
    ],
    "id": 312
  }
  ```

* 响应

  ```javascript
  {
    "result": null,
    "id": 312
  }
  ```


### 已订阅信息流

* 请求

  ```javascript
  {
    "method": "LIST_SUBSCRIPTIONS",
    "id": 3
  }
  ```

* 响应

  ```javascript
  {
    "result": [
      "btcusdt@aggTrade"
    ],
    "id": 3
  }
  ```


### 设定属性
当前，唯一可以设置的属性是设置是否启用`combined`("组合")信息流。
当使用`/ws/`("原始信息流")进行连接时，combined属性设置为`false`，而使用 `/stream/`进行连接时则将属性设置为`true`。

* 请求

  ```javascript
  {
    "method": "SET_PROPERTY",
    "params": [
      "combined",
      true
    ],
    "id": 5
  }
  ```

* 响应

  ```javascript
  {
    "result": null,
    "id": 5
  }
  ```

### 检索属性

* 请求

  ```javascript
  {
    "method": "GET_PROPERTY",
    "params": [
      "combined"
    ],
    "id": 2
  }
  ```

* 响应

  ```javascript
  {
    "result": true, // Indicates that combined is set to true.
    "id": 2
  }
  ```

### 错误信息

错误信息 | 描述
---|---
{"code": 0, "msg": "Unknown property","id": %s} | `SET_PROPERTY` 或 `GET_PROPERTY`中应用的参数无效
{"code": 1, "msg": "Invalid value type: expected Boolean"} | 仅接受`true`或`false`
{"code": 2, "msg": "Invalid request: property name must be a string"}| 提供的属性名无效
{"code": 2, "msg": "Invalid request: request ID must be an unsigned integer"}| 参数`id`未提供或`id`值是无效类型
{"code": 2, "msg": "Invalid request: unknown variant %s, expected one of `SUBSCRIBE`, `UNSUBSCRIBE`, `LIST_SUBSCRIPTIONS`, `SET_PROPERTY`, `GET_PROPERTY` at line 1 column 28"} | 错字提醒，或提供的值不是预期类型
{"code": 2, "msg": "Invalid request: too many parameters"}| 数据中提供了不必要参数
{"code": 2, "msg": "Invalid request: property name must be a string"} | 未提供属性名
{"code": 2, "msg": "Invalid request: missing field `method` at line 1 column 73"} | 数据未提供`method`
{"code":3,"msg":"Invalid JSON: expected value at line %s column %s"} | JSON 语法有误.


# Stream 详细定义
<a id="aggtrade"></a>
## 归集交易
归集交易与逐笔交易的区别在于，同一个taker在同一价格与多个maker成交时，会被归集为一笔成交。

**Stream 名称:** \<symbol\>@aggTrade

**更新速度:** 实时

**Payload:**
```javascript
{
  "e": "aggTrade",      // 事件类型
  "E": 1672515782136,   // 事件时间
  "s": "BNBBTC",        // 交易对
  "a": 12345,           // 归集交易ID
  "p": "0.001",         // 成交价格
  "q": "100",           // 成交数量
  "f": 100,             // 被归集的首个交易ID
  "l": 105,             // 被归集的末次交易ID
  "T": 1672515782136,   // 成交时间
  "m": true,            // 买方是否是做市方。如true，则此次成交是一个主动卖出单，否则是一个主动买入单。
  "M": true             // 请忽略该字段
}
```

<a id="trade"></a>

## 逐笔交易
逐笔交易推送每一笔成交的信息。**成交**，或者说交易的定义是仅有一个吃单者与一个挂单者相互交易。

**Stream 名称:** \<symbol\>@trade

**更新速度:** 实时

**Payload:**
```javascript
{
  "e": "trade",        // 事件类型
  "E": 1672515782136,  // 事件时间
  "s": "BNBBTC",       // 交易对
  "t": 12345,          // 交易ID
  "p": "0.001",        // 成交价格
  "q": "100",          // 成交数量
  "T": 1672515782136,  // 成交时间
  "m": true,           // 买方是否是做市方。如true，则此次成交是一个主动卖出单，否则是一个主动买入单。
  "M": true            // 请忽略该字段
}
```
<a id="kline"></a>
## UTC K线
K线stream逐秒推送所请求的K线种类(最新一根K线)的更新。此更新是基于 `UTC+0` 时区的。

<a id="kline-intervals"></a>
**订阅Kline需要提供间隔参数，最短为分钟线，最长为月线。支持以下间隔:**

m -> 分钟; h -> 小时; d -> 天; w -> 周; M -> 月

* 1m
* 3m
* 5m
* 15m
* 30m
* 1h
* 2h
* 4h
* 6h
* 8h
* 12h
* 1d
* 3d
* 1w
* 1M

**Stream 名称:** \<symbol\>@kline_\<interval\>

**更新速度:** `1s` 1000ms，其它间隔 2000ms

**Payload:**
```javascript
{
  "e": "kline",          // 事件类型
  "E": 1672515782136,    // 事件时间
  "s": "BNBBTC",         // 交易对
  "k": {
    "t": 1672515780000,  // 这根K线的起始时间
    "T": 1672515839999,  // 这根K线的结束时间
    "s": "BNBBTC",       // 交易对
    "i": "1m",           // K线间隔
    "f": 100,            // 这根K线期间第一笔成交ID
    "L": 200,            // 这根K线期间末一笔成交ID
    "o": "0.0010",       // 这根K线期间第一笔成交价
    "c": "0.0020",       // 这根K线期间末一笔成交价
    "h": "0.0025",       // 这根K线期间最高成交价
    "l": "0.0015",       // 这根K线期间最低成交价
    "v": "1000",         // 这根K线期间成交量
    "n": 100,            // 这根K线期间成交数量
    "x": false,          // 这根K线是否完结（是否已经开始下一根K线）
    "q": "1.0000",       // 这根K线期间成交额
    "V": "500",          // 主动买入的成交量
    "Q": "0.500",        // 主动买入的成交额
    "B": "123456"        // 忽略此参数
  }
}
```
## 带有时区偏移量的K线
K线stream逐秒推送所请求的K线种类(最新一根K线)的更新。此更新是基于 `UTC+8` 时区的。

**订阅Kline需要提供的间隔参数:**

参考 [`Kline所支持的间隔参数`](#kline-intervals)

**UTC+8 时区偏移量：**

* K线间隔的开始和结束时间会基于 `UTC+8` 时区。例如， `1d` K线将在 `UTC+8` 当天开始，并在 `UTC+8` 当日完结时随之结束。
* 请注意，Payload中的 `E`（event time），`t`（start time）和 `T`（close time）是 Unix 时间戳，它们始终以 UTC 格式解释。

**Stream 名称:** \<symbol\>@kline_\<interval\>@+08:00

**更新速度:** `1s` 1000ms，其它间隔 2000ms

**Payload:**
```javascript
{
  "e": "kline",         // Event type
  "E": 1672515782136,   // Event time
  "s": "BNBBTC",        // Symbol
  "k": {
    "t": 1672515780000, // Kline start time
    "T": 1672515839999, // Kline close time
    "s": "BNBBTC",      // Symbol
    "i": "1m",          // Interval
    "f": 100,           // First trade ID
    "L": 200,           // Last trade ID
    "o": "0.0010",      // Open price
    "c": "0.0020",      // Close price
    "h": "0.0025",      // High price
    "l": "0.0015",      // Low price
    "v": "1000",        // Base asset volume
    "n": 100,           // Number of trades
    "x": false,         // Is this kline closed?
    "q": "1.0000",      // Quote asset volume
    "V": "500",         // Taker buy base asset volume
    "Q": "0.500",       // Taker buy quote asset volume
    "B": "123456"       // Ignore
  }
}
```

<a id="twentyfourhourminiticker"></a>
## 按Symbol的精简Ticker
按Symbol逐秒刷新的24小时精简ticker信息

**Stream 名称:** \<symbol\>@miniTicker

**更新速度:** 1000ms

**Payload:**
```javascript
  {
    "e": "24hrMiniTicker",  // 事件类型
    "E": 1672515782136,     // 事件时间
    "s": "BNBBTC",          // 交易对
    "c": "0.0025",          // 最新成交价格
    "o": "0.0010",          // 24小时前开始第一笔成交价格
    "h": "0.0025",          // 24小时内最高成交价
    "l": "0.0010",          // 24小时内最低成交加
    "v": "10000",           // 成交量
    "q": "18"               // 成交额
  }
```

<a id="all-markets-mini-ticker"></a>
## 全市场所有Symbol的精简Ticker
同上，只是推送所有交易对

**Stream名称:** !miniTicker@arr

**更新速度:** 1000ms

**Payload:**
```javascript
[
  {
    // 数组每一个元素对应一个交易对，内容与 \<symbol\>@miniTicker相同
  }
]
```

<a id="twentyfourhourticker"></a>

## 按Symbol的完整Ticker
按Symbol逐秒刷新的24小时完整ticker信息

**Stream 名称:** \<symbol\>@ticker

**更新速度:** 1000ms

**Payload:**
```javascript
{
  "e": "24hrTicker",  // 事件类型
  "E": 1672515782136, // 事件时间
  "s": "BNBBTC",      // 交易对
  "p": "0.0015",      // 24小时价格变化
  "P": "250.00",      // 24小时价格变化（百分比）
  "w": "0.0018",      // 平均价格
  "x": "0.0009",      // 整整24小时之前，向前数的最后一次成交价格
  "c": "0.0025",      // 最新成交价格
  "Q": "10",          // 最新成交交易的成交量
  "b": "0.0024",      // 目前最高买单价
  "B": "10",          // 目前最高买单价的挂单量
  "a": "0.0026",      // 目前最低卖单价
  "A": "100",         // 目前最低卖单价的挂单量
  "o": "0.0010",      // 整整24小时前，向后数的第一次成交价格
  "h": "0.0025",      // 24小时内最高成交价
  "l": "0.0010",      // 24小时内最低成交加
  "v": "10000",       // 24小时内成交量
  "q": "18",          // 24小时内成交额
  "O": 0,             // 统计开始时间
  "C": 1675216573749, // 统计结束时间
  "F": 0,             // 24小时内第一笔成交交易ID
  "L": 18150,         // 24小时内最后一笔成交交易ID
  "n": 18151          // 24小时内成交数
}
```

<a id="all-markets-ticker"></a>
## 全市场所有交易对的完整Ticker
同上，只是推送所有交易对

**Stream 名称:** !ticker@arr

**更新速度:** 1000ms

**Payload:**
```javascript
[
  {
    // 对应每一个交易对的 <symbol>@ticker 内容
  }
]
```
<a id="bookticker"></a>
## 按Symbol的最优挂单信息

实时推送指定交易对最优挂单信息
多个 `<symbol>@bookTicker` 可以订阅在一个WebSocket连接上

**Stream 名称:** \<symbol\>@bookTicker

**更新速度:** 实时

**Payload:**
```javascript
{
  "u":400900217,     // order book updateId
  "s":"BNBUSDT",     // 交易对
  "b":"25.35190000", // 买单最优挂单价格
  "B":"31.21000000", // 买单最优挂单数量
  "a":"25.36520000", // 卖单最优挂单价格
  "A":"40.66000000"  // 卖单最优挂单数量
}
```


## 平均价格

平均价格流推送在固定时间间隔内的平均价格变动。

**Stream 名称:** \<symbol\>@avgPrice

**更新速度:** 1000ms

**Payload:**

```javascript
{
  "e": "avgPrice",          // Event type
  "E": 1693907033000,       // Event time
  "s": "BTCUSDT",           // Symbol
  "i": "5m",                // Average price interval
  "w": "25776.86000000",    // Average price
  "T": 1693907032213        // Last trade time
}
```

<a id="depth"></a>

## 有限档深度信息
每秒推送有限档深度信息。levels 表示几档买卖单信息, 可选 5/10/20档

**Stream 名称:** \<symbol\>@depth\<levels\> 或者 \<symbol\>@depth\<levels\>@100ms

**更新速度:** 1000ms 或者 100ms

**Payload:**
```javascript
{
  "lastUpdateId": 160,  // 末次更新ID
  "bids": [             // 买单
    [
      "0.0024",         // 价
      "10",             // 量
      []                // 忽略
    ]
  ],
  "asks": [             // 卖单
    [
      "0.0026",         // 价
      "100",            // 量
      []                // 忽略
    ]
  ]
}
```
<a id="diff-depth"></a>

## 增量深度信息stream
每秒推送orderbook的变化部分（如果有）

**Stream 名称:** \<symbol\>@depth 或者 \<symbol\>@depth@100ms

**更新速度:** 1000ms 或者 100ms

**Payload:**
```javascript
{
  "e": "depthUpdate", // 事件类型
  "E": 1672515782136, // 事件时间
  "s": "BNBBTC",      // 交易对
  "U": 157,           // 从上次推送至今新增的第一个 update Id
  "u": 160,           // 从上次推送至今新增的最后一个 update Id
  "b": [              // 变动的买单深度
    [
      "0.0024",       // 价
      "10",           // 量
      []              // Ignore
    ]
  ],
  "a": [              // 变动的卖单深度
    [
      "0.0026",       // 价
      "100",          // 量
      []              // Ignore
    ]
  ]
}
```

<a id="rolling-window-ticker"></a>
## 按Symbol的滚动窗口统计

单个symbol的滚动窗口统计, 支持多个时间窗口。

**Stream 名称:** \<symbol\>@ticker_\<window_size\>

**Window Sizes:** 1h, 4h, 1d

**更新速度:** 1000ms

*注意*: <br/>
- 该数据流和 \<symbol\>@ticker 不一样。
- `O` (`open time`) 会在每分钟整点开始, 而 `C` (`closing time`)是当前更新时间。
- 实际统计的时间范围会比\<window_size\>多不超过59999ms。

**Payload:**
```javascript
{
  "e": "1hTicker",    // Event type
  "E": 1672515782136, // Event time
  "s": "BNBBTC",      // Symbol
  "p": "0.0015",      // Price change
  "P": "250.00",      // Price change percent
  "o": "0.0010",      // Open price
  "h": "0.0025",      // High price
  "l": "0.0010",      // Low price
  "c": "0.0025",      // Last price
  "w": "0.0018",      // Weighted average price
  "v": "10000",       // Total traded base asset volume
  "q": "18",          // Total traded quote asset volume
  "O": 0,             // Statistics open time
  "C": 86400000,      // Statistics close time
  "F": 0,             // First trade ID
  "L": 18150,         // Last trade Id
  "n": 18151          // Total number of trades
}
```
<a id="all-market-rolling-window-ticker"></a>
## 全市场滚动窗口统计

全市场symbols的滚动窗口ticker统计，计算于多个窗口。<br/>

注意：有变动的ticker才会推送。

**Stream 名称:** !ticker_\<window-size\>@arr

**Window Size:** 1h, 4h, 1d

**更新速度:** 1000ms

> **Payload:**
```javascript
[
  {
    // 同 <symbol>@ticker_<window-size> payload,
    // 间隔内更新的每个symbol。
  }
]
```

<a id="how-to-maintain-orderbook"></a>

## 如何正确在本地维护一个order book副本
1. 打开与 `wss://stream.binance.com:9443/ws/bnbbtc@depth` 的 WebSocket 连接。
2. 开始缓存收到的event。请记录您收到的第一个event的 `U`值。
3. 访问 `https://api.binance.com/api/v3/depth?symbol=BNBBTC&limit=5000` 获取深度快照。
4. 如果快照中的 `lastUpdateId` 小于等于步骤 2 中的 `U` 值，请返回步骤 3。
5. 在收到的event中，丢弃快照中 `u` <= `lastUpdateId` 的所有event。现在第一个event的 `lastUpdateId` 应该在 `[U;u]` 范围以内。
6. 将本地order book设置为快照。它的更新ID 为 `lastUpdateId`。
7. 更新所有缓存的event，以及后续的所有event。

要将event应用于您的本地order book，请遵循以下更新过程：
1. 如果event `u` (最后一次更新 ID) < 您本地order book的更新 ID，请忽略该事件。
2. 如果event `U` (第一次更新 ID) > 您本地order book的更新 ID，则说明出现问题。请丢弃您的本地order book并从头开始开始重建。
3. 对于买价 (`b`) 和卖价 (`a`) 中的每个价位，在order book中设置新的数量：
    * 如果order book中不存在价位，则插入新的数量。
    * 如果数量为零，则从order book中删除此价位。
4. 将order book更新 ID 设置为处理过event中的最后一次更新 ID (`u`)。

> [!NOTE] 
> 由于从 API 检索的深度快照对价位的数量有限制（每侧最多 5000 个），因此除非它们发生变化，否则您将无法了解初始快照之外的价位数量。<br> 
> 因此，在使用这些级别的信息时要小心，因为它们可能无法反映订单簿的完整视图。<br> 
> 但是，对于大多数场景，可以每侧看到 5000 个价位就足以了解市场并进行有效交易。
