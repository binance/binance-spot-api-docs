# 关于本中文翻译版
* 中文文档由英文文档翻译而来，当中文文档内容与英文文档冲突时，以英文文档为准。

# 币安API文档
* 有关 API 和数据流的更改、停机等官方公告将在此处发布：**https://t.me/binance_api_announcements**
* 所有于本文档内给出定义的包括但不限于接口，数据流，参数，响应等，可认为是币安官方提供的内容。
* 而所有未于本文档内给出的内容，币安均不承诺提供任何支持。

文档名 | 描述
------------ | ------------
[errors_CN.md](./errors_CN.md)     | 现货 API 的错误代码及含义
[filters_CN.md](./filters_CN.md)   | 现货 API 使用的过滤器的详细信息
[rest-api_CN.md](./rest-api_CN.md) | 现货 Rest API 接口定义 (`/api`)
[web-socket-api_CN.md](./web-socket-api_CN.md)         | 现货 Websocket API
[web-socket-streams_CN.md](./web-socket-streams_CN.md) | 现货行情数据流接口的描述
[user-data-stream_CN.md](./user-data-stream_CN.md)     | 现货用户数据流接口的描述
[sbe_schemas](./sbe/schemas/) | 现货API的简单二进制编码 (SBE)模式 (Schema)
&#x0020; |
[Wallet, Sub-account](https://binance-docs.github.io/apidocs/spot/cn) | 钱包，子账户接口的描述(`/sapi`)
[Margin, BLVT](https://binance-docs.github.io/apidocs/spot/cn) | 杠杆，杠杆代币接口的描述(`/sapi`)
[Mining](https://binance-docs.github.io/apidocs/spot/cn) | 矿池接口的描述(`/sapi`)
[BSwap, Savings](https://binance-docs.github.io/apidocs/spot/cn) | 流动性挖矿，币安宝接口的描述(`/sapi`)
[USDT-M Futures](https://binance-docs.github.io/apidocs/futures/cn/) | U本位合约相关API (`/fapi`)
[COIN-M Futures](https://binance-docs.github.io/apidocs/delivery/cn/) | 币本位合约相关API (`/dapi`)

# 常见问题


名称 | 描述
------------ | ------------
[spot_glossary_cn](./faqs/spot_glossary_cn.md) | 现货交易 API 术语表
[commissions_faq_cn](./faqs/commissions_faq_cn.md) | 解释 API 上所使用的佣金计算方式
[trailing_stop_faq_cn](./faqs/trailing-stop-faq-cn.md)   | 追踪止盈止损订单(Trailing Stop)详细信息和常见问题
[stp_faq_cn](./faqs/stp_faq_cn.md) | 关于 Self Trade Prevention (STP) 的详细信息
[market_data_only_cn](./faqs/market_data_only_cn.md) | 仅提供市场数据的 API 和 Websocket Streams
[sor_faq](./faqs/sor_faq_cn.md) | 智能指令路由 (SOR)
[order_count_decrement](./faqs/order_count_decrement_cn.md) | 现货下单频率限制的更新
[sbe_faq](./faqs/sbe_faq_cn.md) | 关于在 API 上实施简单二进制编码 (SBE) 的信息

# 更新日志

关于API(包括REST和WebSocket)方面的最新变动，请参考 [更新日志](./CHANGELOG_CN.md)。


# 相关信息

* [Postman Collections](https://github.com/binance/binance-api-postman)
    * 现在你可以通过使用 Postman Collections 来快速体验和使用 API 接口。
* Connectors
    * 以下是以不同编程语言编写的轻量代码库，可作为连接到币安公共 API 的连接器：
        * [Python](https://github.com/binance/binance-connector-python)
        * [Node.js](https://github.com/binance/binance-connector-node)
        * [Ruby](https://github.com/binance/binance-connector-ruby)
        * [DotNET C#](https://github.com/binance/binance-connector-dotnet)
        * [Java](https://github.com/binance/binance-connector-java)
        * [Rust](https://github.com/binance/binance-spot-connector-rust)
        * [PHP](https://github.com/binance/binance-connector-php)
        * [Go](https://github.com/binance/binance-connector-go)
        * [TypeScript](https://github.com/binance/binance-connector-typescript)
* [Swagger](https://github.com/binance/binance-api-swagger)
    * 一个基于 OpenAPI 规范的 RESTful API 接口定义的 YAML 文件，还有便于交互的 Swagger UI 页面。
* [Spot Testnet](https://testnet.binance.vision/)
    * 用户可以使用现货的测试网来体验 SPOT 交易。
    * 现在只能通过 API 来测试交易，没有 UI。
    * 只支持以 `/api/*`开头的接口，`/sapi/*`接口不支持。

# 联系我们

* [Binance API 中文电报群](https://t.me/binance_api_chinese) 或 [Binance API 英文电报群](https://t.me/binance_api_english)
    * 适合咨询关于 API 或者 Websocket 性能方面的疑问和困扰。
    * 可以咨询在文档上面没有的 API 问题。
* [Binance 开发者论坛](https://dev.binance.vision/)
    * 可以提问或者咨询关于 API 或者 Websocket 代码方面问题。
* [Binance 客服](https://www.binance.com/zh-CN/support-center)
    * 咨询关于账户，资金，2FA 等问题。