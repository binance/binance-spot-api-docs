# Binance 的公共 WebSocket API (2023-06-07)

## API 基本信息

* 本篇所列出的 wss 接口的 base URL：**`wss://ws-api.binance.com:443/ws-api/v3`**
  * 如果使用标准443端口时遇到问题，可以使用替代端口9443。
  * [现货测试网](https://testnet.binance.vision)的 base URL 是 `wss://testnet.binance.vision/ws-api/v3`。
* 每个到 base URL 的链接有效期不超过24小时，请妥善处理断线重连。
* WebSocket 服务器将在每3分钟发送一个 **ping 帧**。
  * 如果服务器在10分钟内没有收到 **pong 帧** 响应，会主动断开链接。
  * 允许客户端发送不成对的 **pong 帧**，以防止断开连接。
* 响应中如有数组，数组元素以时间**时间顺序**排列，越早的数据越提前。
* 除非另有说明，所有与时间戳相关的字段均以UTC的**毫秒**为单位。
* 除非另有说明，所有字段名称和值都**大小写敏感**。

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

名称     | 类型    | 是否必需 | 描述
--------- | ------- | --------- | -----------
`id`      | INT / STRING / `null` | YES | 任意的 ID 用于匹配对请求的响应
`method`  | STRING  | YES       | 请求函数名称
`params`  | OBJECT  | NO        | 请求参数。如果没有参数可以省略

* 请求 `id` 是任意的。可以使用 UUID、顺次 ID、当前时间戳等。
  服务器不会以任何方式解释 `id`，只是在响应中回显它。

  可以在一个会话中自由重复使用 ID，不过请注意不要一次发送多个具有相同 ID 的请求，因为否则可能无法区分响应。

* 请求函数名称可以以显式版本为前缀，例如：`"v3/order.place"`

* `params` 的顺序不重要。

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
      "limit": 1200,
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
      "limit": 1200,
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
    <td>速率限制状态。请看 <a href="#速率限制">速率限制</a></td>
  </tr>
</tbody>
</table>

### 状态代码

`status` 字段的状态代码与HTTP的状态代码相同。

一些常见状态代码：

* `200` 代码指示成功响应。
* `4XX` 错误码用于指示错误的请求内容、行为、格式。问题在于请求者。
  * `400` – 错误码表示请求失败，请参阅 `error` 了解原因。
  * `403` – 错误码表示违反 WAF 限制(Web应用程序防火墙)。
  * `409` – 错误码表示请求有一部分成功，一部分失败。请参阅 `error` 了解更多详细。
  * `418` – 表示收到 429 后继续访问，于是被封了。
  * `429` – 错误码表示警告访问频次超限，即将被封IP。
* `5XX` 错误码用于指示Binance服务侧的问题。
  * **重要**：如果响应包含 5xx 状态码，**并不**一定意思请求失败。
    执行状态为 _unknown_，请求可能实际成功。
    请使用 query 函数确认状态。
    建议建立一个新 WebSocket 连接用于确认状态。

有关错误代码和消息的列表，请参阅 [Binance 的错误代码](errors_CN.md)。

# 速率限制

## 连接数量限制

每IP地址、每5分钟最多可以发送300次连接请求。

## 速率限制基本信息

* [`exchangeInfo`](#交易规范信息) 有包含与速率限制相关的信息。
* 根据不同的间隔，有多种频率限制类型。
* 从响应中的可选 `rateLimits` 字段，能看到当前的频率限制状态。
* 如果违反任何速率限制（访问频次限制或下单速率限制），将收到429。

## 如何咨询频率限制

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
      "limit": 1200,
      "count": 70
    }
  ]
}
```

`rate Limits` 数组描述了受请求影响的所有当前的速率限制。

| 名称             | 类型    | 是否必须    | 描述
| --------------- | ------- | --------- | -----------
| `rateLimitType` | ENUM    | YES       | 频率限制类型: `REQUEST_WEIGHT`, `ORDERS`
| `interval`      | ENUM    | YES       | 频率限制间隔: `SECOND`, `MINUTE`, `HOUR`, `DAY`
| `intervalNum`   | INT     | YES       | 频率限制间隔乘数
| `limit`         | INT     | YES       | 每个间隔的请求限制
| `count`         | INT     | YES       | 每个间隔的当前使用情况

频率限制按间隔计算。

例如，`1 MINUTE` 间隔表示每分钟开始。
在 00:01:23.456 提交的请求计入 00:01:00 分钟的限制。
一旦 00:02:00 分钟开始，计数将再次重置为零。

其他间隔的行为方式类似。
例如，`1 DAY` 频率限制是在每天 00:00 UTC 重置，并且 `10 SECOND` 间隔重置为每分钟的 00、10、20...秒。

API 有多种频率限制间隔。
如果您用完了较短的间隔但较长的间隔仍然允许请求，您将不得不等待较短的间隔到期并重置。
如果你用完了更长的间隔，你将不得不等待那个间隔重置，即使较短的频率限制计数为零。

### 如何显示/隐藏频率限制信息

默认情况下，每个响应都包含 `rateLimits` 字段。

但是，频率限制信息可能非常大。
如果您对每个请求的详细频率限制状态不感兴趣，可以从响应中省略 `rateLimits` 字段。

* 请求中的可选 `returnRateLimits` boolean 参数。

  使用 `returnRateLimits` 参数控制是否包含 `rateLimits` 字段以响应单个请求。

  默认请求和响应：

  ```json
  {"id":1,"method":"time"}
  ```

  ```json
  {"id":1,"status":200,"result":{"serverTime":1656400526260},"rateLimits":[{"rateLimitType":"REQUEST_WEIGHT","interval":"MINUTE","intervalNum":1,"limit":1200,"count":70}]}
  ```

  没有频率限制状态的请求和响应：

  ```json
  {"id":2,"method":"time","params":{"returnRateLimits":false}}
  ```

  ```json
  {"id":2,"status":200,"result":{"serverTime":1656400527891}}
  ```

* 连接 URL 中可选的 `returnRateLimits` boolean 参数。

  如果您希望在默认情况下从所有响应中省略 `rateLimits`，可以在 query string 中使用 `returnRateLimits` 参数：

  ```
  wss://ws-api.binance.com/ws-api/v3?returnRateLimits=false
  ```

  这将使通过此连接发出的所有请求的行为就像您已传了 `"returnRateLimits"：false` 一样。

  如果您_想_查看特定请求的频率限制，您需要特定传 `"returnRateLimits"：true` 参数。

**注意:** 如果您在响应中隐藏 `rateLimits` 字段，您的请求仍然还是会受到频率限制的。


## IP 访问限制

* 每个请求都有一个特定的 **权重**，它会添加到您的访问限制中。
  * 大多数请求是1个权重单位。越消耗资源的接口权重就会越大。
  * 连接到 WebSocket API 会用到1个权重。
* 当前权重使用由 `REQUEST_WEIGHT` 频率限制类型指示。
* 请使用[`exchangeInfo`](#交易规范信息)请求来跟踪当前的重量限制。
* 权重是基于**每个 IP 地址**累积的，并由来自该地址的所有连接共享。
* 如果超多限制，客服端会收到 `429`。
  * 这错误代码表示您有责任停止发送请求，不得滥用API。
  * 响应会包含一个 `retryAfter` 字段，指示在什么时候您能重试。
* **屡次违反速率限制或者在收到429后未能退缩将导致自动 IP 封禁和断开连接。**
  * 被禁止 IP 地址的请求失败，状态为 `418`。
  * `retryAfter` 字段表示解除禁令的timestamp。
* 频繁违反限制的封禁时间会**逐渐延长**，**从最短2分钟到最长3天**。

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
      "limit": 1200,
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
      "limit": 1200,
      "count": 2411
    }
  ]
}
```

## 下单速率限制
* 每个下单请求都会计入到**订单限额**。
  * 成功的订单会更新 `ORDERS` 频率限制类型。
  * 被拒绝或不成功的订单可能会也可能不会更新 `ORDERS` 计数。
* 使用 [`account.rateLimits.orders`](#账户订单率限制-user_data) 请求跟踪当前的订单率限制。
* 订单速率限制适用于**每一个账户**，并由账户的所有 API key 共享。
* 如果您超过订单速率限制，请求会失败，状态为 `429`。
  * 这错误代码表示您有责任停止发送请求，不得滥用API。
  * 响应会包含一个 `retryAfter` 字段，指示什么时候能在重试。

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
      "limit": 1200,
      "count": 321
    }
  ]
}
```

# 请求鉴权类型

* 每个函数都有自己的鉴权类型，鉴权类型决定了访问时应当进行何种鉴权。
  * 鉴权类型会在本文档中各个函数名称旁声明。比如 [下新的订单 (TRADE)](#下新的订单-trade)。
  * 如果没有特殊声明即默认为 NONE

鉴权类型 | API key | 签名 | 描述
------------- | -------- | --------- | ------------
`NONE`        |          |           | 公开市场数据
`TRADE`       | required | required  | 在交易所交易，下单和取消订单
`USER_DATA`   | required | required  | 私人账户信息，例如订单状态和交易历史
`USER_STREAM` | required |           | 管理用户数据流订阅
`MARKET_DATA` | required |           | 历史市场数据访问

* 函数鉴权需要指定和验证有效的 API key。
  * API key 可以在币安账户的 [API 管理](https://www.binance.com/zh-CN/support/faq/%E5%A6%82%E4%BD%95%E5%88%9B%E5%BB%BAapi-360002502072) 页面创建。
  * **API key和密钥都是大小写敏感的。** 切勿与任何人共享它们。
    如果您发现您的账户有异常活动，请立即撤销所有keys并联系币安客服。
* API key 可以配置为仅允许访问某些类型的函数鉴权。
  * 例如，可以拥有一个 API key 有 `TRADE` 的权限进行交易，同时使用另一个 API key 有 `USER_DATA` 的权限来监控您的订单状态。
  * 默认情况下，API key 不能 `交易`。您需要先在 API 管理中开通交易权限。
* `TRADE` 和 `USER_DATA` 请求也称为 `SIGNED` 请求。

## SIGNED (TRADE 和 USER_DATA) 请求鉴权

* 为了授权请求，`SIGNED` 请求必须带 `signature` 参数。
* 请参考 [签名请求示例（HMAC）](#SIGNED-请求示例-HMAC)和 [签名请求示例（RSA）](#SIGNED-请求示例-RSA)理解如何计算签名。

## 时间同步安全

* `SIGNED` 请求也必须发 `timestamp` 参数，其值应当是请求发送时刻的 unix 时间戳(毫秒)。
* 还可以发送一个可选参数 `recvWindow`，指定请求保持有效的时间。
  * 如果 `recvWindow` 未发送，**默认为5000毫秒**。
  * 最大 `recvWindow` 为60000毫秒。

* 请求处理逻辑:

  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // 处理请求
  } else {
    // 拒绝请求
  }
  ```

**关于交易时效性** 互联网状况并不完全稳定可靠，因此你的程序本地到币安服务器的时延会有抖动, 这是我们设置 `recvWindow` 的目所在。如果你从事高频交易，对交易时效性有较高的要求，可以灵活设置 `recvWindow` 以达到你的要求。

**建议使用5000毫秒以下的 `recvWindow`！**

## SIGNED 请求示例 (HMAC)

这是有关如何用 HMAC secret key 签署请求的分步指南。

示例 API key 和 secret key：

Key          | Value
------------ | ------------
apiKey       | `vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A`
secretKey    | `NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j`

**警告：请勿与任何人共享您的私钥。** 

示例密钥仅供参考。

请求示例：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol":           "BTCUSDT",
    "side":             "SELL",
    "type":             "LIMIT",
    "timeInForce":      "GTC",
    "quantity":         "0.01000000",
    "price":            "52000.00",
    "newOrderRespType": "ACK",
    "recvWindow":       100,
    "timestamp":        1645423376532,
    "apiKey":           "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature":        "------ 未填写 ------"
  }
}
```

如您所见，目前缺少 `signature` 参数。

**第一步：创建签名内容**

除了 `signature` 之外，获取所有请求 `params`，然后按名称按字母顺序进行排序：

参数              | 取值
---------------- | ------------
apiKey           | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
newOrderRespType | ACK
price            | 52000.00
quantity         | 0.01000000
recvWindow       | 100
side             | SELL
symbol           | BTCUSDT
timeInForce      | GTC
timestamp        | 1645423376532
type             | LIMIT

将参数格式化为 `参数=取值` 对并用 `&` 分隔每个参数：

签名效载荷示例：
```
apiKey=vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A&newOrderRespType=ACK&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT
```

**第二步：计算签名**

1. 将 `secretKey` 解释为 ASCII 数据，将其用作 HMAC-SHA-256 的 key。
2. 将签名有效负载签名为 ASCII 数据。
3. 将 HMAC-SHA-256 输出编码为 hex string。

请注意，`apiKey`、`secretKey` 和有效负载是**大小写敏感的**。
生成的签名值是不区分大小写的。

可以使用 OpenSSL 交叉检查您的签名算法实现：

```console
$ echo -n 'apiKey=vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A&newOrderRespType=ACK&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT' \
  | openssl dgst -hex -sha256 -hmac 'NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j'

cc15477742bd704c29492d96c7ead9414dfd8e0ec4a00f947bb5bb454ddbd08a
```

**第三步：添加 `signature` 到 `params` 中**

最后，使用生成的签名完成请求：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol":           "BTCUSDT",
    "side":             "SELL",
    "type":             "LIMIT",
    "timeInForce":      "GTC",
    "quantity":         "0.01000000",
    "price":            "52000.00",
    "newOrderRespType": "ACK",
    "recvWindow":       100,
    "timestamp":        1645423376532,
    "apiKey":           "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature":        "cc15477742bd704c29492d96c7ead9414dfd8e0ec4a00f947bb5bb454ddbd08a"
  }
}
```

## SIGNED 请求示例 (RSA)

Key          | Value
------------ | ------------
apiKey       | `CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ`

对于这个例子，Private Key 将被引用为 `test-prv-key.pem`。

**警告：请勿与任何人共享您的私钥。** 示例密钥仅供参考。

请求例子：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol":           "BTCUSDT",
    "side":             "SELL",
    "type":             "LIMIT",
    "timeInForce":      "GTC",
    "quantity":         "0.01000000",
    "price":            "52000.00",
    "newOrderRespType": "ACK",
    "recvWindow":       100,
    "timestamp":        1645423376532,
    "apiKey":           "CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ",
    "signature":        "------ FILL ME ------"
  }
}
```

**第一步：计算 Payload**

除了 `signature`，获取请求的所有其它 `params`，然后按名称字母顺序对它们进行排序：

参数              | 取值
---------------- | ------------
apiKey           | CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ
newOrderRespType | ACK
price            | 52000.00
quantity         | 0.01000000
recvWindow       | 100
side             | SELL
symbol           | BTCUSDT
timeInForce      | GTC
timestamp        | 1645423376532
type             | LIMIT

将参数格式化为 `参数=取值` 对并用 `&` 分隔每个参数：

```
apiKey=CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ&newOrderRespType=ACK&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT
```

**第二步：计算签名**

1. 将签名有效负载编码为 ASCII 数据。
2. 使用带有 SHA-256 hash 函数的 RSASSA-PKCS1-v1_5 算法对 payload 进行签名。
3. 将输出编码为 base64 string。

请注意，`apiKey`, payload 和生成的签名值都是**大小写敏感**的。

您可以使用 OpenSSL 交叉检查您的签名算法：

```console
$ echo -n 'apiKey=CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ&newOrderRespType=ACK&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT' \
  | openssl dgst -sha256 -sign test-prv-key.pem \
  | openssl enc -base64 -A

OJJaf8C/3VGrU4ATTR4GiUDqL2FboSE1Qw7UnnoYNfXTXHubIl1iaePGuGyfct4NPu5oVEZCH4Q6ZStfB1w4ssgu0uiB/Bg+fBrRFfVgVaLKBdYHMvT+ljUJzqVaeoThG9oXlduiw8PbS9U8DYAbDvWN3jqZLo4Z2YJbyovyDAvDTr/oC0+vssLqP7NmlNb3fF3Bj7StmOwJvQJTbRAtzxK5PP7OQe+0mbW+D7RqVkUiSswR8qJFWTeSe4nXXNIdZdueYhF/Xf25L+KitJS5IHdIHcKfEw3MQzHFb2ZsGWkjDQwxkwr7Noi0Zaa+gFtxCuatGFm9dFIyx217pmSHtA==
```

**第三步：在请求的 `params` 参数添加签名值**

最后，使用生成的签名完成请求：

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol":           "BTCUSDT",
    "side":             "SELL",
    "type":             "LIMIT",
    "timeInForce":      "GTC",
    "quantity":         "0.01000000",
    "price":            "52000.00",
    "newOrderRespType": "ACK",
    "recvWindow":       100,
    "timestamp":        1645423376532,
    "apiKey":           "CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ",
    "signature":        "OJJaf8C/3VGrU4ATTR4GiUDqL2FboSE1Qw7UnnoYNfXTXHubIl1iaePGuGyfct4NPu5oVEZCH4Q6ZStfB1w4ssgu0uiB/Bg+fBrRFfVgVaLKBdYHMvT+ljUJzqVaeoThG9oXlduiw8PbS9U8DYAbDvWN3jqZLo4Z2YJbyovyDAvDTr/oC0+vssLqP7NmlNb3fF3Bj7StmOwJvQJTbRAtzxK5PP7OQe+0mbW+D7RqVkUiSswR8qJFWTeSe4nXXNIdZdueYhF/Xf25L+KitJS5IHdIHcKfEw3MQzHFb2ZsGWkjDQwxkwr7Noi0Zaa+gFtxCuatGFm9dFIyx217pmSHtA=="
  }
}
```


# 数据源

* API 系统是异步的。响应中一些延迟是正常和预期的。

* 每种函数都会有一个数据源, 用来指示数据的来源，以及它的最新程度。

数据源 | 延迟 | 描述
--------------- | -------- | ------------
撮合引擎 | 最低 | 表示撮合引擎直接产生响应
缓存 | 低 | 表示数据来源于内部或者外部的缓存
数据库 | 中度 | 表示数据直接来源于数据库

* 某些函数有多个数据源（例如，缓存 => 数据库）。

  这代表 API 将按该顺序查找最新数据：首先是缓存，然后是数据库。

# 公共 API 请求

### 术语

这些术语将在整个文档中使用，因此特别建议新用户阅读。

* `base asset` 是指作为交易对中的 `quantity` 资产。对于交易对 BTCUSDT，BTC 将是 `base asset`。
* `quote asset` 是指作为交易对中的 `price` 资产。对于交易对 BTCUSDT，USDT 将是 `quote asset`。

## ENUM 定义
**交易对状态 (status):**

* `PRE_TRADING`
* `TRADING`
* `POST_TRADING`
* `END_OF_DAY`
* `HALT`
* `AUCTION_MATCH`
* `BREAK`

<a id="permissions"></a>

**账户和交易对权限 (permissions):**

* `SPOT`
* `MARGIN`
* `LEVERAGED`
* `TRD_GRP_002`
* `TRD_GRP_003`
* `TRD_GRP_004`
* `TRD_GRP_005`
* `TRD_GRP_006`
* `TRD_GRP_007`
* `TRD_GRP_008`
* `TRD_GRP_009`
* `TRD_GRP_010`
* `TRD_GRP_011`
* `TRD_GRP_012`
* `TRD_GRP_013`


**订单状态 (status):**

状态 | 描述
-----------| --------------
`NEW` | 订单被交易引擎接受
`PARTIALLY_FILLED`| 部分订单被成交
`FILLED` | 订单完全成交
`CANCELED` | 用户撤销了订单
`PENDING_CANCEL` | 撤销中(目前并未使用)
`REJECTED`       | 订单没有被交易引擎接受，也没被处理
`EXPIRED` | 订单被交易引擎取消 （比如 LIMIT FOK 订单没有成交，LIMIT IOC 或者 市价单 没有完全成交）</br> 强平期间被取消的订单 （交易所维护期间被取消的订单）
`EXPIRED_IN_MATCH` | 表示订单由于 STP 而过期（e.g. 带有 `EXPIRE_TAKER` 的订单与订单簿上属于同账户或同 `tradeGroupId` 的订单撮合）

**OCO 状态 (listStatusType):**

状态 | 描述
-----------| --------------
`RESPONSE` | 当`ListStatus`响应失败的操作时使用。(订单完成或取消订单)
`EXEC_STARTED`| 当已经下单或者订单有更新时
`ALL_DONE` | 当订单执行结束或者不在激活状态

**OCO 订单状态 (listOrderStatus):**

状态 | 描述
-----------| --------------
`EXECUTING` | 当已经下单或者订单有更新时
`ALL_DONE`| 当订单执行结束或者不在激活状态
`REJECT` | 当订单状态响应失败(订单完成或取消订单)

**指定订单的类型**
* `OCO`

## 常用请求信息

### 测试连通性

```javascript
{
  "id": "922bcc6e-9de8-440d-9e84-7c80933a8d0d",
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

**数据源:**
缓存

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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### 检查服务器时间

```javascript
{
  "id": "187d3cb2-942d-484c-8271-4e2141bbadb1",
  "method": "time"
}
```

测试与 WebSocket API 的连通性并获取当前服务器时间。

**权重:**
1

**参数:**
NONE

**数据源:**
缓存

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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### 交易规范信息

```javascript
{
  "id": "5494febb-d167-46a2-996d-70533eb4d976",
  "method": "exchangeInfo",
  "params": {
    "symbols": ["BNBBTC"]
  }
}
```

获取交易规则，速率限制，和交易对信息。

**权重:**
10

**参数:**

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td rowspan="3" align="center">NO</td>
        <td>代表单个交易对</td>
    </tr>
    <tr>
        <td><code>symbols</code></td>
        <td>ARRAY of STRING</td>
        <td>代表多个交易对</td>
    </tr>
    <tr>
        <td><code>permissions</code></td>
        <td>ARRAY of STRING</td>
        <td>按权限过滤交易对</td>
    </tr>
</tbody>
</table>

备注：

* 只能指定 `symbol`，`symbols`，`permissions` 参数之一。

* 如果没有参数，`exchangeInfo` 会显示所有 `["SPOT, "MARGIN", "LEVERAGED"]` 权限的交易对。

  * 如果想显示交易的 *所有* 活动交易对，您需要明确请求所有权限。

* `permissions` 接受多个权限或单个权限名称, 比如 `"SPOT"`。

* [可用权限](#permissions)


**数据源:**
缓存

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
        "rateLimitType": "REQUEST_WEIGHT",    // 速率限制类型: REQUEST_WEIGHT，ORDERS，RAW_REQUESTS
        "interval": "MINUTE",                 // 速率限制间隔: SECOND，MINUTE，DAY
        "intervalNum": 1,                     // 速率限制间隔乘数 (i.e.，"1 minute")
        "limit": 1200                         // 每个间隔的速率限制
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
        "rateLimitType": "RAW_REQUESTS",
        "interval": "MINUTE",
        "intervalNum": 5,
        "limit": 6100
      }
    ],
    // 交易所级别过滤器在 "过滤器" 页面上进行了说明：
    // https://github.com/binance/binance-spot-api-docs/blob/master/filters_CN.md
    // 全部交易过滤器是可选的。
    "exchangeFilters": [],
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
        "quoteOrderQtyMarketAllowed": true,
        "allowTrailingStop": true,
        "cancelReplaceAllowed": true,
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
          "SPOT",
          "MARGIN",
          "TRD_GRP_004"
        ],
        "defaultSelfTradePreventionMode": "NONE",
        "allowedSelfTradePreventionModes": [
          "NONE"
        ]
      }
    ]
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 10
    }
  ]
}
```

## 行情接口

### 订单薄深度信息

```javascript
{
  "id": "51e2affb-0aba-4821-ba75-f2625006eb43",
  "method": "depth",
  "params": {
    "symbol": "BNBBTC",
    "limit": 5
  }
}
```

获取当前深度信息。

请注意，此请求返回有限的市场深度。

如果需要持续监控深度信息更新，请考虑使用 WebSocket Streams：

* [`<symbol>@depth<levels>`](web-socket-streams_CN.md#有限档深度信息)
* [`<symbol>@depth`](web-socket-streams_CN.md#增量深度信息stream)

如果需要[维护本地orderbook](web-socket-streams_CN.md#如何正确在本地维护一个orderbook副本)，您可以将 `depth` 请求与 `<symbol>@depth` streams 一起使用。

**权重:**
根据限制调整：

| 限制    | 重量 |
|:---------:|:------:|
|     1–100 |      1 |
|   101–500 |      5 |
|  501–1000 |     10 |
| 1001–5000 |     50 |

**参数:**

名称      | 类型    | 是否必需 | 描述
--------- | ------- | --------- | -----------
`symbol`  | STRING  | YES       |
`limit`   | INT     | NO        | 默认 100; 最大值 5000

**数据源:**
缓存

**响应:**
```javascript
{
  "id": "51e2affb-0aba-4821-ba75-f2625006eb43",
  "status": 200,
  "result": {
    "lastUpdateId": 2731179239,
    // bid 水平从最高价到最低价排序。
    "bids": [
      [
        "0.01379900",   // 价格
        "3.43200000"    // 重量
      ],
      [
        "0.01379800",
        "3.24300000"
      ],
      [
        "0.01379700",
        "10.45500000"
      ],
      [
        "0.01379600",
        "3.82100000"
      ],
      [
        "0.01379500",
        "10.26200000"
      ]
    ],
    // ask 水平从最低价到最高价排序。
    "asks": [
      [
        "0.01380000",
        "5.91700000"
      ],
      [
        "0.01380100",
        "6.01400000"
      ],
      [
        "0.01380200",
        "0.26800000"
      ],
      [
        "0.01380300",
        "0.33800000"
      ],
      [
        "0.01380400",
        "0.26800000"
      ]
    ]
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```


### 最近的交易

```javascript
{
  "id": "409a20bd-253d-41db-a6dd-687862a5882f",
  "method": "trades.recent",
  "params": {
    "symbol": "BNBBTC",
    "limit": 1
  }
}
```

获取最近的交易

如果您需要访问实时交易活动，请考虑使用 WebSocket Streams：

* [`<symbol>@trade`](web-socket-streams_CN.md#逐笔交易)

**权重:**
1

**参数:**

名称      | 类型    | 是否必需 | 描述
--------- | ------- | --------- | -----------
`symbol`  | STRING  | YES       |
`limit`   | INT     | NO        | 默认 500; 最大值 1000

**数据源:**
缓存

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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### 历史交易 (MARKET_DATA)

```javascript
{
  "id": "cffc9c7d-4efc-4ce0-b587-6b87448f052a",
  "method": "trades.historical",
  "params": {
    "symbol": "BNBBTC",
    "fromId": 0,
    "limit": 1,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A"
  }
}
```

获取历史交易。

**权重:**
5

**参数:**

名称      | 类型    | 是否必需 | 描述
--------- | ------- | --------- | -----------
`symbol`  | STRING  | YES       |
`fromId`  | INT     | NO        | 起始交易ID
`limit`   | INT     | NO        | 默认 500; 最大值 1000
`apiKey`  | STRING  | YES       |

备注：

* 如果 `fromId` 未指定，则返回最近的交易。

**数据源:**
数据库

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
      "limit": 1200,
      "count": 5
    }
  ]
}
```

### 归集交易

```javascript
{
  "id": "189da436-d4bd-48ca-9f95-9f613d621717",
  "method": "trades.aggregate",
  "params": {
    "symbol": "BNBBTC",
    "fromId": 50000000,
    "limit": 1
  }
}
```

获取归集交易。

一个 *归集交易* (aggtrade) 代表一个或多个单独的交易。
同时间，同 taker 订单和同价格的执行交易会被聚合为一条归集交易。

如果需要访问实时交易活动，请考虑使用 WebSocket Streams：

* [`<symbol>@aggTrade`](web-socket-streams_CN.md#归集交易)

如果需要历史总交易数据，可以使用 [data.binance.vision](https://github.com/binance/binance-public-data/#aggtrades)。

**权重:**
1

**参数:**

名称        | 类型    | 是否必需 | 描述
----------- | ------- | --------- | -----------
`symbol`    | STRING  | YES       |
`fromId`    | INT     | NO        | 起始归集交易ID
`startTime` | INT     | NO        |
`endTime`   | INT     | NO        |
`limit`     | INT     | NO        | 默认 500; 最大值 1000

备注：

* 如果指定了 `fromId`，则返回归集交易 ID >= `fromId` 的 aggtrades。

  使用 `fromId` 和 `limit` 会对所有 aggtrades 进行分页。

* 如果指定了 `startTime` 和/或 `endTime`，响应中的 aggtrades 会按照执行时间 (`T`) 过滤。

  `fromId` 不能与 `startTime` 和 `endTime` 一起使用。

* 如果未指定条件，则返回最近的归集交易。

**数据源:**
数据库

**响应:**
```javascript
{
  "id": "189da436-d4bd-48ca-9f95-9f613d621717",
  "status": 200,
  "result": [
    {
      "a": 50000000,        // 归集交易ID
      "p": "0.00274100",    // 价格
      "q": "57.19000000",   // 重量
      "f": 59120167,        // 被归集的首个交易ID
      "l": 59120170,        // 被归集的末次交易ID
      "T": 1565877971222,   // 时间戳
      "m": true,            // 买方是否是做市方。如true，则此次成交是一个主动卖出单，否则是一个主动买入单。
      "M": true             // 交易是否是最好价格匹配。
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### K线数据

```javascript
{
  "id": "1dbbeb56-8eea-466a-8f6e-86bdcfa2fc0b",
  "method": "klines",
  "params": {
    "symbol": "BNBBTC",
    "interval": "1h",
    "startTime": 1655969280000,
    "limit": 1
  }
}
```

获取K线数据。

Klines 由其开盘时间和收盘时间为唯一标识。

如果您需要访问实时 kline 更新，请考虑使用 WebSocket Streams：

* [`<symbol>@kline_<interval>`](web-socket-streams_CN.md#K线)

如果需要历史K线数据，可以使用 [data.binance.vision](https://github.com/binance/binance-public-data/#klines)。

**权重:**
1

**参数:**

名称        | 类型    | 是否必需 | 描述
----------- | ------- | --------- | -----------
`symbol`    | STRING  | YES       |
`interval`  | ENUM    | YES       |
`startTime` | INT     | NO        |
`endTime`   | INT     | NO        |
`limit`     | INT     | NO        | 默认 500; 最大值 1000

<a id="kline-intervals"></a>
支持的 kline 间隔（大小写敏感）：

时间间隔  | `interval` 值
--------- | ----------------
seconds   | `1s`
minutes   | `1m`, `3m`, `5m`, `15m`, `30m`
hours     | `1h`, `2h`, `4h`, `6h`, `8h`, `12h`
days      | `1d`, `3d`
weeks     | `1w`
months    | `1M`

备注:

* 如果没有指定 `startTime`，`endTime`，则返回最近的klines。

**数据源:**
数据库

**响应:**
```javascript
{
  "id": "1dbbeb56-8eea-466a-8f6e-86bdcfa2fc0b",
  "status": 200,
  "result": [
    [
      1655971200000,      // 这根K线的起始时间
      "0.01086000",       // 这根K线期间第一笔成交价
      "0.01086600",       // 这根K线期间最高成交价
      "0.01083600",       // 这根K线期间最低成交价
      "0.01083800",       // 这根K线期间末一笔成交价
      "2290.53800000",    // 这根K线期间成交量
      1655974799999,      // 这根K线的结束时间
      "24.85074442",      // 这根K线期间成交额
      2283,               // 这根K线期间成交笔数
      "1171.64000000",    // 主动买入的成交量
      "12.71225884",      // 主动买入的成交额
      "0"                 // 忽略此参数
    ]
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### UI K线数据

```javascript
{
  "id": "b137468a-fb20-4c06-bd6b-625148eec958",
  "method": "uiKlines",
  "params": {
    "symbol": "BNBBTC",
    "interval": "1h",
    "startTime": 1655969280000,
    "limit": 1
  }
}
```

请求参数和响应字段与[`k线`](#K线数据)接口相同。
uiKlines 是返回修改后的k线数据，针对k线图的呈现进行了优化。

**权重:**
1

**参数:**

名称        | 类型    | 是否必需 | 描述
----------- | ------- | --------- | -----------
`symbol`    | STRING  | YES       |
`interval`  | ENUM    | YES       | 请看 [`k线`](#kline-intervals)
`startTime` | INT     | NO        |
`endTime`   | INT     | NO        |
`limit`     | INT     | NO        | 默认 500; 最大值 1000

备注:

* 如果没有指定 `startTime`，`endTime`，则返回最近的klines。

**数据源:**
数据库

**响应:**
```javascript
{
  "id": "b137468a-fb20-4c06-bd6b-625148eec958",
  "status": 200,
  "result": [
    [
      1655971200000,      // 这根K线的起始时间
      "0.01086000",       // 这根K线期间第一笔成交价
      "0.01086600",       // 这根K线期间最高成交价
      "0.01083600",       // 这根K线期间最低成交价
      "0.01083800",       // 这根K线期间末一笔成交价
      "2290.53800000",    // 这根K线期间成交量
      1655974799999,      // 这根K线的结束时间
      "24.85074442",      // 这根K线期间成交额
      2283,               // 这根K线期间成交笔数
      "1171.64000000",    // 主动买入的成交量
      "12.71225884",      // 主动买入的成交额
      "0"                 // 忽略此参数
    ]
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### 当前平均价格

```javascript
{
  "id": "ddbfb65f-9ebf-42ec-8240-8f0f91de0867",
  "method": "avgPrice",
  "params": {
    "symbol": "BNBBTC"
  }
}
```

获取交易对的当前平均价格

**权重:**
1

**参数:**

名称        | 类型    | 是否必需 | 描述
----------- | ------- | --------- | -----------
`symbol`    | STRING  | YES       |

**数据源:**
缓存

**响应:**
```javascript
{
  "id": "ddbfb65f-9ebf-42ec-8240-8f0f91de0867",
  "status": 200,
  "result": {
    "mins": 5,              // 以分钟为单位的价格平均间隔 
    "price": "0.01378135"
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### 24hr 价格变动情况

```javascript
{
  "id": "93fb61ef-89f8-4d6e-b022-4f035a3fadad",
  "method": "ticker.24hr",
  "params": {
    "symbol": "BNBBTC"
  }
}
```

24 小时滚动窗口价格变动数据。
如果您需要持续监控交易统计，请考虑使用 WebSocket Streams:

* [`<symbol>@ticker`](web-socket-streams_CN.md#按Symbol的完整Ticker) 或者 [`!ticker@arr`](web-socket-streams_CN.md#全市场所有交易对的完整Ticker)
* [`<symbol>@miniTicker`](web-socket-streams_CN.md#按Symbol的精简Ticker) 或者 [`!miniTicker@arr`](web-socket-streams_CN.md#全市场所有Symbol的精简Ticker)

如果你想用不同的窗口数量，可以用 [`ticker`](#滚动窗口价格变动统计) 请求。

**权重:**
根据交易对的数量进行调整：

| 交易对     | 重量    |
|:---------:|:------:|
|      1–20 |      1 |
|    21–100 |     20 |
|   101 以上 |     40 |
|  全部交易对 |     40 |

**参数:**

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td rowspan="2" align="center">NO</td>
        <td>获取单个交易对的 ticker</td>
    </tr>
    <tr>
        <td><code>symbols</code></td>
        <td>ARRAY of STRING</td>
        <td>获取多个交易对的 ticker</td>
    </tr>
    <tr>
        <td><code>type</code></td>
        <td>ENUM</td>
        <td align=center>NO</td>
        <td>Ticker 类型: <code>FULL</code> (默认) 或者 <code>MINI</code></td>
    </tr>
</tbody>
</table>

备注:

* `symbol` 和 `symbols` 不能同时用。

* 如果未指定交易对，则返回有关当前在交易所交易的所有交易对的信息。

**数据源:**
缓存

**响应:**

`FULL` 类型，对于单个交易对：

```javascript
{
  "id": "93fb61ef-89f8-4d6e-b022-4f035a3fadad",
  "status": 200,
  "result": {
    "symbol": "BNBBTC",
    "priceChange": "0.00013900",
    "priceChangePercent": "1.020",
    "weightedAvgPrice": "0.01382453",
    "prevClosePrice": "0.01362800",
    "lastPrice": "0.01376700",
    "lastQty": "1.78800000",
    "bidPrice": "0.01376700",
    "bidQty": "4.64600000",
    "askPrice": "0.01376800",
    "askQty": "14.31400000",
    "openPrice": "0.01362800",
    "highPrice": "0.01414900",
    "lowPrice": "0.01346600",
    "volume": "69412.40500000",
    "quoteVolume": "959.59411487",
    "openTime": 1660014164909,
    "closeTime": 1660100564909,
    "firstId": 194696115,       // 第一个交易 ID
    "lastId": 194968287,        // 最后一个交易 ID
    "count": 272173             // 成交笔数
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
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
    "symbol": "BNBBTC",
    "openPrice": "0.01362800",
    "highPrice": "0.01414900",
    "lowPrice": "0.01346600",
    "lastPrice": "0.01376700",
    "volume": "69412.40500000",
    "quoteVolume": "959.59411487",
    "openTime": 1660014164909,
    "closeTime": 1660100564909,
    "firstId": 194696115,       // 第一个交易 ID
    "lastId": 194968287,        // 最后一个交易ID
    "count": 272173             // 成交笔数
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
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
      "symbol": "BNBBTC",
      "priceChange": "0.00016500",
      "priceChangePercent": "1.213",
      "weightedAvgPrice": "0.01382508",
      "prevClosePrice": "0.01360800",
      "lastPrice": "0.01377200",
      "lastQty": "1.01400000",
      "bidPrice": "0.01377100",
      "bidQty": "7.55700000",
      "askPrice": "0.01377200",
      "askQty": "4.37900000",
      "openPrice": "0.01360700",
      "highPrice": "0.01414900",
      "lowPrice": "0.01346600",
      "volume": "69376.27900000",
      "quoteVolume": "959.13277091",
      "openTime": 1660014615517,
      "closeTime": 1660101015517,
      "firstId": 194697254,
      "lastId": 194969483,
      "count": 272230
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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### 滚动窗口价格变动统计

```javascript
{
  "id": "f4b3b507-c8f2-442a-81a6-b2f12daa030f",
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

这个请求类似于 [`ticker.24hr`](#24hr-价格变动情况)，但统计数据是使用指定的任意窗口按需计算的。

**注意：** 窗口大小精度限制为1分钟。
虽然 `closeTime` 是请求的当前时间，`openTime` 总是从分钟边界开始。
因此，有效窗口可能比请求的 `windowSize` 宽59999毫秒。

<details>
<summary>窗口计算示例</summary>


例如，对 `"windowSize": "7d"` 的请求可能会导致以下窗口：

```javascript
"openTime": 1659580020000,
"closeTime": 1660184865291,
```

请求的时间 - `closeTime` - 是 1660184865291（2022年8月11日 02:27:45.291）。
请求的窗口大小应将 `openTime` 设置为7天之前 – 8月4日，02:27:45.291 – 但由于精度有限，它最终会提前一点：1659580020000（2022年8月4日 02:27:00），正好在一分钟开始。

</details>

如果您需要持续监控交易统计，请考虑使用 WebSocket Streams:

* [`<symbol>@ticker_<window_size>`](web-socket-streams_CN.md#按Symbol的滚动窗口统计) 或者 [`!ticker_<window-size>@arr`](web-socket-streams_CN.md#全市场滚动窗口统计)

**权重:**
根据交易对的数量进行调整：

| 交易对   | 重量    |
|:-------:|:------:|
|    1–50 | 2 per symbol |
|  51–100 |    100 |

**参数:**

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td rowspan="2" align="center">YES</td>
        <td>获取单个交易对的 ticker</td>
    </tr>
    <tr>
        <td><code>symbols</code></td>
        <td>ARRAY of STRING</td>
        <td>获取多个交易对的 ticker</td>
    </tr>
    <tr>
        <td><code>type</code></td>
        <td>ENUM</td>
        <td align=center>NO</td>
        <td>Ticker 类型： <code>FULL</code> (默认) 或者 <code>MINI</code></td>
    </tr>
    <tr>
        <td><code>windowSize</code></td>
        <td>ENUM</td>
        <td align=center>NO</td>
        <td>默认 <code>1d</code></td>
    </tr>
</tbody>
</table>

支持的窗口 size:

单位    | `windowSize` 值
------- | ------------------
minutes | `1m`, `2m` ... `59m`
hours   | `1h`, `2h` ... `23h`
days    | `1d`, `2d` ... `7d`

备注：

* 必须指定 `symbol` 或 `symbols`。

* 一个请求中的最大交易对数：100。

* 窗口 size 单位不能合并。
  比如不支持 <code>1d 2h</code>。

**数据源:**
数据库

**响应:**

`FULL` 类型，对于单个交易对：

```javascript
{
  "id": "f4b3b507-c8f2-442a-81a6-b2f12daa030f",
  "status": 200,
  "result": {
    "symbol": "BNBBTC",
    "priceChange": "0.00061500",
    "priceChangePercent": "4.735",
    "weightedAvgPrice": "0.01368242",
    "openPrice": "0.01298900",
    "highPrice": "0.01418800",
    "lowPrice": "0.01296000",
    "lastPrice": "0.01360400",
    "volume": "587179.23900000",
    "quoteVolume": "8034.03382165",
    "openTime": 1659580020000,
    "closeTime": 1660184865291,
    "firstId": 192977765,       // 第一个交易 ID
    "lastId": 195365758,        // 最后交易 ID
    "count": 2387994            // 成交笔数
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 2
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
    "quoteVolume": "8034.03382165",
    "openTime": 1659580020000,
    "closeTime": 1660184865291,
    "firstId": 192977765,       // 第一个交易 ID
    "lastId": 195365758,        // 最后交易 ID
    "count": 2387994            // 成交笔数
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 2
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
      "priceChange": "0.00061500",
      "priceChangePercent": "4.735",
      "weightedAvgPrice": "0.01368242",
      "openPrice": "0.01298900",
      "highPrice": "0.01418800",
      "lowPrice": "0.01296000",
      "lastPrice": "0.01360400",
      "volume": "587169.48600000",
      "quoteVolume": "8033.90114517",
      "openTime": 1659580020000,
      "closeTime": 1660184820927,
      "firstId": 192977765,
      "lastId": 195365700,
      "count": 2387936
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
      "limit": 1200,
      "count": 4
    }
  ]
}
```

### 最新价格

```javascript
{
  "id": "043a7cf2-bde3-4888-9604-c8ac41fcba4d",
  "method": "ticker.price",
  "params": {
    "symbol": "BNBBTC"
  }
}
```

获取交易对最新价格

如果需要访问实时价格更新，请考虑使用 WebSocket Streams:

* [`<symbol>@aggTrade`](web-socket-streams_CN.md#归集交易)
* [`<symbol>@trade`](web-socket-streams_CN.md#逐笔交易)

**权重:**
根据交易对的数量进行调整：

| 参数 | 重量 |
| --------- |:------:|
| `symbol`  |      1 |
| `symbols` |      2 |
| none      |      2 |

**参数:**

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td rowspan="2" align="center">NO</td>
        <td>获取单个交易对的 price</td>
    </tr>
    <tr>
        <td><code>symbols</code></td>
        <td>ARRAY of STRING</td>
        <td>获取多个交易对的 price </td>
    </tr>
</tbody>
</table>

备注：

* `symbol` 和 `symbols` 不能一起使用。

* 如果未指定交易对，则返回有关当前在交易所交易的所有交易对的信息。

**数据源:**
缓存

**响应:**

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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

如果请求是有多个交易对，响应会是数组类型:

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
      "limit": 1200,
      "count": 2
    }
  ]
}
```

### 当前最优挂单

```javascript
{
  "id": "057deb3a-2990-41d1-b58b-98ea0f09e1b4",
  "method": "ticker.book",
  "params": {
    "symbols": [
      "BNBBTC",
      "BTCUSDT"
    ]
  }
}
```

在订单薄获取当前最优价格和数量。

如果您需要访问实时订单薄 ticker 更新，请考虑使用 WebSocket Streams:

* [`<symbol>@bookTicker`](web-socket-streams_CN.md#按Symbol的最优挂单信息)

**权重:**
根据交易对的数量进行调整：

| 参数 | 重量 |
| --------- |:------:|
| `symbol`  |      1 |
| `symbols` |      2 |
| none      |      2 |

**参数:**

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td rowspan="2" align="center">NO</td>
        <td>获取单个交易对的 ticker</td>
    </tr>
    <tr>
        <td><code>symbols</code></td>
        <td>ARRAY of STRING</td>
        <td>获取多个交易对的 ticker</td>
    </tr>
</tbody>
</table>

备注：

* `symbol` 和 `symbols` 不能一起使用。

* 如果未指定交易对，则返回有关当前在交易所交易的所有交易对的信息。

**数据源:**
缓存

**响应:**

```javascript
{
  "id": "9d32157c-a556-4d27-9866-66760a174b57",
  "status": 200,
  "result": {
    "symbol": "BNBBTC",
    "bidPrice": "0.01358000",
    "bidQty": "12.53400000",
    "askPrice": "0.01358100",
    "askQty": "17.83700000"
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

如果请求是有多个交易对，响应会是数组类型:

```javascript
{
  "id": "057deb3a-2990-41d1-b58b-98ea0f09e1b4",
  "status": 200,
  "result": [
    {
      "symbol": "BNBBTC",
      "bidPrice": "0.01358000",
      "bidQty": "12.53400000",
      "askPrice": "0.01358100",
      "askQty": "17.83700000"
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
      "limit": 1200,
      "count": 2
    }
  ]
}
```

## 交易请求

### 下新的订单 (TRADE)

```javascript
{
  "id": "56374a46-3061-486b-a311-99ee972eb648",
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

**权重:**
1

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`symbol`            | STRING  | YES       |
`side`              | ENUM    | YES       | `BUY` 或者 `SELL`
`type`              | ENUM    | YES       |
`timeInForce`       | ENUM    | NO *      |
`price`             | DECIMAL | NO *      |
`quantity`          | DECIMAL | NO *      |
`quoteOrderQty`     | DECIMAL | NO *      |
`newClientOrderId`  | STRING  | NO        | 客户自定义的唯一订单ID。如果未发送，则自动生成。
`newOrderRespType`  | ENUM    | NO        | <p>可选的响应格式: `ACK`，`RESULT`，`FULL`.</p><p>`MARKET`和`LIMIT`订单默认使用`FULL`，其他订单类型默认使用`ACK`。</p>
`stopPrice`         | DECIMAL | NO *      |
`trailingDelta`     | INT     | NO *      | 请看 [Trailing Stop order FAQ](faqs/trailing-stop-faq-cn.md)
`icebergQty`        | DECIMAL | NO        |
`strategyId`        | INT     | NO        | 标识订单策略中订单的任意ID。
`strategyType`      | INT     | NO        | <p>标识订单策略的任意数值。</p><p>小于`1000000`的值是保留的，不能使用。</p>
`selfTradePreventionMode` |ENUM| NO | 允许的 ENUM 取决于交易对的配置。支持的值有 `EXPIRE_TAKER`，`EXPIRE_MAKER`，`EXPIRE_BOTH`，`NONE`。
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | 值不能大于 `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

根据订单 `type`，<a id="order-type">某些参数(*)</a> 可能成为必需参数：

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

* 使用 `quoteOrderQty` 的 `MARKET` 订单遵循 [`LOT_SIZE`](filters_CN.md#LOT_SIZE_订单尺寸) 过滤规则。

  该订单将执行一个名义价值尽可能接近请求的 `quoteOrderQty` 的数量。

**数据源:**
撮合引擎

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
      "limit": 1200,
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
      "limit": 1200,
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
    "cummulativeQuoteQty": "198.33521500",
    "status": "FILLED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL",
    "workingTime": 1660801715639,
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
    ],
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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

## 订单响应中的特定条件时才会出现的字段

订单响应中的有一些字段仅在满足特定条件时才会出现。这些订单响应可以来自下订单，查询订单或取消订单，并且可以包括 OCO 订单类型。
下面列出了这些字段：

名称           | 描述                                                           |显示的条件                                          | 示例 |
----           | -----                                                         | ---                                               | ---|
`icebergQty`   |  冰山订单的数量。                                                | 只有在请求中发送 `icebergQty` 参数时才会出现。         | `"icebergQty": "0.00000000"` |
`preventedMatchId` | 与 `symbol` 结合使用时，可用于查询因为 STP 导致订单失效的过期订单。| 只有在因为 STP 导致订单失效时可见。                    | `"preventedMatchId": 0` |
`preventedQuantity` | 因为 STP 导致订单失效的数量。                                | 只有在因为 STP 导致订单失效时可见。                    | `"preventedQuantity": "1.200000"` |
`stopPrice`    | 用于设置逻辑订单中的触发价。                                       | `STOP_LOSS`，`TAKE_PROFIT`，`STOP_LOSS_LIMIT` 和 `TAKE_PROFIT_LIMIT` 订单时可见。| `"stopPrice": "23500.00000000"` |
`strategyId`   | 策略单ID; 用以关联此订单对应的交易策略。                            | 如果在请求中添加了参数，则会出现。                      | `"strategyId": 37463720` |
`strategyType` | 策略单类型; 用以显示此订单对应的交易策略。                           | 如果在请求中添加了参数，则会出现。                      | `"strategyType": 1000000` |
`trailingDelta`| 用以定义追踪止盈止损订单被触发的价格差。                             | 出现在追踪止损订单中。                                | `"trailingDelta": 10` |
`trailingTime` | 追踪单被激活和跟踪价格变化的时间。                                  | 出现在追踪止损订单中。                                 | `"trailingTime": -1`|


### 测试下单 (TRADE)

```javascript
{
  "id": "6ffebe91-01d9-43ac-be99-57cf062e0e30",
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
1

**参数:**

与 [`order.place`](##下新的订单-trade) 相同。

**数据源:**
缓存

**响应:**
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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### 查询订单 (USER_DATA)

```javascript
{
  "id": "aa62318a-5a97-4f3b-bdc7-640bbe33b291",
  "method": "order.status",
  "params": {
    "symbol": "BTCUSDT",
    "orderId": 12569099453,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "2c3aab5a078ee4ea465ecd95523b77289f61476c2f238ec10c55ea6cb11a6f35",
    "timestamp": 1660801720951
  }
}
```

查询订单状态。

**权重:**
2

**参数:**

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>orderId</code></td>
        <td>INT</td>
        <td rowspan="2">YES</td>
        <td>按 <code>orderId</code> 查找顺序</td>
    </tr>
    <tr>
        <td><code>origClientOrderId</code></td>
        <td>STRING</td>
        <td>按 <code>clientOrderId</code> 查找顺序</td>
    </tr>
    <tr>
        <td><code>apiKey</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>recvWindow</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>值不能大于 <tt>60000</tt></td>
    </tr>
    <tr>
        <td><code>signature</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>timestamp</code></td>
        <td>INT</td>
        <td>YES</td>
        <td></td>
    </tr>
</tbody>
</table>

备注：

* 如果同时指定了 `orderId` 和 `origClientOrderId` 参数，仅使用 `orderId` 并忽略 `origClientOrderId`。

* 对于某些历史订单，`cummulativeQuoteQty` 响应字段可能为负数，意味着此时数据不可用。

**数据源:**
缓存 => 数据库

**响应:**
```javascript
{
  "id": "aa62318a-5a97-4f3b-bdc7-640bbe33b291",
  "status": 200,
  "result": {
    "symbol": "BTCUSDT",
    "orderId": 12569099453,
    "orderListId": -1,                  // OCO订单的ID，不然就是-1
    "clientOrderId": "4d96324ff9d44481926157",
    "price": "23416.10000000",
    "origQty": "0.00847000",
    "executedQty": "0.00847000",
    "cummulativeQuoteQty": "198.33521500",
    "status": "FILLED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL",
    "stopPrice": "0.00000000",          // 始终存在，如果订单类型不使用 stopPrice，则为零
    "trailingDelta": 10,                // 如果订单设置了 trailingDelta 会出现
    "trailingTime": -1,                 // 如果订单设置了 trailingDelta 会出现
    "icebergQty": "0.00000000",         // 始终存在，非冰山订单为零
    "time": 1660801715639,              // 下单时间
    "updateTime": 1660801717945,        // 最后一次更新订单的时间
    "isWorking": true,
    "workingTime": 1660801715639,  
    "origQuoteOrderQty": "0.00000000"   // 始终存在，如果订单类型不使用 quoteOrderQty，则为零
    "strategyId": 37463720,             // 如果订单设置了 strategyId  会出现
    "strategyType": 1000000,            // 如果订单设置了 strategyType 会出现
    "selfTradePreventionMode": "NONE",
    "preventedMatchId": 0,              // 这仅在订单因 STP 而过期时可见
    "preventedQuantity": "1.200000"     // 这仅在订单因 STP 而过期时可见
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 2
    }
  ]
}
```
**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 "订单响应中的特定条件时才会出现的字段" 部分。

### 撤销订单 (TRADE)

```javascript
{
  "id": "5633b6a2-90a9-4192-83e7-925c90b6a2fd",
  "method": "order.cancel",
  "params": {
    "symbol": "BTCUSDT",
    "origClientOrderId": "4d96324ff9d44481926157",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "33d5b721f278ae17a52f004a82a6f68a70c68e7dd6776ed0be77a455ab855282",
    "timestamp": 1660801715830
  }
}
```

取消有效订单。

**权重:**
1

**参数:**

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>orderId</code></td>
        <td>INT</td>
        <td rowspan="2">YES</td>
        <td>按 <code>orderId</code> 取消订单</td>
    </tr>
    <tr>
        <td><code>origClientOrderId</code></td>
        <td>STRING</td>
        <td>按 <code>clientOrderId</code> 取消订单</td>
    </tr>
    <tr>
        <td><code>newClientOrderId</code></td>
        <td>STRING</td>
        <td>NO</td>
        <td>已取消订单的新 ID。如果未发送，则自动生成</td>
    </tr>
    <tr>
      <td><code>cancelRestrictions</code></td>
      <td>ENUM</td>
      <td>NO</td>
      <td>支持的值: <br><code>ONLY_NEW</code> - 如果订单状态为 <code>NEW</code>，撤销将成功。<br> <code>ONLY_PARTIALLY_FILLED</code> - 如果订单状态为 <code>PARTIALLY_FILLED</code>，撤销将成功。</td>
    </tr>
    <tr>
        <td><code>apiKey</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>recvWindow</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>值不能大于 <tt>60000</tt></td>
    </tr>
    <tr>
        <td><code>signature</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>timestamp</code></td>
        <td>INT</td>
        <td>YES</td>
        <td></td>
    </tr>
</tbody>
</table>

备注：

* 如果同时指定了 `orderId` 和 `origClientOrderId` 参数，仅使用 `orderId` 并忽略 `origClientOrderId`。

* `newClientOrderId` 将替换已取消订单的 `clientOrderId`，为新订单腾出空间。

* 如果您取消属于 OCO 对的订单，则整个 OCO 将被取消。

**数据源:**
撮合引擎

**响应:**

取消单个订单时：

```javascript
{
  "id": "5633b6a2-90a9-4192-83e7-925c90b6a2fd",
  "status": 200,
  "result": {
    "symbol": "BTCUSDT",
    "origClientOrderId": "4d96324ff9d44481926157",  // 被取消的 clientOrderId
    "orderId": 12569099453,
    "orderListId": -1,                              // OCO订单的ID，不然就是 -1
    "clientOrderId": "91fe37ce9e69c90d6358c0",      // 请求的 newClientOrderId
    "price": "23416.10000000",
    "origQty": "0.00847000",
    "executedQty": "0.00001000",
    "cummulativeQuoteQty": "0.23416100",
    "status": "CANCELED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL",
    "stopPrice": "0.00000000",          // 如果订单设置了 stopPrice 会出现
    "trailingDelta": 0,                 // 如果订单设置了 trailingDelta 会出现
    "icebergQty": "0.00000000",         // 如果订单设置了 icebergQty 会出现
    "strategyId": 37463720,             // 如果订单设置了 strategyId 会出现
    "strategyType": 1000000,            // 如果订单设置了 strategyType 会出现
    "selfTradePreventionMode": "NONE"
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 2
    }
  ]
}
```

取消 OCO 时：

```javascript
{
 "id": "16eaf097-bbec-44b9-96ff-e97e6e875870",
  "status": 200,
  "result": {
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
    // OCO leg 状态格式与单个订单相同。
    "orderReports": [
      {
        "symbol": "BTCUSDT",
        "origClientOrderId": "bX5wROblo6YeDwa9iTLeyY",
        "orderId": 12569099453,
        "orderListId": 19431,
        "clientOrderId": "OFFXQtxVFZ6Nbcg4PgE2DA",
        "price": "23450.50000000",
        "origQty": "0.00850000"
        "executedQty": "0.00000000",
        "cummulativeQuoteQty": "0.00000000",
        "status": "CANCELED",
        "timeInForce": "GTC",
        "type": "STOP_LOSS_LIMIT",
        "side": "BUY",
        "stopPrice": "23430.00000000",
        "selfTradePreventionMode": "NONE"
      },
      {
        "symbol": "BTCUSDT",
        "origClientOrderId": "Tnu2IP0J5Y4mxw3IATBfmW",
        "orderId": 12569099454,
        "orderListId": 19431,
        "clientOrderId": "OFFXQtxVFZ6Nbcg4PgE2DA",
        "price": "23400.00000000",
        "origQty": "0.00850000"
        "executedQty": "0.00000000",
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
      "limit": 1200,
      "count": 1
    }
  ]
}
```
**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 "订单响应中的特定条件时才会出现的字段" 部分。

#### 关于 `cancelRestrictions`

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

### 撤消挂单再下单 (TRADE)

```javascript
{
  "id": "99de1036-b5e2-4e0f-9b5c-13d751c93a1a",
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
    "signature": "7028fdc187868754d25e42c37ccfa5ba2bab1d180ad55d4c3a7e2de643943dc5",
    "timestamp": 1660813156900
  }
}
```

撤消挂单并在同个交易对上重新下单。

**权重:**
1

**参数:**

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>cancelReplaceMode</code></td>
        <td>ENUM</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>cancelOrderId</code></td>
        <td>INT</td>
        <td rowspan="2">YES</td>
        <td>按 <code>orderId</code> 取消订单</td>
    </tr>
    <tr>
        <td><code>cancelOrigClientOrderId</code></td>
        <td>STRING</td>
        <td>按 <code>clientOrderId</code> 取消订单</td>
    </tr>
    <tr>
        <td><code>cancelNewClientOrderId</code></td>
        <td>STRING</td>
        <td>NO</td>
        <td>已取消订单的新 ID。如果未发送，则自动生成</td>
    </tr>
    <tr>
        <td><code>side</code></td>
        <td>ENUM</td>
        <td>YES</td>
        <td><code>BUY</code> 或者 <code>SELL</code></td>
    </tr>
    <tr>
        <td><code>type</code></td>
        <td>ENUM</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>timeInForce</code></td>
        <td>ENUM</td>
        <td>NO *</td>
        <td></td>
    </tr>
    <tr>
        <td><code>price</code></td>
        <td>DECIMAL</td>
        <td>NO *</td>
        <td></td>
    </tr>
    <tr>
        <td><code>quantity</code></td>
        <td>DECIMAL</td>
        <td>NO *</td>
        <td></td>
    </tr>
    <tr>
        <td><code>quoteOrderQty</code></td>
        <td>DECIMAL</td>
        <td>NO *</td>
        <td></td>
    </tr>
    <tr>
        <td><code>newClientOrderId</code></td>
        <td>STRING</td>
        <td>NO</td>
        <td>客户自定义的唯一订单ID。如果未发送，则自动生成</td>
    </tr>
    <tr>
        <td><code>newOrderRespType</code></td>
        <td>ENUM</td>
        <td>NO</td>
        <td>
            <p>选择响应格式： <code>ACK</code>, <code>RESULT</code>, <code>FULL</code>。</p>
            <p>
                <code>MARKET</code> 和 <code>LIMIT</code> 订单默认推送 <code>FULL</code> 响应，
                 其他订单类型默认为 <code>ACK</code>。
            </p>
        </td>
    </tr>
    <tr>
        <td><code>stopPrice</code></td>
        <td>DECIMAL</td>
        <td>NO *</td>
        <td></td>
    </tr>
    <tr>
        <td><code>trailingDelta</code></td>
        <td>DECIMAL</td>
        <td>NO *</td>
        <td>请看 <a href="faqs/trailing-stop-faq-cn.md">Trailing Stop order FAQ</a></td>
    </tr>
    <tr>
        <td><code>icebergQty</code></td>
        <td>DECIMAL</td>
        <td>NO</td>
        <td></td>
    </tr>
    <tr>
        <td><code>strategyId</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>标识订单策略中订单的任意ID。</td>
    </tr>
    <tr>
        <td><code>strategyType</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>
            <p>标识订单策略的任意数值。</p>
            <p>小于 <tt>1000000</tt> 的值被保留，不能使用。</p>
        </td>
    </tr>
    <tr>
        <td><code>selfTradePreventionMode</code></td>
        <td>ENUM</td>
        <td>NO</td>
        <td>
            <p>允许的 ENUM 取决于交易对的配置。</p>
            <p>支持的值有 <tt>EXPIRE_TAKER</tt>, <tt>EXPIRE_MAKER</tt>, <tt>EXPIRE_BOTH</tt>, <tt>NONE</tt>.</p>
        </td>
    </tr>
    <tr>
        <td><code>cancelRestrictions</code></td>
        <td>ENUM</td>
        <td>NO</td>
        <td>支持的值: <br><code>ONLY_NEW</code> - 如果订单状态为 <code>NEW</code>，撤销将成功。<br> <code>ONLY_PARTIALLY_FILLED</code> - 如果订单状态为 <code>PARTIALLY_FILLED</code>，撤销将成功。</td>
    </tr>
    <tr>
        <td><code>apiKey</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>recvWindow</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>值不能大于 <tt>60000</tt></td>
    </tr>
    <tr>
        <td><code>signature</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>timestamp</code></td>
        <td>INT</td>
        <td>YES</td>
        <td></td>
    </tr>
</tbody>
</table>

类似于 [`order.place`](#下新的订单-trade) 请求，额外的强制参数 (*) 由新订单的 [`type`](#order-type) 确定。

可用的 `cancelReplaceMode` 选项：

* `STOP_ON_FAILURE` – 如果撤销订单请求失败，将不会尝试下新订单
* `ALLOW_FAILURE` – 即使撤销订单请求失败，也会尝试下新订单

<table>
<thead>
    <tr>
        <th>Request</th>
        <th colspan=3>Response</th>
    </tr>
    <tr>
        <th><code>cancelReplaceMode</code></th>
        <th><code>cancelResult</code></th>
        <th><code>newOrderResult</code></th>
        <th><code>status</code></th>
    </tr>
</thead>
<tbody>
    <tr>
        <td rowspan="3"><code>STOP_ON_FAILURE</code></td>
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
        <td rowspan="4"><code>ALLOW_FAILURE</code></td>
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
</tbody>
</table>

备注：

* 如果同时指定了 `cancelOrderId` 和 `cancelOrigClientOrderId` 参数，仅使用 `cancelOrderId` 并忽略 `cancelOrigClientOrderId`。

* `cancelNewClientOrderId` 将替换已撤销订单的 `clientOrderId`，为新订单腾出空间。

* `newClientOrderId` 指定下单的 `clientOrderId` 值。

  仅当前一个订单已成交或过期时，才会接受具有相同 `clientOrderId` 的新订单。

  新订单可以重用已取消订单的旧 `clientOrderId`。

* 此 cancel-replace 操作**不是事务性的**。

  如果一个操作成功但另一个操作失败，则仍然执行成功的操作。

  例如，在 `STOP_ON_FAILURE` 模式下，如果下新订单达失败，旧订单仍然被撤销。

* 过滤器和订单次数限制会在撤销和下订单之前评估。

* 如果未尝试下新订单，订单次数仍会增加。

* 与 [`order.cancel`](#撤销订单-TRADE) 一样，如果您撤销 OCO 的某个边，则整个 OCO 将被撤销。

**数据源:**
撮合引擎

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
      "origClientOrderId": "4d96324ff9d44481926157",  // 请求的 cancelOrigClientOrderId
      "orderId": 125690984230,
      "orderListId": -1,
      "clientOrderId": "91fe37ce9e69c90d6358c0",      // 请求的 cancelNewClientOrderId
      "price": "23450.00000000",
      "origQty": "0.00847000",
      "executedQty": "0.00001000",
      "cummulativeQuoteQty": "0.23450000",
      "status": "CANCELED",
      "timeInForce": "GTC",
      "type": "LIMIT",
      "side": "SELL",
      "selfTradePreventionMode": "NONE"
    },
    // 格式与 "order.place" 格式相同, 受 "newOrderRespType" 影响。
    // 某些字段是可选的，仅在订单中有设置它们时才包括。
    "newOrderResponse": {
      "symbol": "BTCUSDT",
      "orderId": 12569099453,
      "orderListId": -1,
      "clientOrderId": "bX5wROblo6YeDwa9iTLeyY",      // 请求的 newClientOrderId
      "transactTime": 1660813156959,
      "price": "23416.10000000",
      "origQty": "0.00847000",
      "executedQty": "0.00000000",
      "cummulativeQuoteQty": "0.00000000",
      "status": "NEW",
      "timeInForce": "GTC",
      "type": "LIMIT",
      "side": "SELL",
      "selfTradePreventionMode": "NONE"
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
      "limit": 1200,
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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

如果 cancel-replace 模式允许失败并且其中一个操作失败，响应会是 `"status": 409` 和 `"data"` 字段会制定哪个操作成功，哪个失败，以及原因：

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
        "origClientOrderId": "4d96324ff9d44481926157",
        "orderId": 125690984230,
        "orderListId": -1,
        "clientOrderId": "91fe37ce9e69c90d6358c0",
        "price": "23450.00000000",
        "origQty": "0.00847000",
        "executedQty": "0.00001000",
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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

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
        "cummulativeQuoteQty": "0.00000000",
        "status": "NEW",
        "timeInForce": "GTC",
        "type": "LIMIT",
        "side": "SELL",
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
      "limit": 1200,
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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 "订单响应中的特定条件时才会出现的字段" 部分。

### 当前挂单 (USER_DATA)

```javascript
{
  "id": "55f07876-4f6f-4c47-87dc-43e5fff3f2e7",
  "method": "openOrders.status",
  "params": {
    "symbol": "BTCUSDT",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "d632b3fdb8a81dd44f82c7c901833309dd714fe508772a89b0a35b0ee0c48b89",
    "timestamp": 1660813156812
  }
}
```

查询所有挂订单的执行状态。

如果您需要持续监控订单状态更新，请考虑使用 WebSocket Streams：

* [`userDataStream.start`](#Websocket-账户信息) 请求
* [`executionReport`](./user-data-stream_CN.md#订单更新) 更新

**权重:**
根据交易对的数量进行调整：

| 参数 | 重量 |
| --------- | ------ |
| `symbol`  |      3 |
| none      |     40 |

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`symbol`            | STRING  | NO        | 如果省略，则返回所有交易对的挂单
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | 值不能大于 `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

**数据源:**
缓存 => 数据库

**响应:**

挂单的状态报告与 [`order.status`](#查询订单-USER_DATA) 相同。

请注意，某些字段是可选的，仅在订单中有设置它们时才包括。

挂订单始终作为平面列表返回。
如果所有交易对被请求，请使用 `symbol` 字段来告知订单属于哪个交易对。

```javascript
{
  "id": "55f07876-4f6f-4c47-87dc-43e5fff3f2e7",
  "status": 200,
  "result": [
    {
      "symbol": "BTCUSDT",
      "orderId": 12569099453,
      "orderListId": -1,
      "clientOrderId": "4d96324ff9d44481926157",
      "price": "23416.10000000",
      "origQty": "0.00847000",
      "executedQty": "0.00720000",
      "cummulativeQuoteQty": "172.43931000",
      "status": "PARTIALLY_FILLED",
      "timeInForce": "GTC",
      "type": "LIMIT",
      "side": "SELL",
      "stopPrice": "0.00000000",
      "icebergQty": "0.00000000",
      "time": 1660801715639,
      "updateTime": 1660801717945,
      "isWorking": true,
      "workingTime": 1660801715639,
      "origQuoteOrderQty": "0.00000000",
      "selfTradePreventionMode": "NONE"
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 3
    }
  ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 "订单响应中的特定条件时才会出现的字段" 部分。

### 撤销单一交易对的所有挂单 (TRADE)

```javascript
{
  "id": "778f938f-9041-4b88-9914-efbf64eeacc8",
  "method": "openOrders.cancelAll"
  "params": {
    "symbol": "BTCUSDT",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "773f01b6e3c2c9e0c1d217bc043ce383c1ddd6f0e25f8d6070f2b66a6ceaf3a5",
    "timestamp": 1660805557200
  }
}
```

撤销单一交易对的所有挂单,包括 OCO 订单。

**权重:**
1

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`symbol`            | STRING  | YES       |
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | 值不能大于 `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

**数据源:**
撮合引擎

**响应:**

订单和 OCO 的撤销报告的格式与 [`order.cancel`](#撤销订单-TRADE) 中的格式相同。

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
      "price": "23416.10000000",
      "origQty": "0.00847000",
      "executedQty": "0.00001000",
      "cummulativeQuoteQty": "0.23416100",
      "status": "CANCELED",
      "timeInForce": "GTC",
      "type": "LIMIT",
      "side": "SELL",
      "stopPrice": "0.00000000",
      "trailingDelta": 0,
      "icebergQty": "0.00000000",
      "strategyId": 37463720,
      "strategyType": 1000000,
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
          "price": "23450.50000000",
          "origQty": "0.00850000",
          "executedQty": "0.00000000",
          "cummulativeQuoteQty": "0.00000000",
          "status": "CANCELED",
          "timeInForce": "GTC",
          "type": "STOP_LOSS_LIMIT",
          "side": "BUY",
          "stopPrice": "23430.00000000",
          "selfTradePreventionMode": "NONE"
        },
        {
          "symbol": "BTCUSDT",
          "origClientOrderId": "Tnu2IP0J5Y4mxw3IATBfmW",
          "orderId": 12569099454,
          "orderListId": 19431,
          "clientOrderId": "OFFXQtxVFZ6Nbcg4PgE2DA",
          "price": "23400.00000000",
          "origQty": "0.00850000",
          "executedQty": "0.00000000",
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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 "订单响应中的特定条件时才会出现的字段" 部分。

### OCO下单 (TRADE)

```javascript
{
  "id": "56374a46-3061-486b-a311-99ee972eb648",
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
    "signature": "6689c2a36a639ff3915c2904871709990ab65f3c7a9ff13857558fd350315c35",
    "timestamp": 1660801713767
  }
}
```

发送新的OCO(one-cancels-the-other) 订单:
`LIMIT_MAKER` 订单 + `STOP_LOSS`/`STOP_LOSS_LIMIT` 订单(称呼为 *legs*), 其中一个订单的激活会立即取消另一个订单。

**权重:**
1

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`symbol`            | STRING  | YES       |
`side`              | ENUM    | YES       | `BUY` 或者 `SELL`
`price`             | DECIMAL | YES       | Limit 订单的价格
`quantity`          | DECIMAL | YES       |
`listClientOrderId` | STRING  | NO        | OCO 挂单的客户自定义的唯一订单ID。如果未发送，则自动生成
`limitClientOrderId`| STRING  | NO        | Limit 挂单的客户自定义的唯一订单ID。如果未发送，则自动生成
`limitIcebergQty`   | DECIMAL | NO        |
`limitStrategyId`   | INT     | NO        | 标识订单策略中的 limit 订单的任意ID。
`limitStrategyType` | INT     | NO        | <p>标识 limit 订单策略的任意数值</p><p>小于`1000000`的值是保留的，不能使用。</p>
`stopPrice`         | DECIMAL | YES *     | 必须指定 `stopPrice` 或 `trailingDelta`，或两者都指定
`trailingDelta`     | INT     | YES *     | 请看 [追踪止盈止损(Trailing Stop)订单常见问题](faqs/trailing-stop-faq-cn.md)
`stopClientOrderId` | STRING  | NO        | Stop 订单的客户自定义的唯一订单ID。如果未发送，则自动生成
`stopLimitPrice`    | DECIMAL | NO *      |
`stopLimitTimeInForce` | ENUM | NO *      | 有关可用选项，请看 [`order.place`](#timeInForce)
`stopIcebergQty`    | DECIMAL | NO *      |
`stopStrategyId`    | INT     | NO        | 标识订单策略中的 stop 订单的任意ID。
`stopStrategyType`  | INT     | NO        | <p>标识 stop 订单策略的任意数值。</p><p>小于`1000000`的值是保留的，不能使用。</p>
`newOrderRespType`  | ENUM    | NO        | 可选的响应格式: `ACK`，`RESULT`，`FULL` (默认)
`selfTradePreventionMode` |ENUM| NO | 允许的 ENUM 取决于交易对的配置。支持的值有 `EXPIRE_TAKER`，`EXPIRE_MAKER`，`EXPIRE_BOTH`，`NONE`。
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | 值不能大于 `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

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

* 在订单率限制中，OCO 计为2个订单。

**数据源:**
撮合引擎

**响应:**

使用 `newOrderRespType` 参数选择 `orderReports` 的响应格式。
以下示例适用于 `RESULT` 响应类型。
有关更多示例，请参阅 [`order.place`](#下新的订单-trade)。

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
        "cummulativeQuoteQty": "0.00000000",
        "status": "NEW",
        "timeInForce": "GTC",
        "type": "STOP_LOSS_LIMIT",
        "side": "SELL",
        "stopPrice": "23405.00000000",
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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### 查询 OCO (USER_DATA)

```javascript
{
  "id": "b53fd5ff-82c7-4a04-bd64-5f9dc42c2100",
  "method": "orderList.status",
  "params": {
    "origClientOrderId": "08985fedd9ea2cf6b28996"
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "d12f4e8892d46c0ddfbd43d556ff6d818581b3be22a02810c2c20cb719aed6a4",
    "timestamp": 1660801713965
  }
}
```

检查 OCO 的执行状态。

对于单个订单的执行状态，使用 [`order.status`](#查询订单-USER_DATA)。

**权重:**
2

**Parameters**:

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
        <td><code>origClientOrderId</code></td>
        <td>STRING</td>
        <td rowspan="2">YES</td>
        <td>通过 <code>listClientOrderId</code> 获取 OCO </td>
    </tr>
    <tr>
        <td><code>orderListId</code></td>
        <td>INT</td>
        <td>通过 <code>orderListId</code> 获取 OCO</td>
    </tr>
    <tr>
        <td><code>apiKey</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>recvWindow</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>值不能大于 <tt>60000</tt></td>
    </tr>
    <tr>
        <td><code>signature</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>timestamp</code></td>
        <td>INT</td>
        <td>YES</td>
        <td></td>
    </tr>
</tbody>
</table>

备注：

* `origClientOrderId` 指的是 OCO 本身的 `listClientOrderId`。

* 如果同时指定了 `origClientOrderId` 和 `orderListId` 参数，仅使用 `origClientOrderId` 并忽略 `orderListId`。

**数据源:**
数据库

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
      "limit": 1200,
      "count": 2
    }
  ]
}
```

### 撤销 OCO 订单(TRADE)

```javascript
{
  "id": "c5899911-d3f4-47ae-8835-97da553d27d0",
  "method": "orderList.cancel",
  "params": {
    "symbol": "BTCUSDT",
    "orderListId": 1274512,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "4973f4b2fee30bf6d45e4a973e941cc60fdd53c8dd5a25edeac96f5733c0ccee",
    "timestamp": 1660801720210
  }
}
```

取消整个订单列表。

**Weight**:
1

**参数:**

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>orderListId</code></td>
        <td>INT</td>
        <td rowspan="2">YES</td>
        <td>通过 <code>orderListId</code> 撤销 OCO</td>
    </tr>
    <tr>
        <td><code>listClientOrderId</code></td>
        <td>STRING</td>
        <td>通过 <code>listClientId</code> 撤销 OCO</td>
    </tr>
    <tr>
        <td><code>newClientOrderId</code></td>
        <td>STRING</td>
        <td>NO</td>
        <td>已取消 OCO 的新 ID。如果未发送，则自动生成</td>
    </tr>
    <tr>
        <td><code>apiKey</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>recvWindow</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>值不能大于 <tt>60000</tt></td>
    </tr>
    <tr>
        <td><code>signature</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>timestamp</code></td>
        <td>INT</td>
        <td>YES</td>
        <td></td>
    </tr>
</tbody>
</table>

备注：

* 如果同时指定了 `orderListId` 和 `listClientOrderId` 参数，仅使用 `orderListId` 并忽略 `listClientOrderId`。

* 使用 [`order.cancel`](#撤销订单-TRADE) 取消单个 leg 也将会取消整个 OCO。

**数据源:**
撮合引擎

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
        "cummulativeQuoteQty": "0.00000000",
        "status": "CANCELED",
        "timeInForce": "GTC",
        "type": "STOP_LOSS_LIMIT",
        "side": "SELL",
        "stopPrice": "23405.00000000",
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
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### 查询 OCO 挂单 (USER_DATA)

```javascript
{
  "id": "3a4437e2-41a3-4c19-897c-9cadc5dce8b6",
  "method": "openOrderLists.status",
  "params": {
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "1bea8b157dd78c3da30359bddcd999e4049749fe50b828e620e12f64e8b433c9",
    "timestamp": 1660801713831
  }
}
```

查询所有 OCO 挂单的执行状态。

如果您需要持续监控订单状态更新，请考虑使用 WebSocket Streams：

* [`userDataStream.start`](#Websocket-账户信息) 请求
* [`executionReport`](./user-data-stream_CN.md#订单更新) 更新

**Weight**:
3

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | 值不能大于 `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

**数据源:**
数据库

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
      "limit": 1200,
      "count": 3
    }
  ]
}
```

## 账户请求

### 账户信息 (USER_DATA)

```javascript
{
  "id": "605a6d20-6588-4cb9-afa0-b0ab087507ba",
  "method": "account.status",
  "params": {
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "83303b4a136ac1371795f465808367242685a9e3a42b22edb4d977d0696eb45c",
    "timestamp": 1660801839480
  }
}
```

获取当前账户信息。

**权重:**
10

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | 值不能大于 `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

**数据源:**
缓存 => 数据库

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
    ]
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 10
    }
  ]
}
```

### 账户订单率限制 (USER_DATA)

```javascript
{
  "id": "d3783d8d-f8d1-4d2c-b8a0-b7596af5a664",
  "method": "account.rateLimits.orders",
  "params": {
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "76289424d6e288f4dc47d167ac824e859dabf78736f4348abbbac848d719eb94",
    "timestamp": 1660801839500
  }
}
```

查询当前的订单率限制。

**权重:**
20

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | 值不能大于 `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

**数据源:**
缓存

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
      "limit": 1200,
      "count": 20
    }
  ]
}
```

### 账户订单历史 (USER_DATA)

```javascript
{
  "id": "734235c2-13d2-4574-be68-723e818c08f3",
  "method": "allOrders",
  "params": {
    "symbol": "BTCUSDT",
    "startTime": 1660780800000,
    "endTime": 1660867200000,
    "limit": 5,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "f50a972ba7fad92842187643f6b930802d4e20bce1ba1e788e856e811577bd42",
    "timestamp": 1661955123341
  }
}
```

获取所有账户订单； 有效，已取消或已完成。按时间范围过滤。

**权重:**
10

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`symbol`            | STRING  | YES       |
`orderId`           | INT     | NO        | 起始订单ID
`startTime`         | INT     | NO        |
`endTime`           | INT     | NO        |
`limit`             | INT     | NO        | 默认 500; 最大值 1000
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | 值不能大于 `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

备注：

* 如果指定了 `startTime` 和/或 `endTime`，则忽略 `orderId`。

  订单是按照最后一次更新的执行状态的`time`过滤的。

* 如果指定了 `orderId`，返回的订单将是订单ID >= `orderId`

* 如果不指定条件，则返回最近的订单。

* 对于某些历史订单，`cummulativeQuoteQty` 响应字段可能为负数，代表着此时数据还不可用。

**数据源:**
数据库

**响应:**

订单状态报告与 [`order.status`](#查询订单-USER_DATA) 相同。

请注意，某些字段是可选的，仅在订单中有设置它们时才包括。

```javascript
{
  "id": "734235c2-13d2-4574-be68-723e818c08f3",
  "status": 200,
  "result": [
    {
      "symbol": "BTCUSDT",
      "orderId": 12569099453,
      "orderListId": -1,
      "clientOrderId": "4d96324ff9d44481926157",
      "price": "23416.10000000",
      "origQty": "0.00847000",
      "executedQty": "0.00847000",
      "cummulativeQuoteQty": "198.33521500",
      "status": "FILLED",
      "timeInForce": "GTC",
      "type": "LIMIT",
      "side": "SELL",
      "stopPrice": "0.00000000",
      "icebergQty": "0.00000000",
      "time": 1660801715639,
      "updateTime": 1660801717945,
      "isWorking": true,
      "origQuoteOrderQty": "0.00000000",
      "selfTradePreventionMode": "NONE",
      "preventedMatchId": 0,              // 这仅在订单因 STP 而过期时可见
      "preventedQuantity": "1.200000"     // 这仅在订单因 STP 而过期时可见
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 10
    }
  ]
}
```

### 账户 OCO 订单历史 (USER_DATA)

```javascript
{
  "id": "8617b7b3-1b3d-4dec-94cd-eefd929b8ceb",
  "method": "allOrderLists",
  "params": {
    "startTime": 1660780800000,
    "endTime": 1660867200000,
    "limit": 5,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "c8e1484db4a4a02d0e84dfa627eb9b8298f07ebf12fcc4eaf86e4a565b2712c2",
    "timestamp": 1661955123341
  }
}
```

查询所有 OCO 的信息，按时间范围过滤。

**权重:**
10

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`fromId`            | INT     | NO        | 起始的 Order list ID
`startTime`         | INT     | NO        |
`endTime`           | INT     | NO        |
`limit`             | INT     | NO        | 默认 500; 最大值 1000
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | 值不能大于 `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

备注：

* 如果指定了 `startTime` 和/或 `endTime`，则忽略 `fromId`。

  OCO 订单是按照最后一次更新的 OCO 执行状态的 `transactionTime` 过滤的。

* 如果指定了 `fromId`，返回的 OCO 将是 order list ID >= `fromId`。

* 如果不指定条件，则返回最近的 OCO 订单。

**数据源:**
数据库

**响应:**

OCO 的状态报告与 [`orderList.status`](#查询-OCO-user_data) 相同。

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
      "limit": 1200,
      "count": 10
    }
  ]
}
```

### 账户成交历史 (USER_DATA)

```javascript
{
  "id": "f4ce6a53-a29d-4f70-823b-4ab59391d6e8",
  "method": "myTrades",
  "params": {
    "symbol": "BTCUSDT",
    "startTime": 1660780800000,
    "endTime": 1660867200000,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "c5a5ffb79fd4f2e10a92f895d488943a57954edf5933bde3338dfb6ea6d6eefc",
    "timestamp": 1661955125250
  }
}
```

获取账户指定交易对的成交历史，按时间范围过滤。

**权重:**
10

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`symbol`            | STRING  | YES       |
`orderId`           | INT     | NO        |
`startTime`         | INT     | NO        |
`endTime`           | INT     | NO        |
`fromId`            | INT     | NO        | 起始交易 ID
`limit`             | INT     | NO        | 默认 500; 最大值 1000
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | 值不能大于 `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

备注：

* 如果指定了 `fromId`，则返回的交易将是 交易ID >= `fromId`。

* 如果指定了 `startTime` 和/或 `endTime`，则交易按执行时间（`time`）过滤。

  `fromId` 不能与 `startTime` 和 `endTime` 一起使用。

* 如果指定了 `orderId`，则只返回与该订单相关的交易。

  `startTime` 和 `endTime` 不能与 `orderId` 一起使用。

* 如果不指定条件，则返回最近的交易。

**数据源:**
缓存 => 数据库

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
      "limit": 1200,
      "count": 10
    }
  ]
}
```

###  账户的 Prevented Matches (USER_DATA)

```javascript
{
  "id": "g4ce6a53-a39d-4f71-823b-4ab5r391d6y8",
  "method": "myPreventedMatches",
  "params": {
    "symbol": "BTCUSDT",
    "orderId": 35,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "c5a5ffb79fd4f2e10a92f895d488943a57954edf5933bde3338dfb6ea6d6eefc",
    "timestamp": 1673923281052
  }
}
```

获取因 STP 而过期的订单列表。

这些是支持的组合：

* `symbol` + `preventedMatchId`
* `symbol` + `orderId`
* `symbol` + `orderId` + `fromPreventedMatchId` (`limit`  默认为 500)
* `symbol` + `orderId` + `fromPreventedMatchId` + `limit` 

**参数:**

名称                 | 类型   | 是否必需	     | 描述
------------        | ----   | ------------ | ------------
symbol              | STRING | YES          |
preventedMatchId    |LONG    | NO           | 
orderId             |LONG    | NO           |
fromPreventedMatchId|LONG    | NO           |
limit               |INT     | NO           | 默认：`500`；最大：`1000`
recvWindow          | LONG   | NO           | 赋值不得大于 `60000`
timestamp           | LONG   | YES          |

**权重**

情况                             | 权重
--------------------------------| -----
If `symbol` is invalid          | 1
Querying by `preventedMatchId`  | 1
Querying by `orderId`           | 10 

**数据源:**

数据库

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
      "limit": 1200,
      "count": 10
    }
  ]
}
```

## Websocket 账户信息

以下请求管理 [Websocket 帐户信息](user-data-stream_CN.md) 订阅。

**注意：** 账户信息只能在连接到用户数据流服务器的连接上获取, 其服务器URL是 `wss://stream.binance.com:443`.

### 开始用户数据流 (USER_STREAM)

```javascript
{
  "id": "d3df8a61-98ea-4fe0-8f4e-0fcea5d418b0",
  "method": "userDataStream.start",
  "params": {
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A"
  }
}
```

开始新的用户数据流

**注意：** 
数据流将在 60 分钟后关闭，除非定期发送 [`userDataStream.ping`](#ping-user-data-stream-user_stream) 请求。

**权重:**
1

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`apiKey`            | STRING  | YES       |


**数据源:** 
缓存

**响应:**

之后在 WebSocket Stream 上订阅收到的 listen key。


```javascript
{
  "id": "d3df8a61-98ea-4fe0-8f4e-0fcea5d418b0",
  "status": 200,
  "result": {
    "listenKey": "xs0mRXdAKlIPDRFrlPcw0qI41Eh3ixNntmymGyhrhgqo7L6FuLaWArTD7RLP"
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### Ping 账户数据流 (USER_STREAM)

```javascript
{
  "id": "815d5fce-0880-4287-a567-80badf004c74",
  "method": "userDataStream.ping",
  "params": {
    "listenKey": "xs0mRXdAKlIPDRFrlPcw0qI41Eh3ixNntmymGyhrhgqo7L6FuLaWArTD7RLP",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A"
  }
}
```

即使在监听, 用户数据流也会在60分钟后会自动关闭。
若要保持账户数据流的活动状态，必须使用 `userDataStream.ping` 请求定期发送 ping，建议的是在每30分钟发送一次 ping。

**权重:**
1

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`listenKey`         | STRING  | YES       |
`apiKey`            | STRING  | YES       |

**数据源:**
缓存

**响应:**

```javascript
{
  "id": "815d5fce-0880-4287-a567-80badf004c74",
  "status": 200,
  "response": {},
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### 关闭账户数据流 (USER_STREAM)

```javascript
{
  "id": "819e1b1b-8c06-485b-a13e-131326c69599",
  "method": "userDataStream.stop",
  "params": {
    "listenKey": "xs0mRXdAKlIPDRFrlPcw0qI41Eh3ixNntmymGyhrhgqo7L6FuLaWArTD7RLP",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A"
  }
}
```

强制停止和关闭账户数据流

**权重:**
1

**参数:**

名称                | 类型    | 是否必需 | 描述
------------------- | ------- | --------- | ------------
`listenKey`         | STRING  | YES       |
`apiKey`            | STRING  | YES       |

**数据源:**
缓存

**响应:**
```javascript
{
  "id": "819e1b1b-8c06-485b-a13e-131326c69599",
  "status": 200,
  "response": {},
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```
