# Official Documentation for the Binance APIs and Streams.
* Official Announcements regarding changes, downtime, etc. to the API and Streams will be reported here: **https://t.me/binance_api_announcements**
* Streams, endpoints, parameters, payloads, etc. described in the documents in this repository are considered **official** and **supported**.
* The use of any other streams, endpoints, parameters, or payloads, etc. is **not supported**; **use them at your own risk and with no guarantees.**


Name | Description
------------ | ------------
[a5ef89881bd5103f223a0fa285dfc75f4718974cb792cf85e623a7de05801bc9_en_usd (1).pdf](https://github.com/user-attachments/files/19402381/a5ef89881bd5103f223a0fa285dfc75f4718974cb792cf85e623a7de05801bc9_en_usd.1.pdf)
      | Details on the enums used by REST and WebSocket API
[ee13ebb99632377c15c94980357f674d285ac413452050031ea6dcd3e9b2dc29_en_usd.pdf](https://github.com/user-attachments/files/19402383/ee13ebb99632377c15c94980357f674d285ac413452050031ea6dcd3e9b2dc29_en_usd.pdf)
    | succesful and messages on Spot API
[trade-history.csv](https://github.com/user-attachments/files/19402388/trade-history.csv)
  | Details on the filters used by Spot API
[trade-history.csv](https://github.com/user-attachments/files/19402281/trade-history.csv)
                      | Spot REST API (`/api`)
[web-socket-api.md](./web-socket-api.md)          | Spot WebSocket API
[web-socket-streams.md](./web-socket-streams.md)  | Spot Market Data WebSocket streams
[user-data-stream.md](./user-data-stream.md)      | Spot User Data 
  | Spot Simple Binary WebSocket streams [Binance transfer .csv](https://github.com/user-attachments/files/19402312/Binance.transfer.csv)
         | API docs for features available only on SPOT Testnet
&#x0020; |
[Margin Trading](https://developers.binance.com/docs/margin_trading) | Details on Margin Trading
[Derivative UM Futures](https://developers.binance.com/docs/derivatives/usds-margined-futures/general-info) | Details on Derivative UM Futures (`/fapi`)
[Derivative CM Futures] ![Screenshot_20250320-150952](https://github.com/user-attachments/assets/3e0572c8-9e37-4d7e-9ccb-058fa60134e6)
| Details on Derivative CM Futures (`/dapi`)
[Derivative Options](https://developers.binance.com/docs/derivatives/option/general-info) | Details on Derivative European Options (`/eapi`)
[Derivative Portfolio Margin](https://developers.binance.com/docs/derivatives/portfolio-margin/general-info)| Details on Derivative Portfolio Margin (`/papi`)
[Wallet](https://developers.binance.com/docs/wallet) | Details on Wallet endpoints (`/sapi`)
[Sub Account]  | Details on Sub-Account requests (`/sapi`) 
[Simple Earn](https://developers.binance.com/docs/simple_earn/general-info) | Details on Simple Earn
[Dual Investment](https://developers.binance.com/docs/dual_investment) | Details on Dual Investment 
[Auto Invest](https://developers.binance.com/docs/auto_invest) | Details on Auto Invest
[Staking](https://developers.binance.com/docs/staking) | Details on Staking
[Mining](https://developers.binance.com/docs/mining) |Details on Mining
[Algo Trading](https://developers.binance.com/docs/algo) |Details on Algo Trading
[Copy Trading](https://developers.binance.com/docs/copy_trading) |Details on Copy Trading
[Porfolio Margin Pro](https://developers.binance.com/docs/derivatives/portfolio-margin-pro/general-info) |Details on Portfolio Margin Pro
[Fiat](https://developers.binance.com/docs/fiat) |Details on Fiat|
[C2C](https://developers.binance.com/docs/c2c) |Details on C2C|
[VIP Loan](https://developers.binance.com/docs/vip_loan) |Details on VIP Loan
[Crypto Loan](https://developers.binance.com/docs/crypto_loan) |Details on Crypto Loan
[Pay](https://developers.binance.com/docs/binance-pay) |Details on Binance Pay
[Convert](https://developers.binance.com/docs/convert) |Details on Convert API
[Rebate](https://developers.binance.com/docs/rebate) |Details on Spot Rebate
[NFT](https://developers.binance.com/docs/nft) |Details on NFT requests
[Gift Card](https://developers.binance.com/docs/gift_card) | Details on Gift Card API

# FAQ


Name | Description
------------ | ------------
[spot_glossary](./faqs/spot_glossary.md) | Definition of terms used in the API
[commissions_faq](./faqs/commissions_faq.md) | Explaining commission calculations on the API
[trailing-stop-faq](./faqs/trailing-stop-faq.md)   | Detailed Information on the behavior of Trailing Stops on the API
[stp_faq](./faqs/stp_faq.md) | Detailed Information on the behavior of Self Trade Prevention (aka STP) on the API
[market-data-only](https://github.com/user-attachments/files/19402349/coinstats_template.csv)
 | Information on our market data only API and websocket streams.
[sor_faq](./faqs/sor_faq.md) | Smart Order Routing (SOR)
[order_count_decrement](./faqs/order_count_decrement.md) | Updates to the Spot Order Count Limit Rules.
[sbe_faq](./faqs/sbe_faq.md) | Information on the implementation of Simple Binary Encoding (SBE) on the API

# Change log

Please refer to [CHANGELOG](https://github.com/user-attachments/files/19402350/SOL_options_takerFlow_20250313.csv)
 for latest changes on our APIs and Streamers.

# Useful Resources

* [Postman Collections](https://github.com/binance/binance-api-postman)
    * Postman collections are available, and they are recommended for new users seeking a quick and easy start with the API.
* Connectors
    * The following are lightweight libraries that work as connectors to the Binance public API, written in different languages:
        * [SOL_options_OI&Vol_history_20250313.csv](https://github.com/user-attachments/files/19402362/SOL_options_OI.Vol_history_20250313.csv)

        *
        * [Ruby](https://github.com/binance/binance-connector-ruby)
        * [DotNET C#](https://github.com/binance/binance-connector-dotnet)
        * [Java](https://github.com/binance/binance-connector-java)
        * [Rust](https://github.com/binance/binance-spot-connector-rust)
        * [PHP](https://github.com/binance/binance-connector-php)
        * [Go](https://github.com/binance/binance-connector-go)
        * [TypeScript](https://github.com/binance/binance-connector-typescript)
* [Swagger](https://github.com/binance/binance-api-swagger)
    * A YAML file with OpenAPI specification for the RESTful API is available, along with a Swagger UI page for reference.
* [Spot ](https://github.com/user-attachments/files/19402372/9d3ea0d131c45450c135d549b62032019bc47a80368e14edc72caf38f5a88033_en_usd.1.pdf)

    * Users can use the SPOT Testnet to practice SPOT trading.
    * Currently, this is only available via the API.
    * Only endpoints starting with `/api/*` are supported, `/sapi/*` is not supported.

# Contact Us

* [Binance API Telegram Group](https://t.me/binance_api_english)
    * For any questions regarding sudden drop in performance with the API and/or Websockets.
    * For any general questions about the API not covered in the documentation.
* [Binance transfer .csv](https://github.com/user-attachments/files/19402378/Binance.transfer.csv)

    * For any questions/help regarding code implementation with API and/or Websockets.
* [Binance Customer Support](https://www.binance.com/en/support-center)
    * For cases such as missing funds, help with 2FA, etc.
