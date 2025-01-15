# REST行情与交易接口 

**最近更新： 2024-12-17**

## API 基本信息
* 本篇列出接口的 base URL 有:
  * **https://api.binance.com**
  * **https://api-gcp.binance.com**
  * **https://api1.binance.com**
  * **https://api2.binance.com**
  * **https://api3.binance.com**
  * **https://api4.binance.com**
* 上述列表的最后4个接口 (`api1`-`api4`) 会提供更好的性能，但其稳定性略为逊色。因此，请务必使用最适合的URL。
* 所有接口的响应都是 JSON 格式。
* 响应中如有数组，数组元素以时间**升序**排列，越早的数据越提前。
* JSON 响应中的所有时间和时间戳相关字段均以**毫秒为默认单位**。要以微秒为单位接收信息，请添加报文头 `X-MBX-TIME-UNIT：MICROSECOND` 或 `X-MBX-TIME-UNIT：microsecond`。 
* 时间戳参数（例如 `startTime`、`endTime`、`timestamp`）可以以毫秒或微秒为单位传递。
* 对于仅发送公开市场数据的 API，您可以使用接口的 base URL https://data-api.binance.vision 。请参考 [Market Data Only_CN](./faqs/market_data_only_CN.md) 页面。

## HTTP 返回代码

* HTTP `4XX` 错误码用于指示错误的请求内容、行为、格式。问题在于请求者。
* HTTP `403` 错误码表示违反WAF限制(Web应用程序防火墙)。
* HTTP `409` 错误码表示重新下单(cancelReplace)的请求部分成功。(比如取消订单失败，但是下单成功了)
* HTTP `429` 错误码表示警告访问频次超限，即将被封IP。
* HTTP `418` 表示收到429后继续访问，于是被封了。
* HTTP `5XX` 错误码用于指示Binance服务侧的问题。


## 接口错误代码

* 每个接口都有可能抛出异常，异常响应格式如下：

```javascript
{
  "code": -1121,
  "msg": "Invalid symbol."
}
```

* 具体的错误码及其解释在[错误代码汇总](./errors_CN.md)

## 接口的基本信息
* `GET` 方法的接口, 参数必须在 `query string`中发送。
* `POST`, `PUT`, 和 `DELETE` 方法的接口,参数可以在内容形式为`application/x-www-form-urlencoded`的 `query string` 中发送，也可以在 `request body` 中发送。 如果你喜欢，也可以混合这两种方式发送参数。
* 对参数的顺序不做要求。
* 但如果同一个参数名在`query string`和`request body`中都有，`query string`中的会被优先采用。

## 访问限制
### 访问限制基本信息
* 以下是 `intervalLetter` 作为头部值:
  * SECOND => S
  * MINUTE => M
  * HOUR => H
  * DAY => D
* 在 `/api/v3/exchangeInfo`接口中`rateLimits`数组里包含有REST接口(不限于本篇的REST接口)的访问限制。包括带权重的访问频次限制、下单速率限制。参考 [枚举定义](./enums_CN.md) 中有关有限制类型的进一步说明。
* 当您超出请求速率限制时，请求会失败并返回 HTTP 状态代码 429。

### IP 访问限制
* 每个请求将包含一个`X-MBX-USED-WEIGHT-(intervalNum)(intervalLetter)`的头，其中包含当前IP所有请求的已使用权重。
* 每一个接口均有一个相应的权重(weight)，有的接口根据参数不同可能拥有不同的权重。越消耗资源的接口权重就会越大。
* 收到429时，您有责任停止发送请求，不得滥用API。
* **收到429后仍然继续违反访问限制，会被封禁IP，并收到418错误码**
* 频繁违反限制，封禁时间会逐渐延长，**从最短2分钟到最长3天**.
* `Retry-After`的头会与带有418或429的响应发送，并且会给出**以秒为单位**的等待时长(如果是429)以防止禁令，或者如果是418，直到禁令结束。
* **访问限制是基于IP的，而不是API Key**

<a id="unfilled-order-count"></a>

### 未成交订单计数
* 每个成功的订单响应都将包含一个 `X-MBX-ORDER-COUNT-(intervalNum)(intervalLetter)` 报文头，用于标识您在该时间间隔内下了多少订单。<br></br>如果您想要对此进行监控，请参阅 [`GET api/v3/rateLimit/order`](#query-unfilled-order-count)。
* 被拒绝/不成功的订单不保证在响应中有 `X-MBX-ORDER-COUNT-**` 报文头。
* 如果超过此值，您将收到一个 429 错误，而且不带 `Retry-After` 报文头。
* **请注意，如果您的订单一直顺利完成交易，您可以通过 API 持续下订单**。更多信息，请参见[现货未成交订单计数规则](./faqs/order_count_decrement_CN.md)。
* **未成交订单数量是按照每个账户来统计的**。

## 数据来源
* 因为API系统是异步的, 所以返回的数据有延时很正常, 也在预期之中。
* 在每个接口中，都列出了其数据的来源，可以用于理解数据的时效性。

系统一共有3个数据来源，按照更新速度的先后排序。排在前面的数据最新，在后面就有可能存在延迟。
  * **撮合引擎** - 表示数据来源于撮合引擎
  * **缓存** - 表示数据来源于内部或者外部的缓存
  * **数据库** - 表示数据直接来源于数据库

有些接口有不止一个数据源, 比如 `缓存 => 数据库`, 这表示接口会先从第一个数据源检查，如果没有数据，则检查下一个数据源。

## 接口鉴权类型
* 每个接口都有自己的鉴权类型，鉴权类型决定了访问时应当进行何种鉴权。
* 鉴权类型会在本文档中各个接口名称旁声明，如果没有特殊声明即默认为 `NONE`。
* 如果需要 API-keys，应当在HTTP头中以 `X-MBX-APIKEY`字段传递。
* API-keys 与 secret-keys **是大小写敏感的**。
* API-keys可以被配置为只拥有访问一些接口的权限。
 <br>例如, 一个 API-key 仅可用于发送交易指令, 而另一个 API-key 则可访问除交易指令外的所有路径。
* 默认 API-keys 可访问所有鉴权路径.

鉴权类型 | 描述
------------ | ------------
NONE | 不需要鉴权的接口
TRADE | 需要有效的API-KEY和签名
USER_DATA | 需要有效的API-KEY和签名
USER_STREAM | 需要有效的API-KEY
MARKET_DATA | 需要有效的API-KEY

* `TRADE` 和 `USER_DATA` 接口是 签名(SIGNED)接口.

### 需要签名的接口 (TRADE 与 USER_DATA)
* 调用`SIGNED` 接口时，除了接口本身所需的参数外，还需要在`query string` 或 `request body`中传递 `signature`, 即签名参数。
* `签名` **大小写不敏感**.
* 请参考下面 [签名示例](#post-apiv3order-%E7%9A%84%E7%A4%BA%E4%BE%8B) 以了解具体如何做计算签名。

### 时间同步安全
* 签名接口均需要传递 `timestamp`参数，其值应当是请求发送时刻的unix时间戳(毫秒)。
* 服务器收到请求时会判断请求中的时间戳，如果是5000毫秒之前发出的，则请求会被认为无效。这个时间空窗值可以通过发送可选参数 `recvWindow`来定义。
* 逻辑伪代码：

  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```

**关于交易时效性**
互联网状况并不100%可靠，不可完全依赖,因此你的程序本地到币安服务器的时延会有抖动.
这是我们设置`recvWindow`的目的所在，如果你从事高频交易，对交易时效性有较高的要求，可以灵活设置`recvWindow`以达到你的要求。
**不推荐使用5秒以上的recvWindow。最大值不能超过60秒！**


### POST /api/v3/order 的示例

#### HMAC Keys
以下是在linux bash环境下使用 `echo`,`openssl`和`curl`工具实现的一个调用接口下单的示例
apikey、secret仅供示范

Key | Value
------------ | ------------
`apiKey` | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
`secretKey` | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


参数 | 取值
------------ | ------------
`symbol` | LTCBTC
`side` | BUY
`type` | LIMIT
`timeInForce` | GTC
`quantity` | 1
`price` | 0.1
`recvWindow` | 5000
`timestamp` | 1499827319559


**示例 1: 所有参数通过 query string 发送**
* **queryString:** symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 签名:**

    ```
    [linux]$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```


* **curl 调用:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.binance.com/api/v3/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

**示例 2: 所有参数通过 request body 发送**
* **requestBody:** symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 签名:**

    ```
    [linux]$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```


* **curl 调用:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.binance.com/api/v3/order' -d 'symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

**示例 3: 混合使用 query string 与 request body**
* **queryString:** symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC
* **requestBody:** quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 签名:**

    ```
    [linux]$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTCquantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= 0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77
    ```


* **curl 调用:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.binance.com/api/v3/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC' -d 'quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77'
    ```

Note that the signature is different in example 3.
There is no & between "GTC" and "quantity=1".


#### RSA Keys

* 这将逐步介绍如何通过有效的签名发送 payload。
* 我们接受`PKCS#8`格式的RSA密钥。
* 要获取 API Key，您需要在您的账户上上传您的 RSA Public Key。
* 对于这个例子，Private Key 将被引用为 `test-prv-key.pem`。

Key | Value
------------ | ------------
`apiKey` | CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ

参数 | 取值
------------ | ------------
`symbol` | BTCUSDT
`side` | SELL
`type` | LIMIT
`timeInForce` | GTC
`quantity` | 1
`price` | 0.2
`timestamp` | 1668481559918
`recvWindow` | 5000


**第一步: Payload**

将参数列表排列成一个 string。 用 `&` 分隔每个参数。对于上述参数，签名 payload 如下所示：

```console
symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000
```

**第二步: 计算签名**

1. 将签名有效负载编码为 ASCII 数据。
2. 使用带有 SHA-256 hash 函数的 RSASSA-PKCS1-v1_5 算法对 payload 进行签名。

```console
$ echo -n 'symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000' | openssl dgst -sha256 -sign ./test-prv-key.pem
```
3. 将输出编码为 base64 string。

```console
$ echo -n 'symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000' | openssl dgst -sha256 -sign ./test-prv-key.pem | openssl enc -base64 -A
HZ8HOjiJ1s/igS9JA+n7+7Ti/ihtkRF5BIWcPIEluJP6tlbFM/Bf44LfZka/iemtahZAZzcO9TnI5uaXh3++lrqtNonCwp6/245UFWkiW1elpgtVAmJPbogcAv6rSlokztAfWk296ZJXzRDYAtzGH0gq7CgSJKfH+XxaCmR0WcvlKjNQnp12/eKXJYO4tDap8UCBLuyxDnR7oJKLHQHJLP0r0EAVOOSIbrFang/1WOq+Jaq4Efc4XpnTgnwlBbWTmhWDR1pvS9iVEzcSYLHT/fNnMRxFc7u+j3qI//5yuGuu14KR0MuQKKCSpViieD+fIti46sxPTsjSemoUKp0oXA==
```

4. 由于签名可能包含 `/` 和 `=`，这可能会导致发送请求时出现问题。 所以签名必须是 URL 编码的。

```console
HZ8HOjiJ1s%2FigS9JA%2Bn7%2B7Ti%2FihtkRF5BIWcPIEluJP6tlbFM%2FBf44LfZka%2FiemtahZAZzcO9TnI5uaXh3%2B%2BlrqtNonCwp6%2F245UFWkiW1elpgtVAmJPbogcAv6rSlokztAfWk296ZJXzRDYAtzGH0gq7CgSJKfH%2BXxaCmR0WcvlKjNQnp12%2FeKXJYO4tDap8UCBLuyxDnR7oJKLHQHJLP0r0EAVOOSIbrFang%2F1WOq%2BJaq4Efc4XpnTgnwlBbWTmhWDR1pvS9iVEzcSYLHT%2FfNnMRxFc7u%2Bj3qI%2F%2F5yuGuu14KR0MuQKKCSpViieD%2BfIti46sxPTsjSemoUKp0oXA%3D%3D
```

5. curl 命令:

```console
curl -H "X-MBX-APIKEY: CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ" -X POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000&signature=HZ8HOjiJ1s%2FigS9JA%2Bn7%2B7Ti%2FihtkRF5BIWcPIEluJP6tlbFM%2FBf44LfZka%2FiemtahZAZzcO9TnI5uaXh3%2B%2BlrqtNonCwp6%2F245UFWkiW1elpgtVAmJPbogcAv6rSlokztAfWk296ZJXzRDYAtzGH0gq7CgSJKfH%2BXxaCmR0WcvlKjNQnp12%2FeKXJYO4tDap8UCBLuyxDnR7oJKLHQHJLP0r0EAVOOSIbrFang%2F1WOq%2BJaq4Efc4XpnTgnwlBbWTmhWDR1pvS9iVEzcSYLHT%2FfNnMRxFc7u%2Bj3qI%2F%2F5yuGuu14KR0MuQKKCSpViieD%2BfIti46sxPTsjSemoUKp0oXA%3D%3D'
```

下面的示例 Bash 脚本执行上述类似的步骤：

```bash
#!/usr/bin/env bash
# 设置身份验证：
API_KEY="替换成您的 API Key"
PRIVATE_KEY_PATH="test-prv-key.pem"
# 设置您的请求:
API_METHOD="POST"
API_CALL="api/v3/order"
API_PARAMS="symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2"
# 计算签名：
timestamp=$(date +%s000)
api_params_with_timestamp="$API_PARAMS&timestamp=$timestamp"
signature=$(echo -n "$api_params_with_timestamp" \
            | openssl dgst -sha256 -sign "$PRIVATE_KEY_PATH" \
            | openssl enc -base64 -A)
# 发送请求：
curl -H "X-MBX-APIKEY: $API_KEY" -X "$API_METHOD" \
    "https://api.binance.com/$API_CALL?$api_params_with_timestamp" \
    --data-urlencode "signature=$signature"
```

#### Ed25519 Keys

**我们建议使用 Ed25519 API keys**，因为它在所有受支持的 API key 类型中提供最佳性能和安全性。

参数           | 取值
------------  | ------------
`symbol`      | BTCUSDT
`side`        | SELL
`type`        | LIMIT
`timeInForce` | GTC
`quantity`    | 1
`price`       | 0.2
`timestamp`   | 1668481559918

下面的 Python 示例代码能说明如何使用 Ed25519 key 对 payload 进行签名。

```python
#!/usr/bin/env python3
import base64
import requests
import time
from cryptography.hazmat.primitives.serialization import load_pem_private_key

# 设置身份验证：
API_KEY='替换成您的 API Key'
PRIVATE_KEY_PATH='test-prv-key.pem'
# 加载 private key。
# 在这个例子中，private key 没有加密，但我们建议使用强密码以提高安全性。
with open(PRIVATE_KEY_PATH, 'rb') as f:
    private_key = load_pem_private_key(data=f.read(), password=None)
# 设置请求参数：
params = {
    'symbol':       'BTCUSDT',
    'side':         'SELL',
    'type':         'LIMIT',
    'timeInForce':  'GTC',
    'quantity':     '1.0000000',
    'price':        '0.20',
}
# 参数中加时间戳：
timestamp = int(time.time() * 1000) # 以毫秒为单位的 UNIX 时间戳
params['timestamp'] = timestamp
# 参数中加签名：
payload = '&'.join([f'{param}={value}' for param, value in params.items()])
signature = base64.b64encode(private_key.sign(payload.encode('ASCII')))
params['signature'] = signature
# 发送请求：
headers = {
    'X-MBX-APIKEY': API_KEY,
}
response = requests.post(
    'https://api.binance.com/api/v3/order',
    headers=headers,
    data=params,
)
print(response.json())
```

# 公开API接口

## 通用接口

### 术语解释
这里的术语适用于全部文档，建议特别是新手熟读，也便于理解。

* `base asset` 指一个交易对的交易对象，即写在靠前部分的资产名, 比如`BTCUSDT`, `BTC`是`base asset`。
* `quote asset` 指一个交易对的定价资产，即写在靠后部分的资产名, 比如`BTCUSDT`, `USDT`是`quote asset`。

<a id="ping"></a>

### 测试服务器连通性 PING
```
GET /api/v3/ping
```
测试能否联通

**权重:**
1

**参数:**
NONE

**数据源:**
缓存

**响应:**
```javascript
{}
```

<a id="time"></a>

### 获取服务器时间
```
GET /api/v3/time
```
获取服务器时间

**权重:**
1

**参数:**
NONE

**数据源:**
缓存

**响应:**
```javascript
{
  "serverTime": 1499827319559
}
```

<a id="exchangeInfo"></a>

### 交易规范信息
```
GET /api/v3/exchangeInfo
```
获取此时的交易规范信息

**权重:**
20


**参数:**

名称 |类型 |是否必须 | 描述
------------ |------------ |------------ |------------
symbol |STRING| No|示例：curl -X GET "https://api.binance.com/api/v3/exchangeInfo?symbol=BNBBTC" 
symbols |ARRAY OF STRING|No| 示例：curl -X GET "https://api.binance.com/api/v3/exchangeInfo?symbols=%5B%22BNBBTC%22,%22BTCUSDT%22%5D" <br/> 或 <br/> curl -g -X  GET 'https://api.binance.com/api/v3/exchangeInfo?symbols=["BTCUSDT","BNBBTC"]'
permissions |ENUM|No|示例：curl -X GET "https://api.binance.com/api/v3/exchangeInfo?permissions=SPOT" <br/> or <br/> curl -X GET "https://api.binance.com/api/v3/exchangeInfo?permissions=%5B%22MARGIN%22%2C%22LEVERAGED%22%5D" <br/> or <br/> curl -g -X GET 'https://api.binance.com/api/v3/exchangeInfo?permissions=["MARGIN","LEVERAGED"]' |
showPermissionSets|BOOLEAN|No|用于控制是否提供 `permissionSets` 字段的内容。默认为 `true`
symbolStatus|ENUM|No|用于过滤具有此 `tradingStatus` 的交易对。有效值： `TRADING`， `HALT`， `BREAK` <br> 不能与 `symbols` 或 `symbol` 组合使用。|


**备注**:
* 如果参数 `symbol` 或者 `symbols` 提供的交易对不存在, 系统会返回错误并提示交易对不正确。
* 所有的参数都是可选的。
* `permissions` 可以支持单个或多个值（例如 `SPOT`, `["MARGIN","LEVERAGED"]`）。此参数不能与 `symbol` 或 `symbols` 组合使用。
* 如果未提供 `permissions` 参数，那么所有具有 `SPOT`、`MARGIN` 或 `LEVERAGED` 权限的交易对都将被返回。
  * 要显示具有任何权限的交易对，您需要在 `permissions` 中明确指定它们：（例如 `["SPOT","MARGIN",...]`)。有关完整列表，请参阅 [可用权限](enums_CN.md#account-and-symbol-permissions)

<a id="examples-of-symbol-permissions-interpretation-from-the-response"></a>

**解释响应中的 `permissionSets`：**

* `[["A","B"]]` - 有权限"A"**或**权限"B"的账户可以下订单。
* `[["A"],["B"]]` - 有权限"A"**和**权限"B"的账户可以下订单。
* `[["A"],["B","C"]]` - 有权限"A"**和**权限"B"或权限"C"的账户可以下订单。（此处应用的是包含或，而不是排除或，因此账户可以同时拥有权限"B"和权限"C"。）

**数据源:**
缓存

**响应:**
```javascript
{
  "timezone": "UTC",
  "serverTime": 1508631584636,
  "rateLimits": [
    {
      "rateLimitType": "REQUESTS_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200 // 每分钟调用的所有接口权重之和不得超过1200
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 1,
      "limit": 10 // 每秒钟所有订单/撤单次数不得超过10
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 100000 // 每天订单/撤单不得超过10万
    },
    {
      "rateLimitType": "RAW_REQUESTS",
      "interval": "MINUTE",
      "intervalNum": 5,
      "limit": 5000 // 每5分钟调用订单次数不得超过5000
    }
  ],
  "exchangeFilters": [],
  "symbols": [
    {
      "symbol": "ETHBTC",
      "status": "TRADING",
      "baseAsset": "ETH",
      "baseAssetPrecision": 8,
      "quoteAsset": "BTC",
      "quotePrecision": 8,
      "quoteAssetPrecision": 8,
      "orderTypes": ["LIMIT", "MARKET"],
      "icebergAllowed": false,
      "ocoAllowed": true,
      "otoAllowed": true,
      "quoteOrderQtyMarketAllowed": true,
      "allowTrailingStop": false,
      "cancelReplaceAllowed": false,
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
        },
        {
          "filterType": "MIN_NOTIONAL",
          "minNotional": "0.00100000",
          "applyToMarket": true,
          "avgPriceMins": 5
        }
      ],
      "permissions": [],
      "permissionSets": [
        [
          "SPOT",
          "MARGIN"
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
}
```

## 行情接口

<a id="depth"></a>

### 深度信息
```
GET /api/v3/depth
```

**权重:**

限制 | 权重
------------ | ------------
1-100 | 5
101-500 | 25
501-1000 | 50
1001-5000 | 250

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | 默认 100; 最大 5000. 可选值:[5, 10, 20, 50, 100, 500, 1000, 5000] <br/> 如果 limit > 5000, 最多返回5000条数据.

**数据源:**
缓存

**响应:**
```javascript
{
  "lastUpdateId": 1027024,
  "bids": [
    [
      "4.00000000",     // 价位
      "431.00000000",   // 挂单量
      []                // 请忽略.
    ]
  ],
  "asks": [
    [
      "4.00000200",
      "12.00000000",
      []
    ]
  ]
}
```

<a id="trades"></a>

### 近期成交
```
GET /api/v3/trades
```
获取近期成交

**权重:**
25

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 500; max 1000.

**数据源:**
缓存

**响应:**
```javascript
[
  {
    "id": 28457,
    "price": "4.00000100",
    "qty": "12.00000000",
    "time": 1499865549590,
    "isBuyerMaker": true,
    "isBestMatch": true
  }
]
```

### 查询历史成交
```
GET /api/v3/historicalTrades
```

**权重:**
25

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 500; max 1000.
fromId | LONG | NO | 从哪一条成交id开始返回. 缺省返回最近的成交记录


**数据源:**
数据库


**响应:**
```javascript
[
  {
    "id": 28457,
    "price": "4.00000100",
    "qty": "12.00000000",
    "time": 1499865549590,
    "isBuyerMaker": true,
    "isBestMatch": true
  }
]
```

<a id="aggTrades"></a>

### 近期成交(归集)
```
GET /api/v3/aggTrades
```
与trades的区别是，同一个taker在同一时间同一价格与多个maker的成交会被合并为一条记录

**权重:**
2

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
fromId | LONG | NO | 从包含fromID的成交开始返回结果
startTime | LONG | NO | 从该时刻之后的成交记录开始返回结果
endTime | LONG | NO | 返回该时刻为止的成交记录
limit | INT | NO | 默认 500; 最大 1000.

* 如果没有发送任何筛选参数(fromId, startTime, endTime)，默认返回最近的成交记录

**数据源:**
数据库

**响应:**
```javascript
[
  {
    "a": 26129,         // 归集成交ID
    "p": "0.01633102",  // 成交价
    "q": "4.70443515",  // 成交量
    "f": 27781,         // 被归集的首个成交ID
    "l": 27781,         // 被归集的末个成交ID
    "T": 1498793709153, // 成交时间
    "m": true,          // 是否为主动卖出单
    "M": true           // 是否为最优撮合单(可忽略，目前总为最优撮合)
  }
]
```

<a id="klines"></a>

### K线数据
```
GET /api/v3/klines
```
每根K线的开盘时间可视为唯一ID

**权重:**
2

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES |请参考 [`K线间隔`](#kline-intervals)
startTime | LONG | NO |
endTime | LONG | NO |
timeZone |STRING| NO| 默认: 0 (UTC)
limit | INT | NO | Default 500; max 1000.

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

**数据源:**
数据库


**响应:**
```javascript
[
  [
    1499040000000,      // 开盘时间
    "0.01634790",       // 开盘价
    "0.80000000",       // 最高价
    "0.01575800",       // 最低价
    "0.01577100",       // 收盘价(当前K线未结束的即为最新价)
    "148976.11427815",  // 成交量
    1499644799999,      // 收盘时间
    "2434.19055334",    // 成交额
    308,                // 成交笔数
    "1756.87402397",    // 主动买入成交量
    "28.46694368",      // 主动买入成交额
    "17928899.62484339" // 请忽略该参数
  ]
]
```

<a id="uiKlines"></a>

### UIK线数据
```
GET /api/v3/uiKlines
```
请求参数与响应和k线接口相同。

`uiKlines` 返回修改后的k线数据，针对k线图的呈现进行了优化。


**权重:**
2

**参数:**

名称 | 类型 | 是否必需 | 描述
------    | ------ | ------------ | ------------
symbol    | STRING | YES          |
interval  | ENUM   | YES          | 请参考 [`K线间隔`](#kline-intervals)
startTime | LONG   | NO           |
endTime   | LONG   | NO           |
timeZone  | STRING | NO           | Default: 0 (UTC)
limit     | INT    | NO           | 默认 500; 最大 1000.

* 如果未发送 `startTime` 和 `endTime`，默认返回最近的交易。
* `timeZone`支持的值包括：
    * 小时和分钟（例如 `-1:00`，`05:45`）
    * 仅小时（例如 `0`，`8`，`4`）
    * 接受的值范围严格为 [-12:00 到 +14:00]（包括边界）
* 如果提供了`timeZone`，K线间隔将在该时区中解释，而不是在UTC中。
* 请注意，无论`timeZone`如何，`startTime`和`endTime`始终以UTC时区解释。

**数据源:**
数据库

**响应:**
```javascript
[
  [
    1499040000000,      // k线开盘时间
    "0.01634790",       // 开盘价
    "0.80000000",       // 最高价
    "0.01575800",       // 最低价
    "0.01577100",       // 收盘价(当前K线未结束的即为最新价)
    "148976.11427815",  // 成交量
    1499644799999,      // k线收盘时间
    "2434.19055334",    // 成交额
    308,                // 成交笔数
    "1756.87402397",    // 主动买入成交量
    "28.46694368",      // 主动买入成交额
    "0"                 // 请忽略该参数
  ]
]
```

<a id="avgPrice"></a>

### 当前平均价格

```
GET /api/v3/avgPrice
```
**权重:**
2

**参数:**
名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |

**数据源:**
缓存


**响应:**
```javascript
{
  "mins": 5,
  "price": "9.35751834",
  "closeTime": 1694061154503
}
```

<a id="twentyfourhourticker"></a>

### 24hr价格变动情况
```
GET /api/v3/ticker/24hr
```
请注意，不携带symbol参数会返回全部交易对数据，不仅数据庞大，而且权重极高

**权重:**

<table>
<thead>
    <tr>
        <th>参数</th>
        <th>提供Symbol数量</th>
        <th>权重</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td rowspan="2">symbol</td>
        <td>1</td>
        <td>2</td>
    </tr>
    <tr>
        <td>不提供symbol</td>
        <td>80</td>
    </tr>
    <tr>
        <td rowspan="4">symbols</td>
        <td>1-20</td>
        <td>2</td>
    </tr>
    <tr>
        <td>21-100</td>
        <td>40</td>
    </tr>
    <tr>
        <td> >= 101</td>
        <td>80</td>
    </tr>
    <tr>
        <td>不提供symbol</td>
        <td>80</td>
    </tr>
</tbody>
</table>

**参数:**

<table>
<thead>
    <tr>
        <th>名称</th>
        <th>类型</th>
        <th>是否强制要求</th>
        <th>详情</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>symbol</td>
        <td>STRING</td>
        <td>NO</td>
        <td rowspan="2">参数 `symbol` 和 `symbols` 不可以一起使用 <br/> 如果都不提供, 所有symbol的ticker数据都会返回. <br/><br/>
         symbols参数可接受的格式：
         ["BTCUSDT","BNBUSDT"] <br/>
         或 <br/>
         %5B%22BTCUSDT%22,%22BNBUSDT%22%5D
        </td>
     </tr>
     <tr>
        <td>symbols</td>
        <td>STRING</td>
        <td>NO</td>
     </tr>
     <tr>
        <td>type</td>
        <td>ENUM</td>
        <td>NO</td>
        <td>可接受的参数: <tt>FULL</tt> or <tt>MINI</tt>. <br/>如果不提供, 默认值为 <tt>FULL</tt> </td>
    </tr>
</tbody>
</table>

**数据源:**
缓存

**响应 - FULL:**
```javascript
{
  "symbol": "BNBBTC",
  "priceChange": "-94.99999800",
  "priceChangePercent": "-95.960",
  "weightedAvgPrice": "0.29628482",
  "prevClosePrice": "0.10002000",
  "lastPrice": "4.00000200",
  "lastQty": "200.00000000",
  "bidPrice": "4.00000000",
  "bidQty": "100.00000000",
  "askPrice": "4.00000200",
  "askQty": "100.00000000",
  "openPrice": "99.00000000",
  "highPrice": "100.00000000",
  "lowPrice": "0.10000000",
  "volume": "8913.30000000",
  "quoteVolume": "15.30000000",
  "openTime": 1499783499040,
  "closeTime": 1499869899040,
  "firstId": 28385,   // 首笔成交id
  "lastId": 28460,    // 末笔成交id
  "count": 76         // 成交笔数
}
```
OR
```javascript
[
  {
    "symbol": "BNBBTC",
    "priceChange": "-94.99999800",
    "priceChangePercent": "-95.960",
    "weightedAvgPrice": "0.29628482",
    "prevClosePrice": "0.10002000",
    "lastPrice": "4.00000200",
    "lastQty": "200.00000000",
    "bidPrice": "4.00000000",
    "bidQty": "100.00000000",
    "askPrice": "4.00000200",
    "askQty": "100.00000000",
    "openPrice": "99.00000000",
    "highPrice": "100.00000000",
    "lowPrice": "0.10000000",
    "volume": "8913.30000000",
    "quoteVolume": "15.30000000",
    "openTime": 1499783499040,
    "closeTime": 1499869899040,
  "firstId": 28385,   // 首笔成交id
  "lastId": 28460,    // 末笔成交id
  "count": 76         // 成交笔数
  }
]
```


**响应 - MINI**

```javascript
{
  "symbol":      "BNBBTC",          // 交易对
  "openPrice":   "99.00000000",     // 间隔开盘价
  "highPrice":   "100.00000000",    // 间隔最高价
  "lowPrice":    "0.10000000",      // 间隔最低价
  "lastPrice":   "4.00000200",      // 间隔收盘价
  "volume":      "8913.30000000",   // 总交易量 (base asset)
  "quoteVolume": "15.30000000",     // 总交易量 (quote asset)
  "openTime":    1499783499040,     // ticker间隔的开始时间
  "closeTime":   1499869899040,     // ticker间隔的结束时间
  "firstId":     28385,             // 统计时间内的第一笔trade id
  "lastId":      28460,             // 统计时间内的最后一笔trade id
  "count":       76                 // 统计时间内交易笔数
}
```

OR

```javascript
[
  {
    "symbol": "BNBBTC",
    "openPrice": "99.00000000",
    "highPrice": "100.00000000",
    "lowPrice": "0.10000000",
    "lastPrice": "4.00000200",
    "volume": "8913.30000000",
    "quoteVolume": "15.30000000",
    "openTime": 1499783499040,
    "closeTime": 1499869899040,
    "firstId": 28385,
    "lastId": 28460,
    "count": 76
  },
  {
    "symbol": "LTCBTC",
    "openPrice": "0.07000000",
    "highPrice": "0.07000000",
    "lowPrice": "0.07000000",
    "lastPrice": "0.07000000",
    "volume": "11.00000000",
    "quoteVolume": "0.77000000",
    "openTime": 1656908192899,
    "closeTime": 1656994592899,
    "firstId": 0,
    "lastId": 10,
    "count": 11
  }
]
```

### 交易日行情(Ticker)

```
GET /api/v3/ticker/tradingDay
```
交易日价格变动统计。

**权重:**

每个<tt>交易对</tt>占用4个权重. <br/><br/> 
当请求中的交易对数量超过50，此请求的权重将限制在200。

**参数:**

<table>
  <tr>
    <th>参数名</th>
    <th>类型</th>
    <th>是否必需</th>
    <th>描述</th>
  </tr>
  <tr>
    <td>symbol</td>
    <td rowspan="2">STRING</td>
    <td rowspan="2">YES</td>
    <td rowspan="2"> <tt>symbol</tt> 或者 <tt>symbols</tt> 必须提供之一 <br/><br/> <tt>symbols</tt> 可以接受的格式: <br/> ["BTCUSDT","BNBUSDT"] <br/>或者 <br/>%5B%22BTCUSDT%22,%22BNBUSDT%22%5D <br/><br/>  <tt>symbols</tt> 最多可以发送100个.
    </td>
  </tr>
  <tr>
     <td>symbols</td>
  </tr>
  <tr>
     <td>timeZone</td>
     <td>STRING</td>
     <td>NO</td>
     <td>Default: 0 (UTC)</td>
  </tr>
  <tr>
      <td>type</td>
      <td>ENUM</td>
      <td>NO</td>
      <td>可接受值: <tt>FULL</tt> or <tt>MINI</tt>. <br/>默认值: <tt>FULL</tt> </td>
  </tr>
</table>

**注意:**

* `timeZone`支持的值包括：
    * 小时和分钟（例如 `-1:00`，`05:45`）
    * 仅小时（例如 `0`，`8`，`4`）

**数据源:**
数据库

**响应 - FULL**

有 `symbol`:

```javascript
{
  "symbol":             "BTCUSDT",
  "priceChange":        "-83.13000000",         // 绝对价格变动
  "priceChangePercent": "-0.317",               // 相对价格变动百分比
  "weightedAvgPrice":   "26234.58803036",       // 报价成交量 / 成交量
  "openPrice":          "26304.80000000",
  "highPrice":          "26397.46000000",
  "lowPrice":           "26088.34000000",
  "lastPrice":          "26221.67000000",
  "volume":             "18495.35066000",       // 基础资产的成交量
  "quoteVolume":        "485217905.04210480",   // 报价资产的成交量
  "openTime":           1695686400000,
  "closeTime":          1695772799999,
  "firstId":            3220151555,             // 区间内的第一个交易的交易ID
  "lastId":             3220849281,             // 区间内的最后一个交易的交易ID
  "count":              697727                  // 区间内的交易数量
}

```

有 `symbols`:

```javascript
[
  {
    "symbol": "BTCUSDT",
    "priceChange": "-83.13000000",
    "priceChangePercent": "-0.317",
    "weightedAvgPrice": "26234.58803036",
    "openPrice": "26304.80000000",
    "highPrice": "26397.46000000",
    "lowPrice": "26088.34000000",
    "lastPrice": "26221.67000000",
    "volume": "18495.35066000",
    "quoteVolume": "485217905.04210480",
    "openTime": 1695686400000,
    "closeTime": 1695772799999,
    "firstId": 3220151555,
    "lastId": 3220849281,
    "count": 697727
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
]
```

**响应: - MINI**

有 `symbol`:

```javascript
{
  "symbol":         "BTCUSDT",
  "openPrice":      "26304.80000000",
  "highPrice":      "26397.46000000",
  "lowPrice":       "26088.34000000",
  "lastPrice":      "26221.67000000",
  "volume":         "18495.35066000",       // 基础资产的成交量
  "quoteVolume":    "485217905.04210480",   // 报价资产的成交量
  "openTime":       1695686400000,
  "closeTime":      1695772799999,
  "firstId":        3220151555,             // 区间内的第一个交易的交易ID
  "lastId":         3220849281,             // 区间内的最后一个交易的交易ID
  "count":          697727                  // 区间内的交易数量
}
```

有 `symbols`:

```javascript
[
  {
    "symbol": "BTCUSDT",
    "openPrice": "26304.80000000",
    "highPrice": "26397.46000000",
    "lowPrice": "26088.34000000",
    "lastPrice": "26221.67000000",
    "volume": "18495.35066000",
    "quoteVolume": "485217905.04210480",
    "openTime": 1695686400000,
    "closeTime": 1695772799999,
    "firstId": 3220151555,
    "lastId": 3220849281,
    "count": 697727
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
]
```

<a id="ticker-price"></a>

### 最新价格接口
```
GET /api/v3/ticker/price
```
返回最近价格

**权重:**

<table>
<thead>
    <tr>
        <th>参数</th>
        <th>Symbols数量</th>
        <th>权重</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td rowspan="2">symbol</td>
        <td>1</td>
        <td>2</td>
    </tr>
    <tr>
        <td>不提供symbol</td>
        <td>4</td>
    </tr>
    <tr>
        <td>symbols</td>
        <td>不限</td>
        <td>4</td>
    </tr>
</tbody>
</table>


**参数:**


<table>
<thead>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>是否强制</th>
      <th>详情</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>symbol</td>
        <td>STRING</td>
        <td>NO</td>
        <td rowspan="2"> 参数 `symbol` 和 `symbols` 不可以一起使用 <br/> 如果都不提供, 所有symbol的价格数据都会返回. <br/><br/>
        symbols参数可接受的格式：
         ["BTCUSDT","BNBUSDT"] <br/>
         或 <br/>
         %5B%22BTCUSDT%22,%22BNBUSDT%22%5D
        </td>
    </tr>
    <tr>
        <td>symbols</td>
        <td>STRING</td>
        <td>NO</td>
    </tr>
</tbody>
</table>


* 不发送交易对参数，则会返回所有交易对信息

**数据源:**
缓存

**响应:**
```javascript
{
  "symbol": "LTCBTC",
  "price": "4.00000200"
}
```
OR
```javascript
[
  {
    "symbol": "LTCBTC",
    "price": "4.00000200"
  },
  {
    "symbol": "ETHBTC",
    "price": "0.07946600"
  }
]
```

<a id="bookTicker"></a>

### 最优挂单接口
```
GET /api/v3/ticker/bookTicker
```
返回当前最优的挂单(最高买单，最低卖单)

**权重:**

<table>
<thead>
    <tr>
        <th>参数</th>
        <th>Symbols数量</th>
        <th>权重</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td rowspan="2">symbol</td>
        <td>1</td>
        <td>2</td>
    </tr>
    <tr>
        <td>不提供symbol</td>
        <td>4</td>
    </tr>
    <tr>
        <td>symbols</td>
        <td>不限</td>
        <td>4</td>
    </tr>
</tbody>
</table>


**参数:**


<table>
<thead>
    <tr>
      <th>参数名</th>
      <th>类型</th>
      <th>是否强制</th>
      <th>详情</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>symbol</td>
        <td>STRING</td>
        <td>NO</td>
        <td rowspan="2"> 参数 `symbol` 和 `symbols` 不可以一起使用 <br/> 如果都不提供, 所有symbol的bookTicker数据都会返回. <br/><br/>
        symbols参数可接受的格式：
         ["BTCUSDT","BNBUSDT"] <br/>
         或 <br/>
         %5B%22BTCUSDT%22,%22BNBUSDT%22%5D
        </td>
    </tr>
    <tr>
        <td>symbols</td>
        <td>STRING</td>
        <td>NO</td>
    </tr>
</tbody>
</table>

* 不发送交易对参数，则会返回所有交易对信息

**数据源:**
缓存


**响应:**
```javascript
{
  "symbol": "LTCBTC",
  "bidPrice": "4.00000000", // 最优买单价
  "bidQty": "431.00000000", // 挂单量
  "askPrice": "4.00000200", // 最优卖单价
  "askQty": "9.00000000"    // 挂单量
}
```
OR
```javascript
[
  {
    "symbol": "LTCBTC",
    "bidPrice": "4.00000000",
    "bidQty": "431.00000000",
    "askPrice": "4.00000200",
    "askQty": "9.00000000"
  },
  {
    "symbol": "ETHBTC",
    "bidPrice": "0.07946700",
    "bidQty": "9.00000000",
    "askPrice": "100000.00000000",
    "askQty": "1000.00000000"
  }
]
```

<a id="rollingwindowticker"></a>

### 滚动窗口价格变动统计

```
GET /api/v3/ticker
```

**注意:** 此接口和 `GET /api/v3/ticker/24hr` 有所不同.

此接口统计的时间范围比请求的`windowSize`多不超过59999ms.

接口的 `openTime` 是某一分钟的起始，而结束是当前的时间. 所以实际的统计区间会比请求的时间窗口多不超过59999ms.

比如, 结束时间 `closeTime` 是 1641287867099 (January 04, 2022 09:17:47:099 UTC) , `windowSize` 为 `1d`. 那么开始时间 `openTime` 则为 1641201420000 (January 3, 2022, 09:17:00 UTC)

**权重:** 4/交易对. <br/><br/> 如果`symbols`请求的交易对超过50, 上限是200.

**参数:**
<table>
  <tr>
    <th>Name</th>
    <th>Type</th>
    <th>Mandatory</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>symbol</td>
    <td rowspan="2">STRING</td>
    <td rowspan="2">YES</td>
    <td rowspan="2"> 提供 symbol或者symbols 其中之一  <br/> <tt>symbols</tt> 可以传入的格式: <br/> ["BTCUSDT","BNBUSDT"] <br/>or <br/>%5B%22BTCUSDT%22,%22BNBUSDT%22%5D <br/><br/> <tt>symbols</tt> 允许最多100个交易对
    </td>
  </tr>
  <tr>
     <td>symbols</td>
  </tr>
  <tr>
     <td>windowSize</td>
     <td>ENUM</td>
     <td>NO</td>
     <td>默认为 <tt>1d</tt> <br/> <tt>windowSize</tt> 支持的值: <br/> 如果是分钟: <tt>1m</tt>,<tt>2m</tt>....<tt>59m</tt> <br/> 如果是小时: <tt>1h</tt>, <tt>2h</tt>....<tt>23h</tt> <br/> 如果是天: <tt>1d</tt>...<tt>7d</tt> <br/><br/>  不可以组合使用, 比如<tt>1d2h</tt></td>
  </tr>
  <tr>
     <td>type</td>
     <td>ENUM</td>
      <td>NO</td>
      <td>可接受的参数: <tt>FULL</tt> or <tt>MINI</tt>. <br/>如果不提供, 默认值为 <tt>FULL</tt> </td>
  </tr>
</table>


**数据源:** 数据库

**响应 - FULL**

使用参数 `symbol` 返回:

```javascript
{
  "symbol":             "BNBBTC",
  "priceChange":        "-8.00000000",  // 价格变化
  "priceChangePercent": "-88.889",      // 价格变化百分比
  "weightedAvgPrice":   "2.60427807",
  "openPrice":          "9.00000000",
  "highPrice":          "9.00000000",
  "lowPrice":           "1.00000000",
  "lastPrice":          "1.00000000",
  "volume":             "187.00000000",
  "quoteVolume":        "487.00000000",
  "openTime":           1641859200000,  // ticker的开始时间
  "closeTime":          1642031999999,  // ticker的结束时间
  "firstId":            0,              // 统计时间内的第一笔trade id
  "lastId":             60,
  "count":              61              // 统计时间内交易笔数
}
```

使用参数 `symbols` 返回:

```javascript
[
  {
    "symbol": "BTCUSDT",
    "priceChange": "-154.13000000",
    "priceChangePercent": "-0.740",
    "weightedAvgPrice": "20677.46305250",
    "openPrice": "20825.27000000",
    "highPrice": "20972.46000000",
    "lowPrice": "20327.92000000",
    "lastPrice": "20671.14000000",
    "volume": "72.65112300",
    "quoteVolume": "1502240.91155513",
    "openTime": 1655432400000,
    "closeTime": 1655446835460,
    "firstId": 11147809,
    "lastId": 11149775,
    "count": 1967
  },
  {
    "symbol": "BNBBTC",
    "priceChange": "0.00008530",
    "priceChangePercent": "0.823",
    "weightedAvgPrice": "0.01043129",
    "openPrice": "0.01036170",
    "highPrice": "0.01049850",
    "lowPrice": "0.01033870",
    "lastPrice": "0.01044700",
    "volume": "166.67000000",
    "quoteVolume": "1.73858301",
    "openTime": 1655432400000,
    "closeTime": 1655446835460,
    "firstId": 2351674,
    "lastId": 2352034,
    "count": 361
  }
]
```


**响应 - MINI**

使用参数 `symbol` 返回:

```javascript
{
    "symbol": "LTCBTC",
    "openPrice": "0.10000000",
    "highPrice": "2.00000000",
    "lowPrice": "0.10000000",
    "lastPrice": "2.00000000",
    "volume": "39.00000000",
    "quoteVolume": "13.40000000",  // 此k线内所有交易的price(价格) x volume(交易量)的总和
    "openTime": 1656986580000,     // ticker窗口的开始时间
    "closeTime": 1657001016795,    // ticker窗口的结束时间
    "firstId": 0,                  // 首笔成交id
    "lastId": 34,
    "count": 35                    // 统计时间内交易笔数
}
```

使用参数 `symbols` 返回:

```javascript
[
    {
        "symbol": "BNBBTC",
        "openPrice": "0.10000000",
        "highPrice": "2.00000000",
        "lowPrice": "0.10000000",
        "lastPrice": "2.00000000",
        "volume": "39.00000000",
        "quoteVolume": "13.40000000", // 此k线内所有交易的price(价格) x volume(交易量)的总和
        "openTime": 1656986880000,    // ticker窗口的开始时间
        "closeTime": 1657001297799,   // ticker窗口的结束时间
        "firstId": 0,                 // 首笔成交id
        "lastId": 34,
        "count": 35                   // 统计时间内交易笔数
    },
    {
        "symbol": "LTCBTC",
        "openPrice": "0.07000000",
        "highPrice": "0.07000000",
        "lowPrice": "0.07000000",
        "lastPrice": "0.07000000",
        "volume": "33.00000000",
        "quoteVolume": "2.31000000",
        "openTime": 1656986880000,
        "closeTime": 1657001297799,
        "firstId": 0,
        "lastId": 32,
        "count": 33
    }
]
```

## 交易接口
### 下单 (TRADE)
```
POST /api/v3/order 
```

**权重:**
1

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
side | ENUM | YES | 详见枚举定义：[订单方向](./enums_CN.md#side)
type | ENUM | YES | 详见枚举定义：[订单类型](./enums_CN.md#ordertype)
timeInForce | ENUM | NO | 详见枚举定义：[生效时间](./enums.md#timeinforce)
quantity | DECIMAL | NO |
quoteOrderQty | DECIMAL | NO |
price | DECIMAL | NO |
newClientOrderId | STRING | NO | 用户自定义的orderid，如空缺系统会自动赋值。
strategyId |LONG| NO|
strategyType |INT| NO| 不能低于 `1000000`.
stopPrice | DECIMAL | NO | 仅 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT` 需要此参数。
trailingDelta|LONG|NO| 用于 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, 和 `TAKE_PROFIT_LIMIT` 类型的订单。
icebergQty | DECIMAL | NO | 仅有限价单(包括条件限价单与限价做事单)可以使用该参数，含义为创建冰山订单并指定冰山订单的数量。
newOrderRespType | ENUM | NO | 指定响应类型 `ACK`, `RESULT`, or `FULL`; `MARKET` 与 `LIMIT` 订单默认为`FULL`, 其他默认为`ACK`。
selfTradePreventionMode |ENUM| NO | 允许的 ENUM 取决于交易对的配置。支持的值有：[STP 模式](./enums_CN.md#stpmodes)。
recvWindow | LONG | NO |
timestamp | LONG | YES |

根据 order `type`的不同，<a id="order-type">某些参数</a> 有强制要求，具体如下:

Type | 强制要求的参数 | 其他信息
------------ | ------------ | ------------
`LIMIT` | `timeInForce`, `quantity`, `price`
`MARKET` | `quantity` |  市价买卖单可用`quantity`参数来设置`base asset`数量.<br/> 例如：BTCUSDT 市价单，BTC 买卖数量取决于`quantity`参数. <br/><br/>市价买卖单可用`quoteOrderQty`参数来设置`quote asset`数量. 正确的`quantity`取决于市场的流动性与`quoteOrderQty`<br/> 例如: 市价 `BUY` BTCUSDT，单子会基于`quoteOrderQty`- USDT 的数量，购买 BTC.<br/> 市价 `SELL` BTCUSDT，单子会卖出 BTC 来满足`quoteOrderQty`- USDT 的数量.
`STOP_LOSS` | `quantity`, `stopPrice`, `trailingDelta` | 条件满足后会下`MARKET`单子. (例如：达到`stopPrice`或`trailingDelta`被启动)
`STOP_LOSS_LIMIT` | `timeInForce`, `quantity`,  `price`, `stopPrice`, `trailingDelta`
`TAKE_PROFIT` | `quantity`, `stopPrice`, `trailingDelta` | 条件满足后会下`MARKET`单子. (例如：达到`stopPrice`或`trailingDelta`被启动)
`TAKE_PROFIT_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice`, `trailingDelta`
`LIMIT_MAKER` | `quantity`, `price` | 订单大部分情况下与普通的限价单没有区别，但是如果在当前价格会立即吃对手单并成交则下单会被拒绝。因此使用这个订单类型可以保证订单一定是挂单方，不会成为吃单方。

其他:

* 任何`LIMIT`或`LIMIT_MAKER`只要填`icebergQty`参数都可以下冰上订单。
* 冰山订单的 `timeInForce`必须设置为`GTC`。
* `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT_LIMIT` 与 `TAKE_PROFIT` 单子都能同时填上`trailingDelta`与`stopPrice`。
* 填上`quoteOrderQty`的市价单不会触犯过滤器的`LOT_SIZE`限制。订单的`quantity`会尽量满足`quoteOrderQty`的数量。

条件单的触发价格必须:

* 比下单时当前市价高: `STOP_LOSS` `BUY`, `TAKE_PROFIT` `SELL`
* 比下单时当前市价低: `STOP_LOSS` `SELL`, `TAKE_PROFIT` `BUY`

关于 newOrderRespType的三种选择

**数据源:**
撮合引擎

**Response ACK:**
返回速度最快，不包含成交信息，信息量最少
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "orderListId": -1, // 除非此单是订单列表的一部分, 否则此值为 -1
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595
}
```

**Response RESULT:**
返回速度居中，返回吃单成交的少量信息
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "orderListId": -1, // 除非此单是订单列表的一部分, 否则此值为 -1
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595,
  "price": "1.00000000",
  "origQty": "10.00000000",
  "executedQty": "10.00000000",
  "origQuoteOrderQty": "0.000000",
  "cummulativeQuoteQty": "10.00000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "MARKET",
  "side": "SELL"
  "workingTime": 1507725176595,
  "selfTradePreventionMode": "NONE"
}
```

**Response FULL:**
返回速度最慢，返回吃单成交的详细信息
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "orderListId": -1, // 除非此单是订单列表的一部分, 否则此值为 -1
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595,
  "price": "1.00000000",
  "origQty": "10.00000000",
  "executedQty": "10.00000000",
  "origQuoteOrderQty": "0.000000",
  "cummulativeQuoteQty": "10.00000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "MARKET",
  "side": "SELL",
  "workingTime": 1507725176595,
  "selfTradePreventionMode": "NONE",
  "fills": [
    {
      "price": "4000.00000000",
      "qty": "1.00000000",
      "commission": "4.00000000",
      "commissionAsset": "USDT",
      "tradeId": 56
    },
    {
      "price": "3999.00000000",
      "qty": "5.00000000",
      "commission": "19.99500000",
      "commissionAsset": "USDT",
      "tradeId": 57
    },
    {
      "price": "3998.00000000",
      "qty": "2.00000000",
      "commission": "7.99600000",
      "commissionAsset": "USDT",
      "tradeId": 58
    },
    {
      "price": "3997.00000000",
      "qty": "1.00000000",
      "commission": "3.99700000",
      "commissionAsset": "USDT",
      "tradeId": 59
    },
    {
      "price": "3995.00000000",
      "qty": "1.00000000",
      "commission": "3.99500000",
      "commissionAsset": "USDT",
      "tradeId": 60
    }
  ]
}
```

<a id="conditional-fields-in-order-responses"></a>

**订单响应中的特定条件时才会出现的字段**

订单响应中的有一些字段仅在满足特定条件时才会出现。这些订单响应可以来自下订单，查询订单或取消订单，并且可以包括订单列表类型。
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
`trailingTime` | 追踪单被激活和跟踪价格变化的时间。                                  | 出现在追踪止损订单中。                               | `"trailingTime": -1`|
`usedSor` | 用于确定订单是否使用SOR的字段 | 在使用SOR下单时出现 |`"usedSor": true` ｜
`workingFloor` | 用以定义订单是通过 SOR 还是由订单提交到的订单薄（order book）成交的。   |出现在使用了 SOR 的订单中。                             |`"workingFloor": "SOR"`|

### 测试下单接口 (TRADE)

```
POST /api/v3/order/test 
```

用于测试订单请求，但不会提交到撮合引擎

**权重:**

| 条件 | 权重 |
|------------           | ------------ |
|没有 `computeCommissionRates`| 1|
|有 `computeCommissionRates`|20|

**参数:**

除了 [`POST /api/v3/order`](#下单-trade) 所有参数,
下面参数也支持:

参数名                   |类型          | 是否必需    | 描述
------------           | ------------ | ------------ | ------------
computeCommissionRates | BOOLEAN      | NO           | 默认值: `false`

**数据源:**
缓存

**响应:**

没有 `computeCommissionRates`

```javascript
{}
```

有 `computeCommissionRates`

```javascript
{
  "standardCommissionForOrder": {   // 订单交易的标准佣金率
    "maker": "0.00000112",
    "taker": "0.00000114",
  },
  "taxCommissionForOrder": {        // 订单交易的税率
    "maker": "0.00000112",
    "taker": "0.00000114",
  },
  "discount": {                     // 以BNB支付时的标准佣金折扣。
    "enabledForAccount": true,
    "enabledForSymbol": true,
    "discountAsset": "BNB",
    "discount": "0.25000000"        // 当用BNB支付佣金时，在标准佣金上按此比率打折
  }
}
```

### 查询订单 (USER_DATA)
```
GET /api/v3/order
```
查询订单状态

**权重:**
4

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
origClientOrderId | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**注意:**
* 至少需要发送 `orderId` 与 `origClientOrderId`中的一个
* 某些订单中`cummulativeQuoteQty`<0，是由于这些订单是cummulativeQuoteQty功能上线之前的订单。

**数据源:**
缓存 => 数据库

**响应:**
```javascript
{
  "symbol": "LTCBTC",               // 交易对
  "orderId": 1,                     // 系统的订单ID
  "orderListId": -1,                // 除非此单是订单列表的一部分, 否则此值为 -1
  "clientOrderId": "myOrder1",      // 客户自己设置的ID
  "price": "0.1",                   // 订单价格
  "origQty": "1.0",                 // 用户设置的原始订单数量
  "executedQty": "0.0",             // 交易的订单数量
  "origQuoteOrderQty": "0.000000",
  "cummulativeQuoteQty": "0.0",     // 累计交易的金额
  "status": "NEW",                  // 订单状态
  "timeInForce": "GTC",             // 订单的时效方式
  "type": "LIMIT",                  // 订单类型， 比如市价单，现价单等
  "side": "BUY",                    // 订单方向，买还是卖
  "stopPrice": "0.0",               // 止损价格
  "icebergQty": "0.0",              // 冰山数量
  "time": 1499827319559,            // 订单时间
  "updateTime": 1499827319559,      // 最后更新时间
  "isWorking": true,                // 订单是否出现在orderbook中
  "workingTime":1499827319559,      // 订单添加到 order book 的时间
  "origQuoteOrderQty": "0.000000",  // 原始的交易金额
  "selfTradePreventionMode": "NONE" // 如何处理自我交易模式
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](#conditional-fields-in-order-responses) 部分。

### 撤销订单 (TRADE)
```
DELETE /api/v3/order 
```

**权重:**
1

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
origClientOrderId | STRING | NO |
newClientOrderId | STRING | NO |  用户自定义的本次撤销操作的ID(注意不是被撤销的订单的自定义ID)。如无指定会自动赋值。
cancelRestrictions| ENUM | NO | 支持的值: <br>`ONLY_NEW` - 如果订单状态为 `NEW`，撤销将成功。<br> `ONLY_PARTIALLY_FILLED` - 如果订单状态为 `PARTIALLY_FILLED`，撤销将成功。
recvWindow | LONG | NO |
timestamp | LONG | YES |

* `orderId` 与 `origClientOrderId` 必须至少发送一个.
* 如果两个参数一起发送, `orderId`优先被考虑.

**数据源:**
撮合引擎

**响应:**
```javascript
{
  "symbol": "LTCBTC",
  "orderId": 28,
  "orderListId": -1,                // 除非此单是订单列表的一部分, 否则此值为 -1
  "origClientOrderId": "myOrder1",
  "clientOrderId": "cancelMyOrder1",
  "transactTime": 1507725176595,
  "price": "1.00000000",
  "origQty": "10.00000000",
  "executedQty": "8.00000000",
  "origQuoteOrderQty": "0.000000",
  "cummulativeQuoteQty": "8.00000000",
  "status": "CANCELED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "SELL",
  "selfTradePreventionMode": "NONE"
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](#conditional-fields-in-order-responses) 部分。

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

### 撤销单一交易对的所有挂单 (TRADE)

```
DELETE /api/v3/openOrders
```

撤销单一交易对下所有挂单。这也包括了来自订单列表的挂单。

**权重:**
1

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
recvWindow | LONG | NO | 不能大于 ```60000```
timestamp | LONG | YES |

**数据源:**
撮合引擎


**响应:**

```json
[
  {
    "symbol": "BTCUSDT",
    "origClientOrderId": "E6APeyTJvkMvLMYMqu1KQ4",
    "orderId": 11,
    "orderListId": -1,
    "clientOrderId": "pXLV6Hz6mprAcVYpVMTGgx",
    "transactTime": 1684804350068,
    "price": "0.089853",
    "origQty": "0.178622",
    "executedQty": "0.000000",
    "origQuoteOrderQty": "0.000000",
    "cummulativeQuoteQty": "0.000000",
    "status": "CANCELED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "selfTradePreventionMode": "NONE"
  },
  {
    "symbol": "BTCUSDT",
    "origClientOrderId": "A3EF2HCwxgZPFMrfwbgrhv",
    "orderId": 13,
    "orderListId": -1,
    "clientOrderId": "pXLV6Hz6mprAcVYpVMTGgx",
    "transactTime": 1684804350068,
    "price": "0.090430",
    "origQty": "0.178622",
    "executedQty": "0.000000",
    "origQuoteOrderQty": "0.000000",
    "cummulativeQuoteQty": "0.000000",
    "status": "CANCELED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "selfTradePreventionMode": "NONE"
  },
  {
    "orderListId": 1929,
    "contingencyType": "OCO",
    "listStatusType": "ALL_DONE",
    "listOrderStatus": "ALL_DONE",
    "listClientOrderId": "2inzWQdDvZLHbbAmAozX2N",
    "transactionTime": 1585230948299,
    "symbol": "BTCUSDT",
    "orders": [
      {
        "symbol": "BTCUSDT",
        "orderId": 20,
        "clientOrderId": "CwOOIPHSmYywx6jZX77TdL"
      },
      {
        "symbol": "BTCUSDT",
        "orderId": 21,
        "clientOrderId": "461cPg51vQjV3zIMOXNz39"
      }
    ],
    "orderReports": [
      {
        "symbol": "BTCUSDT",
        "origClientOrderId": "CwOOIPHSmYywx6jZX77TdL",
        "orderId": 20,
        "orderListId": 1929,
        "clientOrderId": "pXLV6Hz6mprAcVYpVMTGgx",
        "transactTime": 1684804350068,
        "price": "0.668611",
        "origQty": "0.690354",
        "executedQty": "0.000000",
        "origQuoteOrderQty": "0.000000",
        "cummulativeQuoteQty": "0.000000",
        "status": "CANCELED",
        "timeInForce": "GTC",
        "type": "STOP_LOSS_LIMIT",
        "side": "BUY",
        "stopPrice": "0.378131",
        "icebergQty": "0.017083",
        "selfTradePreventionMode": "NONE"
      },
      {
        "symbol": "BTCUSDT",
        "origClientOrderId": "461cPg51vQjV3zIMOXNz39",
        "orderId": 21,
        "orderListId": 1929,
        "clientOrderId": "pXLV6Hz6mprAcVYpVMTGgx",
        "transactTime": 1684804350068,
        "price": "0.008791",
        "origQty": "0.690354",
        "executedQty": "0.000000",
        "origQuoteOrderQty": "0.000000",
        "cummulativeQuoteQty": "0.000000",
        "status": "CANCELED",
        "timeInForce": "GTC",
        "type": "LIMIT_MAKER",
        "side": "BUY",
        "icebergQty": "0.639962",
        "selfTradePreventionMode": "NONE"
      }
    ]
  }
]
```

### 撤消挂单再下单 (TRADE)

```
POST /api/v3/order/cancelReplace
```
撤消挂单并在同个交易对上重新下单。

在撤消订单和下单前会判断: 1) 过滤器参数, 以及 2) 目前下单数量。

即使请求中没有尝试发送新订单，比如(`newOrderResult: NOT_ATTEMPTED`)，下单的数量仍然会加1。

**权重:**
1

**参数:**
名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
side   |ENUM| YES|
type   |ENUM| YES|
cancelReplaceMode|ENUM|YES| 指定类型：`STOP_ON_FAILURE` - 如果撤消订单失败将不会继续重新下单。<br/> `ALLOW_FAILURE` - 不管撤消订单是否成功都会继续重新下单。
timeInForce|ENUM|NO|
quantity|DECIMAL|NO|
quoteOrderQty |DECIMAL|NO
price |DECIMAL|NO
cancelNewClientOrderId|STRING|NO | 用户自定义的id，如空缺系统会自动赋值
cancelOrigClientOrderId|STRING| NO| 必须提供`cancelOrigClientOrderId` 或者 `cancelOrderId`。 如果两个参数都提供, `cancelOrderId` 会占优先。
cancelOrderId|LONG|NO| 必须提供`cancelOrigClientOrderId` 或者 `cancelOrderId`。 如果两个参数都提供，`cancelOrderId` 会占优先。
newClientOrderId |STRING|NO| 用于辨识新订单。
strategyId |LONG| NO|
strategyType |INT| NO| 不能低于 `1000000`。
stopPrice|DECIMAL|NO|
trailingDelta|LONG|NO|
icebergQty|DECIMAL|NO|
newOrderRespType|ENUM|NO|指定响应类型: <br/> 指定响应类型 `ACK`, `RESULT`, or `FULL`; `MARKET` 与 `LIMIT` 订单默认为`FULL`， 其他默认为`ACK`。
selfTradePreventionMode|ENUM|NO|允许的 ENUM 取决于交易对的配置。支持的值有：[STP 模式](./enums_CN.md#stpmodes)。
cancelRestrictions| ENUM | NO | 支持的值: <br>`ONLY_NEW` - 如果订单状态为 `NEW`，撤销将成功。<br> `ONLY_PARTIALLY_FILLED` - 如果订单状态为 `PARTIALLY_FILLED`，撤销将成功。
orderRateLimitExceededMode| ENUM | NO |支持的值： <br></br> “DO_NOTHING”（默认值）- 仅在账户未超过未成交订单频率限制时，会尝试取消订单。<br></br> “CANCEL_ONLY” - 将始终取消订单。
recvWindow | LONG | NO | 不能大于 `60000`
timestamp | LONG | YES |


如同 `POST /api/v3/order`，额外的强制参数取决于 `type` 。

响应格式根据消息的处理是成功、部分成功还是失败而有所不同。

**数据来源:**
撮合引擎

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
        <td align=right><code>N/A</code></td>
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


**响应：没有超出未成交订单计数时的 Response SUCCESS**
```javascript
// 撤单和下单都成功
{
  "cancelResult": "SUCCESS",
  "newOrderResult": "SUCCESS",
  "cancelResponse": {
    "symbol": "BTCUSDT",
    "origClientOrderId": "DnLo3vTAQcjha43lAZhZ0y",
    "orderId": 9,
    "orderListId": -1,
    "clientOrderId": "osxN3JXAtJvKvCqGeMWMVR",
    "transactTime": 1684804350068,
    "price": "0.01000000",
    "origQty": "0.000100",
    "executedQty": "0.00000000",
    "origQuoteOrderQty": "0.000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "CANCELED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL"
  },
  "newOrderResponse": {
    "symbol": "BTCUSDT",
    "orderId": 10,
    "orderListId": -1,
    "clientOrderId": "wOceeeOzNORyLiQfw7jd8S",
    "transactTime": 1652928801803,
    "price": "0.02000000",
    "origQty": "0.040000",
    "executedQty": "0.00000000",
    "origQuoteOrderQty": "0.000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "fills": []
  }
}
```

**响应：选择了 `STOP_ON_FAILURE` 而且账户没有超出未成交订单计数时, 撤单出现错误**
```javascript
{
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
}
```

**响应：撤单成功而且账户没有超出未成交订单计数时，下单失败**
```javascript
{
  "code": -2021,
  "msg": "Order cancel-replace partially failed.",
  "data": {
    "cancelResult": "SUCCESS",
    "newOrderResult": "FAILURE",
    "cancelResponse": {
      "symbol": "BTCUSDT",
      "origClientOrderId": "86M8erehfExV8z2RC8Zo8k",
      "orderId": 3,
      "orderListId": -1,
      "clientOrderId": "G1kLo6aDv2KGNTFcjfTSFq",
      "price": "0.006123",
      "origQty": "10000.000000",
      "executedQty": "0.000000",
      "origQuoteOrderQty": "0.000000",
      "cummulativeQuoteQty": "0.000000",
      "status": "CANCELED",
      "timeInForce": "GTC",
      "type": "LIMIT_MAKER",
      "side": "SELL"
    },
    "newOrderResponse": {
      "code": -2010,
      "msg": "Order would immediately match and take."
    }
  }
}
```

**响应：选择 `ALLOW_FAILURE` 而且账户没有超出未成交订单计数时, 撤单出现错误**
```javascript
{
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
      "orderId": 11,
      "orderListId": -1,
      "clientOrderId": "pfojJMg6IMNDKuJqDxvoxN",
      "transactTime": 1648540168818
    }
  }
}
```

**响应：选择 `cancelReplaceMode=ALLOW_FAILURE` 而且账户没有超出未成交订单计数时, 撤单和下单失败**
```javascript
{
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
}
```

**响应：选择 `orderRateLimitExceededMode=DO_NOTHING` 而且账户超出未成交订单计数时**

```javascript
{
  "code": -1015,
  "msg": "Too many new orders; current limit is 1 orders per 10 SECOND." 
}
```

**响应：选择 `orderRateLimitExceededMode=CANCEL_ONLY` 而且账户超出未成交订单计数时**

```javascript
{
  "code": -2021,
  "msg": "Order cancel-replace partially failed.",
  "data": {
    "cancelResult": "SUCCESS",
    "newOrderResult": "FAILURE",
    "cancelResponse": {
      "symbol": "LTCBNB",
      "origClientOrderId": "GKt5zzfOxRDSQLveDYCTkc",
      "orderId": 64,
      "orderListId": -1,
      "clientOrderId": "loehOJF3FjoreUBDmv739R",
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
      "msg": "Too many new orders; current limit is 1 orders per 10 SECOND." 
    }
  }
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](#conditional-fields-in-order-responses) 部分。

### 查看账户当前挂单 (USER_DATA)
```
GET /api/v3/openOrders 
```
请小心使用不带symbol参数的调用

**权重:**
带symbol: 6
不带: 80

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

* 不带symbol参数，会返回所有交易对的挂单

**数据源:**
缓存 => 数据库

**响应:**
```javascript
[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "orderListId": -1, // 除非此单是订单列表的一部分, 否则此值为 -1
    "clientOrderId": "myOrder1",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "origQuoteOrderQty": "0.000000",
    "cummulativeQuoteQty": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": 1499827319559,
    "updateTime": 1499827319559,
    "isWorking": true,
    "origQuoteOrderQty": "0.000000",
    "workingTime": 1499827319559,
    "selfTradePreventionMode": "NONE"
  }
]
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](#conditional-fields-in-order-responses) 部分。

### 查询所有订单（包括历史订单） (USER_DATA)
```
GET /api/v3/allOrders 
```

**权重:**
20

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO | 只返回此orderID之后的订单，缺省返回最近的订单
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

**注意:**
* 如设置 `orderId` , 订单量将 >=  `orderId`。否则将返回最新订单。
* 一些历史订单 `cummulativeQuoteQty`  < 0, 是指数据此时不存在。
* 如果设置 `startTime` 和 `endTime`, `orderId` 就不需要设置。
* `startTime`和`endTime`之间的时间不能超过 24 小时。

**数据源:**
数据库

**响应:**
```javascript
[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "orderListId": -1,  // 除非此单是订单列表的一部分, 否则此值为 -1
    "clientOrderId": "myOrder1",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "origQuoteOrderQty": "0.0",
    "cummulativeQuoteQty": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": 1499827319559,
    "updateTime": 1499827319559,
    "isWorking": true,
    "origQuoteOrderQty": "0.000000",
    "workingTime": 1499827319559,
    "selfTradePreventionMode": "NONE"
  }
]
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](#conditional-fields-in-order-responses) 部分。


### 订单列表（Order lists）

#### 发送新 OCO 订单 - 已弃用 (TRADE)

```
POST /api/v3/order/oco 
```

**权重:** 1

发送新的 OCO。

* 价格限制：
    * `SELL`： Limit price > 最后交易价格 > stop Price
    * `BUY`： Limit price < 最后交易价格 < stop Price
* 数量限制：
    * 两条腿的数量必须相同。
    * 不过， `冰山` 交易的数量不一定相同
* `OCO` 将**2个订单**添加到未成交的订单计数， `EXCHANGE_MAX_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器中。

**参数:**

名称 |类型| 是否必需 | 描述
-----|-----|----------| -----------
symbol|STRING| YES|
listClientOrderId|STRING|NO| 整个orderList的唯一ID
side|ENUM|YES| 详见枚举定义：[订单方向](./enums_CN.md#side)
quantity|DECIMAL|YES|
limitClientOrderId|STRING|NO| 限价单的唯一ID
price|DECIMAL|YES|
limitStrategyId |LONG| NO
limitStrategyType | INT| NO | 不能低于 `1000000`
limitIcebergQty|DECIMAL|NO|
trailingDelta|LONG|NO|
stopClientOrderId |STRING|NO| 止损/止损限价单的唯一ID
stopPrice |DECIMAL| YES
stopStrategyId |LONG| NO
stopStrategyType |INT| NO | 不能低于 `1000000`
stopLimitPrice|DECIMAL|NO| 如果提供，须配合提交`stopLimitTimeInForce`
stopIcebergQty|DECIMAL|NO|
stopLimitTimeInForce|ENUM|NO| 有效值 `GTC`/`FOK`/`IOC`
newOrderRespType|ENUM|NO| 详见枚举定义：[订单返回类型](./enums_CN.md#orderresponsetype)
selfTradePreventionMode |ENUM| NO | 允许的 ENUM 取决于交易对的配置。支持的值有：[STP 模式](./enums_CN.md#stpmodes)。
recvWindow|LONG|NO| 不能大于 `60000`
timestamp|LONG|YES|


**数据源:**
撮合引擎

**响应**

```json
{
  "orderListId": 0,
  "contingencyType": "OCO",
  "listStatusType": "EXEC_STARTED",
  "listOrderStatus": "EXECUTING",
  "listClientOrderId": "JYVpp3F0f5CAG15DhtrqLp",
  "transactionTime": 1563417480525,
  "symbol": "LTCBTC",
  "orders": [
    {
      "symbol": "LTCBTC",
      "orderId": 2,
      "clientOrderId": "Kk7sqHb9J6mJWTMDVW7Vos"
    },
    {
      "symbol": "LTCBTC",
      "orderId": 3,
      "clientOrderId": "xTXKaGYd4bluPVp78IVRvl"
    }
  ],
  "orderReports": [
    {
      "symbol": "LTCBTC",
      "orderId": 2,
      "orderListId": 0,
      "clientOrderId": "Kk7sqHb9J6mJWTMDVW7Vos",
      "transactTime": 1563417480525,
      "price": "0.000000",
      "origQty": "0.624363",
      "executedQty": "0.000000",
      "origQuoteOrderQty": "0.000000",
      "cummulativeQuoteQty": "0.000000",
      "status": "NEW",
      "timeInForce": "GTC",
      "type": "STOP_LOSS",
      "side": "BUY",
      "stopPrice": "0.960664",
      "workingTime": -1,
      "selfTradePreventionMode": "NONE"
    },
    {
      "symbol": "LTCBTC",
      "orderId": 3,
      "orderListId": 0,
      "clientOrderId": "xTXKaGYd4bluPVp78IVRvl",
      "transactTime": 1563417480525,
      "price": "0.036435",
      "origQty": "0.624363",
      "executedQty": "0.000000",
      "origQuoteOrderQty": "0.000000",
      "cummulativeQuoteQty": "0.000000",
      "status": "NEW",
      "timeInForce": "GTC",
      "type": "LIMIT_MAKER",
      "side": "BUY",
      "workingTime": 1563417480525,
      "selfTradePreventionMode": "NONE"
    }
  ]
}
```

#### New Order list - OCO (TRADE)

```
POST /api/v3/orderList/oco
```

发送新 one-cancels-the-other (OCO) 订单，激活其中一个订单会立即取消另一个订单。

* OCO 包含了两个订单，分别被称为 **上方订单** 和 **下方订单**。
* 其中一个订单必须是 `LIMIT_MAKER/TAKE_PROFIT/TAKE_PROFIT_LIMIT` 订单，另一个订单必须是 `STOP_LOSS` 或 `STOP_LOSS_LIMIT` 订单。
* 针对价格限制：
  * 如果 OCO 订单方向是 `SELL`： 
    * `LIMIT_MAKER/TAKE_PROFIT_LIMIT` `price` > 最后交易价格 >  `STOP_LOSS/STOP_LOSS_LIMIT` `stopPrice`  
    * `TAKE_PROFIT` `stopPrice` > 最后交易价格 > `STOP_LOSS/STOP_LOSS_LIMIT` `stopPrice`   
  * 如果 OCO 订单方向是 `BUY`：
    * `LIMIT_MAKER/TAKE_PROFIT_LIMIT` `price` < 最后交易价格 < `stopPrice`  
    * `TAKE_PROFIT` `stopPrice` < 最后交易价格 < `STOP_LOSS/STOP_LOSS_LIMIT` `stopPrice`


**权重:** 
1

**参数:**

名称                    | 类型    | 是否必需   | 描述
-----                  |------  | -----     |----
symbol                 |STRING  |Yes        |
listClientOrderId      |STRING  |No         |整个 OCO order list 的唯一ID。 如果未发送则自动生成。 <br> 仅当前一个订单已填满或完全过期时，才会接受具有相同的`listClientOrderId`。 <br> `listClientOrderId` 与 `aboveClientOrderId` 和 `belowCLientOrderId` 不同。
side                   |ENUM    |Yes        |订单方向：`BUY` or `SELL`
quantity               |DECIMAL |Yes        |两个订单的数量。
aboveType              |ENUM    |Yes        |支持值：`STOP_LOSS_LIMIT`, `STOP_LOSS`, `LIMIT_MAKER`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`。
aboveClientOrderId     |STRING  |No         |上方订单的唯一ID。 如果未发送则自动生成。
aboveIcebergQty        |LONG    |No         |请注意，只有当 `aboveTimeInForce` 为 `GTC` 时才能使用。
abovePrice             |DECIMAL |No         |当 `aboveType` 是 `STOP_LOSS_LIMIT`, `LIMIT_MAKER` 或 `TAKE_PROFIT_LIMIT` 时，可用以指定限价。
aboveStopPrice         |DECIMAL |No         |如果 `aboveType` 是 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT` 或 `TAKE_PROFIT_LIMIT` 才能使用。<br> 必须指定 `aboveStopPrice` 或 `aboveTrailingDelta` 或两者。
aboveTrailingDelta     |LONG    |No         |请看 [追踪止盈止损(Trailing Stop)订单常见问题](faqs/trailing-stop-faq_CN.md)。
aboveTimeInForce       |DECIMAL |No         |如果 `aboveType` 是 `STOP_LOSS_LIMIT` 或 `TAKE_PROFIT_LIMIT`，则为必填项。
aboveStrategyId        |LONG     |No         |订单策略中上方订单的 ID。
aboveStrategyType      |INT     |No         |上方订单策略的任意数值。<br>小于 `1000000` 的值被保留，无法使用。
belowType              |ENUM    |Yes        |支持值：`STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`,`TAKE_PROFIT_LIMIT`。
belowClientOrderId     |STRING  |No         |
belowIcebergQty        |LONG    |No         |请注意，只有当 `belowTimeInForce` 为 `GTC` 时才能使用。
belowPrice             |DECIMAL |No         |当 `belowType` 是 `STOP_LOSS_LIMIT`, `LIMIT_MAKER` 或 `TAKE_PROFIT_LIMIT` 时，可用以指定限价。
belowStopPrice         |DECIMAL |No         |如果 `belowType` 是 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT` 或 `TAKE_PROFIT_LIMIT` 才能使用。<br> 必须指定 `belowStopPrice` 或 `belowTrailingDelta` 或两者。
belowTrailingDelta     |LONG    |No         |请看 [追踪止盈止损(Trailing Stop)订单常见问题](faqs/trailing-stop-faq_CN.md)。
belowTimeInForce       |ENUM    |No         |如果`belowType` 是 `STOP_LOSS_LIMIT` 或 `TAKE_PROFIT_LIMIT`，则为必须配合提交的值。
belowStrategyId        |LONG    |No          |订单策略中下方订单的 ID。
belowStrategyType      |INT     |No         |下方订单策略的任意数值。<br>小于 `1000000` 的值被保留，无法使用。
newOrderRespType       |ENUM    |No         |响应格式可选值: `ACK`, `RESULT`, `FULL`。
selfTradePreventionMode|ENUM    |No         |允许的 ENUM 取决于交易对上的配置。 支持值：[STP 模式](./enums_CN.md#stpmodes)。
recvWindow             |LONG   |No          |不能大于 `60000`。
timestamp              |LONG   |Yes         | 

**数据源:**
撮合引擎

**响应:**

使用 `newOrderRespType` 参数来选择 `orderReports` 的响应格式。以下示例适用于 `RESULT` 响应类型。 请参阅 [`POST /api/v3/order`](#下单-trade)了解更多 `orderReports` 的响应类型。

```javascript
{
    "orderListId": 1,
    "contingencyType": "OCO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "lH1YDkuQKWiXVXHPSKYEIp",
    "transactionTime": 1710485608839,
    "symbol": "LTCBTC",
    "orders": [
        {
            "symbol": "LTCBTC",
            "orderId": 10,
            "clientOrderId": "44nZvqpemY7sVYgPYbvPih"
        },
        {
            "symbol": "LTCBTC",
            "orderId": 11,
            "clientOrderId": "NuMp0nVYnciDiFmVqfpBqK"
        }
    ],
    "orderReports": [
        {
            "symbol": "LTCBTC",
            "orderId": 10,
            "orderListId": 1,
            "clientOrderId": "44nZvqpemY7sVYgPYbvPih",
            "transactTime": 1710485608839,
            "price": "1.00000000",
            "origQty": "5.00000000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "NEW",
            "timeInForce": "GTC",
            "type": "STOP_LOSS_LIMIT",
            "side": "SELL",
            "stopPrice": "1.00000000",
            "workingTime": -1,
            "icebergQty": "1.00000000",
            "selfTradePreventionMode": "NONE"
        },
        {
            "symbol": "LTCBTC",
            "orderId": 11,
            "orderListId": 1,
            "clientOrderId": "NuMp0nVYnciDiFmVqfpBqK",
            "transactTime": 1710485608839,
            "price": "3.00000000",
            "origQty": "5.00000000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "NEW",
            "timeInForce": "GTC",
            "type": "LIMIT_MAKER",
            "side": "SELL",
            "workingTime": 1710485608839,
            "selfTradePreventionMode": "NONE"
        }
    ]
}
```

<a id="new-order-list---oto-trade"></a>

#### New Order List - OTO (TRADE)

```
POST /api/v3/orderList/oto
```

发送一个新的 OTO 订单。

* 一个 OTO 订单（One-Triggers-the-Other）是一个包含了两个订单的订单列表.
* 第一个订单被称为**生效订单**，必须为 `LIMIT` 或 `LIMIT_MAKER` 类型的订单。最初，订单簿上只有生效订单。
* 第二个订单被称为**待处理订单**。它可以是任何订单类型，但不包括使用参数 `quoteOrderQty` 的 `MARKET` 订单。只有当生效订单**完全成交**时，待处理订单才会被自动下单。
* 如果生效订单或者待处理订单中的任意一个被单独取消，订单列表中剩余的那个订单也会被随之取消或过期。
* 如果生效订单在下订单列表后**立即完全成交**，则可能会得到订单响应。其中，生效订单的状态为 `FILLED` ，但待处理订单的状态为 `PENDING_NEW`。针对这类情况，如果需要检查当前状态，您可以查询相关的待处理订单。
* `OTO` 订单将**2 个订单**添加到未成交订单计数，`EXCHANGE_MAX_NUM_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器中。

**权重:** 
1

**参数:**

名称                    | 类型    | 是否必需   | 描述
----                   |----   |------    |------
symbol                 |STRING |YES       |
listClientOrderId      |STRING |NO        |整个订单列表的唯一ID。 如果未发送则自动生成。 <br> 仅当前一个订单列表已填满或完全过期时，才会接受含有相同 `listClientOrderId` 值的新订单列表。 <br> `listClientOrderId` 与 `workingClientOrderId` 和 `pendingClientOrderId` 不同。
newOrderRespType       |ENUM   |NO        |用于设置JSON响应的格式。 支持的数值： [订单返回类型](./enums_CN.md#orderresponsetype)
selfTradePreventionMode|ENUM   |NO        |允许的数值取决于交易对上的配置。参考 [STP 模式](./enums_CN.md#stpmodes)
workingType            |ENUM   |YES       |支持的数值： `LIMIT`， `LIMIT_MAKER`
workingSide            |ENUM   |YES       |支持的数值： [订单方向](./enums_CN.md#side)
workingClientOrderId   |STRING |NO        |用于标识生效订单的唯一ID。 <br> 如果未发送则自动生成。
workingPrice           |DECIMAL|YES       |
workingQuantity        |DECIMAL|YES       |用于设置生效订单的数量。 
workingIcebergQty      |DECIMAL|NO       |只有当 `workingTimeInForce` 为 `GTC` 时才能使用。
workingTimeInForce     |ENUM   |NO        |支持的数值： [生效时间](./enums_CN.md#timeinforce)
workingStrategyId      |LONG    |NO        |订单策略中用于标识生效订单的 ID。
workingStrategyType    |INT    |NO        |用于标识生效订单策略的任意数值。<br> 小于 `1000000` 的值被保留，无法使用。
pendingType            |ENUM   |YES       |支持的数值： <a href="#order-type">订单类型</a><br> 请注意，系统不支持使用 `quoteOrderQty` 的 `MARKET` 订单。
pendingSide            |ENUM   |YES       |支持的数值： [订单方向](./enums_CN.md#side)
pendingClientOrderId   |STRING |NO        |用于标识待处理订单的唯一ID。 <br> 如果未发送则自动生成。
pendingPrice           |DECIMAL|NO        |
pendingStopPrice       |DECIMAL|NO        |
pendingTrailingDelta   |DECIMAL|NO        |
pendingQuantity        |DECIMAL|YES       |用于设置待处理订单的数量。 
pendingIcebergQty      |DECIMAL|NO        |只有当 `pendingTimeInForce` 为 `GTC` 或者当 `pendingType` 为 `LIMIT_MAKER` 时才能使用。
pendingTimeInForce     |ENUM   |NO        |支持的数值： [生效时间](./enums_CN.md#timeinforce)
pendingStrategyId      |LONG    |NO        |订单策略中用于标识待处理订单的 ID。
pendingStrategyType    |INT    |NO        |用于标识待处理订单策略的任意数值。 <br> 小于 `1000000` 的值被保留，无法使用。
recvWindow             |LONG   |NO        |不能大于 `60000`。
timestamp              |LONG   |YES       |

<a id="mandatory-parameters-based-on-pendingtype-or-workingtype"></a>

**根据 `pendingType` 或者 `workingType` 的不同值，对于某些参数的强制要求**

根据 `pendingType` 或者`workingType`的不同值，对于某些可选参数有强制要求，具体如下：

|类型                                                   | 强制要求的参数                  | 其他信息|
|----                                                  |----                           |------  
|`workingType` = `LIMIT`                               |`workingTimeInForce`           | 
|`pendingType` = `LIMIT`                                |`pendingPrice`， `pendingTimeInForce`          |
|`pendingType` = `STOP_LOSS` 或 `TAKE_PROFIT`           |`pendingStopPrice` 与/或 `pendingTrailingDelta`|
|`pendingType` = `STOP_LOSS_LIMIT` 或 `TAKE_PROFIT_LIMIT`|`pendingPrice`， `pendingStopPrice` 与/或 `pendingTrailingDelta`， `pendingTimeInForce`|

**数据源:**
撮合引擎

**响应:**

```javascript
{
    "orderListId": 0,
    "contingencyType": "OTO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "yl2ERtcar1o25zcWtqVBTC",
    "transactionTime": 1712289389158,
    "symbol": "ABCDEF",
    "orders": [
        {
            "symbol": "LTCBTC",
            "orderId": 4,
            "clientOrderId": "Bq17mn9fP6vyCn75Jw1xya"
        },
        {
            "symbol": "LTCBTC",
            "orderId": 5,
            "clientOrderId": "arLFo0zGJVDE69cvGBaU0d"
        }
    ],
    "orderReports": [
        {
            "symbol": "LTCBTC",
            "orderId": 4,
            "orderListId": 0,
            "clientOrderId": "Bq17mn9fP6vyCn75Jw1xya",
            "transactTime": 1712289389158,
            "price": "1.00000000",
            "origQty": "1.00000000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "NEW",
            "timeInForce": "GTC",
            "type": "LIMIT",
            "side": "SELL",
            "workingTime": 1712289389158,
            "selfTradePreventionMode": "NONE"
        },
        {
            "symbol": "LTCBTC",
            "orderId": 5,
            "orderListId": 0,
            "clientOrderId": "arLFo0zGJVDE69cvGBaU0d",
            "transactTime": 1712289389158,
            "price": "0.00000000",
            "origQty": "5.00000000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "PENDING_NEW",
            "timeInForce": "GTC",
            "type": "MARKET",
            "side": "BUY",
            "workingTime": -1,
            "selfTradePreventionMode": "NONE"
        }
    ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](#conditional-fields-in-order-responses) 部分。

<a id="new-order-list---otoco-trade"></a>

#### New Order List - OTOCO (TRADE)

```
POST /api/v3/orderList/otoco
```

发送一个新的 OTOCO 订单。

* 一个 OTOCO 订单（One-Triggers-One-Cancels-the-Other）是一个包含了三个订单的订单列表。
* 第一个订单被称为**生效订单**，必须为 `LIMIT` 或 `LIMIT_MAKER` 类型的订单。最初，订单簿上只有生效订单。
    * 生效订单的行为与此一致 [OTO](#new-order-list---oto-trade)
* 一个OTOCO订单有两个待处理订单（pending above 和 pending below），它们构成了一个 OCO 订单列表。只有当生效订单**完全成交**时，待处理订单们才会被自动下单。
    * 待处理上方(pending above)订单和待处理下方(pending below)订单都遵循与 OCO 订单列表相同的规则 [Order List OCO](#new-order-list---oco-trade)。
* `OTOCO` 在未成交订单计数，`EXCHANGE_MAX_NUM_ORDERS` 过滤器和 `MAX_NUM_ORDERS` 过滤器的基础上添加**3个订单**。


**权重:** 
1

**参数:**

名称                    | 类型    | 是否必需   | 描述
----                     |----   |------    |------
symbol                   |STRING |YES       |
listClientOrderId        |STRING |NO        |整个订单列表的唯一ID。 如果未发送则自动生成。 <br> 仅当前一个订单列表已填满或完全过期时，才会接受含有相同 `listClientOrderId` 值的新订单列表。 <br>  `listClientOrderId` 与 `workingClientOrderId`， `pendingAboveClientOrderId` 以及 `pendingBelowClientOrderId` 不同。
newOrderRespType         |ENUM   |NO        |用于设置JSON响应的格式。 支持的数值： [订单返回类型](./enums_CN.md#orderresponsetype)
selfTradePreventionMode  |ENUM   |NO        |允许的数值取决于交易对上的配置。参考 [STP 模式](./enums_CN.md#stpmodes)
workingType              |ENUM   |YES       |支持的数值： `LIMIT`， `LIMIT_MAKER`
workingSide              |ENUM   |YES       |支持的数值： [订单方向](./enums_CN.md#side)
workingClientOrderId     |STRING |NO        |用于标识生效订单的唯一ID。 <br> 如果未发送则自动生成。
workingPrice             |DECIMAL|YES       |
workingQuantity          |DECIMAL|YES        |
workingIcebergQty        |DECIMAL|NO        |只有当 `workingTimeInForce` 为 `GTC` 时才能使用。
workingTimeInForce       |ENUM   |NO        |支持的数值： [生效时间](./enums_CN.md#timeinforce)
workingStrategyId        |LONG    |NO        |订单策略中用于标识生效订单的 ID。
workingStrategyType      |INT    |NO        |用于标识生效订单策略的任意数值。<br> 小于 `1000000` 的值被保留，无法使用。
pendingSide              |ENUM   |YES       |支持的数值： [订单方向](./enums_CN.md#side)
pendingQuantity          |DECIMAL|YES       |
pendingAboveType         |ENUM   |YES       |支持的数值： `STOP_LOSS_LIMIT`, `STOP_LOSS`, `LIMIT_MAKER`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`
pendingAboveClientOrderId|STRING |NO        |用于标识待处理上方订单的唯一ID。 <br> 如果未发送则自动生成。
pendingAbovePrice        |DECIMAL|NO        |当 `pendingAboveType` 是 `STOP_LOSS_LIMIT`, `LIMIT_MAKER` 或 `TAKE_PROFIT_LIMIT` 时，可用以指定限价。
pendingAboveStopPrice    |DECIMAL|NO        |如果 `pendingAboveType` 是 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT` 才能使用。
pendingAboveTrailingDelta|DECIMAL|NO        |参见 [追踪止盈止损(Trailing Stop)订单常见问题](./faqs/trailing-stop-faq_CN.md) 
pendingAboveIcebergQty   |DECIMAL|NO        |只有当 `pendingAboveTimeInForce` 为 `GTC` 时才能使用。
pendingAboveTimeInForce  |ENUM   |NO        |
pendingAboveStrategyId   |LONG    |NO        |订单策略中用于标识待处理上方订单的 ID。
pendingAboveStrategyType |INT    |NO        |用于标识待处理上方订单策略的任意数值。 <br> 小于 `1000000` 的值被保留，无法使用。
pendingBelowType         |ENUM   |NO        |支持的数值： `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`
pendingBelowClientOrderId|STRING |NO        |用于标识待处理下方订单的唯一ID。 <br> 如果未发送则自动生成。
pendingBelowPrice        |DECIMAL|NO        |当 `pendingBelowType` 是 `STOP_LOSS_LIMIT` 或 `TAKE_PROFIT_LIMIT` 时，可用以指定限价。
pendingBelowStopPrice    |DECIMAL|NO        |如果 `pendingBelowType` 是 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT` 才能使用。<br> 必须指定 `pendingBelowStopPrice` 或 `pendingBelowTrailingDelta` 或两者。
pendingBelowTrailingDelta|DECIMAL|NO        |
pendingBelowIcebergQty   |DECIMAL|NO        |只有当 `pendingBelowTimeInForce` 为 `GTC` 时才能使用。
pendingBelowTimeInForce  |ENUM   |NO        |
pendingBelowStrategyId   |LONG    |NO        |订单策略中用于标识待处理下方订单的 ID。
pendingBelowStrategyType |INT    |NO        |用于标识待处理下方订单策略的任意数值。 <br> 小于 `1000000` 的值被保留，无法使用。
recvWindow               |LONG   |NO        |不能大于 `60000`。
timestamp                |LONG   |YES       |

<a id="mandatory-parameters-based-on-pendingabovetype-pendingbelowtype-or-workingtype"></a>

**根据 `pendingAboveType`， `pendingBelowType` 或者`workingType`的不同值，对于某些参数的强制要求**

根据 `pendingAboveType`， `pendingBelowType` 或者`workingType`的不同值，对于某些可选参数有强制要求，具体如下：

|类型                                                        | 强制要求的参数                  | 其他信息|
|----                                                       |----                           |------  
|`workingType` = `LIMIT`                                    |`workingTimeInForce`           | 
|`pendingAboveType` = `LIMIT_MAKER`                                |`pendingAbovePrice`     |
|`pendingAboveType` = `STOP_LOSS/TAKE_PROFIT` | `pendingAboveStopPrice` 与/或  `pendingAboveTrailingDelta` |
|`pendingAboveType` = `STOP_LOSS_LIMIT/TAKE_PROFIT_LIMIT` | `pendingAbovePrice`， `pendingAboveStopPrice` 与/或 `pendingAboveTrailingDelta`， `pendingAboveTimeInForce` |
|`pendingBelowType` = `LIMIT_MAKER`                                |`pendingBelowPrice`          |
|`pendingBelowType` = `STOP_LOSS/TAKE_PROFIT` | `pendingBelowStopPrice` 与/或 `pendingBelowTrailingDelta` |
|`pendingBelowType` = `STOP_LOSS_LIMIT/TAKE_PROFIT_LIMIT` | `pendingBelowPrice`， `pendingBelowStopPrice` 与/或 `pendingBelowTrailingDelta`， `pendingBelowTimeInForce` |



**数据源:**
撮合引擎

**响应:**

```javascript
{
    "orderListId": 1,
    "contingencyType": "OTO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "RumwQpBaDctlUu5jyG5rs0",
    "transactionTime": 1712291372842,
    "symbol": "ABCDEF",
    "orders": [
        {
            "symbol": "LTCBTC",
            "orderId": 6,
            "clientOrderId": "fM9Y4m23IFJVCQmIrlUmMK"
        },
        {
            "symbol": "LTCBTC",
            "orderId": 7,
            "clientOrderId": "6pcQbFIzTXGZQ1e2MkGDq4"
        },
        {
            "symbol": "LTCBTC",
            "orderId": 8,
            "clientOrderId": "r4JMv9cwAYYUwwBZfbussx"
        }
    ],
    "orderReports": [
        {
            "symbol": "LTCBTC",
            "orderId": 6,
            "orderListId": 1,
            "clientOrderId": "fM9Y4m23IFJVCQmIrlUmMK",
            "transactTime": 1712291372842,
            "price": "1.00000000",
            "origQty": "1.00000000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "NEW",
            "timeInForce": "GTC",
            "type": "LIMIT",
            "side": "SELL",
            "workingTime": 1712291372842,
            "selfTradePreventionMode": "NONE"
        },
        {
            "symbol": "LTCBTC",
            "orderId": 7,
            "orderListId": 1,
            "clientOrderId": "6pcQbFIzTXGZQ1e2MkGDq4",
            "transactTime": 1712291372842,
            "price": "1.00000000",
            "origQty": "5.00000000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "PENDING_NEW",
            "timeInForce": "IOC",
            "type": "STOP_LOSS_LIMIT",
            "side": "BUY",
            "stopPrice": "6.00000000",
            "workingTime": -1,
            "selfTradePreventionMode": "NONE"
        },
        {
            "symbol": "LTCBTC",
            "orderId": 8,
            "orderListId": 1,
            "clientOrderId": "r4JMv9cwAYYUwwBZfbussx",
            "transactTime": 1712291372842,
            "price": "3.00000000",
            "origQty": "5.00000000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "PENDING_NEW",
            "timeInForce": "GTC",
            "type": "LIMIT_MAKER",
            "side": "BUY",
            "workingTime": -1,
            "selfTradePreventionMode": "NONE"
        }
    ]
}
```

**注意:** 上面的 payload 没有显示所有可以出现的字段，更多请看 [订单响应中的特定条件时才会出现的字段](#conditional-fields-in-order-responses) 部分。

#### 取消订单列表 (TRADE)

``
DELETE /api/v3/orderList 
``

取消整个订单列表。

**权重:** 1

**参数:**

名称| 类型| 是否必需| 描述
----| ----|------|------
symbol| STRING| YES|
orderListId|LONG|NO| `orderListId` 或 `listClientOrderId` 必须被提供
listClientOrderId|STRING|NO| `orderListId` 或 `listClientOrderId` 必须被提供
newClientOrderId|STRING|NO| 用户自定义的本次撤销操作的ID(注意不是被撤销的订单的自定义ID)。如无指定会自动赋值。
recvWindow|LONG|NO|不能大于 `60000`
timestamp|LONG|YES|

其他注意点:

* 取消订单列表中的单个订单将取消整个订单列表.
* 如果 `orderListId` 和 `listClientOrderId` 一起发送, `orderListId` 优先被考虑.


**数据源:**
撮合引擎

**响应:**

```javascript
{
  "orderListId": 0,
  "contingencyType": "OCO",
  "listStatusType": "ALL_DONE",
  "listOrderStatus": "ALL_DONE",
  "listClientOrderId": "C3wyj4WVEktd7u9aVBRXcN",
  "transactionTime": 1574040868128,
  "symbol": "LTCBTC",
  "orders": [
    {
      "symbol": "LTCBTC",
      "orderId": 2,
      "clientOrderId": "pO9ufTiFGg3nw2fOdgeOXa"
    },
    {
      "symbol": "LTCBTC",
      "orderId": 3,
      "clientOrderId": "TXOvglzXuaubXAaENpaRCB"
    }
  ],
  "orderReports": [
    {
      "symbol": "LTCBTC",
      "origClientOrderId": "pO9ufTiFGg3nw2fOdgeOXa",
      "orderId": 2,
      "orderListId": 0,
      "clientOrderId": "unfWT8ig8i0uj6lPuYLez6",
      "transactTime": 1688005070874,
      "price": "1.00000000",
      "origQty": "10.00000000",
      "executedQty": "0.00000000",
      "origQuoteOrderQty": "0.000000",
      "cummulativeQuoteQty": "0.00000000",
      "status": "CANCELED",
      "timeInForce": "GTC",
      "type": "STOP_LOSS_LIMIT",
      "side": "SELL",
      "stopPrice": "1.00000000"
    },
    {
      "symbol": "LTCBTC",
      "origClientOrderId": "TXOvglzXuaubXAaENpaRCB",
      "orderId": 3,
      "orderListId": 0,
      "clientOrderId": "unfWT8ig8i0uj6lPuYLez6",
      "transactTime": 1688005070874,
      "price": "3.00000000",
      "origQty": "10.00000000",
      "executedQty": "0.00000000",
      "origQuoteOrderQty": "0.000000",
      "cummulativeQuoteQty": "0.00000000",
      "status": "CANCELED",
      "timeInForce": "GTC",
      "type": "LIMIT_MAKER",
      "side": "SELL"
    }
  ]
}
```

#### 查询订单列表 (USER_DATA)

``
GET /api/v3/orderList 
``

根据提供的可选参数检索特定的订单列表。

**权重:** 4

**参数:**

名称| 类型|是否必需| 描述
----|-----|----|----------
orderListId|LONG|NO|   ```orderListId``` 或 ```origClientOrderId``` 必须提供一个。
origClientOrderId|STRING|NO|  ```orderListId``` 或 ```origClientOrderId``` 必须提供一个。
recvWindow|LONG|NO| 赋值不得大于 ```60000```
timestamp|LONG|YES|

**数据源:**
数据库

**响应:**

```javascript
{
    "orderListId": 27,
    "contingencyType": "OCO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "h2USkA5YQpaXHPIrkd96xE",
    "transactionTime": 1565245656253,
    "symbol": "LTCBTC",
    "orders": [
        {
            "symbol": "LTCBTC",
            "orderId": 4,
            "clientOrderId": "qD1gy3kc3Gx0rihm9Y3xwS"
        },
        {
            "symbol": "LTCBTC",
            "orderId": 5,
            "clientOrderId": "ARzZ9I00CPM8i3NhmU9Ega"
        }
    ]
}
```

#### 查询所有订单列表 (USER_DATA)

``
GET /api/v3/allOrderList 
``

根据提供的可选参数检索所有的订单列表。

请注意，`startTime`和`endTime`之间的时间不能超过 24 小时。

**权重:** 20

**参数:**

名称|类型| 是否必需| 描述
----|----|----|---------
fromId|LONG|NO| 提供该项后, `startTime` 和 `endTime` 都不可提供
startTime|LONG|NO|
endTime|LONG|NO|
limit|INT|NO| 默认值: 500; 最大值: 1000
recvWindow|LONG|NO| 赋值不能超过 `60000`
timestamp|LONG|YES|

**数据源:**
数据库

**响应:**

```javascript
[
  {
    "orderListId": 29,
    "contingencyType": "OCO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "amEEAXryFzFwYF1FeRpUoZ",
    "transactionTime": 1565245913483,
    "symbol": "LTCBTC",
    "orders": [
      {
        "symbol": "LTCBTC",
        "orderId": 4,
        "clientOrderId": "oD7aesZqjEGlZrbtRpy5zB"
      },
      {
        "symbol": "LTCBTC",
        "orderId": 5,
        "clientOrderId": "Jr1h6xirOxgeJOUuYQS7V3"
      }
    ]
  },
  {
    "orderListId": 28,
    "contingencyType": "OCO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "hG7hFNxJV6cZy3Ze4AUT4d",
    "transactionTime": 1565245913407,
    "symbol": "LTCBTC",
    "orders": [
      {
        "symbol": "LTCBTC",
        "orderId": 2,
        "clientOrderId": "j6lFOfbmFMRjTYA7rRJ0LP"
      },
      {
        "symbol": "LTCBTC",
        "orderId": 3,
        "clientOrderId": "z0KCjOdditiLS5ekAFtK81"
      }
    ]
  }
]
```

#### 查询订单列表挂单 (USER_DATA)

``
GET /api/v3/openOrderList
``

**权重:** 6

**参数:**

名称| 类型|是否必需| 描述
----|-----|---|------------------
recvWindow|LONG|NO| 赋值不能大于 ```60000```
timestamp|LONG|YES|


**数据源:**
数据库

**响应:**

```javascript
[
  {
    "orderListId": 31,
    "contingencyType": "OCO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "wuB13fmulKj3YjdqWEcsnp",
    "transactionTime": 1565246080644,
    "symbol": "LTCBTC",
    "orders": [
      {
        "symbol": "LTCBTC",
        "orderId": 4,
        "clientOrderId": "r3EH2N76dHfLoSZWIUw1bT"
      },
      {
        "symbol": "LTCBTC",
        "orderId": 5,
        "clientOrderId": "Cv1SnyPD3qhqpbjpYEHbd2"
      }
    ]
  }
]
```

### SOR

<a id="sor-order"></a>

#### 下 SOR 订单 (TRADE)

```
POST /api/v3/sor/order
```
发送使用智能订单路由 (SOR) 的新订单。

**权重:**
1

**参数:**

名称| 类型|是否必需| 描述
------------            | -----  | ------------ | ------------
symbol                  | STRING | YES |
side                    | ENUM   | YES |
type                    | ENUM   | YES |
timeInForce             | ENUM   | NO |
quantity                | DECIMAL| YES |
price                   | DECIMAL| NO |
newClientOrderId        | STRING | NO | 用户自定义的orderid，如空缺系统会自动赋值。如果几个订单具有相同的 `newClientOrderID` 赋值，<br/>那么只有在前一个订单成交后才可以接受下一个订单，否则该订单将被拒绝。
strategyId              |LONG     | NO|
strategyType            |INT     | NO| 赋值不能小于 `1000000`.
icebergQty              | DECIMAL| NO | 仅有限价单可以使用该参数，含义为创建冰山订单并指定冰山订单的数量。
newOrderRespType        | ENUM   | NO | 指定响应类型:
指定响应类型 `ACK`, `RESULT` 或 `FULL`; 默认为 `FULL`。
selfTradePreventionMode |ENUM    | NO | 允许的 ENUM 取决于交易对的配置。支持的值有：[STP 模式](./enums_CN.md#stpmodes)。
recvWindow              | LONG   | NO | 赋值不能大于 `60000`
timestamp               | LONG | YES |

**请注意:** `POST /api/v3/sor/order` 只支持 `限价` 和 `市场` 单， 并不支持 `quoteOrderQty`。

**数据源:**
撮合引擎

**响应:**

```javascript
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
  "workingFloor": "SOR",              
  "selfTradePreventionMode": "NONE",
  "usedSor": true
}
```

#### 测试 SOR 下单接口 (TRADE)

```
POST /api/v3/sor/order/test
```

用于测试使用智能订单路由 (SOR) 的订单请求，但不会提交到撮合引擎


**权重:**
| 条件 | 请求权重 |
| --------- | -------------- |
| 没有 `computeCommissionRates`  |  1 |
| 有 `computeCommissionRates`     | 20 |


**参数:**

除了 [`POST /api/v3/sor/order`](#sor-order) 所有参数,
如下参数也接受:

参数名                   |类型          | 是否必需    | 描述
------------           | ------------ | ------------ | ------------
computeCommissionRates | BOOLEAN      | NO            | 默认值: `false`

**数据源:**
缓存

**响应:**

没有 `computeCommissionRates`

```
{}
```

有 `computeCommissionRates`


```javascript
{
  "standardCommissionForOrder": {  // 订单交易的标准佣金率。
    "maker": "0.00000112",
    "taker": "0.00000114",
  },
  "taxCommissionForOrder": {       // 订单交易的税率。
    "maker": "0.00000112",
    "taker": "0.00000114",
  },
  "discount": {                    // 以BNB支付时的标准佣金折扣。
    "enabledForAccount": true,
    "enabledForSymbol": true,
    "discountAsset": "BNB",
    "discount": "0.25000000"       // 当用BNB支付佣金时，在标准佣金上按此比率打折。
  }
}
```


## 账户接口

### 账户信息 (USER_DATA)
```
GET /api/v3/account 
```

**权重:**
20

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
omitZeroBalances |BOOLEAN| NO | 如果`true`，将隐藏所有零余额。 <br>默认值：`false`
recvWindow | LONG | NO |
timestamp | LONG | YES |

**数据源:**
缓存 => 数据库

**响应:**
```javascript
{
  "makerCommission": 15,
  "takerCommission": 15,
  "buyerCommission": 0,
  "sellerCommission": 0,
  "commissionRates": {
    "maker": "0.00150000",
    "taker": "0.00150000",
    "buyer": "0.00000000",
    "seller": "0.00000000"
  },
  "canTrade": true,
  "canWithdraw": true,
  "canDeposit": true,
  "brokered": false,
  "requireSelfTradePrevention": false,
  "preventSor": false,
  "updateTime": 123456789,
  "accountType": "SPOT",
  "balances": [
    {
      "asset": "BTC",
      "free": "4723846.89208129",
      "locked": "0.00000000"
    },
    {
      "asset": "LTC",
      "free": "4763368.68006011",
      "locked": "0.00000000"
    }
  ],
  "permissions": [
    "SPOT"
  ],
  "uid": 354937868
}
```

### 账户成交历史 (USER_DATA)
```
GET /api/v3/myTrades  
```
获取某交易对的成交历史

**权重:**
20

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId|LONG|NO| 必须要和参数`symbol`一起使用.
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO |返回该fromId之后的成交，缺省返回最近的成交
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |


**备注:**
* 如果设置了`fromId`, 会返回ID大于此`fromId`的交易. 不然则会返回最近的交易.
* `startTime`和`endTime`设置的时间间隔不能超过24小时.
* 支持的所有参数组合:
  * `symbol`
  * `symbol` + `orderId`
  * `symbol` + `startTime`
  * `symbol` + `endTime`
  * `symbol` + `fromId`
  * `symbol` + `startTime` + `endTime`
  * `symbol`+ `orderId` + `fromId`

**数据源:**
数据库

**响应:**
```javascript
[
  {
    "symbol": "BNBBTC",
    "id": 28457,
    "orderId": 100234,
    "price": "4.00000100",
    "qty": "12.00000000",
    "commission": "10.10000000",
    "commissionAsset": "BNB",
    "time": 1499865549590,
    "isBuyer": true,
    "isMaker": false,
    "isBestMatch": true
  }
]
```

<a id="query-unfilled-order-count"></a>

### 查询未成交的订单计数 (USER_DATA)
```
GET /api/v3/rateLimit/order
```
显示用户在所有时间间隔内的未成交订单计数。

**权重:**
40

**参数:**
名称 | 类型| 是否必需 | 描述
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO | 赋值不得大于 `60000`
timestamp | LONG | YES |

**数据源:**
缓存

**响应:**
```javascript
[
  {
    "rateLimitType": "ORDERS",
    "interval": "SECOND",
    "intervalNum": 10,
    "limit": 10000,
    "count": 0
  },
  {
    "rateLimitType": "ORDERS",
    "interval": "DAY",
    "intervalNum": 1,
    "limit": 20000,
    "count": 0
  }
]
```


### 获取 Prevented Matches (USER_DATA)

```
GET /api/v3/myPreventedMatches
```

获取因 STP 而过期的订单列表。

这些是支持的组合：

* `symbol` + `preventedMatchId`
* `symbol` + `orderId`
* `symbol` + `orderId` + `fromPreventedMatchId` (`limit` 默认为 500)
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

**权重:**

情况                         | 权重
----------------------------| -----
如果 `symbol` 是无效的        | 2
通过 `preventedMatchId` 查询 | 2
通过 `orderId` 查询          | 20

**数据源:**

数据库

**响应:**

```json
[
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
]
```

### 查询分配结果 (USER_DATA)

```
GET /api/v3/myAllocations
```

检索由 SOR 订单生成引起的分配结果。

**权重:**
20

**参数:**

名称                 | 类型   | 是否必需         | 描述
------------        | ----   | ------------ | ------------
symbol                   |STRING |Yes        |
startTime                |LONG   |No        |
endTime                  |LONG   |No        |
fromAllocationId         |INT    |No        |
limit                    |INT    |No        |默认值 500； 最大值 1000
orderId                  |LONG   |No        |
recvWindow               |LONG   |No        |不能大于 `60000`
timestamp                |LONG   |No        |

支持的参数组合:

参数                                  | 响应 |
------------------------------------------- | -------- |
`symbol`                                    | 按从最旧到最新排序的分配 |
`symbol` + `startTime`                      | 从 `startTime` 开始的最旧的分配 |
`symbol` + `endTime`                        | 到 `endTime` 为止的最新的分配 |
`symbol` + `startTime` + `endTime`          | 在指定时间范围内的分配 |
`symbol` + `fromAllocationId`               | 从指定 `AllocationId` 开始的分配  |
`symbol` + `orderId`                        | 按从最旧到最新排序并和特定订单关联的分配 |
`symbol` + `orderId` + `fromAllocationId`   | 从指定 `AllocationId` 开始并和特定订单关联的分配 |

**注意:** `startTime` 和 `endTime` 之间的时间不能超过 24 小时。

**数据源:**
数据库

**响应:**

```javascript
[
  {
    "symbol": "BTCUSDT",
    "allocationId": 0,
    "allocationType": "SOR",
    "orderId": 1,
    "orderListId": -1,
    "price": "1.00000000",
    "qty": "5.00000000",
    "quoteQty": "5.00000000",
    "commission": "0.00000000",
    "commissionAsset": "BTC",
    "time": 1687506878118,
    "isBuyer": true,
    "isMaker": false,
    "isAllocator": false
  }
]
```


### 查询佣金费率 (USER_DATA)

```
GET /api/v3/account/commission
```

获取当前账户的佣金费率。


**权重:**
20

**参数:**

参数名         | 类型    | 是否必需 | 描述
------------ | -----   | ------------ | ------------
symbol        | STRING | YES          |

**数据源:**
数据库

**响应:**

```javascript
{
  "symbol": "BTCUSDT",
  "standardCommission": {          // 订单交易的标准佣金率。
    "maker": "0.00000010",
    "taker": "0.00000020",
    "buyer": "0.00000030",
    "seller": "0.00000040" 
  },
  "taxCommission": {              // 订单交易的税率。
    "maker": "0.00000112",
    "taker": "0.00000114",
    "buyer": "0.00000118",
    "seller": "0.00000116" 
  },
  "discount": {                   // 使用BNB支付时的佣金折扣。
    "enabledForAccount": true,
    "enabledForSymbol": true,
    "discountAsset": "BNB",
    "discount": "0.7500000"       // 当用BNB支付佣金时，在标准佣金上按此比率打折。
  }
}
```


## 用户数据流订阅接口
此处仅列出如何得到数据流名称及如何维持有效期的接口，具体订阅方式参考另一篇websocket接口文档

### 新建用户数据流 (USER_STREAM)
```
POST /api/v3/userDataStream
```
从创建起60分钟有效

**权重:**
2

**参数:**
NONE

**数据源:**
缓存


**响应:**
```javascript
{
  "listenKey": "pqia91ma19a5s61cv6a81va65sdf19v8a65a1a5s61cv6a81va65sdf19v8a65a1" // 用于订阅的数据流名
}
```

### Keepalive (USER_STREAM)
```
PUT /api/v3/userDataStream
```
延长用户数据流有效期到60分钟之后。 建议每30分钟调用一次

**权重:**
2

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES

**数据源:**
缓存

**响应:**
```javascript
{}
```

### 关闭用户数据流 (USER_STREAM)
```
DELETE /api/v3/userDataStream
```
关闭用户数据流。

**权重:**
2

**参数:**

名称 | 类型 | 是否必需 | 描述
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES

**数据源:**
缓存

**响应:**
```javascript
{}
```
