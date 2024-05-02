<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [General WSS information](#general-wss-information)
- [API Endpoints](#api-endpoints)
  - [Create a listenKey (USER_STREAM)](#create-a-listenkey-user_stream)
  - [Ping/Keep-alive a listenKey (USER_STREAM)](#pingkeep-alive-a-listenkey-user_stream)
  - [Close a listenKey (USER_STREAM)](#close-a-listenkey-user_stream)
- [Web Socket Payloads](#web-socket-payloads)
  - [Account Update](#account-update)
  - [Balance Update](#balance-update)
  - [Order Update](#order-update)
    - [Conditional Fields in Execution Report](#conditional-fields-in-execution-report)
    - [Execution types](#execution-types)
  - [Listen Key Expired](#listen-key-expired)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# User Data Streams for Binance Spot TESTNET(2024-03-13)
# General WSS information
* The base API endpoint is: **https://testnet.binance.vision/**
* A User Data Stream `listenKey` is valid for 60 minutes after creation.
* Doing a `PUT` on an active `listenKey` will extend its validity for 60 minutes.
* Doing a `DELETE` on an active `listenKey` will close the stream and invalidate the `listenKey`.
* Doing a `POST` on an account with an active `listenKey` will return the currently active `listenKey` and extend its validity for 60 minutes.
* The base websocket endpoint is: **wss://stream.binance.com:9443**
* User Data Streams are accessed at **/ws/\<listenKey\>** or **/stream?streams=\<listenKey\>**
* A single connection to **stream.binance.com** is only valid for 24 hours; expect to be disconnected at the 24 hour mark

# API Endpoints
## Create a listenKey (USER_STREAM)
```
POST /api/v3/userDataStream
```
Start a new user data stream. The stream will close after 60 minutes unless a keepalive is sent. If the account has an active `listenKey`, that `listenKey` will be returned and its validity will be extended for 60 minutes.

**Weight:**
1

**Parameters:**
NONE

**Response:**
```javascript
{
  "listenKey": "pqia91ma19a5s61cv6a81va65sdf19v8a65a1a5s61cv6a81va65sdf19v8a65a1"
}
```

## Ping/Keep-alive a listenKey (USER_STREAM)
```
PUT /api/v3/userDataStream
```
Keepalive a user data stream to prevent a time out. User data streams will close after 60 minutes. It's recommended to send a ping about every 30 minutes.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES

**Response:**
```javascript
{}
```

## Close a listenKey (USER_STREAM)
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

**Response:**
```javascript
{}
```

# Web Socket Payloads
## Account Update

`outboundAccountPosition` is sent any time an account balance has changed and contains the assets that were possibly changed by the event that generated the balance change.

```javascript
{
  "e": "outboundAccountPosition", // Event type
  "E": 1564034571105,             // Event Time
  "u": 1564034571073,             // Time of last account update
  "B": [                          // Balances Array
    {
      "a": "ETH",                 // Asset
      "f": "10000.000000",        // Free
      "l": "0.000000"             // Locked
    }
  ]
}
```

## Balance Update

Balance Update occurs during the following:
* Deposits or withdrawals from the account
* Transfer of funds between accounts (e.g. Spot to Margin)

**Payload**
```javascript
{
  "e": "balanceUpdate",         // Event Type
  "E": 1573200697110,           // Event Time
  "a": "BTC",                   // Asset
  "d": "100.00000000",          // Balance Delta
  "T": 1573200697068            // Clear Time
}
```

## Order Update
Orders are updated with the `executionReport` event.

**Payload:**
```javascript
{
  "e": "executionReport",        // Event type
  "E": 1499405658658,            // Event time
  "s": "ETHBTC",                 // Symbol
  "c": "mUvoqJxFIILMdfAW5iGSOW", // Client order ID
  "S": "BUY",                    // Side
  "o": "LIMIT",                  // Order type
  "f": "GTC",                    // Time in force
  "q": "1.00000000",             // Order quantity
  "p": "0.10264410",             // Order price
  "P": "0.00000000",             // Stop price
  "F": "0.00000000",             // Iceberg quantity
  "g": -1,                       // OrderListId
  "C": "",                       // Original client order ID; This is the ID of the order being canceled
  "x": "NEW",                    // Current execution type
  "X": "NEW",                    // Current order status
  "r": "NONE",                   // Order reject reason; will be an error code.
  "i": 4293153,                  // Order ID
  "l": "0.00000000",             // Last executed quantity
  "z": "0.00000000",             // Cumulative filled quantity
  "L": "0.00000000",             // Last executed price
  "n": "0",                      // Commission amount
  "N": null,                     // Commission asset
  "T": 1499405658657,            // Transaction time
  "t": -1,                       // Trade ID
  "v": 3,                        // Prevented Match Id; This is only visible if the order expired due to STP
  "I": 8641984,                  // Ignore
  "w": true,                     // Is the order on the book?
  "m": false,                    // Is this trade the maker side?
  "M": false,                    // Ignore
  "O": 1499405658657,            // Order creation time
  "Z": "0.00000000",             // Cumulative quote asset transacted quantity
  "Y": "0.00000000",             // Last quote asset transacted quantity (i.e. lastPrice * lastQty)
  "Q": "0.00000000",             // Quote Order Quantity
  "W": 1499405658657,            // Working Time; This is only visible if the order has been placed on the book.
  "V": "NONE"                    // SelfTradePreventionMode
}
```

**Note:** Average price can be found by doing `Z` divided by `z`.

### Conditional Fields in Execution Report

These are fields that appear in the payload only if certain conditions are met.

For additional information on these parameters, please refer to the [Spot Glossary](./faqs/spot_glossary.md).

<table>
  <tr>
    <th>Field</th>
    <th>Name</th>
    <th>Description</th>
    <th>Examples</th>
  </tr>
  <tr>
    <td><code>d</code></td>
    <td>Trailing Delta</td>
    <td rowspan="2">Appears only for trailing stop orders.</td>
    <td><code>"d": 4</code></td>
  </tr>
  <tr>
    <td><code>D</code></td>
    <td>Trailing Time</td>
    <td><code>"D": 1668680518494</code></td>
  </tr>
  <tr>
    <td><code>j</code></td>
    <td>Strategy Id</td>
    <td>Appears only if the <code>strategyId</code> parameter was provided upon order placement.</td>
    <td><code>"j": 1</code></td>
  </tr>
  <tr>
    <td><code>J</code></td>
    <td>Strategy Type</td>
    <td>Appears only if the <code>strategyType</code> parameter was provided upon order placement.</td>
    <td><code>"J": 1000000</code></td>
  </tr>
  <tr>
    <td><code>v</code></td>
    <td>Prevented Match Id</td>
    <td rowspan="9">Appears only for orders that expired due to STP.</td>
    <td><code>"v": 3</code></td>
  </tr>
  <tr>
    <td><code>A</code>
    <td>Prevented Quantity</td>
    <td><code>"A":"3.000000"</code></td>
  </tr>
  <tr>
    <td><code>B</code></td>
    <td>Last Prevented Quantity</td>
    <td><code>"B":"3.000000"</code></td>
  </tr>
  <tr>
    <td><code>u</code></td>
    <td>Trade Group Id</td>
    <td><code>"u":1</code></td>
  </tr>
  <tr>
    <td><code>U</code></td>
    <td>Counter Order Id</td>
    <td><code>"U":37</code></td>
  </tr>
  <tr>
    <td><code>Cs</code></td>
    <td>Counter Symbol</td>
    <td><code>"Cs": "BTCUSDT"</code></td>
  </tr>
  <tr>
    <td><code>pl</code></td>
    <td>Prevented Execution Quantity</td>
    <td><code>"pl":"2.123456"</code></td>
  </tr>
  <tr>
    <td><code>pL</code></td>
    <td>Prevented Execution Price</td>
    <td><code>"pL":"0.10000001"</code></td>
  </tr>
  <tr>
    <td><code>pY</code></td>
    <td>Prevented Execution Quote Qty</td>
    <td><code>"pY":"0.21234562"</code></td>
  </tr>  
  <tr>
    <td><code>W</code></td>
    <td>Working Time</td>
    <td>Appears when the order is working on the book</td>
    <td><code>"W": 1668683798379</code></td>
  </tr>
  <tr>
    <td><code>b</code></td>
    <td>Match Type</td>
    <td rowspan="2">Appears for orders that have allocations</td>
    <td><code>"b":"ONE_PARTY_TRADE_REPORT"</code></td>
  </tr>
  <tr>
    <td><code>a</code></td>
    <td>Allocation ID</td>
    <td><code>"a":1234</code></td>
  </tr>
  <tr>
    <td><code>k</code></td>
    <td>Working Floor</td>
    <td>Appears for orders that potentially have allocations</td>
    <td><code>"k":"SOR"</code></td>
  </tr>
  <tr>
    <td><code>uS</code></td>
    <td>UsedSor</td>
    <td>Appears for orders that used SOR</td>
    <td><code>"uS":true</code></td>
  </tr>
</table>

If it's an order list, an event will be displayed named `ListStatus` in addition to the `executionReport` event.

**Payload**
```javascript
{
  "e": "listStatus",                // Event Type
  "E": 1564035303637,               // Event Time
  "s": "ETHBTC",                    // Symbol
  "g": 2,                           // OrderListId
  "c": "OCO",                       // Contingency Type
  "l": "EXEC_STARTED",              // List Status Type
  "L": "EXECUTING",                 // List Order Status
  "r": "NONE",                      // List Reject Reason
  "C": "F4QN4G8DlFATFlIUQ0cjdD",    // List Client Order ID
  "T": 1564035303625,               // Transaction Time
  "O": [                            // An array of objects
    {
      "s": "ETHBTC",                // Symbol
      "i": 17,                      // OrderId
      "c": "AJYsMjErWJesZvqlJCTUgL" // ClientOrderId
    },
    {
      "s": "ETHBTC",
      "i": 18,
      "c": "bfYPSQdLoqAJeNrOr9adzq"
    }
  ]
}
```

### Execution types

* `NEW` - The order has been accepted into the engine.
* `CANCELED` - The order has been canceled by the user.
* `REPLACED` (currently unused)
* `REJECTED` - The order has been rejected and was not processed. (e.g. Cancel Replace orders where the new order placement was rejected even if the cancel request succeeded.)
* `TRADE` - Part of the order or all of the order's quantity has filled.
* `EXPIRED` - The order was canceled according to the order type's rules (e.g. LIMIT FOK orders with no fill, LIMIT IOC or MARKET orders that partially fill) or by the exchange, (e.g. orders canceled during liquidation, orders canceled during maintenance).
* `TRADE_PREVENTION` - The order has expired due to STP.

Check the [Rest API Documentation](./rest-api.md#enum-definitions) for more relevant enum definitions.

## Listen Key Expired 

This event is sent when the listen key expires.

No more events will be sent after this until a new `listenKey` is created.

This event will not be pushed when the stream is closed normally.

**Payload:**

```javascript
{
  "e": "listenKeyExpired",  // Event type
  "E": "1699596037418",     // Event time
  "listenKey": "OfYGbUzi3PraNagEkdKuFwUHn48brFsItTdsuiIXrucEvD0rhRXZ7I6URWfE8YE8" 
}
```