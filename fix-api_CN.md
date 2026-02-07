# FIX API

> [!NOTE]
> 此 API 只能用于现货 （`SPOT`） 交易所。

<a id="general-api-information"></a>
## 一般 API 信息

* FIX 连接需要 TLS 加密。请使用本地 TCP+TLS 连接或设置本地代理如 [stunnel](https://www.stunnel.org/) 来处理 TLS 加密。
* API 处理请求的超时时间为 10 秒。如果撮合引擎的响应时间超过此时间，API 将返回 “Timeout waiting for response from backend server. Send status unknown; execution status unknown.”。[(-1007 超时)](errors_CN.md#-1007-timeout)
  * 这并不总是意味着该请求在撮合引擎中失败。
  * 如果请求状态未显示在 [WebSocket 账户接口](user-data-stream_CN.md) 中，请执行 API 查询以获取其状态。
* 如果您的请求包含非 ASCII 字符的交易对名称，那么响应中可能包含以 UTF-8 编码的非 ASCII 字符。

**FIX 会话仅支持 Ed25519 密钥。**

关于如何设置 Ed25519 密钥对，请参考 [本教程](https://www.binance.com/zh-CN/support/faq/%E5%A6%82%E4%BD%95%E7%94%9F%E6%88%90ed25519%E5%AF%86%E9%92%A5%E5%AF%B9%E5%9C%A8%E5%B8%81%E5%AE%89%E5%8F%91%E9%80%81api%E8%AF%B7%E6%B1%82-6b9a63f1e3384cf48a2eedb82767a69a)。

### FIX API 订单接入会话

* 端点为：`tcp+tls：//fix-oe.binance.com：9000`
* 支持下单，取消订单和查询当前限制使用情况。
* 支持接收账户的所有 [ExecutionReport`<8>`](#executionreport) 和 [List Status`<N>`](#liststatus)。
* 仅允许带有 `FIX_API` 的 API Key 连接。
* 关于 QuickFIX 模式文件， 请点击 [这里](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml)。

<a id="fix-api-drop-copy-sessions"></a>
### FIX API Drop Copy 会话

* 端点为：`tcp+tls://fix-dc.binance.com:9000`
* 支持接收账户的所有 [ExecutionReport`<8>`](#executionreport) 和 [List Status`<N>`](#liststatus)。
* 仅允许连接带有 `FIX_API` 或 `FIX_API_READ_ONLY` 的 API Key。
* 关于 QuickFIX 模式文件， 请点击 [这里](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml)。
* Drop Copy 会话中的数据存在 1 秒的延迟。

### FIX API Market Data 会话

* 端点为：`tcp+tls：//fix-md.binance.com：9000`
* 支持市场数据流和活动工具查询。
* 不支持下订单或取消订单。
* 仅允许连接带有“FIX_API”或“FIX_API_READ_ONLY”的 API 密钥。

关于 QuickFIX 模式 (Schema) 文件， 请点击 [这里](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml)。

FIX 连接需要 TLS 加密。请使用本地 TCP+TLS 连接或设置本地代理如 [stunnel](https://www.stunnel.org/) 来处理 TLS 加密。

FIX Market Data 的 QuickFIX Schema 可以在 [这里](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml)


### FIX 连接生命周期

* 所有 FIX API 会话将尽最大努力，尽可能长时间地保持开放状态。
* 没有最短连接时间保证，服务器可能随时进入维护状态。
  * 当服务器进入维护状态时，系统将会向客户端**每隔 10 秒发送一条** [News `<B>`](#news) 消息，并**持续 10 分钟**，以提示客户端重新连接。收到此消息后，客户端应建立新会话并关闭旧会话。如果客户端未在规定的时间范围内关闭旧连接，那么服务器会将其注销并关闭会话。
* 连接后，客户端必须发送 Logon `<A>` 请求。有关详情，请参阅 [如何签署登录请求](#signaturecomputation)。
* 客户端应在断开连接之前发送 Logout `<5>` 消息来关闭会话。未能发送注销消息将导致在 2x `HeartInt (108)` 定义的时间间隔内无法将会话的 `SenderCompID (49)` 用于新会话的建立。
* 系统允许在登录过程中协商 `HeartInt (108)` 值。可接受值的范围为 5 到 60 秒。
  * 如果服务器在 `HeartInt (108)` 间隔内没有发送任何消息，将发送 [HeartBeat `<0>`](#heartbeat)。
  * 如果服务器在 `HeartInt (108)` 间隔内没有收到任何消息，将发送 [TestRequest `<1>`](#testrequest)。如果服务器在 `HeartInt (108)` 秒内没有收到来自客户端的包含预期 `TestReqID (112)` 的 HeartBeat `<0>` 消息，服务器将发送 Logout `<5>` 消息并关闭连接。
  * 如果客户端在 `HeartInt (108)` 间隔内未收到任何消息，那么客户端应负责发送 TestRequest `<1>` 消息以确保连接正常。收到这样的 TestRequest `<1>` 后，服务器会发送包含预期 `TestReqID (112)` 的 HeartBeat `<0>` 消息来进行响应。如果客户端在 `HeartInt (108)` 间隔内没有收到服务器的响应，那么客户端应关闭现有的会话和连接，并且建立新的会话和连接。

### API Key 权限

如果您需要使用 FIX API 的订单接入会话，您的 API key 必须配置 `FIX_API` 权限。
如果您需要使用 FIX API 的 Drop Copy 会话，您的 API key 必须配置 `FIX_API_READ_ONLY` 或 `FIX_API` 权限。
要访问 FIX Market Data 会话，您的 API 密钥必须配置为 `FIX_API` 或 `FIX_API_READ_ONLY` 权限

**FIX 会话仅支持 Ed25519 密钥。**

关于如何设置 Ed25519 密钥对，请参考 [本教程](https://www.binance.com/zh-CN/support/faq/%E5%A6%82%E4%BD%95%E7%94%9F%E6%88%90ed25519%E5%AF%86%E9%92%A5%E5%AF%B9%E5%9C%A8%E5%B8%81%E5%AE%89%E5%8F%91%E9%80%81api%E8%AF%B7%E6%B1%82-6b9a63f1e3384cf48a2eedb82767a69a)。

<a id="orderedmode"></a>

### 关于消息处理顺序

初始 [Logon`<A>`](#logon-request) 消息中必需的 `MessageHandling (25035)` 字段控制客户端消息在被撮合引擎处理之前是否可以被重新排序。

| 模式 | 描述 |
|-----------------|--------------------------------------------------------------------------------------------------------|
| `UNORDERED(1)` | 客户端消息可以按任意顺序发送到撮合引擎。|
| `SEQUENTIAL(2)` | 客户端消息始终按 `MsgSeqNum (34)` 顺序发送到撮合引擎。|

在所有模式下，客户端的 `MsgSeqNum (34)` 必须单调递增，即每条后续消息的序列号都比前一条消息正好大 1。

> [!TIP]
> 在有多个消息需要从客户端传输到服务器的情况时， `UNORDERED(1)` 应该会提供更好的性能。

<a id="responsemode"></a>

### 响应模式

默认情况下，所有并发订单录入会话都会接收到账户所有
成功的 [ExecutionReport `<8>` ](#executionreport) 和 [ListStatus `<N>` ](#liststatus) 消息，
包括从其他 FIX 会话和通过非 FIX API 下达的订单。

用户可以在初始消息 [Logon`<A>`](#logon-request) 中使用 `ResponseMode (25036)` 字段来改变这种行为。

- `EVERYTHING(1)`： 默认模式。
- `ONLY_ACKS(2)`： 无论操作成功还是失败，都只接收 ACK 消息。禁用 `ExecutionReport` 推送。

<a id="timingsecurity"></a>

### 时间同步安全

* 所有请求都需要一个 `SendingTime(52)` 字段，该字段应为当前时间戳。
* 另有一个可选字段 `RecvWindow(25000)` ，用以指定请求的有效期（以毫秒为单位）。
  * `RecvWindow(25000)` 扩展为三位小数（例如 6000.346），以便可以指定微秒。
  * 如果未指定 `RecvWindow(25000)`，则仅对 Logon`<A>` 请求默认为 5000 毫秒。对于其他请求，如果未设置，则不会执行 RecvWindow 检查。
  * `RecvWindow(25000)` 的最大有效时间为 60000 毫秒。
* 请求处理逻辑如下：

```javascript
serverTime = getCurrentTime()
if (SendingTime < (serverTime + 1 second) && (serverTime - SendingTime) <= RecvWindow) {
  // 开始处理请求
  serverTime = getCurrentTime()
  if (serverTime - SendingTime) <= RecvWindow {
    // 将请求转发到撮合引擎
  } else {
    // 拒绝请求
  }
  // 结束处理请求
} else {
  // 拒绝请求
}
```

<a id="signaturecomputation"></a>

### 如何签署 Logon<code>&lt;A&gt;</code> 请求

[Logon`<A>`](#logon-main) 消息用于验证您与 FIX API 的连接。
这条消息必须是客户端发送的第一条消息。

* `Username (553)` 字段必须包含 API key。
* `RawData (96)` 字段必须包含使用 API key 生成的有效签名。

签名 payload 是一个文本字符串。该字符串通过按以下所列顺序去连接下列字段来构成， 并且以 SOH 字符作为分隔符：

1. `MsgType (35)`
2. `SenderCompId (49)`
3. `TargetCompId (56)`
4. `MsgSeqNum (34)`
5. `SendingTime (52)`

请使用您的密钥签署 payload。请使用 **base64** 对签名进行编码。
生成的文本字符串是 `RawData (96)` 字段的值。

以下是实现签名算法的 Python 代码示例：

````python
import base64

from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey
from cryptography.hazmat.primitives.serialization import load_pem_private_key

def logon_raw_data(private_key: Ed25519PrivateKey,
                   sender_comp_id: str,
                   target_comp_id: str,
                   msg_seq_num: str,
                   sending_time: str):
    """
    Computes the value of RawData (96) field in Logon<A> message.
    """
    payload = chr(1).join([
        'A',
        sender_comp_id,
        target_comp_id,
        msg_seq_num,
        sending_time,
    ])
    signature = private_key.sign(payload.encode('ASCII'))
    return base64.b64encode(signature).decode('ASCII')


with open('private_key.pem', 'rb') as f:
    private_key = load_pem_private_key(data=f.read(),
                                       password=None)

raw_data = logon_raw_data(private_key,
                          sender_comp_id='5JQmUOsm',
                          target_comp_id='SPOT',
                          msg_seq_num='1',
                          sending_time='20240612-08:52:21.613')
````

以下值可用于验证签名计算实现的正确性：

| 字段             | 取值                   |
|-------------------|-------------------------|
| MsgType (35)      | `A`                     |
| SenderCompID (49) | `EXAMPLE`               |
| TargetCompID (56) | `SPOT`                  |
| MsgSeqNum (34)    | `1`                     |
| SendingTime (52)  | `20240627-11:17:25.223` |

示例计算中使用的 Ed25519 密钥如下所示：

> [!CAUTION]
> 以下密钥仅用于示范说明目的。请务必不要在任何实际应用中使用该密钥，因为它不安全，可能会危及您的加密实现。请在实际使用中生成属于您自己的唯一且安全的密钥。

```
-----BEGIN PRIVATE KEY-----
MC4CAQAwBQYDK2VwBCIEIIJEYWtGBrhACmb9Dvy+qa8WEf0lQOl1s4CLIAB9m89u
-----END PRIVATE KEY-----
```

经过计算生成的签名：
```
4MHXelVVcpkdwuLbl6n73HQUXUf1dse2PCgT1DYqW9w8AVZ1RACFGM+5UdlGPrQHrgtS3CvsRURC1oj73j8gCA==
```

生成的 Logon `<A>` 消息：
```
8=FIX.4.4|9=247|35=A|34=1|49=EXAMPLE|52=20240627-11:17:25.223|56=SPOT|95=88|96=4MHXelVVcpkdwuLbl6n73HQUXUf1dse2PCgT1DYqW9w8AVZ1RACFGM+5UdlGPrQHrgtS3CvsRURC1oj73j8gCA==|98=0|108=30|141=Y|553=sBRXrJx2DsOraMXOaUovEhgVRcjOvCtQwnWj8VxkOh1xqboS02SPGfKi2h8spZJb|25035=2|10=227|
```

## 限制

### 消息限制

* 每个连接都有一个关于 **可以发送到交易所的消息数量** 的限制。
* 消息限制 **不计算从接口发回到客户端的响应消息数量**。
* 违反消息限制会立即导致 [Logout `<5>`](#logout) 并断开连接。
* 要了解当前的限制和使用情况，请发送 [LimitQuery`<XLQ>`](#limitquery) 消息。
  接口将发送 [LimitResponse`<XLR>`](#limitresponse) 消息作为响应，其中包含了有关订单速率限制和消息限制的信息。
* FIX API 订单输入会话的限制为每 10 秒 10,000 条消息。
* FIX Drop Copy 会话的限制为每 60 秒 60 条消息。
* FIX Market Data 会话的限制为每 60 秒 2000 条消息。

<a id="unfilled-order-count"></a>

### 未成交订单计数

* 要了解您在特定时间间隔内下了多少订单，请发送 [LimitQuery`<XLQ>`](#limitquery) 消息。
  系统将发送一条 [LimitResponse`<XLR>`](#limitresponse) 消息作为响应，其中会包含有关未成交订单计数和消息限制的信息。
* **请注意，如果您的订单一直顺利完成交易，您可以在 API 持续下订单**。更多信息，请参见[现货未成交订单计数规则](./faqs/order_count_decrement_CN.md)。
* 如果您超过了未成交的订单计数限制，您的消息将被拒绝，并且信息将用该接口的拒绝消息格式传回给您。
* **未成交订单数量是按照每个账户来统计的**。

<a id="connection-limits"></a>

### 连接限制

* 每个账户都有一个关于 **可以同时建立的 TCP 连接数量** 的限制。
* 当 TCP 连接关闭时，限制值会减少。如果限制值没有立即减少，请等待最长不超过2倍于在 `HeartBtInt (108)` 内所定义的时间。
  比如说，如果 `HeartBtInt` 的值为5， 那么请等待10秒钟。
* 违反限制时， [Reject `<3>`](#reject) 消息会被发送给用户。该消息包含了有关违反连接限制和当前限制的信息。
* FIX 订单接入会话限制：
  * 在 30 秒内 15 次连接尝试的限制
  * 每个账户 最多 10 个并发 TCP 连接的限制
* FIX Drop Copy 会话限制：
  * 在 30 秒内 15 次连接尝试的限制
  * 每个账户 10 个并发 TCP 连接的限制
* FIX Market Data 会话限制:
  * 在 300 秒内 300 次连接尝试的限制
  * 每个账户 100 个并发 TCP 连接的限制
  * 单一连接最多能监听 1000 个数据流

## 错误处理

包含句法错误，缺少必填字段，或引用未知交易对的客户端消息将被服务器拒绝，并返回 [Reject `<3>`](#reject) 消息。

如果有效消息无法被处理并被拒绝，服务器将发送相应的拒绝响应。
请参阅各个消息的相关文档以了解可能会发生的响应。

请参阅响应中的 `Text (58)` 和 `ErrorCode (25016)` 字段以了解拒绝原因。

错误代码列表可以在 [错误代码](errors_CN.md) 页面找到。

## 类型

仅支持可打印的 ASCII 字符和 SOH。

| 类型           | 描述                                                     |
|----------------|-----------------------------------------------------------------|
| `BOOLEAN`      | Enum：`Y` 或 `N`.                                               |
| `CHAR`         | 单个字符。                                               |
| `INT`          | 有符号的 64 位整数。                                          |
| `LENGTH`       | 无符号的 64 位整数。                                        |
| `NUMINGROUP`   | 无符号的 64 位整数。                                        |
| `PRICE`        | 定点数。精度取决于 symbol 定义。 |
| `QTY`          | 定点数。精度取决于 symbol 定义。 |
| `SEQNUM`       | 无符号的 32 位整数。达到最大值 4,294,967,295 后会归 0，然后重新开始计数。|
| `STRING`       | 可打印的 ASCII 字符串。                         |
| `UTCTIMESTAMP` | 表示 UTC 日期时间的字符串。                            |

支持的 `UTCTIMESTAMP` 格式：

* `20011217-09:30:47` - 秒
* `20011217-09:30:47.123` - 毫秒
* `20011217-09:30:47.123456` - 微秒（用于来自交易所的消息）

客户端订单 ID 字段必须符合正则表达式 `^[a-zA-Z0-9-_]{1,36}$`：

* `ClOrdID (11)`
* `OrigClOrdID (41)`
* `MDReqID (262)`
* `ClListID (25014)`
* `OrigClListID (25015)`
* `CancelClOrdID (25034)`

## 消息组件

在示例消息中， `|` 字符用于表示 SOH 字符：

```
8=FIX.4.4|9=113|35=A|34=1|49=SPOT|52=20240612-08:52:21.636837|56=5JQmUOsm|98=0|108=30|25037=4392a152-3481-4499-921a-6d42c50702e2|10=051|
```

<a id="header"></a>

### Header

出现在每条消息的开头。

| Tag   | 名称         | 类型         | 是否必须 | 描述
|-------|--------------|--------------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 8     | BeginString  | STRING       | Y        | 始终为 `FIX.4.4`。 <br></br> 必须是消息的第一个字段。|
| 9     | BodyLength   | LENGTH       | Y        | 消息长度（以字节为单位）。 <br></br> 必须是消息的第二个字段。|
| 35    | MsgType      | STRING       | Y        | 必须是消息的第三个字段。 <br></br> 可能的值： <br></br>`0` - [HEARTBEAT](#heartbeat) <br></br>`1` - [TEST_REQUEST](#testrequest) <br></br>`3` - [REJECT](#reject) <br></br>`5` - [LOGOUT](#logout) <br></br>`8` - [EXECUTION_REPORT](#executionreport) <br></br> `9` - [ORDER_CANCEL_REJECT](#ordercancelreject) <br></br> `A` - [LOGON](#logon-main) <br></br> `D` - [NEW_ORDER_SINGLE](#newordersingle) <br></br> `E` - [NEW_ORDER_LIST](#neworderlist) <br></br> `F` - [ORDER_CANCEL_REQUEST](#ordercancelrequest) <br></br> `N` - [LIST_STATUS](#liststatus) <br></br> `q` - [ORDER_MASS_CANCEL_REQUEST](#ordermasscancelrequest) <br></br> `r` - [ORDER_MASS_CANCEL_REPORT](#ordermasscancelreport) <br></br> `XCN` - [ORDER_CANCEL_REQUEST_AND_NEW_ORDER_SINGLE](#ordercancelrequestandnewordersingle) <br></br> `XLQ` - [LIMIT_QUERY](#limitquery) <br></br> `XLR` - [LIMIT_RESPONSE](#limitresponse) <br></br> `B` - [NEWS](#news) <br></br> `x`- [INSTRUMENT_LIST_REQUEST](#instrumentlistrequest) <br></br> `y` - [INSTRUMENT_LIST](#instrumentlist) <br></br>`V` - [MARKET_DATA_REQUEST](#marketdatarequest) <br></br> `Y` - [MARKET_DATA_REQUEST_REJECT](#marketdatarequestreject) <br></br>`W` - [MARKET_DATA_SNAPSHOT](#marketdatasnapshot) <br></br>`X` - [MARKET_DATA_INCREMENTAL_REFRESH](#marketdataincrementalrefresh) <br></br> `XAK` - [ORDER_AMEND_KEEP_PRIORITY_REQUEST](#orderamendkeeppriorityrequest) <br></br> `XAR` - [ORDER_AMEND_REJECT](#orderamendreject) |
| 49    | SenderCompID | STRING       | Y        | 在账户的活动会话中必须是独特的。<br></br> 必须使用正则表达式：`^[a-zA-Z0-9-_]{1,8}$` |
| 56    | TargetCompID | STRING       | Y        | 在客户端的消息中必须设置为`SPOT`。|
| 34    | MsgSeqNum    | SEQNUM       | Y        | 整数消息序列号。 <br></br> 会导致间隙的值将被拒绝。|
| 52    | SendingTime  | UTCTIMESTAMP | Y        | 消息传输时间（始终以 UTC 表示）。|
| 25000 | RecvWindow   | FLOAT          | N        | 在`SendingTime (52)` 后，用于标识请求有效时间的毫秒数。 <br></br> 在 [Logon`<A>`](#logon-request) 中默认为 `5000` 毫秒，最大值为 `60000` 毫秒。 <br> 支持最多三位小数的精度（例如 6000.346），以便可以指定微秒。|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

<a id="trailer"></a>

### Trailer

出现在每条消息的末尾。

| Tag | 名称     | 类型   | 是否必须 | 描述                                                                                                                                                                                                                                                                                                                                                      |
|-----|----------|--------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 10  | CheckSum | STRING | Y        | 始终为3个字符的数字字符串。其取值是通过对消息中每个前导字符的 ASCII 值进行求和计算得出的，其中包括 start-of-header（SOH）。 <br></br> 结果总和除以 256，余数形成 CheckSum 值。 <br></br> 为保持固定长度，CheckSum 字段采用右对齐并按需要进行零填充。 |

## Administrative Messages

<a id="heartbeat"></a>

### Heartbeat <code>&lt;0&gt;</code>

如果在心跳间隔（[Logon`<A>`](#logon-main) 中的 `HeartBtInt (108)`）的期间没有传出流量，则由服务器发送。

由客户端发送则用于指示会话正常。

由客户端或服务器发送，用于给予 [TestRequest`<1>`](#testrequest) 消息有关响应。

| Tag | 名称      | 类型   | 是否必须 | 描述                                                                                              |
|-----|-----------|--------|----------|----------------------------------------------------------------------------------------------------------|
| 112 | TestReqID | STRING | N        | 当 Heartbeat`<35>` 作为对 TestRequest`<1>` 的响应发送时，必须镜像 TestRequest`<1>` 中的值。 |

<a id="testrequest"></a>

### TestRequest <code>&lt;1&gt;</code>

如果在心跳间隔（[Logon`<A>`](#logon-main) 中的 `HeartBtInt (108)`）的期间没有传入流量，则由服务器发送。

由客户端发送用于请求 [Heartbeat`<0>`](#heartbeat) 响应。

> [!NOTE]
> 如果客户端未能在超时范围内发送带有正确 `TestReqID (112)` 的 Heartbeat`<0>` 来响应 TestRequest`<1>` ，其连接将被断开。

| Tag | 名称      | 类型   | 必填 | 描述                                                            |
|-----|-----------|--------|----------|------------------------------------------------------------------------|
| 112 | TestReqID | STRING | Y        | 任意字符串，必须包含在 Heartbeat`<0>` 响应中。 |

<a id="reject"></a>

### Reject <code>&lt;3&gt;</code>

由服务器发送，用以响应无法处理的无效消息。

如果无法接受新连接，则由服务器发送。
请参阅 [Connection Limits](#connection-limits)。

请参阅 `Text (58)` 和 `ErrorCode (25016)` 字段以了解拒绝原因。

| Tag   | 名称                | 类型   | 必填 | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|-------|---------------------|--------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 45    | RefSeqNum           | INT    | N        | 导致此 Reject`<3>` 发送的被拒绝消息的 `MsgSeqNum (34)`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| 371   | RefTagID            | INT    | N        | 如果存在，标识直接导致此 Reject`<3>` 消息发送的字段。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 372   | RefMsgType          | STRING | N        | 导致此 Reject`<3>` 发送的被拒绝消息的 `MsgType (35)`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| 373   | SessionRejectReason | INT    | N        | 拒绝的原因，可以是以下值之一。 <br></br> 通常伴随附加的文字描述 <br></br> 可能的值： <br></br>`0`- INVALID_TAG_NUMBER <br></br> `1` - REQUIRED_TAG_MISSING <br></br> `2` - TAG_NOT_DEFINED_FOR_THIS_MESSAGE_TYPE <br></br> `3` - UNDEFINED_TAG <br></br> `5` - VALUE_IS_INCORRECT <br></br> `6` - INCORRECT_DATA_FORMAT_FOR_VALUE <br></br> `8` - SIGNATURE_PROBLEM <br></br> `10` - SENDINGTIME_ACCURACY_PROBLEM   <br></br> `12` - XML_VALIDATION_ERROR <br></br> `13` - TAG_APPEARS_MORE_THAN_ONCE <br></br> `14` - TAG_SPECIFIED_OUT_OF_REQUIRED_ORDER <br></br> `15` - REPEATING_GROUP_FIELDS_OUT_OF_ORDER <br></br> `16` - INCORRECT_NUMINGROUP_COUNT_FOR_REPEATING_GROUP<br></br> `99` - OTHER |
| 25016 | ErrorCode           | INT    | N        | API 错误代码（参见 [错误代码](errors_CN.md)）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| 58    | Text                | STRING | N        | 人类可读的错误信息。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |

<a id="logon-main"></a>

### Logon<code>&lt;A&gt;</code>

由客户端发送，用以验证连接。
Logon`<A>` 必须是客户端发送的第一条消息。

由服务器发送，用以响应成功的登录。

> [!NOTE]
> Logon`<A>` 在整个会话期间只能发送一次。

<a id="logon-request"></a>

#### Logon Request

| Tag | 名称     | 类型   | 是否必须 | 描述                                                                                                                                        |
|-------|-----------------|---------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| 98    | EncryptMethod   | INT     | Y        | 必须为 `0`。                                                                                                                                |
| 108   | HeartBtInt      | INT     | Y        | 必须在 [5, 60] 范围内。Heartbeat 间隔（秒）。                                                                                |
| 95    | RawDataLength   | LENGTH  | Y        | 紧随此字段之后的 `RawData (96)` 字段的长度。                                                                           |                                                                                                                                                                                                                                                                                                                              |
| 96    | RawData         | DATA    | Y        | 签名。 [如何签署 Logon`<A>` 请求](#signaturecomputation)。                                                                                |
| 141   | ResetSeqNumFlag | BOOLEAN | Y        | 必须为 `Y`。                                                                                                                                |
| 553   | Username        | STRING  | Y        | API key。**仅支持 Ed25519 API keys。**                                                                                                  |
| 25035 | MessageHandling | INT     | Y        | 可能的值: <br></br> `1` - UNORDERED <br></br> `2` - SEQUENTIAL <br></br> 请参阅 [关于消息处理顺序](#orderedmode) 了解更多信息。 |
| 25036 | ResponseMode    | INT     | N        | 请参阅 [响应模式](#responsemode)。
| 9406  | DropCopyFlag    |BOOLEAN   | N       |登录到 Drop Copy 会话时，必须设置为"Y"。|                                                                                              |

**示例消息:**

```
8=FIX.4.4|9=248|35=A|34=1|49=5JQmUOsm|52=20240612-08:52:21.613|56=SPOT|95=88|96=KhJLbZqADWknfTAcp0ZjyNz36Kxa4ffvpNf9nTIc+K5l35h+vA1vzDRvLAEQckyl6VDOwJ53NOBnmmRYxQvQBQ==|98=0|108=30|141=Y|553=W5rcOD30c0gT4jHK8oX5d5NbzWoa0k4SFVoTHIFNJVZ3NuRpYb6ZyJznj8THyx5d|25035=1|10=000|
```

<a id="logon-response"></a>

#### Logon Response

|Tag | 名称     | 类型   | 是否必须 | 描述                               |
|-------|---------------|--------|----------|-------------------------------------------|
| 98    | EncryptMethod | INT    | Y        | 始终为 `0`。                               |
| 108   | HeartBtInt    | INT    | Y        | 镜像 Logon 请求中的值。     |
| 25037 | UUID          | STRING | Y        | 提供请求服务的 FIX API 的 UUID。 |

**示例消息:**

```
8=FIX.4.4|9=113|35=A|34=1|49=SPOT|52=20240612-08:52:21.636837|56=5JQmUOsm|98=0|108=30|25037=4392a152-3481-4499-921a-6d42c50702e2|10=051|
```

<a id="logout"></a>

### Logout <code>&lt;5&gt;</code>

发送此消息以启动关闭连接的过程，并对 Logout 进行响应。


|Tag | 名称     | 类型   | 是否必须 | 描述  |
|-----|------|--------|----------|-------------|
| 58  | Text | STRING | N        |             |

**示例消息:**

Logout 请求

```
8=FIX.4.4|9=55|35=5|34=3|49=GhQHzrLR|52=20240611-09:44:25.543|56=SPOT|10=249|
```

Logout 响应

```
8=FIX.4.4|9=84|35=5|34=4|49=SPOT|52=20240611-09:44:25.544001|56=GhQHzrLR|58=Logout acknowledgment.|10=212|
```

<a id="news"></a>

### News <code>&lt;B&gt;</code>

当服务器进入维护状态时，系统将会向客户端**每隔 10 秒发送一条** `News` 消息，并**持续 10 分钟**。

如果客户端未在规定的时间范围内关闭旧连接，那么服务器会将其注销并关闭会话。

收到此消息后，客户端应建立新会话并关闭旧会话。

发送的倒计时消息将是：

```
You'll be disconnected in %d seconds. Please reconnect.
```
剩余 10 秒时，系统将发送以下消息：
```
Your connection is about to be closed. Please reconnect.
```
如果客户端在收到上述消息后的 10 秒内没有关闭旧会话，那么服务器会将其注销并关闭会话。

|Tag | 名称     | 类型   | 是否必须 | 描述  |
|-----|------|--------|----------|-------------|
| 148 | Headline | STRING | Y | |

**示例消息:**

```
8=FIX.4.4|9=0000113|35=B|49=SPOT|56=OE|34=4|52=20240924-21:07:35.773537|148=Your connection is about to be closed. Please reconnect.|10=165|
```

### Resend Request <code>&lt;2&gt;</code>

目前不支持重新发送请求。

## Application Messages

### 下单消息

> [!NOTE]
> 以下消息只能用于 FIX 订单接入会话和 FIX Drop Copy 会话

<a id="newordersingle"></a>

#### NewOrderSingle<code>&lt;D&gt;</code>

由客户端发送，用以提交新订单并进行执行。

这个请求会把1个订单添加到 `EXCHANGE_MAX_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器中。

**未成交的订单计数:** 1

请参阅 [支持的订单类型](#ordertype) 了解支持的字段组合。

> [!NOTE]
> 许多字段会根据订单类型变为必填。
> 请参阅 [支持的订单类型](#NewOrderSingle-required-fields)。

| Tag | 名称     | 类型   | 是否必须 | 描述                                                                                                                                                                                                                                                                                                                  |
|-------|--------------------------|---------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 11    | ClOrdID                  | STRING  | Y        | 分配给订单的 `ClOrdID`。                                                                                                                                                                                                                                                                                       |
| 38    | OrderQty                 | QTY     | N        | 订单数量                                                                                                                                                                                                                                                                                                        |
| 40    | OrdType                  | CHAR    | Y        | 请参阅 [表格](#ordertype) 了解支持的订单类型和所需字段。<br></br>可能的值: <br></br> `1` - MARKET <br></br> `2` - LIMIT <br></br> `3` - STOP <br></br> `4` - STOP_LIMIT  <br></br> `P`- PEGGED                                                                                     |                                                                                        |
| 18    | ExecInst                 | CHAR    | N        | 可能的值: <br></br> `6` - PARTICIPATE_DONT_INITIATE                                                                                                                                                                                                                                                                        |
| 44    | Price                    | PRICE   | N        | 订单价格                                                                                                                                                                                                                                                                                                           |
| 54    | Side                     | CHAR    | Y        | 订单方向。<br></br>可能的值: <br></br> `1` - BUY <br></br> `2` - SELL                                                                                                                                                                                                                                                        |
| 55    | Symbol                   | STRING  | Y        | 下单的交易对。                                                                                                                                                                                                                                                                                                |
| 59    | TimeInForce              | CHAR    | N        | 可能的值: <br></br> `1` - GOOD_TILL_CANCEL <br></br> `3` - IMMEDIATE_OR_CANCEL <br></br> `4` - FILL_OR_KILL                                                                                                                                                                                                                          |
| 111   | MaxFloor                 | QTY     | N        | 用于冰山订单，指定订单在订单簿上的可见数量。                                                                                                                                                                                                                                       |
| 152   | CashOrderQty             | QTY     | N        | 以报价资产单位指定的订单数量，用于反向市场订单。                                                                                                                                                                                                                                         |
| 847   | TargetStrategy           | INT     | N        | 该值不能小于 `1000000`。                                                                                                                                                                                                                                                                                                                             |
| 7940  | StrategyID               | INT     | N        |                                                                                                                                                                                                                                                                                      |
| 25001 | SelfTradePreventionMode  | CHAR    | N        | 可能的值: <br></br> `1` - NONE <br></br> `2` - EXPIRE_TAKER <br></br> `3` - EXPIRE_MAKER <br></br> `4` - EXPIRE_BOTH <br></br> `5` - DECREMENT  <br> `6` - TRANSFER                                                                                                                                                                                                                      |
| 211   | PegOffsetValue           | FLOAT   | N        | 使用了 `PegOffsetType` 后，添加到挂钩的偏移值 |
| 1094  | PegPriceType             | CHAR    | N        | 定义了挂钩价格的类型 <br> 可能的值: <br> `4` - MARKET_PEG <br> `5` - PRIMARY_PEG|
| 835   | PegMoveType              | CHAR    | N        | 描述挂钩是固定的还是浮动的。挂钩订单必用且必须为 `1` (FIXED) |
| 836   | PegOffsetType            | CHAR    | N        | 定义了挂钩价格偏移类型。 <br> 可能的值: <br></br> `3`  - PRICE_TIER|
| 1100  | TriggerType              | CHAR    | N        | 可能的值: `4` - PRICE_MOVEMENT                                                                                                                                                                                                                                                                                        |
| 1101  | TriggerAction            | CHAR    | N        | 可能的值: <br></br> `1` - ACTIVATE                                                                                                                                                                                                                                                                                         |
| 1102  | TriggerPrice             | PRICE   | N        | 止盈止损订单的激活价格。请参阅 [表格](#ordertype)                                                                                                                                                                                                                                                              |
| 1107  | TriggerPriceType         | CHAR    | N        | 可能的值: <br></br> `2` - LAST_TRADE                                                                                                                                                                                                                                                                                       |
| 1109  | TriggerPriceDirection    | CHAR    | N        | 用于区分 StopLoss 和 TakeProfit 订单。请参阅 [表格](#ordertype)。<br></br>可能的值: <br></br> `U` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_UP_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE <br></br> `D` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_DOWN_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE |
| 25009 | TriggerTrailingDeltaBips | INT     | N        | 提供以创建追踪订单。                                                                                                                                                                                                                                                                                           |
| 25032 | SOR                      | BOOLEAN | N        | 是否为此订单激活 SOR。                                                                                                                                                                                                                                                                                      |

**示例消息:**

```
8=FIX.4.4|9=114|35=D|34=2|49=qNXO12fH|52=20240611-09:01:46.228|56=SPOT|11=1718096506197867067|38=5|40=2|44=10|54=1|55=LTCBNB|59=4|10=016|
```

**响应:**

* 如果订单被接受， [ExecutionReport`<8>`](#executionreport)的 `ExecType (150)` 值为 `NEW (0)`。
* 如果订单被拒绝， [ExecutionReport`<8>`](#executionreport)的 `ExecType (150)` 值为 `REJECTED (8)`。
* 如果消息被拒绝，则为 [Reject`<3>`](#reject)。

<a id="ordertype"></a>

##### 支持的订单类型

| 订单名称                                | Binance OrderType   | 方向        | 必填字段值                              | 用户值必填字段                   |
|---------------------------------------|---------------------|-------------|------------------------------------------|----------------------------------|
| Market order                          | `MARKET`            | BUY 或 SELL  | <code>40=1&#124;</code>                                 |                                  |
| Limit order                           | `LIMIT`             | BUY 或 SELL  | <code>40=2&#124;</code>                                 |                                  |
| Limit maker order                     | `LIMIT_MAKER`       | BUY 或 SELL  | <code>40=2&#124;18=6&#124;</code>                           |                                  |
| Buy stop loss order                   | `STOP_LOSS`         | BUY         | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102                             |
| Buy trailing stop loss order          | `STOP_LOSS`         | BUY         | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102,25009                        |
| Buy stop loss limit order             | `STOP_LOSS_LIMIT`   | BUY         | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102                             |
| Buy trailing stop loss limit order    | `STOP_LOSS_LIMIT`   | BUY         | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102,25009                        |
| Sell stop loss order                  | `STOP_LOSS`         | SELL        | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102                             |
| Sell trailing stop loss order         | `STOP_LOSS`         | SELL        | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102,25009                        |
| Sell stop loss limit order            | `STOP_LOSS_LIMIT`   | SELL        | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102                             |
| Sell trailing stop loss limit order   | `STOP_LOSS_LIMIT`   | SELL        | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102,25009                        |
| Buy take profit order                 | `TAKE_PROFIT`       | BUY         | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102                             |
| Buy trailing take profit order        | `TAKE_PROFIT`       | BUY         | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102,25009                        |
| Buy trailing take profit order        | `TAKE_PROFIT`       | BUY         | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;</code>         | 25009                             |
| Buy take profit order                 | `TAKE_PROFIT_LIMIT` | BUY         | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102                             |
| Buy trailing take profit limit order  | `TAKE_PROFIT_LIMIT` | BUY         | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102,25009                        |
| Buy trailing take profit limit order  | `TAKE_PROFIT_LIMIT` | BUY         | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;</code>         | 25009                             |
| Sell take profit order                | `TAKE_PROFIT`       | SELL        | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102                             |
| Sell trailing take profit order       | `TAKE_PROFIT`       | SELL        | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102,25009                        |
| Sell trailing take profit order       | `TAKE_PROFIT`       | SELL        | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;</code>         | 25009                             |
| Sell take profit limit order          | `TAKE_PROFIT_LIMIT` | SELL        | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102                             |
| Sell trailing take profit limit order | `TAKE_PROFIT_LIMIT` | SELL        | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102,25009                        |
| Sell trailing take profit limit order | `TAKE_PROFIT_LIMIT` | SELL        | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;</code>         | 25009                             |        |


<a id="NewOrderSingle-required-fields"></a>

基于 Binance OrderType 的必填字段:

| Binance OrderType   | 额外的必填字段                    | 额外的信息
|---------------------|---------------------------------|---------------|
| `LIMIT`             | 38, 44, 59                      |               |
| `MARKET`            | 38 或 152                       | `MARKET` 订单使用 `OrderQty (38)` 字段来指定用户希望以市场价格买入或卖出的 `base asset` 数量。<br></br>  例如，用户可以使用交易对为 `BTCUSDT` 的 `MARKET` 订单来指定买入或卖出多少 BTC。<br></br><br></br> 用户使用 `MARKET` 订单中的 `quoteOrderQty` 字段来指定花费（买入时）或接收（卖出时）`quote` 资产。正确的 `quantity` 将由市场流动性和 `quoteOrderQty` 来决定。 <br></br> 用交易对 BTCUSDT来举例：<br></br> `BUY` 方订单将使用由 `quoteOrderQty` 定义的 USDT 数额来购买尽可能多的 BTC。<br></br> `SELL` 方订单将出售 BTC 来换取尽可能多的由 `CashOrderQty（152)` 定义的 USDT 数额。 |
| `STOP_LOSS`         | 38, 1102 或 25009               | 当满足条件时，将执行 `MARKET` 订单。（例如，满足 `TriggerPrice (1102)` 的条件或激活 `TriggerTrailingDeltaBips (25009)`）|
| `STOP_LOSS_LIMIT`   | 38, 44, 59, 1102 或 25009       |                |
| `TAKE_PROFIT`       | 38, 1102 或 25009               | 当满足条件时，这将执行`MARKET`订单。（例如，满足 `TriggerPrice (1102)` 的条件或激活 `TriggerTrailingDeltaBips (25009)`） |
| `TAKE_PROFIT_LIMIT` | 38, 44, 59, 1102 或 25009       |                |
| `LIMIT_MAKER`       | 38, 44                          | 这是一个 `LIMIT` 订单。如果订单被立即匹配并作为 taker 进行交易，那么该订单将被拒绝。<br></br> 这也称为 POST-ONLY 订单。|

<a id="executionreport"></a>

#### ExecutionReport<code>&lt;8&gt;</code>

每当订单状态发生变化时由服务器发送。

> [!NOTE]
> * 默认情况下，ExecutionReport`<8>` 会发送该账户的所有订单，包括在不同连接中提交的订单。 请参阅 [响应模式](#responsemode) 来了解其他行为选项。
> * FIX API 应该为 ExecutionReport<code>&lt;8&gt;</code> 推送提供更好的性能。

| Tag | 名称     | 类型   | 是否必须 | 描述                                                                                                                                                                                                                                                                                                                  |
|-------|--------------------------|--------------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 17    | ExecID                   | STRING       | N        | 被拒绝订单时省略。                                                                                                                                                                                                                                                                                                  |
| 11    | ClOrdID                  | STRING       | N        | 列表中分配的 `ClOrdID`。                                                                                                                                                                                                                                                                            |
| 41    | OrigClOrdID              | STRING       | N        | 订单的原始 `ClOrdID`。                                                                                                                                                                                                                                                                                             |
| 37    | OrderID                  | INT          | N        | 由交易所分配。                                                                                                                                                                                                                                                                                                        |
| 38    | OrderQty                 | QTY          | N        | 订单数量。                                                                                                                                                                                                                                                                                                       |
| 40    | OrdType                  | CHAR         | Y        | 可能的值: <br></br> `1` - MARKET <br></br> `2` - LIMIT <br></br> `3` - STOP_LOSS <br></br> `4` - STOP_LIMIT <br></br> `P` - PEGGED                                                                                                                                                                                                                               |
| 54    | Side                     | CHAR         | Y        | 可能的值: <br></br> `1` - BUY <br></br> `2` - SELL                                                                                                                                                                                                                                                                              |
| 55    | Symbol                   | STRING       | Y        | 订单的交易对。                                                                                                                                                                                                                                                                                                         |
| 18    | ExecInst                 | CHAR         | N        | 可能的值: <br></br> `6` - PARTICIPATE_DONT_INITIATE                                                                                                                                                                                                                                                                        |
| 44    | Price                    | PRICE        | N        | 订单价格。                                                                                                                                                                                                                                                                                                          |
| 59    | TimeInForce              | CHAR         | N        | 可能的值: <br></br> `1` - GOOD_TILL_CANCEL <br></br> `3` - IMMEDIATE_OR_CANCEL <br></br> `4` - FILL_OR_KILL                                                                                                                                                                                                                          |
| 60    | TransactTime             | UTCTIMESTAMP | N        | 此事件发生的时间戳。                                                                                                                                                                                                                                                                                          |
| 25018 | OrderCreationTime        | INT          | N        |                                                                                                                                                                                                                                                                                                                              |
| 111   | MaxFloor                 | QTY          | N        | 出现在冰山订单中。                                                                                                                                                                                                                                                                                                   |
| 66    | ListID                   | STRING       | N        | 出现在列表订单中。                                                                                                                                                                                                                                                                                                      |
| 152   | CashOrderQty             | QTY          | N        | 以 quote asset 单位指定的订单数量。                                                                                                                                                                                                                                                                                 |
| 847   | TargetStrategy           | INT          | N        | 订单提交请求中的 `TargetStrategy (847)`。                                                                                                                                                                                                                                                                     |
| 7940  | StrategyID               | INT          | N        | 订单提交请求中的 `StrategyID (7940)`。                                                                                                                                                                                                                                                                        |
| 25001 | SelfTradePreventionMode  | CHAR         | N        | 可能的值: <br></br> `1` - NONE <br></br> `2` - EXPIRE_TAKER <br></br> `3` - EXPIRE_MAKER <br></br>`4` - EXPIRE_BOTH <br></br> `5` - DECREMENT  <br> `6` - TRANSFER                                                                                                                                                                                                                        |
| 150   | ExecType                 | CHAR         | Y        | **注意:** 如果订单因 `SelfTradePreventionMode(25013)` 过期，字段 `PreventedMatchID(25024)` 会被显示。 <br></br> 可能的值: <br></br> `0` - NEW <br></br> `4` - CANCELED <br></br> `5` - REPLACED <br></br> `8` - REJECTED <br></br> `F` - TRADE <br></br>`C` - EXPIRED                                                                   |
| 14    | CumQty                   | QTY          | Y        | 在此订单上交易的 base asset 总数。                                                                                                                                                                                                                                                                            |
| 151   | LeavesQty                | QTY          | N        | 剩余的可执行数量。                                                                                                                                                                                                                                                                                    |
| 25017 | CumQuoteQty              | QTY          | N        | 在此订单上交易的 quote asset 总数。                                                                                                                                                                                                                                                                             |
| 1057  | AggressorIndicator       | BOOLEAN      | N        | 出现在 trade execution reports 中。 <br></br>指示订单在交易中是否为 taker。                                                                                                                                                                                                                                |
| 1003  | TradeID                  | STRING       | N        | 出现在 trade execution reports 中。                                                                                                                                                                                                                                                                                          |
| 31    | LastPx                   | PRICE        | N        | 最后一次执行的价格。                                                                                                                                                                                                                                                                                             |
| 32    | LastQty                  | QTY          | Y        | 最后一次执行的数量。                                                                                                                                                                                                                                                                                          |
| 39    | OrdStatus                | CHAR         | Y        | 可能的值: <br></br> `0` - NEW <br></br> `1` - PARTIALLY_FILLED <br></br> `2` - FILLED <br></br> `4` - CANCELED `6` - PENDING_CANCEL<br></br> `8` - REJECTED <br></br> `A` - PENDING_NEW <br></br> `C` - EXPIRED <br></br> 注意 FIX 不支持 `EXPIRED_IN_MATCH` 状态，并在 FIX 中转换为 `EXPIRED`。                                    |                                                                                                                                                                                                                                                                                           |
| 70 | AllocID                  | INT          | N        | 由交易所分配的分配 ID。                                                                                                                                                                                                                                                                                   |
| 574 | MatchType                | INT          | N        | 可能的值:<br></br>`1` - ONE_PARTY_TRADE_REPORT<br></br>`4` - AUTO_MATCH                                                                                                                                                                                                                                                         |
| 25021 | WorkingFloor             | INT          | N        | 出现在可能有分配的订单中。                                                                                                                                                                                                                                                                        |
| 25022 | TrailingTime             | UTCTIMESTAMP          | N        | 仅出现在追踪止损订单中。                                                                                                                                                                                                                                                                                       |
| 636   | WorkingIndicator         | BOOLEAN      | N        | 当此订单进入订单簿时会被设置为 `Y`。                                                                                                                                                                                                                                                                                |
| 25023 | WorkingTime              | UTCTIMESTAMP | N        | 此订单出现在订单簿上的时间。                                                                                                                                                                                                                                                                                  |
| 25024 | PreventedMatchID         | INT          | N        | 仅出现在因 STP 过期的订单中。                                                                                                                                                                                                                                                                             |
| 25025 | PreventedExecutionPrice  | PRICE        | N        | 仅出现在因 STP 过期的订单中。                                                                                                                                                                                                                                                                             |
| 25026 | PreventedExecutionQty    | QTY          | N        | 仅出现在因 STP 过期的订单中。                                                                                                                                                                                                                                                                             |
| 25027 | TradeGroupID             | INT          | N        | 仅出现在因 STP 过期的订单中。                                                                                                                                                                                                                                                                             |
| 25028 | CounterSymbol            | STRING       | N        | 仅出现在因 STP 过期的订单中。                                                                                                                                                                                                                                                                             |
| 25029 | CounterOrderID           | INT          | N        | 仅出现在因 STP 过期的订单中。                                                                                                                                                                                                                                                                             |
| 25030 | PreventedQty             | QTY          | N        | 仅出现在因 STP 过期的订单中。                                                                                                                                                                                                                                                                             |
| 25031 | LastPreventedQty         | QTY          | N        | 仅出现在因 STP 过期的订单中。                                                                                                                                                                                                                                                                             |
| 25032 | SOR                      | BOOLEAN      | N        | 出现在使用 SOR 的订单中。                                                                                                                                                                                                                                                                                            |
| 25016 | ErrorCode                | INT          | N        | API 错误代码（请参阅 [错误代码](errors_CN.md)）。                                                                                                                                                                                                                                                                               |
| 58    | Text                     | STRING       | N        | 可读的错误消息。                                                                                                                                                                                                                                                                                                |
| 136   | NoMiscFees               | NUMINGROUP   | N        | 杂费的重复组数量。                                                                                                                                                                                                                                                                            |
| =>137 | MiscFeeAmt               | QTY          | Y        | 以 `MiscFeeCurr(138)` 资产计价的费用金额                                                                                                                                                                                                                                                                       |
| =>138 | MiscFeeCurr              | STRING       | Y        | 杂费的货币。                                                                                                                                                                                                                                                                                               |
| =>139 | MiscFeeType              | INT          | Y        | 可能的值: <br></br>`4` - EXCHANGE_FEES                                                                                                                                                                                                                                                                                     |
| 1100  | TriggerType              | CHAR         | N        | 可能的值: <br></br>`4` - PRICE_MOVEMENT                                                                                                                                                                                                                                                                                    |
| 1101  | TriggerAction            | CHAR         | N        | 可能的值: <br></br> `1` - ACTIVATE                                                                                                                                                                                                                                                                                         |
| 1102  | TriggerPrice             | PRICE        | N        | 止盈止损订单的激活价格。请参阅 [表格](#ordertype)                                                                                                                                                                                                                                                              |
| 1107  | TriggerPriceType         | CHAR         | N        | 可能的值: <br></br> `2` - LAST_TRADE                                                                                                                                                                                                                                                                                       |
| 1109  | TriggerPriceDirection    | CHAR         | N        | 用于区分止损和止盈订单。请参阅 [表格](#ordertype)。<br></br>可能的值: <br></br> `U` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_UP_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE <br></br> `D` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_DOWN_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE |
| 25009 | TriggerTrailingDeltaBips | INT          | N        | 仅出现在追踪止损订单中。|
| 211   | PegOffsetValue           | FLOAT   | N        | 使用了 `PegOffsetType` 后，添加到挂钩的偏移值 |
| 1094  | PegPriceType             | CHAR    | N        | 定义了挂钩价格的类型 <br> 可能的值: <br> `4` - MARKET_PEG <br> `5` - PRIMARY_PEG|
| 835   | PegMoveType              | CHAR    | N        | 描述挂钩是固定的还是浮动的。挂钩订单必用且必须为 `1` (FIXED) |
| 836   | PegOffsetType            | CHAR    | N        | 定义了挂钩价格偏移类型。 <br> 可能的值: <br></br> `3`  - PRICE_TIER|
| 839   | PeggedPrice              | PRICE   | N        | 订单所挂钩的当前价格|

**示例消息:**

```
8=FIX.4.4|9=330|35=8|34=2|49=SPOT|52=20240611-09:01:46.228950|56=qNXO12fH|11=1718096506197867067|14=0.00000000|17=144|32=0.00000000|37=76|38=5.00000000|39=0|40=2|44=10.00000000|54=1|55=LTCBNB|59=4|60=20240611-09:01:46.228000|150=0|151=5.00000000|636=Y|1057=Y|25001=1|25017=0.00000000|25018=20240611-09:01:46.228000|25023=20240611-09:01:46.228000|10=095|
```

<a id="ordercancelrequest"></a>

#### OrderCancelRequest<code>&lt;F&gt;</code>

由客户发送的，用以取消订单或订单列表。
* 要取消订单，需要 `OrderID (11)` 或 `OrigClOrdID (41)`。
  * 当同时提供 `OrderID (37)` 和 `OrigClOrdID (41)` 两个参数时，系统首先将会使用 `OrderID` 来搜索订单。然后， 查找结果中的 `OrigClOrdID` 的值将会被用来验证订单。如果两个条件都不满足，则请求将被拒绝。
* 要取消订单列表，需要 `ListID (66)` 或 `OrigClListID (25015)`。
  * 当同时提供 `ListID (66)` 和 `OrigClListID (25015)` 两个参数时，系统首先将会使用 `ListID` 来搜索订单列表。然后， 查找结果中的 `OrigClListID` 将会被用来验证订单列表。如果两个条件都不满足，请求将被拒绝。

如果已取消的订单是订单列表的一部分，则整个订单列表将被取消。

**注意：**

* 当仅发送 `orderId` 时，取消订单的执行(单个 Cancel 或作为 Cancel-Replace 的一部分)总是更快。发送 `origClientOrderId` 或同时发送 `orderId` + `origClientOrderId` 会稍慢。

| Tag | 名称     | 类型   | 是否必须 | 描述|
|---   | ---| ---| ---| --- |
|  11 |ClOrdID |STRING |Y |请求的 `ClOrdID`。|
| 41|OrigClOrdID|STRING |N|要取消订单的 `ClOrdID (11)`。|
|37 |Order ID|STRING |N|要取消订单的 `OrderID (37)`。
|25015 |OrigClListID|STRING |N|要取消订单列表的 `ClListID (25014)`。|
|66 |ListID|STRING |N|要取消订单列表的 `ListID (66)`。
|55 | Symbol|STRING |Y  |要取消订单的交易对。
|25002 |CancelRestrictions |INT |N|取消的限制。可能的值：<br></br> `1` - ONLY_NEW <br></br> `2` - ONLY_PARTIALLY_FILLED |

**示例消息：**
```
8=FIX.4.4|9=93|35=F|34=2|49=ieBwvCKy|52=20240613-01:11:13.784|56=SPOT|11=1718241073695674483|37=2|55=LTCBNB|10=210|
```

**响应：**
* 对于每个被取消的订单，[ExecutionReport`<8>`](#executionreport)的 `ExecType (150)` 值为 `CANCELED (4)`。
* 如果订单列表中的订单被取消，则为 [ListStatus`<N>`](#liststatus)。
* 如果取消被拒绝，则为 [OrderCancelReject`<9>`](#ordercancelreject)。
* 如果消息被拒绝，则为 [Reject`<3>`](#reject)。

<a id="ordercancelreject"></a>

#### OrderCancelReject <code>&lt;9&gt;</code>

当 [OrderCancelRequest`<F>`](#ordercancelrequest) 失败时，由服务器发送的消息。

| Tag | 名称     | 类型   | 是否必须 | 描述                                                                                                               |
|-------|--------------------|--------|----------|---------------------------------------------------------------------------------------------------------------------------|
| 11    | ClOrdID            | STRING | Y        | 来自取消请求的 `ClOrdID (11)`。                                                                                     |
| 41    | OrigClOrdID        | STRING | N        | 来自取消请求的 `OrigClOrdID (41)`。                                                                               |
| 37    | OrderID            | INT    | N        | 来自取消请求的 `OrderID (37)`。                                                                                  |
| 25015 | OrigClListID       | STRING | N        | 来自取消请求的 `OrigClListID (25015)`。                                                                           |
| 66    | ListID             | STRING | N        | 来自取消请求的 `ListID (66)`。                                                                                    |
| 55    | Symbol             | STRING | Y        | 来自取消请求的 `Symbol (55)`。                                                                                    |
| 25002 | CancelRestrictions | INT    | N        | 来自取消请求的 `CancelRestrictions (25002)`。                                                                     |
| 434   | CxlRejResponseTo   | CHAR   | Y        | 此 OrderCancelReject`<9>` 响应的请求类型。<br></br> 可能的值： <br></br> `1` - ORDER_CANCEL_REQUEST |
| 25016 | ErrorCode          | INT    | Y        | API error code (参考 [错误代码](errors_CN.md)).                                                                            |
| 58    | Text               | STRING | Y        | 可读的错误消息。                                                                                           |

**示例消息：**

```
8=FIX.4.4|9=137|35=9|34=2|49=SPOT|52=20240613-01:12:41.320869|56=OlZb8ht8|11=1718241161272843932|37=2|55=LTCBNB|58=Unknown order sent.|434=1|25016=-1013|10=087|
```

<a id="ordercancelrequestandnewordersingle"></a>

#### OrderCancelRequestAndNewOrderSingle<code>&lt;XCN&gt;</code>

由客户发送，用以取消订单并提交新订单以供执行。

* 要取消订单，需要 `OrderID (11)` 或 `OrigClOrdId (41)`。
* 当同时提供 `OrderID (37)` 和 `OrigClOrdID (41)` 两个参数时，系统首先将会使用 `OrderID` 来搜索订单。然后， 查找结果中的 `OrigClOrdID` 的值将会被用来验证订单。如果两个条件都不满足，则请求将被拒绝。

在撤消订单和下单前会判断: 1) 过滤器参数, 以及 2) 目前下单数量。

即使请求中没有尝试发送新订单，比如(`newOrderResult: NOT_ATTEMPTED`)，未成交订单的数量仍然会加1。

**未成交的订单计数:** 1

在描述新订单时，请参阅 [支持的订单类型](#ordertype) 了解支持的字段组合。

> [!NOTE]
> 取消始终是优先处理的，紧接着的是提交新订单。

| Tag | 名称     | 类型   | 是否必须 | 描述
| --- |--- | ---| --- |---|
|25033|OrderCancelRequestAndNewOrderSingleMode|INT |Y |用于定义： 如果取消失败，将会采取的后续操作 。 <br> 可能的值 ： <br> `1` - STOP_ON_FAILURE <br></br> `2` -ALLOW_FAILURE |
|25038|OrderRateLimitExceededMode|INT |N|用于定义： 如果超过未成交订单计数，将会如何处理取消请求。<br> 可能的值 ： <br>`1` - DO_NOTHING <br></br> `2` - CANCEL_ONLY |
| 37    | OrderID                                 | INT    | N        | 来自于取消订单的 `OrderID`。                      |
| 25034 | CancelClOrdID                           | STRING | N        | 待取消的 `ClOrdID`。                             |
| 41    | OrigClOrdID                             | STRING | N        | 来自待取消订单的 `ClOrdID`。                      |
| 11    | ClOrdID                                 | STRING | Y        | 用于分配给新订单的 `ClOrdID`。                    |
| 25002 | CancelRestrictions                      | INT    | N        | 取消的限制。可能值 ：<br></br> `1` - ONLY_NEW <br></br> `2` - ONLY_PARTIALLY_FILLED|
| 38    | OrderQty                                | QTY    | N        | 新订单的数量                                                      |
| 40    | OrdType                                 | CHAR   | Y        | 请参阅 [表格]（#ordertype）以了解支持的订单类型以及相关的必填字段 。 <br></br> 可能的值 ：<br></br> `1` - MARKET <br></br> `2` - LIMIT <br></br> `3` - STOP <br></br> `4` - STOP_LIMIT <br></br> `P` - PEGGED         |
| 18    | ExecInst                                | CHAR   | N        | 可能的值： <br></br> `6` - PARTICIPATE_DONT_INITIATE  |
| 44    | Price                                   | PRICE  | N        | 新订单的价格  |
| 54    | Side                                    | CHAR   | Y        | 订单方向。<br></br>可能的值： <br></br> `1` - BUY <br></br> `2` - SELL    |
| 55    | Symbol                                  | STRING | Y        | 取消并下新订单所用的交易对。                                 |
| 59    | TimeInForce                             | CHAR   | N        | 可能的值 ： <br></br> `1` - GOOD_TILL_CANCEL <br></br> `3` - IMMEDIATE_OR_CANCEL <br></br> `4` - FILL_OR_KILL |
| 111   | MaxFloor                                | QTY    | N        | 用于冰山订单，用于定义订单在 orderbook 上的可见数量。|
| 152   | CashOrderQty                            | QTY    | N        | 用于反向市价单，用于定义 quote asset 单位中的订单数量。        |
| 847   | TargetStrategy                          | INT    | N        | 该值不能小于 `1000000`。                                                |
| 7940  | StrategyID                              | INT    | N        |  |
| 25001 | SelfTradePreventionMode                 | CHAR   | N        | 可能的值: <br></br> `1` - NONE <br></br> `2` - EXPIRE_TAKER <br></br> `3` - EXPIRE_MAKER <br></br> `4` - EXPIRE_BOTH <br></br> `5` - DECREMENT   <br> `6` - TRANSFER        |
| 211   | PegOffsetValue                          | FLOAT   | N       | 使用了 `PegOffsetType` 后，添加到挂钩的偏移值 |
| 1094  | PegPriceType                            | CHAR    | N       | 定义了挂钩价格的类型 <br> 可能的值: <br> `4` - MARKET_PEG <br> `5` - PRIMARY_PEG|
| 835   | PegMoveType                             | CHAR    | N       | 描述挂钩是固定的还是浮动的。挂钩订单必用且必须为 `1` (FIXED) |
| 836   | PegOffsetType                           | CHAR    | N       | 定义了挂钩价格偏移类型。 <br> 可能的值: <br></br> `3`  - PRICE_TIER|
| 1100  | TriggerType                             | CHAR   | N        | 可能的值: `4` - PRICE_MOVEMENT            |
| 1101  | TriggerAction                           | CHAR   | N        | 可能的值: <br></br> `1` - ACTIVATE              |
| 1102  | TriggerPrice                            | PRICE  | N        | 止盈止损订单的激活价格。参见 [表格](#ordertype)       |
| 1107  | TriggerPriceType                        | CHAR   | N        | 可能的值 <br></br> `2` - LAST_TRADE                      |
| 1109  | TriggerPriceDirection                   | CHAR   | N        | 用于区分止损订单和止盈订单。参考 [表格](#ordertype) .<br></br> 可能的值 ： <br></br> `U` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_UP_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE <br></br> `D` -TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_DOWN_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE |
| 25009 | TriggerTrailingDeltaBips                | INT    | N        | 用以创建追踪订单。 |

**示例消息：**

```
8=FIX.4.4|9=160|35=XCN|34=2|49=JS8iiXK6|52=20240613-02:31:53.753|56=SPOT|11=1718245913721036458|37=8|38=5|40=2|44=4|54=1|55=LTCBNB|59=1|111=1|25033=1|25034=1718245913721036819|10=229|
```

**响应：**

* 对于每个被取消的订单，[ExecutionReport`<8>`](#executionreport) 的 `ExecType (150)` 值为 `CANCELED (4)`。
* 对于新订单，[ExecutionReport`<8>`](#executionreport) 的 `ExecType (150)` 值为 `NEW (0)`。
* 如果新订单被拒绝，[ExecutionReport`<8>`](#executionreport) 的 `ExecType (150)` 值为 `REJECTED (8)`。
* 如果取消订单的请求被拒绝，则为 [OrderCancelReject`<9>`](#ordercancelreject)。
* 如果消息被拒绝，则为 [Reject`<3>`](#reject)。

<a id="ordermasscancelrequest"></a>

#### OrderMassCancelRequest<code>&lt;q&gt;</code>

由客户端发送，用以取消交易品种上的所有挂单。

> [!NOTE]
> 该账户的所有订单都将被取消，包括从不同连接中下的订单。

| Tag | 名称     | 类型   | 是否必须 | 描述 |
| --- |--- | ---| --- |---|
| 11  | ClOrdID               | STRING | Y        | 此批量取消请求的`ClOrdId`。  |
| 55  | Symbol                | STRING | Y        | 取消订单的交易对。 |
| 530 | MassCancelRequestType | CHAR   | Y        | 可能的值: <br></br> `1` - CANCEL_SYMBOL_ORDERS     |

**示例消息：**

```
8=FIX.4.4|9=95|35=q|34=2|49=dpYPesqv|52=20240613-01:24:36.948|56=SPOT|11=1718241876901971671|55=BTCUSDT|530=1|10=243|
```

**响应：**

* 对于每个被取消的订单，[ExecutionReport`<8>`](#executionreport) 的 `ExecType (150)` 值为 `CANCELED (4)`。
* 对于请求是否被接受或者拒绝，请参考 [OrderMassCancelReport`<r>`](#ordermasscancelreport) 的字段 `MassCancelResponse (531)`。
* 如果消息被拒绝，则为 [Reject`<3>`](#reject)。

<a id="ordermasscancelreport"></a>

由服务器发送，用以响应 [OrderMassCancelRequest`<q>`](#ordermasscancelrequest)。

| Tag | 名称     | 类型   | 是否必须 | 描述 |
| --- |--- | ---| --- |---|
| 55    | Symbol                 | STRING | Y        | 取消请求中的 `Symbol (55)`  。                                       |
| 11    | ClOrdID                | STRING | Y        | 取消请求中的 `ClOrdID （11)`。                                              |
| 530   | MassCancelRequestType  | CHAR   | Y        | 取消请求中的 `MassCancelRequestType (530)`。                            |
| 531   | MassCancelResponse     | CHAR   | Y        | 可能的值： <br></br> `0` - CANCEL_REQUEST_REJECTED <br></br> `1` - CANCEL_SYMBOL_ORDERS |
| 532   | MassCancelRejectReason | INT    | N        | 可能的值： <br></br> `99` - OTHER                                                  |
| 533   | TotalAffectedOrders    | INT    | N        | 取消了多少订单。                                                  |
| 25016 | ErrorCode              | INT    | N        | API 错误代码 (参考 [错误代码](errors_CN.md)).                                      |
| 58    | Text                   | STRING | N        | 可读的错误消息。                                                  |

**示例消息：**

```
8=FIX.4.4|9=109|35=r|34=2|49=SPOT|52=20240613-01:24:36.949763|56=dpYPesqv|11=1718241876901971671|55=LTCBNB|530=1|531=1|533=5|10=083|
```

<a id="neworderlist"></a>

#### NewOrderList<code>&lt;E&gt;</code>

由客户发送，用以提交需要执行的订单列表。
* `OTO` 或 `OTO` 订单将**2 个订单**添加到 `EXCHANGE_MAX_NUM_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器中。
* `OTOCO` 在 `EXCHANGE_MAX_NUM_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器的基础上添加**3个订单**。

**未成交的订单计数:**
* OCO: 2
* OTO: 2
* OTOCO: 3

订单列表中的订单是相互依赖的。
欲了解支持的订单类型和触发说明，请参考[支持的订单列表类型](#order-list-types)。

| Tag | 名称     | 类型   | 是否必须 | 描述|
| --- |--- | ---| --- |---|
| 25014    | ClListID                     | STRING     | Y        | `ClListID`，用于分配给订单列表。              |
| 1385     | ContingencyType              | INT        | N        | 可能的值 ： <br></br> `1` -ONE_CANCELS_THE_OTHER <br></br> `2` - ONE_TRIGGERS_THE_OTHER |
| 25046    | OPO                          | BOOLEAN    | N        | 设置为 `true` 时，将此订单列表设为 [OPO](./faqs/opo_CN.md)。|
| 73       | NoOrders                     | NUMINGROUP | N        | `Orders` 数组中的元素个数。只允许输入2或者3。 |
| =>11     | ClOrdID                      | STRING     | Y        | 用于分配给订单的`ClOrdID`     |
| =>38     | OrderQty                     | QTY        | N        | 订单数量                                                          |
| =>40     | OrdType                      | CHAR       | Y        | 请参阅 [表格](#ordertype) 以了解支持的订单类型及相关的必填字段 。<br></br>可能的值 ： <br></br> `1` - MARKET <br></br> `2` - LIMIT <br></br> `3` - STOP <br></br> `4` - STOP_LIMIT <br></br> `P` - PEGGED                  |
| =>18     | ExecInst                     | CHAR       | N        | 可能的值：<br></br> `6` - PARTICIPATE_DONT_INITIATE                      |
| =>44     | Price                        | PRICE      | N        | 订单价格|
| =>54     | Side                         | CHAR       | Y        | 订单的方向。 可能的值 ：<br></br> `1` - BUY <br></br> `2` - SELL                     |
| =>55     | Symbol                       | STRING     | Y        | 订单的交易对。|
| =>59     | TimeInForce                  | CHAR       | N        | 可能的值 ： <br></br> `1` - GOOD_TILL_CANCEL <br></br> `3` - IMMEDIATE_OR_CANCEL <br></br> `4` - FILL_OR_KILL  |
| =>111    | MaxFloor                     | QTY        | N        | 用于冰山订单，这指定了订单在 order book上的可见数量。|
| =>152    | CashOrderQty                 | QTY        | N        | 对于反向市场订单，在报价资产单位中指定的订单数量。|
| =>847    | TargetStrategy               | INT        | N        | 该值不能小于 `1000000`。    |
| =>7940   | StrategyID                   | INT        | N        | |
| =>25001  | SelfTradePreventionMode      | CHAR       | N        | 可能的值：<br></br> `1` - NONE <br></br>`2` - EXPIRE_TAKER <br></br> `3` - EXPIRE_MAKER <br></br> `4` - EXPIRE_BOTH <br></br> `5` - DECREMENT   <br> `6` - TRANSFER                                                                                                                                                                                                                       |
| =>211    | PegOffsetValue               | FLOAT      | N        | 使用了 `PegOffsetType` 后，添加到挂钩的偏移值 |
| =>1094   | PegPriceType                 | CHAR       | N        | 定义了挂钩价格的类型 <br> 可能的值: <br> `4` - MARKET_PEG <br> `5` - PRIMARY_PEG|
| =>835    | PegMoveType                  | CHAR       | N        | 描述挂钩是固定的还是浮动的。挂钩订单必用且必须为 `1` (FIXED) |
| =>836    | PegOffsetType                | CHAR       | N        | 定义了挂钩价格偏移类型。 <br> 可能的值: <br></br> `3`  - PRICE_TIER|
| =>1100   | TriggerType                  | CHAR       | N        | 可能的值: <br></br> `4` - PRICE_MOVEMENT                                                                                                                                                                                                                                                                                   |
| =>1101   | TriggerAction                | CHAR       | N        | 可能的值: <br></br> `1` - ACTIVATE                                                                                                                                                                                                                                                                                         |
| =>1102   | TriggerPrice                 | PRICE      | N        | 止盈止损订单的激活价格。参见 [表格](#ordertype)                                                                                                                                                                                                                                                              |
| =>1107   | TriggerPriceType             | CHAR       | N        | 可能的值: <br></br> `2` - LAST_TRADE                                                                                                                                                                                                                                                                                       |
| =>1109   | TriggerPriceDirection        | CHAR       | N        | 用于区分止损订单和止盈订单。 参见 [表格](#ordertype).<br></br>可能的值: <br></br> `U` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_UP_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE <br></br> `D` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_DOWN_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE |
| =>25009  | TriggerTrailingDeltaBips     | INT        | N        | 用于追踪订单。 |
| =>25010  | NoListTriggeringInstructions | NUMINGROUP | N        | `ListTriggeringInstructions` 数组中的元素个数。 |
| ==>25011 | ListTriggerType              | CHAR       | N        | 为了触发 pending order， working order 所需进行的操作。 <br></br> 可能的值: <br></br> `1` - ACTIVATED <br></br> `2` - PARTIALLY_FILLED <br></br> `3` - FILLED                                          |
| ==>25012 | ListTriggerTriggerIndex      | INT        | N        | 条件单的触发 Index：0-indexed。         |
| ==>25013 | ListTriggerAction            | CHAR       | N        | 在满足 ListTriggerType 条件后对此订单执行的操作。 <br></br> 可能的值: <br></br> `1` - RELEASE <br></br> `2` - CANCEL   |

**示例消息：**

```
8=FIX.4.4|9=236|35=E|34=2|49=Eg13pOvN|52=20240607-02:19:07.836|56=SPOT|73=2|11=w1717726747805308656|55=LTCBNB|54=2|38=1|40=2|44=0.25|59=1|11=p1717726747805308656|55=LTCBNB|54=2|38=1|40=1|25010=1|25011=3|25012=0|25013=1|1385=2|25014=1717726747805308656|10=171|
```

<a id="order-list-types"></a>

#### 支持的订单列表类型

> [!NOTE]
> 订单必须按照下表中 *订单名称* 中指定的顺序排列。

| 订单列表名称 |应急类型（1385） |订单名称                         |订单方 |允许的币安订单类型 |列出触发指令
| ---        |---           | ---                          | ---         |---             |---|
| OCO             | `1`                     | 1. below order<br></br><br></br>   2. above order                                                  | 1. below order=`SELL`<br></br><br></br>  2. above order=`SELL`                                                                   | 1. below order=`STOP_LOSS` 或 `STOP_LOSS_LIMIT`<br></br><br></br> 2. above order=`LIMIT_MAKER`                                                                                                      | 1. below order:  <br></br><code>25010=1&#124;25011=2&#124;25012=1&#124;25013=2&#124;</code><br></br><br></br>2. above order:        <br></br><code>25010=1&#124;25011=1&#124;25012=0&#124;25013=2&#124;</code>                                                                                                                                                                                                  |
| OCO             | `1`                     | 1. below order<br></br><br></br>   2. above order                                                  | 1. below order=`BUY` <br></br><br></br>  2. above order=`BUY`                                                                    | 1. below order=`LIMIT_MAKER`                   <br></br><br></br> 2. above order=`STOP_LOSS` 或 `STOP_LOSS_LIMIT`                                                                                   | 1. below order:  <br></br><code>25010=1&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code><br></br><br></br>2. above order:        <br></br><code>25010=1&#124;25011=2&#124;25012=0&#124;25013=2&#124;</code>                                                                                                                                                                                                  |
| OCO             | `1`                     | 1. below order<br></br><br></br>   2. above order                                                  | 1. below order=`SELL`<br></br><br></br>  2. above order=`SELL`                                                                   | 1. below order=`STOP_LOSS` 或 `STOP_LOSS_LIMIT`<br></br><br></br> 2. above order= `TAKE_PROFIT`                                                                                                     | 1. below order:  <br></br><code>25010=1&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code><br></br><br></br>2. above order:        <br></br><code>25010=1&#124;25011=1&#124;25012=0&#124;25013=2&#124;</code>                                                                                                                                                                                                  |
| OCO             | `1`                     | 1. below order<br></br><br></br>   2. above order                                                  | 1. below order=`BUY` <br></br><br></br>  2. above order=`BUY`                                                                    | 1. below order=`TAKE_PROFIT`                   <br></br><br></br> 2. above order = `STOP_LOSS` 或 `STOP_LOSS_LIMIT`                                                                                 | 1. below order:  <br></br><code>25010=1&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code><br></br><br></br>2. above order:        <br></br><code>25010=1&#124;25011=1&#124;25012=0&#124;25013=2&#124;</code>                                                                                                                                                                                                  |
| OTO             | `2`                     | 1. working order<br></br><br></br> 2. pending order                                                | 1. working order=`SELL` 或 `BUY`<br></br><br></br> 2. pending order=`SELL` 或 `BUY`                                              | 1. working order=`LIMIT` 或 `LIMIT_MAKER`      <br></br><br></br> 2. pending order=ANY                                                                                                              | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending order:      <br></br><code>25010=1&#124;25011=3&#124;25012=0&#124;25013=1&#124;</code>                                                                                                                                                                                                  |
| OTOCO           | `2`                     | 1. working order<br></br><br></br> 2. pending below order<br></br><br></br> 3. pending above order | 1. working order=`SELL` 或 `BUY`<br></br><br></br> 2. pending below order=`SELL`<br></br><br></br> 3. pending above order=`SELL` | 1. working order=`LIMIT` 或 `LIMIT_MAKER`      <br></br><br></br> 2. pending below order=`STOP_LOSS` 或 `STOP_LOSS_LIMIT`<br></br><br></br> 3. pending above order=`LIMIT_MAKER`                    | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending below order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=2&#124;25012=2&#124;25013=2&#124;</code><br></br><br></br>3. pending above order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code> |
| OTOCO           | `2`                     | 1. working order<br></br><br></br> 2. pending below order<br></br><br></br> 3. pending above order | 1. working order=`SELL` 或 `BUY`<br></br><br></br> 2. pending below order=`BUY`<br></br><br></br>  3. pending above order=`BUY`  | 1. working order=`LIMIT` 或 `LIMIT_MAKER`      <br></br><br></br> 2. pending below order=`LIMIT_MAKER`                   <br></br><br></br> 3. pending above order=`STOP_LOSS` 或 `STOP_LOSS_LIMIT` | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending below order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=2&#124;25013=2&#124;</code><br></br><br></br>3. pending above order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=2&#124;25012=1&#124;25013=2&#124;</code> |
| OTOCO           | `2`                     | 1. working order<br></br><br></br> 2. pending below order<br></br><br></br> 3. pending above order | 1. working order=`SELL` 或 `BUY`<br></br><br></br> 2. pending below order=`SELL`<br></br><br></br> 3. pending above order=`SELL` | 1. working order=`LIMIT` 或 `LIMIT_MAKER`      <br></br><br></br> 2. pending below order=`STOP_LOSS` 或 `STOP_LOSS_LIMIT`<br></br><br></br> 3. pending above order=`TAKE_PROFIT`                    | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending below order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=2&#124;25013=2&#124;</code><br></br><br></br>3. pending above order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code> |
| OTOCO           | `2`                     | 1. working order<br></br><br></br> 2. pending below order<br></br><br></br> 3. pending above order | 1. working order=`SELL` 或 `BUY`<br></br><br></br> 2. pending below order=`BUY`<br></br><br></br>  3. pending above order=`BUY`  | 1. working order=`LIMIT` 或 `LIMIT_MAKER`      <br></br><br></br> 2. pending below order=`TAKE_PROFIT`                   <br></br><br></br> 3. pending above order=`STOP_LOSS` 或 `STOP_LOSS_LIMIT` | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending below order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=2&#124;25013=2&#124;</code><br></br><br></br>3. pending above order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code> |
| OPO             | `2`                     | 1. working order<br></br><br></br> 2. pending order                                                | 1. working order=`BUY`<br></br><br></br> 2. pending order=`SELL`                                            | 1. working order=`LIMIT` or `LIMIT_MAKER`      <br></br><br></br> 2. pending order=ANY                                                                                                              | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending order:      <br></br><code>25010=1&#124;25011=3&#124;25012=0&#124;25013=1&#124;</code>
| OPOCO           | `2`                     | 1. working order<br></br><br></br> 2. pending below order<br></br><br></br> 3. pending above order | 1. working order=`BUY`<br></br><br></br> 2. pending below order=`SELL`<br></br><br></br> 3. pending above order=`SELL` | 1. working order=`LIMIT` or `LIMIT_MAKER`      <br></br><br></br> 2. pending below order=`STOP_LOSS` or `STOP_LOSS_LIMIT`<br></br><br></br> 3. pending above order=`LIMIT_MAKER`                    | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending below order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=2&#124;25012=2&#124;25013=2&#124;</code><br></br><br></br>3. pending above order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code> |
| OPOCO           | `2`                     | 1. working order<br></br><br></br> 2. pending below order<br></br><br></br> 3. pending above order | 1. working order=`BUY`<br><br> 2. pending below order=`SELL`<br></br><br></br> 3. pending above order=`SELL` | 1. working order=`LIMIT` or `LIMIT_MAKER`      <br></br><br></br> 2. pending below order=`STOP_LOSS` or `STOP_LOSS_LIMIT`<br></br><br></br> 3. pending above order=`TAKE_PROFIT` or `TAKE_PROFIT_LIMIT`  | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending below order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=2&#124;25013=2&#124;</code><br></br><br></br>3. pending above order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code> |

<a id="liststatus"></a>

#### ListStatus<code>&lt;N&gt;</code>

每当订单列表状态发生变化时，由服务器发送的消息。

> [!NOTE]
> 默认情况下，ListStatus`<N>` 会含有该账户的所有订单列表，包括在不同连接中提交的订单列表。
> 请参阅 [响应模式](#responsemode) 来了解其他行为选项。

| Tag | 名称     | 类型   | 是否必须 | 描述|
| --- |--- | --- | --- | ---|
| 55       | Symbol                       | STRING       | N        | 订单列表的交易对。  |
| 66       | ListID                       | STRING       | N        | 由交易所分配的订单列表 `ListID`。  |
| 25014    | ClListID                     | STRING       | N        | 分配给请求的订单列表 `ClListID` 。 |
| 25015    | OrigClListID                 | STRING       | N        | |
| 1385     | ContingencyType              | INT          | N        | 可能的值: <br></br> `1` - ONE_CANCELS_THE_OTHER <br></br> `2` - ONE_TRIGGERS_THE_OTHER    |
| 429      | ListStatusType               | INT          | Y        | 可能的值: <br></br> `2` - RESPONSE <br></br>`4` - EXEC_STARTED <br></br> `5` - ALL_DONE <br></br> `100` - UPDATED                                                                         |
| 431      | ListOrderStatus              | INT          | Y        | 可能的值: <br></br> `3` - EXECUTING <br></br> `6` - ALL_DONE  <br></br> `7` - REJECT                                                                            |
| 1386     | ListRejectReason             | INT          | N        | 可能的值: <br></br> `99` - OTHER                                                                                                                      |
| 103      | OrdRejReason                 | INT          | N        | 可能的值: <br></br> `99` - OTHER                                                                                                                      |
| 60       | TransactTime                 | UTCTIMESTAMP | N        | 发生此事件时的时间戳。              |
| 25016    | ErrorCode                    | INT          | N        | API 错误代码 (参考 [错误代码](errors_CN.md) )。 |
| 58       | Text                         | STRING       | N        | 可读的错误消息。   |
| 73       | NoOrders                     | NUMINGROUP   | N        | `Orders` 数组中的元素个数。     |
| =>55     | Symbol                       | STRING       | Y        | 订单的交易对。   |
| =>37     | OrderID                      | INT          | Y        | 由交易所分配的订单 `OrderID`。  |
| =>11     | ClOrdID                      | STRING       | Y        | 分配给请求的订单 `ClOrdID`。 |
| =>25010  | NoListTriggeringInstructions | NUMINGROUP   | N        | `ListTriggeringInstructions` 数组中的元素个数。|
| ==>25011 | ListTriggerType              | CHAR         | N        | 可能的值: <br></br> `1` - ACTIVATED <br></br> `2` - PARTIALLY_FILLED <br></br> `3` - FILLED                                                                     |
| ==>25012 | ListTriggerTriggerIndex      | INT          | N        |    |
| ==>25013 | ListTriggerAction            | CHAR         | N        | 可能的值 <br></br> `1` - RELEASE <br></br> `2` - CANCEL    |


**示例消息：**

```
8=FIX.4.4|9=293|35=N|34=2|49=SPOT|52=20240607-02:19:07.837191|56=Eg13pOvN|55=BTCUSDT|60=20240607-02:19:07.836000|66=25|73=2|55=BTCUSDT|37=52|11=w1717726747805308656|55=BTCUSDT|37=53|11=p1717726747805308656|25010=1|25011=3|25012=0|25013=1|429=4|431=3|1385=2|25014=1717726747805308656|25015=1717726747805308656|10=162|
```

<a id="orderamendkeeppriorityrequest"></a>

#### OrderAmendKeepPriorityRequest<code>&lt;XAK&gt;</code>

由客户发送以减少其订单的原始数量。

这个请求将添加0个订单到 `EXCHANGE_MAX_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器中。

**未成交的订单计数:** 0

**注意：**

* `ClOrdID(11)` 不需要与原订单的 `ClOrdID` 不同。当请求的 `ClOrdID` 与待修改订单的 `ClOrdID` 相同时，`ClOrdID` 将保持不变。
* 当同时提供 `OrderID (37)` 和 `OrigClOrdID (41)` 两个参数时，系统首先将会使用 `OrderID` 来搜索订单。然后， 查找结果中的 `OrigClOrdID (41)` 的值将会被用来验证订单。如果两个条件都不满足，则请求将被拒绝。



| Tag | 名称     | 类型   | 是否必须 | 描述|
| :---- | :---- | :---- | :---- | ----- |
| 11 | ClOrdID | STRING | Y | 分配给请求的订单 `ClOrdID`。  |
| 41 | OrigClOrdID | STRING | N | 待修改订单的 `ClOrdID (11)`。 需提供 `OrigClOrdID (41)` 或 `OrderId (37)`。 |
| 37 | OrderID | INT | N | 待修改订单的 `OrderID (37)`。 需提供 `OrigClOrdID (41)` 或 `OrderId (37)`。 |
| 55 | Symbol  | STRING | Y | 待修改订单的交易对。|
| 38 | OrderQty | QTY | N | 交易的新数量。 必须比订单的原始 OrderQty 小。 |


**示例消息：**

```
8=FIX.4.4|9=103|35=XAK|34=2|49=EXAMPLE|52=20250319-12:35:21.087|56=SPOT|11=O2EIAS01742387721086|37=0|38=0.9|55=BTCUSDT|10=254|
```

**响应：**

* [Reject `<3>`](#reject) 请求会由于缺少必填字段，无效字段，引用无效的符号，或超过消息限制而成为无效请求。
* [OrderAmendReject `<XAR>`](#orderamendreject) 如果请求失败，则由于订单速率限制不足，指向不存在的订单，数量无效等。
* [ExecutionReport `<8>`](#executionReport) 请求成功修改了单个订单。
* [ExecutionReport `<8>`](#executionReport) \+ [ListStatus `<N>`](#liststatus) 请求成功修改了隶属于订单列表中的单个订单。

<a id="orderamendreject"></a>

### OrderAmendReject<code>&lt;XAR&gt;</code>

当 OrderAmendKeepPriorityRequest `<XAK>` 请求失败时，由服务器发送。

| Tag | 名称     | 类型   | 是否必须 | 描述|
| :---- | :---- | :---- | :---- | :---- |
| 11 | ClOrdID | STRING | Y | 待修改订单的 `ClOrdId`。|
| 41 | OrigClOrdID | STRING | N | 待修改订单的 `OrigClOrdId (41)`。|
| 37 | OrderID | INT | N | 待修改订单的 `OrderId (37)`。 |
| 55 | Symbol | STRING | Y | 待修改订单的 `Symbol (55)`。 |
| 38 | OrderQty | QTY | Y |  |
| 25016 | ErrorCode | INT | Y | API 错误代码 (参考 [错误代码](errors_CN.md) )。 |
| 58 | Text | STRING | Y | 人类可读的错误消息。 |


**示例消息：**

```
8=FIX.4.4|9=0000176|35=XAR|49=SPOT|56=OE|34=2|52=20250319-14:27:32.751074|11=1WRGW5J1742394452749|37=0|55=BTCUSDT|38=1.000000|25016=-2038|58=The requested action would change no state; rejecting.|10=235|
```

### Limit Messages

<a id="limitquery"></a>

#### LimitQuery<code>&lt;XLQ&gt;</code>

由客户端发送，用于查询当前限制。

| Tag | 名称     | 类型   | 是否必须 | 描述        |
|------|-------|--------|----------|--------------------|
| 6136 | ReqID | STRING | Y        | 此请求的 ID |

**示例消息：**

```
8=FIX.4.4|9=82|35=XLQ|34=2|49=7buKHZxZ|52=20240614-05:35:35.357|56=SPOT|6136=1718343335357229749|10=170|
```

<a id="limitresponse"></a>

#### LimitResponse<code>&lt;XLR&gt;</code>

由服务器发送，以响应 [LimitQuery`<XLQ>`](#limitquery) 请求。

| Tag | 名称     | 类型   | 是否必须 | 描述|
|------|-------|--------|----------|--------------------|
| 6136    | ReqID                        | STRING     | Y        | `ReqID`。        |
| 25003   | NoLimitIndicators            | NUMINGROUP | Y        | `LimitIndicator` 数组中的元素个数。  |
| =>25004 | LimitType                    | CHAR       | Y        | 可能的值 <br></br> `1` - ORDER_LIMIT <br></br> `2` - MESSAGE_LIMIT   |
| =>25005 | LimitCount                   | INT        | Y        | 此限制的当前使用情况。 |
| =>25006 | LimitMax                     | INT        | Y        | 此限制允许的最大值。        |
| =>25007 | LimitResetInterval           | INT        | N        | 限制重置的频率。       |
| =>25008 | LimitResetIntervalResolution | CHAR       | N        | `LimitResetInterval` 的时间单位。可能的值： <br></br> `s` - SECOND <br></br> `m` - MINUTE <br></br> `h` - HOUR <br></br> `d` - DAY |

**示例消息：**

```
8=FIX.4.4|9=225|35=XLR|34=2|49=SPOT|52=20240614-05:42:42.724057|56=uGnG0ef8|6136=1718343762723730315|25003=3|25004=2|25005=1|25006=1000|25007=10|25008=s|25004=1|25005=0|25006=200|25007=10|25008=s|25004=1|25005=0|25006=200000|25007=1|25008=d|10=241|
```

### 市场数据流

> [!NOTE]
> 以下消息只能用于 FIX 市场数据流。

<a id="instrumentlistrequest"></a>

#### InstrumentListRequest<code>&lt;x&gt;</code>

由客户端发送，用于查询有关交易对的信息。

| Tag | Name                      | Type   | Required | Description                                                                        |
|-----|---------------------------|--------|----------|------------------------------------------------------------------------------------|
| 320 | InstrumentReqID           | STRING | Y        | 此请求的 ID                                                               |
| 559 | InstrumentListRequestType | INT    | Y        | 可能的值： <br></br> `0` - SINGLE_INSTRUMENT <br></br> `4` - ALL_INSTRUMENTS |
| 55  | Symbol                    | STRING | N        | 当 `InstrumentListRequestType` 设置为 `SINGLE_INSTRUMENT (0)` 时是必需的    |

**示例消息:**

```
8=FIX.4.4|9=92|35=x|49=BMDWATCH|56=SPOT|34=2|52=20250114-08:46:56.096691|320=BTCUSDT_INFO|559=0|55=BTCUSDT|10=164|
```

<a id="instrumentlist"></a>

#### InstrumentList<code>&lt;y&gt;</code>

由服务器在响应 [InstrumentListRequest`<x>`](#instrumentlistrequest) 时发送。

> [!NOTE]
> 有关交易对（例如，过滤器）的其他信息可通过 [exchangeInfo]https://github.com/binance/binance-spot-api-docs/blob/master/rest-api_CN.md#exchangeInfo） 请求来获得。

| Tag     | Name                  | Type       | Required | Description                                |
|---------|-----------------------|------------|----------|--------------------------------------------|
| 320     | InstrumentReqID       | STRING     | Y        | `InstrumentReqID` 从请求中得到               |
| 146     | NoRelatedSym          | NUMINGROUP | Y        | 交易数量                                 |
| =>55    | Symbol                | STRING     | Y        | 交易对                                        |
| =>15    | Currency              | STRING     | Y        | 此交易品种的报价资产                |
| 146     | NoRelatedSym          | NUMINGROUP | Y        | 交易品种数量                                 |
| =>55    | Symbol                | STRING     | Y        |                                            |
| =>15    | Currency              | STRING     | Y        | 此交易品种的 Quote asset                |
| =>562   | MinTradeVol           | QTY        | N        | 对应于 [LOT_SIZE](filters_CN.md#lot_size) 过滤器               |
| =>1140  | MaxTradeVol           | QTY        | N        | 对应于 [LOT_SIZE](filters_CN.md#lot_size) 过滤器                  |
| =>25039 | MinQtyIncrement       | QTY        | N        | 对应于 [LOT_SIZE](filters_CN.md#lot_size) 过滤器                |
| =>25040 | MarketMinTradeVol     | QTY        | N       | 对应于 [MARKET_LOT_SIZE](filters_CN.md#market_lot_size) 过滤器 |
| =>25041 | MarketMaxTradeVol     | QTY        | N        | 对应于 [MARKET_LOT_SIZE](filters_CN.md#market_lot_size) 过滤器  |
| =>25042 | MarketMinQtyIncrement | QTY        | N        | 对应于 [MARKET_LOT_SIZE](filters_CN.md#market_lot_size) 过滤器 |
| =>969   | MinPriceIncrement     | PRICE      | N        | 对应于 [PRICE](filters_CN.md#price) 过滤器|                |
| =>2551  | StartPriceRange       | PRICE      | N        | 对应于 [PRICE](filters_CN.md#price) 过滤器|
| =>2552  | EndPriceRange         | PRICE      | N        | 对应于 [PRICE](filters_CN.md#price) 过滤器|

**示例消息:**

```
8=FIX.4.4|9=218|35=y|49=SPOT|56=BMDWATCH|34=2|52=20250114-08:46:56.100147|320=BTCUSDT_INFO|146=1|55=BTCUSDT|15=USDT|562=0.00001000|1140=9000.00000000|25039=0.00001000|25040=0.00000001|25041=76.79001236|25042=0.00000001|969=0.01000000|10=093||
```

<a id="marketdatarequest"></a>

#### MarketDataRequest<code>&lt;V&gt;</code>

由客户发送，用于订阅或取消订阅市场数据流。

<a id="tradestream"></a>

**交易数据流**

交易数据流推送原始交易信息;每笔交易都有唯一的买家和卖家。

**订阅所需的字段：**

- `SubscriptionRequestType` 的值为 `SUBSCRIBE(1)`
- `MDEntryType` 的值为 `TRADE(2)`

**更新速度：** 实时

<a id="symbolbooktickerstream"></a>

**book ticker数据流**

将指定交易对最佳买价，卖价以及数量的更新进行实时推送。

**订阅所需的字段**

- `SubscriptionRequestType` 值为 `SUBSCRIBE(1)`
- `MDEntryType` 值为 `BID (0)`
- `MDEntryType` 值为 `OFFER(1)`
- `MarketDepth` 值为 `1`
\
**Individual Symbol Book Ticker Stream**

将指定交易品种的任何更新实时推送到最佳买价或卖价或数量。

**订阅所需的字段**

- 值为 `SUBSCRIBE(1)` 的 `SubscriptionRequestType`
- 值为 `BID (0)` 的 `MDEntryType`
- 值为 `OFFER(1)` 的 `MDEntryType`
- 值为 `1` 的 `MarketDepth`

**更新速度：** 实时

> [!NOTE]
> 在 [单个交易对订单簿数据流](#symbolbooktickerstream) 中，在[MarketDataIncrementalRefresh`<X>`](#marketdataincrementalrefresh) 消息里 `MDUpdateAction` 取代了之前的 Best Quote，

<a id="diffdepthstream"></a>

**增量深度数据流**

用于本地订单簿管理的价格和数量深度更新。

**订阅所需的字段**

- `SubscriptionRequestType` 值为 `SUBSCRIBE(1)`
- `MDEntryType` 值为 `BID (0)`
- `MDEntryType` 值为 `OFFER (1)`
- `MarketDepth` 值介于 `2` 和 `5000` 之间，用于控制初始快照的大小，对后续的 [MarketDataIncrementalRefresh `<X>` ](#marketdataincrementalrefresh) 消息没有影响

**更新速度：** 100 毫秒

> [!NOTE]
> 由于 [MarketDataSnapshot`<W>`](#marketdatasnapshot) 对价格的深度有限制，所在初始快照之外没有数量变化的价格不会在增量深度信息数据流中。<br></br>因此，即使更新所有推送的数据流信息，依然会导致本地订单簿与真实订单簿有些微差异。<br></br>对于大多数情况，5000 的深度限制就足以理解市场和进行有效的交易。

| Tag   | Name                    | Type       | Required | Description                                                                                                                       |
|:------|-------------------------|------------|----------|-----------------------------------------------------------------------------------------------------------------------------------|
| 262   | MDReqID                 | STRING     | Y        | 此请求的 ID                                                                                                              |
| 263   | SubscriptionRequestType | CHAR       | Y        | 订阅请求类型。 可能的值： <br></br> `1` - SUBSCRIBE <br></br> `2` - UNSUBSCRIBE                             |
| 264   | MarketDepth             | INT        | N        |订阅深度。<br></br>可能的值： <br></br> `1` - Book Ticker 订阅 <br></br> `2`-`5000` - 增量深度流 |
| 266   | AggregatedBook          | NUMINGROUP | N        | 可能的值： <br></br> `Y` - 每个价格每侧 1 个帐簿条目                                                             |
| 146   | NoRelatedSym            | NUMINGROUP | N        | 交易对数量                                                                                                                 |
| =>55  | Symbol                  | STRING     | Y        |                                                                                                                                   |
| 267   | NoMDEntryTypes          | NUMINGROUP | N        | 条目类型数量                                                                                                            |
| =>269 | MDEntryType             | CHAR       | Y        | 可使用的值： <br></br> `0` - BID <br></br> `1` - OFFER <br></br> `2` - TRADE|

**示例消息:**

```
# Subscriptions
# BOOK TICKER Stream
8=FIX.4.4|9=132|35=V|49=TRADER1|56=SPOT|34=4|52=20241122-06:17:14.183428|262=BOOK_TICKER_STREAM|263=1|264=1|266=Y|146=1|55=BTCUSDT|267=2|269=0|269=1|10=010|
# DEPTH Stream
8=FIX.4.4|9=127|35=V|49=TRADER1|56=SPOT|34=7|52=20241122-06:17:14.443822|262=DEPTH_STREAM|263=1|264=10|266=Y|146=1|55=BTCUSDT|267=2|269=0|269=1|10=111|
# TRADE Stream
8=FIX.4.4|9=120|35=V|49=TRADER1|56=SPOT|34=3|52=20241122-06:34:14.775606|262=TRADE_STREAM|263=1|264=1|266=Y|146=1|55=BTCUSDT|267=1|269=2|10=040|
# Unsubscription from TRADE Stream
8=FIX.4.4|9=79|35=V|49=TRADER1|56=SPOT|34=7|52=20241122-06:41:56.966969|262=TRADE_STREAM|263=2|264=1|10=113|
```

<a id="marketdatarequestreject"></a>

### MarketDataRequestReject<code>&lt;Y&gt;</code>

服务器对不正确的 MarketDataRequest 的响应消息 `<V>`.

| Tag   | Name           | Type   | Required | Description                                                                               |
|-------|----------------|--------|----------|-------------------------------------------------------------------------------------------|
| 262   | MDReqID        | STRING | Y        | 不正确的ID [MarketDataRequest`<V>`](#marketdatarequest)                            |
| 281   | MDReqRejReason | CHAR   | N        | 可能的错误原因: <br></br> `1` - DUPLICATE_MDREQID <br></br> `2` - TOO_MANY_SUBSCRIPTIONS |
| 25016 | ErrorCode      | INT    | N        | API 错误代码 [参见 错误代码](errors.md)。  |
| 58    | Text           | STRING | N        | 错误信息。                                                             |


**示例消息:**

```
8=FIX.4.4|9=0000218|35=Y|49=SPOT|56=EXAMPLE|34=5|52=20241019-05:39:36.688964|262=BOOK_TICKER_2|281=2|25016=-1191|58=Similar subscription is already active on this connection. Symbol='BNBBUSD', active subscription id: 'BOOK_TICKER_1'.|10=137|
```

<a id="marketdatasnapshot"></a>

### MarketDataSnapshot<code>&lt;W&gt;</code>

服务器对 [MarketDataRequest`<V>`](#marketdatarequest)，激活 [单个 Symbol Book Ticker Stream](#symbolbooktickerstream) 或 [增量深度流](#diffdepthstream) 订阅的响应消息。

| Tag   | Name             | Type       | Required | Description                                                                             |
|-------|------------------|------------|----------|-----------------------------------------------------------------------------------------|
| 262   | MDReqID          | STRING     | Y        | 激活此订阅的 [MarketDataRequest`<V>`](#marketdatarequest) 的 ID|
| 55    | Symbol           | STRING     | Y        |                                                                                         |
| 25044 | LastBookUpdateID | INT        | N        |                                                                                         |
| 268   | NoMDEntries      | NUMINGROUP | Y        | 条目数                                                                       |
| =>269 | MDEntryType      | CHAR       | Y        | 可能的值： <br></br> `0` - BID <br></br> `1` - OFFER <br></br> `2` - TRADE     |
| =>270 | MDEntryPx        | PRICE      | Y        | 价格                                                                                   |
| =>271 | MDEntrySize      | QTY        | Y        | 数量                                                                                |

**示例消息:**

```
8=FIX.4.4|9=0000107|35=W|49=SPOT|56=EXAMPLE|34=34|52=20241019-05:41:52.867164|262=BOOK_TICKER_1_2|55=BNBBUSD|25044=0|268=0|10=151|
```

<a id="marketdataincrementalrefresh"></a>

### MarketDataIncrementalRefresh<code>&lt;X&gt;</code>

当一个订阅的数据流发生变化时，服务器发出的响应消息。

| Tag     | Name              | Type         | Required | Description                                                                                                                                      |
|---------|-------------------|--------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| 262     | MDReqID           | STRING       | Y        | 激活此订阅的 [MarketDataRequest `<V>`](#marketdatarequest) 的 ID                                                          |
| 893     | LastFragment      | BOOLEAN      | N        | 当该字段出现时，表示消息被分片。分片将会发生在单个 [MarketDataIncrementalRefresh`<X>`](#marketdataincrementalrefresh) 中 `NoMDEntry` 超过 10000 时，为了将其限制在 10000 以内。分片消息的各个片段在数据流中保证是连续的。该字段仅出现在 [交易数据流](#tradestream) 和 [增量深度数据流](#diffdepthstream) 中。   |
| 268     | NoMDEntries       | NUMINGROUP   | Y        | 条目数                                                                                                                               |
| =>279   | MDUpdateAction    | CHAR         | Y        | 可能的值： <br></br> `0` - NEW <br></br> `1` - CHANGE <br></br> `2` - DELETE                                                           |
| =>270   | MDEntryPx         | PRICE        | Y        | 价格                                                                                                                                           |
| =>271   | MDEntrySize       | QTY          | N        | 数量                                                                                                                                         |
| =>269   | MDEntryType       | CHAR         | Y        | 可能的值： <br></br> `0` - BID <br></br> `1` - OFFER <br></br> `2` - TRADE                                                                  |
| =>55    | Symbol            | STRING       | N        | 如果未指定`交易对`，则默认使用同一市场数据流消息中前一个消息的交易对 |
| =>60    | TransactTime      | UTCTIMESTAMP | N        |                                                                                                                                                  |
| =>1003  | TradeID           | INT          | N        |                                                                                                                                                  |
| =>2446  | AggressorSide     | CHAR         | N        | 可能的值： <br></br> `1` - BUY <br></br> `2` - SELL |
| =>25043 | FirstBookUpdateID | INT          | N        |   只会在 [增量深度数据流](#diffdepthstream) 中出现. <br></br> 如果 `LastBookUpdateID` 没有指定，市场数据消息的 `FirstBookUpdateID` 将会默认设置为与之前消息一样的 `FirstBookUpdateID`  |
| =>25044 | LastBookUpdateID  | INT          | N        |   只会在 [增量深度数据流](#diffdepthstream) 和 [单个交易对订单簿数据流](#symbolbooktickerstream) 中出现. <br></br> 如果 `LastBookUpdateID` 没有指定，市场数据消息的 `LastBookUpdateID` 将会默认设置为与之前消息一样的 `LastBookUpdateID`  |


**示例消息:**

```
8=FIX.4.4|9=0000313|35=X|49=SPOT|56=EXAMPLE|34=16|52=20241019-05:40:11.466313|262=TRADE_3|893=N|268=3|279=0|269=2|270=10.00000|271=0.01000|55=BNBBUSD|1003=0|60=20241019-05:40:11.464000|279=0|269=2|270=10.00000|271=0.01000|1003=1|60=20241019-05:40:11.464000|279=0|269=2|270=10.00000|271=0.01000|1003=2|60=20241019-05:40:11.464000|10=125|
```

**示例分片消息:**

> [!NOTE]
> 以下是示例消息，其中 `NoMDEntry` 限制为 *2*，在实际流中，`NoMDEntry` 限制为 *10000*。

[Trade Stream](#tradestream)
```
8=FIX.4.4|9=237|35=X|34=114|49=SPOT|52=20250116-19:36:44.544549|56=EXAMPLE|262=id|268=2|279=0|270=240.00|271=3.00000000|269=2|55=BNBBUSD|60=20250116-19:36:44.196569|1003=67|279=0|270=238.00|271=2.00000000|269=2|60=20250116-19:36:44.196569|1003=68|893=N|10=180|
8=FIX.4.4|9=163|35=X|34=115|49=SPOT|52=20250116-19:36:44.544659|56=EXAMPLE|262=id|268=1|279=0|270=233.00|271=1.00000000|269=2|55=BNBBUSD|60=20250116-19:36:44.196569|1003=69|893=Y|10=243|
```

[Diff. Depth Stream](#diffdepthstream)
```
8=FIX.4.4|9=156|35=X|34=12|49=SPOT|52=20250116-19:45:31.774162|56=EXAMPLE|262=id|268=2|279=2|270=362.00|269=0|55=BNBBUSD|25043=1143|25044=1145|279=2|270=313.00|269=0|893=N|10=047|
8=FIX.4.4|9=171|35=X|34=13|49=SPOT|52=20250116-19:45:31.774263|56=EXAMPLE|262=id|268=2|279=2|270=284.00|269=0|55=BNBBUSD|25043=1143|25044=1145|279=1|270=264.00|271=3.00000000|269=0|893=N|10=239|
8=FIX.4.4|9=149|35=X|34=14|49=SPOT|52=20250116-19:45:31.774281|56=EXAMPLE|262=id|268=1|279=1|270=395.00|271=19.00000000|269=1|55=BNBBUSD|25043=1143|25044=1145|893=Y|10=024|
```

## FIX SBE

FIX SBE（简单二进制编码）可以替代 FIX，请使用 [spot_fix_prod_latest.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_fix_prod_latest.xml) 模式文件。

### SBE

请阅读 [简单二进制编码 （SBE） 常见问题](./faqs/sbe_faq_CN.md) 了解有关将 SBE 与币安 API 配合使用的重要信息。

* 在使用 FIX SBE 之前，请务必阅读并理解 [SBE 规范](https://www.fixtrading.org/standards/sbe-online/)。

* 在对 SBE payload 进行编码和解码时，建议使用 [`SbeTool`](https://github.com/aeron-io/simple-binary-encoding) 所生成的代码，以确保符合FIX SBE 规范。

### 端点

除了在 9000 端口支持的 FIX 编码外，还支持两种请求/响应编码方案，分别在额外的 TCP 端口上。以下是各个 API 的端点说明。

#### 订单录入

* `tcp+tls://fix-oe.binance.com:9001`：发送 FIX 请求；接收 FIX SBE 响应
    * FIX `SbeSchemaId` 标签（=25050）必须设置为 FIX SBE 模式 ID（=1）
    * FIX `SbeSchemaVersion` 标签（=25051）必须设置为 FIX SBE 模式版本（=0）
* `tcp+tls://fix-oe.binance.com:9002`：发送 FIX SBE 请求；接收 FIX SBE 响应

#### Drop Copy（订单副本）

* `tcp+tls://fix-dc.binance.com:9001`：发送 FIX 请求；接收 FIX SBE 响应
    * FIX `SbeSchemaId` 标签（=25050）必须设置为 FIX SBE 模式 ID（=1）
    * FIX `SbeSchemaVersion` 标签（=25051）必须设置为 FIX SBE 模式版本（=0）
* `tcp+tls://fix-dc.binance.com:9002`：发送 FIX SBE 请求；接收 FIX SBE 响应

#### 市场数据

* `tcp+tls://fix-md.binance.com:9001`：发送 FIX 请求；接收 FIX SBE 响应
    * FIX `SbeSchemaId` 标签（=25050）必须设置为 FIX SBE 模式 ID（=1）
    * FIX `SbeSchemaVersion` 标签（=25051）必须设置为 FIX SBE 模式版本（=0）
* `tcp+tls://fix-md.binance.com:9002`：发送 FIX SBE 请求；接收 FIX SBE 响应

### FIX SBE 编码设计

FIX SBE 请求/响应 消息总会包含一个 SOFH (Simple Open Framing Header) 和消息头. 一个 N 字节的FIX SBE 消息包含下面的格式:

`<SOFH (6 字节> <消息头 (20 字节)> <消息 (N 字节)>`

SOFH：有关模式文件中的组合类型 "sofh"。这个字段作为一个帧头用来让FIX SBE 服务器端/客户端 来确定 SBE 消息的长度和保证在反序列化之前已经接收到完整的消息。

注意：
- SOFH 中的两个字段必须是小端序编码（little-endian）
- FIX 服务器仅支持编码类型字段 `0xEB50` , 例如只有小端序（little-endian） 在所有字段被支持

消息头： 这对应于模式文件中的组合类型 "messageHeader".

### Logon

登录签名（RawData）的计算方法如[签名计算](#signaturecomputation)部分所述。

#### FIX SBE `Logon` 请求消息示例

请参阅下方根据上述说明获取的 FIX SBE `Logon` 十六进制消息示例。


| Bytes                                           | Description                 |
|-------------------------------------------------|-----------------------------|
| 0xd1, 0x00, 0x00, 0x00                          | sofh.messageLength          |
| 0x50, 0xeb                                      | sofh.encodingType           |
| 0x0e, 0x00                                      | messageHeader.blockLength   |
| 0x28, 0x4e                                      | messageHeader.templateId    |
| 0x01, 0x00                                      | messageHeader.schemaId      |
| 0x00, 0x00                                      | messageHeader.version       |
| 0x01, 0x00, 0x00, 0x00                          | messageHeader.seqNum        |
| 0x58, 0x7a, 0x5f, 0x99, 0xdb, 0x1b, 0x06, 0x00  | messageHeader.sendingTime   |
| 0x00                                            | Logon.EncryptMethod         |
| 0x1e, 0x00, 0x00, 0x00                          | Logon.HeartBtInt            |
| 0x01                                            | Logon.ResetSeqNumFlag       |
| 0x02                                            | Logon.MessageHandling       |
| 0xff                                            | Logon.ResponseMode          |
| 0xff                                            | Logon.ExecutionReportType   |
| 0xff                                            | Logon.DropCopyFlag          |
| 0xff, 0xff, 0xff, 0xff                          | Logon.RecvWindow            |
| 0x07                                            | Logon.SenderCompId.length   |
| 0x45, 0x58, 0x41, 0x4d, 0x50, 0x4c, 0x45        | Logon.SenderCompId.varData  |
| 0x04                                            | Logon.TargetCompId.length   |
| 0x53, 0x50, 0x4f, 0x54                          | Logon.TargetCompId.varData  |
| 0x58, 0x00                                      | Logon.RawData.length        |
| 0x34, 0x4d, 0x48, 0x58, 0x65, 0x6c, 0x56, 0x56  | Logon.RawData.varData       |
| 0x63, 0x70, 0x6b, 0x64, 0x77, 0x75, 0x4c, 0x62  | Logon.RawData.varData       |
| 0x6c, 0x36, 0x6e, 0x37, 0x33, 0x48, 0x51, 0x55  | Logon.RawData.varData       |
| 0x58, 0x55, 0x66, 0x31, 0x64, 0x73, 0x65, 0x32  | Logon.RawData.varData       |
| 0x50, 0x43, 0x67, 0x54, 0x31, 0x44, 0x59, 0x71  | Logon.RawData.varData       |
| 0x57, 0x39, 0x77, 0x38, 0x41, 0x56, 0x5a, 0x31  | Logon.RawData.varData       |
| 0x52, 0x41, 0x43, 0x46, 0x47, 0x4d, 0x2b, 0x35  | Logon.RawData.varData       |
| 0x55, 0x64, 0x6c, 0x47, 0x50, 0x72, 0x51, 0x48  | Logon.RawData.varData       |
| 0x72, 0x67, 0x74, 0x53, 0x33, 0x43, 0x76, 0x73  | Logon.RawData.varData       |
| 0x52, 0x55, 0x52, 0x43, 0x31, 0x6f, 0x6a, 0x37  | Logon.RawData.varData       |
| 0x33, 0x6a, 0x38, 0x67, 0x43, 0x41, 0x3d, 0x3d  | Logon.RawData.varData       |
| 0x40, 0x00                                      | Logon.Username.length       |
| 0x73, 0x42, 0x52, 0x58, 0x72, 0x4a, 0x78, 0x32  | Logon.Username.varData      |
| 0x44, 0x73, 0x4f, 0x72, 0x61, 0x4d, 0x58, 0x4f  | Logon.Username.varData      |
| 0x61, 0x55, 0x6f, 0x76, 0x45, 0x68, 0x67, 0x56  | Logon.Username.varData      |
| 0x52, 0x63, 0x6a, 0x4f, 0x76, 0x43, 0x74, 0x51  | Logon.Username.varData      |
| 0x77, 0x6e, 0x57, 0x6a, 0x38, 0x56, 0x78, 0x6b  | Logon.Username.varData      |
| 0x4f, 0x68, 0x31, 0x78, 0x71, 0x62, 0x6f, 0x53  | Logon.Username.varData      |
| 0x30, 0x32, 0x53, 0x50, 0x47, 0x66, 0x4b, 0x69  | Logon.Username.varData      |
| 0x32, 0x68, 0x38, 0x73, 0x70, 0x5a, 0x4a, 0x62  | Logon.Username.varData      |


<a id="fix-vs-fix-sbe-schema"></a>
### FIX 与 FIX SBE 模式对比

通用说明：
* `sofh.messageLength` 字段 _必须_ 包含 SOFH 的大小（6 字节）
* FIX SBE 没有 `Checksum` 字段
* 在端口 9002 上发送 FIX SBE 请求时
    * payload 中必须设置所有字段
    * 未设置的可选字段必须设置为相应的 `nullValue`
        * `SbeTool` 生成的编码器可以正确处理这种情况
        * 如果 payload 是手动编码生成的，请参阅 [SBE 规范](https://www.fixtrading.org/standards/sbe-online/) 中有关 `nullValue` 的定义

**Logon（登录）消息：**
* `SenderCompID`、`TargetCompID` 和 `RecvWindow` 字段包含在 `Logon` FIX SBE 消息中，而不是消息头
    * `Logon` 消息中设置的 `RecvWindow` 字段适用于 FIX SBE 会话内的所有交易请求消息。
    * 设置后，`RecvWindow` 字段的单位为微秒。
* 当 `ResponseMode` 字段设置为 `OnlyAcks` 时，`ExecutionReportType` 字段可以设置为 `Mini`，以接收 `ExecutionReportAck` 消息，而非 `ExecutionReport`
    * 注意：只有订单录入和 Drop Copy（订单副本）接口的 9001 端口和 9002 端口会支持 `ExecutionReportType` 字段

**MarketDataIncrementalRefresh（市场数据增量刷新）** 消息：
* FIX 模式中的单条消息被拆分为以下 FIX SBE 消息：`MarketDataIncrementalTrade`、`MarketDataIncrementalBookTicker` 和 `MarketDataIncrementalDepth`
* 市场数据快照和刷新消息中省略了 `MDReqID` 字段，因为这些消息可以通过 `Symbol` 字段和消息的模板 ID 与订阅请求关联起来。
    * `MDReqID` 在 `MarketDataRequest` 消息中是必需的，以便它可以在 `MarketDataRequestReject` 中使用。
    * `MDReqID` 的值在所有订阅中必须是唯一的。

**MarketDataIncrementalTrade（市场数据增量交易）** 消息：
* FIX 模式中可用的 `MDUpdateAction` 字段在 FIX SBE 中被省略，因为其值始终为 `NEW`。

**MarketDataIncrementalBookTicker（市场数据增量订单簿数据流）** 消息：
* FIX SBE 最优挂单信息订阅使用 **自动剔除（auto-culling）**：当系统负载较高时，可能会丢弃过时的事件，而不是将所有事件排队并延迟发送。
    * 例如，如果在时间 T2 生成了一个最优买/卖单报价事件，而此时仍有一个未发送的事件排队在时间 T1（且 T1 < T2），则会丢弃时间 T1 的事件，系统只会发送时间 T2 的事件。此操作是基于每个交易对分别进行的。
* FIX 模式中可用的 `MDUpdateAction` 字段在 FIX SBE 中被省略，因为其值可能源自 `MDEntrySize`。
    * 当 `MDEntrySize` 未设置（`NullVal`）时，`MDUpdateAction` 为 `DELETE`。
    * 当 `MDEntrySize` 已设置时：
        * 如果价格水平已存在于本地订单簿中，则 `MDUpdateAction` 为 `CHANGE`
        * 否则，`MDUpdateAction` 为 `NEW`。

**MarketDataIncrementalDepth（市场数据增量深度）** 消息：
* FIX SBE 深度更新速度：50 毫秒
* FIX 模式中可用的 `MDUpdateAction` 字段在 FIX SBE 中被省略，因为其值可能源自 `MDEntrySize`。
    * 当 `MDEntrySize` 未设置（`NullVal`）时，`MDUpdateAction` 为 `DELETE`。
    * 当 `MDEntrySize` 已设置时：
        * 如果价格水平已存在于本地订单簿中，则 `MDUpdateAction` 为 `CHANGE`
        * 否则，`MDUpdateAction` 为 `NEW`。

### 连接限制

FIX 和 FIX SBE 分享连接限制。

### 错误代码

可能返回以下 FIX SBE 特定错误代码：

| 代码    | 消息                                         | 描述                                                             |
|---------|----------------------------------------------|------------------------------------------------------------------|
| -1152   | 无效的 SBE 消息头。                          | 解码 FIX SBE 请求中的 `messageHeader` 时出错                     |
| -1153   | 指定的 SBE 模式 ID 或版本无效。              | 解析/解码 FIX SBE 模式 ID/版本时出错                             |
| -1177   | 无效的编码类型。                         | 解码 sofh 复合类型中的 `encodingType` 字段时出错                 |
| -1221   | SBE 消息中字段无效或缺失。                   | 解码 FIX SBE 请求时字段无效或缺失                                |

注意：对于语义等效的 FIX 和 FIX SBE 请求，返回的错误代码可能不完全相同。

### 常见问题

有关生成 SBE 解码器和处理模式更新的更多信息，请参见 [SBE FAQ](./faqs/sbe_faq_CN.md)。
