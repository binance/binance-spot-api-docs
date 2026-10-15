<a id="general-api-information"></a>

<a id="websocket-api"></a>

<a id="websocket-api-基本信息"></a>

<a id="public-websocket-api-for-binance"></a>

<a id="general-websocket-api-information"></a>

# Binance 的公共 WebSocket API

## API 基本信息

- 本篇所列出的 wss 接口的 base URL：**`wss://ws-api.binance.com:443/ws-api/v3`**
  - 如果使用标准443端口时遇到问题，可以使用替代端口9443。
  - [现货测试网](https://testnet.binance.vision)的 base URL 是
    `wss://ws-api.testnet.binance.vision/ws-api/v3`。
- 每个到 base URL 的链接有效期不超过24小时，请妥善处理断线重连。
- 当服务器即将关闭时，系统将发送 [`serverShutdown`](#serverShutdown)
  事件，并导致连接断开。请尽快建立新连接，以防止中断。
- 我们支持 HMAC，RSA 以及 Ed25519 Key 类型。 如需进一步了解，请参考
  [API Key 类型](faqs/api_key_types_CN.md)。
- 响应默认为 JSON 格式。如果您想接收 SBE 格式的响应，请参考
  [简单二进制编码 （SBE） 常见问题](./faqs/sbe_faq_CN.md)。
- 如果您的请求包含非 ASCII 字符的交易对名称，那么响应中可能包含以 UTF-8 编码的非 ASCII 字符。
- 即使请求本身不包含非 ASCII 字符，某些方法也可能会返回包含以 UTF-8 编码的非 ASCII 字符的资产和/或交易对名称。
- WebSocket 服务器**每20秒**发送 PING 消息。
  - 如果websocket 服务器没有在一分钟之内收到PONG 消息应答，连接会被断开。
  - 当客户收到PING消息，必须尽快回复PONG消息，同时payload需要和PING消息一致。
  - 服务器允许未经请求的PONG消息，但这不会保证连接不断开。**对于这些PONG 消息，建议payload为空。**
- 除非另有说明，否则数据将按**时间顺序**返回。
  - 如果未指定 `startTime` 或 `endTime`，则返回最近的条目，直至达到限制值。
  - 如果指定 `startTime`，则返回从 `startTime` 到限制值为止最老的条目。
  - 如果指定 `endTime`，则返回截至 `endTime` 和限制值为止最近的条目。
  - 如果同时指定 `startTime` 和 `endTime`，则行为类似于 `startTime`，但不超过 `endTime`。
- JSON 响应中的所有时间和时间戳相关字段均以**UTC 毫秒为默认单位**。要以微秒为单位接收信息，请在 URL 中添加参数
  `timeUnit=MICROSECOND` 或 `timeUnit=microsecond`。
- 时间戳参数（例如 `startTime`、`endTime`、`timestamp`）可以以毫秒或微秒为单位传递。
- 除非另有说明，所有字段名称和值都**大小写敏感**。
- 如需进一步了解枚举或术语，请参考 [现货交易API术语表](faqs/spot_glossary_CN.md) 页面。
- API 处理请求的超时时间为 10 秒。如果撮合引擎的响应时间超过此时间，API 将返回 “Timeout waiting for
  response from backend server. Send status unknown; execution status
  unknown.”。[(-1007 超时)](errors_CN.md#-1007-timeout)
  - 这并不总是意味着该请求在撮合引擎中失败。
  - 如果请求状态未显示在 [用户数据流](user-data-stream_CN.md) 中，请执行 API 查询以获取其状态。
- **请避免在请求中使用 SQL 关键字**，因为这可能会触发 Web 应用防火墙（WAF）规则导致安全拦截。详情请参见
  https://www.binance.com/zh-CN/support/faq/detail/360004492232

<a id="request-format"></a>

## 请求格式

请求必须在 **text 帧** 中以 JSON 格式发送，每帧一个请求。

请求示例:

```json
{
  "id": "e2a85d9f-07a5-4f94-8d5f-789dc3deb097",
  "method": "order.place",
  "params": {
    "symbol": "BTCUSDT",
    "side": "BUY",
    "type": "LIMIT",
    "price": "0.1",
    "quantity": "10",
    "timeInForce": "GTC",
    "timestamp": 1655716096498,
    "apiKey": "T59MTDLWlpRW16JVeZ2Nju5A5C98WkMm8CSzWC4oqynUlTm1zXOxyauT8LmwXEv9",
    "signature": "5942ad337e6779f2f4c62cd1c26dba71c91514400a24990a3e7f5edec9323f90"
  }
}
```

请求字段:

| 名称     | 类型                  | 是否必需 | 描述                           |
| -------- | --------------------- | -------- | ------------------------------ |
| `id`     | INT / STRING / `null` | YES      | 任意的 ID 用于匹配对请求的响应 |
| `method` | STRING                | YES      | 请求函数名称                   |
| `params` | OBJECT                | NO       | 请求参数。如果没有参数可以省略 |

- 请求 `id` 是任意的。可以使用 UUID、顺次 ID、当前时间戳等。服务器不会以任何方式解释
  `id`，只是在响应中回显它。

  可以在一个会话中自由重复使用 ID，不过请注意不要一次发送多个具有相同 ID 的请求，因为否则可能无法区分响应。

- 请求函数名称可以以显式版本为前缀，例如：`"v3/order.place"`

- `params` 的顺序不重要。

<a id="response-format"></a>

## 响应格式

响应在 **text 帧** 中以 JSON 格式返回，每帧一个响应。

成功响应示例:

```json
{
  "id": "e2a85d9f-07a5-4f94-8d5f-789dc3deb097",
  "status": 200,
  "result": {
    "symbol": "BTCUSDT",
    "orderId": 12510053279,
    "orderListId": -1,
    "clientOrderId": "a097fe6304b20a7e4fc436",
    "transactTime": 1655716096505,
    "price": "0.10000000",
    "origQty": "10.00000000",
    "executedQty": "0.00000000",
    "origQuoteOrderQty": "0.000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "workingTime": 1655716096505,
    "selfTradePreventionMode": "NONE"
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 12
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 4043
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 6000,
      "count": 321
    }
  ]
}
```

失败响应示例:

```json
{
  "id": "e2a85d9f-07a5-4f94-8d5f-789dc3deb097",
  "status": 400,
  "error": {
    "code": -2010,
    "msg": "Account has insufficient balance for requested action."
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 13
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 4044
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 6000,
      "count": 322
    }
  ]
}
```

响应字段:

<table>
<thead>
  <tr>
    <th>名称</th>
    <th>类型</th>
    <th>是否必需</th>
    <th>描述</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td><code>id</code></td>
    <td>INT / STRING / <code>null</code></td>
    <td>YES</td>
    <td>与原来请求的ID一样</td>
  </tr>
  <tr>
    <td><code>status</code></td>
    <td>INT</td>
    <td>YES</td>
    <td>响应状态。请看 <a href="#状态代码">状态代码</a></td>
  </tr>
  <tr>
    <td><code>result</code></td>
    <td>OBJECT / ARRAY</td>
    <td rowspan="2">YES</td>
    <td>响应内容。请求成功则显示</td>
  </tr>
  <tr>
    <td><code>error</code></td>
    <td>OBJECT</td>
    <td>错误描述。请求失败则显示</td>
  </tr>
  <tr>
    <td><code>rateLimits</code></td>
    <td>ARRAY</td>
    <td>NO</td>
    <td>速率限制状态。请看 <a href="#ratelimits">速率限制</a></td>
  </tr>
</tbody>
</table>

<a id="status-codes"></a>

### 状态代码

`status` 字段的状态代码与HTTP的状态代码相同。

一些常见状态代码：

- `200` 代码指示成功响应。
- `4XX` 错误码用于指示错误的请求内容、行为、格式。问题在于请求者。
  - `400` – 错误码表示请求失败，请参阅 `error` 了解原因。
  - `403`
    – 错误码表示违反 WAF 限制(Web应用程序防火墙)。这可能表示触发了速率限制或安全拦截。详情请参见
    https://www.binance.com/zh-CN/support/faq/detail/360004492232 。
  - `409` – 错误码表示请求有一部分成功，一部分失败。请参阅 `error` 了解更多详细
  - `418` – 表示收到 429 后继续访问，于是被封了。
  - `429` – 错误码表示警告访问频次超限，即将被封IP。
- `5XX` 错误码用于指示Binance服务侧的问题。
  - **重要**：如果响应包含 5xx 状态码，**并不**一定意思请求失败。执行状态为
    _unknown_，请求可能实际成功。请使用 query 函数确认状态。建议建立一个新 WebSocket 连接用于确认状态。

有关错误代码和消息的列表，请参阅 [Binance 的错误代码](errors_CN.md)。

<a id="conditional-fields-in-order-responses"></a>

## 订单响应中的特定条件时才会出现的字段

订单响应中的有一些字段仅在满足特定条件时才会出现。这些订单响应可以来自下订单，查询订单或取消订单，并且可以包括订单列表类型。下面列出了这些字段：

| 名称                | 描述                                                                  | 显示的条件                                                                        | 示例                                       |
| ------------------- | --------------------------------------------------------------------- | --------------------------------------------------------------------------------- | ------------------------------------------ |
| `icebergQty`        | 冰山订单的数量。                                                      | 只有在请求中发送 `icebergQty` 参数时才会出现。                                    | `"icebergQty": "0.00000000"`               |
| `preventedMatchId`  | 与 `symbol` 结合使用时，可用于查询因为 STP 导致订单失效的过期订单。   | 只有在因为 STP 导致订单失效时可见。                                               | `"preventedMatchId": 0`                    |
| `preventedQuantity` | 因为 STP 导致订单失效的数量。                                         | 只有在因为 STP 导致订单失效时可见。                                               | `"preventedQuantity": "1.200000"`          |
| `stopPrice`         | 用于设置逻辑订单中的触发价。                                          | `STOP_LOSS`，`TAKE_PROFIT`，`STOP_LOSS_LIMIT` 和 `TAKE_PROFIT_LIMIT` 订单时可见。 | `"stopPrice": "23500.00000000"`            |
| `strategyId`        | 策略单ID; 用以关联此订单对应的交易策略。                              | 如果在请求中添加了参数，则会出现。                                                | `"strategyId": 37463720`                   |
| `strategyType`      | 策略单类型; 用以显示此订单对应的交易策略。                            | 如果在请求中添加了参数，则会出现。                                                | `"strategyType": 1000000`                  |
| `trailingDelta`     | 用以定义追踪止盈止损订单被触发的价格差。                              | 出现在追踪止损订单中。                                                            | `"trailingDelta": 10`                      |
| `trailingTime`      | 追踪单被激活和跟踪价格变化的时间。                                    | 出现在追踪止损订单中。                                                            | `"trailingTime": -1`                       |
| `usedSor`           | 用于确定订单是否使用SOR的字段                                         | 在使用SOR下单时出现                                                               | `"usedSor": true`                          |
| `workingFloor`      | 用以定义订单是通过 SOR 还是由订单提交到的订单簿（order book）成交的。 | 出现在使用了 SOR 的订单中。                                                       | `"workingFloor": "SOR"`                    |
| `pegPriceType`      | 挂钩价格类型                                                          | 仅用于挂钩订单                                                                    | `"pegPriceType": "PRIMARY_PEG"`            |
| `pegOffsetType`     | 挂钩价格偏移类型                                                      | 如若需要，仅用于挂钩订单                                                          | `"pegOffsetType": "PRICE_LEVEL"`           |
| `pegOffsetValue`    | 挂钩价格偏移值                                                        | 如若需要，仅用于挂钩订单                                                          | `"pegOffsetValue": 5`                      |
| `peggedPrice`       | 订单对应的当前挂钩价格                                                | 一旦确定，仅用于挂钩订单                                                          | `"peggedPrice": "87523.83710000"`          |
| `expiryReason`      | 订单失效原因                                                          | 当订单失效时                                                                      | `"expiryReason": "INSUFFICIENT_LIQUIDITY"` |

<a id="event-format"></a>

## 事件格式

[用户数据流](user-data-stream_CN.md)中的非 SBE 会话事件以 JSON 格式在 **text 帧**
中发送，每帧一个事件。

[SBE 会话](faqs/sbe_faq_CN.md)中的事件将作为 **二进制帧** 发送。

有关如何在 WebSocket API 中订阅用户数据流的详细信息，请参阅
[`订阅用户数据流`](web-socket-api_CN.md#subscribe-to-user-data-stream-user_stream)。

事件示例:

```javascript
{
    "subscriptionId": 0,
    "event": {
        "e": "outboundAccountPosition",
        "E": 1728972148778,
        "u": 1728972148778,
        "B": [
            {
                "a": "BTC",
                "f": "11818.00000000",
                "l": "182.00000000"
            },
            {
                "a": "USDT",
                "f": "10580.00000000",
                "l": "70.00000000"
            }
        ]
    }
}
```

事件字段:

| 名称             | 类型   | 是否必须 | 描述                                                                                        |
| ---------------- | ------ | -------- | ------------------------------------------------------------------------------------------- |
| `event`          | OBJECT | YES      | 事件 payload。请看 [用户数据流](user-data-stream_CN.md)                                        |
| `subscriptionId` | INT    | NO       | 用以标识事件来自于哪个订阅。详见 [用户数据流订阅](user-data-stream_CN.md#general-information) |

<a id="connectionevents"></a>

<a id="connection-events"></a>

## 连接事件

<a id="serverShutdown"></a>

<a id="server-shutdown"></a>

### 服务器关闭

当服务器即将关闭时，会发送 `serverShutdown` 事件。

```javascript
{
  "event": {
    "e": "serverShutdown", // 事件类型
    "E": 1770123456789     // 事件时间
   }
}
```

请尽快建立新连接，以防中断

<a id="ratelimits"></a>

<a id="rate-limits"></a>

## 速率限制

<a id="connection-limits"></a>

### 连接数量限制

每IP地址、每5分钟最多可以发送300次连接请求。

<a id="general-information-on-rate-limits"></a>

### 速率限制基本信息

- [`exchangeInfo`](web-socket-api_CN.md#exchange-information)
  有包含与速率限制相关的信息。
- 根据不同的间隔，有多种频率限制类型。
- 从响应中的可选 `rateLimits` 字段，能看到当前的频率限制状态。
- 当您超出未成交订单计数或者请求速率限制时，请求会失败并返回 HTTP 状态代码 429。

<a id="how-to-interpret-rate-limits"></a>

#### 如何咨询频率限制

频率限制状态的响应可能如下所示：

```json
{
  "id": "7069b743-f477-4ae3-81db-db9b8df085d2",
  "status": 200,
  "result": {
    "serverTime": 1656400526260
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 6000,
      "count": 70
    }
  ]
}
```

`rate Limits` 数组描述了受请求影响的所有当前的速率限制。

| 名称            | 类型 | 是否必须 | 描述                                            |
| --------------- | ---- | -------- | ----------------------------------------------- |
| `rateLimitType` | ENUM | YES      | 频率限制类型: `REQUEST_WEIGHT`, `ORDERS`        |
| `interval`      | ENUM | YES      | 频率限制间隔: `SECOND`, `MINUTE`, `HOUR`, `DAY` |
| `intervalNum`   | INT  | YES      | 频率限制间隔乘数                                |
| `limit`         | INT  | YES      | 每个间隔的请求限制                              |
| `count`         | INT  | YES      | 每个间隔的当前使用情况                          |

频率限制按间隔计算。

例如，`1 MINUTE`
间隔表示每分钟开始。在 00:01:23.456 提交的请求计入 00:01:00 分钟的限制。一旦 00:02:00 分钟开始，计数将再次重置为零。

其他间隔的行为方式类似。例如，`1 DAY` 频率限制是在每天 00:00 UTC 重置，并且 `10 SECOND`
间隔重置为每分钟的 00、10、20...秒。

API 有多种频率限制间隔。如果您用完了较短的间隔但较长的间隔仍然允许请求，您将不得不等待较短的间隔到期并重置。如果你用完了更长的间隔，你将不得不等待那个间隔重置，即使较短的频率限制计数为零。

<a id="how-to-showhide-rate-limit-information"></a>

#### 如何显示/隐藏频率限制信息

默认情况下，每个响应都包含 `rateLimits` 字段。

但是，频率限制信息可能非常大。如果您对每个请求的详细频率限制状态不感兴趣，可以从响应中省略
`rateLimits` 字段。

- 请求中的可选 `returnRateLimits` boolean 参数。

  使用 `returnRateLimits` 参数控制是否包含 `rateLimits` 字段以响应单个请求。

  默认请求和响应：

  ```json
  { "id": 1, "method": "time" }
  ```

  ```json
  {
    "id": 1,
    "status": 200,
    "result": { "serverTime": 1656400526260 },
    "rateLimits": [
      {
        "rateLimitType": "REQUEST_WEIGHT",
        "interval": "MINUTE",
        "intervalNum": 1,
        "limit": 6000,
        "count": 70
      }
    ]
  }
  ```

  没有频率限制状态的请求和响应：

  ```json
  { "id": 2, "method": "time", "params": { "returnRateLimits": false } }
  ```

  ```json
  { "id": 2, "status": 200, "result": { "serverTime": 1656400527891 } }
  ```

- 连接 URL 中可选的 `returnRateLimits` boolean 参数。

  如果您希望在默认情况下从所有响应中省略 `rateLimits`，可以在 query string 中使用 `returnRateLimits`
  参数：

  ```
  wss://ws-api.binance.com:443/ws-api/v3?returnRateLimits=false
  ```

  这将使通过此连接发出的所有请求的行为就像您已传了 `"returnRateLimits"：false` 一样。

  如果您*想*查看特定请求的频率限制，您需要特定传 `"returnRateLimits"：true` 参数。

**注意:** 如果您在响应中隐藏 `rateLimits` 字段，您的请求仍然还是会受到频率限制的。

<a id="ip-limits"></a>

### IP 访问限制

- 每个请求都有一个特定的 **权重**，它会添加到您的访问限制中。
  - 越消耗资源的接口, 比如查询多个交易对, 权重就会越大。
  - 连接到 WebSocket API 会用到2个权重。
- 当前权重使用由 `REQUEST_WEIGHT` 频率限制类型指示。
- 请使用[`exchangeInfo`](web-socket-api_CN.md#exchange-information)请求来跟踪当前的重量限制。
- 权重是基于**每个 IP 地址**累积的，并由来自该地址的所有连接共享。
- 如果超多限制，客服端会收到 `429`。
  - 这错误代码表示您有责任停止发送请求，不得滥用API。
  - 响应会包含一个 `retryAfter` 字段，指示在什么时候您能重试。
- **屡次违反速率限制或者在收到429后未能退缩将导致自动 IP 封禁和断开连接。**
  - 被禁止 IP 地址的请求失败，状态为 `418`。
  - `retryAfter` 字段表示解除禁令的timestamp。
- 频繁违反限制的封禁时间会**逐渐延长**，**从最短2分钟到最长3天**。

表示在1分钟内使用了（1200权重限制中的）70权重的成功响应：

```json
{
  "id": "7069b743-f477-4ae3-81db-db9b8df085d2",
  "status": 200,
  "result": [],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 6000,
      "count": 70
    }
  ]
}
```

表示已被封禁且封禁将在 epoch `1659146400000` 解锁的失败响应：

```json
{
  "id": "fc93a61a-a192-4cf4-bb2a-a8f0f0c51e06",
  "status": 418,
  "error": {
    "code": -1003,
    "msg": "Way too much request weight used; IP banned until 1659146400000. Please use WebSocket Streams for live updates to avoid bans.",
    "data": {
      "serverTime": 1659142907531,
      "retryAfter": 1659146400000
    }
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 6000,
      "count": 2411
    }
  ]
}
```

<a id="unfilled-order-count"></a>

### 未成交订单计数

- 成功下单将更新 `订单` 速率限制类型。
- 被拒绝或不成功的订单可能会也可能不会更新 `订单` 速率限制类型。
- **请注意，如果您的订单一直顺利完成交易，您可以通过 API 持续下订单**。更多信息，请参见
  [现货未成交订单计数规则](./faqs/order_count_decrement_CN.md)。
- 使用
  [`account.rateLimits.orders`](web-socket-api_CN.md#unfilled-order-count-user_data)
  请求来跟踪您在此时间间隔内下了多少订单。
- 如果超过此值，请求将失败，状态为 `429`。
  - 此状态代码表示您应退出并停止向 API 滥发信息。
  - 状态为 `429` 的响应会包含 `retryAfter` 字段，用以指示何时可以重试请求。
- 这是按 **每一个账户** 维护的，并由该账户的所有 API 密钥共享。

表示在10秒内下了12个订单和在24小时内下了4043个订单的成功响应：

```json
{
  "id": "e2a85d9f-07a5-4f94-8d5f-789dc3deb097",
  "status": 200,
  "result": {
    "symbol": "BTCUSDT",
    "orderId": 12510053279,
    "orderListId": -1,
    "clientOrderId": "a097fe6304b20a7e4fc436",
    "transactTime": 1655716096505,
    "price": "0.10000000",
    "origQty": "10.00000000",
    "executedQty": "0.00000000",
    "origQuoteOrderQty": "0.000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "workingTime": 1655716096505,
    "selfTradePreventionMode": "NONE"
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 12
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 4043
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 6000,
      "count": 321
    }
  ]
}
```

<a id="request-security"></a>

## 请求鉴权类型

- 每个方法都有一个鉴权类型，指示所需的 API 密钥权限，显示在方法名称旁边（例如，[下新订单 (TRADE)](web-socket-api_CN.md#place-new-order-trade)）。
- 如果未指定，则鉴权类型为 `NONE`。
- 除了为 `NONE` 外，所有具有鉴权类型的方法均视为 `SIGNED` 请求（即包含
  `signature`），[listenKey 管理](user-data-stream_CN.md) 除外。
- 具有鉴权类型的方法需要提供有效的 API 密钥并验证通过。
  - API 密钥可在您的 Binance 账户的 [API 管理](https://www.binance.com/en/support/faq/360002502072)
    页面创建。
  - **API 密钥和密钥对均为敏感信息，切勿与他人分享。**
    如果发现账户有异常活动，请立即撤销所有密钥并联系 Binance 支持。
- API 密钥可配置为仅允许访问某些鉴权类型。
  - 例如，您可以拥有具有 `TRADE` 权限的 API 密钥用于交易，同时使用具有 `USER_DATA`
    权限的另一个 API 密钥来监控订单状态。
  - 默认情况下，API 密钥无法进行 `TRADE`，您需要先在 API 管理中启用交易权限。

| 鉴权类型      | 描述                                 |
| ------------- | ------------------------------------ |
| `NONE`        | 公开市场数据                         |
| `TRADE`       | 在交易所交易，下单和取消订单         |
| `USER_DATA`   | 私人账户信息，例如订单状态和交易历史 |
| `USER_STREAM` | 管理用户数据流订阅                   |

<a id="signed-request-security"></a>

### 需要签名的请求

- 为了授权请求，`SIGNED` 请求必须带 `signature` 参数。

<a id="signature-case-sensitivity"></a>

#### 签名区分大小写

- **HMAC：**
  使用 HMAC 生成的签名**不区分大小写**。这意味着无论字母大小写如何，签名字符串都可以被验证。
- **RSA：** 使用 RSA 生成的签名**区分大小写**。
- **Ed25519：** 使用 Ed25519 生成的签名也**区分大小写**。

<a id="timingsecurity"></a>

<a id="timing-security"></a>

### 时间同步安全

- `SIGNED` 请求还需要一个 `timestamp` 参数，该参数应为当前时间戳，单位为毫秒或微秒。（参见
  [通用 API 信息](#general-api-information)）
- 另一个可选参数 `recvWindow`，用以指定请求的有效期，只能以毫秒为单位。
  - `recvWindow` 扩展为三位小数（例如 6000.346），以便可以指定微秒。
  - 如果未发送 `recvWindow`，则 **默认为 5000 毫秒**。
  - `recvWindow` 的最大值为 60000 毫秒。
- 请求处理逻辑如下：

```javascript
serverTime = getCurrentTime()
if (timestamp < (serverTime + 1 second) && (serverTime - timestamp) <= recvWindow) {
  // 开始处理请求
  serverTime = getCurrentTime()
  if (serverTime - timestamp) <= recvWindow {
    // 将请求转发到撮合引擎
  } else {
    // 拒绝请求
  }
  // 结束处理请求
} else {
  // 拒绝请求
}
```

**关于交易时效性**
互联网状况并不完全稳定可靠，因此你的程序本地到币安服务器的时延会有抖动, 这是我们设置 `recvWindow`
的目所在。如果你从事高频交易，对交易时效性有较高的要求，可以灵活设置 `recvWindow` 以达到你的要求。

**建议使用5000毫秒以下的 `recvWindow`！**

<a id="hmac"></a>

<a id="signed-request-example-hmac"></a>

### `SIGNED` 请求示例 (HMAC)

这是有关如何用一个 HMAC secret key 签署请求的分步指南。

示例 API key 和 secret key：

| Key         | Value                                                              |
| ----------- | ------------------------------------------------------------------ |
| `apiKey`    | `vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A` |
| `secretKey` | `NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j` |

**警告：请勿与任何人分享您的 API 密钥和秘钥。**

此处提供的示例密钥仅用于示范说明目的。

交易对名称完全由 ASCII 字符组成的请求示例：

请求示例：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol": "BTCUSDT",
    "side": "SELL",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "0.01000000",
    "price": "52000.00",
    "newOrderRespType": "ACK",
    "recvWindow": 100,
    "timestamp": 1645423376532,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "------ 未填写 ------"
  }
}
```

交易对名称包含非 ASCII 字符的请求示例：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol": "１２３４５６",
    "side": "BUY",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "0.01000000",
    "price": "0.10000000",
    "recvWindow": 5000,
    "timestamp": 1645423376532,
    "apiKey": "4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO",
    "signature": "------ FILL ME ------"
  }
}
```

**第一步：创建签名内容**

除了 `signature` 之外，获取所有请求 `params`，然后**按参数名称的字母顺序对它们进行排序**：

对于第一组示例参数（仅限 ASCII 字符）：

| 参数          | 取值                                                             |
| ------------- | ---------------------------------------------------------------- |
| `apiKey`      | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A |
| `price`       | 52000.00                                                         |
| `quantity`    | 0.01000000                                                       |
| `recvWindow`  | 100                                                              |
| `side`        | SELL                                                             |
| `symbol`      | BTCUSDT                                                          |
| `timeInForce` | GTC                                                              |
| `timestamp`   | 1645423376532                                                    |
| `type`        | LIMIT                                                            |

对于第二组示例参数（包含一些 Unicode 字符）：

| 参数          | 取值                                                             |
| ------------- | ---------------------------------------------------------------- |
| `apiKey`      | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A |
| `price`       | 0.10000000                                                       |
| `quantity`    | 1.00000000                                                       |
| `recvWindow`  | 5000                                                             |
| `side`        | BUY                                                              |
| `symbol`      | １２３４５６                                                     |
| `timeInForce` | GTC                                                              |
| `timestamp`   | 1645423376532                                                    |
| `type`        | LIMIT                                                            |

将参数格式化为 `参数=取值` 对并用 `&` 分隔每个参数对。数值需要采用 UTF-8 编码。

对于第一组示例参数（仅限 ASCII 字符），签名有效负载将如下所示：

```console
apiKey=vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT
```

对于第二组示例参数（包含一些 Unicode 字符），签名有效负载将如下所示：

```console
apiKey=vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A&price=0.10000000&quantity=1.00000000&recvWindow=5000&side=BUY&symbol=１２３４５６&timeInForce=GTC&timestamp=1645423376532&type=LIMIT
```

**第二步：计算签名**

1. 使用 API 密钥中的 `secretKey` 作为 HMAC-SHA-256 算法的签名密钥。
2. 对步骤 1 中构建的签名 payload 进行签名。
3. 将 HMAC-SHA-256 的输出编码为十六进制字符串。

请注意，`apiKey`、`secretKey` 和有效负载是**大小写敏感的**。而生成的签名值是不区分大小写的。

可以使用 OpenSSL 交叉检查您的签名算法实现：

对于第一组示例参数（仅限 ASCII 字符）：

```console
$ echo -n 'apiKey=vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT' \
  | openssl dgst -hex -sha256 -hmac 'NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j'

aa1b5712c094bc4e57c05a1a5c1fd8d88dcd628338ea863fec7b88e59fe2db24
```

对于第二组示例参数（包含一些 Unicode 字符）：

```console
$ echo -n 'apiKey=vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A&price=0.10000000&quantity=1.00000000&recvWindow=5000&side=BUY&symbol=１２３４５６&timeInForce=GTC&timestamp=1645423376532&type=LIMIT' \
  | openssl dgst -hex -sha256 -hmac 'NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j'

b33892ae8e687c939f4468c6268ddd4c40ac1af18ad19a064864c47bae0752cd
```

**第三步：添加 `signature` 到 `params` 中**

通过在对请求中添加 `signature` 参数和签名字串来完成请求。

对于第一组示例参数（仅限 ASCII 字符）：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol": "BTCUSDT",
    "side": "SELL",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "0.01000000",
    "price": "52000.00",
    "recvWindow": 100,
    "timestamp": 1645423376532,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "aa1b5712c094bc4e57c05a1a5c1fd8d88dcd628338ea863fec7b88e59fe2db24"
  }
}
```

对于第二组示例参数（包含一些 Unicode 字符）：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol": "１２３４５６",
    "side": "BUY",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "1.00000000",
    "price": "0.10000000",
    "recvWindow": 5000,
    "timestamp": 1645423376532,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "b33892ae8e687c939f4468c6268ddd4c40ac1af18ad19a064864c47bae0752cd"
  }
}
```

<a id="rsa"></a>

<a id="signed-request-example-rsa"></a>

### `SIGNED` 请求示例 (RSA)

这是有关如何用一个 RSA private key 签署请求的分步指南。

| Key      | Value                                                              |
| -------- | ------------------------------------------------------------------ |
| `apiKey` | `CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ` |

这些示例假设私钥存储在文件 `test-prv-key.pem` 中。

**警告：请勿与任何人分享您的 API 密钥和私钥。**

此处提供的示例密钥仅用于示范说明目的。

交易对名称完全由 ASCII 字符组成的请求示例：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol": "BTCUSDT",
    "side": "SELL",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "0.01000000",
    "price": "52000.00",
    "recvWindow": 100,
    "timestamp": 1645423376532,
    "apiKey": "CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ",
    "signature": "------ FILL ME ------"
  }
}
```

交易对名称包含非 ASCII 字符的请求示例：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol": "１２３４５６",
    "side": "BUY",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "0.01000000",
    "price": "0.10000000",
    "recvWindow": 5000,
    "timestamp": 1645423376532,
    "apiKey": "CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ",
    "signature": "------ FILL ME ------"
  }
}
```

**第一步：创建签名内容**

除了 `signature`，获取请求的所有其它 `params`，然后**按参数名称的字母顺序对它们进行排序**：

对于第一组示例参数（仅限 ASCII 字符）：

| 参数          | 取值                                                             |
| ------------- | ---------------------------------------------------------------- |
| `apiKey`      | CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ |
| `price`       | 52000.00                                                         |
| `quantity`    | 0.01000000                                                       |
| `recvWindow`  | 100                                                              |
| `side`        | SELL                                                             |
| `symbol`      | BTCUSDT                                                          |
| `timeInForce` | GTC                                                              |
| `timestamp`   | 1645423376532                                                    |
| `type`        | LIMIT                                                            |

对于第二组示例参数（包含一些 Unicode 字符）：

| 参数          | 取值                                                             |
| ------------- | ---------------------------------------------------------------- |
| `apiKey`      | CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ |
| `price`       | 0.10000000                                                       |
| `quantity`    | 1.00000000                                                       |
| `recvWindow`  | 5000                                                             |
| `side`        | BUY                                                              |
| `symbol`      | １２３４５６                                                     |
| `timeInForce` | GTC                                                              |
| `timestamp`   | 1645423376532                                                    |
| `type`        | LIMIT                                                            |

将参数格式化为 `参数=取值` 对并用 `&` 分隔每个参数对。数值需要采用 UTF-8 编码。

对于第一组示例参数（仅限 ASCII 字符），签名有效负载将如下所示：

```console
apiKey=CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT
```

对于第二组示例参数（包含一些 Unicode 字符），签名有效负载将如下所示：

```console
apiKey=CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ&price=0.10000000&quantity=1.00000000&recvWindow=5000&side=BUY&symbol=１２３４５６&timeInForce=GTC&timestamp=1645423376532&type=LIMIT
```

**第二步：计算签名**

1. 使用 RSASSA-PKCS1-v1_5 算法和 SHA-256 哈希函数对步骤 1 中构造的签名有效载荷的 UTF-8 字节进行签名。
2. 将输出编码为 base64。

请注意，`apiKey`, payload 和生成的`签名值`都是**大小写敏感**的。

您可以使用 OpenSSL 交叉检查您的签名算法：

对于第一组示例参数（仅限 ASCII 字符）：

```console
$ echo -n 'apiKey=CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT' \
  | openssl dgst -sha256 -sign test-rsa-prv.pem \
  | openssl enc -base64 -A

OJJaf8C/3VGrU4ATTR4GiUDqL2FboSE1Qw7UnnoYNfXTXHubIl1iaePGuGyfct4NPu5oVEZCH4Q6ZStfB1w4ssgu0uiB/Bg+fBrRFfVgVaLKBdYHMvT+ljUJzqVaeoThG9oXlduiw8PbS9U8DYAbDvWN3jqZLo4Z2YJbyovyDAvDTr/oC0+vssLqP7NmlNb3fF3Bj7StmOwJvQJTbRAtzxK5PP7OQe+0mbW+D7RqVkUiSswR8qJFWTeSe4nXXNIdZdueYhF/Xf25L+KitJS5IHdIHcKfEw3MQzHFb2ZsGWkjDQwxkwr7Noi0Zaa+gFtxCuatGFm9dFIyx217pmSHtA==
```

对于第二组示例参数（包含一些 Unicode 字符）：

```console
$ echo -n 'apiKey=CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ&price=0.10000000&quantity=1.00000000&recvWindow=5000&side=BUY&symbol=１２３４５６&timeInForce=GTC&timestamp=1645423376532&type=LIMIT' \
  | openssl dgst -sha256 -sign test-rsa-prv.pem \
  | openssl enc -base64 -A

F3o/79Ttvl2cVYGPfBOF3oEOcm5QcYmTYWpdVIrKve5u+8paMNDAdUE+teqMxFM9HcquetGcfuFpLYtsQames5bDx/tskGM76TWW8HaM+6tuSYBSFLrKqChfA9hQGLYGjAiflf1YBnDhY+7vNbJFusUborNOloOj+ufzP5q42PvI3H0uNy3W5V3pyfXpDGCBtfCYYr9NAqA4d+AQfyllL/zkO9h9JSdozN49t0/hWGoD2dWgSO0Je6MytKEvD4DQXGeqNlBTB6tUXcWnRW+FcaKZ4KYqnxCtb1u8rFXUYgFykr2CbcJLSmw6ydEJ3EZ/NaZopRr+cU0W2m0HZ3qucw==
```

**第三步：在请求的 `params` 参数添加签名值**

通过在对请求中添加 `signature` 参数和签名字串来完成请求。

对于第一组示例参数（仅限 ASCII 字符）：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol": "BTCUSDT",
    "side": "SELL",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "0.01000000",
    "price": "52000.00",
    "newOrderRespType": "ACK",
    "recvWindow": 100,
    "timestamp": 1645423376532,
    "apiKey": "CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ",
    "signature": "OJJaf8C/3VGrU4ATTR4GiUDqL2FboSE1Qw7UnnoYNfXTXHubIl1iaePGuGyfct4NPu5oVEZCH4Q6ZStfB1w4ssgu0uiB/Bg+fBrRFfVgVaLKBdYHMvT+ljUJzqVaeoThG9oXlduiw8PbS9U8DYAbDvWN3jqZLo4Z2YJbyovyDAvDTr/oC0+vssLqP7NmlNb3fF3Bj7StmOwJvQJTbRAtzxK5PP7OQe+0mbW+D7RqVkUiSswR8qJFWTeSe4nXXNIdZdueYhF/Xf25L+KitJS5IHdIHcKfEw3MQzHFb2ZsGWkjDQwxkwr7Noi0Zaa+gFtxCuatGFm9dFIyx217pmSHtA=="
  }
}
```

对于第二组示例参数（包含一些 Unicode 字符）：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol": "１２３４５６",
    "side": "SELL",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "1.00000000",
    "price": "0.10000000",
    "recvWindow": 5000,
    "timestamp": 1645423376532,
    "apiKey": "CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ",
    "signature": "F3o/79Ttvl2cVYGPfBOF3oEOcm5QcYmTYWpdVIrKve5u+8paMNDAdUE+teqMxFM9HcquetGcfuFpLYtsQames5bDx/tskGM76TWW8HaM+6tuSYBSFLrKqChfA9hQGLYGjAiflf1YBnDhY+7vNbJFusUborNOloOj+ufzP5q42PvI3H0uNy3W5V3pyfXpDGCBtfCYYr9NAqA4d+AQfyllL/zkO9h9JSdozN49t0/hWGoD2dWgSO0Je6MytKEvD4DQXGeqNlBTB6tUXcWnRW+FcaKZ4KYqnxCtb1u8rFXUYgFykr2CbcJLSmw6ydEJ3EZ/NaZopRr+cU0W2m0HZ3qucw=="
  }
}
```

<a id="ed25519"></a>

<a id="signed-request-example-ed25519"></a>

### `SIGNED` 请求示例 (Ed25519)

**我们强烈建议使用 Ed25519 API keys，因为它在所有受支持的 API key 类型中提供最佳性能和安全性。**

这是有关如何用一个 Ed25519 private key 签署请求的分步指南。

| Key      | Value                                                              |
| -------- | ------------------------------------------------------------------ |
| `apiKey` | `4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO` |

这些示例假设私钥存储在文件 `test-ed25519-prv.pem` 中。

**警告：请勿与任何人分享您的 API 密钥和私钥。**

此处提供的示例密钥仅用于示范说明目的。

交易对名称完全由 ASCII 字符组成的请求示例：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol": "BTCUSDT",
    "side": "SELL",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "0.01000000",
    "price": "52000.00",
    "recvWindow": 100,
    "timestamp": 1645423376532,
    "apiKey": "4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO",
    "signature": "------ FILL ME ------"
  }
}
```

交易对名称包含非 ASCII 字符的请求示例：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol": "１２３４５６",
    "side": "BUY",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "0.01000000",
    "price": "0.10000000",
    "recvWindow": 5000,
    "timestamp": 1645423376532,
    "apiKey": "4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO",
    "signature": "------ FILL ME ------"
  }
}
```

**第一步：创建签名内容**

除了 `signature`，获取请求的所有其它 `params`，然后**按参数名称的字母顺序对它们进行排序**：

对于第一组示例参数（仅限 ASCII 字符）：

| 参数          | 取值                                                             |
| ------------- | ---------------------------------------------------------------- |
| `apiKey`      | 4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO |
| `price`       | 52000.00                                                         |
| `quantity`    | 0.01000000                                                       |
| `recvWindow`  | 100                                                              |
| `side`        | SELL                                                             |
| `symbol`      | BTCUSDT                                                          |
| `timeInForce` | GTC                                                              |
| `timestamp`   | 1645423376532                                                    |
| `type`        | LIMIT                                                            |

对于第二组示例参数（包含一些 Unicode 字符）：

| 参数          | 取值                                                             |
| ------------- | ---------------------------------------------------------------- |
| `apiKey`      | 4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO |
| `price`       | 0.20000000                                                       |
| `quantity`    | 1.00000000                                                       |
| `recvWindow`  | 5000                                                             |
| `side`        | SELL                                                             |
| `symbol`      | １２３４５６                                                     |
| `timeInForce` | GTC                                                              |
| `timestamp`   | 1668481559918                                                    |
| `type`        | LIMIT                                                            |

将参数格式化为 `参数=取值` 对并用 `&` 分隔每个参数对。数值需要采用 UTF-8 编码。

对于第一组示例参数（仅限 ASCII 字符），签名有效负载将如下所示：

```console
apiKey=4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT
```

对于第二组示例参数（包含一些 Unicode 字符），签名有效负载将如下所示：

```console
apiKey=4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO&price=0.10000000&quantity=1.00000000&recvWindow=5000&side=BUY&symbol=１２３４５６&timeInForce=GTC&timestamp=1645423376532&type=LIMIT
```

**第二步：计算签名**

1. 使用 Ed25519 密钥对步骤 1 中构造的签名有效载荷的 UTF-8 字节进行签名。
2. 将输出编码为 base64。

请注意，`apiKey`, payload 和生成的`签名值`都是**大小写敏感**的。

您可以使用 OpenSSL 交叉检查您的签名算法：

对于第一组示例参数（仅限 ASCII 字符）：

```console
echo -n "apiKey=4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT" \
  | openssl dgst -sign ./test-ed25519-prv.pem \
  | openssl enc -base64 -A

EocljwPl29jDxWYaaRaOo4pJ9wEblFbklJvPugNscLLuKd5vHM2grWjn1z+rY0aJ7r/44enxHL6mOAJuJ1kqCg==
```

对于第二组示例参数（包含一些 Unicode 字符）：

```console
echo -n "apiKey=4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO&price=0.10000000&quantity=1.00000000&recvWindow=5000&side=BUY&symbol=１２３４５６&timeInForce=GTC&timestamp=1645423376532&type=LIMIT" \
  | openssl dgst -sign ./test-ed25519-prv.pem \
  | openssl enc -base64 -A

dtNHJeyKry+cNjiGv+sv5kynO9S40tf8k7D5CfAEQAp0s2scunZj+ovJdz2OgW8XhkB9G3/HmASkA9uY9eyFCA==
```

**第三步：在请求的 `params` 参数添加签名值**

对于第一组示例参数（仅限 ASCII 字符）：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol": "BTCUSDT",
    "side": "SELL",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "0.01000000",
    "price": "52000.00",
    "newOrderRespType": "ACK",
    "recvWindow": 100,
    "timestamp": 1645423376532,
    "apiKey": "4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO",
    "signature": "EocljwPl29jDxWYaaRaOo4pJ9wEblFbklJvPugNscLLuKd5vHM2grWjn1z+rY0aJ7r/44enxHL6mOAJuJ1kqCg=="
  }
}
```

对于第二组示例参数（包含一些 Unicode 字符）：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol": "１２３４５６",
    "side": "SELL",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "quantity": "1.00000000",
    "price": "0.10000000",
    "recvWindow": 5000,
    "timestamp": 1645423376532,
    "apiKey": "4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO",
    "signature": "dtNHJeyKry+cNjiGv+sv5kynO9S40tf8k7D5CfAEQAp0s2scunZj+ovJdz2OgW8XhkB9G3/HmASkA9uY9eyFCA=="
  }
}
```

下面的 Python 示例代码能执行上述所有步骤。

```python
#!/usr/bin/env python3

import base64
import time
import json
from cryptography.hazmat.primitives.serialization import load_pem_private_key
from websocket import create_connection

# 设置身份验证：
API_KEY='替换成您的 API Key'
PRIVATE_KEY_PATH='test-prv-key.pem'

# 加载 private key。
# 在这个例子中，private key 没有加密
# 但我们建议使用强密码以提高安全性。
with open(PRIVATE_KEY_PATH, 'rb') as f:
  private_key = load_pem_private_key(data=f.read(), password=None)

# 设置请求参数：
params = {
    'apiKey':       API_KEY,
    'symbol':       '１２３４５６',
    'side':         'SELL',
    'type':         'LIMIT',
    'timeInForce':  'GTC',
    'quantity':     '1.0000000',
    'price':        '0.10000000',
    'recvWindow':   5000
}

# 参数中加时间戳：
timestamp = int(time.time() * 1000) # UNIX timestamp in milliseconds
params['timestamp'] = timestamp

# 按参数名称的字母顺序进行排序
params = dict(sorted(params.items()))

# 计算签名有效负载
payload = '&'.join([f"{k}={v}" for k,v in params.items()]) # no percent encoding here!

# 对请求进行签名
signature = base64.b64encode(private_key.sign(payload.encode('UTF-8')))
params['signature'] = signature.decode('ASCII')

# 发送请求：
request = {
    'id': 'my_new_order',
    'method': 'order.place',
    'params': params
}

ws = create_connection("wss://ws-api.binance.com:443/ws-api/v3")
ws.send(json.dumps(request))
result =  ws.recv()
ws.close()

print(result)
```

<a id="session-authentication"></a>

## 会话身份验证

**注意：** 仅支持 _Ed25519_ 密钥用于此功能。

如果你不想在每个单独的请求中指定`apiKey`和`signature`，你可以为有效的WebSocket会话进行API密钥身份验证。

一旦完成身份验证，你将不需在需要它们的请求中指定`apiKey`和`signature`。这些请求将代表拥有已验证API密钥的帐户执行。

**注意：** 对于`SIGNED`请求，你仍需要指定`timestamp`参数。

<a id="authenticate-after-connection"></a>

### 连接后进行身份验证

你可以使用会话身份验证请求对已经建立的连接进行身份验证：

- [`session.logon`](web-socket-api_CN.md#log-in-with-api-key-user_data)
  – 进行身份验证，或更改与连接相关联的API密钥。
- [`session.status`](web-socket-api_CN.md#query-session-status)
  – 检查连接状态和当前API密钥。
- [`session.logout`](web-socket-api_CN.md#log-out-of-the-session)
  – 忘记与连接关联的API密钥。

**关于吊销API密钥:**

如果在活动会话期间，由于 _任何_
原因（例如IP地址未被加入白名单、API密钥被删除、API密钥没有正确的权限等），在下一个请求后，会话将被吊销，并显示以下错误消息:

```javascript
{
    "id": null,
    "status": 401,
    "error": {
        "code": -2015,
        "msg": "Invalid API-key, IP, or permissions for action."
    }
}
```

<a id="authorize-_ad-hoc_-requests"></a>

### 授权 _临时_ 请求

WebSocket连接只能通过一个API密钥进行身份验证。默认情况下，经过身份验证的API密钥将用于需要`apiKey`参数的请求。但是，你始终可以为单个请求明确指定`apiKey`和`signature`，覆盖已认证的API密钥，以使用不同的API密钥授权特定请求。

例如，你可能希望用默认密钥来验证 `USER_DATA`，但在下单时使用`TRADE`密钥来签名。

<a id="data-sources"></a>

## 数据源

- API 系统是异步的。响应中一些延迟是正常和预期的。

- 每种函数都会有一个数据源, 用来指示数据的来源，以及它的最新程度。

| 数据源   | 延迟 | 描述                             |
| -------- | ---- | -------------------------------- |
| 撮合引擎 | 最低 | 表示撮合引擎直接产生响应         |
| 缓存     | 低   | 表示数据来源于内部或者外部的缓存 |
| 数据库   | 中度 | 表示数据直接来源于数据库         |

- 某些函数有多个数据源（例如，缓存 => 数据库）。

  这代表 API 将按该顺序查找最新数据：首先是缓存，然后是数据库。

<a id="public-api-requests"></a>

# 公共 API 请求

<a id="general-requests"></a>

## 常用请求信息

<a id="ping"></a>

<a id="test-connectivity"></a>

### 测试连通性

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "ping"
}
```
测试能否联通 WebSocket API。

**注意:**

您也可以使用常规 WebSocket ping 帧来测试连通性，
WebSocket API 将尽快以 pong 帧响应。
`ping` 请求和 `time` 是在应用程序中测试请求-响应处理的安全方法。

**权重:**
1

**参数:**
NONE

**数据源:** 缓存

**响应:**
```javascript
{
    "id": "922bcc6e-9de8-440d-9e84-7c80933a8d0d",
    "status": 200,
    "result": {},
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

<a id="time"></a>

<a id="check-server-time"></a>

### 检查服务器时间

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "time"
}
```
测试与 WebSocket API 的连通性并获取当前服务器时间。

**权重:**
1

**参数:**
NONE

**数据源:** 缓存

**响应:**
```javascript
{
    "id": "187d3cb2-942d-484c-8271-4e2141bbadb1",
    "status": 200,
    "result": {
        "serverTime": 1656400526260
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

<a id="exchangeInfo"></a>
<a id="exchange-info"></a>
<a id="exchangeinfo"></a>

<a id="exchange-information"></a>

### 交易规范信息

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "exchangeInfo"
}
```
获取交易规则，速率限制，和交易对信息。

**权重:**
20

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | NO | 描述单个交易对
`symbols` | ARRAY OF STRING | NO | 描述多个交易对
`permissions` | ARRAY OF STRING | NO | 按权限过滤交易对
`showPermissionSets` | BOOLEAN | NO | 控制 `permissionSets` 字段的内容是否填充。默认为 `true`。
`symbolStatus` | ENUM | NO | 过滤具有此 `tradingStatus` 的交易对。有效值：`TRADING`, `HALT`, `BREAK`。不能与 `symbol` 或 `symbols` 组合使用。

**数据源:** 缓存

**备注:**
* 如果参数 `symbol` 或者 `symbols` 提供的交易对不存在, 系统会返回错误并提示交易对不正确。
* 所有的参数都是可选的。
* `symbol`、`symbols`、`permissions` 参数只能指定其中一个。
* 不带参数时, `exchangeInfo` 显示所有具有 `["SPOT", "MARGIN", "LEVERAGED"]` 权限的交易对。
  * 要列出交易所上所有活跃的交易对, 您需要在 `permissions` 中明确指定它们。
* `permissions` 可以支持单个或多个值（例如 `"SPOT"`, `["MARGIN","LEVERAGED"]`）。

<a id="examples-of-symbol-permissions-interpretation-from-the-response"></a>

**解释响应中的 `permissionSets`：**

* `[["A","B"]]` 有权限"A"**或**权限"B"的账户可以下订单。
* `[["A"],["B"]]` 有权限"A"**和**权限"B"的账户可以下订单。
* `[["A"],["B","C"]]` 有权限"A"**和**权限"B"或权限"C"的账户可以下订单。（此处应用的是包含或，而不是排除或，因此账户可以同时拥有权限"B"和权限"C"。）

**响应:**
```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "status": 200,
    "result": {
        "timezone": "UTC",
        "serverTime": 1655969291181,
        // 全局速率限制。请参阅 "速率限制" 部分。
        "rateLimits": [
            {
                "rateLimitType": "REQUEST_WEIGHT", // 速率限制类型: REQUEST_WEIGHT，ORDERS，CONNECTIONS
                "interval": "MINUTE",              // 速率限制间隔: SECOND，MINUTE，DAY
                "intervalNum": 1,                  // 速率限制间隔乘数 (i.e.，"1 minute")
                "limit": 6000                      // 每个间隔的速率限制
            },
            {
                "rateLimitType": "ORDERS",
                "interval": "SECOND",
                "intervalNum": 10,
                "limit": 50
            },
            {
                "rateLimitType": "ORDERS",
                "interval": "DAY",
                "intervalNum": 1,
                "limit": 160000
            },
            {
                "rateLimitType": "CONNECTIONS",
                "interval": "MINUTE",
                "intervalNum": 5,
                "limit": 300
            }
        ],
        // 交易所级别过滤器在 "过滤器" 页面上进行了说明：
        // https://github.com/binance/binance-spot-api-docs/blob/master/filters_CN.md
        // 全部交易过滤器是可选的。
        "exchangeFilters": [
            {
                "filterType": "EXCHANGE_MAX_NUM_ORDERS",
                "maxNumOrders": 1000
            }
        ],
        "symbols": [
            {
                "symbol": "BNBBTC",
                "status": "TRADING",
                "baseAsset": "BNB",
                "baseAssetPrecision": 8,
                "quoteAsset": "BTC",
                "quotePrecision": 8,
                "quoteAssetPrecision": 8,
                "baseCommissionPrecision": 8,
                "quoteCommissionPrecision": 8,
                "orderTypes": [
                    "LIMIT",
                    "LIMIT_MAKER",
                    "MARKET",
                    "STOP_LOSS_LIMIT",
                    "TAKE_PROFIT_LIMIT"
                ],
                "icebergAllowed": true,
                "ocoAllowed": true,
                "otoAllowed": true,
                "opoAllowed": true,
                "quoteOrderQtyMarketAllowed": true,
                "allowTrailingStop": true,
                "cancelReplaceAllowed": true,
                "amendAllowed": false,
                "pegInstructionsAllowed": true,
                "isSpotTradingAllowed": true,
                "isMarginTradingAllowed": true,
                // 交易对过滤器在"过滤器"页面上进行了说明：
                // https://github.com/binance/binance-spot-api-docs/blob/master/filters_CN.md
                // 全部交易对过滤器是可选的。
                "filters": [
                    {
                        "filterType": "PRICE_FILTER",
                        "minPrice": "0.00000100",
                        "maxPrice": "100000.00000000",
                        "tickSize": "0.00000100"
                    },
                    {
                        "filterType": "LOT_SIZE",
                        "minQty": "0.00100000",
                        "maxQty": "100000.00000000",
                        "stepSize": "0.00100000"
                    }
                ],
                "permissions": [
                    "SPOT"
                ],
                "permissionSets": [
                    [
                        "SPOT",
                        "MARGIN",
                        "TRD_GRP_004"
                    ]
                ],
                "defaultSelfTradePreventionMode": "NONE",
                "allowedSelfTradePreventionModes": [
                    "NONE"
                ]
            }
        ],
        // 可选字段，仅当 SOR 可用时才会被显示出来。
        // https://github.com/binance/binance-spot-api-docs/blob/master/faqs/sor_faq_CN.md
        "sors": [
            {
                "baseAsset": "BTC",
                "symbols": [
                    "BTCUSDT",
                    "BTCUSDC"
                ]
            }
        ]
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 20
        }
    ]
}
```

<a id="executionRules"></a>
<a id="execution-rules"></a>
<a id="executionrules"></a>

<a id="query-execution-rules"></a>

### 查询执行规则

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "executionRules"
}
```
查询交易对的执行规则。

**权重:**
参数 | 权重
--- | ---
`symbol` | 2
`symbols` | 每个 `symbol` 占用 2，最多为 40
`symbolStatus` | 40
无 | 40

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | NO | 查询指定的交易对。
`symbols` | ARRAY OF STRING | NO | 查询多个交易对。
`symbolStatus` | ENUM | NO | 查询指定状态的所有交易对。支持的值：`TRADING`, `HALT`, `BREAK`

**数据源:** 缓存

**注意:** 不允许多个参数组合使用。

**响应:**
```javascript
{
    "id": "5162affb-0aba-4821-b475-f2625006eb43",
    "status": 200,
    "result": {
        "symbolRules": [
            {
                "symbol": "BAZUSD",
                "rules": [
                    {
                        "ruleType": "PRICE_RANGE",
                        "bidLimitMultUp": "1.0001",
                        "bidLimitMultDown": "0.9999",
                        "askLimitMultUp": "1.0001",
                        "askLimitMultDown": "0.9999"
                    }
                ]
            }
        ]
    }
}
```

<a id="market-data-requests"></a>

## 行情接口

<a id="depth"></a>

<a id="order-book"></a>

### 订单簿深度信息

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "depth",
    "params": {
        "symbol": "BNBUSDT"
    }
}
```
获取当前深度信息。

请注意，此请求返回有限的市场深度。

如果需要持续监控深度信息更新，请考虑使用 WebSocket Streams：
  * `<symbol>@depth<levels>`
  * `<symbol>@depth`

如果需要[维护本地orderbook](web-socket-streams_CN.md#how-to-maintain-orderbook)，您可以将 `depth` 请求与 `<symbol>@depth` streams 一起使用。

**权重:**
根据 limit 调整：

|限制|请求权重
------|-------
1-100|  5
101-500| 25
501-1000| 50
1001-5000| 250

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | YES |
`limit` | INT | NO | 默认值： 100； 最大值： 5000。
`symbolStatus` | ENUM | NO | 过滤具有此 `tradingStatus` 的交易对。 <br/> 如果状态不匹配，将返回错误 `-1220 交易对与状态不匹配`

**数据源:** 缓存

**响应:**
```javascript
{
    "id": "51e2affb-0aba-4821-ba75-f2625006eb43",
    "status": 200,
    "result": {
        "lastUpdateId": 2731179239,
        // 买单列表。每项为 [价格, 数量]
        "bids": [
            [
                "0.01379900", // 价位
                "3.43200000"  // 挂单量
            ],
            [
                "0.01379800", // 价位
                "3.24300000"  // 挂单量
            ],
            [
                "0.01379700", // 价位
                "10.45500000" // 挂单量
            ],
            [
                "0.01379600", // 价位
                "3.82100000"  // 挂单量
            ],
            [
                "0.01379500", // 价位
                "10.26200000" // 挂单量
            ]
        ],
        // 卖单列表。每项为 [价格, 数量]
        "asks": [
            [
                "0.01380000", // 价位
                "5.91700000"  // 挂单量
            ],
            [
                "0.01380100", // 价位
                "6.01400000"  // 挂单量
            ],
            [
                "0.01380200", // 价位
                "0.26800000"  // 挂单量
            ],
            [
                "0.01380300", // 价位
                "0.33800000"  // 挂单量
            ],
            [
                "0.01380400", // 价位
                "0.26800000"  // 挂单量
            ]
        ]
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

<a id="trades.recent"></a>
<a id="trades-recent"></a>

<a id="recent-trades"></a>

### 最近的交易

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "trades.recent",
    "params": {
        "symbol": "BNBUSDT"
    }
}
```
获取最近的交易

如果您需要访问实时交易活动，请考虑使用 WebSocket Streams：

* `<symbol>@trade`

**权重:**
25

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | YES |
`limit` | INT | NO | 默认值： 500； 最大值： 1000。

**数据源:** 缓存

**响应:**
```javascript
{
    "id": "409a20bd-253d-41db-a6dd-687862a5882f",
    "status": 200,
    "result": [
        {
            "id": 194686783,
            "price": "0.01361000",
            "qty": "0.01400000",
            "quoteQty": "0.00019054",
            "time": 1660009530807,
            "isBuyerMaker": true,
            "isBestMatch": true
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 25
        }
    ]
}
```

<a id="trades.historical"></a>
<a id="trades-historical"></a>

<a id="historical-trades"></a>

### 历史交易

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "trades.historical",
    "params": {
        "symbol": "BNBUSDT"
    }
}
```
获取历史交易。

**权重:**
25

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | YES |
`fromId` | LONG | NO | 起始交易ID
`limit` | INT | NO | 默认值： 500； 最大值： 1000。

**数据源:** 数据库

备注：

* 如果 `fromId` 未指定，则返回最近的交易。

**响应:**
```javascript
{
    "id": "cffc9c7d-4efc-4ce0-b587-6b87448f052a",
    "status": 200,
    "result": [
        {
            "id": 0,
            "price": "0.00005000",
            "qty": "40.00000000",
            "quoteQty": "0.00200000",
            "time": 1500004800376,
            "isBuyerMaker": true,
            "isBestMatch": true
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 25
        }
    ]
}
```

<a id="blockTrades.historical"></a>
<a id="blockTrades-historical"></a>
<a id="block-trades-historical"></a>
<a id="blocktrades-historical"></a>
<a id="historicalBlockTrades"></a>

<a id="historical-block-trades"></a>

### 查询历史大宗交易

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "blockTrades.historical",
    "params": {
        "symbol": "BNBBTC",
        "fromId": 582
    }
}
```
获取大宗交易记录。

**权重:**
25

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | YES |
`fromId` | LONG | YES | 从该大宗交易ID开始查询
`limit` | LONG | NO | 默认: 500; 最大: 1000

**数据源:** 数据库

**响应:**
```javascript
{
    "id": "cffc9c7d-4efc-4ce0-b587-6b87448f052a",
    "status": 200,
    "result": [
        {
            "id": 582,
            "price": "0.052",
            "qty": "5838",
            "quoteQty": "303.576",
            "time": 1772506983321,
            "isBuyerMaker": true
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 10
        }
    ]
}
```

<a id="trades.aggregate"></a>
<a id="trades-aggregate"></a>

<a id="aggregate-trades"></a>

### 归集交易

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "trades.aggregate",
    "params": {
        "symbol": "BNBUSDT"
    }
}
```
获取归集交易。

一个 *归集交易* (aggtrade) 代表一个或多个单独的交易。

同时间，同 taker 订单和同价格的执行交易会被聚合为一条归集交易。

如果需要访问实时交易活动，请考虑使用 WebSocket Streams：

* `<symbol>@aggTrade`

如果需要历史总交易数据，可以使用 [data.binance.vision](https://github.com/binance/binance-public-data/#aggtrades)。

**权重:**
4

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | YES |
`fromId` | LONG | NO | 起始归集交易ID
`startTime` | LONG | NO | 从该时刻之后的成交记录开始返回结果
`endTime` | LONG | NO | 返回该时刻为止的成交记录
`limit` | INT | NO | 默认值： 500； 最大值： 1000。

**数据源:** 数据库

- 如果指定了 `fromId`，则返回归集交易ID >= `fromId` 的归集交易。使用 `fromId` 和 `limit` 分页查询所有归集交易。
- 如果指定了 `startTime` 和/或 `endTime`，则按执行时间 (`T`) 过滤归集交易。`fromId` 不能与 `startTime` 和 `endTime` 同时使用。
- 如果没有指定条件，将返回最近的归集交易。

**响应:**
```javascript
{
    "id": "189da436-d4bd-48ca-9f95-9f613d621717",
    "status": 200,
    "result": [
        {
            "a": 50000000,      // 归集成交ID
            "p": "0.00274100",  // 成交价
            "q": "57.19000000", // 成交量
            "f": 59120167,      // 被归集的首个成交ID
            "l": 59120170,      // 被归集的末个成交ID
            "T": 1565877971222, // 成交时间
            "m": true,          // 是否为主动卖出单
            "M": true           // 是否为最优撮合单(可忽略，目前总为最优撮合)
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 4
        }
    ]
}
```

<a id="klines"></a>

### K线数据

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "klines",
    "params": {
        "symbol": "BNBUSDT",
        "interval": "1s"
    }
}
```
获取K线数据。

Klines 由其开盘时间和收盘时间为唯一标识。

如果您需要访问实时 kline 更新，请考虑使用 WebSocket Streams：
  * `<symbol>@kline_<interval>`

如果需要历史K线数据，可以使用 [data.binance.vision](https://github.com/binance/binance-public-data/#klines)。

**权重:**
2

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | YES |
`interval` | ENUM | YES |
`startTime` | LONG | NO |
`endTime` | LONG | NO |
`timeZone` | STRING | NO | 默认值： 0 (UTC)
`limit` | INT | NO | 默认值： 500； 最大值： 1000。

**数据源:** 数据库

<a id="kline-intervals"></a>

支持的K线间隔 （区分大小写）：

间隔  | `间隔` 值
--------- | ----------------
seconds -> 秒   | `1s`
minutes -> 分钟 | `1m`， `3m`， `5m`， `15m`， `30m`
hours -> 小时   | `1h`， `2h`， `4h`， `6h`， `8h`， `12h`
days -> 天      | `1d`， `3d`
weeks -> 周    | `1w`
months -> 月  | `1M`

**请注意：**

* 如果未发送`startTime`和`endTime`，将返回最近的K线数据。
* `timeZone`支持的值包括：
    * 小时和分钟（例如 `-1:00`，`05:45`）
    * 仅小时（例如 `0`，`8`，`4`）
    * 接受的值范围严格为 [-12:00 到 +14:00]（包括边界）
* 如果提供了`timeZone`，K线间隔将在该时区中解释，而不是在UTC中。
* 请注意，无论`timeZone`如何，`startTime`和`endTime`始终以UTC时区解释。

**响应:**
```javascript
{
    "id": "1dbbeb56-8eea-466a-8f6e-86bdcfa2fc0b",
    "status": 200,
    "result": [
        [
            1499040000000,     // 开盘时间
            "0.01634790",      // 开盘价
            "0.80000000",      // 最高价
            "0.01575800",      // 最低价
            "0.01577100",      // 收盘价
            "148976.11427815", // 成交量
            1499644799999,     // 收盘时间
            "2434.19055334",   // 成交额
            308,               // 成交笔数
            "1756.87402397",   // 主动买入成交量
            "28.46694368",     // 主动买入成交额
            "0"                // 忽略字段
        ]
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

<a id="uiKlines"></a>
<a id="ui-klines"></a>
<a id="uiklines"></a>

### UI K线数据

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "uiKlines",
    "params": {
        "symbol": "BNBUSDT",
        "interval": "1s"
    }
}
```
请求参数和响应字段与`k线`接口相同。
uiKlines 是返回修改后的k线数据，针对k线图的呈现进行了优化。

**权重:**
2

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | YES |
`interval` | ENUM | YES |
`startTime` | LONG | NO |
`endTime` | LONG | NO |
`timeZone` | STRING | NO | 默认值： 0 (UTC)
`limit` | INT | NO | 默认值： 500； 最大值： 1000。

**数据源:** 数据库

- 如果未发送 `startTime` 和 `endTime`，默认返回最近的交易。
- `timeZone`支持的值包括：
    - 小时和分钟（例如 `-1:00`，`05:45`）
    - 仅小时（例如 `0`，`8`，`4`）
    - 接受的值范围严格为 [-12:00 到 +14:00]（包括边界）
- 如果提供了`timeZone`，K线间隔将在该时区中解释，而不是在UTC中。
- 请注意，无论`timeZone`如何，`startTime`和`endTime`始终以UTC时区解释。

**响应:**
```javascript
{
    "id": "b137468a-fb20-4c06-bd6b-625148eec958",
    "status": 200,
    "result": [
        [
            1499040000000,     // 开盘时间
            "0.01634790",      // 开盘价
            "0.80000000",      // 最高价
            "0.01575800",      // 最低价
            "0.01577100",      // 收盘价
            "148976.11427815", // 成交量
            1499644799999,     // 收盘时间
            "2434.19055334",   // 成交额
            308,               // 成交笔数
            "1756.87402397",   // 主动买入成交量
            "28.46694368",     // 主动买入成交额
            "0"                // 忽略字段
        ]
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

<a id="avgPrice"></a>
<a id="avg-price"></a>
<a id="avgprice"></a>

<a id="current-average-price"></a>

### 当前平均价格

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "avgPrice",
    "params": {
        "symbol": "BNBUSDT"
    }
}
```
获取交易对的当前平均价格

**权重:**
2

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | YES |

**数据源:** 缓存

**响应:**
```javascript
{
    "id": "ddbfb65f-9ebf-42ec-8240-8f0f91de0867",
    "status": 200,
    "result": {
        "mins": 5,                 // 平均价格间隔（以分钟为单位）
        "price": "0.01378135",     // 平均价格
        "closeTime": 1694061154503 // 最近成交时间
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

<a id="ticker.24hr"></a>
<a id="ticker-24hr"></a>
<a id="twentyfourhourticker"></a>

<a id="24hr-ticker-price-change-statistics"></a>

### 24hr 价格变动情况

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "ticker.24hr"
}
```
24 小时滚动窗口价格变动数据。
如果您需要持续监控交易统计，请考虑使用 WebSocket Streams:

* [`<symbol>@ticker`](web-socket-streams_CN.md#individual-symbol-ticker-streams)
* [`<symbol>@miniTicker`](web-socket-streams_CN.md#individual-symbol-mini-ticker-stream) 或者 [`!miniTicker@arr`](web-socket-streams_CN.md#all-market-mini-tickers-stream)

如果你想用不同的窗口数量，可以用 [`ticker`](web-socket-api_CN.md#rolling-window-price-change-statistics) 请求。

**权重:**
根据请求的交易对数量进行调整：

|参数|提供Symbol数量|权重|
|---|---|---|
|symbol| 1 |2|
| |不提供| 80|
|symbols| 1-20 |2|
| | 21-100 |40|
| | 101+ |80|
| |不提供| 80|

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | NO |
`symbols` | ARRAY OF STRING | NO |
`type` | ENUM | NO | Ticker 类型: FULL (默认) 或者 MINI
`symbolStatus` | ENUM | NO | 过滤具有此 `tradingStatus` 的交易对。对于单个交易对，如果状态不匹配，将返回错误 `-1220 交易对与状态不匹配`。对于多个或者全部交易对，响应中不会包括状态不匹配的交易对。有效值： `TRADING`, `HALT`, `BREAK`

**数据源:** 缓存

备注:

* `symbol` 和 `symbols` 不能同时用。

* 如果未指定交易对，则返回有关当前在交易所交易的所有交易对的信息。

**响应:**

`FULL` 类型，对于单个交易对：

```javascript
{
    "id": "9fa2a91b-3fca-4ed7-a9ad-58e3b67483de",
    "status": 200,
    "result": {
        "symbol": "BNBBTC",            // 交易对
        "priceChange": "0.00013900",
        "priceChangePercent": "1.020",
        "weightedAvgPrice": "0.01382453",
        "prevClosePrice": "0.01362800",
        "lastPrice": "0.01376700",     // 间隔收盘价
        "lastQty": "1.78800000",
        "bidPrice": "0.01376700",
        "bidQty": "4.64600000",
        "askPrice": "0.01376800",
        "askQty": "14.31400000",
        "openPrice": "0.01362800",     // 间隔开盘价
        "highPrice": "0.01414900",     // 间隔最高价
        "lowPrice": "0.01346600",      // 间隔最低价
        "volume": "69412.40500000",    // 总交易量 (base asset)
        "quoteVolume": "959.59411487", // 总交易量 (quote asset)
        "openTime": 1660014164909,     // ticker间隔的开始时间
        "closeTime": 1660100564909,    // ticker间隔的结束时间
        "firstId": 194696115,          // 首笔成交id
        "lastId": 194968287,           // 末笔成交id
        "count": 272173                // 成交笔数
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

`MINI` 类型，对于单个交易对：

```javascript
{
    "id": "9fa2a91b-3fca-4ed7-a9ad-58e3b67483de",
    "status": 200,
    "result": {
        "symbol": "BNBBTC",            // 交易对
        "openPrice": "0.01362800",     // 间隔开盘价
        "highPrice": "0.01414900",     // 间隔最高价
        "lowPrice": "0.01346600",      // 间隔最低价
        "lastPrice": "0.01376700",     // 间隔收盘价
        "volume": "69412.40500000",    // 总交易量 (base asset)
        "quoteVolume": "959.59411487", // 总交易量 (quote asset)
        "openTime": 1660014164909,     // ticker间隔的开始时间
        "closeTime": 1660100564909,    // ticker间隔的结束时间
        "firstId": 194696115,          // 第一个交易 ID
        "lastId": 194968287,           // 最后一个交易ID
        "count": 272173                // 成交笔数
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2 // 成交笔数
        }
    ]
}
```

如果请求是有多个交易对，响应会是数组类型:

```javascript
{
    "id": "901be0d9-fd3b-45e4-acd6-10c580d03430",
    "status": 200,
    "result": [
        {
            "symbol": "BNBBTC",            // 交易对
            "priceChange": "0.00016500",
            "priceChangePercent": "1.213",
            "weightedAvgPrice": "0.01382508",
            "prevClosePrice": "0.01360800",
            "lastPrice": "0.01377200",     // 间隔收盘价
            "lastQty": "1.01400000",
            "bidPrice": "0.01377100",
            "bidQty": "7.55700000",
            "askPrice": "0.01377200",
            "askQty": "4.37900000",
            "openPrice": "0.01360700",     // 间隔开盘价
            "highPrice": "0.01414900",     // 间隔最高价
            "lowPrice": "0.01346600",      // 间隔最低价
            "volume": "69376.27900000",    // 总交易量 (base asset)
            "quoteVolume": "959.13277091", // 总交易量 (quote asset)
            "openTime": 1660014615517,     // ticker间隔的开始时间
            "closeTime": 1660101015517,    // ticker间隔的结束时间
            "firstId": 194697254,          // 首笔成交id
            "lastId": 194969483,           // 末笔成交id
            "count": 272230                // 成交笔数
        },
        {
            "symbol": "BTCUSDT",
            "priceChange": "-938.06000000",
            "priceChangePercent": "-3.938",
            "weightedAvgPrice": "23265.34432003",
            "prevClosePrice": "23819.17000000",
            "lastPrice": "22880.91000000",
            "lastQty": "0.00536000",
            "bidPrice": "22880.40000000",
            "bidQty": "0.00424000",
            "askPrice": "22880.91000000",
            "askQty": "0.04276000",
            "openPrice": "23818.97000000",
            "highPrice": "23933.25000000",
            "lowPrice": "22664.69000000",
            "volume": "153508.37606000",
            "quoteVolume": "3571425225.04441220",
            "openTime": 1660014615977,
            "closeTime": 1660101015977,
            "firstId": 1592019902,
            "lastId": 1597301762,
            "count": 5281861
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

<a id="ticker.tradingDay"></a>
<a id="ticker-tradingDay"></a>
<a id="ticker-trading-day"></a>
<a id="ticker-tradingday"></a>

<a id="trading-day-ticker"></a>

### 交易日行情(Ticker)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "ticker.tradingDay",
    "params": {
        "symbols": [
            "BNBBTC",
            "BTCUSDT"
        ],
        "timeZone": "00:00"
    }
}
```
交易日价格变动统计。

**权重:**
每个交易对占用 4 个权重，与 windowSize 无关。当请求中的交易对数量超过 50 时，此请求的权重上限为 200。

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | NO |
`symbols` | ARRAY OF STRING | NO |
`timeZone` | STRING | NO | 默认值： 0 (UTC)
`type` | ENUM | NO | Ticker 类型: FULL (默认) 或者 MINI
`symbolStatus` | ENUM | NO | 过滤具有此 `tradingStatus` 的交易对。对于单个交易对，如果状态不匹配，将返回错误 `-1220 交易对与状态不匹配`。对于多个或者全部交易对，响应中不会包括状态不匹配的交易对。有效值： `TRADING`, `HALT`, `BREAK`

**数据源:** 数据库

**注意:**

* `timeZone`支持的值包括：
    * 小时和分钟（例如 `-1:00`，`05:45`）
    * 仅小时（例如 `0`，`8`，`4`）

**响应:**

`FULL` 类型，有 `symbol`:

```javascript
{
    "id": "f4b3b507-c8f2-442a-81a6-b2f12daa030f",
    "status": 200,
    "result": {
        "symbol": "BTCUSDT",
        "priceChange": "-83.13000000",        // 绝对价格变动
        "priceChangePercent": "-0.317",       // 相对价格变动百分比
        "weightedAvgPrice": "26234.58803036", // 报价成交量 / 成交量
        "openPrice": "26304.80000000",
        "highPrice": "26397.46000000",
        "lowPrice": "26088.34000000",
        "lastPrice": "26221.67000000",
        "volume": "18495.35066000",           // 基础资产的成交量
        "quoteVolume": "485217905.04210480",
        "openTime": 1695686400000,
        "closeTime": 1695772799999,
        "firstId": 3220151555,
        "lastId": 3220849281,
        "count": 697727
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 4
        }
    ]
}
```

`FULL` 类型，有 `symbols`:

```javascript
{
    "id": "f4b3b507-c8f2-442a-81a6-b2f12daa030f",
    "status": 200,
    "result": [
        {
            "symbol": "BTCUSDT",
            "priceChange": "-83.13000000",        // 绝对价格变动
            "priceChangePercent": "-0.317",       // 相对价格变动百分比
            "weightedAvgPrice": "26234.58803036", // 报价成交量 / 成交量
            "openPrice": "26304.80000000",
            "highPrice": "26397.46000000",
            "lowPrice": "26088.34000000",
            "lastPrice": "26221.67000000",
            "volume": "18495.35066000",           // 基础资产的成交量
            "quoteVolume": "485217905.04210480",  // 报价资产的成交量
            "openTime": 1695686400000,
            "closeTime": 1695772799999,
            "firstId": 3220151555,                // 区间内的第一个交易的交易ID
            "lastId": 3220849281,                 // 区间内的最后一个交易的交易ID
            "count": 697727                       // 区间内的交易数量
        },
        {
            "symbol": "BNBUSDT",
            "priceChange": "2.60000000",
            "priceChangePercent": "1.238",
            "weightedAvgPrice": "211.92276958",
            "openPrice": "210.00000000",
            "highPrice": "213.70000000",
            "lowPrice": "209.70000000",
            "lastPrice": "212.60000000",
            "volume": "280709.58900000",
            "quoteVolume": "59488753.54750000",
            "openTime": 1695686400000,
            "closeTime": 1695772799999,
            "firstId": 672397461,
            "lastId": 672496158,
            "count": 98698
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 8
        }
    ]
}
```

`MINI` 类型，有 `symbol`:

```javascript
{
    "id": "f4b3b507-c8f2-442a-81a6-b2f12daa030f",
    "status": 200,
    "result": {
        "symbol": "BTCUSDT",
        "openPrice": "26304.80000000",
        "highPrice": "26397.46000000",
        "lowPrice": "26088.34000000",
        "lastPrice": "26221.67000000",
        "volume": "18495.35066000",          // 基础资产的成交量
        "quoteVolume": "485217905.04210480", // 报价资产的成交量
        "openTime": 1695686400000,
        "closeTime": 1695772799999,
        "firstId": 3220151555,               // 区间内的第一个交易的交易ID
        "lastId": 3220849281,                // 区间内的最后一个交易的交易ID
        "count": 697727                      // 区间内的交易数量
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 4
        }
    ]
}
```

`MINI` 类型，有 `symbols`:

```javascript
{
    "id": "f4b3b507-c8f2-442a-81a6-b2f12daa030f",
    "status": 200,
    "result": [
        {
            "symbol": "BTCUSDT",
            "openPrice": "26304.80000000",
            "highPrice": "26397.46000000",
            "lowPrice": "26088.34000000",
            "lastPrice": "26221.67000000",
            "volume": "18495.35066000",          // 基础资产的成交量
            "quoteVolume": "485217905.04210480", // 报价资产的成交量
            "openTime": 1695686400000,
            "closeTime": 1695772799999,
            "firstId": 3220151555,               // 区间内的第一个交易的交易ID
            "lastId": 3220849281,                // 区间内的最后一个交易的交易ID
            "count": 697727                      // 区间内的交易数量
        },
        {
            "symbol": "BNBUSDT",
            "openPrice": "210.00000000",
            "highPrice": "213.70000000",
            "lowPrice": "209.70000000",
            "lastPrice": "212.60000000",
            "volume": "280709.58900000",
            "quoteVolume": "59488753.54750000",
            "openTime": 1695686400000,
            "closeTime": 1695772799999,
            "firstId": 672397461,
            "lastId": 672496158,
            "count": 98698
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 8
        }
    ]
}
```

<a id="ticker"></a>

<a id="rolling-window-price-change-statistics"></a>

### 滚动窗口价格变动统计

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "ticker",
    "params": {
        "symbols": [
            "BNBBTC",
            "BTCUSDT"
        ],
        "windowSize": "7d"
    }
}
```
使用自定义窗口获取滚动窗口价格变化统计信息。

这个请求类似于 [`ticker.24hr`](web-socket-api_CN.md#24hr-ticker-price-change-statistics)，但统计数据是使用指定的任意窗口按需计算的。

**注意：** 窗口大小精度限制为1分钟。
虽然 `closeTime` 是请求的当前时间，`openTime` 总是从分钟边界开始。
因此，有效窗口可能比请求的 `windowSize` 宽59999毫秒。

<details>
<summary>窗口计算示例</summary>

例如，对 `"windowSize": "7d"` 的请求可能会导致以下窗口：

```javascript
{
    "openTime": 1659580020000,
    "closeTime": 1660184865291
}
```

请求的时间 - `closeTime` - 是 1660184865291（2022年8月11日 02:27:45.291）。
请求的窗口大小应将 `openTime` 设置为7天之前 – 8月4日，02:27:45.291 – 但由于精度有限，它最终会提前一点：1659580020000（2022年8月4日 02:27:00），正好在一分钟开始。
</details>

如果您需要持续监控交易统计，请考虑使用 WebSocket Streams:
  * `<symbol>@ticker_<window_size>` 或者 `!ticker_<window-size>@arr`

**权重:**
根据请求的交易对数量进行调整：

| 交易对 | 权重 |
|:-------:|:------:|
|    1–50 | 每个交易对 4 |
|  51–100 |    200 |

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | NO | 获取单个交易对的ticker
`symbols` | ARRAY OF STRING | NO | 获取多个交易对的ticker
`type` | ENUM | NO | Ticker 类型: FULL (默认) 或者 MINI
`windowSize` | ENUM | NO | 默认为 1d
`symbolStatus` | ENUM | NO | 过滤具有此 `tradingStatus` 的交易对。对于单个交易对，如果状态不匹配，将返回错误 `-1220 交易对与状态不匹配`。对于多个或者全部交易对，响应中不会包括状态不匹配的交易对。有效值： `TRADING`, `HALT`, `BREAK`

**数据源:** 数据库

支持的窗口 size:

单位    | `windowSize` 值
------- | ------------------
minutes | `1m`, `2m` ... `59m`
hours   | `1h`, `2h` ... `23h`
days    | `1d`, `2d` ... `7d`

备注：

* 必须指定 `symbol` 或 `symbols`。

* 一个请求中的最大交易对数：200。

* 窗口 size 单位不能合并。
  比如不支持 <code>1d 2h</code>。

**响应:**

`FULL` 类型，对于单个交易对：

```javascript
{
    "id": "bdb7c503-542c-495c-b797-4d2ee2e91173",
    "status": 200,
    "result": {
        "symbol": "BNBBTC",
        "priceChange": "0.00061500",      // 价格变化
        "priceChangePercent": "4.735",    // 价格变化百分比
        "weightedAvgPrice": "0.01368242", // QuoteVolume / 成交量
        "openPrice": "0.01298900",
        "highPrice": "0.01418800",
        "lowPrice": "0.01296000",
        "lastPrice": "0.01360400",
        "volume": "587179.23900000",
        "quoteVolume": "8034.03382165",   // 此k线内所有交易的price(价格) x volume(交易量)的总和
        "openTime": 1659580020000,        // ticker的开始时间
        "closeTime": 1660184865291,       // ticker的结束时间
        "firstId": 192977765,             // 统计时间内的第一笔trade id
        "lastId": 195365758,
        "count": 2387994                  // 统计时间内交易笔数
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 4
        }
    ]
}
```

`MINI` 类型，对于单个交易对：

```javascript
{
    "id": "bdb7c503-542c-495c-b797-4d2ee2e91173",
    "status": 200,
    "result": {
        "symbol": "BNBBTC",
        "openPrice": "0.01298900",
        "highPrice": "0.01418800",
        "lowPrice": "0.01296000",
        "lastPrice": "0.01360400",
        "volume": "587179.23900000",
        "quoteVolume": "8034.03382165", // 此k线内所有交易的price(价格) x volume(交易量)的总和
        "openTime": 1659580020000,      // ticker的开始时间
        "closeTime": 1660184865291,     // ticker的结束时间
        "firstId": 192977765,           // 第一个交易 ID
        "lastId": 195365758,            // 最后交易 ID
        "count": 2387994                // 成交笔数
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 4 // 成交笔数
        }
    ]
}
```

如果请求是有多个交易对，响应会是数组类型:

```javascript
{
    "id": "f4b3b507-c8f2-442a-81a6-b2f12daa030f",
    "status": 200,
    "result": [
        {
            "symbol": "BNBBTC",
            "priceChange": "0.00061500",      // 价格变化
            "priceChangePercent": "4.735",    // 价格变化百分比
            "weightedAvgPrice": "0.01368242", // QuoteVolume / 成交量
            "openPrice": "0.01298900",
            "highPrice": "0.01418800",
            "lowPrice": "0.01296000",
            "lastPrice": "0.01360400",
            "volume": "587169.48600000",
            "quoteVolume": "8033.90114517",   // 此k线内所有交易的price(价格) x volume(交易量)的总和
            "openTime": 1659580020000,        // ticker的开始时间
            "closeTime": 1660184820927,       // ticker的结束时间
            "firstId": 192977765,             // 统计时间内的第一笔trade id
            "lastId": 195365700,
            "count": 2387936                  // 统计时间内交易笔数
        },
        {
            "symbol": "BTCUSDT",
            "priceChange": "1182.92000000",
            "priceChangePercent": "5.113",
            "weightedAvgPrice": "23349.27074846",
            "openPrice": "23135.33000000",
            "highPrice": "24491.22000000",
            "lowPrice": "22400.00000000",
            "lastPrice": "24318.25000000",
            "volume": "1039498.10978000",
            "quoteVolume": "24271522807.76838630",
            "openTime": 1659580020000,
            "closeTime": 1660184820927,
            "firstId": 1568787779,
            "lastId": 1604337406,
            "count": 35549628
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 8
        }
    ]
}
```

<a id="ticker.price"></a>
<a id="ticker-price"></a>

<a id="symbol-price-ticker"></a>

### 最新价格

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "ticker.price"
}
```
获取交易对最新价格

如果需要访问实时价格更新，请考虑使用 WebSocket Streams:

* `<symbol>@aggTrade`
* `<symbol>@trade`

**权重:**
根据请求的交易对数量进行调整：

|参数|提供Symbol数量|权重|
|---|---|---|
|symbol| 1 |2|
| |不提供| 4|
|symbols| 任意 |4|

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | NO | 获取单个交易对的价格
`symbols` | ARRAY OF STRING | NO | 获取多个交易对的价格
`symbolStatus` | ENUM | NO | 过滤具有此 `tradingStatus` 的交易对。对于单个交易对，如果状态不匹配，将返回错误 `-1220 交易对与状态不匹配`。对于多个或者全部交易对，响应中不会包括状态不匹配的交易对。有效值： `TRADING`, `HALT`, `BREAK`

**数据源:** 缓存

备注：

* `symbol` 和 `symbols` 不能一起使用。

* 如果未指定交易对，则返回有关当前在交易所交易的所有交易对的信息。

**响应 - 当查询单个交易对时:**
```javascript
{
    "id": "043a7cf2-bde3-4888-9604-c8ac41fcba4d",
    "status": 200,
    "result": {
        "symbol": "BNBBTC",
        "price": "0.01361900"
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

**响应 - 当查询多个交易对时:**
```javascript
{
    "id": "e739e673-24c8-4adf-9cfa-b81f30330b09",
    "status": 200,
    "result": [
        {
            "symbol": "BNBBTC",
            "price": "0.01363700"
        },
        {
            "symbol": "BTCUSDT",
            "price": "24267.15000000"
        },
        {
            "symbol": "BNBBUSD",
            "price": "331.10000000"
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 4
        }
    ]
}
```

<a id="ticker.book"></a>
<a id="ticker-book"></a>

<a id="symbol-order-book-ticker"></a>

### 当前最优挂单

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "ticker.book"
}
```
在订单簿获取当前最优价格和数量。

如果您需要访问实时订单簿 ticker 更新，请考虑使用 WebSocket Streams:

* [`<symbol>@bookTicker`](web-socket-streams_CN.md#individual-symbol-book-ticker-streams)

**权重:**
根据请求的交易对数量进行调整：

|参数|提供Symbol数量|权重|
|---|---|---|
|symbol| 1 |2|
| |不提供| 4|
|symbols| 任意 |4|

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | NO | 获取单个交易对的ticker
`symbols` | ARRAY OF STRING | NO | 获取多个交易对的ticker
`symbolStatus` | ENUM | NO | 过滤具有此 `tradingStatus` 的交易对。对于单个交易对，如果状态不匹配，将返回错误 `-1220 交易对与状态不匹配`。对于多个或者全部交易对，响应中不会包括状态不匹配的交易对。有效值： `TRADING`, `HALT`, `BREAK`

**数据源:** 缓存

备注：

* `symbol` 和 `symbols` 不能一起使用。

* 如果未指定交易对，则返回有关当前在交易所交易的所有交易对的信息。

**响应 - 当查询单个交易对时:**
```javascript
{
    "id": "9d32157c-a556-4d27-9866-66760a174b57",
    "status": 200,
    "result": {
        "symbol": "BNBBTC",
        "bidPrice": "0.01358000", // 最优买单价
        "bidQty": "12.53400000",  // 挂单量
        "askPrice": "0.01358100", // 最优卖单价
        "askQty": "17.83700000"   // 挂单量
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

**响应 - 当查询多个交易对时:**
```javascript
{
    "id": "057deb3a-2990-41d1-b58b-98ea0f09e1b4",
    "status": 200,
    "result": [
        {
            "symbol": "BNBBTC",
            "bidPrice": "0.01358000", // 最优买单价
            "bidQty": "12.53400000",  // 挂单量
            "askPrice": "0.01358100", // 最优卖单价
            "askQty": "17.83700000"   // 挂单量
        },
        {
            "symbol": "BTCUSDT",
            "bidPrice": "23980.49000000",
            "bidQty": "0.01000000",
            "askPrice": "23981.31000000",
            "askQty": "0.01512000"
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 4
        }
    ]
}
```

<a id="referencePrice"></a>
<a id="reference-price"></a>
<a id="referenceprice"></a>

<a id="query-reference-price"></a>

### 查询参考价格

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "referencePrice",
    "params": {
        "symbol": "BAZUSD"
    }
}
```

**权重:**
2

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | YES |

**数据源:** 缓存

如果该交易对从未设置过参考价格，则请求会被拒绝并返回：

```json
{
    "code": -2043,
    "msg": "This symbol doesn't have a reference price."
}
```

**响应:**

如果设置了参考价格：

```javascript
{
    "id": "ddbfb65f-9ebf-42ec-8240-8f0f91de0867",
    "status": 200,
    "result": {
        "symbol": "BAZUSD",
        "referencePrice": "0.00501900",
        "timestamp": 1770946889251 // 参考价格生效的时间
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

如果未设置参考价格：

```javascript
{
    "id": "5132affb-0aba-4821-b475-f262504556b43",
    "status": 200,
    "result": {
        "symbol": "BAZUSD",
        "referencePrice": null,
        "timestamp": 1770946889251 // 参考价格生效的时间
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

如果从未设置过参考价格：

```javascript
{
    "id": "5132affa-0aba-4831-b475-f262504556b41",
    "status": 200,
    "result": {
        "code": -2043,
        "msg": "This symbol doesn't have a reference price."
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

<a id="referencePrice.calculation"></a>
<a id="referencePrice-calculation"></a>
<a id="reference-price-calculation"></a>
<a id="referenceprice-calculation"></a>

<a id="query-reference-price-calculation"></a>

### 查询参考价格计算方式

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "referencePrice.calculation",
    "params": {
        "symbol": "BAZUSD"
    }
}
```

**权重:**
2

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`symbol` | STRING | YES |
`symbolStatus` | ENUM | NO |

**数据源:** 缓存

**响应:**

如果参考价格由撮合引擎以算术平均数计算：

```javascript
{
    "id": "ddbfb65f-9ebf-42ec-8240-8f0f91de0867",
    "status": 200,
    "result": {
        "symbol": "BAZUSD",
        "calculationType": "ARITHMETIC_MEAN",
        "bucketCount": 10,          // 如果参考价格由撮合引擎以算术平均数计算
        "bucketWidthMs": 1000,      // 如果参考价格由撮合引擎以算术平均数计算
        "externalCalculationId": 42 // 如果参考价格由撮合引擎外部计算
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

如果参考价格由撮合引擎外部计算：

```javascript
{
    "id": "5132affa-0aba-4831-b475-f262504556b41",
    "status": 200,
    "result": {
        "symbol": "BAZUSD",
        "calculationType": "EXTERNAL",
        "externalCalculationId": 42 // 如果参考价格由撮合引擎外部计算
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

如果参考价格未被计算：

```javascript
{
    "id": "5132affa-0aba-4831-b475-f262504556b41",
    "status": 400,
    "error": {
        "code": -2043,
        "msg": "This symbol doesn't have a reference price."
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 2
        }
    ]
}
```

<a id="authentication-requests"></a>

## 身份验证请求

<a id="session.status"></a>
<a id="session-status"></a>

<a id="query-session-status"></a>

### 查询会话状态

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "session.status"
}
```
查询WebSocket连接的状态，检查用于授权请求的API密钥（如果有的话）。

**权重:**
2

**参数:**
NONE

**数据源:** 缓存

**响应:**
```javascript
{
    "id": "b50c16cd-62c9-4e29-89e4-37f10111f5bf",
    "status": 200,
    "result": {
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A", // 如果连接未经过身份验证，则为 `null`。
        "authorizedSince": 1649729878532,                                             // 如果连接未经过身份验证，则为 `null`。
        "connectedSince": 1649729873021,
        "returnRateLimits": false,
        "serverTime": 1649730611671,
        "userDataStream": true                                                        // User Data Stream 订阅是否有效？
    }
}
```

<a id="session.logout"></a>
<a id="session-logout"></a>

<a id="log-out-of-the-session"></a>

### 退出会话

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "session.logout"
}
```
忘记之前认证的API密钥。
如果连接未经身份验证，此请求不会有任何作用。

请注意，`session.logout`请求后，WebSocket连接仍然保持打开状态。
你可以继续使用连接，但现在必须在需要的地方明确提供`apiKey`和`signature`参数。

**权重:**
2

**参数:**
NONE

**数据源:** 缓存

**响应:**
```javascript
{
    "id": "c174a2b1-3f51-4580-b200-8528bd237cb7",
    "status": 200,
    "result": {
        "apiKey": null,
        "authorizedSince": null,
        "connectedSince": 1649729873021,
        "returnRateLimits": false,
        "serverTime": 1649730611671,
        "userDataStream": false // User Data Stream 订阅是否有效？
    }
}
```

<a id="session.logon"></a>
<a id="session-logon"></a>
<a id="log-in-with-api-key-signed"></a>

<a id="log-in-with-api-key-user_data"></a>

### 用API key登录 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "session.logon",
    "params": {
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```
使用提供的API密钥进行WebSocket连接身份验证。

在调用`session.logon`后，将来的需要`apiKey`和`signature`参数的请求可以省略它们。

请注意，只能认证一个API密钥。
多次调用`session.logon`将更改当前已认证的API密钥。

**注意：** 仅支持 Ed25519 密钥用于此功能。

**权重:**
2

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`recvWindow` | DECIMAL | NO | 最大值为 `60000` 毫秒。 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 缓存

**响应:**
```javascript
{
    "id": "c174a2b1-3f51-4580-b200-8528bd237cb7",
    "status": 200,
    "result": {
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "authorizedSince": 1649729878532,
        "connectedSince": 1649729873021,
        "returnRateLimits": false,
        "serverTime": 1649729878630,
        "userDataStream": false // User Data Stream 订阅是否有效？
    }
}
```

<a id="trading-requests"></a>

## 交易请求

<a id="order.place"></a>
<a id="order-place"></a>

<a id="place-new-order-trade"></a>

### 下新的订单 (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "order.place",
    "params": {
        "symbol": "BTCUSDT",
        "side": "SELL",
        "type": "LIMIT",
        "timeInForce": "GTC",
        "price": "23416.10000000",
        "quantity": "0.00847000",
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
下新的订单。

这个请求会把1个订单添加到 `EXCHANGE_MAX_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器中。

**权重:**
1

**未成交的订单计数:**
1

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`side` | ENUM | YES | 详见枚举定义：[订单方向](enums_CN.md#side)
`type` | ENUM | YES | 详见枚举定义：[订单类型](enums_CN.md#ordertypes)
`timeInForce` | ENUM | NO | 详见枚举定义：[生效时间](enums_CN.md#timeinforce)
`price` | DECIMAL | NO |
`quantity` | DECIMAL | NO |
`quoteOrderQty` | DECIMAL | NO |
`newClientOrderId` | STRING | NO | 用户自定义的orderid，如空缺系统会自动赋值。
`newOrderRespType` | ENUM | NO | `MARKET` 与 `LIMIT` 订单默认为`FULL`, 其他默认为`ACK`。
`stopPrice` | DECIMAL | NO | 仅 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT` 需要此参数。
`trailingDelta` | INT | NO | 参见 追踪止盈止损(Trailing Stop)订单常见问题
`icebergQty` | DECIMAL | NO | 仅有限价单(包括条件限价单与限价做事单)可以使用该参数，含义为创建冰山订单并指定冰山订单的数量。
`strategyId` | LONG | NO |
`strategyType` | INT | NO | 不能低于 `1000000`。
`selfTradePreventionMode` | ENUM | NO | 允许的 ENUM 取决于交易对的配置。
`pegPriceType` | ENUM | NO | 参阅 关于挂钩订单参数的注意事项
`pegOffsetValue` | INT | NO | 用于挂钩的价格水平（最大值：100）。参阅 关于挂钩订单参数的注意事项
`pegOffsetType` | ENUM | NO | 仅支持 `PRICE_LEVEL`。参阅 关于挂钩订单参数的注意事项
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

根据订单 `type`，<a id="order-type">某些参数(*)</a> 可能成为必需参数:

<table>
<thead>
    <tr>
        <th>订单 <code>type</code></th>
        <th>强制要求的参数</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td><code>LIMIT</code></td>
        <td>
        <ul>
            <li><code>timeInForce</code></li>
            <li><code>price</code></li>
            <li><code>quantity</code></li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>LIMIT_MAKER</code></td>
        <td>
        <ul>
            <li><code>price</code></li>
            <li><code>quantity</code></li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>MARKET</code></td>
        <td>
        <ul>
            <li><code>quantity</code> 或者 <code>quoteOrderQty</code></li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>STOP_LOSS</code></td>
        <td>
        <ul>
            <li><code>quantity</code></li>
            <li><code>stopPrice</code> 或者 <code>trailingDelta</code></li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>STOP_LOSS_LIMIT</code></td>
        <td>
        <ul>
            <li><code>timeInForce</code></li>
            <li><code>price</code></li>
            <li><code>quantity</code></li>
            <li><code>stopPrice</code> 或者 <code>trailingDelta</code></li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>TAKE_PROFIT</code></td>
        <td>
        <ul>
            <li><code>quantity</code></li>
            <li><code>stopPrice</code> 或者 <code>trailingDelta</code></li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>TAKE_PROFIT_LIMIT</code></td>
        <td>
        <ul>
            <li><code>timeInForce</code></li>
            <li><code>price</code></li>
            <li><code>quantity</code></li>
            <li><code>stopPrice</code> 或者 <code>trailingDelta</code></li>
        </ul>
        </td>
    </tr>
</tbody>
</table>

支持的订单类型：

<table>
<thead>
    <tr>
        <th>订单 <code>type</code></th>
        <th>描述</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td><code>LIMIT</code></td>
        <td>
        <p>
            以指定的 <code>price</code> 或更好的 <code>price</code>, 买入或卖出 <code>quantity</code>。
        </p>
        </td>
    </tr>
    <tr>
        <td><code>LIMIT_MAKER</code></td>
        <td>
        <p>
            <code>LIMIT</code> 订单，如果它立即匹配并成为吃单方将被拒绝。
        </p>
        <p>
            此订单类型也称为 POST-ONLY 订单。
        </p>
        </td>
    </tr>
    <tr>
        <td><code>MARKET</code></td>
        <td>
        <p>
            以最佳市场价格买入或卖出。
        </p>
        <ul>
            <li>
                <p>
                    带有 <code>quantity</code> 参数的 <code>MARKET</code> 订单指定您要BUY或SELL的<em>base asset</em>的数量。
                    报价资产的实际执行quantity将取决于可用的市场流动性。
                </p>
                <p>
                    例如，在 BTCUSDT 下<code> "quantity": "0.1000" </code> 的市场买单，指定您想以最优惠的价格 BUY 0.1 BTC。
                    如果以最优价格没有足够的 BTC，会继续以次优价格买入，直到您的订单被执行，或者您的 USDT 用完，或者市场上的 BTC 用完。
                </p>
            </li>
            <li>
                <p>
                    使用 <code>quoteOrderQty</code> 的 <code>MARKET</code> 订单 明确的是通过买入(或卖出)想要花费(或获取)的 <em>quote asset</em> 数量。
                    基础资产的实际执行数量将取决于可用的市场流动性。
                </p>
                <p>
                    例如，在 BTCUSDT 下 <code> "quoteOrderQty": "100.00" </code> 的市场买单，指定您想以最优惠的价格以 100 USDT 购买尽可能多的 BTC。
                    同样，卖出订单将卖出尽可能多的可用 BTC，以便您获得 100 USDT（佣金前）。
                </p>
            </li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>STOP_LOSS</code></td>
        <td>
        <p>
            当满足指定条件时，执行给定 <code>quantity</code> 的 <code>MARKET</code> 订单。
        </p>
        <p>
            I.e., 当达到 <code>stopPrice</code> 或激活 <code>trailingDelta</code> 时。
        </p>
        </td>
    </tr>
    <tr>
        <td><code>STOP_LOSS_LIMIT</code></td>
        <td>
        <p>
            当满足指定条件时，执行给定参数的 <code>LIMIT</code> 订单。
        </p>
        </td>
    </tr>
    <tr>
        <td><code>TAKE_PROFIT</code></td>
        <td>
        <p>
            与 <code>STOP_LOSS</code> 类似，但在市场价格向有利方向移动时激活。
        </p>
        </td>
    </tr>
    <tr>
        <td><code>TAKE_PROFIT_LIMIT</code></td>
        <td>
        <p>
            与 <code>STOP_LOSS_LIMIT</code> 类似，但在市场价格向有利方向移动时激活。
        </p>
        </td>
    </tr>
</tbody>
</table>

<a id="pegged-orders-info"></a>
关于挂钩订单参数的注意事项：

* 这些参数仅适用于 `LIMIT`， `LIMIT_MAKER`， `STOP_LOSS_LIMIT` 和 `TAKE_PROFIT_LIMIT` 订单。
* 如果使用了 `pegPriceType`， 那么 `price` 字段将是可选的。 否则，`price` 字段依旧是必须的。
* `pegPriceType=PRIMARY_PEG` 就是主要挂钩（`primary`），这是订单簿上与您的订单同一方向的最佳价格。
* `pegPriceType=MARKET_PEG` 就是市场挂钩（`market`），这是订单簿上与您的订单相反方向的最佳价格。
* 可以通过使用 `pegOffsetType` 和 `pegOffsetValue` 来获取最佳价格以外的价格水平。 这两个参数必须一起使用。

<a id="timeInForce"></a>

可用的 `timeInForce` 选项，设置订单在到期前应该活跃多长时间：

TIF  | 描述
----- | --------------
`GTC` | **Good 'til Canceled** – 成交为止。订单会一直有效，直到被成交或者取消。
`IOC` | **Immediate 或者 Cancel** – 无法立即成交的部分就撤销。订单在失效前会尽量多的成交。
`FOK` | **Fill 或者 Kill** – 无法全部立即成交就撤销。如果无法全部成交，订单会失效。

备注：

* `newClientOrderId` 指定订单的 `clientOrderId` 值。

  仅当前一个订单已成交或过期时，才会接受具有相同 `clientOrderId` 的新订单。

* 任何 `LIMIT` 或 `LIMIT_MAKER` 订单都可以通过指定 `icebergQty` 变成冰山订单。

  带有 `icebergQty` 的订单必须将 `timeInForce` 设置为 `GTC`。

* `STOP_LOSS`/`TAKE_PROFIT` 订单的触发订单价格规则：

  * `stopPrice` 必须高于市场价格：`STOP_LOSS BUY`，`TAKE_PROFIT SELL`
  * `stopPrice` 必须低于市场价格：`STOP_LOSS SELL`，`TAKE_PROFIT BUY`

* 使用 `quoteOrderQty` 的 `MARKET` 订单遵循 [`LOT_SIZE`](filters_CN.md#lot_size) 过滤规则。

  该订单将执行一个名义价值尽可能接近请求的 `quoteOrderQty` 的数量。

**响应:**

使用 `newOrderRespType` 参数可以选择响应格式。
`ACK` 响应类型：

```javascript
{
    "id": "56374a46-3061-486b-a311-99ee972eb648",
    "status": 200,
    "result": {
        "symbol": "BTCUSDT",
        "orderId": 12569099453,
        "orderListId": -1, // 单个订单会一直是 -1
        "clientOrderId": "4d96324ff9d44481926157ec08158a40",
        "transactTime": 1660801715639
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 1
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 1
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

`RESULT` 响应类型：

```javascript
{
    "id": "56374a46-3061-486b-a311-99ee972eb648",
    "status": 200,
    "result": {
        "symbol": "BTCUSDT",
        "orderId": 12569099453,
        "orderListId": -1, // 单个订单会一直是 -1
        "clientOrderId": "4d96324ff9d44481926157ec08158a40",
        "transactTime": 1660801715639,
        "price": "23416.10000000",
        "origQty": "0.00847000",
        "executedQty": "0.00000000",
        "origQuoteOrderQty": "0.000000",
        "cummulativeQuoteQty": "0.00000000",
        "status": "NEW",
        "timeInForce": "GTC",
        "type": "LIMIT",
        "side": "SELL",
        "workingTime": 1660801715639,
        "selfTradePreventionMode": "NONE"
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 1
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 1
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

`FULL` 响应类型：

```javascript
{
    "id": "56374a46-3061-486b-a311-99ee972eb648",
    "status": 200,
    "result": {
        "symbol": "BTCUSDT",
        "orderId": 12569099453,
        "orderListId": -1,
        "clientOrderId": "4d96324ff9d44481926157ec08158a40",
        "transactTime": 1660801715793,
        "price": "23416.10000000",
        "origQty": "0.00847000",
        "executedQty": "0.00847000",
        "origQuoteOrderQty": "0.000000",
        "cummulativeQuoteQty": "198.33521500",
        "status": "FILLED",
        "timeInForce": "GTC",
        "type": "LIMIT",
        "side": "SELL",
        "workingTime": 1660801715639,
        "selfTradePreventionMode": "NONE",
        "stopPrice": "0.00000000",                // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
        "trailingDelta": 10,                      // 订单激活前所需的价格变动。
        "icebergQty": "0.00000000",               // 仅在请求中发送了 icebergQty 参数时出现。
        "strategyId": 1,                          // 仅在下单时提供了 strategyId 参数时出现。
        "strategyType": 1000000,                  // 仅在下单时提供了 strategyType 参数时出现。
        "preventedMatchId": 0,                    // 仅在因 STP 导致订单失效时出现。
        "preventedQuantity": "1.200000",          // 因 STP 导致订单失效的数量。
        "trailingTime": -1,                       // 追踪单被激活和跟踪价格变化的时间。
        "usedSor": true,                          // 用于确定订单是否使用SOR的字段。
        "workingFloor": "SOR",                    // 用以定义订单是通过 SOR 还是由订单簿成交的。
        "pegPriceType": "PRIMARY_PEG",            // 挂钩价格类型。仅用于挂钩订单。
        "pegOffsetType": "PRICE_LEVEL",           // 挂钩价格偏移类型。如若需要，仅用于挂钩订单。
        "pegOffsetValue": 5,                      // 挂钩价格偏移值。如若需要，仅用于挂钩订单。
        "peggedPrice": "87523.83710000",          // 订单对应的当前挂钩价格。一旦确定，仅用于挂钩订单。
        "expiryReason": "INSUFFICIENT_LIQUIDITY", // 订单过期的原因。当订单过期时出现。
        // FULL 响应与 RESULT 响应相同，具有相同的可选字段基于订单类型和参数。FULL响应还包括立即完成订单的交易列表。
        "fills": [
            {
                "price": "23416.10000000",
                "qty": "0.00635000",
                "commission": "0.000000",
                "commissionAsset": "BNB",
                "tradeId": 1650422481
            },
            {
                "price": "23416.50000000",
                "qty": "0.00212000",
                "commission": "0.000000",
                "commissionAsset": "BNB",
                "tradeId": 1650422482
            }
        ]
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 1
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 1
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

<a id="order.test"></a>
<a id="order-test"></a>

<a id="test-new-order-trade"></a>

### 测试下单 (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "order.test",
    "params": {
        "symbol": "BTCUSDT",
        "side": "SELL",
        "type": "LIMIT",
        "timeInForce": "GTC",
        "price": "23416.10000000",
        "quantity": "0.00847000",
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
测试下单。

验证新订单参数并验证您的签名但不会将订单发送到撮合引擎。

**权重:**
| 条件 | 请求权重 |
| --- | --- |
| 没有 `computeCommissionRates` | 1 |
| 有 `computeCommissionRates` | 20 |

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`computeCommissionRates` | BOOLEAN | NO | 默认值: `false` <br> 请参阅[佣金常见问题解答](faqs/commission_faq_CN.md#test-order-diferences) 了解更多信息。
`symbol` | STRING | YES |
`side` | ENUM | YES | 详见枚举定义：[订单方向](enums_CN.md#side)
`type` | ENUM | YES | 详见枚举定义：[订单类型](enums_CN.md#ordertypes)
`timeInForce` | ENUM | NO | 详见枚举定义：[生效时间](enums_CN.md#timeinforce)
`price` | DECIMAL | NO |
`quantity` | DECIMAL | NO |
`quoteOrderQty` | DECIMAL | NO |
`newClientOrderId` | STRING | NO | 用户自定义的orderid，如空缺系统会自动赋值。
`newOrderRespType` | ENUM | NO | 指定响应类型 `ACK`, `RESULT`, or `FULL`; `MARKET` 与 `LIMIT` 订单默认为`FULL`, 其他默认为`ACK`。
`stopPrice` | DECIMAL | NO | 仅 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT` 需要此参数。
`trailingDelta` | INT | NO | 参见 [追踪止盈止损(Trailing Stop)订单常见问题](faqs/trailing-stop-faq_CN.md)
`icebergQty` | DECIMAL | NO | 仅有限价单(包括条件限价单与限价做事单)可以使用该参数，含义为创建冰山订单并指定冰山订单的数量。
`strategyId` | LONG | NO |
`strategyType` | INT | NO | 不能低于 `1000000`。
`selfTradePreventionMode` | ENUM | NO | 允许的 ENUM 取决于交易对的配置。支持的值有：[STP 模式](enums_CN.md#stpmodes)。
`pegPriceType` | ENUM | NO | `PRIMARY_PEG` 或 `MARKET_PEG`。 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`pegOffsetValue` | INT | NO | 用于挂钩的价格水平（最大值：100）。 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`pegOffsetType` | ENUM | NO | 仅支持 `PRICE_LEVEL`。 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 缓存

**响应:**

没有 `computeCommissionRates`:

```javascript
{
    "id": "6ffebe91-01d9-43ac-be99-57cf062e0e30",
    "status": 200,
    "result": {},
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

有 `computeCommissionRates`:

```javascript
{
    "id": "6ffebe91-01d9-43ac-be99-57cf062e0e30",
    "status": 200,
    "result": {
        // 订单交易的标准佣金率。
        "standardCommissionForOrder": {
            "maker": "0.00000112",
            "taker": "0.00000114"
        },
        // 订单交易的特殊佣金率。
        "specialCommissionForOrder": {
            "maker": "0.05000000",
            "taker": "0.06000000"
        },
        // 订单交易的税率。
        "taxCommissionForOrder": {
            "maker": "0.00000112",
            "taker": "0.00000114"
        },
        // 以BNB支付时的标准佣金折扣。
        "discount": {
            "enabledForAccount": true,
            "enabledForSymbol": true,
            "discountAsset": "BNB",
            "discount": "0.25000000" // 当用BNB支付佣金时，在标准佣金上按此比率打折。
        }
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 20
        }
    ]
}
```

<a id="order.cancel"></a>
<a id="order-cancel"></a>

<a id="cancel-order-trade"></a>

### 撤销订单 (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "order.cancel",
    "params": {
        "symbol": "BTCUSDT",
        "origClientOrderId": "4d96324ff9d44481926157",
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
取消有效订单。

**权重:**
1

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`orderId` | LONG | NO |
`origClientOrderId` | STRING | NO |
`newClientOrderId` | STRING | NO | 用户自定义的本次撤销操作的ID(注意不是被撤销的订单的自定义ID)。如无指定会自动赋值。
`cancelRestrictions` | ENUM | NO | 支持的值: <br>`ONLY_NEW` - 如果订单状态为 `NEW`，撤销将成功。<br> `ONLY_PARTIALLY_FILLED` - 如果订单状态为 `PARTIALLY_FILLED`，撤销将成功。 请参阅[关于 `cancelRestrictions`](#regarding-cancelrestrictions) 了解更多信息。
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

备注：

* 当同时提供 `orderId` 和 `origClientOrderId` 两个参数时，系统首先将会使用 `orderId` 来搜索订单。然后， 查找结果中的 `origClientOrderId` 的值将会被用来验证订单。如果两个条件都不满足，则请求将被拒绝。

* `newClientOrderId` 将替换已取消订单的 `clientOrderId`，为新订单腾出空间。

* 如果您取消属于订单列表的订单，则整个订单列表将被取消。

* 当仅发送 `orderId` 时,取消订单的执行(单个 Cancel 或作为 Cancel-Replace 的一部分)总是更快。发送 `origClientOrderId` 或同时发送 `orderId` + `origClientOrderId` 会稍慢。

**响应:**

取消单个订单时：

```javascript
{
    "id": "5633b6a2-90a9-4192-83e7-925c90b6a2fd",
    "status": 200,
    "result": {
        "symbol": "BTCUSDT",
        "origClientOrderId": "4d96324ff9d44481926157", // 被取消的 clientOrderId
        "orderId": 12569099453,
        "orderListId": -1,                             // 订单列表的ID，不然就是 -1
        "clientOrderId": "91fe37ce9e69c90d6358c0",     // 请求的 newClientOrderId
        "transactTime": 1684804350068,
        "price": "23416.10000000",
        "origQty": "0.00847000",
        "executedQty": "0.00001000",
        "origQuoteOrderQty": "0.000000",
        "cummulativeQuoteQty": "0.23416100",
        "status": "CANCELED",
        "timeInForce": "GTC",
        "type": "LIMIT",
        "side": "SELL",
        "stopPrice": "0.00000000",                     // 如果订单设置了 stopPrice 会出现
        "trailingDelta": 0,                            // 如果订单设置了 trailingDelta 会出现
        "icebergQty": "0.00000000",                    // 如果订单设置了 icebergQty 会出现
        "strategyId": 37463720,                        // 如果订单设置了 strategyId 会出现
        "strategyType": 1000000,                       // 如果订单设置了 strategyType 会出现
        "selfTradePreventionMode": "NONE"
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

取消订单列表时：

```javascript
{
    "id": "16eaf097-bbec-44b9-96ff-e97e6e875870",
    "status": 200,
    "result": {
        "symbol": "BTCUSDT",
        "origClientOrderId": "4d96324ff9d44481926157",
        "orderId": 12569099453,
        "orderListId": 19431,
        "clientOrderId": "91fe37ce9e69c90d6358c0",
        "transactTime": 1684804350068,
        "price": "23416.10000000",
        "origQty": "0.00847000",
        "executedQty": "0.00001000",
        "origQuoteOrderQty": "0.000000",
        "cummulativeQuoteQty": "0.23416100",
        "status": "CANCELED",
        "timeInForce": "GTC",
        "type": "LIMIT",
        "side": "SELL",
        "stopPrice": "0.00000000",                // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
        "trailingDelta": 10,                      // 订单激活前所需的价格变动。
        "icebergQty": "0.00000000",               // 仅在请求中发送了 icebergQty 参数时出现。
        "strategyId": 1,                          // 仅在下单时提供了 strategyId 参数时出现。
        "strategyType": 1000000,                  // 仅在下单时提供了 strategyType 参数时出现。
        "selfTradePreventionMode": "NONE",
        "preventedMatchId": 0,                    // 仅在因 STP 导致订单失效时出现。
        "preventedQuantity": "1.200000",          // 因 STP 导致订单失效的数量。
        "trailingTime": -1,                       // 追踪单被激活和跟踪价格变化的时间。
        "usedSor": true,                          // 用于确定订单是否使用SOR的字段。
        "workingFloor": "SOR",                    // 用以定义订单是通过 SOR 还是由订单簿成交的。
        "pegPriceType": "PRIMARY_PEG",            // 挂钩价格类型。仅用于挂钩订单。
        "pegOffsetType": "PRICE_LEVEL",           // 挂钩价格偏移类型。如若需要，仅用于挂钩订单。
        "pegOffsetValue": 5,                      // 挂钩价格偏移值。如若需要，仅用于挂钩订单。
        "peggedPrice": "87523.83710000",          // 订单对应的当前挂钩价格。一旦确定，仅用于挂钩订单。
        "expiryReason": "INSUFFICIENT_LIQUIDITY", // 订单过期的原因。当订单过期时出现。
        "contingencyType": "OCO",
        "listStatusType": "ALL_DONE",
        "listOrderStatus": "ALL_DONE",
        "listClientOrderId": "iuVNVJYYrByz6C4yGOPPK0",
        "transactionTime": 1660803702431,
        "orders": [
            {
                "symbol": "BTCUSDT",
                "orderId": 12569099453,
                "clientOrderId": "bX5wROblo6YeDwa9iTLeyY"
            },
            {
                "symbol": "BTCUSDT",
                "orderId": 12569099454,
                "clientOrderId": "Tnu2IP0J5Y4mxw3IATBfmW"
            }
        ],
        // 订单列表的状态格式与单个订单相同。
        "orderReports": [
            {
                "symbol": "BTCUSDT",
                "origClientOrderId": "bX5wROblo6YeDwa9iTLeyY",
                "orderId": 12569099453,
                "orderListId": 19431,
                "clientOrderId": "OFFXQtxVFZ6Nbcg4PgE2DA",
                "transactTime": 1684804350068,
                "price": "23450.50000000",
                "origQty": "0.00850000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "CANCELED",
                "timeInForce": "GTC",
                "type": "STOP_LOSS_LIMIT",
                "side": "BUY",
                "stopPrice": "23430.00000000", // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
                "selfTradePreventionMode": "NONE"
            },
            {
                "symbol": "BTCUSDT",
                "origClientOrderId": "Tnu2IP0J5Y4mxw3IATBfmW",
                "orderId": 12569099454,
                "orderListId": 19431,
                "clientOrderId": "OFFXQtxVFZ6Nbcg4PgE2DA",
                "transactTime": 1684804350068,
                "price": "23400.00000000",
                "origQty": "0.00850000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "CANCELED",
                "timeInForce": "GTC",
                "type": "LIMIT_MAKER",
                "side": "BUY",
                "selfTradePreventionMode": "NONE"
            }
        ]
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](web-socket-api_CN.md#conditional-fields-in-order-responses) 部分。

<a id="regarding-cancelrestrictions"></a>

**关于 `cancelRestrictions`**

* 如果 `cancelRestrictions` 值不是任何受支持的值，则错误将是：
```json
{
    "code": -1145,
    "msg": "Invalid cancelRestrictions"
}
```
* 如果订单没有通过 `cancelRestrictions` 的条件，错误将是：
```json
{
    "code": -2011,
    "msg": "Order was not canceled due to cancel restrictions."
}
```

<a id="order.cancelReplace"></a>
<a id="order-cancelReplace"></a>
<a id="order-cancel-replace"></a>
<a id="order-cancelreplace"></a>

<a id="cancel-and-replace-order-trade"></a>

### 撤消挂单再下单 (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "order.cancelReplace",
    "params": {
        "symbol": "BTCUSDT",
        "cancelReplaceMode": "ALLOW_FAILURE",
        "cancelOrigClientOrderId": "4d96324ff9d44481926157",
        "side": "SELL",
        "type": "LIMIT",
        "timeInForce": "GTC",
        "price": "23416.10000000",
        "quantity": "0.00847000",
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
* 撤消一个现有订单，并立即重新下单。
* 即使新订单未被尝试（即 `newOrderResult: NOT_ATTEMPTED`），未成交订单数量仍会增加1。
* 通过此接口只能撤消订单列表中的单个订单，但结果与撤消整个订单列表相同。

**权重:**
1

**未成交的订单计数:**
1

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`cancelReplaceMode` | ENUM | YES | 指定类型：`STOP_ON_FAILURE` - 如果撤消订单失败将不会继续重新下单。<br/> `ALLOW_FAILURE` - 不管撤消订单是否成功都会继续重新下单。
`cancelOrderId` | LONG | NO | 必须提供 `cancelOrderId` 或者 `cancelOrigClientOrderId`。 <br></br> 当同时提供 `cancelOrderId` 和 `cancelOrigClientOrderId` 两个参数时，系统首先将会使用 `cancelOrderId` 来搜索订单。<br></br> 然后， 查找结果中的 `cancelOrigClientOrderId` 的值将会被用来验证订单。<br></br> 如果两个条件都不满足，则请求将被拒绝。
`cancelOrigClientOrderId` | STRING | NO | 必须提供 `cancelOrderId` 或者 `cancelOrigClientOrderId`。 <br></br> 当同时提供 `cancelOrderId` 和 `cancelOrigClientOrderId` 两个参数时，系统首先将会使用 `cancelOrderId` 来搜索订单。<br></br> 然后， 查找结果中的 `cancelOrigClientOrderId` 的值将会被用来验证订单。<br></br> 如果两个条件都不满足，则请求将被拒绝。
`cancelNewClientOrderId` | STRING | NO | 用户自定义的id，如空缺系统会自动赋值
`side` | ENUM | YES | 详见枚举定义：[订单方向](enums_CN.md#side)
`type` | ENUM | YES | 详见枚举定义：[订单类型](enums_CN.md#ordertypes)
`timeInForce` | ENUM | NO | 详见枚举定义：[生效时间](enums_CN.md#timeinforce)
`price` | DECIMAL | NO |
`quantity` | DECIMAL | NO |
`quoteOrderQty` | DECIMAL | NO |
`newClientOrderId` | STRING | NO | 用于辨识新订单。
`newOrderRespType` | ENUM | NO | 指定响应类型: <br/> 指定响应类型 `ACK`, `RESULT`, or `FULL`; `MARKET` 与 `LIMIT` 订单默认为`FULL`， 其他默认为`ACK`。
`stopPrice` | DECIMAL | NO | 仅 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT` 需要此参数。
`trailingDelta` | DECIMAL | NO | 参考 [追踪止盈止损(Trailing Stop)订单常见问题](faqs/trailing-stop-faq_CN.md)
`icebergQty` | DECIMAL | NO | 仅有限价单(包括条件限价单与限价做事单)可以使用该参数，含义为创建冰山订单并指定冰山订单的数量。
`strategyId` | LONG | NO |
`strategyType` | INT | NO | 不能低于 `1000000`。
`selfTradePreventionMode` | ENUM | NO | 允许的 ENUM 取决于交易对的配置。支持的值有：[STP 模式](enums_CN.md#stpmodes)。
`cancelRestrictions` | ENUM | NO | 支持的值: <br>`ONLY_NEW` - 如果订单状态为 `NEW`，撤销将成功。<br> `ONLY_PARTIALLY_FILLED` - 如果订单状态为 `PARTIALLY_FILLED`，撤销将成功。 请参阅[关于 `cancelRestrictions`](#regarding-cancelrestrictions) 了解更多信息。
`orderRateLimitExceededMode` | ENUM | NO | 支持的值： <br></br> `DO_NOTHING`（默认值）- 仅在账户未超过未成交订单频率限制时，会尝试取消订单。<br></br> `CANCEL_ONLY` - 将始终取消订单。
`pegPriceType` | ENUM | NO | `PRIMARY_PEG` 或 `MARKET_PEG`。 <br> 参阅 关于挂钩订单参数的注意事项
`pegOffsetValue` | INT | NO | 用于挂钩的价格水平（最大值：100）。 <br> 参阅 关于挂钩订单参数的注意事项
`pegOffsetType` | ENUM | NO | 仅支持 `PRICE_LEVEL`。 <br> 参阅 关于挂钩订单参数的注意事项
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

类似于 [`order.place`](web-socket-api_CN.md#place-new-order-trade) 请求，额外的强制参数 (*) 由新订单的 `type` 确定。

可用的 `cancelReplaceMode` 选项：

* `STOP_ON_FAILURE` – 如果撤销订单请求失败，将不会尝试下新订单。
* `ALLOW_FAILURE` – 即使撤销订单请求失败，也会尝试下新订单。

<table>
<thead>
    <tr>
        <th colspan=3 align=left>请求</th>
        <th colspan=3 align=left>响应</th>
    </tr>
    <tr>
        <th><code>cancelReplaceMode</code></th>
        <th><code>orderRateLimitExceededMode</code></th>
        <th>未成交订单数</th>
        <th><code>cancelResult</code></th>
        <th><code>newOrderResult</code></th>
        <th><code>status</code></th>
    </tr>
</thead>
<tbody>
    <tr>
        <td rowspan="11"><code>STOP_ON_FAILURE</code></td>
        <td rowspan="6"><code>DO_NOTHING</code></td>
        <td rowspan="3">在限制范围内</td>
        <td>✅ <code>SUCCESS</code></td>
        <td>✅ <code>SUCCESS</code></td>
        <td align=right><code>200</code></td>
    </tr>
    <tr>
        <td>❌ <code>FAILURE</code></td>
        <td>➖ <code>NOT_ATTEMPTED</code></td>
        <td align=right><code>400</code></td>
    </tr>
    <tr>
        <td>✅ <code>SUCCESS</code></td>
        <td>❌ <code>FAILURE</code></td>
        <td align=right><code>409</code></td>
    </tr>
    <tr>
        <td rowspan="3">超出限制范围</td>
        <td>✅ <code>SUCCESS</code></td>
        <td>✅ <code>SUCCESS</code></td>
        <td align=right>N/A</td>
    </tr>
    <tr>
        <td>❌ <code>FAILURE</code></td>
        <td>➖ <code>NOT_ATTEMPTED</code></td>
        <td align=right>N/A</td>
    </tr>
    <tr>
        <td>✅ <code>SUCCESS</code></td>
        <td>❌ <code>FAILURE</code></td>
        <td align=right>N/A</td>
    </tr>
    <tr>
        <td rowspan="5"><code>CANCEL_ONLY</code></td>
        <td rowspan="3">在限制范围内</td>
        <td>✅ <code>SUCCESS</code></td>
        <td>✅ <code>SUCCESS</code></td>
        <td align=right><code>200</code></td>
    </tr>
    <tr>
        <td>❌ <code>FAILURE</code></td>
        <td>➖ <code>NOT_ATTEMPTED</code></td>
        <td align=right><code>400</code></td>
    </tr>
    <tr>
        <td>✅ <code>SUCCESS</code></td>
        <td>❌ <code>FAILURE</code></td>
        <td align=right><code>409</code></td>
    </tr>
    <tr>
        <td rowspan="2">超出限制范围</td>
        <td>❌ <code>FAILURE</code></td>
        <td>➖ <code>NOT_ATTEMPTED</code></td>
        <td align=right><code>429</code></td>
    </tr>
    <tr>
        <td>✅ <code>SUCCESS</code></td>
        <td>❌ <code>FAILURE</code></td>
        <td align=right><code>429</code></td>
    </tr>
    <tr>
        <td rowspan="16"><code>ALLOW_FAILURE</code></td>
        <td rowspan="8"><code>DO_NOTHING</code></td>
        <td rowspan="4">在限制范围内</td>
        <td>✅ <code>SUCCESS</code></td>
        <td>✅ <code>SUCCESS</code></td>
        <td align=right><code>200</code></td>
    </tr>
    <tr>
        <td>❌ <code>FAILURE</code></td>
        <td>❌ <code>FAILURE</code></td>
        <td align=right><code>400</code></td>
    </tr>
    <tr>
        <td>❌ <code>FAILURE</code></td>
        <td>✅ <code>SUCCESS</code></td>
        <td align=right><code>409</code></td>
    </tr>
    <tr>
        <td>✅ <code>SUCCESS</code></td>
        <td>❌ <code>FAILURE</code></td>
        <td align=right><code>409</code></td>
    </tr>
    <tr>
    <td rowspan="4">超出限制范围</td>
        <td>✅ <code>SUCCESS</code></td>
        <td>✅ <code>SUCCESS</code></td>
        <td align=right>N/A</td>
    </tr>
    <tr>
        <td>❌ <code>FAILURE</code></td>
        <td>❌ <code>FAILURE</code></td>
        <td align=right>N/A</td>
    </tr>
    <tr>
        <td>❌ <code>FAILURE</code></td>
        <td>✅ <code>SUCCESS</code></td>
        <td align=right>N/A</td>
    </tr>
    <tr>
        <td>✅ <code>SUCCESS</code></td>
        <td>❌ <code>FAILURE</code></td>
        <td align=right>N/A</td>
    </tr>
    <tr>
        <td rowspan="8"><CODE>CANCEL_ONLY</CODE></td>
        <td rowspan="4">在限制范围内</td>
        <td>✅ <code>SUCCESS</code></td>
        <td>✅ <code>SUCCESS</code></td>
        <td align=right><code>200</code></td>
    </tr>
    <tr>
        <td>❌ <code>FAILURE</code></td>
        <td>❌ <code>FAILURE</code></td>
        <td align=right><code>400</code></td>
    </tr>
    <tr>
        <td>❌ <code>FAILURE</code></td>
        <td>✅ <code>SUCCESS</code></td>
        <td align=right><code>409</code></td>
    </tr>
    <tr>
        <td>✅ <code>SUCCESS</code></td>
        <td>❌ <code>FAILURE</code></td>
        <td align=right><code>409</code></td>
    </tr>
    <tr>
        <td rowspan="4">超出限制范围</td>
        <td>✅ <code>SUCCESS</code></td>
        <td>✅ <code>SUCCESS</code></td>
        <td align=right><code>200</code></td>
    </tr>
    <tr>
        <td>❌ <code>FAILURE</code></td>
        <td>❌ <code>FAILURE</code></td>
        <td align=right><code>400</code></td>
    </tr>
    <tr>
        <td>❌ <code>FAILURE</code></td>
        <td>✅ <code>SUCCESS</code></td>
        <td align=right>N/A</td>
    </tr>
    <tr>
        <td>✅ <code>SUCCESS</code></td>
        <td>❌ <code>FAILURE</code></td>
        <td align=right><code>409</code></td>
    </tr>
</tbody>
</table>

备注：

* 当同时提供 `cancelOrderId` 和 `cancelOrigClientOrderId` 两个参数时，系统首先将会使用 `cancelOrderId` 来搜索订单。然后， 查找结果中的 `cancelOrigClientOrderId` 的值将会被用来验证订单。如果两个条件都不满足，则请求将被拒绝。

* `cancelNewClientOrderId` 将替换已撤销订单的 `clientOrderId`，为新订单腾出空间。

* `newClientOrderId` 指定下单的 `clientOrderId` 值。

  仅当前一个订单已成交或过期时，才会接受具有相同 `clientOrderId` 的新订单。

  新订单可以重用已取消订单的旧 `clientOrderId`。

* 此 cancel-replace 操作**不是事务性的**。

  如果一个操作成功但另一个操作失败，则仍然执行成功的操作。

  例如，在 `STOP_ON_FAILURE` 模式下，如果下新订单达失败，旧订单仍然被撤销。

* 过滤器和订单次数限制会在撤销和下订单之前评估。

* 如果未尝试下新订单，订单次数仍会增加。

* 与 [`order.cancel`](#order-cancel) 一样，如果您撤销订单列表内的某个订单，则整个订单列表将被撤销。

* 当仅发送 `orderId` 时,取消订单的执行(单个 Cancel 或作为 Cancel-Replace 的一部分)总是更快。发送 `origClientOrderId` 或同时发送 `orderId` + `origClientOrderId` 会稍慢。

**响应:**

如果撤销订单和下新订单都成功，响应会是 `"status": 200`：

```javascript
{
    "id": "99de1036-b5e2-4e0f-9b5c-13d751c93a1a",
    "status": 200,
    "result": {
        "cancelResult": "SUCCESS",
        "newOrderResult": "SUCCESS",
        // 格式与 "order.cancel" 格式相同。
        // 某些字段是可选的，仅在订单中有设置它们时才包括。
        "cancelResponse": {
            "symbol": "BTCUSDT",
            "origClientOrderId": "4d96324ff9d44481926157", // 请求的 cancelOrigClientOrderId
            "orderId": 125690984230,
            "orderListId": -1,
            "clientOrderId": "91fe37ce9e69c90d6358c0",     // 请求的 cancelNewClientOrderId
            "transactTime": 1684804350068,
            "price": "23450.00000000",
            "origQty": "0.00847000",
            "executedQty": "0.00001000",
            "origQuoteOrderQty": "0.000000",
            "cummulativeQuoteQty": "0.23450000",
            "status": "CANCELED",
            "timeInForce": "GTC",
            "type": "LIMIT",
            "side": "SELL",
            "selfTradePreventionMode": "NONE",
            "icebergQty": "0.00000000",                    // 仅在请求中发送了 icebergQty 参数时出现。
            "preventedMatchId": 0,                         // 仅在因 STP 导致订单失效时出现。
            "preventedQuantity": "1.200000",               // 因 STP 导致订单失效的数量。
            "stopPrice": "0.00000000",                     // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
            "strategyId": 1,                               // 仅在下单时提供了 strategyId 参数时出现。
            "strategyType": 1000000,                       // 仅在下单时提供了 strategyType 参数时出现。
            "trailingDelta": 10,                           // 订单激活前所需的价格变动。
            "trailingTime": -1,                            // 追踪单被激活和跟踪价格变化的时间。
            "usedSor": true,                               // 用于确定订单是否使用SOR的字段。
            "workingFloor": "SOR",                         // 用以定义订单是通过 SOR 还是由订单簿成交的。
            "pegPriceType": "PRIMARY_PEG",                 // 挂钩价格类型。仅用于挂钩订单。
            "pegOffsetType": "PRICE_LEVEL",                // 挂钩价格偏移类型。如若需要，仅用于挂钩订单。
            "pegOffsetValue": 5,                           // 挂钩价格偏移值。如若需要，仅用于挂钩订单。
            "peggedPrice": "87523.83710000",               // 订单对应的当前挂钩价格。一旦确定，仅用于挂钩订单。
            "expiryReason": "INSUFFICIENT_LIQUIDITY"       // 订单过期的原因。当订单过期时出现。
        },
        // 格式与 "order.place" 格式相同, 受 "newOrderRespType" 影响。
        // 某些字段是可选的，仅在订单中有设置它们时才包括。
        "newOrderResponse": {
            "symbol": "BTCUSDT",
            "orderId": 12569099453,
            "orderListId": -1,
            "clientOrderId": "bX5wROblo6YeDwa9iTLeyY",
            "transactTime": 1660813156959,
            "price": "23416.10000000",
            "origQty": "0.00847000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "NEW",
            "timeInForce": "GTC",
            "type": "LIMIT",
            "side": "SELL",
            "selfTradePreventionMode": "NONE",
            "icebergQty": "0.00000000",              // 仅在请求中发送了 icebergQty 参数时出现。
            "preventedMatchId": 0,                   // 仅在因 STP 导致订单失效时出现。
            "preventedQuantity": "1.200000",         // 因 STP 导致订单失效的数量。
            "stopPrice": "0.00000000",               // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
            "strategyId": 1,                         // 仅在下单时提供了 strategyId 参数时出现。
            "strategyType": 1000000,                 // 仅在下单时提供了 strategyType 参数时出现。
            "trailingDelta": 10,                     // 订单激活前所需的价格变动。
            "trailingTime": -1,                      // 追踪单被激活和跟踪价格变化的时间。
            "usedSor": true,                         // 用于确定订单是否使用SOR的字段。
            "workingFloor": "SOR",                   // 用以定义订单是通过 SOR 还是由订单簿成交的。
            "pegPriceType": "PRIMARY_PEG",           // 挂钩价格类型。仅用于挂钩订单。
            "pegOffsetType": "PRICE_LEVEL",          // 挂钩价格偏移类型。如若需要，仅用于挂钩订单。
            "pegOffsetValue": 5,                     // 挂钩价格偏移值。如若需要，仅用于挂钩订单。
            "peggedPrice": "87523.83710000",         // 订单对应的当前挂钩价格。一旦确定，仅用于挂钩订单。
            "expiryReason": "INSUFFICIENT_LIQUIDITY" // 订单过期的原因。当订单过期时出现。
        }
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 1
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 1
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

在 `STOP_ON_FAILURE` 模式，失败的撤销订单会阻止下新订单，响应会是`"status": 400`：

```javascript
{
    "id": "27e1bf9f-0539-4fb0-85c6-06183d36f66c",
    "status": 400,
    "error": {
        "code": -2022,
        "msg": "Order cancel-replace failed.",
        "data": {
            "cancelResult": "FAILURE",
            "newOrderResult": "NOT_ATTEMPTED",
            "cancelResponse": {
                "code": -2011,
                "msg": "Unknown order sent."
            },
            "newOrderResponse": null
        }
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 1
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 1
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

如果两个操作都失败，响应将有 `"status": 400`：

```javascript
{
    "id": "3b3ac45c-1002-4c7d-88e8-630c408ecd87",
    "status": 400,
    "error": {
        "code": -2022,
        "msg": "Order cancel-replace failed.",
        "data": {
            "cancelResult": "FAILURE",
            "newOrderResult": "FAILURE",
            "cancelResponse": {
                "code": -2011,
                "msg": "Unknown order sent."
            },
            "newOrderResponse": {
                "code": -2010,
                "msg": "Order would immediately match and take."
            }
        }
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 1
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 1
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

如果 cancel-replace 模式允许失败并且其中一个操作失败，响应会是 `"status": 409` 和 `"data"` 字段会制定哪个操作成功，哪个失败，以及原因：

撤销订单成功，下新订单失败：

```javascript
{
    "id": "b220edfe-f3c4-4a3a-9d13-b35473783a25",
    "status": 409,
    "error": {
        "code": -2021,
        "msg": "Order cancel-replace partially failed.",
        "data": {
            "cancelResult": "SUCCESS",
            "newOrderResult": "FAILURE",
            "cancelResponse": {
                "symbol": "BTCUSDT",
                "origClientOrderId": "4d96324ff9d44481926157", // 请求的 cancelOrigClientOrderId
                "orderId": 125690984230,
                "orderListId": -1,
                "clientOrderId": "91fe37ce9e69c90d6358c0",     // 请求的 cancelNewClientOrderId
                "transactTime": 1684804350068,
                "price": "23450.00000000",
                "origQty": "0.00847000",
                "executedQty": "0.00001000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.23450000",
                "status": "CANCELED",
                "timeInForce": "GTC",
                "type": "LIMIT",
                "side": "SELL",
                "selfTradePreventionMode": "NONE"
            },
            "newOrderResponse": {
                "code": -2010,
                "msg": "Order would immediately match and take."
            }
        }
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 1
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 1
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

撤销订单失败，下新订单成功：

```javascript
{
    "id": "ce641763-ff74-41ac-b9f7-db7cbe5e93b1",
    "status": 409,
    "error": {
        "code": -2021,
        "msg": "Order cancel-replace partially failed.",
        "data": {
            "cancelResult": "FAILURE",
            "newOrderResult": "SUCCESS",
            "cancelResponse": {
                "code": -2011,
                "msg": "Unknown order sent."
            },
            "newOrderResponse": {
                "symbol": "BTCUSDT",
                "orderId": 12569099453,
                "orderListId": -1,
                "clientOrderId": "bX5wROblo6YeDwa9iTLeyY",
                "transactTime": 1660813156959,
                "price": "23416.10000000",
                "origQty": "0.00847000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "NEW",
                "timeInForce": "GTC",
                "type": "LIMIT",
                "side": "SELL",
                "workingTime": 1669693344508,
                "fills": [],
                "selfTradePreventionMode": "NONE"
            }
        }
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 1
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 1
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

如果 `orderRateLimitExceededMode` 是 `CANCEL_ONLY`，那么无论 `cancelReplaceMode` 的取值，当账户超出未成交订单计数时，响应将有 `"status": 409`:

```javascript
{
    "id": "3b3ac45c-1002-4c7d-88e8-630c408ecd87",
    "status": 409,
    "error": {
        "code": -2021,
        "msg": "Order cancel-replace partially failed.",
        "data": {
            "cancelResult": "SUCCESS",
            "newOrderResult": "FAILURE",
            "cancelResponse": {
                "symbol": "LTCBNB",
                "origClientOrderId": "GKt5zzfOxRDSQLveDYCTkc", // 请求的 cancelOrigClientOrderId
                "orderId": 64,
                "orderListId": -1,
                "clientOrderId": "loehOJF3FjoreUBDmv739R",     // 请求的 cancelNewClientOrderId
                "transactTime": 1715779007228,
                "price": "1.00",
                "origQty": "10.00000000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.00",
                "status": "CANCELED",
                "timeInForce": "GTC",
                "type": "LIMIT",
                "side": "SELL",
                "selfTradePreventionMode": "NONE"
            },
            "newOrderResponse": {
                "code": -1015,
                "msg": "Too many new orders; current limit is 50 orders per 10 SECOND."
            }
        }
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 50
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 50
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

如果 `orderRateLimitExceededMode` 是 `DO_NOTHING`，那么无论 `cancelReplaceMode` 的取值，当账户超出未成交订单计数时，响应将有 `"status": 429`:

```javascript
{
    "id": "3b3ac45c-1002-4c7d-88e8-630c408ecd87",
    "status": 429,
    "error": {
        "code": -1015,
        "msg": "Too many new orders; current limit is 50 orders per 10 SECOND."
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 50
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 50
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](web-socket-api_CN.md#conditional-fields-in-order-responses) 部分。

<a id="order.amend.keepPriority"></a>
<a id="order-amend-keepPriority"></a>
<a id="order-amend-keep-priority"></a>
<a id="order-amend-keeppriority"></a>

<a id="order-amend-keep-priority-trade"></a>

### 修改订单并保留优先级 (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "order.amend.keepPriority",
    "params": {
        "symbol": "BTCUSDT",
        "origClientOrderId": "my_test_order1",
        "newQty": "5",
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
由客户发送以减少其现有当前挂单的原始数量。

这个请求会把0个订单添加到 `EXCHANGE_MAX_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器中。

请阅读 [保留优先权的修改订单常见问题](faqs/order_amend_keep_priority_CN.md) 了解更多信息。

**权重:**
4

**未成交的订单计数:**
0

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`orderId` | LONG | NO | 需提供 `orderId` 或 `origClientOrderId`。
`origClientOrderId` | STRING | NO | 需提供 `orderId` 或 `origClientOrderId`。
`newClientOrderId` | STRING | NO | 订单在被修改后被赋予的新 client order ID。 <br> 如果未发送则自动生成。 <br> 可以将当前 clientOrderId 作为 `newClientOrderId` 发送来重用当前 clientOrderId 的值。
`newQty` | DECIMAL | YES | 交易的新数量。 `newQty` 必须大于0, 但是必须比订单的原始数量小。
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

**响应:**

来自单个订单的响应：

```javascript
{
    "id": "56374a46-3061-486b-a311-89ee972eb648",
    "status": 200,
    "result": {
        "transactTime": 1741923284382,
        "executionId": 16,
        "amendedOrder": {
            "symbol": "BTCUSDT",
            "orderId": 12,
            "orderListId": -1,
            "origClientOrderId": "my_test_order1",
            "clientOrderId": "4zR9HFcEq8gM1tWUqPEUHc",
            "price": "5.00000000",
            "qty": "5.00000000",
            "executedQty": "0.00000000",
            "preventedQty": "0.00000000",
            "quoteOrderQty": "0.00000000",
            "cumulativeQuoteQty": "0.00000000",
            "status": "NEW",
            "timeInForce": "GTC",
            "type": "LIMIT",
            "side": "BUY",
            "workingTime": 1741923284364,
            "selfTradePreventionMode": "NONE"
        }
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

来自订单列表中单个订单的响应：

```javascript
{
    "id": "56374b46-3061-486b-a311-89ee972eb648",
    "status": 200,
    "result": {
        "transactTime": 1741924229819,
        "executionId": 60,
        "amendedOrder": {
            "symbol": "BTUCSDT",
            "orderId": 23,
            "orderListId": 4,
            "origClientOrderId": "my_pending_order",
            "clientOrderId": "xbxXh5SSwaHS7oUEOCI88B",
            "price": "1.00000000",
            "qty": "5.00000000",
            "executedQty": "0.00000000",
            "preventedQty": "0.00000000",
            "quoteOrderQty": "0.00000000",
            "cumulativeQuoteQty": "0.00000000",
            "status": "NEW",
            "timeInForce": "GTC",
            "type": "LIMIT",
            "side": "BUY",
            "workingTime": 1741924204920,
            "selfTradePreventionMode": "NONE",
            "icebergQty": "0.00000000",              // 仅在请求中发送了 icebergQty 参数时出现。
            "preventedMatchId": 0,                   // 仅在因 STP 导致订单失效时出现。
            "preventedQuantity": "1.200000",         // 因 STP 导致订单失效的数量。
            "stopPrice": "0.00000000",               // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
            "strategyId": 1,                         // 仅在下单时提供了 strategyId 参数时出现。
            "strategyType": 1000000,                 // 仅在下单时提供了 strategyType 参数时出现。
            "trailingDelta": 10,                     // 订单激活前所需的价格变动。
            "trailingTime": -1,                      // 追踪单被激活和跟踪价格变化的时间。
            "usedSor": true,                         // 用于确定订单是否使用SOR的字段。
            "workingFloor": "SOR",                   // 用以定义订单是通过 SOR 还是由订单簿成交的。
            "pegPriceType": "PRIMARY_PEG",           // 挂钩价格类型。仅用于挂钩订单。
            "pegOffsetType": "PRICE_LEVEL",          // 挂钩价格偏移类型。如若需要，仅用于挂钩订单。
            "pegOffsetValue": 5,                     // 挂钩价格偏移值。如若需要，仅用于挂钩订单。
            "peggedPrice": "87523.83710000",         // 订单对应的当前挂钩价格。一旦确定，仅用于挂钩订单。
            "expiryReason": "INSUFFICIENT_LIQUIDITY" // 订单过期的原因。当订单过期时出现。
        },
        "listStatus": {
            "orderListId": 4,
            "contingencyType": "OTO",
            "listOrderStatus": "EXECUTING",
            "listClientOrderId": "8nOGLLawudj1QoOiwbroRH",
            "symbol": "BTCUSDT",
            "orders": [
                {
                    "symbol": "BTCUSDT",
                    "orderId": 22,
                    "clientOrderId": "g04EWsjaackzedjC9wRkWD"
                },
                {
                    "symbol": "BTCUSDT",
                    "orderId": 23,
                    "clientOrderId": "xbxXh5SSwaHS7oUEOCI88B"
                }
            ]
        }
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 4
        }
    ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](web-socket-api_CN.md#conditional-fields-in-order-responses) 部分。

<a id="openOrders.cancelAll"></a>
<a id="openOrders-cancelAll"></a>
<a id="open-orders-cancel-all"></a>
<a id="openorders-cancelall"></a>

<a id="cancel-open-orders-trade"></a>

### 撤销单一交易对的所有挂单 (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "openOrders.cancelAll",
    "params": {
        "symbol": "BNBUSDT",
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```
撤销单一交易对的所有挂单,包括交易组。

**权重:**
1

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

**响应:**
```javascript
{
    "id": "778f938f-9041-4b88-9914-efbf64eeacc8",
    "status": 200,
    "result": [
        {
            "symbol": "BTCUSDT",
            "origClientOrderId": "4d96324ff9d44481926157",
            "orderId": 12569099453,
            "orderListId": -1,
            "clientOrderId": "91fe37ce9e69c90d6358c0",
            "transactTime": 1684804350068,
            "price": "23416.10000000",
            "origQty": "0.00847000",
            "executedQty": "0.00001000",
            "origQuoteOrderQty": "0.000000",
            "cummulativeQuoteQty": "0.23416100",
            "status": "CANCELED",
            "timeInForce": "GTC",
            "type": "LIMIT",
            "side": "SELL",
            "stopPrice": "0.00000000",  // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
            "trailingDelta": 0,         // 订单激活前所需的价格变动。
            "trailingTime": -1,         // 追踪单被激活和跟踪价格变化的时间。
            "icebergQty": "0.00000000", // 仅在请求中发送了 icebergQty 参数时出现。
            "strategyId": 37463720,     // 仅在下单时提供了 strategyId 参数时出现。
            "strategyType": 1000000,    // 仅在下单时提供了 strategyType 参数时出现。
            "selfTradePreventionMode": "NONE"
        },
        {
            "orderListId": 19431,
            "contingencyType": "OCO",
            "listStatusType": "ALL_DONE",
            "listOrderStatus": "ALL_DONE",
            "listClientOrderId": "iuVNVJYYrByz6C4yGOPPK0",
            "transactionTime": 1660803702431,
            "symbol": "BTCUSDT",
            "orders": [
                {
                    "symbol": "BTCUSDT",
                    "orderId": 12569099453,
                    "clientOrderId": "bX5wROblo6YeDwa9iTLeyY"
                },
                {
                    "symbol": "BTCUSDT",
                    "orderId": 12569099454,
                    "clientOrderId": "Tnu2IP0J5Y4mxw3IATBfmW"
                }
            ],
            "orderReports": [
                {
                    "symbol": "BTCUSDT",
                    "origClientOrderId": "bX5wROblo6YeDwa9iTLeyY",
                    "orderId": 12569099453,
                    "orderListId": 19431,
                    "clientOrderId": "OFFXQtxVFZ6Nbcg4PgE2DA",
                    "transactTime": 1684804350068,
                    "price": "23450.50000000",
                    "origQty": "0.00850000",
                    "executedQty": "0.00000000",
                    "origQuoteOrderQty": "0.000000",
                    "cummulativeQuoteQty": "0.00000000",
                    "status": "CANCELED",
                    "timeInForce": "GTC",
                    "type": "STOP_LOSS_LIMIT",
                    "side": "BUY",
                    "stopPrice": "23430.00000000", // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
                    "selfTradePreventionMode": "NONE"
                },
                {
                    "symbol": "BTCUSDT",
                    "origClientOrderId": "Tnu2IP0J5Y4mxw3IATBfmW",
                    "orderId": 12569099454,
                    "orderListId": 19431,
                    "clientOrderId": "OFFXQtxVFZ6Nbcg4PgE2DA",
                    "transactTime": 1684804350068,
                    "price": "23400.00000000",
                    "origQty": "0.00850000",
                    "executedQty": "0.00000000",
                    "origQuoteOrderQty": "0.000000",
                    "cummulativeQuoteQty": "0.00000000",
                    "status": "CANCELED",
                    "timeInForce": "GTC",
                    "type": "LIMIT_MAKER",
                    "side": "BUY",
                    "selfTradePreventionMode": "NONE"
                }
            ]
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](web-socket-api_CN.md#conditional-fields-in-order-responses) 部分。

<a id="orderList.place"></a>
<a id="orderList-place"></a>
<a id="order-list-place"></a>
<a id="orderlist-place"></a>
<a id="order-lists"></a>

<a id="place-new-oco---deprecated-trade"></a>

### OCO下单 - 已弃用 (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "orderList.place",
    "params": {
        "symbol": "BTCUSDT",
        "side": "SELL",
        "price": "23420.00000000",
        "quantity": "0.00650000",
        "stopPrice": "23410.00000000",
        "stopLimitPrice": "23405.00000000",
        "stopLimitTimeInForce": "GTC",
        "newOrderRespType": "RESULT",
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
发送新的OCO(one-cancels-the-other) 订单:
`LIMIT_MAKER` 订单 + `STOP_LOSS`/`STOP_LOSS_LIMIT` 订单(称呼为 *legs*), 其中一个订单的激活会立即取消另一个订单。

这个请求会把1个订单添加到 `EXCHANGE_MAX_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器中。

**权重:**
1

**未成交的订单计数:**
1

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`side` | ENUM | YES | 详见枚举定义：[订单方向](enums_CN.md#side)
`price` | DECIMAL | YES |
`quantity` | DECIMAL | YES |
`listClientOrderId` | STRING | NO | 整个orderList的唯一ID
`limitClientOrderId` | STRING | NO | 限价单的唯一ID
`limitIcebergQty` | DECIMAL | NO | 用于使 `LIMIT_MAKER` 腿单成为冰山订单。
`limitStrategyId` | LONG | NO |
`limitStrategyType` | INT | NO | 不能低于 `1000000`
`stopPrice` | DECIMAL | NO |
`trailingDelta` | INT | NO |
`stopClientOrderId` | STRING | NO | 止损/止损限价单的唯一ID
`stopLimitPrice` | DECIMAL | NO | 如果提供，须配合提交`stopLimitTimeInForce`
`stopLimitTimeInForce` | ENUM | NO | 有效值 `GTC`/`FOK`/`IOC`
`stopIcebergQty` | DECIMAL | NO | 与 `STOP_LOSS_LIMIT` 腿单一起使用，使其成为冰山订单。
`stopStrategyId` | LONG | NO |
`stopStrategyType` | INT | NO | 不能低于 `1000000`
`newOrderRespType` | ENUM | NO | JSON 响应格式。支持的数值：[订单返回类型](enums_CN.md#orderresponsetype)
`selfTradePreventionMode` | ENUM | NO | 允许的值取决于交易对的配置。支持的数值：[STP模式](enums_CN.md#stpmodes)
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

备注：

* `listClientOrderId` 参数指定 OCO 对的 `listClientOrderId`。

  只有当前一个 OCO 已满或完全过期时，才会接受具有相同 `listClientOrderId` 的新 OCO。

  `listClientOrderId` 与单个订单的 `clientOrderId` 不同。

* legs 的价格限制：

  | `side` | 价格关系 |
  | ------ | -------------- |
  | `BUY` | `price` < 市场价格 < `stopPrice` |
  | `SELL`| `price` > 市场价格 > `stopPrice` |

* 两个 legs 的 `quantity` 需要相同。

  不过单个 leg 可以设置不同的冰山数量。

  如果使用 `stopIcebergQty`，`stopLimitTimeInForce` 必须是 `GTC`。

* `trailingDelta` 仅适用于 OCO 的 `STOP_LOSS`/`STOP_LOSS_LIMIT` leg。

**响应:**
```javascript
{
    "id": "57833dc0-e3f2-43fb-ba20-46480973b0aa",
    "status": 200,
    "result": {
        "orderListId": 1274512,
        "contingencyType": "OCO",
        "listStatusType": "EXEC_STARTED",
        "listOrderStatus": "EXECUTING",
        "listClientOrderId": "08985fedd9ea2cf6b28996",
        "transactionTime": 1660801713793,
        "symbol": "BTCUSDT",
        "orders": [
            {
                "symbol": "BTCUSDT",
                "orderId": 12569138901,
                "clientOrderId": "BqtFCj5odMoWtSqGk2X9tU"
            },
            {
                "symbol": "BTCUSDT",
                "orderId": 12569138902,
                "clientOrderId": "jLnZpj5enfMXTuhKB1d0us"
            }
        ],
        "orderReports": [
            {
                "symbol": "BTCUSDT",
                "orderId": 12569138901,
                "orderListId": 1274512,
                "clientOrderId": "BqtFCj5odMoWtSqGk2X9tU",
                "transactTime": 1660801713793,
                "price": "23410.00000000",
                "origQty": "0.00650000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "NEW",
                "timeInForce": "GTC",
                "type": "STOP_LOSS_LIMIT",
                "side": "SELL",
                "stopPrice": "23405.00000000", // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
                "workingTime": -1,
                "selfTradePreventionMode": "NONE"
            },
            {
                "symbol": "BTCUSDT",
                "orderId": 12569138902,
                "orderListId": 1274512,
                "clientOrderId": "jLnZpj5enfMXTuhKB1d0us",
                "transactTime": 1660801713793,
                "price": "23420.00000000",
                "origQty": "0.00650000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "NEW",
                "timeInForce": "GTC",
                "type": "LIMIT_MAKER",
                "side": "SELL",
                "workingTime": 1660801713793,
                "selfTradePreventionMode": "NONE"
            }
        ]
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 2
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 2
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

<a id="orderList.place.oco"></a>
<a id="orderList-place-oco"></a>
<a id="order-list-place-oco"></a>
<a id="orderlist-place-oco"></a>

<a id="place-new-order-list---oco-trade"></a>

### 发送新 OCO 订单 (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "orderList.place.oco",
    "params": {
        "symbol": "LTCBNB",
        "side": "BUY",
        "quantity": 1,
        "aboveType": "STOP_LOSS_LIMIT",
        "abovePrice": "1.5",
        "aboveStopPrice": "1.50000001",
        "aboveTimeInForce": "GTC",
        "belowType": "LIMIT_MAKER",
        "belowPrice": "1.49999999",
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
发送新 one-cancels-the-other (OCO) 订单，激活其中一个订单会立即取消另一个订单。

* OCO 包含了两个订单，分别被称为 **上方订单** 和 **下方订单**。
* 其中一个订单必须是 `LIMIT_MAKER/TAKE_PROFIT/TAKE_PROFIT_LIMIT` 订单，另一个订单必须是 `STOP_LOSS` 或 `STOP_LOSS_LIMIT` 订单。
* 针对价格限制：
  * 如果 OCO 订单方向是 `SELL`：
    * `LIMIT_MAKER/TAKE_PROFIT_LIMIT` `price` > 最后交易价格 > `STOP_LOSS/STOP_LOSS_LIMIT`
`stopPrice`
    * `TAKE_PROFIT` `stopPrice` > 最后交易价格 > `STOP_LOSS/STOP_LOSS_LIMIT` `stopPrice`
  * 如果 OCO 订单方向是 `BUY`：
    * `LIMIT_MAKER` `price` < 最后交易价格 < `STOP_LOSS/STOP_LOSS_LIMIT` `stopPrice`
    * `TAKE_PROFIT` `stopPrice` > 最后交易价格 > `STOP_LOSS/STOP_LOSS_LIMIT` `stopPrice`
* OCO 将**2 个订单**添加到 `EXCHANGE_MAX_ORDERS`过滤器和 `MAX_NUM_ORDERS` 过滤器中。

**权重:**
1

**未成交的订单计数:**
2

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`listClientOrderId` | STRING | NO | 整个 OCO order list 的唯一ID。 如果未发送则自动生成。 仅当前一个订单已填满或完全过期时，才会接受具有相同的`listClientOrderId`。 `listClientOrderId` 与 `aboveClientOrderId` 和 `belowClientOrderId` 不同。
`side` | ENUM | YES | 订单方向：`BUY` or `SELL`
`quantity` | DECIMAL | YES | 两个订单的数量。
`aboveType` | ENUM | YES |
`aboveClientOrderId` | STRING | NO | 上方订单的唯一ID。 如果未发送则自动生成。
`aboveIcebergQty` | LONG | NO | 请注意，只有当 `aboveTimeInForce` 为 `GTC` 时才能使用。
`abovePrice` | DECIMAL | NO | 当 `aboveType` 是 `STOP_LOSS_LIMIT`, `LIMIT_MAKER` 或 `TAKE_PROFIT_LIMIT` 时，可用以指定限价。
`aboveStopPrice` | DECIMAL | NO | 如果 `aboveType` 是 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT` 或 `TAKE_PROFIT_LIMIT` 才能使用。 必须指定 `aboveStopPrice` 或 `aboveTrailingDelta` 或两者。
`aboveTrailingDelta` | LONG | NO | 请看 [追踪止盈止损(Trailing Stop)订单常见问题](faqs/trailing-stop-faq_CN.md)
`aboveTimeInForce` | ENUM | NO | 如果 `aboveType` 是 `STOP_LOSS_LIMIT` 或 `TAKE_PROFIT_LIMIT`，则为必填项。
`aboveStrategyId` | LONG | NO | 订单策略中上方订单的 ID。
`aboveStrategyType` | INT | NO | 上方订单策略的任意数值。小于 `1000000` 的值被保留，无法使用。
`abovePegPriceType` | ENUM | NO | `PRIMARY_PEG` 或 `MARKET_PEG`。 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`abovePegOffsetType` | ENUM | NO |
`abovePegOffsetValue` | INT | NO | 最大值： 100。
`belowType` | ENUM | YES | 支持值：`STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`。
`belowClientOrderId` | STRING | NO | 下方订单的唯一ID。 如果未发送则自动生成。
`belowIcebergQty` | LONG | NO | 请注意，只有当 `belowTimeInForce` 为 `GTC` 时才能使用。
`belowPrice` | DECIMAL | NO | 当 `belowType` 是 `STOP_LOSS_LIMIT`, `LIMIT_MAKER` 或 `TAKE_PROFIT_LIMIT` 时，可用以指定限价。
`belowStopPrice` | DECIMAL | NO | 如果 `belowType` 是 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT` 或 `TAKE_PROFIT_LIMIT` 才能使用。 必须指定 `belowStopPrice` 或 `belowTrailingDelta` 或两者。
`belowTrailingDelta` | LONG | NO | 请看 [追踪止盈止损(Trailing Stop)订单常见问题](faqs/trailing-stop-faq_CN.md)
`belowTimeInForce` | ENUM | NO | 如果 `belowType` 是 `STOP_LOSS_LIMIT` 或 `TAKE_PROFIT_LIMIT`，则为必须配合提交的值。
`belowStrategyId` | LONG | NO | 订单策略中下方订单的 ID。
`belowStrategyType` | INT | NO | 下方订单策略的任意数值。小于 `1000000` 的值被保留，无法使用。
`belowPegPriceType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`belowPegOffsetType` | ENUM | NO |
`belowPegOffsetValue` | INT | NO | 最大值： 100。
`newOrderRespType` | ENUM | NO | 响应格式可选值: `ACK`, `RESULT`, `FULL`。
`selfTradePreventionMode` | ENUM | NO | 允许的 ENUM 取决于交易对上的配置。 支持值：[STP 模式](enums_CN.md#stpmodes)
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

**响应:**
```javascript
{
    "id": "56374a46-3261-486b-a211-99ed972eb648",
    "status": 200,
    "result": {
        "orderListId": 2,
        "contingencyType": "OCO",
        "listStatusType": "EXEC_STARTED",
        "listOrderStatus": "EXECUTING",
        "listClientOrderId": "cKPMnDCbcLQILtDYM4f4fX",
        "transactionTime": 1711062760648,
        "symbol": "LTCBNB",
        "orders": [
            {
                "symbol": "LTCBNB",
                "orderId": 2,
                "clientOrderId": "0m6I4wfxvTUrOBSMUl0OPU"
            },
            {
                "symbol": "LTCBNB",
                "orderId": 3,
                "clientOrderId": "Z2IMlR79XNY5LU0tOxrWyW"
            }
        ],
        "orderReports": [
            {
                "symbol": "LTCBNB",
                "orderId": 2,
                "orderListId": 2,
                "clientOrderId": "0m6I4wfxvTUrOBSMUl0OPU",
                "transactTime": 1711062760648,
                "price": "1.50000000",
                "origQty": "1.000000",
                "executedQty": "0.000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "NEW",
                "timeInForce": "GTC",
                "type": "STOP_LOSS_LIMIT",
                "side": "BUY",
                "stopPrice": "1.50000001", // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
                "workingTime": -1,
                "selfTradePreventionMode": "NONE"
            },
            {
                "symbol": "LTCBNB",
                "orderId": 3,
                "orderListId": 2,
                "clientOrderId": "Z2IMlR79XNY5LU0tOxrWyW",
                "transactTime": 1711062760648,
                "price": "1.49999999",
                "origQty": "1.000000",
                "executedQty": "0.000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "NEW",
                "timeInForce": "GTC",
                "type": "LIMIT_MAKER",
                "side": "BUY",
                "workingTime": 1711062760648,
                "selfTradePreventionMode": "NONE"
            }
        ]
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 2
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 2
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

<a id="orderList.place.oto"></a>
<a id="orderList-place-oto"></a>
<a id="order-list-place-oto"></a>
<a id="orderlist-place-oto"></a>

<a id="place-new-order-list---oto-trade"></a>

### 发送新订单列表 - OTO (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "orderList.place.oto",
    "params": {
        "symbol": "LTCBNB",
        "workingType": "LIMIT",
        "workingSide": "SELL",
        "workingPrice": 1,
        "workingQuantity": 1,
        "workingTimeInForce": "GTC",
        "pendingType": "MARKET",
        "pendingSide": "BUY",
        "pendingQuantity": 1,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
发送一个新的 OTO 订单。


* 一个 OTO 订单（One-Triggers-the-Other）是一个包含了两个订单的订单列表.

* 第一个订单被称为**生效订单**，必须为 `LIMIT` 或 `LIMIT_MAKER` 类型的订单。最初，订单簿上只有生效订单。

* 第二个订单被称为**待处理订单**。它可以是任何订单类型，但不包括使用参数 `quoteOrderQty` 的 `MARKET` 订单。只有当生效订单**完全成交**时，待处理订单才会被自动下单。

* 如果生效订单或者待处理订单中的任意一个被单独取消，订单列表中剩余的那个订单也会被随之取消或过期。

* 如果生效订单在下订单列表后**立即完全成交**，则可能会得到订单响应。其中，生效订单的状态为 `FILLED` ，但待处理订单的状态为
`PENDING_NEW`。针对这类情况，如果需要检查当前状态，您可以查询相关的待处理订单。

* `OTO` 订单将**2 个订单**添加到 `EXCHANGE_MAX_NUM_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器中。

**权重:**
1

**未成交的订单计数:**
2

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`listClientOrderId` | STRING | NO | 整个订单列表的唯一ID。 如果未发送则自动生成。 仅当前一个订单列表已填满或完全过期时，才会接受含有相同 `listClientOrderId` 值的新订单列表。 `listClientOrderId` 与 `workingClientOrderId` 和 `pendingClientOrderId` 不同。
`newOrderRespType` | ENUM | NO | 用于设置JSON响应的格式。 支持的数值： [订单返回类型](enums_CN.md#orderresponsetype)
`selfTradePreventionMode` | ENUM | NO | 允许的数值取决于交易对上的配置。参考 [STP 模式](enums_CN.md#stpmodes)
`workingType` | ENUM | YES | 支持的数值： `LIMIT`， `LIMIT_MAKER`
`workingSide` | ENUM | YES | 支持的数值： [订单方向](enums_CN.md#side)
`workingClientOrderId` | STRING | NO | 用于标识生效订单的唯一ID。 如果未发送则自动生成。
`workingPrice` | DECIMAL | YES |
`workingQuantity` | DECIMAL | YES | 用于设置生效订单的数量。
`workingIcebergQty` | DECIMAL | NO | 仅当 `workingTimeInForce` 为 `GTC` 或 `workingType` 为 `LIMIT_MAKER` 时，才能使用此功能。
`workingTimeInForce` | ENUM | NO | 支持的数值： [生效时间](enums_CN.md#timeinforce)
`workingStrategyId` | LONG | NO | 订单策略中用于标识生效订单的 ID。
`workingStrategyType` | INT | NO | 用于标识生效订单策略的任意数值。 小于 `1000000` 的值被保留，无法使用。
`workingPegPriceType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`workingPegOffsetType` | ENUM | NO |
`workingPegOffsetValue` | INT | NO | 最大值： 100。
`pendingType` | ENUM | YES | 支持的数值： [订单类型](enums_CN.md#ordertypes) 请注意，系统不支持使用 `quoteOrderQty` 的 `MARKET` 订单。
`pendingSide` | ENUM | YES | 支持的数值： [订单方向](enums_CN.md#side)
`pendingClientOrderId` | STRING | NO | 用于标识待处理订单的唯一ID。 如果未发送则自动生成。
`pendingPrice` | DECIMAL | NO |
`pendingStopPrice` | DECIMAL | NO |
`pendingTrailingDelta` | DECIMAL | NO |
`pendingQuantity` | DECIMAL | YES | 用于设置待处理订单的数量。
`pendingIcebergQty` | DECIMAL | NO | 只有当 `pendingTimeInForce` 为 `GTC` 或者当 `pendingType` 为 `LIMIT_MAKER` 时才能使用。
`pendingTimeInForce` | ENUM | NO | 支持的数值： [生效时间](enums_CN.md#timeinforce)
`pendingStrategyId` | LONG | NO | 订单策略中用于标识待处理订单的 ID。
`pendingStrategyType` | INT | NO | 用于标识待处理订单策略的任意数值。 小于 `1000000` 的值被保留，无法使用。
`pendingPegOffsetType` | ENUM | NO |
`pendingPegPriceType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`pendingPegOffsetValue` | INT | NO | 最大值： 100。
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

<a id="mandatory-parameters-based-on-pendingtype-or-workingtype"></a>

**根据 `pendingType` 或者 `workingType` 的不同值，对于某些参数的强制要求**

根据 `pendingType` 或者`workingType`的不同值，对于某些可选参数有强制要求，具体如下：

|类型                                                   | 强制要求的参数                  | 其他信息|
|----                                                  |----                           |------
|`workingType` = `LIMIT`                               |`workingTimeInForce`           |
|`pendingType` = `LIMIT`                                |`pendingPrice`， `pendingTimeInForce`          |
|`pendingType` = `STOP_LOSS` 或 `TAKE_PROFIT`           |`pendingStopPrice` 与/或 `pendingTrailingDelta`|
|`pendingType` = `STOP_LOSS_LIMIT` 或 `TAKE_PROFIT_LIMIT`|`pendingPrice`， `pendingStopPrice` 与/或 `pendingTrailingDelta`， `pendingTimeInForce`|

**响应:**
```javascript
{
    "id": "1712544395950",
    "status": 200,
    "result": {
        "orderListId": 626,
        "contingencyType": "OTO",
        "listStatusType": "EXEC_STARTED",
        "listOrderStatus": "EXECUTING",
        "listClientOrderId": "KA4EBjGnzvSwSCQsDdTrlf",
        "transactionTime": 1712544395981,
        "symbol": "LTCBNB",
        "orders": [
            {
                "symbol": "LTCBNB",
                "orderId": 13,
                "clientOrderId": "YiAUtM9yJjl1a2jXHSp9Ny"
            },
            {
                "symbol": "LTCBNB",
                "orderId": 14,
                "clientOrderId": "9MxJSE1TYkmyx5lbGLve7R"
            }
        ],
        "orderReports": [
            {
                "symbol": "LTCBNB",
                "orderId": 13,
                "orderListId": 626,
                "clientOrderId": "YiAUtM9yJjl1a2jXHSp9Ny",
                "transactTime": 1712544395981,
                "price": "1.000000",
                "origQty": "1.000000",
                "executedQty": "0.000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.000000",
                "status": "NEW",
                "timeInForce": "GTC",
                "type": "LIMIT",
                "side": "SELL",
                "workingTime": 1712544395981,
                "selfTradePreventionMode": "NONE"
            },
            {
                "symbol": "LTCBNB",
                "orderId": 14,
                "orderListId": 626,
                "clientOrderId": "9MxJSE1TYkmyx5lbGLve7R",
                "transactTime": 1712544395981,
                "price": "0.000000",
                "origQty": "1.000000",
                "executedQty": "0.000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.000000",
                "status": "PENDING_NEW",
                "timeInForce": "GTC",
                "type": "MARKET",
                "side": "BUY",
                "workingTime": -1,
                "selfTradePreventionMode": "NONE"
            }
        ]
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 10000000,
            "count": 10
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 1000,
            "count": 38
        }
    ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](web-socket-api_CN.md#conditional-fields-in-order-responses) 部分。

<a id="orderList.place.otoco"></a>
<a id="orderList-place-otoco"></a>
<a id="order-list-place-otoco"></a>
<a id="orderlist-place-otoco"></a>

<a id="place-new-order-list---otoco-trade"></a>

### 发送新订单列表 - OTOCO (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "orderList.place.otoco",
    "params": {
        "symbol": "LTCBNB",
        "workingType": "LIMIT",
        "workingSide": "BUY",
        "workingPrice": 1.5,
        "workingQuantity": 1,
        "workingTimeInForce": "GTC",
        "pendingSide": "SELL",
        "pendingQuantity": 5,
        "pendingAboveType": "STOP_LOSS",
        "pendingAboveStopPrice": 0.5,
        "pendingBelowType": "LIMIT_MAKER",
        "pendingBelowPrice": 5,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
发送一个新的 OTOCO 订单。

* 一个 OTOCO 订单（One-Triggers-One-Cancels-the-Other）是一个包含了三个订单的订单列表。
* 第一个订单被称为**生效订单**，必须为 `LIMIT` 或 `LIMIT_MAKER` 类型的订单。最初，订单簿上只有生效订单。
  * 生效订单的行为与此一致 [OTO](#orderList-place-oto)
* 一个OTOCO订单有两个待处理订单（pending above 和 pending below），它们构成了一个 OCO 订单列表。只有当生效订单**完全成交**时，待处理订单们才会被自动下单。
  * 待处理上方(pending above)订单和待处理下方(pending below)订单都遵循与 OCO 订单列表相同的规则 [OrderList OCO](web-socket-api_CN.md#place-new-order-list---oco-trade)。
* `OTOCO` 在 `EXCHANGE_MAX_NUM_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器的基础上添加**3个订单**。

**权重:**
1

**未成交的订单计数:**
3

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`listClientOrderId` | STRING | NO | 整个订单列表的唯一ID。 如果未发送则自动生成。 仅当前一个订单列表已填满或完全过期时，才会接受含有相同 `listClientOrderId` 值的新订单列表。 `listClientOrderId` 与 `workingClientOrderId`、`pendingAboveClientOrderId` 和 `pendingBelowClientOrderId` 不同。
`newOrderRespType` | ENUM | NO | 用于设置JSON响应的格式。 支持的数值： [订单返回类型](enums_CN.md#orderresponsetype)
`selfTradePreventionMode` | ENUM | NO | 允许的数值取决于交易对上的配置。参考 [STP 模式](enums_CN.md#stpmodes)
`workingType` | ENUM | YES | 支持的数值： `LIMIT`， `LIMIT_MAKER`
`workingSide` | ENUM | YES | 支持的数值： [订单方向](enums_CN.md#side)
`workingClientOrderId` | STRING | NO | 用于标识生效订单的唯一ID。 如果未发送则自动生成。
`workingPrice` | DECIMAL | YES |
`workingQuantity` | DECIMAL | YES | 用于设置生效订单的数量。
`workingIcebergQty` | DECIMAL | NO | 仅当 `workingTimeInForce` 为 `GTC` 或 `workingType` 为 `LIMIT_MAKER` 时，才能使用此功能。
`workingTimeInForce` | ENUM | NO | 支持的数值： [生效时间](enums_CN.md#timeinforce)
`workingStrategyId` | LONG | NO | 订单策略中用于标识生效订单的 ID。
`workingStrategyType` | INT | NO | 用于标识生效订单策略的任意数值。 小于 `1000000` 的值被保留，无法使用。
`workingPegPriceType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`workingPegOffsetType` | ENUM | NO |
`workingPegOffsetValue` | INT | NO | 最大值： 100。
`pendingSide` | ENUM | YES | 支持的数值： [订单方向](enums_CN.md#side)
`pendingQuantity` | DECIMAL | YES | 用于设置待处理订单的数量。
`pendingAboveType` | ENUM | YES | 支持的数值： `STOP_LOSS_LIMIT`, `STOP_LOSS`, `LIMIT_MAKER`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`
`pendingAboveClientOrderId` | STRING | NO | 用于标识待处理上方订单的唯一ID。 如果未发送则自动生成。
`pendingAbovePrice` | DECIMAL | NO | 当 `pendingAboveType` 是 `STOP_LOSS_LIMIT`, `LIMIT_MAKER` 或 `TAKE_PROFIT_LIMIT` 时，可用以指定限价。
`pendingAboveStopPrice` | DECIMAL | NO | 如果 `pendingAboveType` 是 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT` 才能使用。
`pendingAboveTrailingDelta` | DECIMAL | NO | 参见 [追踪止盈止损(Trailing Stop)订单常见问题](faqs/trailing-stop-faq_CN.md)
`pendingAboveIcebergQty` | DECIMAL | NO | 只有当 `pendingAboveTimeInForce` 为 `GTC` 或 `pendingAboveType` 为 `LIMIT_MAKER` 时才能使用。
`pendingAboveTimeInForce` | ENUM | NO | 如果 `pendingAboveType` 是 `STOP_LOSS_LIMIT` 或 `TAKE_PROFIT_LIMIT`，则为必填项。
`pendingAboveStrategyId` | LONG | NO | 订单策略中用于标识待处理上方订单的 ID。
`pendingAboveStrategyType` | INT | NO | 用于标识待处理上方订单策略的任意数值。 小于 `1000000` 的值被保留，无法使用。
`pendingAbovePegPriceType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`pendingAbovePegOffsetType` | ENUM | NO |
`pendingAbovePegOffsetValue` | INT | NO | 最大值： 100。
`pendingBelowType` | ENUM | NO | 支持的数值： `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`
`pendingBelowClientOrderId` | STRING | NO | 用于标识待处理下方订单的唯一ID。 如果未发送则自动生成。
`pendingBelowPrice` | DECIMAL | NO | 当 `pendingBelowType` 是 `STOP_LOSS_LIMIT` 或 `TAKE_PROFIT_LIMIT` 时，可用以指定限价。
`pendingBelowStopPrice` | DECIMAL | NO | 如果 `pendingBelowType` 是 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT` 才能使用。 必须指定 `pendingBelowStopPrice` 或 `pendingBelowTrailingDelta` 或两者。
`pendingBelowTrailingDelta` | DECIMAL | NO | 参见 [追踪止盈止损(Trailing Stop)订单常见问题](faqs/trailing-stop-faq_CN.md)
`pendingBelowIcebergQty` | DECIMAL | NO | 只有当 `pendingBelowTimeInForce` 为 `GTC` 或 `pendingBelowType` 为 `LIMIT_MAKER` 时才能使用。
`pendingBelowTimeInForce` | ENUM | NO | 如果 `pendingBelowType` 是 `STOP_LOSS_LIMIT` 或 `TAKE_PROFIT_LIMIT`，则为必须配合提交的值。
`pendingBelowStrategyId` | LONG | NO | 订单策略中用于标识待处理下方订单的 ID。
`pendingBelowStrategyType` | INT | NO | 用于标识待处理下方订单策略的任意数值。 小于 `1000000` 的值被保留，无法使用。
`pendingBelowPegPriceType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`pendingBelowPegOffsetType` | ENUM | NO |
`pendingBelowPegOffsetValue` | INT | NO | 最大值： 100。
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

<a id="mandatory-parameters-based-on-pendingabovetype-pendingbelowtype-or-workingtype"></a>

根据 `pendingAboveType`， `pendingBelowType` 或者`workingType`的不同值，对于某些可选参数有强制要求，具体如下：

|类型                                                        | 强制要求的参数                  | 其他信息|
|----                                                       |----                           |------
|`workingType` = `LIMIT`                                    |`workingTimeInForce`           |
|`pendingAboveType` = `STOP_LOSS/TAKE_PROFIT` | `pendingAboveStopPrice` 与/或  `pendingAboveTrailingDelta` |
|`pendingAboveType` = `STOP_LOSS_LIMIT/TAKE_PROFIT_LIMIT` | `pendingAbovePrice`， `pendingAboveStopPrice` 与/或 `pendingAboveTrailingDelta`， `pendingAboveTimeInForce` |
|`pendingAboveType` = `LIMIT_MAKER`                                |`pendingAbovePrice`     |
|`pendingBelowType` = `STOP_LOSS/TAKE_PROFIT` | `pendingBelowStopPrice` 与/或 `pendingBelowTrailingDelta` |
|`pendingBelowType` = `STOP_LOSS_LIMIT/TAKE_PROFIT_LIMIT` | `pendingBelowPrice`， `pendingBelowStopPrice` 与/或 `pendingBelowTrailingDelta`， `pendingBelowTimeInForce` |
|`pendingBelowType` = `LIMIT_MAKER`                                |`pendingBelowPrice`          |

**响应:**
```javascript
{
    "id": "1712544408508",
    "status": 200,
    "result": {
        "orderListId": 629,
        "contingencyType": "OTO",
        "listStatusType": "EXEC_STARTED",
        "listOrderStatus": "EXECUTING",
        "listClientOrderId": "GaeJHjZPasPItFj4x7Mqm6",
        "transactionTime": 1712544408537,
        "symbol": "LTCBNB",
        "orders": [
            {
                "symbol": "LTCBNB",
                "orderId": 23,
                "clientOrderId": "OVQOpKwfmPCfaBTD0n7e7H"
            },
            {
                "symbol": "LTCBNB",
                "orderId": 24,
                "clientOrderId": "YcCPKCDMQIjNvLtNswt82X"
            },
            {
                "symbol": "LTCBNB",
                "orderId": 25,
                "clientOrderId": "ilpIoShcFZ1ZGgSASKxMPt"
            }
        ],
        "orderReports": [
            {
                "symbol": "LTCBNB",
                "orderId": 23,
                "orderListId": 629,
                "clientOrderId": "OVQOpKwfmPCfaBTD0n7e7H",
                "transactTime": 1712544408537,
                "price": "1.500000",
                "origQty": "1.000000",
                "executedQty": "0.000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.000000",
                "status": "NEW",
                "timeInForce": "GTC",
                "type": "LIMIT",
                "side": "BUY",
                "workingTime": 1712544408537,
                "selfTradePreventionMode": "NONE"
            },
            {
                "symbol": "LTCBNB",
                "orderId": 24,
                "orderListId": 629,
                "clientOrderId": "YcCPKCDMQIjNvLtNswt82X",
                "transactTime": 1712544408537,
                "price": "0.000000",
                "origQty": "5.000000",
                "executedQty": "0.000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.000000",
                "status": "PENDING_NEW",
                "timeInForce": "GTC",
                "type": "STOP_LOSS",
                "side": "SELL",
                "stopPrice": "0.500000",
                "workingTime": -1,
                "selfTradePreventionMode": "NONE"
            },
            {
                "symbol": "LTCBNB",
                "orderId": 25,
                "orderListId": 629,
                "clientOrderId": "ilpIoShcFZ1ZGgSASKxMPt",
                "transactTime": 1712544408537,
                "price": "5.000000",
                "origQty": "5.000000",
                "executedQty": "0.000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.000000",
                "status": "PENDING_NEW",
                "timeInForce": "GTC",
                "type": "LIMIT_MAKER",
                "side": "SELL",
                "workingTime": -1,
                "selfTradePreventionMode": "NONE"
            }
        ]
    },
    "rateLimits": [
        {
            "rateLimitType": "ORDERS",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 10000000,
            "count": 18
        },
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 1000,
            "count": 65
        }
    ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](web-socket-api_CN.md#conditional-fields-in-order-responses) 部分。

<a id="sor.order.place"></a>
<a id="sor-order-place"></a>
<a id="sor"></a>

<a id="opo-trade"></a>

### 下 SOR 订单 (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "sor.order.place",
    "params": {
        "symbol": "BTCUSDT",
        "side": "BUY",
        "type": "LIMIT",
        "timeInForce": "GTC",
        "price": 31000,
        "quantity": 0.5,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
发送使用智能订单路由 (SOR) 的新订单。这个请求会把1个订单添加到 `EXCHANGE_MAX_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器中。

参阅 [智能订单路由 (SOR)](faqs/sor_faq_CN.md) 了解更多详情。

**权重:**
1

**未成交的订单计数:**
1

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`side` | ENUM | YES | `BUY` 或 `SELL`
`type` | ENUM | YES | 仅支持 `LIMIT` 和 `MARKET` 订单。
`timeInForce` | ENUM | NO | 仅适用于 `LIMIT` 订单类型。
`price` | DECIMAL | NO |
`quantity` | DECIMAL | YES |
`newClientOrderId` | STRING | NO | 用户自定义的orderid，如空缺系统会自动赋值。如果几个订单具有相同的 `newClientOrderID` 赋值，<br/>那么只有在前一个订单成交后才可以接受下一个订单，否则该订单将被拒绝。
`newOrderRespType` | ENUM | NO | 指定响应类型:
`icebergQty` | DECIMAL | NO | 仅有限价单可以使用该参数，含义为创建冰山订单并指定冰山订单的数量。
`strategyId` | LONG | NO |
`strategyType` | INT | NO | 赋值不能小于 `1000000`.
`selfTradePreventionMode` | ENUM | NO | 允许的 ENUM 取决于交易对的配置。支持的值有：[STP 模式](enums_CN.md#stpmodes)。
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

**注意:** `sor.order.place` 只支持 `限价` 和 `市场` 单， 并不支持 `quoteOrderQty`。

**响应:**
```javascript
{
    "id": "3a4437e2-41a3-4c19-897c-9cadc5dce8b6",
    "status": 200,
    "result": [
        {
            "symbol": "BTCUSDT",
            "orderId": 2,
            "orderListId": -1,
            "clientOrderId": "sBI1KM6nNtOfj5tccZSKly",
            "transactTime": 1689149087774,
            "price": "31000.00000000",
            "origQty": "0.50000000",
            "executedQty": "0.50000000",
            "origQuoteOrderQty": "0.000000",
            "cummulativeQuoteQty": "14000.00000000",
            "status": "FILLED",
            "timeInForce": "GTC",
            "type": "LIMIT",
            "side": "BUY",
            "workingTime": 1689149087774,
            "fills": [
                {
                    "matchType": "ONE_PARTY_TRADE_REPORT",
                    "price": "28000.00000000",
                    "qty": "0.50000000",
                    "commission": "0.00000000",
                    "commissionAsset": "BTC",
                    "tradeId": -1,
                    "allocId": 0
                }
            ],
            "workingFloor": "SOR",                   // 用以定义订单是通过 SOR 还是由订单簿成交的。
            "selfTradePreventionMode": "NONE",
            "usedSor": true,                         // 用于确定订单是否使用SOR的字段。
            "stopPrice": "0.00000000",               // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
            "trailingDelta": 10,                     // 订单激活前所需的价格变动。
            "icebergQty": "0.00000000",              // 仅在请求中发送了 icebergQty 参数时出现。
            "strategyId": 1,                         // 仅在下单时提供了 strategyId 参数时出现。
            "strategyType": 1000000,                 // 仅在下单时提供了 strategyType 参数时出现。
            "preventedMatchId": 0,                   // 仅在因 STP 导致订单失效时出现。
            "preventedQuantity": "1.200000",         // 因 STP 导致订单失效的数量。
            "trailingTime": -1,                      // 追踪单被激活和跟踪价格变化的时间。
            "pegPriceType": "PRIMARY_PEG",           // 挂钩价格类型。仅用于挂钩订单。
            "pegOffsetType": "PRICE_LEVEL",          // 挂钩价格偏移类型。如若需要，仅用于挂钩订单。
            "pegOffsetValue": 5,                     // 挂钩价格偏移值。如若需要，仅用于挂钩订单。
            "peggedPrice": "87523.83710000",         // 订单对应的当前挂钩价格。一旦确定，仅用于挂钩订单。
            "expiryReason": "INSUFFICIENT_LIQUIDITY" // 订单过期的原因。当订单过期时出现。
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

<a id="sor.order.test"></a>
<a id="sor-order-test"></a>

<a id="opoco-trade"></a>

### 测试 SOR 下单接口 (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "sor.order.test",
    "params": {
        "symbol": "BTCUSDT",
        "side": "BUY",
        "type": "LIMIT",
        "timeInForce": "GTC",
        "price": 0.1,
        "quantity": 0.1,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
用于测试使用智能订单路由 (SOR) 的订单请求，但不会提交到撮合引擎。

**权重:**
| 条件 | 请求权重 |
| --- | --- |
| 没有 `computeCommissionRates` | 1 |
| 有 `computeCommissionRates` | 20 |

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`computeCommissionRates` | BOOLEAN | NO | 默认值: `false`
`symbol` | STRING | YES |
`side` | ENUM | YES | 详见枚举定义：[订单方向](enums_CN.md#side)
`type` | ENUM | YES | 详见枚举定义：[订单类型](enums_CN.md#ordertypes)
`timeInForce` | ENUM | NO | 详见枚举定义：[生效时间](enums_CN.md#timeinforce)
`price` | DECIMAL | NO |
`quantity` | DECIMAL | YES |
`newClientOrderId` | STRING | NO | 用户自定义的orderid，如空缺系统会自动赋值。如果几个订单具有相同的 `newClientOrderID` 赋值，那么只有在前一个订单成交后才可以接受下一个订单，否则该订单将被拒绝。
`newOrderRespType` | ENUM | NO | 指定响应类型: `ACK`, `RESULT` 或 `FULL`; 默认为 `FULL`。
`icebergQty` | DECIMAL | NO | 仅有限价单可以使用该参数，含义为创建冰山订单并指定冰山订单的数量。
`strategyId` | LONG | NO |
`strategyType` | INT | NO | 赋值不能小于 `1000000`。
`selfTradePreventionMode` | ENUM | NO | 允许的 ENUM 取决于交易对的配置。支持的值有：[STP 模式](enums_CN.md#stpmodes)。
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 缓存

**响应:**

没有 `computeCommissionRates`:

```javascript
{
    "id": "3a4437e2-41a3-4c19-897c-9cadc5dce8b6",
    "status": 200,
    "result": {},
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

有 `computeCommissionRates`:

```javascript
{
    "id": "3a4437e2-41a3-4c19-897c-9cadc5dce8b6",
    "status": 200,
    "result": {
        // 订单交易的标准佣金率。
        "standardCommissionForOrder": {
            "maker": "0.00000112",
            "taker": "0.00000114"
        },
        // 订单交易的税率。
        "taxCommissionForOrder": {
            "maker": "0.00000112",
            "taker": "0.00000114"
        },
        // 以BNB支付时的标准佣金折扣。
        "discount": {
            "enabledForAccount": true,
            "enabledForSymbol": true,
            "discountAsset": "BNB",
            "discount": "0.25000000" // 当用BNB支付佣金时，在标准佣金上按此比率打折。
        }
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 20
        }
    ]
}
```

<a id="orderList.cancel"></a>
<a id="orderList-cancel"></a>
<a id="order-list-cancel"></a>
<a id="orderlist-cancel"></a>

<a id="cancel-order-list-trade"></a>

### 撤销订单列表订单 (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "orderList.cancel",
    "params": {
        "symbol": "BTCUSDT",
        "orderListId": 1274512,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
取消整个订单列表。

**权重:**
1

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`orderListId` | INT | NO | `orderListId` 或 `listClientOrderId` 必须被提供
`listClientOrderId` | STRING | NO | `orderListId` 或 `listClientOrderId` 必须被提供
`newClientOrderId` | STRING | NO | 用户自定义的本次撤销操作的ID(注意不是被撤销的订单的自定义ID)。如无指定会自动赋值。
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

备注：

* 如果同时指定了 `orderListId` 和 `listClientOrderId` 参数，首先将会用`orderListId` 进行搜索，然后将检索结果中的 `listClientOrderId` 与订单进行比对。如果两个条件都不满足，则请求将被拒绝。

* 使用 [`order.cancel`](#order-cancel) 撤销订单列表内的某个订单，则整个订单列表将被撤销。

**响应:**
```javascript
{
    "id": "c5899911-d3f4-47ae-8835-97da553d27d0",
    "status": 200,
    "result": {
        "orderListId": 1274512,
        "contingencyType": "OCO",
        "listStatusType": "ALL_DONE",
        "listOrderStatus": "ALL_DONE",
        "listClientOrderId": "6023531d7edaad348f5aff",
        "transactionTime": 1660801720215,
        "symbol": "BTCUSDT",
        "orders": [
            {
                "symbol": "BTCUSDT",
                "orderId": 12569138901,
                "clientOrderId": "BqtFCj5odMoWtSqGk2X9tU"
            },
            {
                "symbol": "BTCUSDT",
                "orderId": 12569138902,
                "clientOrderId": "jLnZpj5enfMXTuhKB1d0us"
            }
        ],
        "orderReports": [
            {
                "symbol": "BTCUSDT",
                "orderId": 12569138901,
                "orderListId": 1274512,
                "clientOrderId": "BqtFCj5odMoWtSqGk2X9tU",
                "transactTime": 1660801720215,
                "price": "23410.00000000",
                "origQty": "0.00650000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "CANCELED",
                "timeInForce": "GTC",
                "type": "STOP_LOSS_LIMIT",
                "side": "SELL",
                "stopPrice": "23405.00000000", // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
                "selfTradePreventionMode": "NONE"
            },
            {
                "symbol": "BTCUSDT",
                "orderId": 12569138902,
                "orderListId": 1274512,
                "clientOrderId": "jLnZpj5enfMXTuhKB1d0us",
                "transactTime": 1660801720215,
                "price": "23420.00000000",
                "origQty": "0.00650000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "CANCELED",
                "timeInForce": "GTC",
                "type": "LIMIT_MAKER",
                "side": "SELL",
                "selfTradePreventionMode": "NONE"
            }
        ]
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 1
        }
    ]
}
```

<a id="orderList.place.opo"></a>
<a id="orderList-place-opo"></a>
<a id="order-list-place-opo"></a>
<a id="orderlist-place-opo"></a>

<a id="place-new-order-using-sor-trade"></a>

### 新订单列表 - OPO (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "orderList.place.opo",
    "params": {
        "symbol": "BTCUSDT",
        "workingType": "LIMIT",
        "workingSide": "BUY",
        "workingPrice": "101496",
        "workingQuantity": "0.0007",
        "workingTimeInForce": "GTC",
        "pendingType": "MARKET",
        "pendingSide": "SELL",
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
发送一个 [OPO](faqs/opo_CN.md) 订单。

* OPO 会向 `EXCHANGE_MAX_NUM_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器中添加 2 个订单。

**权重:**
1

**未成交的订单计数:**
2

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`listClientOrderId` | STRING | NO | 订单列表中的唯一 ID。如果未发送，则自动生成。只有当之前的同一 `listClientOrderId` 的订单列表已成交或完全过期后，才接受新的同一 `listClientOrderId` 的订单列表。`listClientOrderId` 与 `workingClientOrderId` 和 `pendingClientOrderId` 不同。
`newOrderRespType` | ENUM | NO | JSON 响应格式。支持的数值：[订单返回类型](enums_CN.md#orderresponsetype)
`selfTradePreventionMode` | ENUM | NO | 允许的数值取决于交易对的配置。支持的数值：[STP模式](enums_CN.md#stpmodes)
`workingType` | ENUM | YES | 支持的数值：`LIMIT`，`LIMIT_MAKER`
`workingSide` | ENUM | YES | 支持的数值：[订单方向](enums_CN.md#side)
`workingClientOrderId` | STRING | NO | 生效订单中挂单的任意唯一 ID。如果未发送，则自动生成。
`workingPrice` | DECIMAL | YES | 生效订单价格
`workingQuantity` | DECIMAL | YES | 设置生效订单的数量
`workingIcebergQty` | DECIMAL | NO | 仅当 `workingTimeInForce` 为 `GTC` 或 `workingType` 为 `LIMIT_MAKER` 时可用
`workingTimeInForce` | ENUM | NO | 支持的数值：[生效时间](enums_CN.md#timeinforce)
`workingStrategyId` | LONG | NO | 用于标识订单策略中生效订单的任意数字值
`workingStrategyType` | INT | NO | 用于标识生效订单策略的任意数字值。小于 `1000000` 的值为保留值，不能使用。
`workingPegPriceType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`workingPegOffsetType` | ENUM | NO |
`workingPegOffsetValue` | INT | NO | 最大值： 100。
`pendingType` | ENUM | YES | 支持的数值：[订单类型](enums_CN.md#ordertypes)。注意，不支持使用 `quoteOrderQty` 的 `MARKET` 订单。
`pendingSide` | ENUM | YES | 支持的数值：[订单方向](enums_CN.md#side)
`pendingClientOrderId` | STRING | NO | 待处理订单中挂单的任意唯一 ID。如果未发送，则自动生成。
`pendingPrice` | DECIMAL | NO | 待处理订单价格
`pendingStopPrice` | DECIMAL | NO | 待处理订单止损价格
`pendingTrailingDelta` | DECIMAL | NO | 待处理订单跟踪止损差值
`pendingIcebergQty` | DECIMAL | NO | 仅当 `pendingTimeInForce` 为 `GTC` 或 `pendingType` 为 `LIMIT_MAKER` 时可用
`pendingTimeInForce` | ENUM | NO | 支持的数值：[生效时间](enums_CN.md#timeinforce)
`pendingStrategyId` | LONG | NO | 用于标识订单策略中待处理订单的任意数字值
`pendingStrategyType` | INT | NO | 用于标识待处理订单策略的任意数字值。小于 `1000000` 的值为保留值，不能使用。
`pendingPegPriceType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`pendingPegOffsetType` | ENUM | NO |
`pendingPegOffsetValue` | INT | NO | 最大值： 100。
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

**响应:**
```javascript
{
    "id": "eabb220d-1c53-4f8a-8bca-a1cd3594b3ce",
    "status": 200,
    "result": {
        "orderListId": 2,
        "contingencyType": "OTO",
        "listStatusType": "EXEC_STARTED",
        "listOrderStatus": "EXECUTING",
        "listClientOrderId": "OiOgqvRagBefpzdM5gjYX3",
        "transactionTime": 1762941318142,
        "symbol": "BTCUSDT",
        "orders": [
            {
                "symbol": "BTCUSDT",
                "orderId": 2,
                "clientOrderId": "pUzhKBbc0ZVdMScIRAqitH"
            },
            {
                "symbol": "BTCUSDT",
                "orderId": 3,
                "clientOrderId": "x7ISSjywZxFXOdzwsThNnd"
            }
        ],
        "orderReports": [
            {
                "symbol": "BTCUSDT",
                "orderId": 2,
                "orderListId": 2,
                "clientOrderId": "pUzhKBbc0ZVdMScIRAqitH",
                "transactTime": 1762941318142,
                "price": "101496.00000000",
                "origQty": "0.00070000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.00000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "NEW",
                "timeInForce": "GTC",
                "type": "LIMIT",
                "side": "BUY",
                "workingTime": 1762941318142,
                "selfTradePreventionMode": "NONE"
            },
            {
                "symbol": "BTCUSDT",
                "orderId": 3,
                "orderListId": 2,
                "clientOrderId": "x7ISSjywZxFXOdzwsThNnd",
                "transactTime": 1762941318142,
                "price": "0.00000000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.00000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "PENDING_NEW",
                "timeInForce": "GTC",
                "type": "MARKET",
                "side": "SELL",
                "workingTime": -1,
                "selfTradePreventionMode": "NONE"
            }
        ]
    }
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](web-socket-api_CN.md#conditional-fields-in-order-responses) 部分。

<a id="orderList.place.opoco"></a>
<a id="orderList-place-opoco"></a>
<a id="order-list-place-opoco"></a>
<a id="orderlist-place-opoco"></a>

<a id="test-new-order-using-sor-trade"></a>

### 新订单列表 - OPOCO (TRADE)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "orderList.place.opoco",
    "params": {
        "symbol": "BTCUSDT",
        "workingType": "LIMIT",
        "workingSide": "BUY",
        "workingPrice": "102496",
        "workingQuantity": "0.0017",
        "workingTimeInForce": "GTC",
        "pendingSide": "SELL",
        "pendingAboveType": "LIMIT_MAKER",
        "pendingAbovePrice": "104261",
        "pendingBelowType": "STOP_LOSS_LIMIT",
        "pendingBelowPrice": "101613",
        "pendingBelowStopPrice": "10100",
        "pendingBelowTimeInForce": "IOC",
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
发送一个 [OPOCO](faqs/opo_CN.md) 订单。

**权重:**
1

**未成交的订单计数:**
3

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`listClientOrderId` | STRING | NO | 订单列表中的唯一 ID。如果未发送，则自动生成。只有当之前的同一 `listClientOrderId` 的订单列表已成交或完全过期后，才接受新的同一 `listClientOrderId` 的订单列表。`listClientOrderId` 与 `workingClientOrderId`、`pendingAboveClientOrderId` 和 `pendingBelowClientOrderId` 不同。
`newOrderRespType` | ENUM | NO | JSON 响应格式。支持的数值：[订单返回类型](enums_CN.md#orderresponsetype)
`selfTradePreventionMode` | ENUM | NO | 允许的数值取决于交易对的配置。支持的数值：[STP模式](enums_CN.md#stpmodes)
`workingType` | ENUM | YES |
`workingSide` | ENUM | YES | 支持的数值：[订单方向](enums_CN.md#side)
`workingClientOrderId` | STRING | NO | 生效订单中挂单的任意唯一 ID。如果未发送，则自动生成。
`workingPrice` | DECIMAL | YES | 生效订单价格
`workingQuantity` | DECIMAL | YES | 设置生效订单的数量
`workingIcebergQty` | DECIMAL | NO | 仅当 `workingTimeInForce` 为 `GTC` 或 `workingType` 为 `LIMIT_MAKER` 时可用
`workingTimeInForce` | ENUM | NO | 支持的数值：[生效时间](enums_CN.md#timeinforce)
`workingStrategyId` | LONG | NO | 用于标识订单策略中生效订单的任意数字值
`workingStrategyType` | INT | NO | 用于标识生效订单策略的任意数字值。小于 `1000000` 的值为保留值，不能使用。
`workingPegPriceType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`workingPegOffsetType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`workingPegOffsetValue` | INT | NO | 用于挂钩的价格水平（最大值：100）。 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`pendingSide` | ENUM | YES | 支持的数值：[订单方向](enums_CN.md#side)
`pendingAboveType` | ENUM | YES | 支持的数值：`STOP_LOSS_LIMIT`，`STOP_LOSS`，`LIMIT_MAKER`，`TAKE_PROFIT`，`TAKE_PROFIT_LIMIT`
`pendingAboveClientOrderId` | STRING | NO | 待处理"上方"订单中开放订单的任意唯一 ID。如果未发送，则自动生成。
`pendingAbovePrice` | DECIMAL | NO | 当 `pendingAboveType` 为 `STOP_LOSS_LIMIT`、`LIMIT_MAKER` 或 `TAKE_PROFIT_LIMIT` 时，可用于指定限价。
`pendingAboveStopPrice` | DECIMAL | NO | 当 `pendingAboveType` 为 `STOP_LOSS`、`STOP_LOSS_LIMIT`、`TAKE_PROFIT`、`TAKE_PROFIT_LIMIT` 时可用。
`pendingAboveTrailingDelta` | DECIMAL | NO | 详见 [追踪止盈止损常见问题](faqs/trailing-stop-faq_CN.md)
`pendingAboveIcebergQty` | DECIMAL | NO | 仅当 `pendingAboveTimeInForce` 为 `GTC` 或 `pendingAboveType` 为 `LIMIT_MAKER` 时可用。
`pendingAboveTimeInForce` | ENUM | NO | 如果 `pendingAboveType` 是 `STOP_LOSS_LIMIT` 或 `TAKE_PROFIT_LIMIT`，则为必填项。
`pendingAboveStrategyId` | LONG | NO | 用于标识订单策略中待处理上方订单的任意数字值。
`pendingAboveStrategyType` | INT | NO | 用于标识待处理上方订单策略的任意数字值。小于 `1000000` 的值为保留值，不能使用。
`pendingAbovePegPriceType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`pendingAbovePegOffsetType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`pendingAbovePegOffsetValue` | INT | NO | 用于挂钩的价格水平（最大值：100）。 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`pendingBelowType` | ENUM | NO | 支持的数值：`STOP_LOSS`，`STOP_LOSS_LIMIT`，`TAKE_PROFIT`，`TAKE_PROFIT_LIMIT`
`pendingBelowClientOrderId` | STRING | NO | 待处理"下方"订单中开放订单的任意唯一 ID。如果未发送，则自动生成。
`pendingBelowPrice` | DECIMAL | NO | 当 `pendingBelowType` 为 `STOP_LOSS_LIMIT` 或 `TAKE_PROFIT_LIMIT` 时，可用于指定限价。
`pendingBelowStopPrice` | DECIMAL | NO | 当 `pendingBelowType` 为 `STOP_LOSS`、`STOP_LOSS_LIMIT`、`TAKE_PROFIT` 或 `TAKE_PROFIT_LIMIT` 时可用。`pendingBelowStopPrice`、`pendingBelowTrailingDelta` 或两者之一必须被指定。
`pendingBelowTrailingDelta` | DECIMAL | NO | 详见 [追踪止盈止损常见问题](faqs/trailing-stop-faq_CN.md)
`pendingBelowIcebergQty` | DECIMAL | NO | 仅当 `pendingBelowTimeInForce` 为 `GTC` 或 `pendingBelowType` 为 `LIMIT_MAKER` 时可用。
`pendingBelowTimeInForce` | ENUM | NO | 支持的数值：[生效时间](enums_CN.md#timeinforce)
`pendingBelowStrategyId` | LONG | NO | 用于标识订单策略中待处理下方订单的任意数字值。
`pendingBelowStrategyType` | INT | NO | 用于标识待处理下方订单策略的任意数字值。小于 `1000000` 为保留值，不能使用。
`pendingBelowPegPriceType` | ENUM | NO | 参阅 [关于挂钩订单参数的注意事项](faqs/pegged_orders_CN.md)
`pendingBelowPegOffsetType` | ENUM | NO |
`pendingBelowPegOffsetValue` | INT | NO | 最大值： 100。
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 撮合引擎

**响应:**
```javascript
{
    "id": "606063bf-43ea-40a6-a463-5c66fbd53b56",
    "status": 200,
    "result": {
        "orderListId": 1,
        "contingencyType": "OTO",
        "listStatusType": "EXEC_STARTED",
        "listOrderStatus": "EXECUTING",
        "listClientOrderId": "TVbG6ymkYMXTj7tczbOsBf",
        "transactionTime": 1763000139104,
        "symbol": "BTCUSDT",
        "orders": [
            {
                "symbol": "BTCUSDT",
                "orderId": 6,
                "clientOrderId": "3czuJSeyjPwV9Xo28j1Dv3"
            },
            {
                "symbol": "BTCUSDT",
                "orderId": 7,
                "clientOrderId": "kyIKnMLKQclE5FmyYgaMSo"
            },
            {
                "symbol": "BTCUSDT",
                "orderId": 8,
                "clientOrderId": "i76cGJWN9J1FpADS56TtQZ"
            }
        ],
        "orderReports": [
            {
                "symbol": "BTCUSDT",
                "orderId": 6,
                "orderListId": 1,
                "clientOrderId": "3czuJSeyjPwV9Xo28j1Dv3",
                "transactTime": 1763000139104,
                "price": "102496.00000000",
                "origQty": "0.00170000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.00000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "NEW",
                "timeInForce": "GTC",
                "type": "LIMIT",
                "side": "BUY",
                "workingTime": 1763000139104,
                "selfTradePreventionMode": "NONE"
            },
            {
                "symbol": "BTCUSDT",
                "orderId": 7,
                "orderListId": 1,
                "clientOrderId": "kyIKnMLKQclE5FmyYgaMSo",
                "transactTime": 1763000139104,
                "price": "101613.00000000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.00000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "PENDING_NEW",
                "timeInForce": "IOC",
                "type": "STOP_LOSS_LIMIT",
                "side": "SELL",
                "stopPrice": "10100.00000000",
                "workingTime": -1,
                "selfTradePreventionMode": "NONE"
            },
            {
                "symbol": "BTCUSDT",
                "orderId": 8,
                "orderListId": 1,
                "clientOrderId": "i76cGJWN9J1FpADS56TtQZ",
                "transactTime": 1763000139104,
                "price": "104261.00000000",
                "executedQty": "0.00000000",
                "origQuoteOrderQty": "0.00000000",
                "cummulativeQuoteQty": "0.00000000",
                "status": "PENDING_NEW",
                "timeInForce": "GTC",
                "type": "LIMIT_MAKER",
                "side": "SELL",
                "workingTime": -1,
                "selfTradePreventionMode": "NONE"
            }
        ]
    }
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](web-socket-api_CN.md#conditional-fields-in-order-responses) 部分。

<a id="account-requests"></a>

## 账户请求

<a id="account.status"></a>
<a id="account-status"></a>

<a id="account-information-user_data"></a>

### 账户信息 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "account.status",
    "params": {
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```
获取当前账户信息。

**权重:**
20

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`omitZeroBalances` | BOOLEAN | NO | 如果`true`，将隐藏所有零余额。默认值：`false`。
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 缓存 => 数据库

**响应:**
```javascript
{
    "id": "605a6d20-6588-4cb9-afa0-b0ab087507ba",
    "status": 200,
    "result": {
        "makerCommission": 15,
        "takerCommission": 15,
        "buyerCommission": 0,
        "sellerCommission": 0,
        "canTrade": true,
        "canWithdraw": true,
        "canDeposit": true,
        "commissionRates": {
            "maker": "0.00150000",
            "taker": "0.00150000",
            "buyer": "0.00000000",
            "seller": "0.00000000"
        },
        "brokered": false,
        "requireSelfTradePrevention": false,
        "preventSor": false,
        "updateTime": 1660801833000,
        "accountType": "SPOT",
        "balances": [
            {
                "asset": "BNB",
                "free": "0.00000000",
                "locked": "0.00000000"
            },
            {
                "asset": "BTC",
                "free": "1.3447112",
                "locked": "0.08600000"
            },
            {
                "asset": "USDT",
                "free": "1021.21000000",
                "locked": "0.00000000"
            }
        ],
        "permissions": [
            "SPOT"
        ],
        "uid": 354937868
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 20
        }
    ]
}
```

<a id="order.status"></a>
<a id="order-status"></a>

<a id="query-order-user_data"></a>

### 查询订单 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "order.status",
    "params": {
        "symbol": "BTCUSDT",
        "orderId": 12569099453,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
查询订单状态。

**权重:**
4

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`orderId` | LONG | NO | 按 `orderId` 查找订单
`origClientOrderId` | STRING | NO | 按 `clientOrderId` 查找订单
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 缓存 => 数据库

备注：

* 当同时提供 `orderId` 和 `origClientOrderId` 两个参数时，系统首先将会使用 `orderId` 来搜索订单。然后， 查找结果中的 `origClientOrderId` 的值将会被用来验证订单。如果两个条件都不满足，则请求将被拒绝。

* 对于某些历史订单，`cummulativeQuoteQty` 响应字段可能为负数，意味着此时数据不可用。

**响应:**
```javascript
{
    "id": "aa62318a-5a97-4f3b-bdc7-640bbe33b291",
    "status": 200,
    "result": {
        "symbol": "BTCUSDT",
        "orderId": 12569099453,
        "orderListId": -1,                       // 仅当订单属于订单列表时返回。
        "clientOrderId": "4d96324ff9d44481926157",
        "price": "23416.10000000",
        "origQty": "0.00847000",
        "executedQty": "0.00847000",
        "origQuoteOrderQty": "0.000000",         // 始终返回；若订单类型不使用 `quoteOrderQty` 则为 0。
        "cummulativeQuoteQty": "198.33521500",
        "status": "FILLED",
        "timeInForce": "GTC",
        "type": "LIMIT",
        "side": "SELL",
        "stopPrice": "0.00000000",               // 始终返回；若订单类型不使用 `stopPrice` 则为 0。
        "trailingDelta": 10,                     // 仅当订单设置了 `trailingDelta` 时返回。
        "trailingTime": -1,                      // 仅当订单设置了 `trailingDelta` 时返回。
        "icebergQty": "0.00000000",              // 始终返回；非冰山订单为 0。
        "time": 1660801715639,                   // 下单时间。
        "updateTime": 1660801717945,             // 订单最后更新时间。
        "isWorking": true,
        "workingTime": 1660801715639,
        "strategyId": 37463720,                  // 仅当订单设置了 `strategyId` 时返回。
        "strategyType": 1000000,                 // 仅当订单设置了 `strategyType` 时返回。
        "selfTradePreventionMode": "NONE",
        "preventedMatchId": 0,                   // 仅当订单因 STP 过期时返回。
        "preventedQuantity": "1.200000",         // 仅当订单因 STP 过期时返回。
        "usedSor": true,                         // 用于确定订单是否使用SOR的字段。
        "workingFloor": "SOR",                   // 用以定义订单是通过 SOR 还是由订单簿成交的。
        "pegPriceType": "PRIMARY_PEG",           // 挂钩价格类型。仅用于挂钩订单。
        "pegOffsetType": "PRICE_LEVEL",          // 挂钩价格偏移类型。如若需要，仅用于挂钩订单。
        "pegOffsetValue": 5,                     // 挂钩价格偏移值。如若需要，仅用于挂钩订单。
        "peggedPrice": "87523.83710000",         // 订单对应的当前挂钩价格。一旦确定，仅用于挂钩订单。
        "expiryReason": "INSUFFICIENT_LIQUIDITY" // 订单过期的原因。当订单过期时出现。
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 4
        }
    ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](web-socket-api_CN.md#conditional-fields-in-order-responses) 部分。

<a id="openOrders.status"></a>
<a id="openOrders-status"></a>
<a id="open-orders-status"></a>
<a id="openorders-status"></a>

<a id="current-open-orders-user_data"></a>

### 当前挂单 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "openOrders.status",
    "params": {
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```
查询所有挂订单的执行状态。

如果您需要持续监控订单状态更新，请考虑使用 WebSocket Streams：

* 如果会话已通过验证，请使用 [`userDataStream.subscribe`](web-socket-api_CN.md#subscribe-to-user-data-stream-user_stream)
* 如果通过签名订阅的方式订阅，请使用 [`userDataStream.subscribe.signature`](web-socket-api_CN.md#subscribe-to-user-data-stream-through-signature-subscription-user_stream)

**权重:**
| 参数 | 权重 |
| --------- | ------ |
| `symbol`  |      6 |
| 不提供    |     80 |

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | NO | 如果省略，则返回所有交易对的挂单
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 缓存 => 数据库

**响应:**
```javascript
{
    "id": "55f07876-4f6f-4c47-87dc-43e5fff3f2e7",
    "status": 200,
    "result": [
        {
            "symbol": "BTCUSDT",
            "orderId": 12569099453,
            "orderListId": -1,                       // 仅当订单属于订单列表时返回。
            "clientOrderId": "4d96324ff9d44481926157",
            "price": "23416.10000000",
            "origQty": "0.00847000",
            "executedQty": "0.00720000",
            "origQuoteOrderQty": "0.000000",         // 始终返回；若订单类型不使用 `quoteOrderQty` 则为 0。
            "cummulativeQuoteQty": "172.43931000",
            "status": "PARTIALLY_FILLED",
            "timeInForce": "GTC",
            "type": "LIMIT",
            "side": "SELL",
            "time": 1660801715639,                   // 下单时间。
            "updateTime": 1660801717945,             // 订单最后更新时间。
            "isWorking": true,
            "workingTime": 1660801715639,
            "selfTradePreventionMode": "NONE",
            "icebergQty": "0.00000000",              // 仅在请求中发送了 icebergQty 参数时出现。
            "preventedMatchId": 0,                   // 仅在因 STP 导致订单失效时出现。
            "preventedQuantity": "1.200000",         // 因 STP 导致订单失效的数量。
            "stopPrice": "0.00000000",               // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
            "strategyId": 1,                         // 仅在下单时提供了 strategyId 参数时出现。
            "strategyType": 1000000,                 // 仅在下单时提供了 strategyType 参数时出现。
            "trailingDelta": 10,                     // 订单激活前所需的价格变动。
            "trailingTime": -1,                      // 追踪单被激活和跟踪价格变化的时间。
            "usedSor": true,                         // 用于确定订单是否使用SOR的字段。
            "workingFloor": "SOR",                   // 用以定义订单是通过 SOR 还是由订单簿成交的。
            "pegPriceType": "PRIMARY_PEG",           // 挂钩价格类型。仅用于挂钩订单。
            "pegOffsetType": "PRICE_LEVEL",          // 挂钩价格偏移类型。如若需要，仅用于挂钩订单。
            "pegOffsetValue": 5,                     // 挂钩价格偏移值。如若需要，仅用于挂钩订单。
            "peggedPrice": "87523.83710000",         // 订单对应的当前挂钩价格。一旦确定，仅用于挂钩订单。
            "expiryReason": "INSUFFICIENT_LIQUIDITY" // 订单过期的原因。当订单过期时出现。
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 6
        }
    ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](web-socket-api_CN.md#conditional-fields-in-order-responses) 部分。

<a id="allOrders"></a>
<a id="all-orders"></a>
<a id="allorders"></a>

<a id="account-order-history-user_data"></a>

### 账户订单历史 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "allOrders",
    "params": {
        "symbol": "BNBUSDT",
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```
查询所有订单（活动、已撤销、已成交）信息，并按时间范围过滤。

**权重:**
20

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`orderId` | LONG | NO | 起始订单ID
`startTime` | LONG | NO | 毫秒时间戳
`endTime` | LONG | NO | 毫秒时间戳
`limit` | INT | NO | 默认值: 500; 最大值: 1000
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 数据库

备注：

* 如果指定了 `startTime` 和/或 `endTime`，则忽略 `orderId`。

  订单是按照最后一次更新的执行状态的`time`过滤的。

* 如果指定了 `orderId`，返回的订单将是订单ID >= `orderId`

* 如果不指定条件，则返回最近的订单。

* 对于某些历史订单，`cummulativeQuoteQty` 响应字段可能为负数，代表着此时数据还不可用。

* `startTime`和`endTime`之间的时间不能超过 24 小时。

**响应:**
```javascript
{
    "id": "734235c2-13d2-4574-be68-723e818c08f3",
    "status": 200,
    "result": [
        {
            "symbol": "BTCUSDT",
            "orderId": 12569099453,
            "orderListId": -1,                       // 仅当订单属于订单列表时返回。
            "clientOrderId": "4d96324ff9d44481926157",
            "price": "23416.10000000",
            "origQty": "0.00847000",
            "executedQty": "0.00847000",
            "cummulativeQuoteQty": "198.33521500",
            "status": "FILLED",
            "timeInForce": "GTC",
            "type": "LIMIT",
            "side": "SELL",
            "time": 1660801715639,                   // 下单时间。
            "updateTime": 1660801717945,             // 订单最后更新时间。
            "isWorking": true,
            "workingTime": 1660801715639,
            "origQuoteOrderQty": "0.000000",         // 始终返回；若订单类型不使用 `quoteOrderQty` 则为 0。
            "selfTradePreventionMode": "NONE",
            "preventedMatchId": 0,                   // 仅当订单因 STP 过期时返回。
            "preventedQuantity": "1.200000",         // 仅当订单因 STP 过期时返回。
            "icebergQty": "0.00000000",              // 仅在请求中发送了 icebergQty 参数时出现。
            "stopPrice": "0.00000000",               // 出现在 STOP_LOSS, TAKE_PROFIT, STOP_LOSS_LIMIT 和 TAKE_PROFIT_LIMIT 订单中。
            "strategyId": 1,                         // 仅在下单时提供了 strategyId 参数时出现。
            "strategyType": 1000000,                 // 仅在下单时提供了 strategyType 参数时出现。
            "trailingDelta": 10,                     // 订单激活前所需的价格变动。
            "trailingTime": -1,                      // 追踪单被激活和跟踪价格变化的时间。
            "usedSor": true,                         // 用于确定订单是否使用SOR的字段。
            "workingFloor": "SOR",                   // 用以定义订单是通过 SOR 还是由订单簿成交的。
            "pegPriceType": "PRIMARY_PEG",           // 挂钩价格类型。仅用于挂钩订单。
            "pegOffsetType": "PRICE_LEVEL",          // 挂钩价格偏移类型。如若需要，仅用于挂钩订单。
            "pegOffsetValue": 5,                     // 挂钩价格偏移值。如若需要，仅用于挂钩订单。
            "peggedPrice": "87523.83710000",         // 订单对应的当前挂钩价格。一旦确定，仅用于挂钩订单。
            "expiryReason": "INSUFFICIENT_LIQUIDITY" // 订单过期的原因。当订单过期时出现。
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 20
        }
    ]
}
```

<a id="orderList.status"></a>
<a id="orderList-status"></a>
<a id="order-list-status"></a>
<a id="orderlist-status"></a>

<a id="query-order-list-user_data"></a>

### 查询订单列表 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "orderList.status",
    "params": {
        "origClientOrderId": "08985fedd9ea2cf6b28996",
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
检查订单列表的执行状态。

对于单个订单的执行状态，使用 `order.status`。

**权重:**
4

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`origClientOrderId` | STRING | NO | 通过 `listClientOrderId` 获取订单列表。必须提供 `orderListId` 或 `origClientOrderId`。
`orderListId` | INT | NO | 通过 `orderListId` 获取订单列表。必须提供 `orderListId` 或 `origClientOrderId`。
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 数据库

备注：

* `origClientOrderId` 指的是订单列表本身的 `listClientOrderId`。

* 如果同时指定了 `origClientOrderId` 和 `orderListId` 参数，仅使用 `origClientOrderId` 并忽略 `orderListId`。

**响应:**
```javascript
{
    "id": "b53fd5ff-82c7-4a04-bd64-5f9dc42c2100",
    "status": 200,
    "result": {
        "orderListId": 1274512,
        "contingencyType": "OCO",
        "listStatusType": "EXEC_STARTED",
        "listOrderStatus": "EXECUTING",
        "listClientOrderId": "08985fedd9ea2cf6b28996",
        "transactionTime": 1660801713793,
        "symbol": "BTCUSDT",
        "orders": [
            {
                "symbol": "BTCUSDT",
                "orderId": 12569138901,
                "clientOrderId": "BqtFCj5odMoWtSqGk2X9tU"
            },
            {
                "symbol": "BTCUSDT",
                "orderId": 12569138902,
                "clientOrderId": "jLnZpj5enfMXTuhKB1d0us"
            }
        ]
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 4
        }
    ]
}
```

<a id="openOrderLists.status"></a>
<a id="openOrderLists-status"></a>
<a id="open-order-lists-status"></a>
<a id="openorderlists-status"></a>

<a id="current-open-order-lists-user_data"></a>

### 查询订单列表挂单 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "openOrderLists.status",
    "params": {
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```
查询所有订单列表挂单的执行状态。

如果您需要持续监控订单状态更新，请考虑使用 WebSocket Streams：

* 如果会话已通过验证，请使用 [`userDataStream.subscribe`](web-socket-api_CN.md#subscribe-to-user-data-stream-user_stream)
* 如果通过签名订阅的方式订阅，请使用 [`userDataStream.subscribe.signature`](web-socket-api_CN.md#subscribe-to-user-data-stream-through-signature-subscription-user_stream)

**权重:**
6

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 缓存 => 数据库

**响应:**
```javascript
{
    "id": "3a4437e2-41a3-4c19-897c-9cadc5dce8b6",
    "status": 200,
    "result": [
        {
            "orderListId": 0,
            "contingencyType": "OCO",
            "listStatusType": "EXEC_STARTED",
            "listOrderStatus": "EXECUTING",
            "listClientOrderId": "08985fedd9ea2cf6b28996",
            "transactionTime": 1660801713793,
            "symbol": "BTCUSDT",
            "orders": [
                {
                    "symbol": "BTCUSDT",
                    "orderId": 4,
                    "clientOrderId": "CUhLgTXnX5n2c0gWiLpV4d"
                },
                {
                    "symbol": "BTCUSDT",
                    "orderId": 5,
                    "clientOrderId": "1ZqG7bBuYwaF4SU8CwnwHm"
                }
            ]
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 6
        }
    ]
}
```

<a id="allOrderLists"></a>
<a id="all-order-lists"></a>
<a id="allorderlists"></a>

<a id="account-order-list-history-user_data"></a>

### 账户订单列表历史 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "allOrderLists",
    "params": {
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```
查询所有订单列表的信息，按时间范围过滤。

**权重:**
20

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`fromId` | INT | NO | 起始的 Order list ID
`startTime` | LONG | NO | 毫秒时间戳
`endTime` | LONG | NO | 毫秒时间戳
`limit` | INT | NO | 默认值: 500; 最大值: 1000
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 数据库

备注：

* 如果指定了 `startTime` 和/或 `endTime`，则忽略 `fromId`。

  订单列表是按照最后一次更新的订单列表执行状态的 `transactionTime` 过滤的。

* 如果指定了 `fromId`，返回的订单列表将是 order list ID >= `fromId`。

* 如果不指定条件，则返回最近的订单列表。

* `startTime`和`endTime`之间的时间不能超过 24 小时

**响应:**
```javascript
{
    "id": "8617b7b3-1b3d-4dec-94cd-eefd929b8ceb",
    "status": 200,
    "result": [
        {
            "orderListId": 1274512,
            "contingencyType": "OCO",
            "listStatusType": "EXEC_STARTED",
            "listOrderStatus": "EXECUTING",
            "listClientOrderId": "08985fedd9ea2cf6b28996",
            "transactionTime": 1660801713793,
            "symbol": "BTCUSDT",
            "orders": [
                {
                    "symbol": "BTCUSDT",
                    "orderId": 12569138901,
                    "clientOrderId": "BqtFCj5odMoWtSqGk2X9tU"
                },
                {
                    "symbol": "BTCUSDT",
                    "orderId": 12569138902,
                    "clientOrderId": "jLnZpj5enfMXTuhKB1d0us"
                }
            ]
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 20
        }
    ]
}
```

<a id="myTrades"></a>
<a id="my-trades"></a>
<a id="mytrades"></a>

<a id="account-trade-history-user_data"></a>

### 账户成交历史 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "myTrades",
    "params": {
        "symbol": "BNBUSDT",
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```
获取账户指定交易对的成交历史，按时间范围过滤。

**权重:**
条件| 权重|
---| ---
|没有 orderId|20|
|有 orderId|5|

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`orderId` | LONG | NO | 必须要和参数`symbol`一起使用。
`startTime` | LONG | NO | 毫秒时间戳
`endTime` | LONG | NO | 毫秒时间戳
`fromId` | INT | NO | 起始交易 ID
`limit` | INT | NO | 默认值: 500; 最大值: 1000
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 缓存 => 数据库

说明：
- 指定 `fromId` 时，返回 trade ID >= `fromId` 的成交记录。
- 指定 `startTime` 和/或 `endTime` 时，按执行时间（`time`）过滤。
- `fromId` 不能与 `startTime`、`endTime` 同时使用。
- 指定 `orderId` 时，仅返回该订单相关成交。
- `startTime`、`endTime` 不能与 `orderId` 同时使用。
- 未指定条件时，返回最近成交记录。
- `startTime` 与 `endTime` 间隔不能超过 24 小时。

**响应:**
```javascript
{
    "id": "f4ce6a53-a29d-4f70-823b-4ab59391d6e8",
    "status": 200,
    "result": [
        {
            "symbol": "BTCUSDT",
            "id": 1650422481,
            "orderId": 12569099453,
            "orderListId": -1,
            "price": "23416.10000000",
            "qty": "0.00635000",
            "quoteQty": "148.69223500",
            "commission": "0.00000000",
            "commissionAsset": "BNB",
            "time": 1660801715793,
            "isBuyer": false,
            "isMaker": true,
            "isBestMatch": true
        },
        {
            "symbol": "BTCUSDT",
            "id": 1650422482,
            "orderId": 12569099453,
            "orderListId": -1,
            "price": "23416.50000000",
            "qty": "0.00212000",
            "quoteQty": "49.64298000",
            "commission": "0.00000000",
            "commissionAsset": "BNB",
            "time": 1660801715793,
            "isBuyer": false,
            "isMaker": true,
            "isBestMatch": true
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 20
        }
    ]
}
```

<a id="account.rateLimits.orders"></a>
<a id="account-rateLimits-orders"></a>
<a id="account-rate-limits-orders"></a>
<a id="account-ratelimits-orders"></a>
<a id="query-unfilled-order-count"></a>

<a id="unfilled-order-count-user_data"></a>

### 查询未成交的订单计数 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "account.rateLimits.orders",
    "params": {
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```
显示用户在所有时间间隔内的未成交订单计数。

**权重:**
40

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 缓存

**响应:**
```javascript
{
    "id": "d3783d8d-f8d1-4d2c-b8a0-b7596af5a664",
    "status": 200,
    "result": [
        {
            "rateLimitType": "ORDERS",
            "interval": "SECOND",
            "intervalNum": 10,
            "limit": 50,
            "count": 0
        },
        {
            "rateLimitType": "ORDERS",
            "interval": "DAY",
            "intervalNum": 1,
            "limit": 160000,
            "count": 0
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 40
        }
    ]
}
```

<a id="myPreventedMatches"></a>
<a id="my-prevented-matches"></a>
<a id="mypreventedmatches"></a>

<a id="account-prevented-matches-user_data"></a>

### 账户的 Prevented Matches (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "myPreventedMatches",
    "params": {
        "symbol": "BTCUSDT",
        "orderId": 35,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
获取因 STP 而过期的订单列表。

这些是支持的组合：

* `symbol` + `preventedMatchId`
* `symbol` + `orderId`
* `symbol` + `orderId` + `fromPreventedMatchId` (`limit`  默认为 500)
* `symbol` + `orderId` + `fromPreventedMatchId` + `limit`

**权重:**
情况                          | 权重
----                          | -----
如果 `symbol` 不合法           | 2
通过 `preventedMatchId` 查询   | 2
通过 `orderId` 查询            | 20

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`preventedMatchId` | LONG | NO | Prevented match ID
`orderId` | LONG | NO | 订单 ID
`fromPreventedMatchId` | LONG | NO | 起始 prevented match ID
`limit` | INT | NO | 默认值: 500; 最大值: 1000
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 数据库

**响应:**
```javascript
{
    "id": "g4ce6a53-a39d-4f71-823b-4ab5r391d6y8",
    "status": 200,
    "result": [
        {
            "symbol": "BTCUSDT",
            "preventedMatchId": 1,
            "takerOrderId": 5,
            "makerSymbol": "BTCUSDT",
            "makerOrderId": 3,
            "tradeGroupId": 1,
            "selfTradePreventionMode": "EXPIRE_MAKER",
            "price": "1.100000",
            "makerPreventedQuantity": "1.300000",
            "transactTime": 1669101687094
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 20
        }
    ]
}
```

<a id="myAllocations"></a>
<a id="my-allocations"></a>
<a id="myallocations"></a>

<a id="account-allocations-user_data"></a>

### 查询分配结果 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "myAllocations",
    "params": {
        "symbol": "BTCUSDT",
        "orderId": 500,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
        "timestamp": 1660801715431
    }
}
```
检索由 SOR 订单生成引起的分配结果。

**权重:**
20

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`startTime` | LONG | NO | 毫秒时间戳
`endTime` | LONG | NO | 毫秒时间戳
`fromAllocationId` | INT | NO | 起始分配 ID
`limit` | INT | NO | 默认值: 500; 最大值: 1000
`orderId` | LONG | NO | 订单 ID
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | NO | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 数据库

支持的参数组合:

参数                                        | 响应      |
------------------------------------------- | -------- |
`symbol`                                    | 按从最旧到最新排序的分配 |
`symbol` + `startTime`                      | 从 `startTime` 开始的最旧的分配 |
`symbol` + `endTime`                        | 到 `endTime` 为止的最新的分配 |
`symbol` + `startTime` + `endTime`          | 在指定时间范围内的分配  |
`symbol` + `fromAllocationId`               | 从指定 `AllocationId` 开始的分配  |
`symbol` + `orderId`                        | 按从最旧到最新排序并和特定订单关联的分配 |
`symbol` + `orderId` + `fromAllocationId`   | 从指定 `AllocationId` 开始并和特定订单关联的分配 |

**注意:** `startTime` 和 `endTime` 之间的时间不能超过 24 小时。

**响应:**
```javascript
{
    "id": "g4ce6a53-a39d-4f71-823b-4ab5r391d6y8",
    "status": 200,
    "result": [
        {
            "symbol": "BTCUSDT",
            "allocationId": 0,
            "allocationType": "SOR",
            "orderId": 500,
            "orderListId": -1,
            "price": "1.00000000",
            "qty": "0.10000000",
            "quoteQty": "0.10000000",
            "commission": "0.00000000",
            "commissionAsset": "BTC",
            "time": 1687319487614,
            "isBuyer": false,
            "isMaker": false,
            "isAllocator": false
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 20
        }
    ]
}
```

<a id="account.commission"></a>
<a id="account-commission"></a>

<a id="account-commission-rates-user_data"></a>

### 账户佣金费率 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "account.commission",
    "params": {
        "symbol": "BNBUSDT",
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```
获取当前账户的佣金费率。

**权重:**
20

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 数据库

**响应:**
```javascript
{
    "id": "d3df8a61-98ea-4fe0-8f4e-0fcea5d418b0",
    "status": 200,
    "result": {
        "symbol": "BTCUSDT",
        // 订单成交的标准佣金率。
        "standardCommission": {
            "maker": "0.00000010",
            "taker": "0.00000020",
            "buyer": "0.00000030",
            "seller": "0.00000040"
        },
        // 订单成交的特殊佣金率。
        "specialCommission": {
            "maker": "0.01000000",
            "taker": "0.02000000",
            "buyer": "0.03000000",
            "seller": "0.04000000"
        },
        // 订单成交的税费率。
        "taxCommission": {
            "maker": "0.00000112",
            "taker": "0.00000114",
            "buyer": "0.00000118",
            "seller": "0.00000116"
        },
        // 使用 BNB 支付时的标准佣金折扣信息。
        "discount": {
            "enabledForAccount": true,
            "enabledForSymbol": true,
            "discountAsset": "BNB",
            "discount": "0.75000000" // 当用BNB支付佣金时，在标准佣金上按此比率打折。
        }
    },
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 20
        }
    ]
}
```

<a id="order.amendments"></a>
<a id="order-amendments"></a>

<a id="query-order-amendments-user_data"></a>

### 查询改单 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "order.amendments",
    "params": {
        "symbol": "BNBUSDT",
        "orderId": 1,
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```
查询对一个订单的所有改单操作。

**权重:**
4

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`orderId` | LONG | YES | 订单 ID
`fromExecutionId` | LONG | NO | 起始执行 ID
`limit` | LONG | NO | 默认值: 500; 最大值: 1000
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 数据库

**响应:**
```javascript
{
    "id": "6f5ebe91-01d9-43ac-be99-57cf062e0e30",
    "status": 200,
    "result": [
        {
            "symbol": "BTCUSDT",
            "orderId": 23,
            "executionId": 60,
            "origClientOrderId": "my_pending_order",
            "newClientOrderId": "xbxXh5SSwaHS7oUEOCI88B",
            "origQty": "7.00000000",
            "newQty": "5.00000000",
            "time": 1741924229819
        }
    ],
    "rateLimits": [
        {
            "rateLimitType": "REQUEST_WEIGHT",
            "interval": "MINUTE",
            "intervalNum": 1,
            "limit": 6000,
            "count": 4
        }
    ]
}
```

<a id="myFilters"></a>
<a id="my-filters"></a>
<a id="myfilters"></a>

<a id="query-relevant-filters-user_data"></a>

### 查询相关过滤器 (USER_DATA)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "myFilters",
    "params": {
        "symbol": "BNBUSDT",
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```
用于检索一个账户上指定交易对的 [filters](filters_CN.md) 列表。这是唯一一个目前会显示账户是否应用了
[`MAX_ASSET`](filters_CN.md#max_asset) 过滤器的方法。

**权重:**
40

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`symbol` | STRING | YES |
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 缓存

**响应:**
```javascript
{
    "id": "1758009606869",
    "status": 200,
    "result": {
        "exchangeFilters": [
            {
                "filterType": "EXCHANGE_MAX_NUM_ORDERS",
                "maxNumOrders": 1000
            }
        ],
        "symbolFilters": [
            {
                "filterType": "PRICE_FILTER",
                "minPrice": "0.00000100",
                "maxPrice": "100000.00000000",
                "tickSize": "0.00000100"
            }
        ],
        "assetFilters": [
            {
                "filterType": "MAX_ASSET",
                "limit": "1000000.00000000",
                "asset": "JPY"
            }
        ],
        "rateLimits": [
            {
                "rateLimitType": "REQUEST_WEIGHT",
                "interval": "MINUTE",
                "intervalNum": 1,
                "limit": 6000,
                "count": 40
            }
        ]
    }
}
```

<a id="user-data-stream-requests"></a>

## 用户数据流请求

<a id="user-data-stream-subscription"></a>

### 用户数据流订阅

<a id=general_info_user_data_stream_subscriptions></a>
**常规信息：**

* [用户数据流订阅](user-data-stream_CN.md)允许您通过 WebSocket 连接接收与指定帐户相关的所有事件。
* 有两种方式可以启用订阅：
  * 如果您拥有已通过验证的会话，则可以使用 [`userDataStream.subscribe`](#user-data-stream-subscribe) 来订阅该已通过验证帐户的相关事件。
  * 在任何会话中，无论是否通过验证，如果您能为相关账户提供 API Key， 那么您可以使用 [`userdataStream.subscribe.signature`](#user-data-stream-subscribe-signature) 来订阅一个或多个账户的相关事件。
  * 一个帐户在指定连接上只能有一个有效订阅。
* 订阅由用订阅时返回的 `subscriptionId` 标识。该 `subscriptionId` 允许您将收到的事件映射到给定的订阅。
* 可以使用 [`session.subscriptions`](#session-subscriptions) 查找会话的所有有效订阅。
* 限制
  * 单个会话最多可同时支持**1,000 个有效订阅**。
    * 尝试启用超出此限制的新订阅将导致错误。
    * 如果您的帐户非常活跃，我们建议您不要一次启用过多的订阅，以免连接过载。
  * 单个会话在其生命周期内最多可处理**65,535 个订阅**。
    * 如果达到此限制，您将收到错误，并且必须重新建立连接才能启用新的订阅。
* 要验证用户数据流订阅的状态，请检查 [`session.status`](#query-session-status) 中的 `userDataStream` 字段：
  * `null` - 此 WebSocket API 上**不提供**用户数据流订阅。
  * `true` - 此会话中**至少有一个有效订阅**。
  * `false` - 此会话中**没有有效订阅**。

<a id="userDataStream.subscribe"></a>
<a id="userDataStream-subscribe"></a>
<a id="user-data-stream-subscribe"></a>
<a id="userdatastream-subscribe"></a>

<a id="subscribe-to-user-data-stream-user_stream"></a>

### 订阅用户数据流 (USER_STREAM)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "userDataStream.subscribe",
    "params": {
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A"
    }
}
```
订阅当前 WebSocket 连接中的用户数据流。

**注意：**
  - 此方法需要使用 Ed25519 密钥并经过鉴权的 WebSocket 连接。请参考 [`session.logon`](web-socket-api_CN.md#log-in-with-api-key-user_data)。
  - 如果需要查看订阅状态，可以通过 [`session.status`](web-socket-api_CN.md#query-session-status) 查询，当 `userDataStream` 字段值为 `true` 时，表示您有一个有效的订阅。
  - 用户数据流在 JSON 和 [SBE 会话](faqs/sbe_faq_CN.md) 中均可用。
    - 有关事件格式详情，请参阅 [用户数据流](user-data-stream_CN.md)。
    - 对于 SBE，仅支持 SBE 模式 2:1 或更高版本。

**权重:**
2

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |

**响应:**
```javascript
{
    "id": "d3df8a21-98ea-4fe0-8f4e-0fcea5d418b7",
    "status": 200,
    "result": {
        "subscriptionId": 0
    }
}
```

<a id="userDataStream.unsubscribe"></a>
<a id="userDataStream-unsubscribe"></a>
<a id="user-data-stream-unsubscribe"></a>
<a id="userdatastream-unsubscribe"></a>

<a id="unsubscribe-from-user-data-stream"></a>

### 取消订阅用户数据流

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "userDataStream.unsubscribe"
}
```
取消订阅当前 WebSocket 连接中的用户数据流。

请注意 `session.logout` 只会关闭由 `userDataStream.subscribe` 创建的订阅，并不会关闭通过 `userDataStream.subscribe.signature` 创建的订阅。

**权重:**
2

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`subscriptionId` | INT | NO | 不带参数调用时，将关闭所有订阅。带 subscriptionId 参数调用时，将尝试关闭该订阅ID对应的订阅（如果存在）。

**响应:**
```javascript
{
    "id": "d3df8a21-98ea-4fe0-8f4e-0fcea5d418b7",
    "status": 200,
    "result": {}
}
```

<a id="session.subscriptions"></a>
<a id="session-subscriptions"></a>
<a id="session-subscription"></a>

<a id="listing-all-subscriptions"></a>

### 显示所有订阅

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "session.subscriptions"
}
```
**注意：**

* 用户按需要跟踪相关帐户的对应订阅情况。

**权重:**
2

**参数:**
NONE

**数据源:** 缓存

**响应:**
```javascript
{
    "id": "d3df5a22-88ea-4fe0-9f4e-0fcea5d418b7",
    "status": 200,
    "result": [
        {
            "subscriptionId": 0
        },
        {
            "subscriptionId": 1
        }
    ]
}
```

<a id="userDataStream.subscribe.signature"></a>
<a id="userDataStream-subscribe-signature"></a>
<a id="user-data-stream-subscribe-signature"></a>
<a id="userdatastream-subscribe-signature"></a>
<a id="user-data-signature"></a>

<a id="subscribe-to-user-data-stream-through-signature-subscription-user_stream"></a>

### 通过签名订阅的方式订阅用户数据流 (USER_STREAM)

```javascript
{
    "id": "5494febb-d167-46a2-996d-70533eb4d976",
    "method": "userDataStream.subscribe.signature",
    "params": {
        "timestamp": 1770736694138,
        "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
        "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88"
    }
}
```

**权重:**
2

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
`apiKey` | STRING | YES |
`recvWindow` | DECIMAL | NO | 最大值： 60000。 <br/> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。
`timestamp` | LONG | YES | 用于请求签名的 Unix 时间戳，单位为毫秒或微秒。该值必须反映客户端当前时间，并会在签名接口中由服务器进行校验。
`signature` | STRING | YES |

**数据源:** 缓存

**响应:**
```javascript
{
    "id": "d3df8a22-98ea-4fe0-9f4e-0fcea5d418b7",
    "status": 200,
    "result": {
        "subscriptionId": 0
    }
}
```
