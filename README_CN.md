# 关于本中文翻译版
* 中文文档由英文文档翻译而来，当中文文档内容与英文文档冲突时，以英文文档为准
# 币安API文档
* 币安API英文Telegram群 **https://t.me/binance_api_english**
* 币安API中文Telegram群 **https://t.me/binance_api_chinese**
* 所有于本文档内给出定义的包括但不限于接口、数据流、参数、响应等，可认为是币安官方提供的内容。
* 而所有未于本文档内给出的内容，币安均不承诺提供任何支持。

文档名 | 描述
------------ | ------------
[rest-api_CN.md](./rest-api_CN.md) | 通用Rest接口定义 (/api)
[errors_CN.md](./errors_CN.md) | 错误代码及含义
[web-socket-streams_CN.md](./web-socket-streams_CN.md) | 行情数据流接口的描述
[user-data-stream_CN.md](./user-data-stream_CN.md) | 用户数据流接口的描述
[Wallet, Sub-account](https://binance-docs.github.io/apidocs/spot/cn) | 钱包，子账户接口的描述(/sapi)
[Margin, BLVT](https://binance-docs.github.io/apidocs/spot/cn) | 杠杆，杠杆代币接口的描述(/sapi)
[Mining](https://binance-docs.github.io/apidocs/spot/cn) | 矿池接口的描述(/sapi)
[BSwap, Savings](https://binance-docs.github.io/apidocs/spot/cn) | 流动性挖矿，币安宝接口的描述(/sapi)
[USDT-M Futures](https://binance-docs.github.io/apidocs/futures/cn/) | U本位合约相关API (/fapi)
[COIN-M Futures](https://binance-docs.github.io/apidocs/delivery/cn/) | 币本位合约相关API (/dapi)

# 常见问题


名称 | 描述
------------ | ------------
[spot_glossary_cn.md](./faqs/spot_glossary_cn.md) | 现货交易API术语表
[trailing-stop-faq-cn.md](./faqs/trailing-stop-faq-cn.md)   | 追踪止盈止损订单(Trailing Stop)详细信息和常见问题


# 相关信息

* [Postman Collections](https://github.com/binance/binance-api-postman)
    * 现在你可以通过Postman collection来快速体验、使用API接口.
* Connector
    * 一个轻量级的代码库，提供让用户直接调用API的方法。
    * 支持所有现货的接口.
    * [Python](https://github.com/binance/binance-connector-python)
    * [Node.js](https://github.com/binance/binance-connector-node)
    * [Ruby](https://github.com/binance/binance-connector-ruby)
    * [DotNET](https://github.com/binance/binance-connector-dotnet)
    * [Java](https://github.com/binance/binance-connector-java)
    * [Rust](https://github.com/binance/binance-spot-connector-rust)
    * [PHP](https://github.com/binance/binance-connector-php)
* [Swagger](https://github.com/binance/binance-api-swagger)
    * 一个基于OpenAPI规范的RESTful API接口定义的YAML文件，还有便于交互的 Swagger UI 页面。
* [Spot Testnet](https://testnet.binance.vision/)
    * 用户可以使用现货的测试网来体验SPOT交易.
    * 现在只能通过API来测试交易, 没有UI.
    * 只支持以 `/api/*`开头的接口, `/sapi/*`接口不支持.

# 联系我们

* [Binance API 中文电报群](https://t.me/Binance_api_Chinese)
    * 适合咨询关于API或者Websocket性能方面的疑问和困扰.
    * 可以咨询在文档上面没有的API问题.
* [Binance 开发者论坛](https://dev.binance.vision/)
    * 可以提问或者咨询关于API或者Websocket代码方面问题.
* [Binance 客服](https://www.binance.com/zh-CN/support-center)
    * 咨询关于账户, 资金， 2FA等问题.
