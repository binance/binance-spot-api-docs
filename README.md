# Official Documentation for the Binance APIs and Streams.
* Official Announcements regarding changes, downtime, etc. to the API and Streams will be reported here: **https://t.me/binance_api_announcements**
* Streams, endpoints, parameters, payloads, etc. described in the documents in this repository are considered **official** and **supported**.
* The use of any other streams, endpoints, parameters, or payloads, etc. is **not supported**; **use them at your own risk and with no guarantees.**


Name | Description
------------ | ------------
[enums.md](./enums.md)      | Details on the enums used by REST and WebSocket API
[errors.md](./errors.md)    | Error codes and messages of Spot API
[filters.md](./filters.md)  | Details on the filters used by Spot API
[rest-api.md](./rest-api.md)                      | Spot REST API (`/api`)
[web-socket-api.md](./web-socket-api.md)          | Spot WebSocket API
[fix-api.md](fix-api.md)                             | FIX API
[web-socket-streams.md](./web-socket-streams.md)  | Spot Market Data WebSocket streams
[sbe-market-data-streams.md](./sbe-market-data-streams.md) | SBE Market Data Streams
[user-data-stream.md](./user-data-stream.md)      | Spot User Data WebSocket streams
[sbe_schemas](./sbe/schemas/)   | Spot Simple Binary Encoding (SBE) schemas
[testnet](./testnet/)           | API docs for features available only on SPOT Testnet
[demo-mode](./demo-mode/) | Page regarding Demo Mode
 &#x0020; |
[Margin Trading](https://developers.binance.com/docs/margin_trading) | Details on Margin Trading
[Derivative UM Futures](https://developers.binance.com/docs/derivatives/usds-margined-futures/general-info) | Details on Derivative UM Futures (`/fapi`)
[Derivative CM Futures](https://developers.binance.com/docs/derivatives/coin-margined-futures/general-info) | Details on Derivative CM Futures (`/dapi`)
[Derivative Options](https://developers.binance.com/docs/derivatives/option/general-info) | Details on Derivative European Options (`/eapi`)
[Derivative Portfolio Margin](https://developers.binance.com/docs/derivatives/portfolio-margin/general-info)| Details on Derivative Portfolio Margin (`/papi`)
[Wallet](https://developers.binance.com/docs/wallet) | Details on Wallet endpoints (`/sapi`)
[Sub Account](https://developers.binance.com/docs/sub_account/general-info)  | Details on Sub-Account requests (`/sapi`)
[Simple Earn](https://developers.binance.com/docs/simple_earn/general-info) | Details on Simple Earn
[Dual Investment](https://developers.binance.com/docs/dual_investment) | Details on Dual Investment
[Auto Invest](https://developers.binance.com/docs/auto_invest) | Details on Auto Invest
[Staking](https://developers.binance.com/docs/staking) | Details on Staking
[Mining](https://developers.binance.com/docs/mining) |Details on Mining
[Algo Trading](https://developers.binance.com/docs/algo) |Details on Algo Trading
[Copy Trading](https://developers.binance.com/docs/copy_trading) |Details on Copy Trading
[Portfolio Margin Pro](https://developers.binance.com/docs/derivatives/portfolio-margin-pro/general-info) |Details on Portfolio Margin Pro
[Fiat](https://developers.binance.com/docs/fiat) |Details on Fiat|
[C2C](https://developers.binance.com/docs/c2c) |Details on C2C|
[VIP Loan](https://developers.binance.com/docs/vip_loan) |Details on VIP Loan
[Crypto Loan](https://developers.binance.com/docs/crypto_loan) |Details on Crypto Loan
[Pay](https://developers.binance.com/docs/binance-pay) |Details on Binance Pay
[Convert](https://developers.binance.com/docs/convert) |Details on Convert API
[Rebate](https://developers.binance.com/docs/rebate) |Details on Spot Rebate
[NFT](https://developers.binance.com/docs/nft) |Details on NFT requests
[Gift Card](https://developers.binance.com/docs/gift_card) | Details on Gift Card API

### FAQ

Name | Description
------------ | ------------
[api_key_types](./faqs/api_key_types.md) | API Key Types
[spot_glossary](./faqs/spot_glossary.md) | Definition of terms used in the API
[commission_faq](./faqs/commission_faq.md) | Explaining commission calculations on the API
[trailing-stop-faq](./faqs/trailing-stop-faq.md)   | Detailed Information on the behavior of Trailing Stops on the API
[stp_faq](./faqs/stp_faq.md) | Detailed Information on the behavior of Self Trade Prevention (aka STP) on the API
[market_orders_faq](./faqs/market_orders_faq.md)| Detailed information on the behavior of Market Orders
[market-data-only](./faqs/market_data_only.md) | Information on our market data only API and WebSocket streams.
[sor_faq](./faqs/sor_faq.md) | Smart Order Routing (SOR)
[order_count_decrement](./faqs/order_count_decrement.md) | Updates to the Spot Order Count Limit Rules.
[order_amend_keep_priority](./faqs/order_amend_keep_priority.md) | Detailed Information on the behavior of Order Amend Keep Priority
[pegged_orders](./faqs/pegged_orders.md) | Detailed Information on Pegged Orders
[sbe_faq](./faqs/sbe_faq.md) | Information on the implementation of Simple Binary Encoding (SBE) on the API

### Change log

Please refer to [CHANGELOG](./CHANGELOG.md) for latest changes on our APIs and Streamers.

### Useful Resources

* [Postman Collections](https://github.com/binance/binance-api-postman)
    * Postman collections are available, and they are recommended for new users seeking a quick and easy start with the API.
* Connectors
    * The following are lightweight libraries that work as connectors to the Binance public API, written in different languages:
        * [Python](https://github.com/binance/binance-connector-python)
        * [Node.js](https://github.com/binance/binance-connector-node)
        * [Ruby](https://github.com/binance/binance-connector-ruby)
        * [DotNET C#](https://github.com/binance/binance-connector-dotnet)
        * [Java](https://github.com/binance/binance-connector-java)
        * [Rust](https://github.com/binance/binance-spot-connector-rust)
        * [PHP](https://github.com/binance/binance-connector-php)
        * [Go](https://github.com/binance/binance-connector-go)
        * [TypeScript](https://github.com/binance/binance-connector-typescript)
* FIX Connector - This provides access to the exchange using the FIX protocol.
    * [Python](https://github.com/binance/binance-fix-connector-python)
* [Swagger](https://github.com/binance/binance-api-swagger)
    * A YAML file with OpenAPI specification for the RESTful API is available, along with a Swagger UI page for reference.
* [Spot Testnet](https://testnet.binance.vision/)
    * Users can use the SPOT Testnet to practice SPOT trading.
    * Currently, this is only available via the API.
    * Only endpoints starting with `/api/*` are supported, `/sapi/*` is not supported.

### Contact Us

* [Binance API Telegram Group](https://t.me/binance_api_english)
    * For any questions regarding sudden drop in performance with the API and/or WebSockets.
    * For any general questions about the API not covered in the documentation.
* [Binance Developers](https://dev.binance.vision/)
    * For any questions/help regarding code implementation with API and/or WebSockets.
* [Binance Customer Support](https://www.binance.com/en/support-center)
    * For cases such as missing funds, help with 2FA, etc.
