# 简单二进制编码 （SBE） 常见问题

本文档的目标是解释下列疑问:

* 如何在现货交易API中启用 `SBE` 响应。
* 如何对 `SBE` 的响应进行解码。

SBE 是一种用于实现低延迟的序列化格式。

本实现是基于 `FIX SBE` 规范。
* [GitHub repository](https://github.com/FIXTradingCommunity/fix-simple-binary-encoding)
* [HTML document](https://www.fixtrading.org/standards/sbe-online)

## 如何获取 SBE 响应

* `Accept` 报文头必须包含 `application/sbe`。
* 在 `X-MBX-SBE` 报文头中以 `<ID>:<VERSION>` 的形式提供 `schema ID` 和 `version`。

样本请求：

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

## 支持的 APIs

目前只有用于现货交易的 REST API 支持 `SBE`。

## SBE 模式

* 将被使用的模式 (schema) 会被保存在此仓库 (repository) 中。
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
* 如果请求指定的 `<ID>:<VERSION>` 已被停用，请求将以 `HTTP 400` 失败而告终。
* 关于模式生命周期的 `JSON` 文件将被保存在此仓库中。这个文件包含了关于实时交易所和现货测试网的最新、被废止和被停用模式的具体发生日期。<br> 以下是一个基于上述假设时间线的 `JSON` 示例：

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

1. 下载模式 ([`spot_latest.xml`](../sbe/schemas/spot_latest.xml))
2. 克隆并构建 [`simple-binary-encoding`](https://github.com/real-logic/simple-binary-encoding)：
```shell
 $ git clone https://github.com/real-logic/simple-binary-encoding.git
 $ cd simple-binary-encoding
 $ ./gradlew
```
3. 运行 `SbeTool` 代码生成器。（请参考这里分别使用[Java](https://github.com/binance/binance-sbe-java-sample-app) 和 [C++](https://github.com/binance/binance-sbe-cpp-sample-app) 解码交易所信息 payload 的样本。）

### 十进制字段编码

不同于 `FIX SBE` 的规范，十进制字段的尾数和指数字段被单独编码为原始字段，以便使负载量和消息内编码字段的数量达到最小化。

### 模式文件中的自定义字段属性

在模式文件中添加了一些以 `mbx:` 为前缀的字段属性，供文档使用：
- `mbx:jsonPath`：包含了 `JSON` 响应中相应字段的名称
- `mbx:exponent`：指向对应于尾数字段的指数域
- `mbx:jsonValue`: 包含了 `JSON` 响应中等价 `ENUM` 值的名称



