<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Public Rest API for Binance SPOT Testnet](#public-rest-api-for-binance-spot-testnet)
  - [General API Information](#general-api-information)
  - [HTTP Return Codes](#http-return-codes)
  - [Error Codes](#error-codes)
  - [General Information on Endpoints](#general-information-on-endpoints)
  - [LIMITS](#limits)
    - [General Info on Limits](#general-info-on-limits)
    - [IP Limits](#ip-limits)
    - [Unfilled Order Count](#unfilled-order-count)
  - [Data Sources](#data-sources)
  - [Request Security](#request-security)
    - [SIGNED Endpoint security](#signed-endpoint-security)
      - [Signature Case Sensitivity](#signature-case-sensitivity)
    - [Timing security](#timing-security)
    - [SIGNED Endpoint Examples for POST /api/v3/order](#signed-endpoint-examples-for-post-apiv3order)
      - [HMAC Keys](#hmac-keys)
      - [RSA Keys](#rsa-keys)
      - [Ed25519 Keys](#ed25519-keys)
- [Public API Endpoints](#public-api-endpoints)
  - [General endpoints](#general-endpoints)
    - [Test connectivity](#test-connectivity)
    - [Check server time](#check-server-time)
    - [Exchange information](#exchange-information)
  - [Market Data endpoints](#market-data-endpoints)
    - [Order book](#order-book)
    - [Recent trades list](#recent-trades-list)
    - [Old trade lookup](#old-trade-lookup)
    - [Compressed/Aggregate trades list](#compressedaggregate-trades-list)
    - [Kline/Candlestick data](#klinecandlestick-data)
    - [UIKlines](#uiklines)
    - [Current average price](#current-average-price)
    - [24hr ticker price change statistics](#24hr-ticker-price-change-statistics)
    - [Trading Day Ticker](#trading-day-ticker)
    - [Symbol price ticker](#symbol-price-ticker)
    - [Symbol order book ticker](#symbol-order-book-ticker)
    - [Rolling window price change statistics](#rolling-window-price-change-statistics)
  - [Trading endpoints](#trading-endpoints)
    - [New order (TRADE)](#new-order-trade)
    - [Test new order (TRADE)](#test-new-order-trade)
    - [Query order (USER_DATA)](#query-order-user_data)
    - [Cancel order (TRADE)](#cancel-order-trade)
    - [Cancel All Open Orders on a Symbol (TRADE)](#cancel-all-open-orders-on-a-symbol-trade)
    - [Cancel an Existing Order and Send a New Order (TRADE)](#cancel-an-existing-order-and-send-a-new-order-trade)
    - [Order Amend Keep Priority (TRADE)](#order-amend-keep-priority-trade)
    - [Order lists](#order-lists)
      - [New Order list - OCO (TRADE)](#new-order-list---oco-trade)
      - [New Order list - OTO (TRADE)](#new-order-list---oto-trade)
      - [New Order list - OTOCO (TRADE)](#new-order-list---otoco-trade)
      - [New Order List - OPO (TRADE)](#new-order-list---opo-trade)
      - [New Order List - OPOCO (TRADE)](#new-order-list---opoco-trade)
      - [Cancel Order list (TRADE)](#cancel-order-list-trade)
    - [SOR](#sor)
      - [New order using SOR (TRADE)](#new-order-using-sor-trade)
      - [Test new order using SOR (TRADE)](#test-new-order-using-sor-trade)
  - [Account Endpoints](#account-endpoints)
    - [Account information (USER_DATA)](#account-information-user_data)
    - [Current open orders (USER_DATA)](#current-open-orders-user_data)
    - [All orders (USER_DATA)](#all-orders-user_data)
    - [Query Order list (USER_DATA)](#query-order-list-user_data)
    - [Query all Order lists (USER_DATA)](#query-all-order-lists-user_data)
    - [Query Open Order lists (USER_DATA)](#query-open-order-lists-user_data)
    - [Account trade list (USER_DATA)](#account-trade-list-user_data)
    - [Query Unfilled Order Count (USER_DATA)](#query-unfilled-order-count-user_data)
    - [Query Prevented Matches (USER_DATA)](#query-prevented-matches-user_data)
    - [Query Allocations (USER_DATA)](#query-allocations-user_data)
    - [Query Commission Rates (USER_DATA)](#query-commission-rates-user_data)
    - [Query Order Amendments (USER_DATA)](#query-order-amendments-user_data)
    - [Query relevant filters (USER_DATA)](#query-relevant-filters-user_data)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Public Rest API for Binance SPOT Testnet

## General API Information
* The base endpoint is **https://testnet.binance.vision/api**
* Responses are in JSON by default. To receive responses in SBE, refer to the [SBE FAQ](../faqs/sbe_faq.md) page.
* Data is returned in **chronological order**, unless noted otherwise.
  * Without `startTime` or `endTime`, returns the most recent items up to the limit.
  * With `startTime`, returns oldest items from `startTime` up to the limit.
  * With `endTime`, returns most recent items up to `endTime` and the limit.
  * With both, behaves like `startTime` but does not exceed `endTime`.
* All time and timestamp related fields in the JSON responses are in **milliseconds by default.** To receive the information in microseconds, please add the header `X-MBX-TIME-UNIT:MICROSECOND` or `X-MBX-TIME-UNIT:microsecond`.
* We support HMAC, RSA, and Ed25519 keys. For more information, please see [API Key types](../faqs/api_key_types.md).
* Timestamp parameters (e.g. `startTime`, `endTime`, `timestamp`) can be passed in milliseconds or microseconds.
* If there are enums or terms you want clarification on, please see the [SPOT Glossary](../faqs/spot_glossary.md) for more information.
* APIs have a timeout of 10 seconds when processing a request. If a response from the Matching Engine takes longer than this, the API responds with "Timeout waiting for response from backend server. Send status unknown; execution status unknown." [(-1007 TIMEOUT)](errors.md#-1007-timeout)
  * This does not always mean that the request failed in the Matching Engine.
  * If the status of the request has not appeared in [User Data Stream](user-data-stream.md), please perform an API query for its status.
* **Please avoid SQL keywords in requests** as they may trigger a security block by a WAF (Web Application Firewall) rule. See https://www.binance.com/en/support/faq/detail/360004492232 for more details.
* If your request contains a symbol name containing non-ASCII characters, then the response may contain non-ASCII characters encoded in UTF-8.
* Some endpoints may return asset and/or symbol names containing non-ASCII characters encoded in UTF-8 even if the request did not contain non-ASCII characters.

## HTTP Return Codes

* HTTP `4XX` return codes are used for malformed requests; the issue is on the sender's side.
* HTTP `403` return code is used when a WAF (Web Application Firewall) rule has been violated. This can indicate a rate limit violation or a security block. See https://www.binance.com/en/support/faq/detail/360004492232 for more details.
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

## LIMITS

### General Info on Limits
* The following `intervalLetter` values for headers:
    * SECOND => S
    * MINUTE => M
    * HOUR => H
    * DAY => D
* `intervalNum` describes the amount of the interval. For example, `intervalNum` 5 with `intervalLetter` M means "Every 5 minutes".
* The `/api/v3/exchangeInfo` `rateLimits` array contains objects related to the exchange's `RAW_REQUESTS`, `REQUEST_WEIGHT`, and `ORDERS` rate limits. These are further defined in the `ENUM definitions` section under `Rate limiters (rateLimitType)`.
* Requests fail with HTTP status code 429 when you exceed the request rate limit.

### IP Limits
* Every request will contain `X-MBX-USED-WEIGHT-(intervalNum)(intervalLetter)` in the response headers which has the current used weight for the IP for all request rate limiters defined.
* Each route has a `weight` which determines for the number of requests each endpoint counts for. Heavier endpoints and endpoints that do operations on multiple symbols will have a heavier `weight`.
* When a 429 is received, it's your obligation as an API to back off and not spam the API.
* **Repeatedly violating rate limits and/or failing to back off after receiving 429s will result in an automated IP ban (HTTP status 418).**
* IP bans are tracked and **scale in duration** for repeat offenders, **from 2 minutes to 3 days**.
* A `Retry-After` header is sent with a 418 or 429 responses and will give the **number of seconds** required to wait, in the case of a 429, to prevent a ban, or, in the case of a 418, until the ban is over.
* **The limits on the API are based on the IPs, not the API keys.**

### Unfilled Order Count
* Every successful order response will contain a `X-MBX-ORDER-COUNT-(intervalNum)(intervalLetter)` header indicating how many orders you have placed for that interval. <br></br> To monitor this, refer to [`GET api/v3/rateLimit/order`](#query-unfilled-order-count).
* Rejected/unsuccessful orders are not guaranteed to have `X-MBX-ORDER-COUNT-**` headers in the response.
* If you have exceeded this, you will receive a 429 error with the `Retry-After` header.
* **Please note that if your orders are consistently filled by trades, you can continuously place orders on the API**. For more information, please see [Spot Unfilled Order Count Rules](../faqs/order_count_decrement.md).
* **The number of unfilled orders is tracked for each account.**

## Data Sources
* The API system is asynchronous, so some delay in the response is normal and expected.
* Each endpoint has a data source indicating where the data is being retrieved, and thus which endpoints have the most up-to-date response.

These are the three sources, ordered by least to most potential for delays in data updates.

  * **Matching Engine** - the data is from the Matching Engine
  * **Memory** - the data is from a server's local or external memory
  * **Database** - the data is taken directly from a database

Some endpoints can have more than 1 data source. (e.g. Memory => Database)
This means that the endpoint will check the first Data Source, and if it cannot find the value it's looking for it will check the next one.

## Request Security

* Each endpoint has a security type indicating required API key permissions, shown next to the endpoint name (e.g., [New order (TRADE)](#place-new-order-trade)).
* If unspecified, the security type is `NONE`.
* Except for `NONE`, all endpoints with a security type are considered `SIGNED` requests (i.e. including a `signature`), except for [listenKey management](#user-data-stream-requests).
* Secure endpoints require a valid API key to be specified and authenticated.
  * API keys can be created on the [SPOT Test Network](https://testnet.binance.vision) upon logging in with your Github account.
  * **Both API key and secret key are sensitive.** Never share them with anyone.
    If you notice unusual activity in your account, immediately revoke all the keys and contact Binance support.
* API keys can be configured to allow access only to certain types of secure endpoints.
  * For example, you can have an API key with `TRADE` permission for trading,
    while using a separate API key with `USER_DATA` permission to monitor your order status.
  * By default, an API key cannot `TRADE`. You need to enable trading in API Management first.

Security type | Description
------------- | ------------
`NONE`        | Public market data
`TRADE`       | Trading on the exchange, placing and canceling orders
`USER_DATA`   | Private account information, such as order status and your trading history
`USER_STREAM` | Managing User Data Stream subscriptions

### SIGNED Endpoint security

* `SIGNED` endpoints require an additional parameter, `signature`, to be sent in the `query string` or `request body`.

#### Signature Case Sensitivity

* **HMAC:** Signatures generated using HMAC are **not case-sensitive**. This means the signature string can be verified regardless of letter casing.
* **RSA:** Signatures generated using RSA are **case-sensitive**.
* **Ed25519:** Signatures generated using Ed25519 are also **case-sensitive**

Please consult [SIGNED request example (HMAC)](#hmac-keys), [SIGNED request example (RSA)](#rsa-keys), and [SIGNED request example (Ed25519)](#ed25519-keys) on how to compute signature, depending on which API key type you are using.

<a id="timingsecurity"></a>

### Timing security
* `SIGNED` requests also require a `timestamp` parameter which should be the current timestamp either in milliseconds or microseconds. (See [General API Information](#general-api-information))
* An additional optional parameter, `recvWindow`, specifies for how long the request stays valid and may only be specified in milliseconds.
  * `recvWindow` supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
  * If `recvWindow` is not sent, **it defaults to 5000 milliseconds**.
  * Maximum `recvWindow` is 60000 milliseconds.
* Request processing logic is as follows:

```javascript
serverTime = getCurrentTime()
if (timestamp < (serverTime + 1 second) && (serverTime - timestamp) <= recvWindow) {
  // begin processing request
  serverTime = getCurrentTime()
  if (serverTime - timestamp) <= recvWindow {
    // forward request to Matching Engine
  } else {
    // reject request
  }
  // finish processing request
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

### SIGNED Endpoint Examples for POST /api/v3/order

#### HMAC Keys

The signature payload of your request is the query string concatenated without separator to the HTTP body. Any non-ASCII character must be percent-encoded before signing.

Here is a step-by-step example of how to send a valid signed payload from the Linux command line using `echo`, `openssl`, and `curl`. There is one example with a symbol name comprised entirely of ASCII characters and one example with a symbol name containing non-ASCII characters.

Example API key and secret key:

Key | Value
------------ | ------------
`apiKey` | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
`secretKey` | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j

**WARNING: DO NOT SHARE YOUR API KEY AND SECRET KEY WITH ANYONE.**

The example keys are provided here only for illustrative purposes.

Example of request with a symbol name comprised entirely of ASCII characters:

Parameter | Value
------------ | ------------
`symbol` | LTCBTC
`side` | BUY
`type` | LIMIT
`timeInForce` | GTC
`quantity` | 1
`price` | 0.1
`recvWindow` | 5000
`timestamp` | 1499827319559

Example of a request with a symbol name containing non-ASCII characters:

Parameter | Value
------------ | ------------
`symbol` | １２３４５６
`side` | BUY
`type` | LIMIT
`timeInForce` | GTC
`quantity` | 1
`price` | 0.1
`recvWindow` | 5000
`timestamp` | 1499827319559

**Step 1: Construct the signature payload**

1. Format parameters as `parameter=value` pairs separated by `&`.
2. Percent-encode the string.

For the first set of example parameters (ASCII only), the `parameter=value` string should look like this:

```console
symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
```

After percent-encoding, the signature payload should look like this:

```console
symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
```

For the second set of example parameters (some non-ASCII characters), the `parameter=value` string should look like this:

```console
symbol=１２３４５６&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
```

After percent-encoding, the signature payload should look like this:

```console
symbol=%EF%BC%91%EF%BC%92%EF%BC%93%EF%BC%94%EF%BC%95%EF%BC%96&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
```

**Step 2: Compute the signature**

1. Use the `secretKey` of your API key as the signing key for the HMAC-SHA-256 algorithm.
2. Sign the signature payload constructed in Step 1.
3. Encode the HMAC-SHA-256 output as a hex string.

Note that `secretKey` and the payload are **case-sensitive**, while the resulting signature value is case-insensitive.

**Example commands**

For the first set of example parameters (ASCII only):

```console
$ echo -n "symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"

c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
```

For the second set of example parameters (some non-ASCII characters):

```console
$ echo -n "symbol=%EF%BC%91%EF%BC%92%EF%BC%93%EF%BC%94%EF%BC%95%EF%BC%96&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"

e1353ec6b14d888f1164ae9af8228a3dbd508bc82eb867db8ab6046442f33ef3
```

**Step 3: Add signature to the request**

Complete the request by adding the `signature` parameter to the query string.

For the first set of example parameters (ASCII only):

```console
curl -s -v -H "X-MBX-APIKEY: $apiKey" -X POST "https://testnet.binance.vision/api/v3/order?symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71"
```

For the second set of example parameters (some non-ASCII characters)

```console
curl -s -v -H "X-MBX-APIKEY: $apiKey" -X POST "https://testnet.binance.vision/api/v3/order?symbol=%EF%BC%91%EF%BC%92%EF%BC%93%EF%BC%94%EF%BC%95%EF%BC%96&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=e1353ec6b14d888f1164ae9af8228a3dbd508bc82eb867db8ab6046442f33ef3"
```

Here is a sample Bash script performing all the steps above:

```bash
apiKey="vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A"
secretKey="NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"

payload="symbol=LTCBTC&side=BUY&type=LIMIT&timeInForce=GTC&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559"

# Sign the request

signature=$(echo -n "$payload" | openssl dgst -sha256 -hmac "$secretKey")
signature=${signature#*= }    # Keep only the part after the "= "

# Send the request

curl -H "X-MBX-APIKEY: $apiKey" -X POST "https://testnet.binance.vision/api/v3/order?$payload&signature=$signature"

```

#### RSA Keys

The signature payload of your request is the query string concatenated without separator to the HTTP body. Any non-ASCII character must be percent-encoded before signing.

To get your API key, you need to upload your RSA Public Key to your account and a corresponding API key will be provided for you.

Only `PKCS#8` keys are supported.

There is one example with a symbol name comprised entirely of ASCII characters and one example with a symbol name containing non-ASCII characters.

These examples assume the private key is stored in the file `./test-prv-key.pem`.

Key | Value
------------ | ------------
`apiKey` | CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ

Example of request with a symbol name comprised entirely of ASCII characters:

Parameter | Value
------------ | ------------
`symbol` | BTCUSDT
`side` | SELL
`type` | LIMIT
`timeInForce` | GTC
`quantity` | 1
`price` | 0.2
`timestamp` | 1668481559918
`recvWindow` | 5000

Example of a request with a symbol name containing non-ASCII characters:

Parameter | Value
------------ | ------------
`symbol` | １２３４５６
`side` | SELL
`type` | LIMIT
`timeInForce` | GTC
`quantity` | 1
`price` | 0.2
`timestamp` | 1668481559918
`recvWindow` | 5000

**Step 1: Construct the signature payload**

1. Format parameters as `parameter=value` pairs separated by `&`.
2. Percent-encode the string.

For the first set of example parameters (ASCII only), the `parameter=value` string should look like this:

```console
symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000
```

After percent-encoding, the signature payload should look like this:

```console
symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000
```

For the second set of example parameters (some non-ASCII characters), the `parameter=value` string should look like this:

```console
symbol=１２３４５６=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000
```

After percent-encoding, the signature payload should look like this:

```console
symbol=%EF%BC%91%EF%BC%92%EF%BC%93%EF%BC%94%EF%BC%95%EF%BC%96&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000
```

**Step 2: Compute the signature**

1. Sign the signature payload constructed in Step 1 using the RSASSA-PKCS1-v1_5 algorithm with SHA-256 hash function.
2. Encode the output in base64.

Note that the payload and the resulting `signature` are **case-sensitive**.

For the first set of example parameters (ASCII only):

```console
$  echo -n 'symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000' | openssl dgst -sha256 -sign ./test-prv-key.pem | openssl enc -base64 -A | tr -d '\n'
HZ8HOjiJ1s/igS9JA+n7+7Ti/ihtkRF5BIWcPIEluJP6tlbFM/Bf44LfZka/iemtahZAZzcO9TnI5uaXh3++lrqtNonCwp6/245UFWkiW1elpgtVAmJPbogcAv6rSlokztAfWk296ZJXzRDYAtzGH0gq7CgSJKfH+XxaCmR0WcvlKjNQnp12/eKXJYO4tDap8UCBLuyxDnR7oJKLHQHJLP0r0EAVOOSIbrFang/1WOq+Jaq4Efc4XpnTgnwlBbWTmhWDR1pvS9iVEzcSYLHT/fNnMRxFc7u+j3qI//5yuGuu14KR0MuQKKCSpViieD+fIti46sxPTsjSemoUKp0oXA==
```

For the second set of example parameters (some non-ASCII characters):

```console
$  echo -n 'symbol=%EF%BC%91%EF%BC%92%EF%BC%93%EF%BC%94%EF%BC%95%EF%BC%96&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000' | openssl dgst -sha256 -sign ./test-prv-key.pem | openssl enc -base64 -A | tr -d '\n'

qJtv66wyp/1mZE+mIFAAMUoTe8xkmLN7/eAZjuC9x1ocxovItHLl/sNK7Wq8QjgiHqGn0bb8P7yVvGBEd1gFe71NQ8aM0M+JNIMz5UFxfeA53rXjFlvsyH1Sig+OuO9Nz5nhCaJ6bEfj2iuv7w27pB3L8MVqmoCi6D9C/QMiLxtPaR70CxtnvoOlIgPmpv2bQy029A31NEK19ieVLkoyp1EUkXRaX3v0mohx8yMnUG1dhX9nUg3Oy8TYZ03DQy7kHDGkMKisNX7rt/GuGx1HIgjFclDGLsbAFIodvSLjm9FbseasMELoxlAJDlwRnW8zo5sQmL0Fz7ao935QBynrng==
```

3. Percent-encode the base64 string.

For the first set of example parameters (ASCII only):

```console
HZ8HOjiJ1s%2FigS9JA%2Bn7%2B7Ti%2FihtkRF5BIWcPIEluJP6tlbFM%2FBf44LfZka%2FiemtahZAZzcO9TnI5uaXh3%2B%2BlrqtNonCwp6%2F245UFWkiW1elpgtVAmJPbogcAv6rSlokztAfWk296ZJXzRDYAtzGH0gq7CgSJKfH%2BXxaCmR0WcvlKjNQnp12%2FeKXJYO4tDap8UCBLuyxDnR7oJKLHQHJLP0r0EAVOOSIbrFang%2F1WOq%2BJaq4Efc4XpnTgnwlBbWTmhWDR1pvS9iVEzcSYLHT%2FfNnMRxFc7u%2Bj3qI%2F%2F5yuGuu14KR0MuQKKCSpViieD%2BfIti46sxPTsjSemoUKp0oXA%3D%3D
```

For the second set of example parameters (some non-ASCII characters):

```console
qJtv66wyp%2F1mZE%2BmIFAAMUoTe8xkmLN7%2FeAZjuC9x1ocxovItHLl%2FsNK7Wq8QjgiHqGn0bb8P7yVvGBEd1gFe71NQ8aM0M%2BJNIMz5UFxfeA53rXjFlvsyH1Sig%2BOuO9Nz5nhCaJ6bEfj2iuv7w27pB3L8MVqmoCi6D9C%2FQMiLxtPaR70CxtnvoOlIgPmpv2bQy029A31NEK19ieVLkoyp1EUkXRaX3v0mohx8yMnUG1dhX9nUg3Oy8TYZ03DQy7kHDGkMKisNX7rt%2FGuGx1HIgjFclDGLsbAFIodvSLjm9FbseasMELoxlAJDlwRnW8zo5sQmL0Fz7ao935QBynrng%3D%3D
```

**Step 3: Add signature to the request**

Complete the request by adding the `signature` parameter to the query string.

For the first set of example parameters (ASCII only):

```console
curl -H "X-MBX-APIKEY: CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ" -X POST 'https://testnet.binance.vision/api/v3/order?symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000&signature=HZ8HOjiJ1s%2FigS9JA%2Bn7%2B7Ti%2FihtkRF5BIWcPIEluJP6tlbFM%2FBf44LfZka%2FiemtahZAZzcO9TnI5uaXh3%2B%2BlrqtNonCwp6%2F245UFWkiW1elpgtVAmJPbogcAv6rSlokztAfWk296ZJXzRDYAtzGH0gq7CgSJKfH%2BXxaCmR0WcvlKjNQnp12%2FeKXJYO4tDap8UCBLuyxDnR7oJKLHQHJLP0r0EAVOOSIbrFang%2F1WOq%2BJaq4Efc4XpnTgnwlBbWTmhWDR1pvS9iVEzcSYLHT%2FfNnMRxFc7u%2Bj3qI%2F%2F5yuGuu14KR0MuQKKCSpViieD%2BfIti46sxPTsjSemoUKp0oXA%3D%3D'
```

For the second set of example parameters (some non-ASCII characters):

```console
curl -H "X-MBX-APIKEY: CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ" -X POST 'https://testnet.binance.vision/api/v3/order?symbol=%EF%BC%91%EF%BC%92%EF%BC%93%EF%BC%94%EF%BC%95%EF%BC%96&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000&signature=qJtv66wyp%2F1mZE%2BmIFAAMUoTe8xkmLN7%2FeAZjuC9x1ocxovItHLl%2FsNK7Wq8QjgiHqGn0bb8P7yVvGBEd1gFe71NQ8aM0M%2BJNIMz5UFxfeA53rXjFlvsyH1Sig%2BOuO9Nz5nhCaJ6bEfj2iuv7w27pB3L8MVqmoCi6D9C%2FQMiLxtPaR70CxtnvoOlIgPmpv2bQy029A31NEK19ieVLkoyp1EUkXRaX3v0mohx8yMnUG1dhX9nUg3Oy8TYZ03DQy7kHDGkMKisNX7rt%2FGuGx1HIgjFclDGLsbAFIodvSLjm9FbseasMELoxlAJDlwRnW8zo5sQmL0Fz7ao935QBynrng%3D%3D'
```

Here is a sample Bash script performing all the steps above:

```bash
function rawurlencode {
  local string="${1}"
  local strlen=${#string}
  local encoded=""
  local pos c o

  for (( pos=0 ; pos<strlen ; pos++ )); do
     c=${string:$pos:1}
     case "$c" in
        [-_.~a-zA-Z0-9] ) o="${c}" ;;
        * )               printf -v o '%%%02x' "'$c"
     esac
     encoded+="${o}"
  done
  echo "${encoded}"
}

API_KEY="put your own API Key here"
PRIVATE_KEY_PATH="test-prv-key.pem"
# Set up the request:
API_METHOD="POST"
API_CALL="api/v3/order"
API_PARAMS="symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2"
# Sign the request:
timestamp=$(date +%s000)
api_params_with_timestamp="$API_PARAMS&timestamp=$timestamp"

rawSignature=$(echo -n $api_params_with_timestamp | openssl dgst -keyform PEM -sha256 -sign $PRIVATE_KEY_PATH | openssl enc -base64 | tr -d '\n')

# Percent-encode the signature
signature=$(rawurlencode "$rawSignature")

# Send the request:
curl -H "X-MBX-APIKEY: $API_KEY" -X "$API_METHOD" \
    "https://testnet.binance.vision/$API_CALL?$api_params_with_timestamp" \
    --data-urlencode "signature=$signature"
```

#### Ed25519 Keys

**Note: It is highly recommended to use Ed25519 API keys as it should provide the best performance and security out of all supported key types.**

The signature payload of your request is the query string concatenated without separator to the HTTP body. Any non-ASCII character must be percent-encoded before signing.

There is one example with a symbol name comprised entirely of ASCII characters and one example with a symbol name containing non-ASCII characters.

These examples assume the private key is stored in the file `./test-prv-key.pem`.

Key | Value
------------ | ------------
`apiKey` | 4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO

Example of request with a symbol name comprised entirely of ASCII characters.

Parameter     | Value
------------  | ------------
`symbol`      | BTCUSDT
`side`        | SELL
`type`        | LIMIT
`timeInForce` | GTC
`quantity`    | 1
`price`       | 0.2
`timestamp`   | 1668481559918
`recvWindow`  | 5000

Example of a request with a symbol name containing non-ASCII characters.

Parameter     | Value
------------  | ------------
`symbol`      | １２３４５６
`side`        | SELL
`type`        | LIMIT
`timeInForce` | GTC
`quantity`    | 1
`price`       | 0.2
`timestamp`   | 1668481559918
`recvWindow`  | 5000


**Step 1: Construct the signature payload**

1. Format parameters as `parameter=value` pairs separated by `&`.
2. Percent-encode the string.

For the first set of example parameters (ASCII only), the `parameter=value` string should look like this:

```console
symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000
```

After percent-encoding, the signature payload should look like this:

```console
symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000
```

For the second set of example parameters (some non-ASCII characters), the `parameter=value` string should look like this:

```console
symbol=１２３４５６&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000
```

After percent-encoding, the signature payload should look like this:

```console
symbol=%EF%BC%91%EF%BC%92%EF%BC%93%EF%BC%94%EF%BC%95%EF%BC%96&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000
```

**Step 2: Compute the signature**

1. Sign the payload.
2. Encode the output as a base64 string.

Note that the payload and the resulting `signature` are **case-sensitive**.

For the first set of example parameters (ASCII only):

```console
echo -n "symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000" | openssl dgst -keyform PEM -sha256 -sign ./test-prv-key.pem | openssl enc -base64 | tr -d '\n'

HaZnek7KOGa/k5+f6Q1nw8lzMUpo36mRVvvLHCMUCXxlmdQQGZge1luAUKnleD/DYeD19YrqzeHbb6xU3MkSIXKhAO1MaYq48uGVYb3vJScEZVOutgMInrZzUcCWNulNkfcbmExSiymCZ5xQBw5QDuzpuDFqRZ1Xt+BZxEHBN9OYQKpoe0+ovjnXyVOaH8VUKhE/ghUWnThrXJr+hmSc5t7ggjiVPQc7pGn3qSNGCQwdpkQC9GHMr/r+8n6qeEKMYB5j/1wC4d8Jae8FQiU8xcXR0NlUgV2LAw61/ZJv5BTJpa+z5Lv1W9v6jHQWRX2O8uaG3KU/lR3spR7+oGlWOw=
```

For the second set of example parameters (some non-ASCII characters):

```console
echo -n "symbol=%EF%BC%91%EF%BC%92%EF%BC%93%EF%BC%94%EF%BC%95%EF%BC%96&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000" | openssl dgst -keyform PEM -sha256 -sign ./test-prv-key.pem | openssl enc -base64 | tr -d '\n'

qJtv66wyp/1mZE+mIFAAMUoTe8xkmLN7/eAZjuC9x1ocxovItHLl/sNK7Wq8QjgiHqGn0bb8P7yVvGBEd1gFe71NQ8aM0M+JNIMz5UFxfeA53rXjFlvsyH1Sig+OuO9Nz5nhCaJ6bEfj2iuv7w27pB3L8MVqmoCi6D9C/QMiLxtPaR70CxtnvoOlIgPmpv2bQy029A31NEK19ieVLkoyp1EUkXRaX3v0mohx8yMnUG1dhX9nUg3Oy8TYZ03DQy7kHDGkMKisNX7rt/GuGx1HIgjFclDGLsbAFIodvSLjm9FbseasMELoxlAJDlwRnW8zo5sQmL0Fz7ao935QBynrng==
```

3. Percent-encode the base64 string.

For the first set of example parameters (ASCII only):

```console
HaZnek7KOGa%2Fk5%2Bf6Q1nw8lzMUpo36mRVvvLHCMUCXxlmdQQGZge1luAUKnleD%2FDYeD19YrqzeHbb6xU3MkSIXKhAO1MaYq48uGVYb3vJScEZVOutgMInrZzUcCWNulNkfcbmExSiymCZ5xQBw5QDuzpuDFqRZ1Xt%2BBZxEHBN9OYQKpoe0%2BovjnXyVOaH8VUKhE%2FghUWnThrXJr%2BhmSc5t7ggjiVPQc7pGn3qSNGCQwdpkQC9GHMr%2Fr%2B8n6qeEKMYB5j%2F1wC4d8Jae8FQiU8xcXR0NlUgV2LAw61%2FZJv5BTJpa%2Bz5Lv1W9v6jHQWRX2O8uaG3KU%2FlR3spR7%2BoGlWOw%3D
```

For the second set of example parameters (some non-ASCII characters):

```console
qJtv66wyp%2F1mZE%2BmIFAAMUoTe8xkmLN7%2FeAZjuC9x1ocxovItHLl%2FsNK7Wq8QjgiHqGn0bb8P7yVvGBEd1gFe71NQ8aM0M%2BJNIMz5UFxfeA53rXjFlvsyH1Sig%2BOuO9Nz5nhCaJ6bEfj2iuv7w27pB3L8MVqmoCi6D9C%2FQMiLxtPaR70CxtnvoOlIgPmpv2bQy029A31NEK19ieVLkoyp1EUkXRaX3v0mohx8yMnUG1dhX9nUg3Oy8TYZ03DQy7kHDGkMKisNX7rt%2FGuGx1HIgjFclDGLsbAFIodvSLjm9FbseasMELoxlAJDlwRnW8zo5sQmL0Fz7ao935QBynrng%3D%3D
```

**Step 3: Add signature to the request**

Complete the request by adding the `signature` parameter to the query string.

For the first set of example parameters (ASCII only):

```console
curl -H "X-MBX-APIKEY: 4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO" -X POST 'hhttps://testnet.binance.vision/api/v3/order?symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000&signature=HaZnek7KOGa%2Fk5%2Bf6Q1nw8lzMUpo36mRVvvLHCMUCXxlmdQQGZge1luAUKnleD%2FDYeD19YrqzeHbb6xU3MkSIXKhAO1MaYq48uGVYb3vJScEZVOutgMInrZzUcCWNulNkfcbmExSiymCZ5xQBw5QDuzpuDFqRZ1Xt%2BBZxEHBN9OYQKpoe0%2BovjnXyVOaH8VUKhE%2FghUWnThrXJr%2BhmSc5t7ggjiVPQc7pGn3qSNGCQwdpkQC9GHMr%2Fr%2B8n6qeEKMYB5j%2F1wC4d8Jae8FQiU8xcXR0NlUgV2LAw61%2FZJv5BTJpa%2Bz5Lv1W9v6jHQWRX2O8uaG3KU%2FlR3spR7%2BoGlWOw%3D'
```

For the second set of example parameters (some non-ASCII characters):

```console
curl -H "X-MBX-APIKEY: 4yNzx3yWC5bS6YTwEkSRaC0nRmSQIIStAUOh1b6kqaBrTLIhjCpI5lJH8q8R8WNO" -X POST 'https://testnet.binance.vision/api/v3/order?symbol=%EF%BC%91%EF%BC%92%EF%BC%93%EF%BC%94%EF%BC%95%EF%BC%96&&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2&timestamp=1668481559918&recvWindow=5000&signature=qJtv66wyp%2F1mZE%2BmIFAAMUoTe8xkmLN7%2FeAZjuC9x1ocxovItHLl%2FsNK7Wq8QjgiHqGn0bb8P7yVvGBEd1gFe71NQ8aM0M%2BJNIMz5UFxfeA53rXjFlvsyH1Sig%2BOuO9Nz5nhCaJ6bEfj2iuv7w27pB3L8MVqmoCi6D9C%2FQMiLxtPaR70CxtnvoOlIgPmpv2bQy029A31NEK19ieVLkoyp1EUkXRaX3v0mohx8yMnUG1dhX9nUg3Oy8TYZ03DQy7kHDGkMKisNX7rt%2FGuGx1HIgjFclDGLsbAFIodvSLjm9FbseasMELoxlAJDlwRnW8zo5sQmL0Fz7ao935QBynrng%3D%3D'
```

Here is a sample Python script performing all the steps above:

```python
#!/usr/bin/env python3

import base64
import requests
import time
import urllib.parse
from cryptography.hazmat.primitives.serialization import load_pem_private_key

# Set up authentication
API_KEY='put your own API Key here'
PRIVATE_KEY_PATH='test-prv-key.pem'

# Load the private key.
# In this example the key is expected to be stored without encryption,
# but we recommend using a strong password for improved security.
with open(PRIVATE_KEY_PATH, 'rb') as f:
    private_key = load_pem_private_key(data=f.read(), password=None)

# Set up the request parameters
params = {
    'symbol':       'BTCUSDT',
    'side':         'SELL',
    'type':         'LIMIT',
    'timeInForce':  'GTC',
    'quantity':     '1.0000000',
    'price':        '0.20',
}

# Timestamp the request
timestamp = int(time.time() * 1000) # UNIX timestamp in milliseconds
params['timestamp'] = timestamp

# Sign the request
payload = urllib.parse.urlencode(params, encoding='UTF-8')
signature = base64.b64encode(private_key.sign(payload.encode('ASCII')))
params['signature'] = signature

# Send the request
headers = {
    'X-MBX-APIKEY': API_KEY,
}
response = requests.post(
    'https://testnet.binance.vision/api/v3/order',
    headers=headers,
    data=params,
)
print(response.json())
```

# Public API Endpoints

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

<a id="exchangeInfo"></a>

### Exchange information
```
GET /api/v3/exchangeInfo
```
Current exchange trading rules and symbol information

**Weight:**
20

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol |STRING| No| Example: curl -X GET "https://api.binance.com/api/v3/exchangeInfo?symbol=BNBBTC"
symbols |ARRAY OF STRING|No| Examples: curl -X GET "https://api.binance.com/api/v3/exchangeInfo?symbols=%5B%22BNBBTC%22,%22BTCUSDT%22%5D" <br/> or <br/> curl -g -X  GET 'https://api.binance.com/api/v3/exchangeInfo?symbols=["BTCUSDT","BNBBTC"]'
permissions |ENUM|No|Examples: curl -X GET "https://api.binance.com/api/v3/exchangeInfo?permissions=SPOT" <br/> or <br/> curl -X GET "https://api.binance.com/api/v3/exchangeInfo?permissions=%5B%22MARGIN%22%2C%22LEVERAGED%22%5D" <br/> or <br/> curl -g -X GET 'https://api.binance.com/api/v3/exchangeInfo?permissions=["MARGIN","LEVERAGED"]' |
showPermissionSets|BOOLEAN|No|Controls whether the content of the `permissionSets` field is populated or not. Defaults to `true`
symbolStatus|ENUM|No|Filters for symbols that have this `tradingStatus`. <br> Valid values: `TRADING`, `HALT`, `BREAK` <br> Cannot be used in combination with `symbols` or `symbol`.|

**Notes:**
* If the value provided to `symbol` or `symbols` do not exist, the endpoint will throw an error saying the symbol is invalid.
* All parameters are optional.
* `permissions` can support single or multiple values (e.g. `SPOT`, `["MARGIN","LEVERAGED"]`). This cannot be used in combination with `symbol` or `symbols`.
* If `permissions` parameter not provided, all symbols that have either `SPOT`, `MARGIN`, or `LEVERAGED` permission will be exposed.
  * To display symbols with any permission you need to specify them explicitly in `permissions`: (e.g. `["SPOT","MARGIN",...]`.). See [Account and Symbol Permissions](enums.md#account-and-symbol-permissions) for the full list.

<a id="examples-of-symbol-permissions-interpretation-from-the-response"></a>

**Examples of Symbol Permissions Interpretation from the Response:**

* `[["A","B"]]` means you may place an order if your account has either permission "A" **or** permission "B".
* `[["A"],["B"]]` means you can place an order if your account has permission "A" **and** permission "B".
* `[["A"],["B","C"]]` means you can place an order if your account has permission "A" **and** permission "B" or permission "C". (Inclusive or is applied here, not exclusive or, so your account may have both permission "B" and permission "C".)

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
      "otoAllowed": true,
      "opoAllowed": true,
      "quoteOrderQtyMarketAllowed": true,
      "allowTrailingStop": false,
      "cancelReplaceAllowed":false,
      "amendAllowed":false,
      "pegInstructionsAllowed": true,
      "isSpotTradingAllowed": true,
      "isMarginTradingAllowed": true,
      "filters": [
        // These are defined in the Filters section.
        // All filters are optional
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
  // Optional field. Present only when SOR is available.
  // https://github.com/binance/binance-spot-api-docs/blob/master/faqs/sor_faq.md
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

## Market Data endpoints
### Order book
```
GET /api/v3/depth
```

**Weight:**
Adjusted based on the limit:

|Limit|Request Weight
------|-------
1-100|  5
101-500| 25
501-1000| 50
1001-5000| 250

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default: 100; Maximum: 5000. <br/> If limit > 5000, only 5000 entries will be returned.
symbolStatus|ENUM|NO|Filters for symbols that have this `tradingStatus`. <br/>A status mismatch returns error `-1220 SYMBOL_DOES_NOT_MATCH_STATUS`.<br/> Valid values: `TRADING`, `HALT`, `BREAK`

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
25

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default: 500; Maximum: 1000.

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

### Old trade lookup
```
GET /api/v3/historicalTrades
```
Get older trades.

**Weight:**
25

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default: 500; Maximum: 1000.
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
4

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
fromId | LONG | NO | ID to get aggregate trades from INCLUSIVE.
startTime | LONG | NO | Timestamp in ms to get aggregate trades from INCLUSIVE.
endTime | LONG | NO | Timestamp in ms to get aggregate trades until INCLUSIVE.
limit | INT | NO | Default: 500; Maximum: 1000.

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
<a id="klines"></a>
### Kline/Candlestick data
```
GET /api/v3/klines
```
Kline/candlestick bars for a symbol.
Klines are uniquely identified by their open time.

**Weight:**
2

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
interval | ENUM | YES |
startTime | LONG | NO |
endTime | LONG | NO |
timeZone |STRING| NO| Default: 0 (UTC)
limit | INT | NO | Default: 500; Maximum: 1000.

<a id="kline-intervals"></a>
Supported kline intervals (case-sensitive):

Interval  | `interval` value
--------- | ----------------
seconds   | `1s`
minutes   | `1m`, `3m`, `5m`, `15m`, `30m`
hours     | `1h`, `2h`, `4h`, `6h`, `8h`, `12h`
days      | `1d`, `3d`
weeks     | `1w`
months    | `1M`

**Notes:**

* If `startTime` and `endTime` are not sent, the most recent klines are returned.
* Supported values for `timeZone`:
  * Hours and minutes (e.g. `-1:00`, `05:45`)
  * Only hours (e.g. `0`, `8`, `4`)
  * Accepted range is strictly [-12:00 to +14:00] inclusive
* If `timeZone` provided, kline intervals are interpreted in that timezone instead of UTC.
* Note that `startTime` and `endTime` are always interpreted in UTC, regardless of `timeZone`.

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
<a id="uiKlines"></a>
### UIKlines
```
GET /api/v3/uiKlines
```
The request is similar to klines having the same parameters and response.

`uiKlines` return modified kline data, optimized for presentation of candlestick charts.

**Weight:**
2

**Parameters:**

Name      | Type   | Mandatory    | Description
------    | ------ | ------------ | ------------
symbol    | STRING | YES          |
interval  | ENUM   | YES          |See [`klines`](#kline-intervals)
startTime | LONG   | NO           |
endTime   | LONG   | NO           |
timeZone  |STRING  | NO           | Default: 0 (UTC)
limit     | INT    | NO           | Default: 500; Maximum: 1000.

* If `startTime` and `endTime` are not sent, the most recent klines are returned.
* Supported values for `timeZone`:
  * Hours and minutes (e.g. `-1:00`, `05:45`)
  * Only hours (e.g. `0`, `8`, `4`)
  * Accepted range is strictly [-12:00 to +14:00] inclusive
* If `timeZone` provided, kline intervals are interpreted in that timezone instead of UTC.
* Note that `startTime` and `endTime` are always interpreted in UTC, regardless of `timeZone`.

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
```
GET /api/v3/avgPrice
```
Current average price for a symbol.

**Weight:**
2

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |

**Data Source:**
Memory

**Response:**
```javascript
{
  "mins": 5,                    // Average price interval (in minutes)
  "price": "9.35751834",        // Average price
  "closeTime": 1694061154503    // Last trade time
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
        <td>2</td>
    </tr>
    <tr>
        <td>symbol parameter is omitted</td>
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
        <td>101 or more</td>
        <td>80</td>
    </tr>
    <tr>
        <td>symbols parameter is omitted</td>
        <td>80</td>
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
     <tr>
        <td>symbolStatus</td>
        <td>ENUM</td>
        <td>NO</td>
        <td>Filters for symbols that have this <code>tradingStatus</code>.<br>For a single symbol, a status mismatch returns error <code>-1220 SYMBOL_DOES_NOT_MATCH_STATUS</code>. <br>For multiple or all symbols, non-matching ones are simply excluded from the response.<br>Valid values: <code>TRADING</code>, <code>HALT</code>, <code>BREAK</code> </td>
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

**Response - MINI:**

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


### Trading Day Ticker

```
GET /api/v3/ticker/tradingDay
```
Price change statistics for a trading day.

**Weight:**

4 for each requested <tt>symbol</tt>. <br/><br/> The weight for this request will cap at 200 once the number of `symbols` in the request is more than 50.

**Parameters:**

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
     <td>timeZone</td>
     <td>STRING</td>
     <td>NO</td>
     <td>Default: 0 (UTC)</td>
  </tr>
  <tr>
      <td>type</td>
      <td>ENUM</td>
      <td>NO</td>
      <td>Supported values: <tt>FULL</tt> or <tt>MINI</tt>. <br/>If none provided, the default is <tt>FULL</tt> </td>
  </tr>
  <tr>
      <td>symbolStatus</td>
      <td>ENUM</td>
      <td>NO</td>
      <td>Filters for symbols that have this <code>tradingStatus</code>.<br>For a single symbol, a status mismatch returns error <code>-1220 SYMBOL_DOES_NOT_MATCH_STATUS</code>. <br>For multiple symbols, non-matching ones are simply excluded from the response.<br>Valid values: <code>TRADING</code>, <code>HALT</code>, <code>BREAK</code> </td>
  </tr>
</table>

**Notes:**

* Supported values for `timeZone`:
  * Hours and minutes (e.g. `-1:00`, `05:45`)
  * Only hours (e.g. `0`, `8`, `4`)

**Data Source:**
Database

**Response - FULL:**

With `symbol`:

```javascript
{
  "symbol":             "BTCUSDT",
  "priceChange":        "-83.13000000",         // Absolute price change
  "priceChangePercent": "-0.317",               // Relative price change in percent
  "weightedAvgPrice":   "26234.58803036",       // quoteVolume / volume
  "openPrice":          "26304.80000000",
  "highPrice":          "26397.46000000",
  "lowPrice":           "26088.34000000",
  "lastPrice":          "26221.67000000",
  "volume":             "18495.35066000",       // Volume in base asset
  "quoteVolume":        "485217905.04210480",   // Volume in quote asset
  "openTime":           1695686400000,
  "closeTime":          1695772799999,
  "firstId":            3220151555,             // Trade ID of the first trade in the interval
  "lastId":             3220849281,             // Trade ID of the last trade in the interval
  "count":              697727                  // Number of trades in the interval
}

```

With `symbols`:

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

**Response - MINI:**

With `symbol`:

```javascript
{
  "symbol":         "BTCUSDT",
  "openPrice":      "26304.80000000",
  "highPrice":      "26397.46000000",
  "lowPrice":       "26088.34000000",
  "lastPrice":      "26221.67000000",
  "volume":         "18495.35066000",       // Volume in base asset
  "quoteVolume":    "485217905.04210480",   // Volume in quote asset
  "openTime":       1695686400000,
  "closeTime":      1695772799999,
  "firstId":        3220151555,             // Trade ID of the first trade in the interval
  "lastId":         3220849281,             // Trade ID of the last trade in the interval
  "count":          697727                  // Number of trades in the interval
}
```

With `symbols`:

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
        <td>2</td>
    </tr>
    <tr>
        <td>symbol parameter is omitted</td>
        <td>4</td>
    </tr>
    <tr>
        <td>symbols</td>
        <td>Any</td>
        <td>4</td>
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
    <tr>
        <td>symbolStatus</td>
        <td>ENUM</td>
        <td>NO</td>
        <td>Filters for symbols that have this <code>tradingStatus</code>.<br>For a single symbol, a status mismatch returns error <code>-1220 SYMBOL_DOES_NOT_MATCH_STATUS</code>. <br>For multiple or all symbols, non-matching ones are simply excluded from the response.<br>Valid values: <code>TRADING</code>, <code>HALT</code>, <code>BREAK</code> </td>
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
        <td>2</td>
    </tr>
    <tr>
        <td>symbol parameter is omitted</td>
        <td>4</td>
    </tr>
    <tr>
        <td>symbols</td>
        <td>Any</td>
        <td>4</td>
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
    <tr>
        <td>symbolStatus</td>
        <td>ENUM</td>
        <td>NO</td>
        <td>Filters for symbols that have this <code>tradingStatus</code>.<br>For a single symbol, a status mismatch returns error <code>-1220 SYMBOL_DOES_NOT_MATCH_STATUS</code>. <br>For multiple or all symbols, non-matching ones are simply excluded from the response.<br>Valid values: <code>TRADING</code>, <code>HALT</code>, <code>BREAK</code> </td>
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

4 for each requested <tt>symbol</tt> regardless of <tt>windowSize</tt>. <br/><br/> The weight for this request will cap at 200 once the number of `symbols` in the request is more than 50.

**Parameters:**

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
  <tr>
      <td>symbolStatus</td>
      <td>ENUM</td>
      <td>NO</td>
      <td>Filters for symbols that have this <code>tradingStatus</code>.<br>For a single symbol, a status mismatch returns error <code>-1220 SYMBOL_DOES_NOT_MATCH_STATUS</code>.<br>For multiple symbols, non-matching ones are simply excluded from the response.<br>Valid values: <code>TRADING</code>, <code>HALT</code>, <code>BREAK</code></td>
  </tr>
</table>

**Data Source:**
Database

**Response - FULL:**

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

**Response - MINI:**

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

## Trading endpoints

### New order (TRADE)
```
POST /api/v3/order
```
Send in a new order.

This adds 1 order to the `EXCHANGE_MAX_ORDERS` filter and the `MAX_NUM_ORDERS` filter.

**Weight:**
1

**Unfilled Order Count:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
side | ENUM | YES |Please see [Enums](enums.md#side) for supported values.
type | ENUM | YES |Please see [Enums](enums.md#ordertypes) for supported values.
timeInForce | ENUM | NO |Please see [Enums](enums.md#timeinforce) for supported values.
quantity | DECIMAL | NO |
quoteOrderQty|DECIMAL|NO|
price | DECIMAL | NO |
newClientOrderId | STRING | NO | A unique id among open orders. Automatically generated if not sent.<br/> Orders with the same `newClientOrderID` can be accepted only when the previous one is filled, otherwise the order will be rejected.
strategyId |LONG| NO|
strategyType |INT| NO| The value cannot be less than `1000000`.
stopPrice | DECIMAL | NO | Used with `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders.
trailingDelta|LONG|NO| See [Trailing Stop order FAQ](../faqs/trailing-stop-faq.md).
icebergQty | DECIMAL | NO | Used with `LIMIT`, `STOP_LOSS_LIMIT`, and `TAKE_PROFIT_LIMIT` to create an iceberg order.
newOrderRespType | ENUM | NO | Set the response JSON. `ACK`, `RESULT`, or `FULL`; `MARKET` and `LIMIT` order types default to `FULL`, all other orders default to `ACK`.
selfTradePreventionMode |ENUM| NO | The allowed enums is dependent on what is configured on the symbol. The possible supported values are: [STP Modes](enums.md#stpmodes).
pegPriceType | ENUM | NO | `PRIMARY_PEG` or `MARKET_PEG`. <br> See [Pegged Orders Info](#pegged-orders-info)|
pegOffsetValue | INT | NO | Price level to peg the price to (max: 100). <br>See [Pegged Orders Info](#pegged-orders-info)  |
pegOffsetType | ENUM | NO | Only `PRICE_LEVEL` is supported. <br> See [Pegged Orders Info](#pegged-orders-info) |
recvWindow | DECIMAL | NO |The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp | LONG | YES |


<a id="order-type">Some additional</a> mandatory parameters based on order `type`:

Type | Additional mandatory parameters | Additional Information
------------ | ------------| ------
`LIMIT` | `timeInForce`, `quantity`, `price`|
`MARKET` | `quantity` or `quoteOrderQty`| `MARKET` orders using the `quantity` field specifies the amount of the `base asset` the user wants to buy or sell at the market price. <br/> E.g. MARKET order on BTCUSDT will specify how much BTC the user is buying or selling. <br/><br/> `MARKET` orders using `quoteOrderQty` specifies the amount the user wants to spend (when buying) or receive (when selling) the `quote` asset; the correct `quantity` will be determined based on the market liquidity and `quoteOrderQty`. <br/> E.g. Using the symbol BTCUSDT: <br/> `BUY` side, the order will buy as many BTC as `quoteOrderQty` USDT can. <br/> `SELL` side, the order will sell as much BTC needed to receive `quoteOrderQty` USDT.
`STOP_LOSS` | `quantity`, `stopPrice` or `trailingDelta`| This will execute a `MARKET` order when the conditions are met. (e.g. `stopPrice` is met or `trailingDelta` is activated)
`STOP_LOSS_LIMIT` | `timeInForce`, `quantity`,  `price`, `stopPrice` or `trailingDelta`
`TAKE_PROFIT` | `quantity`, `stopPrice` or `trailingDelta` | This will execute a `MARKET` order when the conditions are met. (e.g. `stopPrice` is met or `trailingDelta` is activated)
`TAKE_PROFIT_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice` or `trailingDelta` |
`LIMIT_MAKER` | `quantity`, `price`| This is a `LIMIT` order that will be rejected if the order immediately matches and trades as a taker. <br/> This is also known as a POST-ONLY order.

<a id="pegged-orders-info">Notes on using parameters for Pegged Orders:</a>

* These parameters are allowed for `LIMIT`, `LIMIT_MAKER`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT_LIMIT` orders.
* If `pegPriceType` is specified, `price` becomes optional. Otherwise, it is still mandatory.
* `pegPriceType=PRIMARY_PEG` means the primary peg, that is the best price on the same side of the order book as your order.
* `pegPriceType=MARKET_PEG` means the market peg, that is the best price on the opposite side of the order book from your order.
* Use `pegOffsetType` and `pegOffsetValue` to request a price level other than the best one. These parameters must be specified together.

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

**Response - ACK:**
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "orderListId": -1, // Unless it's part of an order list, value will be -1
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595
}
```

**Response - RESULT:**
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "orderListId": -1, // Unless it's part of an order list, value will be -1
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595,
  "price": "0.00000000",
  "origQty": "10.00000000",
  "executedQty": "10.00000000",
  "origQuoteOrderQty": "0.000000",
  "cummulativeQuoteQty": "10.00000000",
  "status": "FILLED",
  "timeInForce": "GTC",
  "type": "MARKET",
  "side": "SELL",
  "workingTime": 1507725176595,
  "selfTradePreventionMode": "NONE"
}
```

**Response - FULL:**
```javascript
{
  "symbol": "BTCUSDT",
  "orderId": 28,
  "orderListId": -1, // Unless it's part of an order list, value will be -1
  "clientOrderId": "6gCrw2kRUAF9CvJDGP16IP",
  "transactTime": 1507725176595,
  "price": "0.00000000",
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
**Conditional fields in Order Responses**

There are fields in the order responses (e.g. order placement, order query, order cancellation) that appear only if certain conditions are met.

These fields can apply to order lists.

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
`usedSor`      | Field that determines whether order used SOR | Appears when placing orders using SOR|`"usedSor": true`
`workingFloor` | Field that determines whether the order is being filled by the SOR or by the order book the order was submitted to.|Appears when placing orders using SOR|`"workingFloor": "SOR"`|
`pegPriceType` |  Price peg type  |      Only for pegged orders  | `"pegPriceType": "PRIMARY_PEG"` |
`pegOffsetType` | Price peg offset type | Only for pegged orders, if requested  | `"pegOffsetType": "PRICE_LEVEL"` |
`pegOffsetValue` | Price peg offset value  | Only for pegged orders, if requested  | `"pegOffsetValue": 5` |
`peggedPrice` | Current price order is pegged at | Only for pegged orders, once determined | `"peggedPrice": "87523.83710000"` |

### Test new order (TRADE)
```
POST /api/v3/order/test
```
Test new order creation and signature/recvWindow long.
Creates and validates a new order but does not send it into the matching engine.

**Weight:**

|Condition| Request Weight|
|------------           | ------------ |
|Without `computeCommissionRates`| 1|
|With `computeCommissionRates`|20|

**Parameters:**

In addition to all parameters accepted by [`POST /api/v3/order`](#new-order-trade),
the following optional parameters are also accepted:

Name                   |Type          | Mandatory    | Description
------------           | ------------ | ------------ | ------------
computeCommissionRates | BOOLEAN      | NO           | Default: `false` <br> See [Commissions FAQ](../faqs/commission_faq.md#test-order-diferences) to learn more.

**Data Source:**
Memory

**Response:**

Without `computeCommissionRates`

```javascript
{}
```

With `computeCommissionRates`

```javascript
{
  "standardCommissionForOrder": {  //Standard commission rates on trades from the order.
    "maker": "0.00000112",
    "taker": "0.00000114"
  },
  "specialCommissionForOrder": {    //Special commission rates on trades from the order.
    "maker": "0.05000000",
    "taker": "0.06000000"
  },
  "taxCommissionForOrder": {       //Tax commission rates for trades from the order.
    "maker": "0.00000112",
    "taker": "0.00000114"
  },
  "discount": {                    //Discount on standard commissions when paying in BNB.
    "enabledForAccount": true,
    "enabledForSymbol": true,
    "discountAsset": "BNB",
    "discount": "0.25000000"       //Standard commission is reduced by this rate when paying commission in BNB.
  }
}
```

### Query order (USER_DATA)
```
GET /api/v3/order
```
Check an order's status.

**Weight:**
4

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
origClientOrderId | STRING | NO |
recvWindow | DECIMAL | NO | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp | LONG | YES |

**Notes:**
* Either `orderId` or `origClientOrderId` must be sent.
* If both `orderId` and `origClientOrderId` are provided, the `orderId` is searched first, then the `origClientOrderId` from that result is checked against that order. If both conditions are not met the request will be rejected.
* For some historical orders `cummulativeQuoteQty` will be < 0, meaning the data is not available at this time.

**Data Source:**
Memory => Database

**Response:**
```javascript
{
  "symbol": "LTCBTC",
  "orderId": 1,
  "orderListId": -1,                // This field will always have a value of -1 if not an order list.
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
DELETE /api/v3/order
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
recvWindow        | DECIMAL | NO           | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp         | LONG   | YES          |

Notes:
* Either `orderId` or `origClientOrderId` must be sent.
* If both `orderId` and `origClientOrderId` are provided, the `orderId` is searched first, then the `origClientOrderId` from that result is checked against that order. If both conditions are not met the request will be rejected.

**Data Source:**
Matching Engine

**Response:**
```javascript
{
  "symbol": "LTCBTC",
  "origClientOrderId": "myOrder1",
  "orderId": 4,
  "orderListId": -1, // Unless it's part of an order list, value will be -1
  "clientOrderId": "cancelMyOrder1",
  "transactTime": 1684804350068,
  "price": "2.00000000",
  "origQty": "1.00000000",
  "executedQty": "0.00000000",
  "origQuoteOrderQty": "0.000000",
  "cummulativeQuoteQty": "0.00000000",
  "status": "CANCELED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "selfTradePreventionMode": "NONE"
}
```

**Notes:**
* The payload above does not show all fields that can appear in the order response. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).
* The performance for canceling an order (single cancel or as part of a cancel-replace) is always better when only `orderId` is sent. Sending `origClientOrderId` or both `orderId` + `origClientOrderId` will be slower.

<a id="regarding-cancelrestrictions"></a>

**Regarding `cancelRestrictions`**

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
DELETE /api/v3/openOrders
```
Cancels all active orders on a symbol.
This includes orders that are part of an order list.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
recvWindow | DECIMAL | NO | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp | LONG | YES |

**Data Source:**
Matching Engine

**Response:**
```javascript
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
    "transactTime": 1684804350069,
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
        "transactTime": 1688005070874,
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
        "transactTime": 1688005070874,
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

### Cancel an Existing Order and Send a New Order (TRADE)

```
POST /api/v3/order/cancelReplace
```
Cancels an existing order and places a new order on the same symbol.

Filters and Order Count are evaluated before the processing of the cancellation and order placement occurs.

A new order that was not attempted (i.e. when `newOrderResult: NOT_ATTEMPTED`), will still increase the unfilled order count by 1.

**Weight:**
1

**Unfilled Order Count:**
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
cancelOrigClientOrderId|STRING| NO| Either `cancelOrderId` or `cancelOrigClientOrderId` must be sent. <br></br> If both `cancelOrderId` and `cancelOrigClientOrderId` parameters are provided, the `cancelOrderId` is searched first, then the `cancelOrigClientOrderId` from that result is checked against that order. <br></br> If both conditions are not met the request will be rejected.
cancelOrderId|LONG|NO| Either `cancelOrderId` or `cancelOrigClientOrderId` must be sent. <br></br>If both `cancelOrderId` and `cancelOrigClientOrderId` parameters are provided, the `cancelOrderId` is searched first, then the `cancelOrigClientOrderId` from that result is checked against that order. <br></br>If both conditions are not met the request will be rejected.
newClientOrderId |STRING|NO| Used to identify the new order.
strategyId |LONG| NO|
strategyType |INT| NO| The value cannot be less than `1000000`.
stopPrice|DECIMAL|NO|
trailingDelta|LONG|NO|See [Trailing Stop order FAQ](../faqs/trailing-stop-faq.md)
icebergQty|DECIMAL|NO|
newOrderRespType|ENUM|NO|Allowed values: <br/> `ACK`, `RESULT`, `FULL` <br/> `MARKET` and `LIMIT` orders types default to `FULL`; all other orders default to `ACK`
selfTradePreventionMode |ENUM| NO |The allowed enums is dependent on what is configured on the symbol. The possible supported values are: [STP Modes](./enums.md#stpmodes).
cancelRestrictions| ENUM   | NO           | Supported values: <br>`ONLY_NEW` - Cancel will succeed if the order status is `NEW`.<br> `ONLY_PARTIALLY_FILLED ` - Cancel will succeed if order status is `PARTIALLY_FILLED`. For more information please refer to [Regarding `cancelRestrictions`](#regarding-cancelrestrictions)
orderRateLimitExceededMode|ENUM|No| Supported values: <br> `DO_NOTHING` (default)- will only attempt to cancel the order if account has not exceeded the unfilled order rate limit<br> `CANCEL_ONLY` - will always cancel the order|
pegPriceType | ENUM | NO | `PRIMARY_PEG` or `MARKET_PEG` <br> See [Pegged Orders](#pegged-orders-info) |
pegOffsetValue | INT | NO | Price level to peg the price to (max: 100) <br> See [Pegged Orders](#pegged-orders-info)  |
pegOffsetType | ENUM | NO | Only `PRICE_LEVEL` is supported  <br> See [Pegged Orders](#pegged-orders-info)|
recvWindow | DECIMAL | NO | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp | LONG | YES |


Similar to `POST /api/v3/order`, additional mandatory parameters are determined by `type`.

Response format varies depending on whether the processing of the message succeeded, partially succeeded, or failed.

**Data Source:**
Matching Engine

<table>
<thead>
    <tr>
        <th colspan=3 align=left>Request</th>
        <th colspan=3 align=left>Response</th>
    </tr>
    <tr>
        <th><code>cancelReplaceMode</code></th>
        <th><code>orderRateLimitExceededMode</code></th>
        <th>Unfilled Order Count</th>
        <th><code>cancelResult</code></th>
        <th><code>newOrderResult</code></th>
        <th><code>status</code></th>
    </tr>
</thead>
<tbody>
    <tr>
        <td rowspan="11"><code>STOP_ON_FAILURE</code></td>
        <td rowspan="6"><code>DO_NOTHING</code></td>
        <td rowspan="3">Within Limits</td>
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
        <td rowspan="3">Exceeds Limits</td>
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
        <td rowspan="3">Within Limits</td>
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
        <td rowspan="2">Exceeds Limits</td>
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
        <td rowspan="4">Within Limits</td>
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
     <td rowspan="4">Exceeds Limits</td>
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
        <td rowspan="4">Within Limits</td>
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
        <td rowspan="4">Exceeds Limits</td>
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

**Response SUCCESS and account has not exceeded the unfilled order count:**

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
    "transactTime": 1684804350068,
    "price": "0.01000000",
    "origQty": "0.000100",
    "executedQty": "0.00000000",
    "origQuoteOrderQty": "0.000000",
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
    "origQuoteOrderQty": "0.000000",
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

**Response when Cancel Order Fails with STOP_ON FAILURE and account has not exceeded their unfilled order count:**
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

**Response when Cancel Order Succeeds but New Order Placement Fails and account has not exceeded their unfilled order count:**
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
      "transactTime": 1684804350068,
      "price": "0.006123",
      "origQty": "10000.000000",
      "executedQty": "0.000000",
      "origQuoteOrderQty": "0.000000",
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

**Response when Cancel Order fails with ALLOW_FAILURE and account has not exceeded their unfilled order count:**

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

**Response when both Cancel Order and New Order Placement fail using `cancelReplaceMode=ALLOW_FAILURE` and account has not exceeded their unfilled order count:**

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

**Response when using `orderRateLimitExceededMode=DO_NOTHING` and account's unfilled order count has been exceeded:**

```javascript
{
  "code": -1015,
  "msg": "Too many new orders; current limit is 1 orders per 10 SECOND."
}
```

**Response when using `orderRateLimitExceededMode=CANCEL_ONLY` and account's unfilled order count has been exceeded:**

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

**Notes:**
* The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).
* The performance for canceling an order (single cancel or as part of a cancel-replace) is always better when only `orderId` is sent. Sending `origClientOrderId` or both `orderId` + `origClientOrderId` will be slower.

### Order Amend Keep Priority (TRADE)

```
PUT /api/v3/order/amend/keepPriority
```

Reduce the quantity of an existing open order.

This adds 0 orders to the `EXCHANGE_MAX_ORDERS` filter and the `MAX_NUM_ORDERS` filter.

Read [Order Amend Keep Priority FAQ](../faqs/order_amend_keep_priority.md) to learn more.

**Weight**:
4

**Unfilled Order Count:**
0

**Parameters:**

Name | Type | Mandatory | Description |
:---- | :---- | :---- | :---- |
symbol | STRING | YES |  |
orderId | LONG | NO\* | `orderId` or `origClientOrderId` must be sent  |
origClientOrderId | STRING | NO\* | `orderId` or `origClientOrderId` must be sent  |
newClientOrderId | STRING | NO\* | The new client order ID for the order after being amended.  <br> If not sent, one will be randomly generated. <br> It is possible to reuse the current clientOrderId by sending it as the `newClientOrderId`. |
newQty | DECIMAL | YES | `newQty` must be greater than 0 and less than the order's quantity.|
recvWindow | DECIMAL | NO | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp | LONG | YES |


**Data Source**: Matching Engine

**Response:**
Response for a single order:

```json
{
  "transactTime": 1741926410255,
  "executionId": 75,
  "amendedOrder":
  {
    "symbol": "BTCUSDT",
    "orderId": 33,
    "orderListId": -1,
    "origClientOrderId": "5xrgbMyg6z36NzBn2pbT8H",
    "clientOrderId": "PFaq6hIHxqFENGfdtn4J6Q",
    "price": "6.00000000",
    "qty": "5.00000000",
    "executedQty": "0.00000000",
    "preventedQty": "0.00000000",
    "quoteOrderQty": "0.00000000",
    "cumulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL",
    "workingTime": 1741926410242,
    "selfTradePreventionMode": "NONE"
  }
}
```

Response for an order that is part of an Order list:

```json
{
  "transactTime": 1741669661670,
  "executionId": 22,
  "amendedOrder":
  {
    "symbol": "BTCUSDT",
    "orderId": 9,
    "orderListId": 1,
    "origClientOrderId": "W0fJ9fiLKHOJutovPK3oJp",
    "clientOrderId": "UQ1Np3bmQ71jJzsSDW9Vpi",
    "price": "0.00000000",
    "qty": "4.00000000",
    "executedQty": "0.00000000",
    "preventedQty": "0.00000000",
    "quoteOrderQty": "0.00000000",
    "cumulativeQuoteQty": "0.00000000",
    "status": "PENDING_NEW",
    "timeInForce": "GTC",
    "type": "MARKET",
    "side": "BUY",
    "selfTradePreventionMode": "NONE"
  },
  "listStatus":
  {
    "orderListId": 1,
    "contingencyType": "OTO",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "AT7FTxZXylVSwRoZs52mt3",
    "symbol": "BTCUSDT",
    "orders":
    [
      {
        "symbol": "BTCUSDT",
        "orderId": 8,
        "clientOrderId": "GkwwHZUUbFtZOoH1YsZk9Q"
      },
      {
        "symbol": "BTCUSDT",
        "orderId": 9,
        "clientOrderId": "UQ1Np3bmQ71jJzsSDW9Vpi"
      }
    ]
  }
}
```

**Note:** The payloads above do not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

### Order lists

#### New Order list - OCO (TRADE)

```
POST /api/v3/orderList/oco
```

Send in an one-cancels-the-other (OCO) pair, where activation of one order immediately cancels the other.

* An OCO has 2 orders called the **above order** and **below order**.
* One of the orders must be a `LIMIT_MAKER/TAKE_PROFIT/TAKE_PROFIT_LIMIT` order and the other must be `STOP_LOSS` or `STOP_LOSS_LIMIT` order.
* Price restrictions
  * If the OCO is on the `SELL` side:
    * `LIMIT_MAKER/TAKE_PROFIT_LIMIT` `price` \> Last Traded Price \>  `STOP_LOSS/STOP_LOSS_LIMIT` `stopPrice`
    * `TAKE_PROFIT stopPrice `> Last Traded Price > `STOP_LOSS/STOP_LOSS_LIMIT stopPrice`
  * If the OCO is on the `BUY` side:
    * `LIMIT_MAKER/TAKE_PROFIT_LIMIT price` \< Last Traded Price \< `stopPrice`
    * `TAKE_PROFIT stopPrice` \< Last Traded Price \< `STOP_LOSS/STOP_LOSS_LIMIT stopPrice`
* OCOs add **2 orders** to the `EXCHANGE_MAX_ORDERS` filter and the `MAX_NUM_ORDERS` filter.

**Weight:**
1

**Unfilled Order Count:**
2

**Parameters:**

Name                   |Type    | Mandatory | Description
-----                  |------  | -----     |----
symbol                 |STRING  |Yes        |
listClientOrderId      |STRING  |No         |Arbitrary unique ID among open order lists. Automatically generated if not sent. <br> A new order list with the same `listClientOrderId` is accepted only when the previous one is filled or completely expired. <br> `listClientOrderId` is distinct from the `aboveClientOrderId` and the `belowCLientOrderId`.
side                   |ENUM    |Yes        |`BUY` or `SELL`
quantity               |DECIMAL |Yes       |Quantity for both orders of the order list.
aboveType              |ENUM   |Yes       |Supported values: `STOP_LOSS_LIMIT`, `STOP_LOSS`, `LIMIT_MAKER`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`
aboveClientOrderId     |STRING  |No         |Arbitrary unique ID among open orders for the above order. Automatically generated if not sent
aboveIcebergQty        |LONG    |No         |Note that this can only be used if `aboveTimeInForce` is `GTC`.
abovePrice             |DECIMAL |No         |Can be used if `aboveType` is `STOP_LOSS_LIMIT` , `LIMIT_MAKER`, or `TAKE_PROFIT_LIMIT` to specify the limit price.
aboveStopPrice         |DECIMAL |No         |Can be used if `aboveType` is `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`. <br>Either `aboveStopPrice` or `aboveTrailingDelta` or both, must be specified.
aboveTrailingDelta     |LONG    |No         |See [Trailing Stop order FAQ](../faqs/trailing-stop-faq.md).
aboveTimeInForce       |ENUM    |No         |Required if `aboveType` is `STOP_LOSS_LIMIT` or `TAKE_PROFIT_LIMIT`.
aboveStrategyId        |LONG     |No         |Arbitrary numeric value identifying the above order within an order strategy.
aboveStrategyType      |INT     |No         |Arbitrary numeric value identifying the above order strategy. <br>Values smaller than 1000000 are reserved and cannot be used.
abovePegPriceType      |ENUM | NO | See [Pegged Orders](#pegged-orders-info) |
abovePegOffsetType     |ENUM | NO |  |
abovePegOffsetValue    |INT | NO |  |
belowType              |ENUM    |Yes        |Supported values: `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`,`TAKE_PROFIT_LIMIT`
belowClientOrderId     |STRING  |No         |Arbitrary unique ID among open orders for the below order. Automatically generated if not sent
belowIcebergQty        |LONG    |No         |Note that this can only be used if `belowTimeInForce` is `GTC`.
belowPrice             |DECIMAL |No         |Can be used if `belowType` is `STOP_LOSS_LIMIT`, `TAKE_PROFIT_LIMIT`, or `LIMIT_MAKER` to specify the limit price.
belowStopPrice         |DECIMAL |No         |Can be used if `belowType` is `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT` or `TAKE_PROFIT_LIMIT`. Either `belowStopPrice` or `belowTrailingDelta` or both, must be specified. <br>Either `belowStopPrice` or `belowTrailingDelta` or both, must be specified.
belowTrailingDelta     |LONG    |No         |See [Trailing Stop order FAQ](../faqs/trailing-stop-faq.md).
belowTimeInForce       |ENUM    |No         |Required if `belowType` is `STOP_LOSS_LIMIT` or `TAKE_PROFIT_LIMIT`.
belowStrategyId        |LONG    |No          |Arbitrary numeric value identifying the below order within an order strategy.
belowStrategyType      |INT     |No         |Arbitrary numeric value identifying the below order strategy. <br>Values smaller than 1000000 are reserved and cannot be used.
belowPegPriceType      |ENUM | NO |  |See [Pegged Orders](#pegged-orders-info) |
belowPegOffsetType     |ENUM | NO |  |
belowPegOffsetValue    |INT | NO |  |
newOrderRespType       |ENUM    |No         |Select response format: `ACK`, `RESULT`, `FULL`
selfTradePreventionMode|ENUM    |No         |The allowed enums is dependent on what is configured on the symbol. Supported values: [STP Modes](./enums.md#stpmodes)
recvWindow             |DECIMAL  |No          |The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp              |LONG   |Yes          |

**Data Source:**
Matching Engine

**Response:**

Response format for `orderReports` is selected using the `newOrderRespType` parameter. The following example is for the `RESULT` response type. See [`POST /api/v3/order`](#new-order-trade) for more examples.

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

#### New Order list - OTO (TRADE)

```
POST /api/v3/orderList/oto
```

Place an OTO.

* An OTO (One-Triggers-the-Other) is an order list comprised of 2 orders.
* The first order is called the **working order** and must be `LIMIT` or `LIMIT_MAKER`. Initially, only the working order goes on the order book.
* The second order is called the **pending order**. It can be any order type except for `MARKET` orders using parameter `quoteOrderQty`. The pending order is only placed on the order book when the working order gets **fully filled**.
* If either the working order or the pending order is cancelled individually, the other order in the order list will also be canceled or expired.
* When the order list is placed, if the working order gets **immediately fully filled**, the placement response will show the working order as `FILLED` but the pending order will still appear as `PENDING_NEW`. You need to query the status of the pending order again to see its updated status.
* OTOs add **2 orders** to the `EXCHANGE_MAX_NUM_ORDERS` filter and `MAX_NUM_ORDERS` filter.

**Weight:** 1

**Unfilled Order Count:**
2

**Parameters:**

Name                   |Type   |Mandatory | Description
----                   |----   |------    |------
symbol                 |STRING |YES       |
listClientOrderId      |STRING |NO        |Arbitrary unique ID among open order lists. Automatically generated if not sent. <br>A new order list with the same listClientOrderId is accepted only when the previous one is filled or completely expired. <br> `listClientOrderId` is distinct from the `workingClientOrderId` and the `pendingClientOrderId`.
newOrderRespType       |ENUM   |NO        |Format of the JSON response. Supported values: [Order Response Type](./enums.md#orderresponsetype)
selfTradePreventionMode|ENUM   |NO        |The allowed values are dependent on what is configured on the symbol. Supported values: [STP Modes](./enums.md#stpmodes)
workingType            |ENUM   |YES       |Supported values: `LIMIT`,`LIMIT_MAKER`
workingSide            |ENUM   |YES       |Supported values: [Order Side](./enums.md#side)
workingClientOrderId   |STRING |NO        |Arbitrary unique ID among open orders for the working order.<br> Automatically generated if not sent.
workingPrice           |DECIMAL|YES       |
workingQuantity        |DECIMAL|YES       |Sets the quantity for the working order.
workingIcebergQty      |DECIMAL|NO       |This can only be used if `workingTimeInForce` is `GTC`, or if `workingType` is `LIMIT_MAKER`.
workingTimeInForce     |ENUM   |NO        |Supported values: [Time In Force](./enums.md#timeinforce)
workingStrategyId      |LONG   |NO        |Arbitrary numeric value identifying the working order within an order strategy.
workingStrategyType    |INT    |NO        |Arbitrary numeric value identifying the working order strategy. <br> Values smaller than 1000000 are reserved and cannot be used.
workingPegPriceType    |ENUM   | NO       | See [Pegged Orders](#pegged-order-info) |
workingPegOffsetType   |ENUM   | NO       |
workingPegOffsetValue  |INT | NO |  |
pendingType            |ENUM   |YES       |Supported values: [Order Types](#order-type)<br> Note that `MARKET` orders using `quoteOrderQty` are not supported.
pendingSide            |ENUM   |YES       |Supported values: [Order Side](./enums.md#side)
pendingClientOrderId   |STRING |NO        |Arbitrary unique ID among open orders for the pending order.<br> Automatically generated if not sent.
pendingPrice           |DECIMAL|NO        |
pendingStopPrice       |DECIMAL|NO        |
pendingTrailingDelta   |DECIMAL|NO        |
pendingQuantity        |DECIMAL|YES       |Sets the quantity for the pending order.
pendingIcebergQty      |DECIMAL|NO        |This can only be used if `pendingTimeInForce` is `GTC` or if `pendingType` is `LIMIT_MAKER`.
pendingTimeInForce     |ENUM   |NO        |Supported values: [Time In Force](./enums.md#timeinforce)
pendingStrategyId      |LONG   |NO        |Arbitrary numeric value identifying the pending order within an order strategy.
pendingStrategyType    |INT    |NO        |Arbitrary numeric value identifying the pending order strategy. <br> Values smaller than 1000000 are reserved and cannot be used.
pendingPegPriceType    |ENUM   |NO        |See [Pegged Orders](#pegged-order-info) |
pendingPegOffsetType   |ENUM | NO |  |
pendingPegOffsetValue  |INT | NO |  |
recvWindow             |DECIMAL  |NO        |The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp              |LONG   |YES       |

<a id="mandatory-parameters-based-on-pendingtype-or-workingtype"></a>

**Mandatory parameters based on `pendingType` or `workingType`**

Depending on the `pendingType` or `workingType`, some optional parameters will become mandatory.

|Type                                                  |Additional mandatory parameters|Additional information|
|----                                                  |----                           |------
|`workingType` = `LIMIT`                               |`workingTimeInForce`           |
|`pendingType` = `LIMIT`                                |`pendingPrice`, `pendingTimeInForce`          |
|`pendingType` = `STOP_LOSS` or `TAKE_PROFIT`           |`pendingStopPrice` and/or `pendingTrailingDelta`|
|`pendingType` = `STOP_LOSS_LIMIT` or `TAKE_PROFIT_LIMIT`|`pendingPrice`, `pendingStopPrice` and/or `pendingTrailingDelta`, `pendingTimeInForce`|

**Data Source:**

Matching Engine

**Response:**

```javascript
{
    "orderListId": 0,
    "contingencyType": "OTO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "yl2ERtcar1o25zcWtqVBTC",
    "transactionTime": 1712289389158,
    "symbol": "LTCBTC",
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

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

#### New Order list - OTOCO (TRADE)

```
POST /api/v3/orderList/otoco
```

Place an OTOCO.

* An OTOCO (One-Triggers-One-Cancels-the-Other) is an order list comprised of 3 orders.
* The first order is called the **working order** and must be `LIMIT` or `LIMIT_MAKER`. Initially, only the working order goes on the order book.
  * The behavior of the working order is the same as the [OTO](#new-order-list---oto-trade).
* OTOCO has 2 pending orders (pending above and pending below), forming an OCO pair. The pending orders are only placed on the order book when the working order gets **fully filled**.
    * The rules of the pending above and pending below follow the same rules as the [Order list OCO](#new-order-list---oco-trade).
* OTOCOs add **3 orders** to the `EXCHANGE_MAX_NUM_ORDERS` filter and `MAX_NUM_ORDERS` filter.

**Weight:** 1

**Unfilled Order Count:**
3

**Parameters:**

Name                     |Type   |Mandatory | Description
----                     |----   |------    |------
symbol                   |STRING |YES       |
listClientOrderId        |STRING |NO        |Arbitrary unique ID among open order lists. Automatically generated if not sent. <br>A new order list with the same listClientOrderId is accepted only when the previous one is filled or completely expired. <br> `listClientOrderId` is distinct from the `workingClientOrderId`, `pendingAboveClientOrderId`, and the `pendingBelowClientOrderId`.
newOrderRespType         |ENUM   |NO        |Format of the JSON response. Supported values: [Order Response Type](./enums.md#orderresponsetype)
selfTradePreventionMode  |ENUM   |NO        |The allowed values are dependent on what is configured on the symbol. Supported values: [STP Modes](./enums.md#stpmodes)
workingType              |ENUM   |YES       |Supported values: `LIMIT`, `LIMIT_MAKER`
workingSide              |ENUM   |YES       |Supported values: [Order side](./enums.md#side)
workingClientOrderId     |STRING |NO        |Arbitrary unique ID among open orders for the working order.<br> Automatically generated if not sent.
workingPrice             |DECIMAL|YES       |
workingQuantity          |DECIMAL|YES        |
workingIcebergQty        |DECIMAL|NO        |This can only be used if `workingTimeInForce` is `GTC` or if `workingType` is `LIMIT_MAKER`.|
workingTimeInForce       |ENUM   |NO        |Supported values: [Time In Force](./enums.md#timeinforce)
workingStrategyId        |LONG    |NO        |Arbitrary numeric value identifying the working order within an order strategy.
workingStrategyType      |INT    |NO        |Arbitrary numeric value identifying the working order strategy. <br> Values smaller than 1000000 are reserved and cannot be used.
workingPegPriceType      | ENUM   | NO      | See [Pegged Orders](#pegged-orders-info) |
workingPegOffsetType     | ENUM | NO |  |
workingPegOffsetValue    | INT | NO |  |
pendingSide              |ENUM   |YES       |Supported values: [Order side](./enums.md#side)
pendingQuantity          |DECIMAL|YES       |
pendingAboveType         |ENUM   |YES       |Supported values: `STOP_LOSS_LIMIT`, `STOP_LOSS`, `LIMIT_MAKER`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT`
pendingAboveClientOrderId|STRING |NO        |Arbitrary unique ID among open orders for the pending above order.<br> Automatically generated if not sent.
pendingAbovePrice        |DECIMAL|NO        |Can be used if `pendingAboveType` is `STOP_LOSS_LIMIT` , `LIMIT_MAKER`, or `TAKE_PROFIT_LIMIT` to specify the limit price.
pendingAboveStopPrice    |DECIMAL|NO        |Can be used if `pendingAboveType` is `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, or `TAKE_PROFIT_LIMIT`
pendingAboveTrailingDelta|DECIMAL|NO        |See [Trailing Stop FAQ](../faqs/trailing-stop-faq.md)
pendingAboveIcebergQty   |DECIMAL|NO        |This can only be used if `pendingAboveTimeInForce` is `GTC` or if `pendingAboveType` is `LIMIT_MAKER`.
pendingAboveTimeInForce  |ENUM   |NO        |
pendingAboveStrategyId   |LONG    |NO        |Arbitrary numeric value identifying the pending above order within an order strategy.
pendingAboveStrategyType |INT    |NO        |Arbitrary numeric value identifying the pending above order strategy. <br> Values smaller than 1000000 are reserved and cannot be used.
pendingAbovePegPriceType | ENUM | NO        |  See [Pegged Orders](#pegged-orders-info) |
pendingAbovePegOffsetType | ENUM | NO |  |
pendingAbovePegOffsetValue | INT | NO |  |
pendingBelowType         |ENUM   |NO        |Supported values: `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`,`TAKE_PROFIT_LIMIT`
pendingBelowClientOrderId|STRING |NO        |Arbitrary unique ID among open orders for the pending below order.<br> Automatically generated if not sent.
pendingBelowPrice        |DECIMAL|NO        |Can be used if `pendingBelowType` is `STOP_LOSS_LIMIT` or `TAKE_PROFIT_LIMIT` to specify limit price.
pendingBelowStopPrice    |DECIMAL|NO        |Can be used if `pendingBelowType` is `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, or `TAKE_PROFIT_LIMIT`. <br>Either `pendingBelowStopPrice` or `pendingBelowTrailingDelta` or both, must be specified.
pendingBelowTrailingDelta|DECIMAL|NO        |
pendingBelowIcebergQty   |DECIMAL|NO        |This can only be used if `pendingBelowTimeInForce` is `GTC` or if `pendingBelowType` is `LIMIT_MAKER`.
pendingBelowTimeInForce  |ENUM   |NO        |Supported values: [Time In Force](./enums.md#timeinforce)
pendingBelowStrategyId   |LONG   |NO        |Arbitrary numeric value identifying the pending below order within an order strategy.
pendingBelowStrategyType |INT    |NO        |Arbitrary numeric value identifying the pending below order strategy. <br> Values smaller than 1000000 are reserved and cannot be used.
pendingBelowPegPriceType | ENUM  |NO        |  See [Pegged Orders](#pegged-orders-info) |
pendingBelowPegOffsetType | ENUM | NO |  |
pendingBelowPegOffsetValue | INT | NO |  |
recvWindow               |DECIMAL |NO        |The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp                |LONG   |YES       |

<a id="mandatory-parameters-based-on-pendingabovetype-pendingbelowtype-or-workingtype"></a>

**Mandatory parameters based on `pendingAboveType`, `pendingBelowType` or `workingType`**

Depending on the `pendingAboveType`/`pendingBelowType` or `workingType`, some optional parameters will become mandatory.

|Type                                                       |Additional mandatory parameters|Additional information|
|----                                                       |----                           |------
|`workingType` = `LIMIT`                                    |`workingTimeInForce`           |
|`pendingAboveType`= `LIMIT_MAKER`                                |`pendingAbovePrice`     |
|`pendingAboveType` = `STOP_LOSS/TAKE_PROFIT`        |`pendingAboveStopPrice` and/or `pendingAboveTrailingDelta`|
|`pendingAboveType=STOP_LOSS_LIMIT/TAKE_PROFIT_LIMIT` |`pendingAbovePrice`, `pendingAboveStopPrice` and/or `pendingAboveTrailingDelta`, `pendingAboveTimeInForce`|
|`pendingBelowType`= `LIMIT_MAKER`                                |`pendingBelowPrice`          |
|`pendingBelowType= STOP_LOSS/TAKE_PROFIT`         |`pendingBelowStopPrice` and/or `pendingBelowTrailingDelta`|
|`pendingBelowType=STOP_LOSS_LIMIT/TAKE_PROFIT_LIMIT` |`pendingBelowPrice`, `pendingBelowStopPrice` and/or `pendingBelowTrailingDelta`, `pendingBelowTimeInForce`|

**Data Source:**

Matching Engine

**Response:**

```javascript
{
    "orderListId": 1,
    "contingencyType": "OTO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "RumwQpBaDctlUu5jyG5rs0",
    "transactionTime": 1712291372842,
    "symbol": "LTCBTC",
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

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

#### New Order List - OPO (TRADE)

```
POST /api/v3/orderList/opo
```

Place an [OPO](../faqs/opo.md).

* OPOs add 2 orders to the EXCHANGE_MAX_NUM_ORDERS filter and MAX_NUM_ORDERS filter.

**Weight:** 1

**Unfilled Order Count:** 2

**Parameters:**

| Name | Type | Mandatory | Description |
| ----- | ----- | ----- | ----- |
| symbol | STRING | YES |  |
| listClientOrderId | STRING | NO | Arbitrary unique ID among open order lists. Automatically generated if not sent. A new order list with the same listClientOrderId is accepted only when the previous one is filled or completely expired. `listClientOrderId` is distinct from the `workingClientOrderId` and the `pendingClientOrderId`. |
| newOrderRespType | ENUM | NO | Format of the JSON response. Supported values: [Order Response Type](./enums.md#orderresponsetype) |
| selfTradePreventionMode | ENUM | NO | The allowed values are dependent on what is configured on the symbol. Supported values: [STP Modes](./enums.md#stpmodes) |
| workingType | ENUM | YES | Supported values: `LIMIT`,`LIMIT_MAKER` |
| workingSide | ENUM | YES | Supported values: [Order Side](./enums.md#side) |
| workingClientOrderId | STRING | NO | Arbitrary unique ID among open orders for the working order. Automatically generated if not sent. |
| workingPrice | DECIMAL | YES |  |
| workingQuantity | DECIMAL | YES | Sets the quantity for the working order. |
| workingIcebergQty | DECIMAL | NO | This can only be used if `workingTimeInForce` is `GTC`, or if `workingType` is `LIMIT_MAKER`. |
| workingTimeInForce | ENUM | NO | Supported values: [Time In Force](./enums.md#timeinforce) |
| workingStrategyId | LONG | NO | Arbitrary numeric value identifying the working order within an order strategy. |
| workingStrategyType | INT | NO | Arbitrary numeric value identifying the working order strategy. Values smaller than 1000000 are reserved and cannot be used. |
| workingPegPriceType | ENUM | NO | See [Pegged Orders](#pegged-orders-info) |
| workingPegOffsetType | ENUM | NO |  |
| workingPegOffsetValue | INT | NO |  |
| pendingType | ENUM | YES | Supported values: [Order Types](#order-type) Note that `MARKET` orders using `quoteOrderQty` are not supported. |
| pendingSide | ENUM | YES | Supported values: [Order Side](./enums.md#side) |
| pendingClientOrderId | STRING | NO | Arbitrary unique ID among open orders for the pending order. Automatically generated if not sent. |
| pendingPrice | DECIMAL | NO |  |
| pendingStopPrice | DECIMAL | NO |  |
| pendingTrailingDelta | DECIMAL | NO |  |
| pendingIcebergQty | DECIMAL | NO | This can only be used if `pendingTimeInForce` is `GTC` or if `pendingType` is `LIMIT_MAKER`. |
| pendingTimeInForce | ENUM | NO | Supported values: [Time In Force](./enums.md#timeinforce) |
| pendingStrategyId | LONG | NO | Arbitrary numeric value identifying the pending order within an order strategy. |
| pendingStrategyType | INT | NO | Arbitrary numeric value identifying the pending order strategy. Values smaller than 1000000 are reserved and cannot be used. |
| pendingPegPriceType | ENUM | NO | See [Pegged Orders](#pegged-orders-info) |
| pendingPegOffsetType | ENUM | NO |  |
| pendingPegOffsetValue | INT | NO |  |
| recvWindow | DECIMAL | NO | The value cannot be greater than `60000`. Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified. |
| timestamp | LONG | YES |  |

**Data Source**: Matching Engine

**Response:**

```javascript
{
    "orderListId": 0,
    "contingencyType": "OTO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "H94qCqO27P74OEiO4X8HOG",
    "transactionTime": 1762998011671,
    "symbol": "BTCUSDT",
    "orders": [
        {
            "symbol": "BTCUSDT",
            "orderId": 2,
            "clientOrderId": "JX6xfdjo0wysiGumfHNmPu"
        },
        {
            "symbol": "BTCUSDT",
            "orderId": 3,
            "clientOrderId": "2ZJCY0IjOhuYIMLGN8kU8S"
        }
    ],
    "orderReports": [
        {
            "symbol": "BTCUSDT",
            "orderId": 2,
            "orderListId": 0,
            "clientOrderId": "JX6xfdjo0wysiGumfHNmPu",
            "transactTime": 1762998011671,
            "price": "102264.00000000",
            "origQty": "0.00060000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.00000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "NEW",
            "timeInForce": "GTC",
            "type": "LIMIT",
            "side": "BUY",
            "workingTime": 1762998011671,
            "selfTradePreventionMode": "NONE"
        },
        {
            "symbol": "BTCUSDT",
            "orderId": 3,
            "orderListId": 0,
            "clientOrderId": "2ZJCY0IjOhuYIMLGN8kU8S",
            "transactTime": 1762998011671,
            "price": "0.00000000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.00000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "PENDING_NEW",
            "timeInForce": "GTC",
            "type": "MARKET",
            "side": "SELL",
            "workingTime": -1,
            "selfTradePreventionMode": "NONE"
        }
    ]
}
```

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

#### New Order List - OPOCO (TRADE)

```
POST /api/v3/orderList/opoco
```

Place an [OPOCO](../faqs/opo.md).

**Weight**: 1

**Unfilled Order Count:** 3

**Parameters:**

| Name | Type | Mandatory | Description |
| ----- | ----- | ----- | ----- |
| symbol | STRING | YES |  |
| listClientOrderId | STRING | NO | Arbitrary unique ID among open order lists. Automatically generated if not sent. A new order list with the same listClientOrderId is accepted only when the previous one is filled or completely expired. `listClientOrderId` is distinct from the `workingClientOrderId`, `pendingAboveClientOrderId`, and the `pendingBelowClientOrderId`. |
| newOrderRespType | ENUM | NO | Format of the JSON response. Supported values: [Order Response Type](./enums.md#orderresponsetype) |
| selfTradePreventionMode | ENUM | NO | The allowed values are dependent on what is configured on the symbol. Supported values: [STP Modes](./enums.md#stpmodes) |
| workingType | ENUM | YES | Supported values: `LIMIT`, `LIMIT_MAKER` |
| workingSide | ENUM | YES | Supported values: [Order side](./enums.md#side) |
| workingClientOrderId | STRING | NO | Arbitrary unique ID among open orders for the working order. Automatically generated if not sent. |
| workingPrice | DECIMAL | YES |  |
| workingQuantity | DECIMAL | YES |  |
| workingIcebergQty | DECIMAL | NO | This can only be used if `workingTimeInForce` is `GTC` or if `workingType` is `LIMIT_MAKER`. |
| workingTimeInForce | ENUM | NO | Supported values: [Time In Force](./enums.md#timeinforce) |
| workingStrategyId | LONG | NO | Arbitrary numeric value identifying the working order within an order strategy. |
| workingStrategyType | INT | NO | Arbitrary numeric value identifying the working order strategy. Values smaller than 1000000 are reserved and cannot be used. |
| workingPegPriceType | ENUM | NO | See [Pegged Orders](#pegged-orders-info) |
| workingPegOffsetType | ENUM | NO |  |
| workingPegOffsetValue | INT | NO |  |
| pendingSide | ENUM | YES | Supported values: [Order side](./enums.md#side) |
| pendingAboveType | ENUM | YES | Supported values: `STOP_LOSS_LIMIT`, `STOP_LOSS`, `LIMIT_MAKER`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT` |
| pendingAboveClientOrderId | STRING | NO | Arbitrary unique ID among open orders for the pending above order. Automatically generated if not sent. |
| pendingAbovePrice | DECIMAL | NO | Can be used if `pendingAboveType` is `STOP_LOSS_LIMIT` , `LIMIT_MAKER`, or `TAKE_PROFIT_LIMIT` to specify the limit price. |
| pendingAboveStopPrice | DECIMAL | NO | Can be used if `pendingAboveType` is `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, `TAKE_PROFIT_LIMIT` |
| pendingAboveTrailingDelta | DECIMAL | NO | See [Trailing Stop FAQ](../faqs/trailing-stop-faq.md) |
| pendingAboveIcebergQty | DECIMAL | NO | This can only be used if `pendingAboveTimeInForce` is `GTC` or if `pendingAboveType` is `LIMIT_MAKER`. |
| pendingAboveTimeInForce | ENUM | NO |  |
| pendingAboveStrategyId | LONG | NO | Arbitrary numeric value identifying the pending above order within an order strategy. |
| pendingAboveStrategyType | INT | NO | Arbitrary numeric value identifying the pending above order strategy. Values smaller than 1000000 are reserved and cannot be used. |
| pendingAbovePegPriceType | ENUM | NO | See [Pegged Orders](#pegged-orders-info) |
| pendingAbovePegOffsetType | ENUM | NO |  |
| pendingAbovePegOffsetValue | INT | NO |  |
| pendingBelowType | ENUM | NO | Supported values: `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`,`TAKE_PROFIT_LIMIT` |
| pendingBelowClientOrderId | STRING | NO | Arbitrary unique ID among open orders for the pending below order. Automatically generated if not sent. |
| pendingBelowPrice | DECIMAL | NO | Can be used if `pendingBelowType` is `STOP_LOSS_LIMIT` or `TAKE_PROFIT_LIMIT` to specify limit price |
| pendingBelowStopPrice | DECIMAL | NO | Can be used if `pendingBelowType` is `STOP_LOSS`, `STOP_LOSS_LIMIT, TAKE_PROFIT or TAKE_PROFIT_LIMIT`. Either `pendingBelowStopPrice` or `pendingBelowTrailingDelta` or both, must be specified. |
| pendingBelowTrailingDelta | DECIMAL | NO |  |
| pendingBelowIcebergQty | DECIMAL | NO | This can only be used if `pendingBelowTimeInForce` is `GTC`, or if `pendingBelowType` is `LIMIT_MAKER`. |
| pendingBelowTimeInForce | ENUM | NO | Supported values: [Time In Force](./enums.md#timeinforce) |
| pendingBelowStrategyId | LONG | NO | Arbitrary numeric value identifying the pending below order within an order strategy. |
| pendingBelowStrategyType | INT | NO | Arbitrary numeric value identifying the pending below order strategy. Values smaller than 1000000 are reserved and cannot be used. |
| pendingBelowPegPriceType | ENUM | NO | See [Pegged Orders](#pegged-orders-info) |
| pendingBelowPegOffsetType | ENUM | NO |  |
| pendingBelowPegOffsetValue | INT | NO |  |
| recvWindow | DECIMAL | NO | The value cannot be greater than `60000`. Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified. |
| timestamp | LONG | YES |  |

**Response**

```javascript
{
    "orderListId": 2,
    "contingencyType": "OTO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "bcedxMpQG6nFrZUPQyshoL",
    "transactionTime": 1763000506354,
    "symbol": "BTCUSDT",
    "orders": [
        {
            "symbol": "BTCUSDT",
            "orderId": 9,
            "clientOrderId": "OLSBhMWaIlLSzZ9Zm7fnKB"
        },
        {
            "symbol": "BTCUSDT",
            "orderId": 10,
            "clientOrderId": "mfif39yPTHsB3C0FIXznR2"
        },
        {
            "symbol": "BTCUSDT",
            "orderId": 11,
            "clientOrderId": "yINkaXSJeoi3bU5vWMY8Z8"
        }
    ],
    "orderReports": [
        {
            "symbol": "BTCUSDT",
            "orderId": 9,
            "orderListId": 2,
            "clientOrderId": "OLSBhMWaIlLSzZ9Zm7fnKB",
            "transactTime": 1763000506354,
            "price": "102496.00000000",
            "origQty": "0.00170000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.00000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "NEW",
            "timeInForce": "GTC",
            "type": "LIMIT",
            "side": "BUY",
            "workingTime": 1763000506354,
            "selfTradePreventionMode": "NONE"
        },
        {
            "symbol": "BTCUSDT",
            "orderId": 10,
            "orderListId": 2,
            "clientOrderId": "mfif39yPTHsB3C0FIXznR2",
            "transactTime": 1763000506354,
            "price": "101613.00000000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.00000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "PENDING_NEW",
            "timeInForce": "GTC",
            "type": "STOP_LOSS_LIMIT",
            "side": "SELL",
            "stopPrice": "10100.00000000",
            "workingTime": -1,
            "selfTradePreventionMode": "NONE"
        },
        {
            "symbol": "BTCUSDT",
            "orderId": 11,
            "orderListId": 2,
            "clientOrderId": "yINkaXSJeoi3bU5vWMY8Z8",
            "transactTime": 1763000506354,
            "price": "104261.00000000",
            "executedQty": "0.00000000",
            "origQuoteOrderQty": "0.00000000",
            "cummulativeQuoteQty": "0.00000000",
            "status": "PENDING_NEW",
            "timeInForce": "GTC",
            "type": "LIMIT_MAKER",
            "side": "SELL",
            "workingTime": -1,
            "selfTradePreventionMode": "NONE"
        }
    ]
}
```

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

#### Cancel Order list (TRADE)

```
DELETE /api/v3/orderList
```
Cancel an entire Order list

**Weight:**
1

**Parameters:**

Name| Type| Mandatory| Description
----| ----|------|------
symbol|STRING| YES|
orderListId|LONG|NO| Either `orderListId` or `listClientOrderId` must be provided
listClientOrderId|STRING|NO| Either `orderListId` or `listClientOrderId` must be provided
newClientOrderId|STRING|NO| Used to uniquely identify this cancel. Automatically generated by default
recvWindow|DECIMAL|NO| The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp|LONG|YES|

**Notes:**
* Canceling an individual order from an order list will cancel the entire order list.
* If both `orderListId` and `listClientOrderId` parameters are provided, the `orderListId` is searched first, then the `listClientOrderId` from that result is checked against that order. If both conditions are not met the request will be rejected.

**Data Source:**
Matching Engine

**Response:**

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
      "stopPrice": "1.00000000",
      "selfTradePreventionMode": "NONE"
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
      "side": "SELL",
      "selfTradePreventionMode": "NONE"
    }
  ]
}
```

### SOR

#### New order using SOR (TRADE)

```
POST /api/v3/sor/order
```
Places an order using smart order routing (SOR).

This adds 1 order to the `EXCHANGE_MAX_ORDERS` filter and the `MAX_NUM_ORDERS` filter.

Read [SOR FAQ](../faqs/sor_faq.md) to learn more.

**Weight:**
1

**Unfilled Order Count:**
1


**Parameters:**

Name                    | Type   | Mandatory | Description
------------            | -----  | ------------ | ------------
symbol                  | STRING | YES |
side                    | ENUM   | YES |
type                    | ENUM   | YES |
timeInForce             | ENUM   | NO |
quantity                | DECIMAL| YES |
price                   | DECIMAL| NO |
newClientOrderId        | STRING | NO | A unique id among open orders. Automatically generated if not sent.<br/> Orders with the same `newClientOrderID` can be accepted only when the previous one is filled, otherwise the order will be rejected.
strategyId              |LONG     | NO|
strategyType            |INT     | NO| The value cannot be less than `1000000`.
icebergQty              | DECIMAL| NO | Used with `LIMIT` to create an iceberg order.
newOrderRespType        | ENUM   | NO | Set the response JSON. `ACK`, `RESULT`, or `FULL`. Default to `FULL`
selfTradePreventionMode |ENUM    | NO | The allowed enums is dependent on what is configured on the symbol. The possible supported values are: [STP Modes](enums.md#stpmodes).
recvWindow              | DECIMAL  | NO |The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp               | LONG | YES |

**Note:** `POST /api/v3/sor/order` only supports `LIMIT` and `MARKET` orders. `quoteOrderQty` is not supported.

**Data Source:**
Matching Engine

**Response:**

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

#### Test new order using SOR (TRADE)

```
POST /api/v3/sor/order/test
```

Test new order creation and signature/recvWindow using smart order routing (SOR).
Creates and validates a new order but does not send it into the matching engine.

**Weight:**
| Condition | Request Weight |
| --------- | -------------- |
| Without `computeCommissionRates`  |  1 |
| With `computeCommissionRates`     | 20 |


**Parameters:**

In addition to all parameters accepted by [`POST /api/v3/sor/order`](#new-order-using-sor-trade),
the following optional parameters are also accepted:

Name                   |Type          | Mandatory    | Description
------------           | ------------ | ------------ | ------------
computeCommissionRates | BOOLEAN      | NO            | Default: `false`


**Data Source:**
Memory

**Response:**

Without `computeCommissionRates`

```
{}
```

With `computeCommissionRates`


```javascript
{
  "standardCommissionForOrder": {  //Standard commission rates on trades from the order.
    "maker": "0.00000112",
    "taker": "0.00000114"
  },
  "taxCommissionForOrder": {       //Tax commission rates for trades from the order
    "maker": "0.00000112",
    "taker": "0.00000114"
  },
  "discount": {                    //Discount on standard commissions when paying in BNB.
    "enabledForAccount": true,
    "enabledForSymbol": true,
    "discountAsset": "BNB",
    "discount": "0.25000000"       //Standard commission is reduced by this rate when paying commission in BNB.
  }
}
```



## Account Endpoints

### Account information (USER_DATA)
```
GET /api/v3/account
```
Get current account information.

**Weight:**
20

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
omitZeroBalances |BOOLEAN| NO | When set to `true`, emits only the non-zero balances of an account. <br>Default value: `false`
recvWindow | DECIMAL | NO | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
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

### Current open orders (USER_DATA)
```
GET /api/v3/openOrders
```
Get all open orders on a symbol. **Careful** when accessing this with no symbol.

**Weight:**
6 for a single symbol; **80** when the symbol parameter is omitted

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
recvWindow | DECIMAL | NO | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
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
    "orderListId": -1, // Unless it's part of an order list, value will be -1
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
GET /api/v3/allOrders
```
Get all account orders; active, canceled, or filled.

**Weight:**
20

**Data Source:**
Database

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; Maximum: 1000.
recvWindow | DECIMAL | NO | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp | LONG | YES |

**Notes:**
* If `orderId` is set, it will get orders >= that `orderId`. Otherwise most recent orders are returned.
* For some historical orders `cummulativeQuoteQty` will be < 0, meaning the data is not available at this time.
* If `startTime` and/or `endTime` provided, `orderId`  is not required.
* The time between `startTime` and `endTime` can't be longer than 24 hours.

**Response:**
```javascript
[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "orderListId": -1, //Unless it's part of an order list, value will be -1
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

### Query Order list (USER_DATA)

```
GET /api/v3/orderList
```
Retrieves a specific order list based on provided optional parameters.

**Weight:**

4

**Parameters**:

Name| Type|Mandatory| Description
----|-----|----|----------
orderListId|LONG|NO*| Query order list by `orderListId`. <br>`orderListId` or `origClientOrderId` must be provided.
origClientOrderId|STRING|NO*| Query order list by `listClientOrderId`. <br>`orderListId` or `origClientOrderId` must be provided.
recvWindow|DECIMAL|NO| The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
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

### Query all Order lists (USER_DATA)

```
GET /api/v3/allOrderList
```

Retrieves all order lists based on provided optional parameters

Note that the time between `startTime` and `endTime` can't be longer than 24 hours.

**Weight:**

20

**Parameters:**

Name|Type| Mandatory| Description
----|----|----|---------
fromId|LONG|NO| If supplied, neither `startTime` or `endTime` can be provided
startTime|LONG|NO|
endTime|LONG|NO|
limit|INT|NO| Default: 500; Maximum: 1000
recvWindow|DECIMAL|NO| The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
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

### Query Open Order lists (USER_DATA)

```
GET /api/v3/openOrderList
```

**Weight**:
6

**Parameters:**

Name| Type|Mandatory| Description
----|-----|---|------------------
recvWindow|DECIMAL|NO| The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
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

### Account trade list (USER_DATA)
```
GET /api/v3/myTrades
```
Get trades for a specific account and symbol.

**Weight:**

Condition| Weight|
---| ---
|Without orderId|20|
|With orderId|5|


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId|LONG|NO| This can only be used in combination with `symbol`.
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO | TradeId to fetch from. Default gets most recent trades.
limit | INT | NO | Default: 500; Maximum: 1000.
recvWindow | DECIMAL | NO | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
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
<a id="query-unfilled-order-count"></a>
### Query Unfilled Order Count (USER_DATA)
```
GET /api/v3/rateLimit/order
```

Displays the user's unfilled order count for all intervals.

**Weight:**
40

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | DECIMAL | NO | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
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
limit               |INT     | NO           | Default: `500`; Maximum: `1000`
recvWindow          | DECIMAL   | NO           | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp           | LONG   | YES          |

**Weight:**

Case                            | Weight
----                            | -----
If `symbol` is invalid          | 2
Querying by `preventedMatchId`  | 2
Querying by `orderId`           | 20

**Data Source:**

Database

**Response:**

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

### Query Allocations (USER_DATA)

```
GET /api/v3/myAllocations
```

Retrieves allocations resulting from SOR order placement.

**Weight:**
20

**Parameters:**

Name                     | Type  |Mandatory | Description
-----                    | ---   |----      | ---------
symbol                   |STRING |Yes        |
startTime                |LONG   |No        |
endTime                  |LONG   |No        |
fromAllocationId         |INT    |No        |
limit                    |INT    |No        |Default: 500; Maximum: 1000
orderId                  |LONG   |No        |
recvWindow               |DECIMAL  |No        |The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp                |LONG   |No        |

Supported parameter combinations:

Parameters                                  | Response |
------------------------------------------- | -------- |
`symbol`                                    | allocations from oldest to newest |
`symbol` + `startTime`                      | oldest allocations since `startTime` |
`symbol` + `endTime`                        | newest allocations until `endTime` |
`symbol` + `startTime` + `endTime`          | allocations within the time range |
`symbol` + `fromAllocationId`               | allocations by allocation ID |
`symbol` + `orderId`                        | allocations related to an order starting with oldest |
`symbol` + `orderId` + `fromAllocationId`   | allocations related to an order by allocation ID |

**Note:** The time between `startTime` and `endTime` can't be longer than 24 hours.

**Data Source:**
Database

**Response:**

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

### Query Commission Rates (USER_DATA)

```
GET /api/v3/account/commission
```

Get current account commission rates.


**Weight:**
20

**Parameters:**

Name         | Type    | Mandatory | Description
------------ | -----   | ------------ | ------------
symbol        | STRING | YES          |

**Data Source:**
Database

**Response:**

```javascript
{
  "symbol": "BTCUSDT",
  "standardCommission": {         //Commission rates on trades from the order.
    "maker": "0.00000010",
    "taker": "0.00000020",
    "buyer": "0.00000030",
    "seller": "0.00000040"
  },
  "specialCommission": {         // Special commission rates from the order.
    "maker": "0.01000000",
    "taker": "0.02000000",
    "buyer": "0.03000000",
    "seller": "0.04000000"
  },
  "taxCommission": {              //Tax commission rates for trades from the order.
    "maker": "0.00000112",
    "taker": "0.00000114",
    "buyer": "0.00000118",
    "seller": "0.00000116"
  },
  "discount": {                   //Discount commission when paying in BNB
    "enabledForAccount": true,
    "enabledForSymbol": true,
    "discountAsset": "BNB",
    "discount": "0.75000000"      //Standard commission is reduced by this rate when paying commission in BNB.
  }
}
```

### Query Order Amendments (USER_DATA)

```
GET /api/v3/order/amendments
```

Queries all amendments of a single order.

**Weight**:
4

**Parameters:**

Name | Type | Mandatory | Description |
:---- | :---- | :---- | :---- |
symbol | STRING | YES |  |
orderId | LONG | YES |  |
fromExecutionId | LONG | NO |  |
limit | LONG | NO | Default:500; Maximum: 1000 |
recvWindow | DECIMAL | NO | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp | LONG | YES |

**Data Source:**

Database

**Response:**

```json
[
  {
      "symbol": "BTCUSDT",
      "orderId": 9,
      "executionId": 22,
      "origClientOrderId": "W0fJ9fiLKHOJutovPK3oJp",
      "newClientOrderId": "UQ1Np3bmQ71jJzsSDW9Vpi",
      "origQty": "5.00000000",
      "newQty": "4.00000000",
      "time": 1741669661670
  },
  {
      "symbol": "BTCUDST",
      "orderId": 9,
      "executionId": 25,
      "origClientOrderId": "UQ1Np3bmQ71jJzsSDW9Vpi",
      "newClientOrderId": "5uS0r35ohuQyDlCzZuYXq2",
      "origQty": "4.00000000",
      "newQty": "3.00000000",
      "time": 1741672924895
  }
]
```

<a id="myFilters"></a>
### Query relevant filters (USER_DATA)

```
GET /api/v3/myFilters
```

Retrieves the list of [filters](filters.md) relevant to an account on a given symbol. This is the only endpoint that shows if an account has [`MAX_ASSET`](filters.md#max_asset) filters applied to it.

**Weight:**
40

**Parameters:**

Name       | Type         | Mandatory    | Description
---------- | ------------ | ------------ | ------------
symbol     | STRING       | YES          |
recvWindow | DECIMAL      | NO           | The value cannot be greater than `60000`. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
timestamp  | LONG         | YES          |

**Data Source:**
Memory

**Response:**

```javascript
{
  "exchangeFilters": [
    {
      "filterType": "EXCHANGE_MAX_NUM_ORDERS",
      "maxNumOrders": 1000
    }
  ],
  "symbolFilters": [
    {
      "filterType": "MAX_NUM_ORDER_LISTS",
      "maxNumOrderLists": 20
    }
  ],
  "assetFilters": [
    {
      "filterType": "MAX_ASSET",
      "asset": "JPY",
      "limit": "1000000.00000000"
    }
  ]
}
```
