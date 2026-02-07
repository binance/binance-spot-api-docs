# SBE 市场数据流

## WSS 基本信息

* 基本访问地址是 **stream-sbe.binance.com** 或 **stream-sbe.binance.com:9443**。
* 要以 JSON 格式检索市场数据，请参阅 [此页面](web-socket-streams_CN.md)。
* 可以在[此处](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/stream_1_0.xml)找到用于对数据流进行解码的 SBE 模式。
* stream 名称中所有交易对均为**小写**。
* 订阅单个streams时，可用的URL格式示例： **/ws/\<streamName\>**。
* 订阅组合streams时，可用的URL格式示例： **/stream?streams=\<streamName1\>/\<streamName2\>/\<streamName3\>**。
* 每个到**stream-sbe.binance.com**的链接有效期不超过24小时，请妥善处理断线重连。
* 所有时间和时间戳相关的字段均以 **微秒** 为单位。
* **需要 API Key 身份验证。**
  * 只允许使用 Ed25519 密钥。
  * 打开连接时，请将您的 API Key 放在 `X-MBX-APIKEY` 标头中。时间戳和签名不是必需的。
  * 无需额外的 API 密钥权限即可访问公开市场数据。交易对白名单也不会影响对 SBE 市场数据流的访问。
  * 但是，如果 API 密钥使用 IP 白名单，则仅允许指定的 IP 地址使用 API 密钥。
* WebSocket 服务器会**每20秒**发送 PING 帧。
  * 如果websocket 服务器没有在一分钟之内收到 PONG 帧响应，连接会被断开。
  * 当客户收到 PING 帧，必须尽快回复 PONG 帧，同时 payload 需要和 PING  帧一致。
  * 服务器允许未经请求的 PONG 帧，但这不会保证连接不断开。**对于这种类型的 PONG 帧，建议设置其 payload 为空。**
* [支持实时订阅和取消订阅](web-socket-streams_CN.md#实时订阅/取消数据流)。
  * 您必须以 JSON 格式发送订阅请求，并且还将以 JSON 格式接收订阅响应。
  * 您可以通过查看 websocket 帧类型来区分订阅响应和市场数据事件：订阅响应始终以 text  帧（包含 JSON）发送，而事件始终以 二进制帧（包含 SBE）发送。
* 如果您的请求包含非 ASCII 字符的交易对名称，那么数据流事件中可能包含以 UTF-8 编码的非 ASCII 字符。

## WebSocket 连接限制

* WebSocket服务器**每秒最多接受5个消息**。
  * 消息包括:
    * PING 帧
    * PONG 帧
    * Text  帧JSON格式的控制请求
  * 由服务器推送的事件不受速率限制。
  * 如果用户发送的消息数超过限制，连接会被断开连接。反复被断开连接的IP有可能被服务器屏蔽。
* 单个连接最多可以订阅1024个 Streams。
* 每个IP地址的请求限制为 **每5分钟最多可以发送300次连接请求**。

## 可供用户使用的 Stream

### 逐笔交易

实时推送的原始交易信息

**SBE 消息名称:** `TradesStreamEvent`

**Stream 名称**: \<symbol\>@trade

**更新速度:** 实时

### 最优挂单信息

当订单簿发生变化时，会实时推送最优买入价和卖出价和数量。

**SBE 消息名称:** `BestBidAskStreamEvent`

**Stream 名称**: \<symbol\>@bestBidAsk

**更新速度**: 实时

<a id="auto-culling"></a>
SBE 最优挂单信息使用 **自动剔除（auto-culling）**：当系统负载较高时，可能会丢弃过时的事件，而不是将所有事件排队并延迟发送。

例如，如果在时间 T2 生成了一个最优买/卖报价事件，而此时仍有一个未发送的事件排队在时间 T1（且 T1 < T2），则会丢弃时间 T1 的事件，系统只会发送时间 T2 的事件。此操作是基于每个交易对分别进行的。

### 增量深度信息stream

定期推送订单簿的增量更新。使用此流来维护本地订单簿。

[如何管理本地订单簿。](web-socket-streams_CN.md#如何正确在本地维护一个order-book副本)

**SBE 消息名称:** `DepthDiffStreamEvent`

**Stream 名称**: \<symbol\>@depth

**更新速度:** 50ms

### 有限档深度信息

订单簿前 20 档的快照，定期推送。

**SBE 消息名称:** `DepthSnapshotStreamEvent`

**Stream 名称**: \<symbol\>@depth20

**更新速度:** 50ms
