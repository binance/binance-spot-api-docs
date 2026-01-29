# 关于本中文翻译版
* 中文文档由英文文档翻译而来，当中文文档内容与英文文档冲突时，以英文文档为准。

### 币安API文档
* 有关 API 和数据流的更改、停机等官方公告将在此处发布：**https://t.me/binance_api_announcements**
* 所有于本文档内给出定义的包括但不限于接口，数据流，参数，响应等，可认为是币安官方提供的内容。
* 而所有未于本文档内给出的内容，币安均不承诺提供任何支持。

文档名 | 描述
------------ | ------------
[enums_CN.md](./enums_CN.md)      | 适用于 Rest API 和 WebSocket API 的枚举定义
[errors_CN.md](./errors_CN.md)     | 现货 API 的错误代码及含义
[filters_CN.md](./filters_CN.md)   | 现货 API 使用的过滤器的详细信息
[rest-api_CN.md](./rest-api_CN.md) | 现货 Rest API 接口定义 (`/api`)
[fix-api_CN.md](fix-api_CN.md)            | 现货 FIX API
[web-socket-api_CN.md](./web-socket-api_CN.md)         | 现货 WebSocket API
[web-socket-streams_CN.md](./web-socket-streams_CN.md) | 现货行情数据流接口的描述
[sbe-market-data-streams_CN.md](./sbe-market-data-streams_CN.md)|SBE 市场数据流
[user-data-stream_CN.md](./user-data-stream_CN.md)     | 现货用户数据流接口的描述
[sbe_schemas](./sbe/schemas/) | 现货API的简单二进制编码 (SBE)模式 (Schema)
[testnet](./testnet) | 仅在现货测试网中可用的 API 文档
[demo-mode](./demo-mode) | 关于模拟交易的描述
&#x0020; |
[Margin Trading](https://developers.binance.com/docs/zh-CN/margin_trading/Introduction) | 关于杠杆交易的描述
[Derivative UM Futures](https://developers.binance.com/docs/zh-CN/derivatives/usds-margined-futures/general-info) | 关于U本位合约相关接口的描述 (`/fapi`)
[Derivative CM Futures](https://developers.binance.com/docs/zh-CN/derivatives/coin-margined-futures/general-info) | 关于币本位合约相关接口的描述 (`/dapi`)
[Derivative Options](https://developers.binance.com/docs/zh-CN/derivatives/option/general-info) | 关于欧式期权的描述 (`/eapi`)
[Derivative Portfolio Margin](https://developers.binance.com/docs/zh-CN/derivatives/portfolio-margin/general-info)| 关于统一账户的描述 (`/papi`)
[Wallet](https://developers.binance.com/docs/zh-CN/wallet/Introduction) | 关于钱包的描述 (`/sapi`)
[Sub Account](https://developers.binance.com/docs/zh-CN/sub_account/Introduction)  | 关于子母账户接口的描述 (`/sapi`)
[Simple Earn](https://developers.binance.com/docs/zh-CN/simple_earn/Introduction) | 关于赚币的描述
[Dual Investment](https://developers.binance.com/docs/binance-spot-api-docs/CHANGELOG) | 关于双币投资接口的描述
[Auto Invest](https://developers.binance.com/docs/zh-CN/auto_invest/Introduction) | 关于定投接口的描述
[Staking](https://developers.binance.com/docs/zh-CN/staking/Introduction) | 关于ETH质押接口的描述
[Mining](https://developers.binance.com/docs/zh-CN/mining/Introduction) | 关于矿池接口的描述
[Algo Trading](https://developers.binance.com/docs/zh-CN/algo/Introduction) | 关于策略交易的描述
[Copy Trading](https://developers.binance.com/docs/zh-CN/copy_trading/Introduction) | 关于跟单交易的描述
[Porfolio Margin Pro](https://developers.binance.com/docs/zh-CN/derivatives/portfolio-margin-pro/general-info) | 关于统一账户专业版的描述
[Fiat](https://developers.binance.com/docs/zh-CN/fiat/Introduction) | 关于法币的描述|
[C2C](https://developers.binance.com/docs/zh-CN/c2c/Introduction) | 关于 C2C 接口的描述|
[VIP Loan](https://developers.binance.com/docs/zh-CN/vip_loan/Introduction) | 关于 VIP 借币的描述
[Crypto Loan](https://developers.binance.com/docs/zh-CN/crypto_loan/Introduction) | 关于质押借币的描述
[Pay](https://developers.binance.com/docs/zh-CN/binance-pay/introduction) |关于币安 Pay 的描述
[Convert](https://developers.binance.com/docs/zh-CN/convert/Introduction) | 关于闪兑接口的描述
[Rebate](https://developers.binance.com/docs/zh-CN/rebate/Introduction) | 关于返佣的描述
[NFT](https://developers.binance.com/docs/zh-CN/nft/Introduction) | 关于 NFT 的描述
[Gift Card](https://developers.binance.com/docs/zh-CN/gift_card/Introduction) | 关于礼品卡的描述

### 常见问题


名称 | 描述
------------ | ------------
[api_key_types](./faqs/api_key_types_CN.md) | API 密钥类型
[spot_glossary_CN](./faqs/spot_glossary_CN.md) | 现货交易 API 术语表
[commission_faq_CN](./faqs/commission_faq_CN.md) | 解释 API 上所使用的佣金计算方式
[trailing_stop_faq_CN](./faqs/trailing-stop-faq_CN.md)   | 追踪止盈止损订单(Trailing Stop)详细信息和常见问题
[stp_faq_CN](./faqs/stp_faq_CN.md) | 关于 Self Trade Prevention (STP) 的详细信息
[market_orders_faq_CN](./faqs/market_orders_faq_CN.md)| 关于市价单行为的详细信息
[market_data_only_CN](./faqs/market_data_only_CN.md) | 仅提供市场数据的 API 和 WebSocket Streams
[sor_faq_CN](./faqs/sor_faq_CN.md) | 智能指令路由 (SOR)
[order_amend_keep_priority_CN](./faqs/order_amend_keep_priority_CN.md)| 关于修改订单并保留其优先级的详细信息
[pegged_orders](./faqs/pegged_orders_CN.md) |  关于挂钩订单的详细信息
[order_count_decrement_CN](./faqs/order_count_decrement_CN.md) | 现货下单频率限制的更新
[sbe_faq_CN](./faqs/sbe_faq_CN.md) | 关于在 API 上实施简单二进制编码 (SBE) 的信息

### 更新日志

关于 API 和数据流方面的最新变动，请参考 [更新日志](./CHANGELOG_CN.md)。


### 相关信息

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
* FIX Connector - 这提供了使用 FIX 协议对交易所的访问.
    * [Python](https://github.com/binance/binance-fix-connector-python)
* [Swagger](https://github.com/binance/binance-api-swagger)
    * 一个基于 OpenAPI 规范的 RESTful API 接口定义的 YAML 文件，还有便于交互的 Swagger UI 页面。
* [Spot Testnet](https://testnet.binance.vision/)
    * 用户可以使用现货的测试网来体验 SPOT 交易。
    * 现在只能通过 API 来测试交易，没有 UI。
    * 只支持以 `/api/*`开头的接口，`/sapi/*`接口不支持。

### 联系我们

* [Binance API 中文电报群](https://t.me/binance_api_chinese) 或 [Binance API 英文电报群](https://t.me/binance_api_english)
    * 适合咨询关于 API 或者 Websocket 性能方面的疑问和困扰。
    * 可以咨询在文档上面没有的 API 问题。
* [Binance 开发者论坛](https://dev.binance.vision/)
    * 可以提问或者咨询关于 API 或者 Websocket 代码方面问题。
* [Binance 客服](https://www.binance.com/zh-CN/support-center)
    * 咨询关于账户，资金，2FA 等问题。
