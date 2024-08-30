# Official Documentation for the Binance APIs and Streams.
* Official Announcements regarding changes, downtime, etc. to the API and Streams will be reported here: **https://t.me/binance_api_announcements**
* Streams, endpoints, parameters, payloads, etc. described in the documents in this repository are considered **official** and **supported**.
* The use of any other streams, endpoints, parameters, or payloads, etc. is **not supported**; **use them at your own risk and with no guarantees.**

# Fix travel rule api documentation:
# For NZ travel rule content: isAddressOwner should be 1: Yes, 2: No
# Add comments to withdrawal/deposit API regarding url parameters
2024-07-09
# Update travel rule questionnaire content:
# Add withdrawal/deposit questionnaire for Bahrain: Bahrain users can now use sAPI to withdraw/deposit funds
# Update deposit questionnaire for Japan: Adding new required filled isAttested and fix some text issue
2024-06-21
# Adding local entity withdrawal/deposit APIs to support travel rule requirements:
# POST /sapi/v1/localentity/withdraw/apply
# GET /sapi/v1/localentity/withdraw/history
# PUT /sapi/v1/localentity/deposit/provide-info
# GET /sapi/v1/localentity/deposit/history
-2024-06-04
# Wallet Endpoints adjustment: for internal transfers, the txid prefix has been replaced to “Off-chain transfer”on 28 May 2024. “internal transfer” flag is no longer available in the TXID field, including historical transactions, the following endpoints are impacted:
# GET /sapi/v1/capital/deposit/hisrec
# GET /sapi/v1/capital/withdraw/history
# GET /sapi/v1/capital/deposit/subHisrec
2024-05-22
# Update Sub Account Endpoint:
# GET /sapi/v1/sub-account/transfer/subUserHistory: update response field # fromAccountType and toAccountType. Return USDT_FUTURE/COIN_FUTURE in order to differentiate 2 futures wallets.
# New Wallet Endpoint:
GET /sapi/v1/account/info: To fetch the “VIP Level”, “whether Margin account is enabled” and “whether Futures account is enabled”
2024-04-08
Update Wallet Endpoint:
GET /sapi/v1/capital/config/getall: delete response field resetAddressStatus
2024-01-15
New Endpoints for Wallet:
GET /sapi/v1/spot/delist-schedule: Query spot delist schedule
Update Endpoints for Wallet:
GET /sapi/v1/asset/dribblet：add parameter accountType
POST /sapi/v1/asset/dust-btc：add parameter accountType
POST /sapi/v1/asset/dust：add parameter accountType
2023-11-21
New endpoint for Wallet:
GET /sapi/v1/capital/deposit/address/list: Fetch deposit address list with network.
2023-11-02
Changes to Wallet Endpoint:
GET /sapi/v1/account/apiRestrictions: add new response field enablePortfolioMarginTrading
2023-09-22
New endpoints for Wallet:
GET /sapi/v1/asset/wallet/balance: query user wallet balance
GET /sapi/v1/asset/custody/transfer-history: query user delegation history(For Master Account)
2023-09-04
Rate limit adjustment for Wallet Endpoint:
GET /sapi/v1/capital/withdraw/history: UID rate limit is adjusted to 18000, maxmium 10 requests per second. Please refer to the endpoint description for detail
2023-05-18
New endpoints for Wallet：
POST /sapi/v1/capital/deposit/credit-apply: apply deposit credit for expired address
2023-05-09
Update endpoints for Wallet:
POST /sapi/v1/asset/transfer: add enum MAIN_PORTFOLIO_MARGIN and PORTFOLIO_MARGIN_MAIN
2023-02-02
Update endpoints for Wallet:
Universal Transfer POST /sapi/v1/asset/transfer support option transfer
2022-12-26
New endpoints for wallet:
GET /sapi/v1/capital/contract/convertible-coins: Get a user's auto-conversion settings in deposit/withdrawal
POST /sapi/v1/capital/contract/convertible-coins: User can use it to turn on or turn off the BUSD auto-conversion from/to a specific stable coin.
2022-11-18
New endpoint for Wallet:
GET /sapi/v1/asset/ledger-transfer/cloud-mining/queryByPage: The query of Cloud-Mining payment and refund history
2022-11-02
Update endpoints for Wallet:
POST /sapi/v1/capital/withdraw/apply: Weight changed to Weight(UID): 600
2022-10-28
Update endpoints for Wallet:
POST /sapi/v1/asset/convert-transfer: New parameter accountType
POST /sapi/v1/asset/convert-transfer/queryByPage: request method is changed to GET, new parameter clientTranId
2022-09-29
New endpoints for Wallet:
POST /sapi/v1/asset/convert-transfer: Convert transfer, convert between BUSD and stablecoins.
POST /sapi/v1/asset/convert-transfer/queryByPage: Query convert transfer
2022-07-01
New endpoint for Wallet:
POST /sapi/v3/asset/getUserAsset to get user assets.
2022-2-17
The following updates will take effect on February 24, 2022 08:00 AM UTC

Update endpoint for Wallet：
GET /sapi/v1/accountSnapshot
The time limit of this endpoint is shortened to only support querying the data of the latest month

2022-02-09
New endpoint for Wallet:
POST /sapi/v1/asset/dust-btc to get assets that can be converted into BNB
2021-12-30
Update endpoint for Wallet：
As the Mining account is merged into Funding account, transfer types MAIN_MINING, MINING_MAIN, MINING_UMFUTURE, MARGIN_MINING, and MINING_MARGIN will be discontinued in Universal Transfer endpoint POST /sapi/v1/asset/transfer on January 05, 2022 08:00 AM UTC
2021-11-19
Update endpoint for Wallet:
New fieldinfoadded inGET /sapi/v1/capital/withdraw/historyto show the reason for withdrawal failure
2021-11-18
The following updates will take effect on November 25, 2021 08:00 AM UTC

Update endpoint for Wallet：
GET /sapi/v1/accountSnapshot
The query time range of both endpoints are shortened to support data query within the last 6 months only, where startTime does not support selecting a timestamp beyond 6 months. If you do not specify startTime and endTime, the data of the last 7 days will be returned by default.

2021-11-17
The following endpoints will be discontinued on November 17, 2021 13:00 PM UTC:
POST /sapi/v1/account/apiRestrictions/ipRestriction to support user enable and disable IP restriction for an API Key
POST /sapi/v1/account/apiRestrictions/ipRestriction/ipList to support user add IP list for an API Key
GET /sapi/v1/account/apiRestrictions/ipRestriction to support user query IP restriction for an API Key
DELETE /sapi/v1/account/apiRestrictions/ipRestriction/ipList to support user delete IP list for an API Key
2021-11-16
New endpoints for Sub-Account:
POST /sapi/v1/sub-account/subAccountApi/ipRestriction to support master account enable and disable IP restriction for a sub-account API Key
POST /sapi/v1/sub-account/subAccountApi/ipRestriction/ipList to support master account add IP list for a sub-account API Key
GET /sapi/v1/sub-account/subAccountApi/ipRestriction to support master account query IP restriction for a sub-account API Key
DELETE /sapi/v1/sub-account/subAccountApi/ipRestriction/ipList to support master account delete IP list for a sub-account API Key
2021-11-05
Update endpoint for Wallet:
New parameter walletType added in POST /sapi/v1/capital/withdraw/apply to support user choose wallet type spot wallet and funding wallet when withdraw crypto.
2021-11-04
The following updates will take effect on November 11, 2021 08:00 AM UTC

Update endpoints for Wallet and Futures：
GET /sapi/v1/asset/transfer
GET /sapi/v1/futures/transfer
The query time range of both endpoints are shortened to support data query within the last 6 months only, where startTime does not support selecting a timestamp beyond 6 months. If you do not specify startTime and endTime, the data of the last 7 days will be returned by default.

2021-10-22
# Update endpoint for Wallet:
# New transfer types # #MAIN_FUNDING,FUNDING_MAIN,FUNDING_UMFUTURE,UMFUTURE_FUNDING,MARGIN_FUNDING,# FUNDING_MARGIN,FUNDING_CMFUTUREand CMFUTURE_FUNDING added in Universal # #Transfer endpoint POST /sapi/v1/asset/transfer and GET /sapi/v1/asset/transfer to support transfer assets among funding account and other accounts
# As the C2C account, Binance Payment, Binance Card and other business account are merged into a Funding account, transfer types # #MAIN_C2C,C2C_MAIN,C2C_UMFUTURE,C2C_MINING,UMFUTURE_C2C,MINING_C2C,MARGIN_C2C,C2C_MARGIN,MAIN_PAYand PAY_MAIN will be discontinued in Universal # #Transfer endpoint POST /sapi/v1/asset/transfer and GET # #/sapi/v1/asset/transfer on November 04, 2021 08:00 AM UTC
2021-09-03
# Update endpoint for Wallet: _ New fields sameAddress,depositDust and specialWithdrawTipsadded in GET /sapi/v1/capital/config/getall sameAddress means if the coin needs to provide memo to withdraw depositDust means # #minimum creditable amount specialWithdrawTips means special tips for # #withdraw _ New field confirmNoadded in GET /sapi/v1/capital/withdraw/history to support query confirm times for # #withdraw history
2021-08-27
# Update endpoint for Wallet:
# New parameter withdrawOrderIdadded in GET # #/sapi/v1/capital/withdraw/history to support user query withdraw history by withdrawOrderId
# New field unlockConfirmadded in GET /sapi/v1/capital/deposit/hisrec to # support query network confirm times for unlocking
2021-08-20
# Update endpoint for Wallet:
# New parametersfromSymbol,toSymboland new transfer types ISOLATEDMARGIN_MARGIN, MARGIN_ISOLATEDMARGINand # ## ISOLATEDMARGIN_ISOLATEDMARGIN added in POST /sapi/v1/asset/transfer and GET /sapi/v1/asset/transfer to support user transfer assets between # # #Margin(cross) account and Margin(isolated) account
2021-07-16
# New endpoint for Wallet:
# GET /sapi/v1/account/apiRestrictions to query user API Key permission
# 2021-07-09
# New endpoint for Wallet:
# POST /sapi/v1/asset/get-funding-asset to query funding wallet, includes # # Binance Pay, Binance Card, Binance Gift Card, Stock Token
2021-06-24
# Update endpoints for Wallet:
# GET /sapi/v1/capital/withdraw/history added default value 1000, max value 1000 for the parameterlimit
# GET /sapi/v1/capital/deposit/hisrec added default value 1000, max value 1000 for the parameterlimit
2021-05-26
# Update endpoint for Wallet:
# New transfer types MAIN_PAY ,PAY_MAIN added in Universal Transfer endpoint # POST /sapi/v1/asset/transfer and GET /sapi/v1/asset/transfer to support trasnfer assets between spot account and pay account
2020-12-30
# New endpoint for Wallet:
# POST /sapi/v1/asset/transfer to support user universal transfer among # # Spot, Margin, Futures, C2C, MINING accounts.
# GET /sapi/v1/asset/transfer to get user universal transfer history.
2020-04-02
# New fields in response to endpointGET /sapi/v1/capital/config/getall：
minConfirm for min number for balance confirmation
# unLockConfirm for confirmation number for balance unlock
2020-03-13
# New parameter transactionFeeFlag is available in endpoint:
# POST /sapi/v1/capital/withdraw/apply and
# POST /wapi/v3/withdraw.html
2020-01-15
# New parameter withdrawOrderId for client customized withdraw id for endpoint POST /wapi/v3/withdraw.html.
# New field withdrawOrderId in response to GET /wapi/v3/withdrawHistory.html
2019-12-25
# Added time interval limit in
# GET /sapi/v1/capital/withdraw/history,
# GET /wapi/v3/withdrawHistory.html,
# GET /sapi/v1/capital/deposit/hisrec and
# GET /wapi/v3/depositHistory.html: _ The default startTime is 90 days from current time, and the default endTime is current time. _ Please notice the default startTime and endTime to make sure that time interval is within 0-90 days. * If both startTime and endTime are sent, time between startTime and endTime must be less than 90 days.
2019-12-18
# New endpoint to get daily snapshot of account:
# GET /sapi/v1/accountSnapshot
2019-11-30
# Added parameter sideEffectType in POST /sapi/v1/margin/order (HMAC SHA256) with enums:

# NO_SIDE_EFFECT for normal trade order;
# MARGIN_BUY for margin trade order;
# AUTO_REPAY for making auto repayment after order filled.
# New field marginBuyBorrowAmount and marginBuyBorrowAsset in FULL response to POST /sapi/v1/margin/order (HMAC SHA256)

# 2024-10-29
# New sapi endpoints for wallet.
# POST /sapi/v1/capital/withdraw/apply (HMAC SHA256): withdraw.
# Get /sapi/v1/capital/withdraw/history (HMAC SHA256): fetch withdraw history with network.
2019-10-14
# New sapi endpoints for wallet.
# GET /sapi/v1/capital/config/getall (HMAC SHA256): get all coins' information for user.
# GET /sapi/v1/capital/deposit/hisrec (HMAC SHA256): fetch deposit history # with network.
# GET /sapi/v1/capital/deposit/address (HMAC SHA256): fetch deposit address with network.

Name | Description
------------ | ------------
[enums.md](./enums.md)      | Details on the enums used by REST and WebSocket API
[errors.md](./errors.md)    | Error codes and messages of Spot API
[filters.md](./filters.md)  | Details on the filters used by Spot API
[rest-api.md](./rest-api.md)                      | Spot REST API (`/api`)
[web-socket-api.md](./web-socket-api.md)          | Spot WebSocket API
[web-socket-streams.md](./web-socket-streams.md)  | Spot Market Data WebSocket streams
[user-data-stream.md](./user-data-stream.md)      | Spot User Data WebSocket streams
[sbe_schemas](./sbe/schemas/)   | Spot Simple Binary Encoding (SBE) schemas
[testnet](./testnet/)           | API docs for features available only on SPOT Testnet
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
[market-data-only](./faqs/market_data_only.md) | Information on our market data only API and websocket streams.
[sor_faq](./faqs/sor_faq.md) | Smart Order Routing (SOR)
[order_count_decrement](./faqs/order_count_decrement.md) | Updates to the Spot Order Count Limit Rules.
[sbe_faq](./faqs/sbe_faq.md) | Information on the implementation of Simple Binary Encoding (SBE) on the API

# Change log

Please refer to [CHANGELOG](./CHANGELOG.md) for latest changes on our APIs and Streamers.

# Useful Resources

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
* [Swagger](https://github.com/binance/binance-api-swagger)
    * A YAML file with OpenAPI specification for the RESTful API is available, along with a Swagger UI page for reference.
* [Spot Testnet](https://testnet.binance.vision/)
    * Users can use the SPOT Testnet to practice SPOT trading.
    * Currently, this is only available via the API.
    * Only endpoints starting with `/api/*` are supported, `/sapi/*` is not supported.

# Contact Us

* [Binance API Telegram Group](https://t.me/binance_api_english)
    * For any questions regarding sudden drop in performance with the API and/or Websockets.
    * For any general questions about the API not covered in the documentation.
* [Binance Developers](https://dev.binance.vision/)
    * For any questions/help regarding code implementation with API and/or Websockets.
* [Binance Customer Support](https://www.binance.com/en/support-center)
    * For cases such as missing funds, help with 2FA, etc.
