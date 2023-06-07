<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

  - [General API Information](#general-api-information)
  - [HTTP Return Codes](#http-return-codes)
  - [Error Codes](#error-codes)
  - [General Information on Endpoints](#general-information-on-endpoints)
- [LIMITS](#limits)
  - [General Info on Limits](#general-info-on-limits)
  - [IP Limits](#ip-limits)
  - [Order Rate Limits](#order-rate-limits)
- [Data Sources](#data-sources)
- [Endpoint security type](#endpoint-security-type)
- [SIGNED (TRADE and USER_DATA) Endpoint security](#signed-trade-and-user_data-endpoint-security)
  - [Timing security](#timing-security)
  - [SIGNED Endpoint Examples for POST /api/v3/order](#signed-endpoint-examples-for-post-apiv3order)
    - [HMAC Keys](#hmac-keys)
      - [Example 1: As a request body](#example-1-as-a-request-body)
      - [Example 2: As a query string](#example-2-as-a-query-string)
      - [Example 3: Mixed query string and request body](#example-3-mixed-query-string-and-request-body)
    - [RSA Keys](#rsa-keys)
- [Public API Endpoints](#public-api-endpoints)
    - [Terminology](#terminology)
  - [ENUM definitions](#enum-definitions)
  - [General endpoints](#general-endpoints)
    - [Test connectivity](#test-connectivity)
    - [Check server time](#check-server-time)
    - [Exchange information](#exchange-information)
  - [Market Data endpoints](#market-data-endpoints)
    - [Order book](#order-book)
    - [Recent trades list](#recent-trades-list)
    - [Old trade lookup (MARKET_DATA)](#old-trade-lookup-market_data)
    - [Compressed/Aggregate trades list](#compressedaggregate-trades-list)
    - [Kline/Candlestick data](#klinecandlestick-data)
    - [UIKlines](#uiklines)
    - [Current average price](#current-average-price)
    - [24hr ticker price change statistics](#24hr-ticker-price-change-statistics)
    - [Symbol price ticker](#symbol-price-ticker)
    - [Symbol order book ticker](#symbol-order-book-ticker)
    - [Rolling window price change statistics](#rolling-window-price-change-statistics)
  - [Account endpoints](#account-endpoints)
    - [New order  (TRADE)](#new-order--trade)
      - [Conditional fields in Order Responses](#conditional-fields-in-order-responses)
    - [Test new order (TRADE)](#test-new-order-trade)
    - [Query order (USER_DATA)](#query-order-user_data)
    - [Cancel order (TRADE)](#cancel-order-trade)
      - [Regarding `cancelRestrictions`](#regarding-cancelrestrictions)
    - [Cancel All Open Orders on a Symbol (TRADE)](#cancel-all-open-orders-on-a-symbol-trade)
    - [Cancel an Existing Order and Send a New Order (TRADE)](#cancel-an-existing-order-and-send-a-new-order-trade)
    - [Current open orders (USER_DATA)](#current-open-orders-user_data)
    - [All orders (USER_DATA)](#all-orders-user_data)
    - [New OCO (TRADE)](#new-oco-trade)
    - [Cancel OCO (TRADE)](#cancel-oco-trade)
    - [Query OCO (USER_DATA)](#query-oco-user_data)
    - [Query all OCO (USER_DATA)](#query-all-oco-user_data)
    - [Query Open OCO (USER_DATA)](#query-open-oco-user_data)
    - [Account information (USER_DATA)](#account-information-user_data)
    - [Account trade list (USER_DATA)](#account-trade-list-user_data)
    - [Query Current Order Count Usage (TRADE)](#query-current-order-count-usage-trade)
    - [Query Prevented Matches (USER_DATA)](#query-prevented-matches-user_data)
  - [User data stream endpoints](#user-data-stream-endpoints)
    - [Start user data stream (USER_STREAM)](#start-user-data-stream-user_stream)
    - [Keepalive user data stream (USER_STREAM)](#keepalive-user-data-stream-user_stream)
    - [Close user data stream (USER_STREAM)](#close-user-data-stream-user_stream)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Public Rest API for Binance (2023-06-07)

## General API Information
* The following base endpoints are available. Please use whichever works best for your setup:
  * **https://api.binance.com**
  * **https://api-gcp.binance.com**
  * **https://api1.binance.com**
  * **https://api2.binance.com**
  * **https://api3.binance.com**
  * **https://api4.binance.com**
* The last 4 endpoints in the point above (`api1`-`api4`) might give better performance but have less stability. 
* All endpoints return either a JSON object or array.
* Data is returned in **ascending** order. Oldest first, newest last.
* All time and timestamp related fields are in **milliseconds**.
* For APIs that only send public market data, please use the base endpoint **https://data-api.binance.vision**. Please refer to [Market Data Only](.faqs/market_data_only.md) page.

## HTTP Return Codes

* HTTP `4XX` return codes are used for malformed requests; the issue is on the sender's side.
* HTTP `403` return code is used when the WAF Limit (Web Application Firewall) has been violated.
* HTTP `409` return code is used when a cancelReplace order partially succeeds. (i.e. if the cancellation of the order fails but the new order placement succeeds.)
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `418` return code is used when an IP has been auto-banned for continuing to send requests after receiving `429` codes.
* HTTP `5XX` return codes are used for internal errors; the issue is on Binance's side.
  It is important to **NOT** treat this as a failure operation; the execution status is
  **UNKNOWN** and could have been a success.


## Error Codes
* Any endpoint can return an ERROR

Sample Payload below:
```javascript
{
  "code": -1121,
  "msg": "Invalid symbol."
}
```
* Specific error codes and messages are defined in [Errors Codes](./errors.md).

## General Information on Endpoints
* For `GET` endpoints, parameters must be sent as a `query string`.
* For `POST`, `PUT`, and `DELETE` endpoints, the parameters may be sent as a
  `query string` or in the `request body` with content type
  `application/x-www-form-urlencoded`. You may mix parameters between both the
  `query string` and `request body` if you wish to do so.
* Parameters may be sent in any order.
* If a parameter sent in both the `query string` and `request body`, the
  `query string` parameter will be used.

# LIMITS

## General Info on Limits
* The following `intervalLetter` values for headers:
    * SECOND => S
    * MINUTE => M
    * HOUR => H
    * DAY => D
* `intervalNum` describes the amount of the interval. For example, `intervalNum` 5 with `intervalLetter` M means "Every 5 minutes".
* The `/api/v3/exchangeInfo` `rateLimits` array contains objects related to the exchange's `RAW_REQUESTS`, `REQUEST_WEIGHT`, and `ORDERS` rate limits. These are further defined in the `ENUM definitions` section under `Rate limiters (rateLimitType)`.
* A 429 will be returned when either request rate limit or order rate limit is violated.

## IP Limits
* Every request will contain `X-MBX-USED-WEIGHT-(intervalNum)(intervalLetter)` in the response headers which has the current used weight for the IP for all request rate limiters defined.
* Each route has a `weight` which determines for the number of requests each endpoint counts for. Heavier endpoints and endpoints that do operations on multiple symbols will have a heavier `weight`.
* When a 429 is received, it's your obligation as an API to back off and not spam the API.
* **Repeatedly violating rate limits and/or failing to back off after receiving 429s will result in an automated IP ban (HTTP status 418).**
* IP bans are tracked and **scale in duration** for repeat offenders, **from 2 minutes to 3 days**.
* A `Retry-After` header is sent with a 418 or 429 responses and will give the **number of seconds** required to wait, in the case of a 429, to prevent a ban, or, in the case of a 418, until the ban is over.
* **The limits on the API are based on the IPs, not the API keys.**

## Order Rate Limits
* Every successful order response will contain a `X-MBX-ORDER-COUNT-(intervalNum)(intervalLetter)` header which has the current order count for the account for all order rate limiters defined. To monitor order count usage, refer to `GET api/v3/rateLimit/order`.
* When the order count exceeds the limit, you will receive a 429 error without the `Retry-After` header. Please check the Order Rate Limit rules using `GET api/v3/exchangeInfo` and wait for reactivation accordingly.
* Rejected/unsuccessful orders are not guaranteed to have `X-MBX-ORDER-COUNT-**` headers in the response.
* **The order rate limit is counted against each account**.

# Data Sources
* The API system is asynchronous, so some delay in the response is normal and expected.
* Each endpoint has a data source indicating where the data is being retrieved, and thus which endpoints have the most up-to-date response.

These are the three sources, ordered by which is has the most up-to-date response to the one with potential delays in updates.

  * **Matching Engine** - the data is from the matching Engine
  * **Memory** - the data is from a server's local or external memory
  * **Database** - the data is taken directly from a database

Some endpoints can have more than 1 data source. (e.g. Memory => Database)
This means that the endpoint will check the first Data Source, and if it cannot find the value it's looking for it will check the next one.

# Endpoint security type
* Each endpoint has a security type that determines how you will
  interact with it. This is stated next to the NAME of the endpoint.
* If no security type is stated, assume the security type is NONE.
* API-keys are passed into the Rest API via the `X-MBX-APIKEY`
  header.
* API-keys and secret-keys **are case sensitive**.
* API-keys can be configured to only access certain types of secure endpoints.
 For example, one API-key could be used for TRADE only, while another API-key
 can access everything except for TRADE routes.
* By default, API-keys can access all secure routes.

Security Type | Description
------------ | ------------
NONE | Endpoint can be accessed freely.
TRADE | Endpoint requires sending a valid API-Key and signature.
USER_DATA | Endpoint requires sending a valid API-Key and signature.
USER_STREAM | Endpoint requires sending a valid API-Key.
MARKET_DATA | Endpoint requires sending a valid API-Key.


* `TRADE` and `USER_DATA` endpoints are `SIGNED` endpoints.

# SIGNED (TRADE and USER_DATA) Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, to be
  sent in the  `query string` or `request body`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* The `signature` is **not case sensitive**.
* `totalParams` is defined as the `query string` concatenated with the
  `request body`.

## Timing security
* A `SIGNED` endpoint also requires a parameter, `timestamp`, to be sent which
  should be the millisecond timestamp of when the request was created and sent.
* An additional parameter, `recvWindow`, may be sent to specify the number of
  milliseconds after `timestamp` the request is valid for. If `recvWindow`
  is not sent, **it defaults to 5000**.
* The logic is as follows:

  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```

**Serious trading is about timing.** Networks can be unstable and unreliable,
which can lead to requests taking varying amounts of time to reach the
servers. With `recvWindow`, you can specify that the request must be
processed within a certain number of milliseconds or be rejected by the
server.


**It is recommended to use a small recvWindow of 5000 or less! The max cannot go beyond 60,000!**


## SIGNED Endpoint Examples for POST /api/v3/order

### HMAC Keys
Here is a step-by-step example of how to send a valid signed payload from the
Linux command line using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
apiKey | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


Parameter | Value
------------ | ------------
symbol | LTCBTC
side | BUY
type | LIMIT
timeInForce | GTC
quantity | 1
price | 0.1
recvWindow | 5000
timestamp | 1499827319559

#### Example 1: As a request body
* **requestBody:** symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.binance.com/api/v3/order' -d 'symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

#### Example 2: As a query string
* **queryString:** symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.binance.com/api/v3/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

#### Example 3: Mixed query string and request body
* **queryString:** symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC
* **requestBody:** quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTCquantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= 0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.binance.com/api/v3/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC' -d 'quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77'
    ```

Note that the signature is different in example 3.
There is no & between "GTC" and "quantity=1".


### RSA Keys

This will be a step by step process how to create the signature payload to send a valid signed payload.

We support `PKCS#8` currently.

To get your API key, you need to upload your RSA Public Key to your account and a corresponding API key will be provided for you.

For this example, the private key will be referenced as `./test-prv-key.pem`

Key | Value
------------ | ------------
apiKey | CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ

Parameter | Value
------------ | ------------
symbol | BTCUSDT
side | SELL
type | LIMIT
timeInForce | GTC
quantity | 1
price | 0.2
timestamp | 1668481559918
recvWindow | 5000


**Step 1: Construct the payload**

Arrange the list of parameters into a string. Separate each parameter with a `&`.

For the parameters above, the signature payload would look like this:

```console
symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000
```

**Step 2: Compute the signature:**

1. Encode signature payload as ASCII data.
2. Sign payload using RSASSA-PKCS1-v1_5 algorithm with SHA-256 hash function.

```console
$ echo -n 'symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000' | openssl dgst -sha256 -sign ./test-prv-key.pem
```
3. Encode output as base64 string.

```console
$  echo -n 'symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000' | openssl dgst -sha256 -sign ./test-prv-key.pem | openssl enc -base64 -A
HZ8HOjiJ1s/igS9JA+n7+7Ti/ihtkRF5BIWcPIEluJP6tlbFM/Bf44LfZka/iemtahZAZzcO9TnI5uaXh3++lrqtNonCwp6/245UFWkiW1elpgtVAmJPbogcAv6rSlokztAfWk296ZJXzRDYAtzGH0gq7CgSJKfH+XxaCmR0WcvlKjNQnp12/eKXJYO4tDap8UCBLuyxDnR7oJKLHQHJLP0r0EAVOOSIbrFang/1WOq+Jaq4Efc4XpnTgnwlBbWTmhWDR1pvS9iVEzcSYLHT/fNnMRxFc7u+j3qI//5yuGuu14KR0MuQKKCSpViieD+fIti46sxPTsjSemoUKp0oXA==
```
4. Since the signature may contain `/` and `=`, this could cause issues with sending the request. So the signature has to be URL encoded.

```console
HZ8HOjiJ1s%2FigS9JA%2Bn7%2B7Ti%2FihtkRF5BIWcPIEluJP6tlbFM%2FBf44LfZka%2FiemtahZAZzcO9TnI5uaXh3%2B%2BlrqtNonCwp6%2F245UFWkiW1elpgtVAmJPbogcAv6rSlokztAfWk296ZJXzRDYAtzGH0gq7CgSJKfH%2BXxaCmR0WcvlKjNQnp12%2FeKXJYO4tDap8UCBLuyxDnR7oJKLHQHJLP0r0EAVOOSIbrFang%2F1WOq%2BJaq4Efc4XpnTgnwlBbWTmhWDR1pvS9iVEzcSYLHT%2FfNnMRxFc7u%2Bj3qI%2F%2F5yuGuu14KR0MuQKKCSpViieD%2BfIti46sxPTsjSemoUKp0oXA%3D%3D
```

5. The curl command:

```console
curl -H "X-MBX-APIKEY: CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ" -X POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000&signature=HZ8HOjiJ1s%2FigS9JA%2Bn7%2B7Ti%2FihtkRF5BIWcPIEluJP6tlbFM%2FBf44LfZka%2FiemtahZAZzcO9TnI5uaXh3%2B%2BlrqtNonCwp6%2F245UFWkiW1elpgtVAmJPbogcAv6rSlokztAfWk296ZJXzRDYAtzGH0gq7CgSJKfH%2BXxaCmR0WcvlKjNQnp12%2FeKXJYO4tDap8UCBLuyxDnR7oJKLHQHJLP0r0EAVOOSIbrFang%2F1WOq%2BJaq4Efc4XpnTgnwlBbWTmhWDR1pvS9iVEzcSYLHT%2FfNnMRxFc7u%2Bj3qI%2F%2F5yuGuu14KR0MuQKKCSpViieD%2BfIti46sxPTsjSemoUKp0oXA%3D%3D'
```

A sample Bash script below does the similar steps said above.

```bash
API_KEY="put your own API Key here"
PRIVATE_KEY_PATH="test-prv-key.pem"
# Set up the request:
API_METHOD="POST"
API_CALL="api/v3/order"
API_PARAMS="symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2"
# Sign the request:
timestamp=$(date +%s000)
api_params_with_timestamp="$API_PARAMS&timestamp=$timestamp"
signature=$(echo -n "$api_params_with_timestamp" \
            | openssl dgst -sha256 -sign "$PRIVATE_KEY_PATH" \
            | openssl enc -base64 -A)
# Send the request:
curl -H "X-MBX-APIKEY: $API_KEY" -X "$API_METHOD" \
    "https://api.binance.com/$API_CALL?$api_params_with_timestamp" \
    --data-urlencode "signature=$signature"
```


# Public API Endpoints
### Terminology

These terms will be used throughout the documentation, so it is recommended especially for new users to read to help their understanding of the API.

* `base asset` refers to the asset that is the `quantity` of a symbol. For the symbol BTCUSDT, BTC would be the `base asset`.
* `quote asset` refers to the asset that is the `price` of a symbol. For the symbol BTCUSDT, USDT would be the `quote asset`.


## ENUM definitions
**Symbol status (status):**

* `PRE_TRADING`
* `TRADING`
* `POST_TRADING`
* `END_OF_DAY`
* `HALT`
* `AUCTION_MATCH`
* `BREAK`

**Account and Symbol Permissions (permissions):**

* `SPOT`
* `MARGIN`
* `LEVERAGED`
* `TRD_GRP_002`
* `TRD_GRP_003`
* `TRD_GRP_004`
* `TRD_GRP_005`
* `TRD_GRP_006`
* `TRD_GRP_007`
* `TRD_GRP_008`
* `TRD_GRP_009`
* `TRD_GRP_010`
* `TRD_GRP_011`
* `TRD_GRP_012`
* `TRD_GRP_013`


**Order status (status):**

Status | Description
-----------| --------------
`NEW` | The order has been accepted by the engine.
`PARTIALLY_FILLED`| A part of the order has been filled.
`FILLED` | The order has been completed.
`CANCELED` | The order has been canceled by the user.
`PENDING_CANCEL` | Currently unused
`REJECTED`       | The order was not accepted by the engine and not processed.
`EXPIRED` | The order was canceled according to the order type's rules (e.g. LIMIT FOK orders with no fill, LIMIT IOC or MARKET orders that partially fill) <br/> or by the exchange, (e.g. orders canceled during liquidation, orders canceled during maintenance)
`EXPIRED_IN_MATCH` | The order was expired by the exchange due to STP. (e.g. an order with `EXPIRE_TAKER` will match with existing orders on the book with the same account or same `tradeGroupId`)

**OCO Status (listStatusType):**

Status | Description
-----------| --------------
`RESPONSE` | This is used when the ListStatus is responding to a failed action. (E.g. Orderlist placement or cancellation)
`EXEC_STARTED`| The order list has been placed or there is an update to the order list status.
`ALL_DONE` | The order list has finished executing and thus no longer active.

**OCO Order Status (listOrderStatus):**

Status | Description
-----------| --------------
`EXECUTING` | Either an order list has been placed or there is an update to the status of the list.
`ALL_DONE`| An order list has completed execution and thus no longer active.
`REJECT` | The List Status is responding to a failed action either during order placement or order canceled

**ContingencyType**
* `OCO`

**Order types (orderTypes, type):**

* `LIMIT`
* `MARKET`
* `STOP_LOSS`
* `STOP_LOSS_LIMIT`
* `TAKE_PROFIT`
* `TAKE_PROFIT_LIMIT`
* `LIMIT_MAKER`

**Order Response Type (newOrderRespType):**

* `ACK`
* `RESULT`
* `FULL`

**Order side (side):**

* `BUY`
* `SELL`

**Time in force (timeInForce):**

This sets how long an order will be active before expiration.

Status | Description
-----------| --------------
`GTC` | Good Til Canceled <br/> An order will be on the book unless the order is canceled.
`IOC` | Immediate Or Cancel <br/> An order will try to fill the order as much as it can before the order expires.
`FOK`| Fill or Kill <br/> An order will expire if the full order cannot be filled upon execution.

**Kline/Candlestick chart intervals:**

s-> seconds; m -> minutes; h -> hours; d -> days; w -> weeks; M -> months

* 1s
* 1m
* 3m
* 5m
* 15m
* 30m
* 1h
* 2h
* 4h
* 6h
* 8h
* 12h
* 1d
* 3d
* 1w
* 1M

**Rate limiters (rateLimitType)**
* REQUEST_WEIGHT

    ```json
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200
    }
    ```

* ORDERS

    ```json
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 1,
      "limit": 10
    }
    ```
* RAW_REQUESTS

    ```json
    {
      "rateLimitType": "RAW_REQUESTS",
      "interval": "MINUTE",
      "intervalNum": 5,
      "limit": 5000
    }
    ```

**Rate limit intervals (interval)**

* SECOND
* MINUTE
* DAY

## General endpoints
### Test connectivity
```
GET /api/v3/ping
```
Test connectivity to the Rest API.

**Weight:**
1

**Parameters:**
NONE

**Data Source:**
Memory

**Response:**
```javascript
{}
```

### Check server time
```
GET /api/v3/time
```
Test connectivity to the Rest API and get the current server time.

**Weight:**
1

**Parameters:**
NONE

**Data Source:**
Memory

**Response:**
```javascript
{
  "serverTime": 1499827319559
}
```

### Exchange information
```
GET /api/v3/exchangeInfo
```
Current exchange trading rules and symbol information

**Weight:**
10

**Parameters:**

There are 4 possible options:

|Options|Example|
----- | ----|
|No parameter|curl -X GET "https://api.binance.com/api/v3/exchangeInfo"|
|symbol|curl -X GET "https://api.binance.com/api/v3/exchangeInfo?symbol=BNBBTC"|
|symbols| curl -X GET "https://api.binance.com/api/v3/exchangeInfo?symbols=%5B%22BNBBTC%22,%22BTCUSDT%22%5D" <br/> or <br/> curl -g -X  GET 'https://api.binance.com/api/v3/exchangeInfo?symbols=["BTCUSDT","BNBBTC"]' |
|permissions| curl -X GET "https://api.binance.com/api/v3/exchangeInfo?permissions=SPOT" <br/> or <br/> curl -X GET "https://api.binance.com/api/v3/exchangeInfo?permissions=%5B%22MARGIN%22%2C%22LEVERAGED%22%5D" <br/> or <br/> curl -g -X GET 'https://api.binance.com/api/v3/exchangeInfo?permissions=["MARGIN","LEVERAGED"]' |

**Notes**:
* If the value provided to `symbol` or `symbols` do not exist, the endpoint will throw an error saying the symbol is invalid.
* All parameters are optional.
* `permissions` can support single or multiple values (e.g. `SPOT`, `["MARGIN","LEVERAGED"]`)
* If `permissions` parameter not provided, the default values will be `["SPOT","MARGIN","LEVERAGED"]`.
  * If one wants to view all symbols on `GET /api/v3/exchangeInfo`, then one has to search with all permissions explicitly specified
  (i.e. `permissions=["SPOT","MARGIN","LEVERAGED","TRD_GRP_002","TRD_GRP_003","TRD_GRP_004","TRD_GRP_005","TRD_GRP_006","TRD_GRP_007","TRD_GRP_008","TRD_GRP_009","TRD_GRP_010","TRD_GRP_011","TRD_GRP_012","TRD_GRP_013"]`)

**Data Source:**
Memory

**Response:**
```javascript
{
  "timezone": "UTC",
  "serverTime": 1565246363776,
  "rateLimits": [
    {
      // These are defined in the `ENUM definitions` section under `Rate Limiters (rateLimitType)`.
      // All limits are optional
    }
  ],
  "exchangeFilters": [
    // These are the defined filters in the `Filters` section.
    // All filters are optional.
  ],
  "symbols": [
    {
      "symbol": "ETHBTC",
      "status": "TRADING",
      "baseAsset": "ETH",
      "baseAssetPrecision": 8,
      "quoteAsset": "BTC",
      "quotePrecision": 8, // will be removed in future api versions (v4+)
      "quoteAssetPrecision": 8,
      "baseCommissionPrecision": 8,
      "quoteCommissionPrecision": 8,
      "orderTypes": [
        "LIMIT",
        "LIMIT_MAKER",
        "MARKET",
        "STOP_LOSS",
        "STOP_LOSS_LIMIT",
        "TAKE_PROFIT",
        "TAKE_PROFIT_LIMIT"
      ],
      "icebergAllowed": true,
      "ocoAllowed": true,
      "quoteOrderQtyMarketAllowed": true,
      "allowTrailingStop": false,
      "cancelReplaceAllowed":false,
      "isSpotTradingAllowed": true,
      "isMarginTradingAllowed": true,
      "filters": [
        // These are defined in the Filters section.
        // All filters are optional
      ],
      "permissions": [
        "SPOT",
        "MARGIN"
      ],
      "defaultSelfTradePreventionMode": "NONE",
      "allowedSelfTradePreventionModes": [
        "NONE"
      ]
    }
  ]
}
```

## Market Data endpoints
### Order book
```
GET /api/v3/depth
```

**Weight:**
Adjusted based on the limit:

|Limit|Request Weight
------|-------
1-100|  1
101-500| 5
501-1000| 10
1001-5000| 50

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 100; max 5000. <br/> If limit > 5000. then the response will truncate to 5000.

**Data Source:**
Memory

**Response:**
```javascript
{
  "lastUpdateId": 1027024,
  "bids": [
    [
      "4.00000000",     // PRICE
      "431.00000000"    // QTY
    ]
  ],
  "asks": [
    [
      "4.00000200",
      "12.00000000"
    ]
  ]
}
```


### Recent trades list
```
GET /api/v3/trades
```
Get recent trades.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 500; max 1000.

**Data Source:**
Memory

**Response:**
```javascript
[
  {
    "id": 28457,
    "price": "4.00000100",
    "qty": "12.00000000",
    "quoteQty": "48.000012",
    "time": 1499865549590,
    "isBuyerMaker": true,
    "isBestMatch": true
  }
]
```

### Old trade lookup (MARKET_DATA)
```
GET /api/v3/historicalTrades
```
Get older trades.

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 500; max 1000.
fromId | LONG | NO | TradeId to fetch from. Default gets most recent trades.

**Data Source:**
Database

**Response:**
```javascript
[
  {
    "id": 28457,
    "price": "4.00000100",
    "qty": "12.00000000",
    "quoteQty": "48.000012",
    "time": 1499865549590,
    "isBuyerMaker": true,
    "isBestMatch": true
  }
]
```

### Compressed/Aggregate trades list
```
GET /api/v3/aggTrades
```
Get compressed, aggregate trades. Trades that fill at the time, from the same taker order, with the same price will have the quantity aggregated.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
fromId | LONG | NO | ID to get aggregate trades from INCLUSIVE.
startTime | LONG | NO | Timestamp in ms to get aggregate trades from INCLUSIVE.
endTime | LONG | NO | Timestamp in ms to get aggregate trades until INCLUSIVE.
limit | INT | NO | Default 500; max 1000.

* If fromId, startTime, and endTime are not sent, the most recent aggregate trades will be returned.

**Data Source:**
Database

**Response:**
```javascript
[
  {
    "a": 26129,         // Aggregate tradeId
    "p": "0.01633102",  // Price
    "q": "4.70443515",  // Quantity
    "f": 27781,         // First tradeId
    "l": 27781,         // Last tradeId
    "T": 1498793709153, // Timestamp
    "m": true,          // Was the buyer the maker?
    "M": true           // Was the trade the best price match?
  }
]
```

### Kline/Candlestick data
```
GET /api/v3/klines
```
Kline/candlestick bars for a symbol.
Klines are uniquely identified by their open time.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.

* If startTime and endTime are not sent, the most recent klines are returned.

**Data Source:**
Database

**Response:**
```javascript
[
  [
    1499040000000,      // Kline open time
    "0.01634790",       // Open price
    "0.80000000",       // High price
    "0.01575800",       // Low price
    "0.01577100",       // Close price
    "148976.11427815",  // Volume
    1499644799999,      // Kline Close time
    "2434.19055334",    // Quote asset volume
    308,                // Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368",      // Taker buy quote asset volume
    "0"                 // Unused field, ignore.
  ]
]
```

### UIKlines

The request is similar to klines having the same parameters and response.

`uiKlines` return modified kline data, optimized for presentation of candlestick charts.

```
GET /api/v3/uiKlines
```

**Weight:**
1

**Parameters:**

Name      | Type   | Mandatory    | Description
------    | ------ | ------------ | ------------
symbol    | STRING | YES          |
interval  | ENUM   | YES          |
startTime | LONG   | NO           |
endTime   | LONG   | NO           |
limit     | INT    | NO           | Default 500; max 1000.

* If `startTime` and `endTime` are not sent, the most recent klines are returned.

**Data Source:**
Database

**Response:**
```javascript
[
  [
    1499040000000,      // Kline open time
    "0.01634790",       // Open price
    "0.80000000",       // High price
    "0.01575800",       // Low price
    "0.01577100",       // Close price
    "148976.11427815",  // Volume
    1499644799999,      // Kline close time
    "2434.19055334",    // Quote asset volume
    308,                // Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368",      // Taker buy quote asset volume
    "0"                 // Unused field. Ignore.
  ]
]
```

### Current average price
Current average price for a symbol.
```
GET /api/v3/avgPrice
```
**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |

**Data Source:**
Memory

**Response:**
```javascript
{
  "mins": 5,
  "price": "9.35751834"
}
```


### 24hr ticker price change statistics
```
GET /api/v3/ticker/24hr
```
24 hour rolling window price change statistics. **Careful** when accessing this with no symbol.

**Weight:**

<table>
<thead>
    <tr>
        <th>Parameter</th>
        <th>Symbols Provided</th>
        <th>Weight</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td rowspan="2">symbol</td>
        <td>1</td>
        <td>1</td>
    </tr>
    <tr>
        <td>symbol parameter is omitted</td>
        <td>40</td>
    </tr>
    <tr>
        <td rowspan="4">symbols</td>
        <td>1-20</td>
        <td>1</td>
    </tr>
    <tr>
        <td>21-100</td>
        <td>20</td>
    </tr>
    <tr>
        <td>101 or more</td>
        <td>40</td>
    </tr>
    <tr>
        <td>symbols parameter is omitted</td>
        <td>40</td>
    </tr>
</tbody>
</table>

**Parameters:**

<table>
<thead>
    <tr>
        <th>Name</th>
        <th>Type</th>
        <th>Mandatory</th>
        <th>Description</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>symbol</td>
        <td>STRING</td>
        <td>NO</td>
        <td rowspan="2">Parameter symbol and symbols cannot be used in combination. <br/> If neither parameter is sent, tickers for all symbols will be returned in an array. <br/><br/>
         Examples of accepted format for the symbols parameter:
         ["BTCUSDT","BNBUSDT"] <br/>
         or <br/>
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
        <td>Supported values: <tt>FULL</tt> or <tt>MINI</tt>. <br/>If none provided, the default is <tt>FULL</tt> </td>
     </tr>
</tbody>
</table>

**Data Source:**
Memory

**Response - FULL:**
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
  "firstId": 28385,   // First tradeId
  "lastId": 28460,    // Last tradeId
  "count": 76         // Trade count
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
    "firstId": 28385,   // First tradeId
    "lastId": 28460,    // Last tradeId
    "count": 76         // Trade count
  }
]
```

**Response - MINI**

```javascript
{
  "symbol":      "BNBBTC",          // Symbol Name
  "openPrice":   "99.00000000",     // Opening price of the Interval
  "highPrice":   "100.00000000",    // Highest price in the interval
  "lowPrice":    "0.10000000",      // Lowest  price in the interval
  "lastPrice":   "4.00000200",      // Closing price of the interval
  "volume":      "8913.30000000",   // Total trade volume (in base asset)
  "quoteVolume": "15.30000000",     // Total trade volume (in quote asset)
  "openTime":    1499783499040,     // Start of the ticker interval
  "closeTime":   1499869899040,     // End of the ticker interval
  "firstId":     28385,             // First tradeId considered
  "lastId":      28460,             // Last tradeId considered
  "count":       76                 // Total trade count
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


### Symbol price ticker
```
GET /api/v3/ticker/price
```
Latest price for a symbol or symbols.

**Weight:**

<table>
<thead>
    <tr>
        <th>Parameter</th>
        <th>Symbols Provided</th>
        <th>Weight</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td rowspan="2">symbol</td>
        <td>1</td>
        <td>1</td>
    </tr>
    <tr>
        <td>symbol parameter is omitted</td>
        <td>2</td>
    </tr>
    <tr>
        <td>symbols</td>
        <td>Any</td>
        <td>2</td>
    </tr>
</tbody>
</table>

**Parameters:**

<table>
<thead>
    <tr>
      <th>Name</th>
      <th>Type</th>
      <th>Mandatory</th>
      <th>Description</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>symbol</td>
        <td>STRING</td>
        <td>NO</td>
        <td rowspan="2"> Parameter symbol and symbols cannot be used in combination. <br/> If neither parameter is sent, prices for all symbols will be returned in an array. <br/><br/>
        Examples of accepted format for the symbols parameter:
         ["BTCUSDT","BNBUSDT"] <br/>
         or <br/>
         %5B%22BTCUSDT%22,%22BNBUSDT%22%5D</td>
    </tr>
    <tr>
        <td>symbols</td>
        <td>STRING</td>
        <td>NO</td>
    </tr>
</tbody>
</table>


**Data Source:**
Memory

**Response:**
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

### Symbol order book ticker
```
GET /api/v3/ticker/bookTicker
```
Best price/qty on the order book for a symbol or symbols.

**Weight:**

<table>
<thead>
    <tr>
        <th>Parameter</th>
        <th>Symbols Provided</th>
        <th>Weight</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td rowspan="2">symbol</td>
        <td>1</td>
        <td>1</td>
    </tr>
    <tr>
        <td>symbol parameter is omitted</td>
        <td>2</td>
    </tr>
    <tr>
        <td>symbols</td>
        <td>Any</td>
        <td>2</td>
    </tr>
</tbody>
</table>

**Parameters:**

<table>
<thead>
    <tr>
      <th>Name</th>
      <th>Type</th>
      <th>Mandatory</th>
      <th>Description</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>symbol</td>
        <td>STRING</td>
        <td>NO</td>
        <td rowspan="2"> Parameter symbol and symbols cannot be used in combination. <br/> If neither parameter is sent, bookTickers for all symbols will be returned in an array.
         <br/><br/>
        Examples of accepted format for the symbols parameter:
         ["BTCUSDT","BNBUSDT"] <br/>
         or <br/>
         %5B%22BTCUSDT%22,%22BNBUSDT%22%5D</td>
    </tr>
    <tr>
        <td>symbols</td>
        <td>STRING</td>
        <td>NO</td>
    </tr>
</tbody>
</table>


**Data Source:**
Memory

**Response:**
```javascript
{
  "symbol": "LTCBTC",
  "bidPrice": "4.00000000",
  "bidQty": "431.00000000",
  "askPrice": "4.00000200",
  "askQty": "9.00000000"
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

### Rolling window price change statistics
```
GET /api/v3/ticker
```

**Note:** This endpoint is different from the `GET /api/v3/ticker/24hr` endpoint.

The window used to compute statistics will be no more than 59999ms from the requested `windowSize`.

`openTime` for `/api/v3/ticker` always starts on a minute, while the `closeTime` is the current time of the request.
As such, the effective window will be up to 59999ms wider than `windowSize`.

E.g. If the `closeTime` is 1641287867099 (January 04, 2022 09:17:47:099 UTC) , and the `windowSize` is `1d`. the `openTime` will be: 1641201420000 (January 3, 2022, 09:17:00)

**Weight:**

2 for each requested <tt>symbol</tt> regardless of <tt>windowSize</tt>. <br/><br/> The weight for this request will cap at 100 once the number of `symbols` in the request is more than 50.

**Parameters**

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
    <td rowspan="2">Either <tt>symbol</tt> or <tt>symbols</tt> must be provided <br/><br/> Examples of accepted format for the <tt>symbols</tt> parameter: <br/> ["BTCUSDT","BNBUSDT"] <br/>or <br/>%5B%22BTCUSDT%22,%22BNBUSDT%22%5D <br/><br/> The maximum number of <tt>symbols</tt> allowed in a request is 100.
    </td>
  </tr>
  <tr>
     <td>symbols</td>
  </tr>
  <tr>
     <td>windowSize</td>
     <td>ENUM</td>
     <td>NO</td>
     <td>Defaults to <tt>1d</tt> if no parameter provided <br/> Supported <tt>windowSize</tt> values: <br/> <tt>1m</tt>,<tt>2m</tt>....<tt>59m</tt> for minutes <br/> <tt>1h</tt>, <tt>2h</tt>....<tt>23h</tt> - for hours <br/> <tt>1d</tt>...<tt>7d</tt> - for days <br/><br/> Units cannot be combined (e.g. <tt>1d2h</tt> is not allowed)</td>
  </tr>
  <tr>
      <td>type</td>
      <td>ENUM</td>
      <td>NO</td>
      <td>Supported values: <tt>FULL</tt> or <tt>MINI</tt>. <br/>If none provided, the default is <tt>FULL</tt> </td>
  </tr>
</table>

**Data Source:**
Database

**Response - FULL**

When using `symbol`:

```javascript
{
  "symbol":             "BNBBTC",
  "priceChange":        "-8.00000000",  // Absolute price change
  "priceChangePercent": "-88.889",      // Relative price change in percent
  "weightedAvgPrice":   "2.60427807",   // QuoteVolume / Volume
  "openPrice":          "9.00000000",
  "highPrice":          "9.00000000",
  "lowPrice":           "1.00000000",
  "lastPrice":          "1.00000000",
  "volume":             "187.00000000",
  "quoteVolume":        "487.00000000", // Sum of (price * volume) for all trades
  "openTime":           1641859200000,  // Open time for ticker window
  "closeTime":          1642031999999,  // Close time for ticker window
  "firstId":            0,              // Trade IDs
  "lastId":             60,
  "count":              61              // Number of trades in the interval
}

```
or

When using `symbols`:

```javascript
[
  {
    "symbol": "BTCUSDT",
    "priceChange": "-154.13000000",        // Absolute price change
    "priceChangePercent": "-0.740",        // Relative price change in percent
    "weightedAvgPrice": "20677.46305250",  // QuoteVolume / Volume
    "openPrice": "20825.27000000",
    "highPrice": "20972.46000000",
    "lowPrice": "20327.92000000",
    "lastPrice": "20671.14000000",
    "volume": "72.65112300",
    "quoteVolume": "1502240.91155513",     // Sum of (price * volume) for all trades
    "openTime": 1655432400000,             // Open time for ticker window
    "closeTime": 1655446835460,            // Close time for ticker window
    "firstId": 11147809,                   // Trade IDs
    "lastId": 11149775,
    "count": 1967                          // Number of trades in the interval
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

**Response - MINI**

When using `symbol`:

```javascript
{
    "symbol": "LTCBTC",
    "openPrice": "0.10000000",
    "highPrice": "2.00000000",
    "lowPrice": "0.10000000",
    "lastPrice": "2.00000000",
    "volume": "39.00000000",
    "quoteVolume": "13.40000000",  // Sum of (price * volume) for all trades
    "openTime": 1656986580000,     // Open time for ticker window
    "closeTime": 1657001016795,    // Close time for ticker window
    "firstId": 0,                  // Trade IDs
    "lastId": 34,
    "count": 35                    // Number of trades in the interval
}
```

OR

When using `symbols`:

```javascript
[
    {
        "symbol": "BNBBTC",
        "openPrice": "0.10000000",
        "highPrice": "2.00000000",
        "lowPrice": "0.10000000",
        "lastPrice": "2.00000000",
        "volume": "39.00000000",
        "quoteVolume": "13.40000000", // Sum of (price * volume) for all trades
        "openTime": 1656986880000,    // Open time for ticker window
        "closeTime": 1657001297799,   // Close time for ticker window
        "firstId": 0,                 // Trade IDs
        "lastId": 34,
        "count": 35                   // Number of trades in the interval
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

## Account endpoints
### New order  (TRADE)
```
POST /api/v3/order  (HMAC SHA256)
```
Send in a new order.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
side | ENUM | YES |
type | ENUM | YES |
timeInForce | ENUM | NO |
quantity | DECIMAL | NO |
quoteOrderQty|DECIMAL|NO|
price | DECIMAL | NO |
newClientOrderId | STRING | NO | A unique id among open orders. Automatically generated if not sent.<br/> Orders with the same `newClientOrderID` can be accepted only when the previous one is filled, otherwise the order will be rejected.
strategyId |INT| NO|
strategyType |INT| NO| The value cannot be less than `1000000`.
stopPrice | DECIMAL | NO | Used with `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders.
trailingDelta|LONG|NO| Used with `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders.
icebergQty | DECIMAL | NO | Used with `LIMIT`, `STOP_LOSS_LIMIT`, and `TAKE_PROFIT_LIMIT` to create an iceberg order.
newOrderRespType | ENUM | NO | Set the response JSON. `ACK`, `RESULT`, or `FULL`; `MARKET` and `LIMIT` order types default to `FULL`, all other orders default to `ACK`.
selfTradePreventionMode |ENUM| NO | The allowed enums is dependent on what is configured on the symbol. The possible supported values are `EXPIRE_TAKER`, `EXPIRE_MAKER`, `EXPIRE_BOTH`, `NONE`.
recvWindow | LONG | NO |The value cannot be greater than ```60000```
timestamp | LONG | YES |


Some additional mandatory parameters based on order `type`:

Type | Additional mandatory parameters | Additional Information
------------ | ------------| ------
`LIMIT` | `timeInForce`, `quantity`, `price`|
`MARKET` | `quantity` or `quoteOrderQty`| `MARKET` orders using the `quantity` field specifies the amount of the `base asset` the user wants to buy or sell at the market price. <br/> E.g. MARKET order on BTCUSDT will specify how much BTC the user is buying or selling. <br/><br/> `MARKET` orders using `quoteOrderQty` specifies the amount the user wants to spend (when buying) or receive (when selling) the `quote` asset; the correct `quantity` will be determined based on the market liquidity and `quoteOrderQty`. <br/> E.g. Using the symbol BTCUSDT: <br/> `BUY` side, the order will buy as many BTC as `quoteOrderQty` USDT can. <br/> `SELL` side, the order will sell as much BTC needed to receive `quoteOrderQty` USDT.
`STOP_LOSS` | `quantity`, `stopPrice` or `trailingDelta`| This will execute a `MARKET` order when the conditions are met. (e.g. `stopPrice` is met or `trailingDelta` is activated)
`STOP_LOSS_LIMIT` | `timeInForce`, `quantity`,  `price`, `stopPrice` or `trailingDelta`
`TAKE_PROFIT` | `quantity`, `stopPrice` or `trailingDelta` | This will execute a `MARKET` order when the conditions are met. (e.g. `stopPrice` is met or `trailingDelta` is activated)
`TAKE_PROFIT_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice` or `trailingDelta` |
`LIMIT_MAKER` | `quantity`, `price`| This is a `LIMIT` order that will be rejected if the order immediately matches and trades as a taker. <br/> This is also known as a POST-ONLY order.

Other info:

* Any `LIMIT` or `LIMIT_MAKER` type order can be made an iceberg order by sending an `icebergQty`.
* Any order with an `icebergQty` MUST have `timeInForce` set to `GTC`.
* For `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT_LIMIT` and `TAKE_PROFIT` orders, `trailingDelta` can be combined with `stopPrice`.
* `MARKET` orders using `quoteOrderQty` will not break `LOT_SIZE` filter rules; the order will execute a `quantity` that will have the notional value as close as possible to `quoteOrderQty`.
Trigger order price rules against market price for both MARKET and LIMIT versions:

* Price above market price: `STOP_LOSS` `BUY`, `TAKE_PROFIT` `SELL`
* Price below market price: `STOP_LOSS` `SELL`, `TAKE_PROFIT` `BUY`

**Data Source:**
Matching Engine

**Response ACK:**
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "orderListId": -1, // Unless OCO, value will be -1
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595
}
```

**Response RESULT:**
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "orderListId": -1, // Unless OCO, value will be -1
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595,
  "price": "0.00000000",
  "origQty": "10.00000000",
  "executedQty": "10.00000000",
  "cummulativeQuoteQty": "10.00000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "MARKET",
  "side": "SELL",
  "workingTime": 1507725176595,
  "selfTradePreventionMode": "NONE"
}
```

**Response FULL:**
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "orderListId": -1, // Unless OCO, value will be -1
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595,
  "price": "0.00000000",
  "origQty": "10.00000000",
  "executedQty": "10.00000000",
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

#### Conditional fields in Order Responses

There are fields in the order responses (e.g. order placement, order query, order cancellation) that appear only if certain conditions are met.

These fields can apply to OCO Orders.

The fields are listed below:

Field          |Description                                                      |Visibility conditions                                           | Examples |
----           | -----                                                           | ---                                                            |---       |
`icebergQty`   | Quantity for the iceberg order | Appears only if the parameter `icebergQty` was sent in the request.| `"icebergQty": "0.00000000"`
`preventedMatchId` |  When used in combination with `symbol`, can be used to query a prevented match. | Appears only if the order expired due to STP.| `"preventedMatchId": 0`
`preventedQuantity` | Order quantity that expired due to STP | Appears only if the order expired due to STP. | `"preventedQuantity": "1.200000"`
`stopPrice`    | Price when the algorithmic order will be triggered | Appears for `STOP_LOSS`. `TAKE_PROFIT`, `STOP_LOSS_LIMIT` and `TAKE_PROFIT_LIMIT` orders.|`"stopPrice": "23500.00000000"`
`strategyId`   | Can be used to label an order that's part of an order strategy. |Appears if the parameter was populated in the request.| `"strategyId": 37463720`
`strategyType` | Can be used to label an order that is using an order strategy.|Appears if the parameter was populated in the request.| `"strategyType": 1000000`
`trailingDelta`| Delta price change required before order activation| Appears for Trailing Stop Orders.|`"trailingDelta": 10`
`trailingTime` | Time when the trailing order is now active and tracking price changes| Appears only for Trailing Stop Orders.| `"trailingTime": -1`

### Test new order (TRADE)
```
POST /api/v3/order/test (HMAC SHA256)
```
Test new order creation and signature/recvWindow long.
Creates and validates a new order but does not send it into the matching engine.

**Weight:**
1

**Parameters:**

Same as `POST /api/v3/order`

**Data Source:**
Memory

**Response:**
```javascript
{}
```

### Query order (USER_DATA)
```
GET /api/v3/order (HMAC SHA256)
```
Check an order's status.

**Weight:**
2

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
origClientOrderId | STRING | NO |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

Notes:
* Either `orderId` or `origClientOrderId` must be sent.
* For some historical orders `cummulativeQuoteQty` will be < 0, meaning the data is not available at this time.

**Data Source:**
Memory => Database

**Response:**
```javascript
{
  "symbol": "LTCBTC",
  "orderId": 1,
  "orderListId": -1                 // This field will always have a value of -1 if not an OCO.
  "clientOrderId": "myOrder1",
  "price": "0.1",
  "origQty": "1.0",
  "executedQty": "0.0",
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
  "workingTime":1499827319559,
  "origQuoteOrderQty": "0.000000",
  "selfTradePreventionMode": "NONE"
}
```

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

### Cancel order (TRADE)
```
DELETE /api/v3/order  (HMAC SHA256)
```
Cancel an active order.

**Weight:**
1

**Parameters:**

Name              | Type   | Mandatory    | Description
------------      | ------ | ------------ | ------------
symbol            | STRING | YES          |
orderId           | LONG   | NO           |
origClientOrderId | STRING | NO           |
newClientOrderId  | STRING | NO           |  Used to uniquely identify this cancel. Automatically generated by default.
cancelRestrictions| ENUM   | NO           | Supported values: <br>`ONLY_NEW` - Cancel will succeed if the order status is `NEW`.<br> `ONLY_PARTIALLY_FILLED ` - Cancel will succeed if order status is `PARTIALLY_FILLED`.
recvWindow        | LONG   | NO           | The value cannot be greater than `60000`.
timestamp         | LONG   | YES          |

Either `orderId` or `origClientOrderId` must be sent.
If both parameters are sent, `orderId` takes precedence.

**Data Source:**
Matching Engine

**Response:**
```javascript
{
  "symbol": "LTCBTC",
  "origClientOrderId": "myOrder1",
  "orderId": 4,
  "orderListId": -1, // Unless part of an OCO, the value will always be -1.
  "clientOrderId": "cancelMyOrder1",
  "price": "2.00000000",
  "origQty": "1.00000000",
  "executedQty": "0.00000000",
  "cummulativeQuoteQty": "0.00000000",
  "status": "CANCELED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "selfTradePreventionMode": "NONE"
}
```

**Note:** The payload above does not show all fields that can appear in the order response. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

#### Regarding `cancelRestrictions`

* If the `cancelRestrictions` value is not any of the supported values, the error will be:
```json
{
    "code": -1145,
    "msg": "Invalid cancelRestrictions"
}
```
* If the order did not pass the conditions for `cancelRestrictions`, the error will be:
```json
{
    "code": -2011,
    "msg": "Order was not canceled due to cancel restrictions."
}
```

### Cancel All Open Orders on a Symbol (TRADE)
```
DELETE /api/v3/openOrders (HMAC SHA256)
```
Cancels all active orders on a symbol.
This includes OCO orders.

**Weight**
1

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Data Source:**
Matching Engine

**Response**
```javascript
[
  {
    "symbol": "BTCUSDT",
    "origClientOrderId": "E6APeyTJvkMvLMYMqu1KQ4",
    "orderId": 11,
    "orderListId": -1,
    "clientOrderId": "pXLV6Hz6mprAcVYpVMTGgx",
    "price": "0.089853",
    "origQty": "0.178622",
    "executedQty": "0.000000",
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
    "price": "0.090430",
    "origQty": "0.178622",
    "executedQty": "0.000000",
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
        "price": "0.668611",
        "origQty": "0.690354",
        "executedQty": "0.000000",
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
        "price": "0.008791",
        "origQty": "0.690354",
        "executedQty": "0.000000",
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

### Cancel an Existing Order and Send a New Order (TRADE)

```
POST /api/v3/order/cancelReplace
```
Cancels an existing order and places a new order on the same symbol.

Filters and Order Count are evaluated before the processing of the cancellation and order placement occurs.

A new order that was not attempted (i.e. when `newOrderResult: NOT_ATTEMPTED` ), will still increase the order count by 1.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
side   |ENUM| YES|
type   |ENUM| YES|
cancelReplaceMode|ENUM|YES| The allowed values are: <br/> `STOP_ON_FAILURE` - If the cancel request fails, the new order placement will not be attempted. <br/> `ALLOW_FAILURE` - new order placement will be attempted even if cancel request fails.
timeInForce|ENUM|NO|
quantity|DECIMAL|NO|
quoteOrderQty |DECIMAL|NO
price |DECIMAL|NO
cancelNewClientOrderId|STRING|NO| Used to uniquely identify this cancel. Automatically generated by default.
cancelOrigClientOrderId|STRING| NO| Either the `cancelOrigClientOrderId` or `cancelOrderId` must be provided. If both are provided, `cancelOrderId` takes precedence.
cancelOrderId|LONG|NO| Either the `cancelOrigClientOrderId` or `cancelOrderId` must be provided. If both are provided, `cancelOrderId` takes precedence.
newClientOrderId |STRING|NO| Used to identify the new order.
strategyId |INT| NO|
strategyType |INT| NO| The value cannot be less than `1000000`.
stopPrice|DECIMAL|NO|
trailingDelta|LONG|NO|
icebergQty|DECIMAL|NO|
newOrderRespType|ENUM|NO|Allowed values: <br/> `ACK`, `RESULT`, `FULL` <br/> `MARKET` and `LIMIT` orders types default to `FULL`; all other orders default to `ACK`
selfTradePreventionMode |ENUM| NO | The allowed enums is dependent on what is configured on the symbol. The possible supported values are `EXPIRE_TAKER`, `EXPIRE_MAKER`, `EXPIRE_BOTH`, `NONE`.
cancelRestrictions| ENUM   | NO           | Supported values: <br>`ONLY_NEW` - Cancel will succeed if the order status is `NEW`.<br> `ONLY_PARTIALLY_FILLED ` - Cancel will succeed if order status is `ONLY_PARTIALLY_FILLED`. For more information please refer to [Regarding `cancelRestrictions`](#regarding-cancelrestrictions)
recvWindow | LONG | NO | The value cannot be greater than `60000`
timestamp | LONG | YES |


Similar to `POST /api/v3/order`, additional mandatory parameters are determined by `type`.

Response format varies depending on whether the processing of the message succeeded, partially succeeded, or failed.

**Data Source:**
Matching Engine

**Response SUCCESS:**

```javascript
// Both the cancel order placement and new order placement succeeded.
{
  "cancelResult": "SUCCESS",
  "newOrderResult": "SUCCESS",
  "cancelResponse": {
    "symbol": "BTCUSDT",
    "origClientOrderId": "DnLo3vTAQcjha43lAZhZ0y",
    "orderId": 9,
    "orderListId": -1,
    "clientOrderId": "osxN3JXAtJvKvCqGeMWMVR",
    "price": "0.01000000",
    "origQty": "0.000100",
    "executedQty": "0.00000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "CANCELED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL",
    "selfTradePreventionMode": "NONE"
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
    "cummulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "workingTime": 1669277163808,
    "fills": [],
    "selfTradePreventionMode": "NONE"
  }
}
```

**Response when Cancel Order Fails with STOP_ON FAILURE:**
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

**Response when Cancel Order Succeeds but New Order Placement Fails:**
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
      "cummulativeQuoteQty": "0.000000",
      "status": "CANCELED",
      "timeInForce": "GTC",
      "type": "LIMIT_MAKER",
      "side": "SELL",
      "selfTradePreventionMode": "NONE"
    },
    "newOrderResponse": {
      "code": -2010,
      "msg": "Order would immediately match and take."
    }
  }
}
```

**Response when Cancel Order fails with ALLOW_FAILURE:**

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

**Response when both Cancel Order and New Order Placement fail:**

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

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

### Current open orders (USER_DATA)
```
GET /api/v3/openOrders  (HMAC SHA256)
```
Get all open orders on a symbol. **Careful** when accessing this with no symbol.

**Weight:**
3 for a single symbol; **40** when the symbol parameter is omitted

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

* If the symbol is not sent, orders for all symbols will be returned in an array.

**Data Source:**
Memory => Database

**Response:**
```javascript
[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "orderListId": -1, // Unless OCO, the value will always be -1
    "clientOrderId": "myOrder1",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
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

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

### All orders (USER_DATA)
```
GET /api/v3/allOrders (HMAC SHA256)
```
Get all account orders; active, canceled, or filled.

**Weight:**
10 with symbol

**Data Source:**
Database

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Notes:**
* If `orderId` is set, it will get orders >= that `orderId`. Otherwise most recent orders are returned.
* For some historical orders `cummulativeQuoteQty` will be < 0, meaning the data is not available at this time.
* If `startTime` and/or `endTime` provided, `orderId`  is not required.

**Response:**
```javascript
[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "orderListId": -1, // Unless OCO, the value will always be -1
    "clientOrderId": "myOrder1",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
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
    "selfTradePreventionMode": "NONE",
  }
]
```

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

### New OCO (TRADE)

```
POST /api/v3/order/oco (HMAC SHA256)
```

**Weight**: 1

Send in a new OCO

**Parameters**:

Name |Type| Mandatory | Description
-----|-----|----------| -----------
symbol|STRING| YES|
listClientOrderId|STRING|NO| A unique Id for the entire orderList
side|ENUM|YES|
quantity|DECIMAL|YES|
limitClientOrderId|STRING|NO| A unique Id for the limit order
price|DECIMAL|YES|
limitStrategyId |INT| NO
limitStrategyType | INT| NO | The value cannot be less than `1000000`.
limitIcebergQty|DECIMAL|NO| Used to make the `LIMIT_MAKER` leg an iceberg order.
trailingDelta|LONG|NO|
stopClientOrderId |STRING|NO| A unique Id for the stop loss/stop loss limit leg
stopPrice |DECIMAL| YES
stopStrategyId |INT| NO
stopStrategyType |INT| NO | The value cannot be less than `1000000`.
stopLimitPrice|DECIMAL|NO | If provided, `stopLimitTimeInForce` is required.
stopIcebergQty|DECIMAL|NO| Used with `STOP_LOSS_LIMIT` leg to make an iceberg order.
stopLimitTimeInForce|ENUM|NO| Valid values are `GTC`/`FOK`/`IOC`
newOrderRespType|ENUM|NO| Set the response JSON.
selfTradePreventionMode |ENUM| NO | The allowed enums is dependent on what is configured on the symbol. The possible supported values are `EXPIRE_TAKER`, `EXPIRE_MAKER`, `EXPIRE_BOTH`, `NONE`.
recvWindow|LONG|NO| The value cannot be greater than `60000`
timestamp|LONG|YES|


Additional Info:
* Price Restrictions:
    * `SELL`: Limit Price > Last Price > Stop Price
    * `BUY`: Limit Price < Last Price < Stop Price
* Quantity Restrictions:
    * Both legs must have the same quantity.
    * ```ICEBERG``` quantities however do not have to be the same
* Order Rate Limit
    * `OCO` counts as 2 orders against the order rate limit.

**Data Source:**
Matching Engine

**Response:**

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


### Cancel OCO (TRADE)

```
DELETE /api/v3/orderList (HMAC SHA256)
```

**Weight**: 1

Cancel an entire Order List

**Parameters:**

Name| Type| Mandatory| Description
----| ----|------|------
symbol|STRING| YES|
orderListId|LONG|NO| Either ```orderListId``` or ```listClientOrderId``` must be provided
listClientOrderId|STRING|NO| Either ```orderListId``` or ```listClientOrderId``` must be provided
newClientOrderId|STRING|NO| Used to uniquely identify this cancel. Automatically generated by default
recvWindow|LONG|NO| The value cannot be greater than ```60000```
timestamp|LONG|YES|

Additional notes:
* Canceling an individual leg will cancel the entire OCO
* If both `orderListId` and `listClientOrderId` are sent, `orderListId` takes precedence.

**Data Source:**
Matching Engine

**Response**

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
      "price": "1.00000000",
      "origQty": "10.00000000",
      "executedQty": "0.00000000",
      "cummulativeQuoteQty": "0.00000000",
      "status": "CANCELED",
      "timeInForce": "GTC",
      "type": "STOP_LOSS_LIMIT",
      "side": "SELL",
      "stopPrice": "1.00000000",
      "selfTradePreventionMode": "NONE"
    },
    {
      "symbol": "LTCBTC",
      "origClientOrderId": "TXOvglzXuaubXAaENpaRCB",
      "orderId": 3,
      "orderListId": 0,
      "clientOrderId": "unfWT8ig8i0uj6lPuYLez6",
      "price": "3.00000000",
      "origQty": "10.00000000",
      "executedQty": "0.00000000",
      "cummulativeQuoteQty": "0.00000000",
      "status": "CANCELED",
      "timeInForce": "GTC",
      "type": "LIMIT_MAKER",
      "side": "SELL",
      "selfTradePreventionMode": "NONE"
    }
  ]
}
```


### Query OCO (USER_DATA)

```
GET /api/v3/orderList (HMAC SHA256)
```

**Weight**: 2

Retrieves a specific OCO based on provided optional parameters

**Parameters**:

Name| Type|Mandatory| Description
----|-----|----|----------
orderListId|LONG|NO|  Either ```orderListId``` or ```listClientOrderId``` must be provided
origClientOrderId|STRING|NO| Either ```orderListId``` or ```listClientOrderId``` must be provided
recvWindow|LONG|NO| The value cannot be greater than ```60000```
timestamp|LONG|YES|

**Data Source:**
Database

**Response:**

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


### Query all OCO (USER_DATA)

```
GET /api/v3/allOrderList (HMAC SHA256)
```

**Weight**: 10

Retrieves all OCO based on provided optional parameters

**Parameters**

Name|Type| Mandatory| Description
----|----|----|---------
fromId|LONG|NO| If supplied, neither ```startTime``` or ```endTime``` can be provided
startTime|LONG|NO|
endTime|LONG|NO|
limit|INT|NO| Default Value: 500; Max Value: 1000
recvWindow|LONG|NO| The value cannot be greater than ```60000```
timestamp|LONG|YES|

**Data Source:**
Database

**Response:**

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

### Query Open OCO (USER_DATA)

```
GET /api/v3/openOrderList (HMAC SHA256)
```

Weight: 3

**Parameters**

Name| Type|Mandatory| Description
----|-----|---|------------------
recvWindow|LONG|NO| The value cannot be greater than ```60000```
timestamp|LONG|YES|

**Data Source:**
Database

**Response:**

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

### Account information (USER_DATA)
```
GET /api/v3/account (HMAC SHA256)
```
Get current account information.

**Weight:**
10

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Data Source:**
Memory => Database

**Response:**
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
  "brokered":false,
  "requireSelfTradePrevention": false,
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
  ]
}
```

### Account trade list (USER_DATA)
```
GET /api/v3/myTrades  (HMAC SHA256)
```
Get trades for a specific account and symbol.

**Weight:**
10 with symbol

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId|LONG|NO| This can only be used in combination with `symbol`.
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO | TradeId to fetch from. Default gets most recent trades.
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Notes:**
* If `fromId` is set, it will get trades >= that `fromId`.
Otherwise most recent trades are returned.
* The time between `startTime` and `endTime` can't be longer than 24 hours.
* These are the supported combinations of all parameters:
  * `symbol`
  * `symbol` + `orderId`
  * `symbol` + `startTime`
  * `symbol` + `endTime`
  * `symbol` + `fromId`
  * `symbol` + `startTime` + `endTime`
  * `symbol`+ `orderId` + `fromId`

**Data Source:**
Memory => Database

**Response:**
```javascript
[
  {
    "symbol": "BNBBTC",
    "id": 28457,
    "orderId": 100234,
    "orderListId": -1,
    "price": "4.00000100",
    "qty": "12.00000000",
    "quoteQty": "48.000012",
    "commission": "10.10000000",
    "commissionAsset": "BNB",
    "time": 1499865549590,
    "isBuyer": true,
    "isMaker": false,
    "isBestMatch": true
  }
]
```

### Query Current Order Count Usage (TRADE)
```
GET /api/v3/rateLimit/order
```

Displays the user's current order count usage for all intervals.


**Weight:**
20

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Data Source:**
Memory

**Response:**

```json
[

  {
    "rateLimitType": "ORDERS",
    "interval": "SECOND",
    "intervalNum": 10,
    "limit": 50,
    "count": 0
  },
  {
    "rateLimitType": "ORDERS",
    "interval": "DAY",
    "intervalNum": 1,
    "limit": 160000,
    "count": 0
  }
]
```

### Query Prevented Matches (USER_DATA)

```
GET /api/v3/myPreventedMatches
```

Displays the list of orders that were expired due to STP.

These are the combinations supported:

* `symbol` + `preventedMatchId`
* `symbol` + `orderId`
* `symbol` + `orderId` + `fromPreventedMatchId` (`limit` will default to 500)
* `symbol` + `orderId` + `fromPreventedMatchId` + `limit`

**Parameters:**

Name                | Type   | Mandatory    | Description
------------        | ----   | ------------ | ------------
symbol              | STRING | YES          |
preventedMatchId    |LONG    | NO           |
orderId             |LONG    | NO           |
fromPreventedMatchId|LONG    | NO           |
limit               |INT     | NO           | Default: `500`; Max: `1000`
recvWindow          | LONG   | NO           | The value cannot be greater than `60000`
timestamp           | LONG   | YES          |

**Weight**

Case                            | Weight
----                            | -----
If `symbol` is invalid          | 1
Querying by `preventedMatchId`  | 1
Querying by `orderId`           | 10

**Data Source:**

Database

**Response:**

```json
[
  {
    "symbol": "BTCUSDT",
    "preventedMatchId": 1,
    "takerOrderId": 5,
    "makerOrderId": 3,
    "tradeGroupId": 1,
    "selfTradePreventionMode": "EXPIRE_MAKER",
    "price": "1.100000",
    "makerPreventedQuantity": "1.300000",
    "transactTime": 1669101687094
  }
]
```


## User data stream endpoints
Specifics on how user data streams work can be found [here.](https://github.com/binance/binance-spot-api-docs/blob/master/user-data-stream.md)

### Start user data stream (USER_STREAM)
```
POST /api/v3/userDataStream
```
Start a new user data stream. The stream will close after 60 minutes unless a keepalive is sent.

**Weight:**
1

**Parameters:**
NONE

**Data Source:**
Memory

**Response:**
```javascript
{
  "listenKey": "pqia91ma19a5s61cv6a81va65sdf19v8a65a1a5s61cv6a81va65sdf19v8a65a1"
}
```

### Keepalive user data stream (USER_STREAM)
```
PUT /api/v3/userDataStream
```
Keepalive a user data stream to prevent a time out. User data streams will close after 60 minutes. It's recommended to send a ping about every 30 minutes.

**Weight:**
1

**Data Source"**
Memory

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES

**Response:**
```javascript
{}
```

### Close user data stream (USER_STREAM)
```
DELETE /api/v3/userDataStream
```
Close out a user data stream.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES

**Data Source:**
Memory

**Response:**
```javascript
{}
```
