<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

  - [General API Information](#general-api-information)
  - [Request format](#request-format)
  - [Response format](#response-format)
    - [Status codes](#status-codes)
- [Rate limits](#rate-limits)
  - [Connection limits](#connection-limits)
  - [General information on rate limits](#general-information-on-rate-limits)
    - [How to interpret rate limits](#how-to-interpret-rate-limits)
    - [How to show/hide rate limit information](#how-to-showhide-rate-limit-information)
  - [IP limits](#ip-limits)
  - [Order rate limits](#order-rate-limits)
- [Request security](#request-security)
  - [SIGNED (TRADE and USER_DATA) request security](#signed-trade-and-user_data-request-security)
  - [Timing security](#timing-security)
  - [SIGNED request example (HMAC)](#signed-request-example-hmac)
  - [SIGNED request example (RSA)](#signed-request-example-rsa)
- [Data sources](#data-sources)
- [Public API requests](#public-api-requests)
    - [Terminology](#terminology)
  - [ENUM definitions](#enum-definitions)
  - [General requests](#general-requests)
    - [Test connectivity](#test-connectivity)
    - [Check server time](#check-server-time)
    - [Exchange information](#exchange-information)
  - [Market data requests](#market-data-requests)
    - [Order book](#order-book)
    - [Recent trades](#recent-trades)
    - [Historical trades (MARKET_DATA)](#historical-trades-market_data)
    - [Aggregate trades](#aggregate-trades)
    - [Klines](#klines)
    - [UI Klines](#ui-klines)
    - [Current average price](#current-average-price)
    - [24hr ticker price change statistics](#24hr-ticker-price-change-statistics)
    - [Rolling window price change statistics](#rolling-window-price-change-statistics)
    - [Symbol price ticker](#symbol-price-ticker)
    - [Symbol order book ticker](#symbol-order-book-ticker)
  - [Trading requests](#trading-requests)
    - [Place new order (TRADE)](#place-new-order-trade)
      - [Conditional fields in Order Responses](#conditional-fields-in-order-responses)
    - [Test new order (TRADE)](#test-new-order-trade)
    - [Query order (USER_DATA)](#query-order-user_data)
    - [Cancel order (TRADE)](#cancel-order-trade)
      - [Regarding `cancelRestrictions`](#regarding-cancelrestrictions)
    - [Cancel and replace order (TRADE)](#cancel-and-replace-order-trade)
    - [Current open orders (USER_DATA)](#current-open-orders-user_data)
    - [Cancel open orders (TRADE)](#cancel-open-orders-trade)
    - [Place new OCO (TRADE)](#place-new-oco-trade)
    - [Query OCO (USER_DATA)](#query-oco-user_data)
    - [Cancel OCO (TRADE)](#cancel-oco-trade)
    - [Current open OCOs (USER_DATA)](#current-open-ocos-user_data)
  - [Account requests](#account-requests)
    - [Account information (USER_DATA)](#account-information-user_data)
    - [Account order rate limits (USER_DATA)](#account-order-rate-limits-user_data)
    - [Account order history (USER_DATA)](#account-order-history-user_data)
    - [Account OCO history (USER_DATA)](#account-oco-history-user_data)
    - [Account trade history (USER_DATA)](#account-trade-history-user_data)
    - [Account prevented matches (USER_DATA)](#account-prevented-matches-user_data)
  - [User Data Stream requests](#user-data-stream-requests)
    - [Start user data stream (USER_STREAM)](#start-user-data-stream-user_stream)
    - [Ping user data stream (USER_STREAM)](#ping-user-data-stream-user_stream)
    - [Stop user data stream (USER_STREAM)](#stop-user-data-stream-user_stream)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Public WebSocket API for Binance (2023-06-07)

## General API Information

* The base endpoint is: **`wss://ws-api.binance.com:443/ws-api/v3`**
  * If you experience issues with the standard 443 port, alternative port 9443 is also available.
  * The base endpoint for [testnet](https://testnet.binance.vision/) is: `wss://testnet.binance.vision/ws-api/v3`
* A single connection to the API is only valid for 24 hours; expect to be disconnected after the 24-hour mark.
* WebSocket server will send a **ping frame** every 3 minutes.
  * If the server does not receive a **pong frame** response within 10 minutes, you will be disconnected.
  * Unsolicited pong frames are allowed and will prevent disconnection.
* Lists are returned in **chronological order**, unless noted otherwise.
* All timestamps are in **milliseconds** in UTC, unless noted otherwise.
* All field names and values are **case-sensitive**, unless noted otherwise.

## Request format

Requests must be sent as JSON in **text frames**, one request per frame.

Example of request:

```json
{
  "id": "e2a85d9f-07a5-4f94-8d5f-789dc3deb097",
  "method": "order.place",
  "params": {
    "symbol": "BTCUSDT",
    "side": "BUY",
    "type": "LIMIT",
    "price": "0.1",
    "quantity": "10",
    "timeInForce": "GTC",
    "timestamp": 1655716096498,
    "apiKey": "T59MTDLWlpRW16JVeZ2Nju5A5C98WkMm8CSzWC4oqynUlTm1zXOxyauT8LmwXEv9",
    "signature": "5942ad337e6779f2f4c62cd1c26dba71c91514400a24990a3e7f5edec9323f90"
  }
}
```

Request fields:

Name     | Type    | Mandatory | Description
--------- | ------- | --------- | -----------
`id`      | INT / STRING / `null` | YES | Arbitrary ID used to match responses to requests
`method`  | STRING  | YES       | Request method name
`params`  | OBJECT  | NO        | Request parameters. May be omitted if there are no parameters

* Request `id` is truly arbitrary. You can use UUIDs, sequential IDs, current timestamp, etc.
  The server does not interpret `id` in any way, simply echoing it back in the response.

  You can freely reuse IDs within a session.
  However, be careful to not send more than one request at a time with the same ID,
  since otherwise it might be impossible to tell the responses apart.

* Request method names may be prefixed with explicit version: e.g., `"v3/order.place"`.

* The order of `params` is not significant.

## Response format

Responses are returned as JSON in **text frames**, one response per frame.

Example of successful response:

```json
{
  "id": "e2a85d9f-07a5-4f94-8d5f-789dc3deb097",
  "status": 200,
  "result": {
    "symbol": "BTCUSDT",
    "orderId": 12510053279,
    "orderListId": -1,
    "clientOrderId": "a097fe6304b20a7e4fc436",
    "transactTime": 1655716096505,
    "price": "0.10000000",
    "origQty": "10.00000000",
    "executedQty": "0.00000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "workingTime": 1655716096505,
    "selfTradePreventionMode": "NONE"
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 12
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 4043
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 321
    }
  ]
}
```

Example of failed response:

```json
{
  "id": "e2a85d9f-07a5-4f94-8d5f-789dc3deb097",
  "status": 400,
  "error": {
    "code": -2010,
    "msg": "Account has insufficient balance for requested action."
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 13
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 4044
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 322
    }
  ]
}
```

Response fields:

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
    <td><code>id</code></td>
    <td>INT / STRING / <code>null</code></td>
    <td>YES</td>
    <td>Same as in the original request</td>
  </tr>
  <tr>
    <td><code>status</code></td>
    <td>INT</td>
    <td>YES</td>
    <td>Response status. See <a href="#status-codes">Status codes</a></td>
  </tr>
  <tr>
    <td><code>result</code></td>
    <td>OBJECT / ARRAY</td>
    <td rowspan="2">YES</td>
    <td>Response content. Present if request succeeded</td>
  </tr>
  <tr>
    <td><code>error</code></td>
    <td>OBJECT</td>
    <td>Error description. Present if request failed</td>
  </tr>
  <tr>
    <td><code>rateLimits</code></td>
    <td>ARRAY</td>
    <td>NO</td>
    <td>Rate limiting status. See <a href="#rate-limits">Rate limits</a></td>
  </tr>
</tbody>
</table>

### Status codes

Status codes in the `status` field are the same as in HTTP.

Here are some common status codes that you might encounter:

* `200` indicates a successful response.
* `4XX` status codes indicate invalid requests; the issue is on your side.
  * `400` – your request failed, see `error` for the reason.
  * `403` – you have been blocked by the Web Application Firewall.
  * `409` – your request partially failed but also partially succeeded, see `error` for details.
  * `418` – you have been auto-banned for repeated violation of rate limits.
  * `429` – you have exceeded API request rate limit, please slow down.
* `5XX` status codes indicate internal errors; the issue is on Binance's side.
  * **Important:** If a response contains 5xx status code, it **does not** necessarily mean that your request has failed.
    Execution status is _unknown_ and the request might have actually succeeded.
    Please use query methods to confirm the status.
    You might also want to establish a new WebSocket connection for that.

See [Error codes for Binance](errors.md) for a list of error codes and messages.

# Rate limits

## Connection limits

There is a limit of **300 connections per attempt every 5 minutes**. 

The connection is per **IP address**.

## General information on rate limits

* Current API rate limits can be queried using the [`exchangeInfo`](#exchange-information) request.
* There are multiple rate limit types across multiple intervals.
* Responses can indicate current rate limit status in the optional `rateLimits` field.
* Requests fail with status `429` when order rate limits or request rate limits are violated.

### How to interpret rate limits

A response with rate limit status may look like this:

```json
{
  "id": "7069b743-f477-4ae3-81db-db9b8df085d2",
  "status": 200,
  "result": {
    "serverTime": 1656400526260
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 70
    }
  ]
}
```

The `rateLimits` array describes all currently active rate limits affected by the request.

| Name            | Type    | Mandatory | Description
| --------------- | ------- | --------- | -----------
| `rateLimitType` | ENUM    | YES       | Rate limit type: `REQUEST_WEIGHT`, `ORDERS`
| `interval`      | ENUM    | YES       | Rate limit interval: `SECOND`, `MINUTE`, `HOUR`, `DAY`
| `intervalNum`   | INT     | YES       | Rate limit interval multiplier
| `limit`         | INT     | YES       | Request limit per interval
| `count`         | INT     | YES       | Current usage per interval

Rate limits are accounted by intervals.

For example, a `1 MINUTE` interval starts every minute.
Request submitted at 00:01:23.456 counts towards the 00:01:00 minute's limit.
Once the 00:02:00 minute starts, the count will reset to zero again.

Other intervals behave in a similar manner.
For example, `1 DAY` rate limit resets at 00:00 UTC every day,
and `10 SECOND` interval resets at 00, 10, 20... seconds of each minute.

APIs have multiple rate-limiting intervals.
If you exhaust a shorter interval but the longer interval still allows requests,
you will have to wait for the shorter interval to expire and reset.
If you exhaust a longer interval, you will have to wait for that interval to reset,
even if shorter rate limit count is zero.

### How to show/hide rate limit information

`rateLimits` field is included with every response by default.

However, rate limit information can be quite bulky.
If you are not interested in detailed rate limit status of every request,
the `rateLimits` field can be omitted from responses to reduce their size.

* Optional `returnRateLimits` boolean parameter in request.

  Use `returnRateLimits` parameter to control whether to include `rateLimits` fields in response to individual requests.

  Default request and response:

  ```json
  {"id":1,"method":"time"}
  ```

  ```json
  {"id":1,"status":200,"result":{"serverTime":1656400526260},"rateLimits":[{"rateLimitType":"REQUEST_WEIGHT","interval":"MINUTE","intervalNum":1,"limit":1200,"count":70}]}
  ```

  Request and response without rate limit status:

  ```json
  {"id":2,"method":"time","params":{"returnRateLimits":false}}
  ```

  ```json
  {"id":2,"status":200,"result":{"serverTime":1656400527891}}
  ```

* Optional `returnRateLimits` boolean parameter in connection URL.

  If you wish to omit `rateLimits` from all responses by default,
  use `returnRateLimits` parameter in the query string instead:

  ```
  wss://ws-api.binance.com/ws-api/v3?returnRateLimits=false
  ```

  This will make all requests made through this connection behave as if you have passed `"returnRateLimits": false`.

  If you _want_ to see rate limits for a particular request,
  you need to explicitly pass the `"returnRateLimits": true` parameter.

**Note:** Your requests are still rate limited if you hide the `rateLimits` field in responses.

## IP limits

* Every request has a certain **weight**, added to your limit as you perform requests.
  * Most requests cost 1 unit of weight, heavier requests acting on multiple symbols cost more.
  * Connecting to WebSocket API costs 1 weight.
* Current weight usage is indicated by the `REQUEST_WEIGHT` rate limit type.
* Use the [`exchangeInfo`](#exchange-information) request
  to keep track of the current weight limits.
* Weight is accumulated **per IP address** and is shared by all connections from that address.
* If you go over the weight limit, requests fail with status `429`.
  * This status code indicates you should back off and stop spamming the API.
  * Rate-limited responses include a `retryAfter` field, indicating when you can retry the request.
* **Repeatedly violating rate limits and/or failing to back off after receiving 429s will result in an automated IP ban and you will be disconnected.**
  * Requests from a banned IP address fail with status `418`.
  * `retryAfter` field indicates the timestamp when the ban will be lifted.
* IP bans are tracked and **scale in duration** for repeat offenders, **from 2 minutes to 3 days**.

Successful response indicating that in 1 minute you have used 70 weight out of your 1200 limit:

```json
{
  "id": "7069b743-f477-4ae3-81db-db9b8df085d2",
  "status": 200,
  "result": [],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 70
    }
  ]
}
```

Failed response indicating that you are banned and the ban will last until epoch `1659146400000`:

```json
{
  "id": "fc93a61a-a192-4cf4-bb2a-a8f0f0c51e06",
  "status": 418,
  "error": {
    "code": -1003,
    "msg": "Way too much request weight used; IP banned until 1659146400000. Please use WebSocket Streams for live updates to avoid bans.",
    "data": {
      "serverTime": 1659142907531,
      "retryAfter": 1659146400000
    }
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 2411
    }
  ]
}
```

## Order rate limits

* Every request to place an order counts towards your **order limit**.
  * Successfully placed orders update the `ORDERS` rate limit type.
  * Rejected or unsuccessful orders might or might not update the `ORDERS` count.
* Use the [`account.rateLimits.orders`](#account-order-rate-limits-user_data) request to keep track of the current order rate limits.
* Order rate limit is maintained **per account** and is shared by all API keys of the account.
* If you go over the order rate limit, requests fail with status `429`.
  * This status code indicates you should back off and stop spamming the API.
  * Rate-limited responses include a `retryAfter` field, indicating when you can retry the request.

Successful response indicating that you have placed 12 orders in 10 seconds, and 4043 orders in the past 24 hours:

```json
{
  "id": "e2a85d9f-07a5-4f94-8d5f-789dc3deb097",
  "status": 200,
  "result": {
    "symbol": "BTCUSDT",
    "orderId": 12510053279,
    "orderListId": -1,
    "clientOrderId": "a097fe6304b20a7e4fc436",
    "transactTime": 1655716096505,
    "price": "0.10000000",
    "origQty": "10.00000000",
    "executedQty": "0.00000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "workingTime": 1655716096505,
    "selfTradePreventionMode": "NONE"
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 12
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 4043
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 321
    }
  ]
}
```

# Request security

* Every method has a security type which determines how to call it.
  * Security type is stated next to the method name.
    For example, [Place new order (TRADE)](#place-new-order-trade).
  * If no security type is stated, the security type is NONE.

Security type | API key  | Signature | Description
------------- | -------- | --------- | ------------
`NONE`        |          |           | Public market data
`TRADE`       | required | required  | Trading on the exchange, placing and canceling orders
`USER_DATA`   | required | required  | Private account information, such as order status and your trading history
`USER_STREAM` | required |           | Managing User Data Stream subscriptions
`MARKET_DATA` | required |           | Historical market data access

* Secure methods require a valid API key to be specified and authenticated.
  * API keys can be created on the [API Management](https://www.binance.com/en/support/faq/360002502072) page of your Binance account.
  * **Both API key and secret key are sensitive.** Never share them with anyone.
    If you notice unusual activity in your account, immediately revoke all the keys and contact Binance support.
* API keys can be configured to allow access only to certain types of secure methods.
  * For example, you can have an API key with `TRADE` permission for trading,
    while using a separate API key with `USER_DATA` permission to monitor your order status.
  * By default, an API key cannot `TRADE`. You need to enable trading in API Management first.
* `TRADE` and `USER_DATA` requests are also known as `SIGNED` requests.

## SIGNED (TRADE and USER_DATA) request security

* `SIGNED` requests require an additional parameter: `signature`, authorizing the request.
* Please consult [SIGNED request example (HMAC)](#signed-request-example-hmac) and [SIGNED request example (RSA)](#signed-request-example-rsa) on how to compute signature.

## Timing security

* `SIGNED` requests also require a `timestamp` parameter which should be the current millisecond timestamp.
* An additional optional parameter, `recvWindow`, specifies for how long the request stays valid.
  * If `recvWindow` is not sent, **it defaults to 5000 milliseconds**.
  * Maximum `recvWindow` is 60000 milliseconds.
* Request processing logic is as follows:

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

**It is recommended to use a small `recvWindow` of 5000 or less!**

## SIGNED request example (HMAC)

Here is a step-by-step guide on how to sign requests using HMAC secret key.

Example API key and secret key:

Key          | Value
------------ | ------------
apiKey       | `vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A`
secretKey    | `NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j`

**WARNING: DO NOT SHARE YOUR API KEY AND SECRET KEY WITH ANYONE.**

The example keys are provided here only for illustrative purposes.

Example of request:

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol":           "BTCUSDT",
    "side":             "SELL",
    "type":             "LIMIT",
    "timeInForce":      "GTC",
    "quantity":         "0.01000000",
    "price":            "52000.00",
    "newOrderRespType": "ACK",
    "recvWindow":       100,
    "timestamp":        1645423376532,
    "apiKey":           "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature":        "------ FILL ME ------"
  }
}
```

As you can see, the `signature` parameter is currently missing.

**Step 1. Construct the signature payload**

Take all request `params` except for the `signature`, sort them by name in alphabetical order:

Parameter        | Value
---------------- | ------------
apiKey           | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
newOrderRespType | ACK
price            | 52000.00
quantity         | 0.01000000
recvWindow       | 100
side             | SELL
symbol           | BTCUSDT
timeInForce      | GTC
timestamp        | 1645423376532
type             | LIMIT

Format parameters as `parameter=value` pairs separated by `&`.

Resulting signature payload:

```
apiKey=vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A&newOrderRespType=ACK&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT
```

**Step 2. Compute the signature**

1. Interpret `secretKey` as ASCII data, using it as a key for HMAC-SHA-256.
2. Sign signature payload as ASCII data.
3. Encode HMAC-SHA-256 output as a hex string.

Note that `apiKey`, `secretKey`, and the payload are **case-sensitive**, while resulting signature value is case-insensitive.

You can cross-check your signature algorithm implementation with OpenSSL:

```console
$ echo -n 'apiKey=vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A&newOrderRespType=ACK&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT' \
  | openssl dgst -hex -sha256 -hmac 'NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j'

cc15477742bd704c29492d96c7ead9414dfd8e0ec4a00f947bb5bb454ddbd08a
```

**Step 3. Add `signature` to request `params`**

Finally, complete the request by adding the `signature` parameter with the signature string.

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol":           "BTCUSDT",
    "side":             "SELL",
    "type":             "LIMIT",
    "timeInForce":      "GTC",
    "quantity":         "0.01000000",
    "price":            "52000.00",
    "newOrderRespType": "ACK",
    "recvWindow":       100,
    "timestamp":        1645423376532,
    "apiKey":           "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature":        "cc15477742bd704c29492d96c7ead9414dfd8e0ec4a00f947bb5bb454ddbd08a"
  }
}
```

## SIGNED request example (RSA)

Here is a step-by-step guide on how to sign requests using your RSA private key.

Key          | Value
------------ | ------------
apiKey       | `CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ`

In this example, we assume the private key is stored in the `test-prv-key.pem` file.

**WARNING: DO NOT SHARE YOUR API KEY AND PRIVATE KEY WITH ANYONE.**

The example keys are provided here only for illustrative purposes.

Example of request:

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol":           "BTCUSDT",
    "side":             "SELL",
    "type":             "LIMIT",
    "timeInForce":      "GTC",
    "quantity":         "0.01000000",
    "price":            "52000.00",
    "newOrderRespType": "ACK",
    "recvWindow":       100,
    "timestamp":        1645423376532,
    "apiKey":           "CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ",
    "signature":        "------ FILL ME ------"
  }
}
```

**Step 1. Construct the signature payload**

Take all request `params` except for the `signature`, sort them by name in alphabetical order:

Parameter        | Value
---------------- | ------------
apiKey           | CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ
newOrderRespType | ACK
price            | 52000.00
quantity         | 0.01000000
recvWindow       | 100
side             | SELL
symbol           | BTCUSDT
timeInForce      | GTC
timestamp        | 1645423376532
type             | LIMIT

Format parameters as `parameter=value` pairs separated by `&`.

Resulting signature payload:

```
apiKey=CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ&newOrderRespType=ACK&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT
```

**Step 2. Compute the signature**

1. Encode signature payload as ASCII data.
2. Sign payload using RSASSA-PKCS1-v1_5 algorithm with SHA-256 hash function.
3. Encode output as base64 string.

Note that `apiKey`, the payload, and the resulting `signature` are **case-sensitive**.

You can cross-check your signature algorithm implementation with OpenSSL:

```console
$ echo -n 'apiKey=CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ&newOrderRespType=ACK&price=52000.00&quantity=0.01000000&recvWindow=100&side=SELL&symbol=BTCUSDT&timeInForce=GTC&timestamp=1645423376532&type=LIMIT' \
  | openssl dgst -sha256 -sign test-prv-key.pem \
  | openssl enc -base64 -A

OJJaf8C/3VGrU4ATTR4GiUDqL2FboSE1Qw7UnnoYNfXTXHubIl1iaePGuGyfct4NPu5oVEZCH4Q6ZStfB1w4ssgu0uiB/Bg+fBrRFfVgVaLKBdYHMvT+ljUJzqVaeoThG9oXlduiw8PbS9U8DYAbDvWN3jqZLo4Z2YJbyovyDAvDTr/oC0+vssLqP7NmlNb3fF3Bj7StmOwJvQJTbRAtzxK5PP7OQe+0mbW+D7RqVkUiSswR8qJFWTeSe4nXXNIdZdueYhF/Xf25L+KitJS5IHdIHcKfEw3MQzHFb2ZsGWkjDQwxkwr7Noi0Zaa+gFtxCuatGFm9dFIyx217pmSHtA==
```

**Step 3. Add `signature` to request `params`**

Finally, complete the request by adding the `signature` parameter with the signature string.

```json
{
  "id": "4885f793-e5ad-4c3b-8f6c-55d891472b71",
  "method": "order.place",
  "params": {
    "symbol":           "BTCUSDT",
    "side":             "SELL",
    "type":             "LIMIT",
    "timeInForce":      "GTC",
    "quantity":         "0.01000000",
    "price":            "52000.00",
    "newOrderRespType": "ACK",
    "recvWindow":       100,
    "timestamp":        1645423376532,
    "apiKey":           "CAvIjXy3F44yW6Pou5k8Dy1swsYDWJZLeoK2r8G4cFDnE9nosRppc2eKc1T8TRTQ",
    "signature":        "OJJaf8C/3VGrU4ATTR4GiUDqL2FboSE1Qw7UnnoYNfXTXHubIl1iaePGuGyfct4NPu5oVEZCH4Q6ZStfB1w4ssgu0uiB/Bg+fBrRFfVgVaLKBdYHMvT+ljUJzqVaeoThG9oXlduiw8PbS9U8DYAbDvWN3jqZLo4Z2YJbyovyDAvDTr/oC0+vssLqP7NmlNb3fF3Bj7StmOwJvQJTbRAtzxK5PP7OQe+0mbW+D7RqVkUiSswR8qJFWTeSe4nXXNIdZdueYhF/Xf25L+KitJS5IHdIHcKfEw3MQzHFb2ZsGWkjDQwxkwr7Noi0Zaa+gFtxCuatGFm9dFIyx217pmSHtA=="
  }
}
```


# Data sources

* The API system is asynchronous. Some delay in the response is normal and expected.

* Each method has a data source indicating where the data is coming from, and thus how up-to-date it is.

Data Source     | Latency  | Description
--------------- | -------- | -----------
Matching Engine | lowest   | The matching engine produces the response directly
Memory          | low      | Data is fetched from API server's local or external memory cache
Database        | moderate | Data is retrieved from the database

* Some methods have more than one data source (e.g., Memory => Database).

  This means that the API will look for the latest data in that order:
  first in the cache, then in the database.

# Public API requests

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

<a id="permissions"></a>

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
`EXPIRED` | The order was canceled according to the order type's rules (e.g. LIMIT FOK orders with no fill, LIMIT IOC or MARKET orders that partially fill) <br> or by the exchange, (e.g. orders canceled during liquidation, orders canceled during maintenance)
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

## General requests

### Test connectivity

```javascript
{
  "id": "922bcc6e-9de8-440d-9e84-7c80933a8d0d",
  "method": "ping"
}
```

Test connectivity to the WebSocket API.

**Note:**
You can use regular WebSocket ping frames to test connectivity as well,
WebSocket API will respond with pong frames as soon as possible.
`ping` request along with `time` is a safe way to test request-response handling in your application.

**Weight:**
1

**Parameters:**
NONE

**Data Source:**
Memory

**Response:**
```javascript
{
  "id": "922bcc6e-9de8-440d-9e84-7c80933a8d0d",
  "status": 200,
  "result": {},
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### Check server time

```javascript
{
  "id": "187d3cb2-942d-484c-8271-4e2141bbadb1",
  "method": "time"
}
```

Test connectivity to the WebSocket API and get the current server time.

**Weight:**
1

**Parameters:**
NONE

**Data Source:**
Memory

**Response:**
```javascript
{
  "id": "187d3cb2-942d-484c-8271-4e2141bbadb1",
  "status": 200,
  "result": {
    "serverTime": 1656400526260
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### Exchange information

```javascript
{
  "id": "5494febb-d167-46a2-996d-70533eb4d976",
  "method": "exchangeInfo",
  "params": {
    "symbols": ["BNBBTC"]
  }
}
```

Query current exchange trading rules, rate limits, and symbol information.

**Weight:**
10

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td rowspan="3" align="center">NO</td>
        <td>Describe a single symbol</td>
    </tr>
    <tr>
        <td><code>symbols</code></td>
        <td>ARRAY of STRING</td>
        <td>Describe multiple symbols</td>
    </tr>
    <tr>
        <td><code>permissions</code></td>
        <td>ARRAY of STRING</td>
        <td>Filter symbols by permissions</td>
    </tr>
</tbody>
</table>

Notes:

* Only one of `symbol`, `symbols`, `permissions` parameters can be specified.

* Without parameters, `exchangeInfo` displays all symbols with `["SPOT, "MARGIN", "LEVERAGED"]` permissions.

  * In order to list *all* active symbols on the exchange, you need to explicitly request all permissions.

* `permissions` accepts either a list of permissions, or a single permission name. E.g. `"SPOT"`.

* [Available Permissions](#permissions)


**Data Source:**
Memory

**Response:**
```javascript
{
  "id": "5494febb-d167-46a2-996d-70533eb4d976",
  "status": 200,
  "result": {
    "timezone": "UTC",
    "serverTime": 1655969291181,
    // Global rate limits. See "Rate limits" section.
    "rateLimits": [
      {
        "rateLimitType": "REQUEST_WEIGHT",    // Rate limit type: REQUEST_WEIGHT, ORDERS, RAW_REQUESTS
        "interval": "MINUTE",                 // Rate limit interval: SECOND, MINUTE, DAY
        "intervalNum": 1,                     // Rate limit interval multiplier (i.e., "1 minute")
        "limit": 1200                         // Rate limit per interval
      },
      {
        "rateLimitType": "ORDERS",
        "interval": "SECOND",
        "intervalNum": 10,
        "limit": 50
      },
      {
        "rateLimitType": "ORDERS",
        "interval": "DAY",
        "intervalNum": 1,
        "limit": 160000
      },
      {
        "rateLimitType": "RAW_REQUESTS",
        "interval": "MINUTE",
        "intervalNum": 5,
        "limit": 6100
      }
    ],
    // Exchange filters are explained on the "Filters" page:
    // https://github.com/binance/binance-spot-api-docs/blob/master/filters.md
    // All exchange filters are optional.
    "exchangeFilters": [],
    "symbols": [
      {
        "symbol": "BNBBTC",
        "status": "TRADING",
        "baseAsset": "BNB",
        "baseAssetPrecision": 8,
        "quoteAsset": "BTC",
        "quotePrecision": 8,
        "quoteAssetPrecision": 8,
        "baseCommissionPrecision": 8,
        "quoteCommissionPrecision": 8,
        "orderTypes": [
          "LIMIT",
          "LIMIT_MAKER",
          "MARKET",
          "STOP_LOSS_LIMIT",
          "TAKE_PROFIT_LIMIT"
        ],
        "icebergAllowed": true,
        "ocoAllowed": true,
        "quoteOrderQtyMarketAllowed": true,
        "allowTrailingStop": true,
        "cancelReplaceAllowed": true,
        "isSpotTradingAllowed": true,
        "isMarginTradingAllowed": true,
        // Symbol filters are explained on the "Filters" page:
        // https://github.com/binance/binance-spot-api-docs/blob/master/filters.md
        // All symbol filters are optional.
        "filters": [
          {
            "filterType": "PRICE_FILTER",
            "minPrice": "0.00000100",
            "maxPrice": "100000.00000000",
            "tickSize": "0.00000100"
          },
          {
            "filterType": "LOT_SIZE",
            "minQty": "0.00100000",
            "maxQty": "100000.00000000",
            "stepSize": "0.00100000"
          }
        ],
        "permissions": [
          "SPOT",
          "MARGIN",
          "TRD_GRP_004"
        ],
        "defaultSelfTradePreventionMode": "NONE",
        "allowedSelfTradePreventionModes": [
          "NONE"
        ]
      }
    ]
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 10
    }
  ]
}
```

## Market data requests

### Order book

```javascript
{
  "id": "51e2affb-0aba-4821-ba75-f2625006eb43",
  "method": "depth",
  "params": {
    "symbol": "BNBBTC",
    "limit": 5
  }
}
```

Get current order book.

Note that this request returns limited market depth.

If you need to continuously monitor order book updates, please consider using WebSocket Streams:

* [`<symbol>@depth<levels>`](web-socket-streams.md#partial-book-depth-streams)
* [`<symbol>@depth`](web-socket-streams.md#diff-depth-stream)

You can use `depth` request together with `<symbol>@depth` streams to [maintain a local order book](web-socket-streams.md#how-to-manage-a-local-order-book-correctly).

**Weight:**
Adjusted based on the limit:

|  Limit    | Weight |
|:---------:|:------:|
|     1–100 |      1 |
|   101–500 |      5 |
|  501–1000 |     10 |
| 1001–5000 |     50 |

**Parameters:**

Name      | Type    | Mandatory | Description
--------- | ------- | --------- | -----------
`symbol`  | STRING  | YES       |
`limit`   | INT     | NO        | Default 100; max 5000

**Data Source:**
Memory

**Response:**
```javascript
{
  "id": "51e2affb-0aba-4821-ba75-f2625006eb43",
  "status": 200,
  "result": {
    "lastUpdateId": 2731179239,
    // Bid levels are sorted from highest to lowest price.
    "bids": [
      [
        "0.01379900",   // Price
        "3.43200000"    // Quantity
      ],
      [
        "0.01379800",
        "3.24300000"
      ],
      [
        "0.01379700",
        "10.45500000"
      ],
      [
        "0.01379600",
        "3.82100000"
      ],
      [
        "0.01379500",
        "10.26200000"
      ]
    ],
    // Ask levels are sorted from lowest to highest price.
    "asks": [
      [
        "0.01380000",
        "5.91700000"
      ],
      [
        "0.01380100",
        "6.01400000"
      ],
      [
        "0.01380200",
        "0.26800000"
      ],
      [
        "0.01380300",
        "0.33800000"
      ],
      [
        "0.01380400",
        "0.26800000"
      ]
    ]
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```


### Recent trades

```javascript
{
  "id": "409a20bd-253d-41db-a6dd-687862a5882f",
  "method": "trades.recent",
  "params": {
    "symbol": "BNBBTC",
    "limit": 1
  }
}
```

Get recent trades.

If you need access to real-time trading activity, please consider using WebSocket Streams:

* [`<symbol>@trade`](web-socket-streams.md#trade-streams)

**Weight:**
1

**Parameters:**

Name      | Type    | Mandatory | Description
--------- | ------- | --------- | -----------
`symbol`  | STRING  | YES       |
`limit`   | INT     | NO        | Default 500; max 1000

**Data Source:**
Memory

**Response:**
```javascript
{
  "id": "409a20bd-253d-41db-a6dd-687862a5882f",
  "status": 200,
  "result": [
    {
      "id": 194686783,
      "price": "0.01361000",
      "qty": "0.01400000",
      "quoteQty": "0.00019054",
      "time": 1660009530807,
      "isBuyerMaker": true,
      "isBestMatch": true
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### Historical trades (MARKET_DATA)

```javascript
{
  "id": "cffc9c7d-4efc-4ce0-b587-6b87448f052a",
  "method": "trades.historical",
  "params": {
    "symbol": "BNBBTC",
    "fromId": 0,
    "limit": 1,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A"
  }
}
```

Get historical trades.

**Weight:**
5

**Parameters:**

Name      | Type    | Mandatory | Description
--------- | ------- | --------- | -----------
`symbol`  | STRING  | YES       |
`fromId`  | INT     | NO        | Trade ID to begin at
`limit`   | INT     | NO        | Default 500; max 1000
`apiKey`  | STRING  | YES       |

Notes:

* If `fromId` is not specified, the most recent trades are returned.

**Data Source:**
Database

**Response:**
```javascript
{
  "id": "cffc9c7d-4efc-4ce0-b587-6b87448f052a",
  "status": 200,
  "result": [
    {
      "id": 0,
      "price": "0.00005000",
      "qty": "40.00000000",
      "quoteQty": "0.00200000",
      "time": 1500004800376,
      "isBuyerMaker": true,
      "isBestMatch": true
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 5
    }
  ]
}
```

### Aggregate trades

```javascript
{
  "id": "189da436-d4bd-48ca-9f95-9f613d621717",
  "method": "trades.aggregate",
  "params": {
    "symbol": "BNBBTC",
    "fromId": 50000000,
    "limit": 1
  }
}
```

Get aggregate trades.

An *aggregate trade* (aggtrade) represents one or more individual trades.
Trades that fill at the same time, from the same taker order, with the same price –
those trades are collected into an aggregate trade with total quantity of the individual trades.

If you need access to real-time trading activity, please consider using WebSocket Streams:

* [`<symbol>@aggTrade`](web-socket-streams.md#aggregate-trade-streams)

If you need historical aggregate trade data,
please consider using [data.binance.vision](https://github.com/binance/binance-public-data/#aggtrades).

**Weight:**
1

**Parameters:**

Name        | Type    | Mandatory | Description
----------- | ------- | --------- | -----------
`symbol`    | STRING  | YES       |
`fromId`    | INT     | NO        | Aggregate trade ID to begin at
`startTime` | INT     | NO        |
`endTime`   | INT     | NO        |
`limit`     | INT     | NO        | Default 500; max 1000

Notes:

* If `fromId` is specified, return aggtrades with aggregate trade ID >= `fromId`.

  Use `fromId` and `limit` to page through all aggtrades.

* If `startTime` and/or `endTime` are specified, aggtrades are filtered by execution time (`T`).

  `fromId` cannot be used together with `startTime` and `endTime`.

* If no condition is specified, the most recent aggregate trades are returned.

**Data Source:**
Database

**Response:**
```javascript
{
  "id": "189da436-d4bd-48ca-9f95-9f613d621717",
  "status": 200,
  "result": [
    {
      "a": 50000000,        // Aggregate trade ID
      "p": "0.00274100",    // Price
      "q": "57.19000000",   // Quantity
      "f": 59120167,        // First trade ID
      "l": 59120170,        // Last trade ID
      "T": 1565877971222,   // Timestamp
      "m": true,            // Was the buyer the maker?
      "M": true             // Was the trade the best price match?
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### Klines

```javascript
{
  "id": "1dbbeb56-8eea-466a-8f6e-86bdcfa2fc0b",
  "method": "klines",
  "params": {
    "symbol": "BNBBTC",
    "interval": "1h",
    "startTime": 1655969280000,
    "limit": 1
  }
}
```

Get klines (candlestick bars).

Klines are uniquely identified by their open & close time.

If you need access to real-time kline updates, please consider using WebSocket Streams:

* [`<symbol>@kline_<interval>`](web-socket-streams.md#klinecandlestick-streams)

If you need historical kline data,
please consider using [data.binance.vision](https://github.com/binance/binance-public-data/#klines).

**Weight:**
1

**Parameters:**

Name        | Type    | Mandatory | Description
----------- | ------- | --------- | -----------
`symbol`    | STRING  | YES       |
`interval`  | ENUM    | YES       |
`startTime` | INT     | NO        |
`endTime`   | INT     | NO        |
`limit`     | INT     | NO        | Default 500; max 1000

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

Notes:

* If `startTime`, `endTime` are not specified, the most recent klines are returned.

**Data Source:**
Database

**Response:**
```javascript
{
  "id": "1dbbeb56-8eea-466a-8f6e-86bdcfa2fc0b",
  "status": 200,
  "result": [
    [
      1655971200000,      // Kline open time
      "0.01086000",       // Open price
      "0.01086600",       // High price
      "0.01083600",       // Low price
      "0.01083800",       // Close price
      "2290.53800000",    // Volume
      1655974799999,      // Kline close time
      "24.85074442",      // Quote asset volume
      2283,               // Number of trades
      "1171.64000000",    // Taker buy base asset volume
      "12.71225884",      // Taker buy quote asset volume
      "0"                 // Unused field, ignore
    ]
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### UI Klines

```javascript
{
  "id": "b137468a-fb20-4c06-bd6b-625148eec958",
  "method": "uiKlines",
  "params": {
    "symbol": "BNBBTC",
    "interval": "1h",
    "startTime": 1655969280000,
    "limit": 1
  }
}
```

Get klines (candlestick bars) optimized for presentation.

This request is similar to [`klines`](#klines), having the same parameters and response.
`uiKlines` return modified kline data, optimized for presentation of candlestick charts.

**Weight:**
1

**Parameters:**

Name        | Type    | Mandatory | Description
----------- | ------- | --------- | -----------
`symbol`    | STRING  | YES       |
`interval`  | ENUM    | YES       | See [`klines`](#kline-intervals)
`startTime` | INT     | NO        |
`endTime`   | INT     | NO        |
`limit`     | INT     | NO        | Default 500; max 1000

Notes:

* If `startTime`, `endTime` are not specified, the most recent klines are returned.

**Data Source:**
Database

**Response:**
```javascript
{
  "id": "b137468a-fb20-4c06-bd6b-625148eec958",
  "status": 200,
  "result": [
    [
      1655971200000,      // Kline open time
      "0.01086000",       // Open price
      "0.01086600",       // High price
      "0.01083600",       // Low price
      "0.01083800",       // Close price
      "2290.53800000",    // Volume
      1655974799999,      // Kline close time
      "24.85074442",      // Quote asset volume
      2283,               // Number of trades
      "1171.64000000",    // Taker buy base asset volume
      "12.71225884",      // Taker buy quote asset volume
      "0"                 // Unused field, ignore
    ]
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### Current average price

```javascript
{
  "id": "ddbfb65f-9ebf-42ec-8240-8f0f91de0867",
  "method": "avgPrice",
  "params": {
    "symbol": "BNBBTC"
  }
}
```

Get current average price for a symbol.

**Weight:**
1

**Parameters:**

Name        | Type    | Mandatory | Description
----------- | ------- | --------- | -----------
`symbol`    | STRING  | YES       |

**Data Source:**
Memory

**Response:**
```javascript
{
  "id": "ddbfb65f-9ebf-42ec-8240-8f0f91de0867",
  "status": 200,
  "result": {
    "mins": 5,              // Price averaging interval in minutes
    "price": "0.01378135"
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### 24hr ticker price change statistics

```javascript
{
  "id": "93fb61ef-89f8-4d6e-b022-4f035a3fadad",
  "method": "ticker.24hr",
  "params": {
    "symbol": "BNBBTC"
  }
}
```

Get 24-hour rolling window price change statistics.

If you need to continuously monitor trading statistics, please consider using WebSocket Streams:

* [`<symbol>@ticker`](web-socket-streams.md#individual-symbol-ticker-streams) or [`!ticker@arr`](web-socket-streams.md#all-market-tickers-stream)
* [`<symbol>@miniTicker`](web-socket-streams.md#individual-symbol-mini-ticker-stream) or [`!miniTicker@arr`](web-socket-streams.md#all-market-mini-tickers-stream)

If you need different window sizes,
use the [`ticker`](#rolling-window-price-change-statistics) request.

**Weight:**
Adjusted based on the number of requested symbols:

| Symbols     | Weight |
|:-----------:|:------:|
|        1–20 |      1 |
|      21–100 |     20 |
| 101 or more |     40 |
| all symbols |     40 |

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td rowspan="2" align="center">NO</td>
        <td>Query ticker for a single symbol</td>
    </tr>
    <tr>
        <td><code>symbols</code></td>
        <td>ARRAY of STRING</td>
        <td>Query ticker for multiple symbols</td>
    </tr>
    <tr>
        <td><code>type</code></td>
        <td>ENUM</td>
        <td align="center">NO</td>
        <td>Ticker type: <code>FULL</code> (default) or <code>MINI</code></td>
    </tr>
</tbody>
</table>

Notes:

* `symbol` and `symbols` cannot be used together.

* If no symbol is specified, returns information about all symbols currently trading on the exchange.

**Data Source:**
Memory

**Response:**

`FULL` type, for a single symbol:

```javascript
{
  "id": "93fb61ef-89f8-4d6e-b022-4f035a3fadad",
  "status": 200,
  "result": {
    "symbol": "BNBBTC",
    "priceChange": "0.00013900",
    "priceChangePercent": "1.020",
    "weightedAvgPrice": "0.01382453",
    "prevClosePrice": "0.01362800",
    "lastPrice": "0.01376700",
    "lastQty": "1.78800000",
    "bidPrice": "0.01376700",
    "bidQty": "4.64600000",
    "askPrice": "0.01376800",
    "askQty": "14.31400000",
    "openPrice": "0.01362800",
    "highPrice": "0.01414900",
    "lowPrice": "0.01346600",
    "volume": "69412.40500000",
    "quoteVolume": "959.59411487",
    "openTime": 1660014164909,
    "closeTime": 1660100564909,
    "firstId": 194696115,       // First trade ID
    "lastId": 194968287,        // Last trade ID
    "count": 272173             // Number of trades
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

`MINI` type, for a single symbol:

```javascript
{
  "id": "9fa2a91b-3fca-4ed7-a9ad-58e3b67483de",
  "status": 200,
  "result": {
    "symbol": "BNBBTC",
    "openPrice": "0.01362800",
    "highPrice": "0.01414900",
    "lowPrice": "0.01346600",
    "lastPrice": "0.01376700",
    "volume": "69412.40500000",
    "quoteVolume": "959.59411487",
    "openTime": 1660014164909,
    "closeTime": 1660100564909,
    "firstId": 194696115,       // First trade ID
    "lastId": 194968287,        // Last trade ID
    "count": 272173             // Number of trades
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

If more than one symbol is requested, response returns an array:

```javascript
{
  "id": "901be0d9-fd3b-45e4-acd6-10c580d03430",
  "status": 200,
  "result": [
    {
      "symbol": "BNBBTC",
      "priceChange": "0.00016500",
      "priceChangePercent": "1.213",
      "weightedAvgPrice": "0.01382508",
      "prevClosePrice": "0.01360800",
      "lastPrice": "0.01377200",
      "lastQty": "1.01400000",
      "bidPrice": "0.01377100",
      "bidQty": "7.55700000",
      "askPrice": "0.01377200",
      "askQty": "4.37900000",
      "openPrice": "0.01360700",
      "highPrice": "0.01414900",
      "lowPrice": "0.01346600",
      "volume": "69376.27900000",
      "quoteVolume": "959.13277091",
      "openTime": 1660014615517,
      "closeTime": 1660101015517,
      "firstId": 194697254,
      "lastId": 194969483,
      "count": 272230
    },
    {
      "symbol": "BTCUSDT",
      "priceChange": "-938.06000000",
      "priceChangePercent": "-3.938",
      "weightedAvgPrice": "23265.34432003",
      "prevClosePrice": "23819.17000000",
      "lastPrice": "22880.91000000",
      "lastQty": "0.00536000",
      "bidPrice": "22880.40000000",
      "bidQty": "0.00424000",
      "askPrice": "22880.91000000",
      "askQty": "0.04276000",
      "openPrice": "23818.97000000",
      "highPrice": "23933.25000000",
      "lowPrice": "22664.69000000",
      "volume": "153508.37606000",
      "quoteVolume": "3571425225.04441220",
      "openTime": 1660014615977,
      "closeTime": 1660101015977,
      "firstId": 1592019902,
      "lastId": 1597301762,
      "count": 5281861
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### Rolling window price change statistics

```javascript
{
  "id": "f4b3b507-c8f2-442a-81a6-b2f12daa030f",
  "method": "ticker",
  "params": {
    "symbols": [
      "BNBBTC",
      "BTCUSDT"
    ],
    "windowSize": "7d"
  }
}
```

Get rolling window price change statistics with a custom window.

This request is similar to [`ticker.24hr`](#24hr-ticker-price-change-statistics),
but statistics are computed on demand using the arbitrary window you specify.

**Note:** Window size precision is limited to 1 minute.
While the `closeTime` is the current time of the request, `openTime` always start on a minute boundary.
As such, the effective window might be up to 59999 ms wider than the requested `windowSize`.

<details>
<summary>Window computation example</summary>

For example, a request for `"windowSize": "7d"` might result in the following window:

```javascript
"openTime": 1659580020000,
"closeTime": 1660184865291,
```

Time of the request – `closeTime` – is 1660184865291 (August 11, 2022 02:27:45.291).
Requested window size should put the `openTime` 7 days before that – August 4, 02:27:45.291 –
but due to limited precision it ends up a bit earlier: 1659580020000 (August 4, 2022 02:27:00),
exactly at the start of a minute.

</details>

If you need to continuously monitor trading statistics, please consider using WebSocket Streams:

* [`<symbol>@ticker_<window_size>`](web-socket-streams.md#individual-symbol-rolling-window-statistics-streams) or [`!ticker_<window-size>@arr`](web-socket-streams.md#all-market-rolling-window-statistics-streams)

**Weight:**
Adjusted based on the number of requested symbols:

| Symbols | Weight |
|:-------:|:------:|
|    1–50 | 2 per symbol |
|  51–100 |    100 |

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td rowspan="2" align="center">YES</td>
        <td>Query ticker of a single symbol</td>
    </tr>
    <tr>
        <td><code>symbols</code></td>
        <td>ARRAY of STRING</td>
        <td>Query ticker for multiple symbols</td>
    </tr>
    <tr>
        <td><code>type</code></td>
        <td>ENUM</td>
        <td align="center">NO</td>
        <td>Ticker type: <code>FULL</code> (default) or <code>MINI</code></td>
    </tr>
    <tr>
        <td><code>windowSize</code></td>
        <td>ENUM</td>
        <td align="center">NO</td>
        <td>Default <code>1d</code></td>
    </tr>
</tbody>
</table>

Supported window sizes:

Unit    | `windowSize` value
------- | ------------------
minutes | `1m`, `2m` ... `59m`
hours   | `1h`, `2h` ... `23h`
days    | `1d`, `2d` ... `7d`

Notes:

* Either `symbol` or `symbols` must be specified.

* Maximum number of symbols in one request: 100.

* Window size units cannot be combined.
  E.g., <code>1d 2h</code> is not supported.

**Data Source:**
Database

**Response:**

`FULL` type, for a single symbol:

```javascript
{
  "id": "f4b3b507-c8f2-442a-81a6-b2f12daa030f",
  "status": 200,
  "result": {
    "symbol": "BNBBTC",
    "priceChange": "0.00061500",
    "priceChangePercent": "4.735",
    "weightedAvgPrice": "0.01368242",
    "openPrice": "0.01298900",
    "highPrice": "0.01418800",
    "lowPrice": "0.01296000",
    "lastPrice": "0.01360400",
    "volume": "587179.23900000",
    "quoteVolume": "8034.03382165",
    "openTime": 1659580020000,
    "closeTime": 1660184865291,
    "firstId": 192977765,       // First trade ID
    "lastId": 195365758,        // Last trade ID
    "count": 2387994            // Number of trades
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 2
    }
  ]
}
```

`MINI` type, for a single symbol:

```javascript
{
  "id": "bdb7c503-542c-495c-b797-4d2ee2e91173",
  "status": 200,
  "result": {
    "symbol": "BNBBTC",
    "openPrice": "0.01298900",
    "highPrice": "0.01418800",
    "lowPrice": "0.01296000",
    "lastPrice": "0.01360400",
    "volume": "587179.23900000",
    "quoteVolume": "8034.03382165",
    "openTime": 1659580020000,
    "closeTime": 1660184865291,
    "firstId": 192977765,       // First trade ID
    "lastId": 195365758,        // Last trade ID
    "count": 2387994            // Number of trades
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 2
    }
  ]
}
```

If more than one symbol is requested, response returns an array:

```javascript
{
  "id": "f4b3b507-c8f2-442a-81a6-b2f12daa030f",
  "status": 200,
  "result": [
    {
      "symbol": "BNBBTC",
      "priceChange": "0.00061500",
      "priceChangePercent": "4.735",
      "weightedAvgPrice": "0.01368242",
      "openPrice": "0.01298900",
      "highPrice": "0.01418800",
      "lowPrice": "0.01296000",
      "lastPrice": "0.01360400",
      "volume": "587169.48600000",
      "quoteVolume": "8033.90114517",
      "openTime": 1659580020000,
      "closeTime": 1660184820927,
      "firstId": 192977765,
      "lastId": 195365700,
      "count": 2387936
    },
    {
      "symbol": "BTCUSDT",
      "priceChange": "1182.92000000",
      "priceChangePercent": "5.113",
      "weightedAvgPrice": "23349.27074846",
      "openPrice": "23135.33000000",
      "highPrice": "24491.22000000",
      "lowPrice": "22400.00000000",
      "lastPrice": "24318.25000000",
      "volume": "1039498.10978000",
      "quoteVolume": "24271522807.76838630",
      "openTime": 1659580020000,
      "closeTime": 1660184820927,
      "firstId": 1568787779,
      "lastId": 1604337406,
      "count": 35549628
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 4
    }
  ]
}
```

### Symbol price ticker

```javascript
{
  "id": "043a7cf2-bde3-4888-9604-c8ac41fcba4d",
  "method": "ticker.price",
  "params": {
    "symbol": "BNBBTC"
  }
}
```

Get the latest market price for a symbol.

If you need access to real-time price updates, please consider using WebSocket Streams:

* [`<symbol>@aggTrade`](web-socket-streams.md#aggregate-trade-streams)
* [`<symbol>@trade`](web-socket-streams.md#trade-streams)

**Weight:**
Adjusted based on the number of requested symbols:

| Parameter | Weight |
| --------- |:------:|
| `symbol`  |      1 |
| `symbols` |      2 |
| none      |      2 |

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td rowspan="2" align="center">NO</td>
        <td>Query price for a single symbol</td>
    </tr>
    <tr>
        <td><code>symbols</code></td>
        <td>ARRAY of STRING</td>
        <td>Query price for multiple symbols</td>
    </tr>
</tbody>
</table>

Notes:

* `symbol` and `symbols` cannot be used together.

* If no symbol is specified, returns information about all symbols currently trading on the exchange.

**Data Source:**
Memory

**Response:**

```javascript
{
  "id": "043a7cf2-bde3-4888-9604-c8ac41fcba4d",
  "status": 200,
  "result": {
    "symbol": "BNBBTC",
    "price": "0.01361900"
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

If more than one symbol is requested, response returns an array:

```javascript
{
  "id": "e739e673-24c8-4adf-9cfa-b81f30330b09",
  "status": 200,
  "result": [
    {
      "symbol": "BNBBTC",
      "price": "0.01363700"
    },
    {
      "symbol": "BTCUSDT",
      "price": "24267.15000000"
    },
    {
      "symbol": "BNBBUSD",
      "price": "331.10000000"
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 2
    }
  ]
}
```

### Symbol order book ticker

```javascript
{
  "id": "057deb3a-2990-41d1-b58b-98ea0f09e1b4",
  "method": "ticker.book",
  "params": {
    "symbols": [
      "BNBBTC",
      "BTCUSDT"
    ]
  }
}
```

Get the current best price and quantity on the order book.

If you need access to real-time order book ticker updates, please consider using WebSocket Streams:

* [`<symbol>@bookTicker`](web-socket-streams.md#individual-symbol-book-ticker-streams)

**Weight:**
Adjusted based on the number of requested symbols:

| Parameter | Weight |
| --------- |:------:|
| `symbol`  |      1 |
| `symbols` |      2 |
| none      |      2 |

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td rowspan="2" align="center">NO</td>
        <td>Query ticker for a single symbol</td>
    </tr>
    <tr>
        <td><code>symbols</code></td>
        <td>ARRAY of STRING</td>
        <td>Query ticker for multiple symbols</td>
    </tr>
</tbody>
</table>

Notes:

* `symbol` and `symbols` cannot be used together.

* If no symbol is specified, returns information about all symbols currently trading on the exchange.

**Data Source:**
Memory

**Response:**

```javascript
{
  "id": "9d32157c-a556-4d27-9866-66760a174b57",
  "status": 200,
  "result": {
    "symbol": "BNBBTC",
    "bidPrice": "0.01358000",
    "bidQty": "12.53400000",
    "askPrice": "0.01358100",
    "askQty": "17.83700000"
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

If more than one symbol is requested, response returns an array:

```javascript
{
  "id": "057deb3a-2990-41d1-b58b-98ea0f09e1b4",
  "status": 200,
  "result": [
    {
      "symbol": "BNBBTC",
      "bidPrice": "0.01358000",
      "bidQty": "12.53400000",
      "askPrice": "0.01358100",
      "askQty": "17.83700000"
    },
    {
      "symbol": "BTCUSDT",
      "bidPrice": "23980.49000000",
      "bidQty": "0.01000000",
      "askPrice": "23981.31000000",
      "askQty": "0.01512000"
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 2
    }
  ]
}
```

## Trading requests

### Place new order (TRADE)

```javascript
{
  "id": "56374a46-3061-486b-a311-99ee972eb648",
  "method": "order.place",
  "params": {
    "symbol": "BTCUSDT",
    "side": "SELL",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "price": "23416.10000000",
    "quantity": "0.00847000",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
    "timestamp": 1660801715431
  }
}
```

Send in a new order.

**Weight:**
1

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`symbol`            | STRING  | YES       |
`side`              | ENUM    | YES       | `BUY` or `SELL`
`type`              | ENUM    | YES       |
`timeInForce`       | ENUM    | NO *      |
`price`             | DECIMAL | NO *      |
`quantity`          | DECIMAL | NO *      |
`quoteOrderQty`     | DECIMAL | NO *      |
`newClientOrderId`  | STRING  | NO        | Arbitrary unique ID among open orders. Automatically generated if not sent
`newOrderRespType`  | ENUM    | NO        | <p>Select response format: `ACK`, `RESULT`, `FULL`.</p><p>`MARKET` and `LIMIT` orders use `FULL` by default, other order types default to `ACK`.</p>
`stopPrice`         | DECIMAL | NO *      |
`trailingDelta`     | INT     | NO *      | See [Trailing Stop order FAQ](faqs/trailing-stop-faq.md)
`icebergQty`        | DECIMAL | NO        |
`strategyId`        | INT     | NO        | Arbitrary numeric value identifying the order within an order strategy.
`strategyType`      | INT     | NO        | <p>Arbitrary numeric value identifying the order strategy.</p><p>Values smaller than `1000000` are reserved and cannot be used.</p>
`selfTradePreventionMode` |ENUM | NO      | The allowed enums is dependent on what is configured on the symbol. The possible supported values are `EXPIRE_TAKER`, `EXPIRE_MAKER`, `EXPIRE_BOTH`, `NONE`.
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | The value cannot be greater than `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

<a id="order-type">Certain parameters (*)</a> become mandatory based on the order `type`:

<table>
<thead>
    <tr>
        <th>Order <code>type</code></th>
        <th>Mandatory parameters</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td><code>LIMIT</code></td>
        <td>
        <ul>
            <li><code>timeInForce</code></li>
            <li><code>price</code></li>
            <li><code>quantity</code></li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>LIMIT_MAKER</code></td>
        <td>
        <ul>
            <li><code>price</code></li>
            <li><code>quantity</code></li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>MARKET</code></td>
        <td>
        <ul>
            <li><code>quantity</code> or <code>quoteOrderQty</code></li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>STOP_LOSS</code></td>
        <td>
        <ul>
            <li><code>quantity</code></li>
            <li><code>stopPrice</code> or <code>trailingDelta</code></li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>STOP_LOSS_LIMIT</code></td>
        <td>
        <ul>
            <li><code>timeInForce</code></li>
            <li><code>price</code></li>
            <li><code>quantity</code></li>
            <li><code>stopPrice</code> or <code>trailingDelta</code></li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>TAKE_PROFIT</code></td>
        <td>
        <ul>
            <li><code>quantity</code></li>
            <li><code>stopPrice</code> or <code>trailingDelta</code></li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>TAKE_PROFIT_LIMIT</code></td>
        <td>
        <ul>
            <li><code>timeInForce</code></li>
            <li><code>price</code></li>
            <li><code>quantity</code></li>
            <li><code>stopPrice</code> or <code>trailingDelta</code></li>
        </ul>
        </td>
    </tr>
</tbody>
</table>

Supported order types:

<table>
<thead>
    <tr>
        <th>Order <code>type</code></th>
        <th>Description</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td><code>LIMIT</code></td>
        <td>
        <p>
            Buy or sell <code>quantity</code> at the specified <code>price</code> or better.
        </p>
        </td>
    </tr>
    <tr>
        <td><code>LIMIT_MAKER</code></td>
        <td>
        <p>
            <code>LIMIT</code> order that will be rejected if it immediately matches and trades as a taker.
        </p>
        <p>
            This order type is also known as a POST-ONLY order.
        </p>
        </td>
    </tr>
    <tr>
        <td><code>MARKET</code></td>
        <td>
        <p>
            Buy or sell at the best available market price.
        </p>
        <ul>
            <li>
                <p>
                    <code>MARKET</code> order with <code>quantity</code> parameter
                    specifies the amount of the <em>base asset</em> you want to buy or sell.
                    Actually executed quantity of the quote asset will be determined by available market liquidity.
                </p>
                <p>
                    E.g., a MARKET BUY order on BTCUSDT for <code>"quantity": "0.1000"</code>
                    specifies that you want to buy 0.1 BTC at the best available price.
                    If there is not enough BTC at the best price, keep buying at the next best price,
                    until either your order is filled, or you run out of USDT, or market runs out of BTC.
                </p>
            </li>
            <li>
                <p>
                    <code>MARKET</code> order with <code>quoteOrderQty</code> parameter
                    specifies the amount of the <em>quote asset</em> you want to spend (when buying) or receive (when selling).
                    Actually executed quantity of the base asset will be determined by available market liquidity.
                </p>
                <p>
                    E.g., a MARKET BUY on BTCUSDT for <code>"quoteOrderQty": "100.00"</code>
                    specifies that you want to buy as much BTC as you can for 100 USDT at the best available price.
                    Similarly, a SELL order will sell as much available BTC as needed for you to receive 100 USDT
                    (before commission).
                </p>
            </li>
        </ul>
        </td>
    </tr>
    <tr>
        <td><code>STOP_LOSS</code></td>
        <td>
        <p>
            Execute a <code>MARKET</code> order for given <code>quantity</code> when specified conditions are met.
        </p>
        <p>
            I.e., when <code>stopPrice</code> is reached, or when <code>trailingDelta</code> is activated.
        </p>
        </td>
    </tr>
    <tr>
        <td><code>STOP_LOSS_LIMIT</code></td>
        <td>
        <p>
            Place a <code>LIMIT</code> order with given parameters when specified conditions are met.
        </p>
        </td>
    </tr>
    <tr>
        <td><code>TAKE_PROFIT</code></td>
        <td>
        <p>
            Like <code>STOP_LOSS</code> but activates when market price moves in the favorable direction.
        </p>
        </td>
    </tr>
    <tr>
        <td><code>TAKE_PROFIT_LIMIT</code></td>
        <td>
        <p>
            Like <code>STOP_LOSS_LIMIT</code> but activates when market price moves in the favorable direction.
        </p>
        </td>
    </tr>
</tbody>
</table>

<a id="timeInForce"></a>

Available `timeInForce` options,
setting how long the order should be active before expiration:

 TIF  | Description
----- | --------------
`GTC` | **Good 'til Canceled** – the order will remain on the book until you cancel it, or the order is completely filled.
`IOC` | **Immediate or Cancel** – the order will be filled for as much as possible, the unfilled quantity immediately expires.
`FOK` | **Fill or Kill** – the order will expire unless it cannot be immediately filled for the entire quantity.

Notes:

* `newClientOrderId` specifies `clientOrderId` value for the order.

  A new order with the same `clientOrderId` is accepted only when the previous one is filled or expired.

* Any `LIMIT` or `LIMIT_MAKER` order can be made into an iceberg order by specifying the `icebergQty`.

  An order with an `icebergQty` must have `timeInForce` set to `GTC`.

* Trigger order price rules for `STOP_LOSS`/`TAKE_PROFIT` orders:

  * `stopPrice` must be above market price: `STOP_LOSS BUY`, `TAKE_PROFIT SELL`
  * `stopPrice` must be below market price: `STOP_LOSS SELL`, `TAKE_PROFIT BUY`

* `MARKET` orders using `quoteOrderQty` follow [`LOT_SIZE`](filters.md#lot_size) filter rules.

  The order will execute a quantity that has notional value as close as possible to requested `quoteOrderQty`.

**Data Source:**
Matching Engine

**Response:**

Response format is selected by using the `newOrderRespType` parameter.

`ACK` response type:

```javascript
{
  "id": "56374a46-3061-486b-a311-99ee972eb648",
  "status": 200,
  "result": {
    "symbol": "BTCUSDT",
    "orderId": 12569099453,
    "orderListId": -1, // always -1 for singular orders
    "clientOrderId": "4d96324ff9d44481926157ec08158a40",
    "transactTime": 1660801715639
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 1
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 1
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

`RESULT` response type:

```javascript
{
  "id": "56374a46-3061-486b-a311-99ee972eb648",
  "status": 200,
  "result": {
    "symbol": "BTCUSDT",
    "orderId": 12569099453,
    "orderListId": -1, // always -1 for singular orders
    "clientOrderId": "4d96324ff9d44481926157ec08158a40",
    "transactTime": 1660801715639,
    "price": "23416.10000000",
    "origQty": "0.00847000",
    "executedQty": "0.00000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL",
    "workingTime": 1660801715639,
    "selfTradePreventionMode": "NONE"
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 1
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 1
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

`FULL` response type:

```javascript
{
  "id": "56374a46-3061-486b-a311-99ee972eb648",
  "status": 200,
  "result": {
    "symbol": "BTCUSDT",
    "orderId": 12569099453,
    "orderListId": -1,
    "clientOrderId": "4d96324ff9d44481926157ec08158a40",
    "transactTime": 1660801715793,
    "price": "23416.10000000",
    "origQty": "0.00847000",
    "executedQty": "0.00847000",
    "cummulativeQuoteQty": "198.33521500",
    "status": "FILLED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL",
    "workingTime": 1660801715793,
    // FULL response is identical to RESULT response, with the same optional fields
    // based on the order type and parameters. FULL response additionally includes
    // the list of trades which immediately filled the order.
    "fills": [
      {
        "price": "23416.10000000",
        "qty": "0.00635000",
        "commission": "0.000000",
        "commissionAsset": "BNB",
        "tradeId": 1650422481
      },
      {
        "price": "23416.50000000",
        "qty": "0.00212000",
        "commission": "0.000000",
        "commissionAsset": "BNB",
        "tradeId": 1650422482
      }
    ]
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 1
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 1
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
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

```javascript
{
  "id": "6ffebe91-01d9-43ac-be99-57cf062e0e30",
  "method": "order.test",
  "params": {
    "symbol": "BTCUSDT",
    "side": "SELL",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "price": "23416.10000000",
    "quantity": "0.00847000",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "15af09e41c36f3cc61378c2fbe2c33719a03dd5eba8d0f9206fbda44de717c88",
    "timestamp": 1660801715431
  }
}
```

Test order placement.

Validates new order parameters and verifies your signature
but does not send the order into the matching engine.

**Weight:**
1

**Parameters:**

Same as for [`order.place`](#place-new-order-trade).

**Data Source:**
Memory

**Response:**
```javascript
{
  "id": "6ffebe91-01d9-43ac-be99-57cf062e0e30",
  "status": 200,
  "result": {},
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### Query order (USER_DATA)

```javascript
{
  "id": "aa62318a-5a97-4f3b-bdc7-640bbe33b291",
  "method": "order.status",
  "params": {
    "symbol": "BTCUSDT",
    "orderId": 12569099453,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "2c3aab5a078ee4ea465ecd95523b77289f61476c2f238ec10c55ea6cb11a6f35",
    "timestamp": 1660801720951
  }
}
```

Check execution status of an order.

**Weight:**
2

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>orderId</code></td>
        <td>INT</td>
        <td rowspan="2">YES</td>
        <td>Lookup order by <code>orderId</code></td>
    </tr>
    <tr>
        <td><code>origClientOrderId</code></td>
        <td>STRING</td>
        <td>Lookup order by <code>clientOrderId</code></td>
    </tr>
    <tr>
        <td><code>apiKey</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>recvWindow</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>The value cannot be greater than <tt>60000</tt></td>
    </tr>
    <tr>
        <td><code>signature</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>timestamp</code></td>
        <td>INT</td>
        <td>YES</td>
        <td></td>
    </tr>
</tbody>
</table>

Notes:

* If both `orderId` and `origClientOrderId` parameters are specified,
  only `orderId` is used and `origClientOrderId` is ignored.

* For some historical orders the `cummulativeQuoteQty` response field may be negative,
  meaning the data is not available at this time.

**Data Source:**
Memory => Database

**Response:**
```javascript
{
  "id": "aa62318a-5a97-4f3b-bdc7-640bbe33b291",
  "status": 200,
  "result": {
    "symbol": "BTCUSDT",
    "orderId": 12569099453,
    "orderListId": -1,                  // set only for legs of an OCO
    "clientOrderId": "4d96324ff9d44481926157",
    "price": "23416.10000000",
    "origQty": "0.00847000",
    "executedQty": "0.00847000",
    "cummulativeQuoteQty": "198.33521500",
    "status": "FILLED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL",
    "stopPrice": "0.00000000",          // always present, zero if order type does not use stopPrice
    "trailingDelta": 10,                // present only if trailingDelta set for the order
    "trailingTime": -1,                 // present only if trailingDelta set for the order
    "icebergQty": "0.00000000",         // always present, zero for non-iceberg orders
    "time": 1660801715639,              // time when the order was placed
    "updateTime": 1660801717945,        // time of the last update to the order
    "isWorking": true,
    "workingTime": 1660801715639,
    "origQuoteOrderQty": "0.00000000"   // always present, zero if order type does not use quoteOrderQty
    "strategyId": 37463720,             // present only if strategyId set for the order
    "strategyType": 1000000,            // present only if strategyType set for the order
    "selfTradePreventionMode": "NONE",
    "preventedMatchId": 0,              // present only if the order expired due to STP
    "preventedQuantity": "1.200000"     // present only if the order expired due to STP
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 2
    }
  ]
}
```

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

### Cancel order (TRADE)

```javascript
{
  "id": "5633b6a2-90a9-4192-83e7-925c90b6a2fd",
  "method": "order.cancel",
  "params": {
    "symbol": "BTCUSDT",
    "origClientOrderId": "4d96324ff9d44481926157",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "33d5b721f278ae17a52f004a82a6f68a70c68e7dd6776ed0be77a455ab855282",
    "timestamp": 1660801715830
  }
}
```

Cancel an active order.

**Weight:**
1

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>orderId</code></td>
        <td>INT</td>
        <td rowspan="2">YES</td>
        <td>Cancel order by <code>orderId</code></td>
    </tr>
    <tr>
        <td><code>origClientOrderId</code></td>
        <td>STRING</td>
        <td>Cancel order by <code>clientOrderId</code></td>
    </tr>
    <tr>
        <td><code>newClientOrderId</code></td>
        <td>STRING</td>
        <td>NO</td>
        <td>New ID for the canceled order. Automatically generated if not sent</td>
    </tr>
    <tr>
      <td><code>cancelRestrictions</code></td>
      <td>ENUM</td>
      <td>NO</td>
      <td>Supported values: <br><code>ONLY_NEW</code> - Cancel will succeed if the order status is <code>NEW</code>.<br> <code>ONLY_PARTIALLY_FILLED</code> - Cancel will succeed if order status is <code>PARTIALLY_FILLED</code>.</td>
    </tr>
    <tr>
        <td><code>apiKey</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>recvWindow</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>The value cannot be greater than <tt>60000</tt></td>
    </tr>
    <tr>
        <td><code>signature</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>timestamp</code></td>
        <td>INT</td>
        <td>YES</td>
        <td></td>
    </tr>
</tbody>
</table>

Notes:

* If both `orderId` and `origClientOrderId` parameters are specified,
  only `orderId` is used and `origClientOrderId` is ignored.

* `newClientOrderId` will replace `clientOrderId` of the canceled order, freeing it up for new orders.

* If you cancel an order that is a part of an OCO pair, the entire OCO is canceled.

**Data Source:**
Matching Engine

**Response:**

When an individual order is canceled:

```javascript
{
  "id": "5633b6a2-90a9-4192-83e7-925c90b6a2fd",
  "status": 200,
  "result": {
    "symbol": "BTCUSDT",
    "origClientOrderId": "4d96324ff9d44481926157",  // clientOrderId that was canceled
    "orderId": 12569099453,
    "orderListId": -1,                              // set only for legs of an OCO
    "clientOrderId": "91fe37ce9e69c90d6358c0",      // newClientOrderId from request
    "price": "23416.10000000",
    "origQty": "0.00847000",
    "executedQty": "0.00001000",
    "cummulativeQuoteQty": "0.23416100",
    "status": "CANCELED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL",
    "stopPrice": "0.00000000",          // present only if stopPrice set for the order
    "trailingDelta": 0,                 // present only if trailingDelta set for the order
    "icebergQty": "0.00000000",         // present only if icebergQty set for the order
    "strategyId": 37463720,             // present only if strategyId set for the order
    "strategyType": 1000000,            // present only if strategyType set for the order
    "selfTradePreventionMode": "NONE"
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

When an OCO is canceled:

```javascript
{
  "id": "16eaf097-bbec-44b9-96ff-e97e6e875870",
  "status": 200,
  "result": {
    "orderListId": 19431,
    "contingencyType": "OCO",
    "listStatusType": "ALL_DONE",
    "listOrderStatus": "ALL_DONE",
    "listClientOrderId": "iuVNVJYYrByz6C4yGOPPK0",
    "transactionTime": 1660803702431,
    "symbol": "BTCUSDT",
    "orders": [
      {
        "symbol": "BTCUSDT",
        "orderId": 12569099453,
        "clientOrderId": "bX5wROblo6YeDwa9iTLeyY"
      },
      {
        "symbol": "BTCUSDT",
        "orderId": 12569099454,
        "clientOrderId": "Tnu2IP0J5Y4mxw3IATBfmW"
      }
    ],
    // OCO leg status format is the same as for individual orders.
    "orderReports": [
      {
        "symbol": "BTCUSDT",
        "origClientOrderId": "bX5wROblo6YeDwa9iTLeyY",
        "orderId": 12569099453,
        "orderListId": 19431,
        "clientOrderId": "OFFXQtxVFZ6Nbcg4PgE2DA",
        "price": "23450.50000000",
        "origQty": "0.00850000"
        "executedQty": "0.00000000",
        "cummulativeQuoteQty": "0.00000000",
        "status": "CANCELED",
        "timeInForce": "GTC",
        "type": "STOP_LOSS_LIMIT",
        "side": "BUY",
        "stopPrice": "23430.00000000",
        "selfTradePreventionMode": "NONE"
      },
      {
        "symbol": "BTCUSDT",
        "origClientOrderId": "Tnu2IP0J5Y4mxw3IATBfmW",
        "orderId": 12569099454,
        "orderListId": 19431,
        "clientOrderId": "OFFXQtxVFZ6Nbcg4PgE2DA",
        "price": "23400.00000000",
        "origQty": "0.00850000"
        "executedQty": "0.00000000",
        "cummulativeQuoteQty": "0.00000000",
        "status": "CANCELED",
        "timeInForce": "GTC",
        "type": "LIMIT_MAKER",
        "side": "BUY",
        "selfTradePreventionMode": "NONE"
      }
    ]
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

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

### Cancel and replace order (TRADE)

```javascript
{
  "id": "99de1036-b5e2-4e0f-9b5c-13d751c93a1a",
  "method": "order.cancelReplace",
  "params": {
    "symbol": "BTCUSDT",
    "cancelReplaceMode": "ALLOW_FAILURE",
    "cancelOrigClientOrderId": "4d96324ff9d44481926157",
    "side": "SELL",
    "type": "LIMIT",
    "timeInForce": "GTC",
    "price": "23416.10000000",
    "quantity": "0.00847000",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "7028fdc187868754d25e42c37ccfa5ba2bab1d180ad55d4c3a7e2de643943dc5",
    "timestamp": 1660813156900
  }
}
```

Cancel an existing order and immediately place a new order instead of the canceled one.

**Weight:**
1

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>cancelReplaceMode</code></td>
        <td>ENUM</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>cancelOrderId</code></td>
        <td>INT</td>
        <td rowspan="2">YES</td>
        <td>Cancel order by <code>orderId</code></td>
    </tr>
    <tr>
        <td><code>cancelOrigClientOrderId</code></td>
        <td>STRING</td>
        <td>Cancel order by <code>clientOrderId</code></td>
    </tr>
    <tr>
        <td><code>cancelNewClientOrderId</code></td>
        <td>STRING</td>
        <td>NO</td>
        <td>New ID for the canceled order. Automatically generated if not sent</td>
    </tr>
    <tr>
        <td><code>side</code></td>
        <td>ENUM</td>
        <td>YES</td>
        <td><code>BUY</code> or <code>SELL</code></td>
    </tr>
    <tr>
        <td><code>type</code></td>
        <td>ENUM</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>timeInForce</code></td>
        <td>ENUM</td>
        <td>NO *</td>
        <td></td>
    </tr>
    <tr>
        <td><code>price</code></td>
        <td>DECIMAL</td>
        <td>NO *</td>
        <td></td>
    </tr>
    <tr>
        <td><code>quantity</code></td>
        <td>DECIMAL</td>
        <td>NO *</td>
        <td></td>
    </tr>
    <tr>
        <td><code>quoteOrderQty</code></td>
        <td>DECIMAL</td>
        <td>NO *</td>
        <td></td>
    </tr>
    <tr>
        <td><code>newClientOrderId</code></td>
        <td>STRING</td>
        <td>NO</td>
        <td>Arbitrary unique ID among open orders. Automatically generated if not sent</td>
    </tr>
    <tr>
        <td><code>newOrderRespType</code></td>
        <td>ENUM</td>
        <td>NO</td>
        <td>
            <p>Select response format: <code>ACK</code>, <code>RESULT</code>, <code>FULL</code>.</p>
            <p>
                <code>MARKET</code> and <code>LIMIT</code> orders produce <code>FULL</code> response by default,
                other order types default to <code>ACK</code>.
            </p>
        </td>
    </tr>
    <tr>
        <td><code>stopPrice</code></td>
        <td>DECIMAL</td>
        <td>NO *</td>
        <td></td>
    </tr>
    <tr>
        <td><code>trailingDelta</code></td>
        <td>DECIMAL</td>
        <td>NO *</td>
        <td>See <a href="faqs/trailing-stop-faq.md">Trailing Stop order FAQ</a></td>
    </tr>
    <tr>
        <td><code>icebergQty</code></td>
        <td>DECIMAL</td>
        <td>NO</td>
        <td></td>
    </tr>
    <tr>
        <td><code>strategyId</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>Arbitrary numeric value identifying the order within an order strategy.</td>
    </tr>
    <tr>
        <td><code>strategyType</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>
            <p>Arbitrary numeric value identifying the order strategy.</p>
            <p>Values smaller than <tt>1000000</tt> are reserved and cannot be used.</p>
        </td>
    </tr>
    <tr>
        <td><code>selfTradePreventionMode</code></td>
        <td>ENUM</td>
        <td>NO</td>
        <td>
            <p>The allowed enums is dependent on what is configured on the symbol.</p>
            <p>The possible supported values are <tt>EXPIRE_TAKER</tt>, <tt>EXPIRE_MAKER</tt>, <tt>EXPIRE_BOTH</tt>, <tt>NONE</tt>.</p>
        </td>
    </tr>
    <tr>
      <td><code>cancelRestrictions</code></td>
      <td>ENUM</td>
      <td>NO</td>
      <td>Supported values: <br><code>ONLY_NEW</code> - Cancel will succeed if the order status is <code>NEW</code>.<br> <code>ONLY_PARTIALLY_FILLED</code> - Cancel will succeed if order status is <code>PARTIALLY_FILLED</code>. For more information please refer to <a href="#regarding-cancelrestrictions">Regarding <code>cancelRestrictions</code></a>.</td>
    </tr>
    <tr>
        <td><code>apiKey</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>recvWindow</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>The value cannot be greater than <tt>60000</tt></td>
    </tr>
    <tr>
        <td><code>signature</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>timestamp</code></td>
        <td>INT</td>
        <td>YES</td>
        <td></td>
    </tr>
</tbody>
</table>

Similar to the [`order.place`](#place-new-order-trade) request,
additional mandatory parameters (*) are determined by the new order [`type`](#order-type).

Available `cancelReplaceMode` options:

* `STOP_ON_FAILURE` – if cancellation request fails, new order placement will not be attempted
* `ALLOW_FAILURE` – new order placement will be attempted even if the cancel request fails

<table>
<thead>
    <tr>
        <th>Request</th>
        <th colspan=3>Response</th>
    </tr>
    <tr>
        <th><code>cancelReplaceMode</code></th>
        <th><code>cancelResult</code></th>
        <th><code>newOrderResult</code></th>
        <th><code>status</code></th>
    </tr>
</thead>
<tbody>
    <tr>
        <td rowspan="3"><code>STOP_ON_FAILURE</code></td>
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
        <td rowspan="4"><code>ALLOW_FAILURE</code></td>
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
</tbody>
</table>

Notes:

* If both `cancelOrderId` and `cancelOrigClientOrderId` parameters are specified,
  only `cancelOrderId` is used and `cancelOrigClientOrderId` is ignored.

* `cancelNewClientOrderId` will replace `clientOrderId` of the canceled order, freeing it up for new orders.

* `newClientOrderId` specifies `clientOrderId` value for the placed order.

  A new order with the same `clientOrderId` is accepted only when the previous one is filled or expired.

  The new order can reuse old `clientOrderId` of the canceled order.

* This cancel-replace operation is **not transactional**.

  If one operation succeeds but the other one fails, the successful operation is still executed.

  For example, in `STOP_ON_FAILURE` mode, if the new order placement fails, the old order is still canceled.

* Filters and order count limits are evaluated before cancellation and order placement occurs.

* If new order placement is not attempted, your order count is still incremented.

* Like [`order.cancel`](#cancel-order-trade), if you cancel a leg of an OCO, the entire OCO is canceled.

**Data Source:**
Matching Engine

**Response:**

If both cancel and placement succeed, you get the following response with `"status": 200`:

```javascript
{
  "id": "99de1036-b5e2-4e0f-9b5c-13d751c93a1a",
  "status": 200,
  "result": {
    "cancelResult": "SUCCESS",
    "newOrderResult": "SUCCESS",
    // Format is identical to "order.cancel" format.
    // Some fields are optional and are included only for orders that set them.
    "cancelResponse": {
      "symbol": "BTCUSDT",
      "origClientOrderId": "4d96324ff9d44481926157",  // cancelOrigClientOrderId from request
      "orderId": 125690984230,
      "orderListId": -1,
      "clientOrderId": "91fe37ce9e69c90d6358c0",      // cancelNewClientOrderId from request
      "price": "23450.00000000",
      "origQty": "0.00847000",
      "executedQty": "0.00001000",
      "cummulativeQuoteQty": "0.23450000",
      "status": "CANCELED",
      "timeInForce": "GTC",
      "type": "LIMIT",
      "side": "SELL",
      "selfTradePreventionMode": "NONE"
    },
    // Format is identical to "order.place" format, affected by "newOrderRespType".
    // Some fields are optional and are included only for orders that set them.
    "newOrderResponse": {
      "symbol": "BTCUSDT",
      "orderId": 12569099453,
      "orderListId": -1,
      "clientOrderId": "bX5wROblo6YeDwa9iTLeyY",      // newClientOrderId from request
      "transactTime": 1660813156959,
      "price": "23416.10000000",
      "origQty": "0.00847000",
      "executedQty": "0.00000000",
      "cummulativeQuoteQty": "0.00000000",
      "status": "NEW",
      "timeInForce": "GTC",
      "type": "LIMIT",
      "side": "SELL",
      "selfTradePreventionMode": "NONE"
    }
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 1
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 1
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

In `STOP_ON_FAILURE` mode, failed order cancellation prevents new order from being placed
and returns the following response with `"status": 400`:

```javascript
{
  "id": "27e1bf9f-0539-4fb0-85c6-06183d36f66c",
  "status": 400,
  "error": {
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
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 1
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 1
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

If cancel-replace mode allows failure and one of the operations fails,
you get a response with `"status": 409`,
and the `"data"` field detailing which operation succeeded, which failed, and why:

```javascript
{
  "id": "b220edfe-f3c4-4a3a-9d13-b35473783a25",
  "status": 409,
  "error": {
    "code": -2021,
    "msg": "Order cancel-replace partially failed.",
    "data": {
      "cancelResult": "SUCCESS",
      "newOrderResult": "FAILURE",
      "cancelResponse": {
        "symbol": "BTCUSDT",
        "origClientOrderId": "4d96324ff9d44481926157",
        "orderId": 125690984230,
        "orderListId": -1,
        "clientOrderId": "91fe37ce9e69c90d6358c0",
        "price": "23450.00000000",
        "origQty": "0.00847000",
        "executedQty": "0.00001000",
        "cummulativeQuoteQty": "0.23450000",
        "status": "CANCELED",
        "timeInForce": "GTC",
        "type": "LIMIT",
        "side": "SELL",
        "selfTradePreventionMode": "NONE"
      },
      "newOrderResponse": {
        "code": -2010,
        "msg": "Order would immediately match and take."
      }
    }
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 1
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 1
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

```javascript
{
  "id": "ce641763-ff74-41ac-b9f7-db7cbe5e93b1",
  "status": 409,
  "error": {
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
        "orderId": 12569099453,
        "orderListId": -1,
        "clientOrderId": "bX5wROblo6YeDwa9iTLeyY",
        "transactTime": 1660813156959,
        "price": "23416.10000000",
        "origQty": "0.00847000",
        "executedQty": "0.00000000",
        "cummulativeQuoteQty": "0.00000000",
        "status": "NEW",
        "timeInForce": "GTC",
        "type": "LIMIT",
        "side": "SELL",
        "workingTime": 1669693344508,
        "fills": [],
        "selfTradePreventionMode": "NONE"
      }
    }
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 1
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 1
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

If both operations fail, response will have `"status": 400`:

```javascript
{
  "id": "3b3ac45c-1002-4c7d-88e8-630c408ecd87",
  "status": 400,
  "error": {
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
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 1
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 1
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

### Current open orders (USER_DATA)

```javascript
{
  "id": "55f07876-4f6f-4c47-87dc-43e5fff3f2e7",
  "method": "openOrders.status",
  "params": {
    "symbol": "BTCUSDT",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "d632b3fdb8a81dd44f82c7c901833309dd714fe508772a89b0a35b0ee0c48b89",
    "timestamp": 1660813156812
  }
}
```

Query execution status of all open orders.

If you need to continuously monitor order status updates, please consider using WebSocket Streams:

* [`userDataStream.start`](#user-data-stream-requests) request
* [`executionReport`](./user-data-stream.md#order-update) user data stream event

**Weight:**
Adjusted based on the number of requested symbols:

| Parameter | Weight |
| --------- | ------ |
| `symbol`  |      3 |
| none      |     40 |

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`symbol`            | STRING  | NO        | If omitted, open orders for all symbols are returned
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | The value cannot be greater than `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

**Data Source:**
Memory => Database

**Response:**

Status reports for open orders are identical to [`order.status`](#query-order-user_data).

Note that some fields are optional and included only for orders that set them.

Open orders are always returned as a flat list.
If all symbols are requested, use the `symbol` field to tell which symbol the orders belong to.

```javascript
{
  "id": "55f07876-4f6f-4c47-87dc-43e5fff3f2e7",
  "status": 200,
  "result": [
    {
      "symbol": "BTCUSDT",
      "orderId": 12569099453,
      "orderListId": -1,
      "clientOrderId": "4d96324ff9d44481926157",
      "price": "23416.10000000",
      "origQty": "0.00847000",
      "executedQty": "0.00720000",
      "cummulativeQuoteQty": "172.43931000",
      "status": "PARTIALLY_FILLED",
      "timeInForce": "GTC",
      "type": "LIMIT",
      "side": "SELL",
      "stopPrice": "0.00000000",
      "icebergQty": "0.00000000",
      "time": 1660801715639,
      "updateTime": 1660801717945,
      "isWorking": true,
      "workingTime": 1660801715639,
      "origQuoteOrderQty": "0.00000000",
      "selfTradePreventionMode": "NONE"
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 3
    }
  ]
}
```

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

### Cancel open orders (TRADE)

```javascript
{
  "id": "778f938f-9041-4b88-9914-efbf64eeacc8",
  "method": "openOrders.cancelAll"
  "params": {
    "symbol": "BTCUSDT",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "773f01b6e3c2c9e0c1d217bc043ce383c1ddd6f0e25f8d6070f2b66a6ceaf3a5",
    "timestamp": 1660805557200
  }
}
```

Cancel all open orders on a symbol,
including OCO orders.

**Weight:**
1

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`symbol`            | STRING  | YES       |
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | The value cannot be greater than `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

**Data Source:**
Matching Engine

**Response:**

Cancellation reports for orders and OCOs have the same format as in [`order.cancel`](#cancel-order-trade).

```javascript
{
  "id": "778f938f-9041-4b88-9914-efbf64eeacc8",
  "status": 200,
  "result": [
    {
      "symbol": "BTCUSDT",
      "origClientOrderId": "4d96324ff9d44481926157",
      "orderId": 12569099453,
      "orderListId": -1,
      "clientOrderId": "91fe37ce9e69c90d6358c0",
      "price": "23416.10000000",
      "origQty": "0.00847000",
      "executedQty": "0.00001000",
      "cummulativeQuoteQty": "0.23416100",
      "status": "CANCELED",
      "timeInForce": "GTC",
      "type": "LIMIT",
      "side": "SELL",
      "stopPrice": "0.00000000",
      "trailingDelta": 0,
      "trailingTime": -1,
      "icebergQty": "0.00000000",
      "strategyId": 37463720,
      "strategyType": 1000000,
      "selfTradePreventionMode": "NONE"
    },
    {
      "orderListId": 19431,
      "contingencyType": "OCO",
      "listStatusType": "ALL_DONE",
      "listOrderStatus": "ALL_DONE",
      "listClientOrderId": "iuVNVJYYrByz6C4yGOPPK0",
      "transactionTime": 1660803702431,
      "symbol": "BTCUSDT",
      "orders": [
        {
          "symbol": "BTCUSDT",
          "orderId": 12569099453,
          "clientOrderId": "bX5wROblo6YeDwa9iTLeyY"
        },
        {
          "symbol": "BTCUSDT",
          "orderId": 12569099454,
          "clientOrderId": "Tnu2IP0J5Y4mxw3IATBfmW"
        }
      ],
      "orderReports": [
        {
          "symbol": "BTCUSDT",
          "origClientOrderId": "bX5wROblo6YeDwa9iTLeyY",
          "orderId": 12569099453,
          "orderListId": 19431,
          "clientOrderId": "OFFXQtxVFZ6Nbcg4PgE2DA",
          "price": "23450.50000000",
          "origQty": "0.00850000",
          "executedQty": "0.00000000",
          "cummulativeQuoteQty": "0.00000000",
          "status": "CANCELED",
          "timeInForce": "GTC",
          "type": "STOP_LOSS_LIMIT",
          "side": "BUY",
          "stopPrice": "23430.00000000",
          "selfTradePreventionMode": "NONE"
        },
        {
          "symbol": "BTCUSDT",
          "origClientOrderId": "Tnu2IP0J5Y4mxw3IATBfmW",
          "orderId": 12569099454,
          "orderListId": 19431,
          "clientOrderId": "OFFXQtxVFZ6Nbcg4PgE2DA",
          "price": "23400.00000000",
          "origQty": "0.00850000",
          "executedQty": "0.00000000",
          "cummulativeQuoteQty": "0.00000000",
          "status": "CANCELED",
          "timeInForce": "GTC",
          "type": "LIMIT_MAKER",
          "side": "BUY",
          "selfTradePreventionMode": "NONE"
        }
      ]
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

**Note:** The payload above does not show all fields that can appear. Please refer to [Conditional fields in Order Responses](#conditional-fields-in-order-responses).

### Place new OCO (TRADE)

```javascript
{
  "id": "56374a46-3061-486b-a311-99ee972eb648",
  "method": "orderList.place",
  "params": {
    "symbol": "BTCUSDT",
    "side": "SELL",
    "price": "23420.00000000",
    "quantity": "0.00650000",
    "stopPrice": "23410.00000000",
    "stopLimitPrice": "23405.00000000",
    "stopLimitTimeInForce": "GTC",
    "newOrderRespType": "RESULT",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "6689c2a36a639ff3915c2904871709990ab65f3c7a9ff13857558fd350315c35",
    "timestamp": 1660801713767
  }
}
```

Send in a new one-cancels-the-other (OCO) pair:
`LIMIT_MAKER` + `STOP_LOSS`/`STOP_LOSS_LIMIT` orders (called *legs*),
where activation of one order immediately cancels the other.

**Weight:**
1

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`symbol`            | STRING  | YES       |
`side`              | ENUM    | YES       | `BUY` or `SELL`
`price`             | DECIMAL | YES       | Price for the limit order
`quantity`          | DECIMAL | YES       |
`listClientOrderId` | STRING  | NO        | Arbitrary unique ID among open OCOs. Automatically generated if not sent
`limitClientOrderId`| STRING  | NO        | Arbitrary unique ID among open orders for the limit order. Automatically generated if not sent
`limitIcebergQty`   | DECIMAL | NO        |
`limitStrategyId`   | INT     | NO        | Arbitrary numeric value identifying the limit order within an order strategy.
`limitStrategyType` | INT     | NO        | <p>Arbitrary numeric value identifying the limit order strategy.</p><p>Values smaller than `1000000` are reserved and cannot be used.</p>
`stopPrice`         | DECIMAL | YES *     | Either `stopPrice` or `trailingDelta`, or both must be specified
`trailingDelta`     | INT     | YES *     | See [Trailing Stop order FAQ](faqs/trailing-stop-faq.md)
`stopClientOrderId` | STRING  | NO        | Arbitrary unique ID among open orders for the stop order. Automatically generated if not sent
`stopLimitPrice`    | DECIMAL | NO *      |
`stopLimitTimeInForce` | ENUM | NO *      | See [`order.place`](#timeInForce) for available options
`stopIcebergQty`    | DECIMAL | NO *      |
`stopStrategyId`    | INT     | NO        | Arbitrary numeric value identifying the stop order within an order strategy.
`stopStrategyType`  | INT     | NO        | <p>Arbitrary numeric value identifying the stop order strategy.</p><p>Values smaller than `1000000` are reserved and cannot be used.</p>
`newOrderRespType`  | ENUM    | NO        | Select response format: `ACK`, `RESULT`, `FULL` (default)
`selfTradePreventionMode` |ENUM | NO      | The allowed enums is dependent on what is configured on the symbol. The possible supported values are `EXPIRE_TAKER`, `EXPIRE_MAKER`, `EXPIRE_BOTH`, `NONE`.
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | The value cannot be greater than `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

Notes:

* `listClientOrderId` parameter specifies `listClientOrderId` for the OCO pair.

  A new OCO with the same `listClientOrderId` is accepted only when the previous one is filled or completely expired.

  `listClientOrderId` is distinct from `clientOrderId` of individual orders.

* `limitClientOrderId` and `stopClientOrderId` specify `clientOrderId` values for both legs of the OCO.

  A new order with the same `clientOrderId` is accepted only when the previous one is filled or expired.

* Price restrictions on the legs:

  | `side` | Price relation |
  | ------ | -------------- |
  | `BUY`  | `price` < market price < `stopPrice` |
  | `SELL` | `price` > market price > `stopPrice` |

* Both legs have the same `quantity`.

  However, you can set different iceberg quantity for individual legs.

  If `stopIcebergQty` is used, `stopLimitTimeInForce` must be `GTC`.

* `trailingDelta` applies only to the `STOP_LOSS`/`STOP_LOSS_LIMIT` leg of the OCO.

* OCO counts as 2 orders against the order rate limit.

**Data Source:**
Matching Engine

**Response:**

Response format for `orderReports` is selected using the `newOrderRespType` parameter.
The following example is for `RESULT` response type.
See [`order.place`](#place-new-order-trade) for more examples.

```javascript
{
  "id": "57833dc0-e3f2-43fb-ba20-46480973b0aa",
  "status": 200,
  "result": {
    "orderListId": 1274512,
    "contingencyType": "OCO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "08985fedd9ea2cf6b28996",
    "transactionTime": 1660801713793,
    "symbol": "BTCUSDT",
    "orders": [
      {
        "symbol": "BTCUSDT",
        "orderId": 12569138901,
        "clientOrderId": "BqtFCj5odMoWtSqGk2X9tU"
      },
      {
        "symbol": "BTCUSDT",
        "orderId": 12569138902,
        "clientOrderId": "jLnZpj5enfMXTuhKB1d0us"
      }
    ],
    "orderReports": [
      {
        "symbol": "BTCUSDT",
        "orderId": 12569138901,
        "orderListId": 1274512,
        "clientOrderId": "BqtFCj5odMoWtSqGk2X9tU",
        "transactTime": 1660801713793,
        "price": "23410.00000000",
        "origQty": "0.00650000",
        "executedQty": "0.00000000",
        "cummulativeQuoteQty": "0.00000000",
        "status": "NEW",
        "timeInForce": "GTC",
        "type": "STOP_LOSS_LIMIT",
        "side": "SELL",
        "stopPrice": "23405.00000000",
        "workingTime": -1,
        "selfTradePreventionMode": "NONE"
      },
      {
        "symbol": "BTCUSDT",
        "orderId": 12569138902,
        "orderListId": 1274512,
        "clientOrderId": "jLnZpj5enfMXTuhKB1d0us",
        "transactTime": 1660801713793,
        "price": "23420.00000000",
        "origQty": "0.00650000",
        "executedQty": "0.00000000",
        "cummulativeQuoteQty": "0.00000000",
        "status": "NEW",
        "timeInForce": "GTC",
        "type": "LIMIT_MAKER",
        "side": "SELL",
        "workingTime": 1660801713793,
        "selfTradePreventionMode": "NONE"
      }
    ]
  },
  "rateLimits": [
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 10,
      "limit": 50,
      "count": 2
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "intervalNum": 1,
      "limit": 160000,
      "count": 2
    },
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### Query OCO (USER_DATA)

```javascript
{
  "id": "b53fd5ff-82c7-4a04-bd64-5f9dc42c2100",
  "method": "orderList.status",
  "params": {
    "origClientOrderId": "08985fedd9ea2cf6b28996"
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "d12f4e8892d46c0ddfbd43d556ff6d818581b3be22a02810c2c20cb719aed6a4",
    "timestamp": 1660801713965
  }
}
```

Check execution status of an OCO.

For execution status of individual orders, use [`order.status`](#query-order-user_data).

**Weight:**
2

**Parameters**:

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
        <td><code>origClientOrderId</code></td>
        <td>STRING</td>
        <td rowspan="2">YES</td>
        <td>Query OCO by <code>listClientOrderId</code></td>
    </tr>
    <tr>
        <td><code>orderListId</code></td>
        <td>INT</td>
        <td>Query OCO by <code>orderListId</code></td>
    </tr>
    <tr>
        <td><code>apiKey</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>recvWindow</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>The value cannot be greater than <tt>60000</tt></td>
    </tr>
    <tr>
        <td><code>signature</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>timestamp</code></td>
        <td>INT</td>
        <td>YES</td>
        <td></td>
    </tr>
</tbody>
</table>

Notes:

* `origClientOrderId` refers to `listClientOrderId` of the OCO itself.

* If both `origClientOrderId` and `orderListId` parameters are specified,
  only `origClientOrderId` is used and `orderListId` is ignored.

**Data Source:**
Database

**Response:**

```javascript
{
  "id": "b53fd5ff-82c7-4a04-bd64-5f9dc42c2100",
  "status": 200,
  "result": {
    "orderListId": 1274512,
    "contingencyType": "OCO",
    "listStatusType": "EXEC_STARTED",
    "listOrderStatus": "EXECUTING",
    "listClientOrderId": "08985fedd9ea2cf6b28996",
    "transactionTime": 1660801713793,
    "symbol": "BTCUSDT",
    "orders": [
      {
        "symbol": "BTCUSDT",
        "orderId": 12569138901,
        "clientOrderId": "BqtFCj5odMoWtSqGk2X9tU"
      },
      {
        "symbol": "BTCUSDT",
        "orderId": 12569138902,
        "clientOrderId": "jLnZpj5enfMXTuhKB1d0us"
      }
    ]
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 2
    }
  ]
}
```

### Cancel OCO (TRADE)

```javascript
{
  "id": "c5899911-d3f4-47ae-8835-97da553d27d0",
  "method": "orderList.cancel",
  "params": {
    "symbol": "BTCUSDT",
    "orderListId": 1274512,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "4973f4b2fee30bf6d45e4a973e941cc60fdd53c8dd5a25edeac96f5733c0ccee",
    "timestamp": 1660801720210
  }
}
```

Cancel an active OCO.

**Weight**:
1

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
        <td><code>symbol</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>orderListId</code></td>
        <td>INT</td>
        <td rowspan="2">YES</td>
        <td>Cancel OCO by <code>orderListId</code></td>
    </tr>
    <tr>
        <td><code>listClientOrderId</code></td>
        <td>STRING</td>
        <td>Cancel OCO by <code>listClientId</code></td>
    </tr>
    <tr>
        <td><code>newClientOrderId</code></td>
        <td>STRING</td>
        <td>NO</td>
        <td>New ID for the canceled OCO. Automatically generated if not sent</td>
    </tr>
    <tr>
        <td><code>apiKey</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>recvWindow</code></td>
        <td>INT</td>
        <td>NO</td>
        <td>The value cannot be greater than <tt>60000</tt></td>
    </tr>
    <tr>
        <td><code>signature</code></td>
        <td>STRING</td>
        <td>YES</td>
        <td></td>
    </tr>
    <tr>
        <td><code>timestamp</code></td>
        <td>INT</td>
        <td>YES</td>
        <td></td>
    </tr>
</tbody>
</table>

Notes:

* If both `orderListId` and `listClientOrderId` parameters are specified,
  only `orderListId` is used and `listClientOrderId` is ignored.

* Canceling an individual leg with [`order.cancel`](#cancel-order-trade) will cancel the entire OCO as well.

**Data Source:**
Matching Engine

**Response:**

```javascript
{
  "id": "c5899911-d3f4-47ae-8835-97da553d27d0",
  "status": 200,
  "result": {
    "orderListId": 1274512,
    "contingencyType": "OCO",
    "listStatusType": "ALL_DONE",
    "listOrderStatus": "ALL_DONE",
    "listClientOrderId": "6023531d7edaad348f5aff",
    "transactionTime": 1660801720215,
    "symbol": "BTCUSDT",
    "orders": [
      {
        "symbol": "BTCUSDT",
        "orderId": 12569138901,
        "clientOrderId": "BqtFCj5odMoWtSqGk2X9tU"
      },
      {
        "symbol": "BTCUSDT",
        "orderId": 12569138902,
        "clientOrderId": "jLnZpj5enfMXTuhKB1d0us"
      }
    ],
    "orderReports": [
      {
        "symbol": "BTCUSDT",
        "orderId": 12569138901,
        "orderListId": 1274512,
        "clientOrderId": "BqtFCj5odMoWtSqGk2X9tU",
        "transactTime": 1660801720215,
        "price": "23410.00000000",
        "origQty": "0.00650000",
        "executedQty": "0.00000000",
        "cummulativeQuoteQty": "0.00000000",
        "status": "CANCELED",
        "timeInForce": "GTC",
        "type": "STOP_LOSS_LIMIT",
        "side": "SELL",
        "stopPrice": "23405.00000000",
        "selfTradePreventionMode": "NONE"
      },
      {
        "symbol": "BTCUSDT",
        "orderId": 12569138902,
        "orderListId": 1274512,
        "clientOrderId": "jLnZpj5enfMXTuhKB1d0us",
        "transactTime": 1660801720215,
        "price": "23420.00000000",
        "origQty": "0.00650000",
        "executedQty": "0.00000000",
        "cummulativeQuoteQty": "0.00000000",
        "status": "CANCELED",
        "timeInForce": "GTC",
        "type": "LIMIT_MAKER",
        "side": "SELL",
        "selfTradePreventionMode": "NONE"
      }
    ]
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### Current open OCOs (USER_DATA)

```javascript
{
  "id": "3a4437e2-41a3-4c19-897c-9cadc5dce8b6",
  "method": "openOrderLists.status",
  "params": {
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "1bea8b157dd78c3da30359bddcd999e4049749fe50b828e620e12f64e8b433c9",
    "timestamp": 1660801713831
  }
}
```

Query execution status of all open OCOs.

If you need to continuously monitor order status updates, please consider using WebSocket Streams:

* [`userDataStream.start`](#user-data-stream-requests) request
* [`executionReport`](./user-data-stream.md#order-update) user data stream event

**Weight**:
3

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | The value cannot be greater than `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

**Data Source:**
Database

**Response:**

```javascript
{
  "id": "3a4437e2-41a3-4c19-897c-9cadc5dce8b6",
  "status": 200,
  "result": [
    {
      "orderListId": 0,
      "contingencyType": "OCO",
      "listStatusType": "EXEC_STARTED",
      "listOrderStatus": "EXECUTING",
      "listClientOrderId": "08985fedd9ea2cf6b28996",
      "transactionTime": 1660801713793,
      "symbol": "BTCUSDT",
      "orders": [
        {
          "symbol": "BTCUSDT",
          "orderId": 4,
          "clientOrderId": "CUhLgTXnX5n2c0gWiLpV4d"
        },
        {
          "symbol": "BTCUSDT",
          "orderId": 5,
          "clientOrderId": "1ZqG7bBuYwaF4SU8CwnwHm"
        }
      ]
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 3
    }
  ]
}
```

## Account requests

### Account information (USER_DATA)

```javascript
{
  "id": "605a6d20-6588-4cb9-afa0-b0ab087507ba",
  "method": "account.status",
  "params": {
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "83303b4a136ac1371795f465808367242685a9e3a42b22edb4d977d0696eb45c",
    "timestamp": 1660801839480
  }
}
```

Query information about your account.

**Weight:**
10

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | The value cannot be greater than `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

**Data Source:**
Memory => Database

**Response:**
```javascript
{
  "id": "605a6d20-6588-4cb9-afa0-b0ab087507ba",
  "status": 200,
  "result": {
    "makerCommission": 15,
    "takerCommission": 15,
    "buyerCommission": 0,
    "sellerCommission": 0,
    "canTrade": true,
    "canWithdraw": true,
    "canDeposit": true,
    "commissionRates": {
      "maker": "0.00150000",
      "taker": "0.00150000",
      "buyer": "0.00000000",
      "seller": "0.00000000"
    },
    "brokered": false,
    "requireSelfTradePrevention": false,
    "updateTime": 1660801833000,
    "accountType": "SPOT",
    "balances": [
      {
        "asset": "BNB",
        "free": "0.00000000",
        "locked": "0.00000000"
      },
      {
        "asset": "BTC",
        "free": "1.3447112",
        "locked": "0.08600000"
      },
      {
        "asset": "USDT",
        "free": "1021.21000000",
        "locked": "0.00000000"
      }
    ],
    "permissions": [
      "SPOT"
    ]
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 10
    }
  ]
}
```

### Account order rate limits (USER_DATA)

```javascript
{
  "id": "d3783d8d-f8d1-4d2c-b8a0-b7596af5a664",
  "method": "account.rateLimits.orders",
  "params": {
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "76289424d6e288f4dc47d167ac824e859dabf78736f4348abbbac848d719eb94",
    "timestamp": 1660801839500
  }
}
```

Query your current order rate limit.

**Weight:**
20

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | The value cannot be greater than `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

**Data Source:**
Memory

**Response:**

```javascript
{
  "id": "d3783d8d-f8d1-4d2c-b8a0-b7596af5a664",
  "status": 200,
  "result": [
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
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 20
    }
  ]
}
```

### Account order history (USER_DATA)

```javascript
{
  "id": "734235c2-13d2-4574-be68-723e818c08f3",
  "method": "allOrders",
  "params": {
    "symbol": "BTCUSDT",
    "startTime": 1660780800000,
    "endTime": 1660867200000,
    "limit": 5,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "f50a972ba7fad92842187643f6b930802d4e20bce1ba1e788e856e811577bd42",
    "timestamp": 1661955123341
  }
}
```

Query information about all your orders – active, canceled, filled – filtered by time range.

**Weight:**
10

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`symbol`            | STRING  | YES       |
`orderId`           | INT     | NO        | Order ID to begin at
`startTime`         | INT     | NO        |
`endTime`           | INT     | NO        |
`limit`             | INT     | NO        | Default 500; max 1000
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | The value cannot be greater than `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

Notes:

* If `startTime` and/or `endTime` are specified, `orderId` is ignored.

  Orders are filtered by `time` of the last execution status update.

* If `orderId` is specified, return orders with order ID >= `orderId`.

* If no condition is specified, the most recent orders are returned.

* For some historical orders the `cummulativeQuoteQty` response field may be negative,
  meaning the data is not available at this time.

**Data Source:**
Database

**Response:**

Status reports for orders are identical to [`order.status`](#query-order-user_data).

Note that some fields are optional and included only for orders that set them.

```javascript
{
  "id": "734235c2-13d2-4574-be68-723e818c08f3",
  "status": 200,
  "result": [
    {
      "symbol": "BTCUSDT",
      "orderId": 12569099453,
      "orderListId": -1,
      "clientOrderId": "4d96324ff9d44481926157",
      "price": "23416.10000000",
      "origQty": "0.00847000",
      "executedQty": "0.00847000",
      "cummulativeQuoteQty": "198.33521500",
      "status": "FILLED",
      "timeInForce": "GTC",
      "type": "LIMIT",
      "side": "SELL",
      "stopPrice": "0.00000000",
      "icebergQty": "0.00000000",
      "time": 1660801715639,
      "updateTime": 1660801717945,
      "isWorking": true,
      "workingTime": 1660801715639,
      "origQuoteOrderQty": "0.00000000",
      "selfTradePreventionMode": "NONE",
      "preventedMatchId": 0,            // This field only appears if the order expired due to STP.
      "preventedQuantity": "1.200000"   // This field only appears if the order expired due to STP.
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 10
    }
  ]
}
```

### Account OCO history (USER_DATA)

```javascript
{
  "id": "8617b7b3-1b3d-4dec-94cd-eefd929b8ceb",
  "method": "allOrderLists",
  "params": {
    "startTime": 1660780800000,
    "endTime": 1660867200000,
    "limit": 5,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "c8e1484db4a4a02d0e84dfa627eb9b8298f07ebf12fcc4eaf86e4a565b2712c2",
    "timestamp": 1661955123341
  }
}
```

Query information about all your OCOs, filtered by time range.

**Weight:**
10

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`fromId`            | INT     | NO        | Order list ID to begin at
`startTime`         | INT     | NO        |
`endTime`           | INT     | NO        |
`limit`             | INT     | NO        | Default 500; max 1000
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | The value cannot be greater than `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

Notes:

* If `startTime` and/or `endTime` are specified, `fromId` is ignored.

  OCOs are filtered by `transactionTime` of the last OCO execution status update.

* If `fromId` is specified, return OCOs with order list ID >= `fromId`.

* If no condition is specified, the most recent OCOs are returned.

**Data Source:**
Database

**Response:**

Status reports for OCOs are identical to [`orderList.status`](#query-oco-user_data).

```javascript
{
  "id": "8617b7b3-1b3d-4dec-94cd-eefd929b8ceb",
  "status": 200,
  "result": [
    {
      "orderListId": 1274512,
      "contingencyType": "OCO",
      "listStatusType": "EXEC_STARTED",
      "listOrderStatus": "EXECUTING",
      "listClientOrderId": "08985fedd9ea2cf6b28996",
      "transactionTime": 1660801713793,
      "symbol": "BTCUSDT",
      "orders": [
        {
          "symbol": "BTCUSDT",
          "orderId": 12569138901,
          "clientOrderId": "BqtFCj5odMoWtSqGk2X9tU"
        },
        {
          "symbol": "BTCUSDT",
          "orderId": 12569138902,
          "clientOrderId": "jLnZpj5enfMXTuhKB1d0us"
        }
      ]
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 10
    }
  ]
}
```

### Account trade history (USER_DATA)

```javascript
{
  "id": "f4ce6a53-a29d-4f70-823b-4ab59391d6e8",
  "method": "myTrades",
  "params": {
    "symbol": "BTCUSDT",
    "startTime": 1660780800000,
    "endTime": 1660867200000,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "c5a5ffb79fd4f2e10a92f895d488943a57954edf5933bde3338dfb6ea6d6eefc",
    "timestamp": 1661955125250
  }
}
```

Query information about all your trades, filtered by time range.

**Weight:**
10

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`symbol`            | STRING  | YES       |
`orderId`           | INT     | NO        |
`startTime`         | INT     | NO        |
`endTime`           | INT     | NO        |
`fromId`            | INT     | NO        | First trade ID to query
`limit`             | INT     | NO        | Default 500; max 1000
`apiKey`            | STRING  | YES       |
`recvWindow`        | INT     | NO        | The value cannot be greater than `60000`
`signature`         | STRING  | YES       |
`timestamp`         | INT     | YES       |

Notes:

* If `fromId` is specified, return trades with trade ID >= `fromId`.

* If `startTime` and/or `endTime` are specified, trades are filtered by execution time (`time`).

  `fromId` cannot be used together with `startTime` and `endTime`.

* If `orderId` is specified, only trades related to that order are returned.

  `startTime` and `endTime` cannot be used together with `orderId`.

* If no condition is specified, the most recent trades are returned.

**Data Source:**
Memory => Database

**Response:**

```javascript
{
  "id": "f4ce6a53-a29d-4f70-823b-4ab59391d6e8",
  "status": 200,
  "result": [
    {
      "symbol": "BTCUSDT",
      "id": 1650422481,
      "orderId": 12569099453,
      "orderListId": -1,
      "price": "23416.10000000",
      "qty": "0.00635000",
      "quoteQty": "148.69223500",
      "commission": "0.00000000",
      "commissionAsset": "BNB",
      "time": 1660801715793,
      "isBuyer": false,
      "isMaker": true,
      "isBestMatch": true
    },
    {
      "symbol": "BTCUSDT",
      "id": 1650422482,
      "orderId": 12569099453,
      "orderListId": -1,
      "price": "23416.50000000",
      "qty": "0.00212000",
      "quoteQty": "49.64298000",
      "commission": "0.00000000",
      "commissionAsset": "BNB",
      "time": 1660801715793,
      "isBuyer": false,
      "isMaker": true,
      "isBestMatch": true
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 10
    }
  ]
}
```


### Account prevented matches (USER_DATA)

```javascript
{
  "id": "g4ce6a53-a39d-4f71-823b-4ab5r391d6y8",
  "method": "myPreventedMatches",
  "params": {
    "symbol": "BTCUSDT",
    "orderId": 35,
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A",
    "signature": "c5a5ffb79fd4f2e10a92f895d488943a57954edf5933bde3338dfb6ea6d6eefc",
    "timestamp": 1673923281052
  }
}
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

```javascript
{
  "id": "g4ce6a53-a39d-4f71-823b-4ab5r391d6y8",
  "status": 200,
  "result": [
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
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 10
    }
  ]
}
```

## User Data Stream requests

The following requests manage [User Data Stream](user-data-stream.md) subscriptions.

**Note:** The user data can ONLY be retrieved by *a separate* Websocket connection via the **User Data Streams** url (i.e. `wss://stream.binance.com:443`).

### Start user data stream (USER_STREAM)

```javascript
{
  "id": "d3df8a61-98ea-4fe0-8f4e-0fcea5d418b0",
  "method": "userDataStream.start",
  "params": {
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A"
  }
}
```

Start a new user data stream.

**Note:** the stream will close in 60 minutes
unless [`userDataStream.ping`](#ping-user-data-stream-user_stream) requests are sent regularly.

**Weight:**
1

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`apiKey`            | STRING  | YES       |


**Data Source:**
Memory

**Response:**

Subscribe to the received listen key on WebSocket Stream afterwards.


```javascript
{
  "id": "d3df8a61-98ea-4fe0-8f4e-0fcea5d418b0",
  "status": 200,
  "result": {
    "listenKey": "xs0mRXdAKlIPDRFrlPcw0qI41Eh3ixNntmymGyhrhgqo7L6FuLaWArTD7RLP"
  },
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### Ping user data stream (USER_STREAM)

```javascript
{
  "id": "815d5fce-0880-4287-a567-80badf004c74",
  "method": "userDataStream.ping",
  "params": {
    "listenKey": "xs0mRXdAKlIPDRFrlPcw0qI41Eh3ixNntmymGyhrhgqo7L6FuLaWArTD7RLP",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A"
  }
}
```

Ping a user data stream to keep it alive.

User data streams close automatically after 60 minutes,
even if you're listening to them on WebSocket Streams.
In order to keep the stream open, you have to regularly send pings using the `userDataStream.ping` request.

It is recommended to send a ping once every 30 minutes.

**Weight:**
1

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`listenKey`         | STRING  | YES       |
`apiKey`            | STRING  | YES       |

**Data Source:**
Memory

**Response:**

```javascript
{
  "id": "815d5fce-0880-4287-a567-80badf004c74",
  "status": 200,
  "response": {},
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```

### Stop user data stream (USER_STREAM)

```javascript
{
  "id": "819e1b1b-8c06-485b-a13e-131326c69599",
  "method": "userDataStream.stop",
  "params": {
    "listenKey": "xs0mRXdAKlIPDRFrlPcw0qI41Eh3ixNntmymGyhrhgqo7L6FuLaWArTD7RLP",
    "apiKey": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A"
  }
}
```

Explicitly stop and close the user data stream.

**Weight:**
1

**Parameters:**

Name                | Type    | Mandatory | Description
------------------- | ------- | --------- | ------------
`listenKey`         | STRING  | YES       |
`apiKey`            | STRING  | YES       |

**Data Source:**
Memory

**Response:**
```javascript
{
  "id": "819e1b1b-8c06-485b-a13e-131326c69599",
  "status": 200,
  "response": {},
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 1200,
      "count": 1
    }
  ]
}
```
