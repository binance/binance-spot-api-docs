# 简单二进制编码 （SBE） 常见问题

本文档的目标是解释下列疑问:

* 如何在现货交易API中启用 `SBE` 响应。
* 如何对 `SBE` 的响应进行解码。

SBE 是一种用于实现低延迟的序列化格式。

本实现是基于 `FIX SBE` 规范。
* [GitHub repository](https://github.com/FIXTradingCommunity/fix-simple-binary-encoding)
* [HTML document](https://www.fixtrading.org/standards/sbe-online)

## 如何获取 SBE 响应

### REST API:

* `Accept` 报文头必须包含 `application/sbe`。
* 在 `X-MBX-SBE` 报文头中以 `<ID>:<VERSION>` 的形式提供 `schema ID` 和 `version`。

样本请求(REST):

```
curl -sX GET -H "Accept: application/sbe" -H "X-MBX-SBE: 1:0" 'https://api.binance.com/api/v3/exchangeInfo?symbol=BTCUSDT'
```

**注意：**

* 如果你只在 `Accept` 报文头中提供了 `application/sbe`
  	* 如果交易所不支持 `SBE`，你将收到一个**406 不可接受**的响应。
	* 如果在 `X-MBX-SBE` 报文头中提供的 XML 模式是属于格式错误或不正确的情况，那你得到的响应将会是一个 `SBE` 解码错误。
	* 如果 `X-MBX-SBE` 报文头缺失，那你得到的响应将会是一个 `SBE` 解码错误。
* 如果你在 `Accept` 报文头中同时提供了 `application/sbe` 和 `application/json`：
  	* 如果交易所不支持 `SBE`，那么响应将会被回退到 `JSON`。
	* 如果在 `X-MBX-SBE` 报文头中提供的 XML 模式是属于格式错误或不正确的情况，那么响应将会被回退到 `JSON`。
	* 如果 `X-MBX-SBE` 报文头缺失，那么响应将会被回退到 `JSON`。	


### WebSocket API:

* 在请求的URL中添加 `responseFormat=sbe`。
* 添加schema ID 和 version 到参数 `sbeSchemaId=<SCHEMA_ID>` 和 `sbeSchemaVersion=<SCHEMA_VERSION>`。

样本请求 (WebSocket):

```bash
id=$(date +%s%3N)
method="exchangeInfo"
params='{"symbol":"BTCUSDT"}'

request=$( jq -n \
        --arg id "$id" \
        --arg method "$method" \
        --argjson params "$params" \
        '{id: $id, method: $method, params: $params}' )

response=$(echo $request | websocat -n1 'wss://ws-api.binance.com:443/ws-api/v3?responseFormat=sbe&sbeSchemaId=1&sbeSchemaVersion=0')
```

**注意：**

* 如果你只在连接URL中添加 `responseFormat=sbe` :
    * 如果交易所没有开启 SBE，请求返回 HTTP 400.
    * 如果 `sbeSchemaId=<SCHEMA_ID>` 或者 `sbeSchemaVersion=<SCHEMA_VERSION>` 格式不正确或者无效，请求返回 HTTP 400.
* 如果你同时提供 `responseFormat=sbe` 和 `responseFormat=json`, 请求返回 HTTP 400.
* 在HTTP握手期间的所有错误响应都编码为JSON，`Content-Type`头设置为`application/json;charset=UTF-8`.
* 一旦成功建立了启用了SBE的WebSocket会话，在该会话中的所有方法响应都编码为SBE，即使在SBE被禁用的情况下也是如此。
    * 这意味着，如果在您的WebSocket连接处于活动状态时禁用了SBE，那么在对您的后续请求做出响应时，您将会收到一个被SBE编码了的“SBE未启用”错误。
* 就目前而言，我们不建议使用`websocat`发送任何请求，因为我们观察到了它解码二进制帧的问题。上面的样本仅用作参考，显示获取SBE响应的URL。


## 支持的 APIs

目前现货交易的 REST API 和 WebSocket API 支持 `SBE`。

## SBE 模式

* 将被使用的模式 (schema) 会被保存在此仓库 (repository) 中，[请看这里](https://github.com/binance/binance-spot-api-docs/tree/master/sbe/schemas)。
* 对于模式的任何更新将会被记录在[更改日志](../CHANGELOG_CN.md)中。

**关于对旧版本的支持：**

* SBE 模式通过两个 XML 属性进行版本控制，`id` 和 `version`。
	* 当引入破坏性更改时，`id` 会增加。当这种情况发生时，`version` 会被重置为0。
	* 当引入非破坏性更改时，`version` 会增加。当这种情况发生时，`id` 不会被修改。
* 当新模式发布时，旧模式会被废止。 **即便新模式只引入非破坏性更改，这种情况也会导致废止。**
* 已废止的模式将在被废止后**依然得到至少6个月的支持**。<br>用以下这个假设的时间线为例:
	* 3024年1月：发布模式 id 1 version 0。这是第一版，一旦交易所启用 `SBE`，用户就立刻可以开始使用该模式。
	* 3024年3月：发布模式 id 1 version 1。这个模式引入了一个非破坏性的变化。
		* 模式 id 1 version 0 此时已被废止，但还可以至少再被使用6个月。
	* 3024年8月：发布模式 id 2 version 0。这个模式引入了一个破坏性的变化。
		* 模式 id 1 version 0 已被废止，但还可以再被使用至少1个月。
		* 模式 id 1 version 1 此时已被废止，但还可以再被使用至少6个月。
	* 3024年9月：自模式 id 1 version 1 发布以来已经过去6个月。
  		* 模式 id 1 version 0 已被停用。
	* 3025年2月：发布模式id 2 版本 1。这个模式引入了一个非破坏性的变化。
		* 模式 id 1 version 1 已被停用。
		* 模式 id 2 version 0 此时已被废止，但还可以再被使用至少另外6个月。
* HTTP将在针对 `X-MBX-SBE header` 中已被废止的 `SBE` 模式版本请求的响应中包含一个 `X-MBX-SBE-DEPRECATED` 报文头 。
* 对于WebSocket响应，如果在其连接URL中指定了已弃用的`sbeSchemaId`和`sbeSchemaVersion`，`sbeSchemaIdVersionDeprecated`字段将被设置为`true`。
* 指定已废止的`<ID>:<VERSION>`（REST API）或`sbeSchemaId`和`sbeSchemaVersion` （WebSocket API）的请求将会返回HTTP 400错误。
* 关于模式生命周期的 `JSON` 文件将被保存在此仓库中，[请看这里](https://github.com/binance/binance-spot-api-docs/tree/master/sbe/schemas)。这个文件包含了关于实时交易所和现货测试网的最新、被废止和被停用模式的具体发生日期。<br> 以下是一个基于上述假设时间线的 `JSON` 示例：

```json
{
    "environment": "PROD",
    "latestSchema": {
        "id": 2,
        "version": 1,
        "releaseDate": "3025-02-01" 
    },
    "deprecatedSchemas": [
        {
            "id": 2,
            "version": 0,
            "releaseDate": "3024-08-01",
            "deprecatedDate": "3025-02-01" 
        }
    ],
    "retiredSchemas": [
        {
            "id": 1,
            "version": 1,
            "releaseDate": "3024-03-01",
            "deprecatedDate": "3024-08-01", 
            "retiredDate": "3025-02-01",
        },
        {
            "id": 1,
            "version": 0,
            "releaseDate": "3024-01-01",
            "deprecatedDate": "3024-03-01",
            "retiredDate": "3024-09-01",
        }
    ]
}
```

## 生成解码器：

1. 下载模式：
    * [`spot_prod_latest.xml`](../sbe/schemas/spot_prod_latest.xml) 适用于实时交易所。
    * [`spot_testnet_latest.xml`](../sbe/schemas/spot_testnet_latest.xml) 适用于 [现货测试网](https://testnet.binance.vision)。
2. 克隆并构建 [`simple-binary-encoding`](https://github.com/real-logic/simple-binary-encoding)：
```shell
 $ git clone https://github.com/real-logic/simple-binary-encoding.git
 $ cd simple-binary-encoding
 $ ./gradlew
```
3. 运行 `SbeTool` 代码生成器。（请参考这里分别使用[Java](https://github.com/binance/binance-sbe-java-sample-app), [C++](https://github.com/binance/binance-sbe-cpp-sample-app) 和 [Rust](https://github.com/binance/binance-sbe-rust-sample-app) 解码交易所信息 payload 的样本。）

### 十进制字段编码

不同于 `FIX SBE` 的规范，十进制字段的尾数和指数字段被单独编码为原始字段，以便使负载量和消息内编码字段的数量达到最小化。

### 时间戳字段编码

SBE响应中的时间戳(Timestamps)是以微秒为单位。这与包含毫秒时间戳的JSON响应不同。

### 模式文件中的自定义字段属性

在模式文件中添加了一些以 `mbx:` 为前缀的字段属性，供文档使用：
- `mbx:exponent`：指向对应于尾数字段的指数域
- `mbx:jsonPath`：包含了 `JSON` 响应中相应字段的名称
- `mbx:jsonValue`: 包含了 `JSON` 响应中等价 `ENUM` 值的名称



