<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [FIX API](#fix-api)
  - [General API Information](#general-api-information)
    - [FIX API Order Entry sessions](#fix-api-order-entry-sessions)
    - [FIX API Drop Copy sessions](#fix-api-drop-copy-sessions)
    - [FIX API Market Data sessions](#fix-api-market-data-sessions)
    - [FIX Connection Lifecycle](#fix-connection-lifecycle)
    - [API Key Permissions](#api-key-permissions)
    - [On message processing order](#on-message-processing-order)
    - [Response Mode](#response-mode)
    - [Timing Security](#timing-security)
    - [How to sign Logon `<A>` request](#how-to-sign-logon-a-request)
  - [Limits](#limits)
    - [Message Limits](#message-limits)
    - [Connection Limits](#connection-limits)
    - [Unfilled Order Count](#unfilled-order-count)
  - [Error Handling](#error-handling)
  - [Types](#types)
  - [Message Components](#message-components)
    - [Header](#header)
    - [Trailer](#trailer)
  - [Administrative Messages](#administrative-messages)
    - [Heartbeat `<0>`](#heartbeat-0)
    - [TestRequest `<1>`](#testrequest-1)
    - [Reject `<3>`](#reject-3)
    - [Logon `<A>`](#logon-a)
      - [Logon Request](#logon-request)
      - [Logon Response](#logon-response)
    - [Logout `<5>`](#logout-5)
    - [News `<B>`](#news-b)
    - [Resend Request `<2>`](#resend-request-2)
  - [Application Messages](#application-messages)
    - [Order Entry Messages](#order-entry-messages)
      - [NewOrderSingle `<D>`](#newordersingle-d)
        - [Supported Order Types](#supported-order-types)
      - [ExecutionReport `<8>`](#executionreport-8)
      - [OrderCancelRequest `<F>`](#ordercancelrequest-f)
      - [OrderCancelReject `<9>`](#ordercancelreject-9)
      - [OrderCancelRequestAndNewOrderSingle `<XCN>`](#ordercancelrequestandnewordersingle-xcn)
      - [OrderMassCancelRequest `<q>`](#ordermasscancelrequest-q)
      - [OrderMassCancelReport `<r>`](#ordermasscancelreport-r)
      - [NewOrderList `<E>`](#neworderlist-e)
      - [Supported Order List Types](#supported-order-list-types)
      - [ListStatus `<N>`](#liststatus-n)
      - [OrderAmendKeepPriorityRequest `<XAK>`](#orderamendkeeppriorityrequest-xak)
    - [OrderAmendReject `<XAR>`](#orderamendreject-xar)
    - [Limit Messages](#limit-messages)
      - [LimitQuery `<XLQ>`](#limitquery-xlq)
      - [LimitResponse `<XLR>`](#limitresponse-xlr)
    - [Market Data Messages](#market-data-messages)
      - [InstrumentListRequest `<x>`](#instrumentlistrequest-x)
      - [InstrumentList `<y>`](#instrumentlist-y)
      - [MarketDataRequest `<V>`](#marketdatarequest-v)
    - [MarketDataRequestReject `<Y>`](#marketdatarequestreject-y)
    - [MarketDataSnapshot `<W>`](#marketdatasnapshot-w)
    - [MarketDataIncrementalRefresh `<X>`](#marketdataincrementalrefresh-x)
  - [FIX SBE](#fix-sbe)
    - [SBE](#sbe)
    - [Endpoints](#endpoints)
      - [Order Entry](#order-entry)
      - [Drop Copy](#drop-copy)
      - [Market data](#market-data)
    - [FIX SBE encoding layout](#fix-sbe-encoding-layout)
    - [Logon](#logon)
      - [Sample FIX SBE `Logon` request message](#sample-fix-sbe-logon-request-message)
    - [FIX vs. FIX SBE](#fix-vs-fix-sbe)
    - [Limits](#limits-1)
    - [Errors](#errors)
    - [FAQ](#faq)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# FIX API

> [!NOTE]
> This API can only be used with the SPOT Exchange.

## General API Information

* FIX connections require TLS encryption. Please either use native TCP+TLS connection or set up a local proxy such as [stunnel](https://www.stunnel.org/) to handle TLS encryption.
* APIs have a timeout of 10 seconds when processing a request. If a response from the Matching Engine takes longer than this, the API responds with "Timeout waiting for response from backend server. Send status unknown; execution status unknown." [(-1007 TIMEOUT)](errors.md#-1007-timeout)
  * This does not always mean that the request failed in the Matching Engine.
  * If the status of the request has not appeared in [User Data Stream](user-data-stream.md), please perform an API query for its status.
* If your request contains a symbol name containing non-ASCII characters, then the response may contain non-ASCII characters encoded in UTF-8.

**FIX sessions only support Ed25519 keys.** </br>
You can setup and configure your API key permissions on [Spot Test Network](https://testnet.binance.vision/).

### FIX API Order Entry sessions

* Endpoint is: `tcp+tls://fix-oe.testnet.binance.vision:9000`
* Supports placing orders, canceling orders, and querying current limit usage.
* Supports receiving all of the account's [ExecutionReport`<8>`](#executionreport) and [List Status`<N>`](#liststatus).
* Only API keys with `FIX_API` are allowed to connect.
* QuickFIX Schema can be found [here](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml).

### FIX API Drop Copy sessions

* Endpoint is: `tcp+tls://fix-dc.testnet.binance.vision:9000`
* Supports receiving all of the account's [ExecutionReport`<8>`](#executionreport) and [List Status`<N>`](#liststatus).
* Only API keys with `FIX_API` or `FIX_API_READ_ONLY` are allowed to connect.
* QuickFIX Schema can be found [here](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml).
* Data in Drop Copy sessions is delayed by 1 second.

### FIX API Market Data sessions

* Endpoint is: `tcp+tls://fix-md.testnet.binance.vision:9000`
* Supports market data streams and active instruments queries.
* Does not support placing or canceling orders.
* Only API keys with `FIX_API` or `FIX_API_READ_ONLY` are allowed to connect.
* QuickFIX Schema can be found [here](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml).

### FIX Connection Lifecycle

* All FIX API sessions will remain open for as long as possible, on a best-effort basis.
* There is no minimum connection time guarantee; a server can enter maintenance at any time.
  * When a server enters maintenance, a [News `<B>`](#news) message will be sent to clients **every 10 seconds for 10 minutes**, prompting clients to reconnect. Upon receiving this message, a client is expected to establish a new session and close the old one. If the client does not close the old session within the time frame, the server will proceed to log it out and close the session.
* After connecting, the client must send a Logon `<A>` request. For more information please refer to [How to sign a Logon request](#signaturecomputation).
* The client should send a Logout `<5>` message to close the session before disconnecting. Failure to send the logout message will result in the session’s `SenderCompID (49)` being unusable for new session establishment for a duration of 2x the `HeartInt (108)` interval.
* The system allows negotiation of the `HeartInt (108)` value during the logon process. Accepted values range between 5 and 60 seconds.
  * If the server has not sent any messages within a `HeartInt (108)` interval, a [HeartBeat `<0>`](#heartbeat)  will be sent.
  * If the server has not received any messages within a `HeartInt (108)` interval, a [TestRequest `<1>`](#testrequest) will be sent. If the server does not receive a HeartBeat `<0>` containing the expected `TestReqID (112)` from the client within `HeartInt (108)` seconds, the server will send a Logout `<5>` message and close the connection.
  * If the client has not received any messages within a `HeartInt (108)` interval, the client is responsible for sending a TestRequest `<1>` to ensure the connection is healthy. Upon receiving such a TestRequest `<1>`, the server will respond with a Heartbeat `<0>` containing the expected `TestReqID (112)`. If the client does not receive the server’s response within a `HeartInt (108)` interval, the client should close the session and connection and establish new ones.

### API Key Permissions

To access the FIX API order entry sessions, your API key must be configured with the `FIX_API` permission.

To access the FIX Drop Copy sessions, your API key must be configured with either `FIX_API_READ_ONLY` or `FIX_API` permission.

To access the FIX Market Data sessions, your API key must be configured with either `FIX_API` or `FIX_API_READ_ONLY` permission.

**FIX sessions only support Ed25519 keys.**

<a id="orderedmode"></a>

### On message processing order

The `MessageHandling (25035)` field required in the initial [Logon`<A>`](#logon-request) message controls whether messages from the client may be reordered before they are processed by the Matching Engine.

| Mode            | Description                                                                                |
|-----------------|--------------------------------------------------------------------------------------------|
| `UNORDERED(1)`  | Messages from the client are allowed to be sent to the matching engine in any order.       |
| `SEQUENTIAL(2)` | Messages from the client are always sent to the matching engine in `MsgSeqNum (34)` order. |

In all modes, the client's `MsgSeqNum (34)` must increase monotonically, with each subsequent message having a sequence number that is exactly 1 greater than the previous message.

> [!TIP]
> `UNORDERED(1)` should offer better performance when there are multiple messages in flight from the client to the server.

<a id="responsemode"></a>

### Response Mode

By default, all concurrent order entry sessions receive all of the account's
successful [ExecutionReport`<8>`](#executionreport) and [ListStatus`<N>`](#liststatus) messages,
including those in response to orders placed from other FIX sessions and via non-FIX APIs.

Use the `ResponseMode (25036)` field in the initial [Logon`<A>`](#logon-request) message
to change this behavior.

- `EVERYTHING(1)`: The default mode.
- `ONLY_ACKS(2)`: Receive only ACK messages whether operation succeeded or failed. Disables ExecutionReport push.

<a id="timingsecurity"></a>

### Timing Security

* All requests require a `SendingTime(52)` field which should be the current timestamp.
* An additional optional field, `RecvWindow(25000)`, specifies for how long the request stays valid in milliseconds.
  * `RecvWindow(25000)` supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified.
  * If `RecvWindow(25000)` is not specified, it defaults to 5000 milliseconds only for the Logon`<A>` request. For other requests if unset, the RecvWindow check is not executed.
  * Maximum `RecvWindow(25000)` is 60000 milliseconds.
* Request processing logic is as follows:

```javascript
serverTime = getCurrentTime()
if (SendingTime < (serverTime + 1 second) && (serverTime - SendingTime) <= RecvWindow) {
  // begin processing request
  serverTime = getCurrentTime()
  if (serverTime - SendingTime) <= RecvWindow {
    // forward request to Matching Engine
  } else {
    // reject request
  }
  // finish processing request
} else {
  // reject request
}
```

<a id="signaturecomputation"></a>

### How to sign Logon `<A>` request

The [Logon`<A>`](#logon-main) message authenticates your connection to the FIX API.
This must be the first message sent by the client.

* The `Username (553)` field is required to contain the API key.
* The `RawData (96)` field is required to contain a valid signature made with the API key.

The signature payload is a text string constructed by concatenating the values of the following fields in this exact order,
separated by the SOH character:

1. `MsgType (35)`
2. `SenderCompId (49)`
3. `TargetCompId (56)`
4. `MsgSeqNum (34)`
5. `SendingTime (52)`


Sign the payload using your private key.
Encode the signature with **base64**.
The resulting text string is the value of the `RawData (96)` field.

Here is a sample Python code implementing the signature algorithm:

````python
import base64

from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey
from cryptography.hazmat.primitives.serialization import load_pem_private_key

def logon_raw_data(private_key: Ed25519PrivateKey,
                   sender_comp_id: str,
                   target_comp_id: str,
                   msg_seq_num: str,
                   sending_time: str):
    """
    Computes the value of RawData (96) field in Logon<A> message.
    """
    payload = chr(1).join([
        'A',
        sender_comp_id,
        target_comp_id,
        msg_seq_num,
        sending_time,
    ])
    signature = private_key.sign(payload.encode('ASCII'))
    return base64.b64encode(signature).decode('ASCII')


with open('private_key.pem', 'rb') as f:
    private_key = load_pem_private_key(data=f.read(),
                                       password=None)

raw_data = logon_raw_data(private_key,
                          sender_comp_id='5JQmUOsm',
                          target_comp_id='SPOT',
                          msg_seq_num='1',
                          sending_time='20240612-08:52:21.613')
````

The values presented below can be used to validate the correctness of the signature computation implementation:

| Field             | Value                   |
|-------------------|-------------------------|
| MsgType (35)      | `A`                     |
| SenderCompID (49) | `EXAMPLE`               |
| TargetCompID (56) | `SPOT`                  |
| MsgSeqNum (34)    | `1`                     |
| SendingTime (52)  | `20240627-11:17:25.223` |

The Ed25519 private key used in the example computation is shown below:

> [!CAUTION]
> The following secret key is provided solely for illustrative purposes. Do not use this key in any real-world application as it is not secure and may compromise your cryptographic implementation. Always generate your own unique and secure keys for actual use.

```
-----BEGIN PRIVATE KEY-----
MC4CAQAwBQYDK2VwBCIEIIJEYWtGBrhACmb9Dvy+qa8WEf0lQOl1s4CLIAB9m89u
-----END PRIVATE KEY-----
```

Computed signature:
```
4MHXelVVcpkdwuLbl6n73HQUXUf1dse2PCgT1DYqW9w8AVZ1RACFGM+5UdlGPrQHrgtS3CvsRURC1oj73j8gCA==
```

Resulting Logon `<A>` message:
```
8=FIX.4.4|9=247|35=A|34=1|49=EXAMPLE|52=20240627-11:17:25.223|56=SPOT|95=88|96=4MHXelVVcpkdwuLbl6n73HQUXUf1dse2PCgT1DYqW9w8AVZ1RACFGM+5UdlGPrQHrgtS3CvsRURC1oj73j8gCA==|98=0|108=30|141=Y|553=sBRXrJx2DsOraMXOaUovEhgVRcjOvCtQwnWj8VxkOh1xqboS02SPGfKi2h8spZJb|25035=2|10=227|
```

## Limits

### Message Limits

* Each connection has a limit on **how many messages can be sent to the exchange**.
* The message limit **does not count the messages sent in response to the client**.
* Breaching the message limit results in immediate [Logout `<5>`](#logout) and disconnection.
* To understand current limits and usage, please send a [LimitQuery`<XLQ>`](#limitquery) message.
  A [LimitResponse`<XLR>`](#limitresponse) message will be sent in response, containing information about Order Rate
  Limits and Message Limits.
* FIX Order entry sessions have a limit of 10,000 messages every 10 seconds.
* FIX Drop Copy sessions have a limit of 60 messages every 60 seconds.
* FIX Market Data sessions have a limit of 2000 messages every 60 seconds.

<a id="connection-limits"></a>

### Connection Limits

* Each Account has a limit on how many TCP connections can be established at the same time.
* The limit is reduced when the TCP connection is closed. If the reduction of connections is not immediate, please wait up to twice the value of `HeartBtInt (108)` for the change to take effect.
  For example, if the current value of `HeartBtInt` is 5, please wait up to 10 seconds.
* Upon breaching the limit a [Reject `<3>`](#reject) will be sent containing information about the connection limit
  breach and the current limit.
* FIX Order Entry limits:
   * 15 connection attempts within 30 seconds
   * Maximum of 10 concurrent TCP connections per account
* FIX Drop Copy limits:
    * 15 connection attempts within 30 seconds
    * Maximum of 10 concurrent TCP connections per account
* FIX Market Data limits
  * 300 connection attempts within 300 seconds
  * Maximum of 100 concurrent TCP connections per account
  * A single connection can listen to a maximum of 1000 streams.

### Unfilled Order Count

* To understand how many orders you have placed within a certain time interval, please send a [LimitQuery`<XLQ>`](#limitquery) message.
  A [LimitResponse`<XLR>`](#limitresponse) message will be sent in response, containing information about Unfilled Order Count and Message Limits.
* **Please note that if your orders are consistently filled by trades, you can continuously place orders on the API**. For more information, please see [Spot Unfilled Order Count Rules](../faqs/order_count_decrement.md).
* If you exceed the unfilled order count your message will be rejected, and information will be transferred back to you in a reject message specific to that endpoint.
* **The number of unfilled orders is tracked for each account.**

## Error Handling

Client messages that contain syntax errors, missing required fields, or refer to unknown symbols will be rejected by the server with a [Reject `<3>`](#reject) message.

If a valid message cannot be processed and is rejected, an appropriate reject response will be sent.
Please refer to the individual message documentation for possible responses.

Please refer to the `Text (58)` and `ErrorCode (25016)` fields in responses for the reject reason.

The list of error codes can be found on the [Error codes](errors.md) page.

## Types

Only printable ASCII characters and SOH are supported.

| Type           | Description                                                                                 |
|----------------|---------------------------------------------------------------------------------------------|
| `BOOLEAN`      | Enum: `Y` or `N`.                                                                           |
| `CHAR`         | Single character.                                                                           |
| `INT`          | Signed 64-bit integer.                                                                      |
| `LENGTH`       | Unsigned 64-bit integer.                                                                    |
| `NUMINGROUP`   | Unsigned 64-bit integer.                                                                    |
| `PRICE`        | Fixed-point number. Precision depends on the symbol definition.                             |
| `QTY`          | Fixed-point number. Precision depends on the symbol definition.                             |
| `SEQNUM`       | Unsigned 32-bit integer. Rolls over to 0 after reaching its maximum value of 4,294,967,295. |
| `STRING`       | Sequence of printable ASCII characters.                                                     |
| `UTCTIMESTAMP` | String representing datetime in UTC.                                                        |

Supported `UTCTIMESTAMP` formats:

* `20011217-09:30:47` - seconds
* `20011217-09:30:47.123` - milliseconds
* `20011217-09:30:47.123456` - microseconds (always used in messages from the exchange)

Client order ID fields must conform to the regex `^[a-zA-Z0-9-_]{1,36}$`:

* `ClOrdID (11)`
* `OrigClOrdID (41)`
* `MDReqID (262)`
* `ClListID (25014)`
* `OrigClListID (25015)`
* `CancelClOrdID (25034)`

## Message Components

> [!NOTE]
> In example messages, the `|` character is used to represent SOH character:

```
8=FIX.4.4|9=113|35=A|34=1|49=SPOT|52=20240612-08:52:21.636837|56=5JQmUOsm|98=0|108=30|25037=4392a152-3481-4499-921a-6d42c50702e2|10=051|
```

<a id="header"></a>

### Header

Appears at the start of every message.

| Tag   | Name         | Type         | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|-------|--------------|--------------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 8     | BeginString  | STRING       | Y        | Always `FIX.4.4`. <br></br> Must be the first field the message.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| 9     | BodyLength   | LENGTH       | Y        | Message length in bytes. <br></br> Must be the second field in the message.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| 35    | MsgType      | STRING       | Y        | Must be the third field in the message. <br></br> Possible values: <br></br>`0` - [HEARTBEAT](#heartbeat) <br></br>`1` - [TEST_REQUEST](#testrequest) <br></br>`3` - [REJECT](#reject) <br></br>`5` - [LOGOUT](#logout) <br></br>`8` - [EXECUTION_REPORT](#executionreport) <br></br> `9` - [ORDER_CANCEL_REJECT](#ordercancelreject) <br></br> `A` - [LOGON](#logon-main) <br></br> `D` - [NEW_ORDER_SINGLE](#newordersingle) <br></br> `E` - [NEW_ORDER_LIST](#neworderlist) <br></br> `F` - [ORDER_CANCEL_REQUEST](#ordercancelrequest) <br></br> `N` - [LIST_STATUS](#liststatus) <br></br> `q` - [ORDER_MASS_CANCEL_REQUEST](#ordermasscancelrequest) <br></br> `r` - [ORDER_MASS_CANCEL_REPORT](#ordermasscancelreport) <br></br> `XCN` - [ORDER_CANCEL_REQUEST_AND_NEW_ORDER_SINGLE](#ordercancelrequestandnewordersingle) <br></br> `XLQ` - [LIMIT_QUERY](#limitquery) <br></br> `XLR` - [LIMIT_RESPONSE](#limitresponse) <br></br> `B` - [NEWS](#news) <br></br> `x`- [INSTRUMENT_LIST_REQUEST](#instrumentlistrequest) <br></br> `y` - [INSTRUMENT_LIST](#instrumentlist)  <br></br>`V` - [MARKET_DATA_REQUEST](#marketdatarequest) <br></br> `Y` - [MARKET_DATA_REQUEST_REJECT](#marketdatarequestreject) <br></br>`W` - [MARKET_DATA_SNAPSHOT](#marketdatasnapshot) <br></br>`X` - [MARKET_DATA_INCREMENTAL_REFRESH](#marketdataincrementalrefresh) <br></br> `XAK` - [ORDER_AMEND_KEEP_PRIORITY_REQUEST](#orderamendkeeppriorityrequest) <br></br> `XAR` - [ORDER_AMEND_REJECT](#orderamendreject) |
| 49    | SenderCompID | STRING       | Y        | Must be unique across an account's active sessions.  <br></br> Must obey regex: `^[a-zA-Z0-9-_]{1,8}$`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| 56    | TargetCompID | STRING       | Y        | A string identifying this TCP connection.<br></br>On messages from client required to be set to `SPOT`. <br></br>Must be unique across TCP connections. <br></br> Must conform to the regex: `^[a-zA-Z0-9-_]{1,8}$`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |                                                                                                                                      |
| 34    | MsgSeqNum    | SEQNUM       | Y        | Integer message sequence number. <br></br> Values that will cause a gap will be rejected.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| 52    | SendingTime  | UTCTIMESTAMP | Y        | Time of message transmission (always expressed in UTC). |
| 25000 | RecvWindow   | FLOAT          | N        | Number of milliseconds after `SendingTime (52)` the request is valid for. <br></br> Defaults to `5000` milliseconds in [Logon`<A>`](#logon-request) and has a max value of `60000` milliseconds. <br> Supports up to three decimal places of precision (e.g., 6000.346) so that microseconds may be specified. |

<a id="trailer"></a>

### Trailer

Appears at the end of every message.

| Tag | Name     | Type   | Required | Description                                                                                                                                                                                                                                                                                                                                                      |
|-----|----------|--------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 10  | CheckSum | STRING | Y        | Always three-character numeric string, calculated by summing the ASCII values of each preceding character in the message, including start-of-header (SOH) characters. <br></br> The resultant sum is divided by 256, with the remainder forming the CheckSum value. <br></br> To maintain a fixed length, the CheckSum field is right-justified and zero-padded as needed. |

## Administrative Messages

<a id="heartbeat"></a>

### Heartbeat `<0>`

Sent by the server if there is no outgoing traffic during the heartbeat interval (`HeartBtInt (108)` in [Logon`<A>`](#logon-main)).

Sent by the client to indicate that the session is healthy.

Sent by the client or the server in response to a [TestRequest`<1>`](#testrequest) message.

| Tag | Name      | Type   | Required | Description                                                                                              |
|-----|-----------|--------|----------|----------------------------------------------------------------------------------------------------------|
| 112 | TestReqID | STRING | N        | When Heartbeat`<35>` is sent in response to TestRequest`<1>`, must mirror the value in TestRequest`<1>`. |

<a id="testrequest"></a>

### TestRequest `<1>`

Sent by the server if there is no incoming traffic during the heartbeat interval (`HeartBtInt (108)` in [Logon`<A>`](#logon-main)).

Sent by the client to request a [Heartbeat`<0>`](#heartbeat) response.

> [!NOTE]
> If the client does not respond to TestRequest`<1>` with Heartbeat`<0>` with a correct `TestReqID (112)`  within timeout, the connection will be dropped.

| Tag | Name      | Type   | Required | Description                                                            |
|-----|-----------|--------|----------|------------------------------------------------------------------------|
| 112 | TestReqID | STRING | Y        | Arbitrary string that must be included in the Heartbeat`<0>` response. |

<a id="reject"></a>

### Reject `<3>`

Sent by the server in response to an invalid message that cannot be processed.

Sent by the server if a new connection cannot be accepted.
Please refer to [Connection Limits](#connection-limits).

Please refer to the `Text (58)` and `ErrorCode (25016)` fields for the reject reason.

| Tag   | Name                | Type   | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|-------|---------------------|--------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 45    | RefSeqNum           | INT    | N        | The `MsgSeqNum (34)` of the rejected message that caused issuance of this Reject`<3>`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| 371   | RefTagID            | INT    | N        | When present, identifies the field that directly caused the issuance of this Reject`<3>` message.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 372   | RefMsgType          | STRING | N        | The `MsgType (35)` of the rejected message that caused issuance of this Reject`<3>`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| 373   | SessionRejectReason | INT    | N        | A reason for the reject, can be one of the values below. <br></br> Usually accompanied by additional Text description <br></br> Possible values: <br></br>`0`- INVALID_TAG_NUMBER <br></br> `1` - REQUIRED_TAG_MISSING <br></br> `2` - TAG_NOT_DEFINED_FOR_THIS_MESSAGE_TYPE <br></br> `3` - UNDEFINED_TAG <br></br> `5` - VALUE_IS_INCORRECT <br></br> `6` - INCORRECT_DATA_FORMAT_FOR_VALUE <br></br> `8` - SIGNATURE_PROBLEM <br></br> `10` - SENDINGTIME_ACCURACY_PROBLEM   <br></br> `12` - XML_VALIDATION_ERROR <br></br> `13` - TAG_APPEARS_MORE_THAN_ONCE <br></br> `14` - TAG_SPECIFIED_OUT_OF_REQUIRED_ORDER <br></br> `15` - REPEATING_GROUP_FIELDS_OUT_OF_ORDER <br></br> `16` - INCORRECT_NUMINGROUP_COUNT_FOR_REPEATING_GROUP<br></br> `99` - OTHER |
| 25016 | ErrorCode           | INT    | N        | API error code (see [Error Codes](errors.md)).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| 58    | Text                | STRING | N        | Human-readable error message.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |

<a id="logon-main"></a>

### Logon `<A>`

Sent by the client to authenticate the connection.
Logon`<A>` must be the first message sent by the client.

Sent by the server in response to a successful logon.

> [!NOTE]
> Logon`<A>` can only be sent once for the entirety of the session.

<a id="logon-request"></a>

#### Logon Request

| Tag   | Name            | Type    | Required | Description                                                                                                                                        |
|-------|-----------------|---------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| 98    | EncryptMethod   | INT     | Y        | Required to be `0`.                                                                                                                                |
| 108   | HeartBtInt      | INT     | Y        | Required to be within range [5, 60]. Heartbeat interval in seconds.                                                                                |
| 95    | RawDataLength   | LENGTH  | Y        | Length of the `RawData (96)` field that comes strictly after this field.                                                                           |                                                                                                                                                                                                                                                                                                                              |
| 96    | RawData         | DATA    | Y        | Signature. [How to sign Logon`<A>` request](#signaturecomputation).                                                                                |
| 141   | ResetSeqNumFlag | BOOLEAN | Y        | Required to be `Y`.                                                                                                                                |
| 553   | Username        | STRING  | Y        | API key. **Only Ed25519 API keys are supported.**                                                                                                  |
| 25035 | MessageHandling | INT     | Y        | Possible values: <br></br> `1` - UNORDERED <br></br> `2` - SEQUENTIAL <br></br> Please refer to [On message order processing](#orderedmode) for more information. |
| 25036 | ResponseMode    | INT     | N        | Please refer to [Response Mode](#responsemode).                                                                                                    |
| 9406  | DropCopyFlag    |BOOLEAN  | N        |Must be set to 'Y' when logging into Drop Copy sessions.|

**Sample message:**

```
8=FIX.4.4|9=248|35=A|34=1|49=5JQmUOsm|52=20240612-08:52:21.613|56=SPOT|95=88|96=KhJLbZqADWknfTAcp0ZjyNz36Kxa4ffvpNf9nTIc+K5l35h+vA1vzDRvLAEQckyl6VDOwJ53NOBnmmRYxQvQBQ==|98=0|108=30|141=Y|553=W5rcOD30c0gT4jHK8oX5d5NbzWoa0k4SFVoTHIFNJVZ3NuRpYb6ZyJznj8THyx5d|25035=1|10=000|
```

<a id="logon-response"></a>

#### Logon Response

| Tag   | Name          | Type   | Required | Description                               |
|-------|---------------|--------|----------|-------------------------------------------|
| 98    | EncryptMethod | INT    | Y        | Always `0`.                               |
| 108   | HeartBtInt    | INT    | Y        | Mirrors value from the Logon request.     |
| 25037 | UUID          | STRING | Y        | UUID of the FIX API serving the requests. |

**Sample message:**

```
8=FIX.4.4|9=113|35=A|34=1|49=SPOT|52=20240612-08:52:21.636837|56=5JQmUOsm|98=0|108=30|25037=4392a152-3481-4499-921a-6d42c50702e2|10=051|
```

<a id="logout"></a>

### Logout `<5>`

Sent to initiate the process of closing the connection, and also when responding to Logout.

| Tag | Name | Type   | Required | Description |
|-----|------|--------|----------|-------------|
| 58  | Text | STRING | N        |             |

**Sample messages:**

Logout Request

```
8=FIX.4.4|9=55|35=5|34=3|49=GhQHzrLR|52=20240611-09:44:25.543|56=SPOT|10=249|
```

Logout Response

```
8=FIX.4.4|9=84|35=5|34=4|49=SPOT|52=20240611-09:44:25.544001|56=GhQHzrLR|58=Logout acknowledgment.|10=212|
```

<a id="news"></a>

### News `<B>`

When the server enters maintenance, a `News` message will be sent to clients **every 10 seconds for 10 minutes**.
After this period, clients will be logged out and their sessions will be closed.

Upon receiving this message, clients are expected to establish a new session and close the old one.


The countdown message sent will be:

```
You'll be disconnected in %d seconds. Please reconnect.
```
When there are 10 seconds remaining, the following message will be sent:
```
Your connection is about to be closed. Please reconnect.
```
If the client does not close the old session within 10 seconds of receiving the above message, the server will log it out and close the session.

| Tag | Name | Type | Required | Description |
| :---- | :---- | :---- | :---- | :---- |
| 148 | Headline | STRING | Y |    |

**Sample message:**

```
8=FIX.4.4|9=0000113|35=B|49=SPOT|56=OE|34=4|52=20240924-21:07:35.773537|148=Your connection is about to be closed. Please reconnect.|10=165|
```

### Resend Request `<2>`

Resend requests are currently not supported.

## Application Messages

### Order Entry Messages

> [!NOTE]
> The messages below can only be used for the FIX Order Entry and FIX Drop Copy Sessions.

<a id="newordersingle"></a>

#### NewOrderSingle `<D>`

Sent by the client to submit a new order for execution.

This adds 1 order to the `EXCHANGE_MAX_ORDERS` filter and the `MAX_NUM_ORDERS` filter.

**Unfilled Order Count:** 1

Please refer to [Supported Order Types](#ordertype) for supported field combinations.

> [!NOTE]
> Many fields become required based on the order type.
> Please refer to [Supported Order Types](#NewOrderSingle-required-fields).

| Tag   | Name                     | Type    | Required | Description                                                                                                                                                                                                                                                                                                                  |
|-------|--------------------------|---------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 11    | ClOrdID                  | STRING  | Y        | `ClOrdID` to be assigned to the order.                                                                                                                                                                                                                                                                                       |
| 38    | OrderQty                 | QTY     | N        | Quantity of the order                                                                                                                                                                                                                                                                                                        |
| 40    | OrdType                  | CHAR    | Y        | See the [table](#ordertype) to understand supported order types and the required fields to use them.<br></br>Possible values: <br></br> `1` - MARKET <br></br> `2` - LIMIT <br></br> `3` - STOP <br></br> `4` - STOP_LIMIT  <br></br> `P`- PEGGED                                                                                     |
| 18    | ExecInst                 | CHAR    | N        | Possible values: <br></br> `6` - PARTICIPATE_DONT_INITIATE                                                                                                                                                                                                                                                                        |
| 44    | Price                    | PRICE   | N        | Price of the order                                                                                                                                                                                                                                                                                                           |
| 54    | Side                     | CHAR    | Y        | Side of the order.<br></br>Possible values: <br></br> `1` - BUY <br></br> `2` - SELL                                                                                                                                                                                                                                                        |
| 55    | Symbol                   | STRING  | Y        | Symbol to place the order on.                                                                                                                                                                                                                                                                                                |
| 59    | TimeInForce              | CHAR    | N        | Possible values: <br></br> `1` - GOOD_TILL_CANCEL <br></br> `3` - IMMEDIATE_OR_CANCEL <br></br> `4` - FILL_OR_KILL                                                                                                                                                                                                                          |
| 111   | MaxFloor                 | QTY     | N        | Used for iceberg orders, this specifies the visible quantity of the order on the book.                                                                                                                                                                                                                                       |
| 152   | CashOrderQty             | QTY     | N        | Quantity of the order specified in the quote asset units, for reverse market orders.                                                                                                                                                                                                                                         |
| 847   | TargetStrategy           | INT     | N        | The value cannot be less than `1000000`.                                                                                                                                                                                                                                                                                                                             |
| 7940  | StrategyID               | INT     | N        |                                                                                                                                                                                                                                                                                      |
| 25001 | SelfTradePreventionMode  | CHAR    | N        | Possible values: <br></br> `1` - NONE <br></br> `2` - EXPIRE_TAKER <br></br> `3` - EXPIRE_MAKER <br></br> `4` - EXPIRE_BOTH <br></br> `5` - DECREMENT  <br> `6` - TRANSFER                                                                                                                                                                                                                    |
|211 | PegOffsetValue | FLOAT | N | Amount added to the peg in the context of the PegOffsetType |
|1094 | PegPriceType | CHAR | N | Defines the type of peg <br> Possible values: <br> `4` - MARKET_PEG <br> `5` - PRIMARY_PEG|
|835 | PegMoveType | CHAR | N | Describes whether peg is fixed or floats. Required for Pegged Orders and must be set to `1` (FIXED) |
|836 | PegOffsetType | CHAR | N | Type of price peg offset. <br> Possible values: <br></br> `3`  - PRICE_TIER|
| 1100  | TriggerType              | CHAR    | N        | Possible values: `4` - PRICE_MOVEMENT                                                                                                                                                                                                                                                                                        |
| 1101  | TriggerAction            | CHAR    | N        | Possible values: <br></br> `1` - ACTIVATE                                                                                                                                                                                                                                                                                         |
| 1102  | TriggerPrice             | PRICE   | N        | Activation price for contingent orders. See [table](#ordertype)                                                                                                                                                                                                                                                              |
| 1107  | TriggerPriceType         | CHAR    | N        | Possible values: <br></br> `2` - LAST_TRADE                                                                                                                                                                                                                                                                                       |
| 1109  | TriggerPriceDirection    | CHAR    | N        | Used to differentiate between StopLoss and TakeProfit orders. See [table](#ordertype).<br></br>Possible values: <br></br> `U` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_UP_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE <br></br> `D` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_DOWN_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE |
| 25009 | TriggerTrailingDeltaBips | INT     | N        | Provide to create trailing orders.                                                                                                                                                                                                                                                                                           |
| 25032 | SOR                      | BOOLEAN | N        | Whether to activate SOR for this order.                                                                                                                                                                                                                                                                                      |

**Sample message:**

```
8=FIX.4.4|9=114|35=D|34=2|49=qNXO12fH|52=20240611-09:01:46.228|56=SPOT|11=1718096506197867067|38=5|40=2|44=10|54=1|55=LTCBNB|59=4|10=016|
```

**Response:**

* [ExecutionReport`<8>`](#executionreport) with `ExecType (150)` value `NEW (0)` if the order was accepted.
* [ExecutionReport`<8>`](#executionreport) with `ExecType (150)` value `REJECTED (8)` if the order was rejected.
* [Reject`<3>`](#reject) if the message is rejected.

<a id="ordertype"></a>

##### Supported Order Types

| Order name                            | Binance OrderType   | Side        | required field values                    | required fields with user values |
|---------------------------------------|---------------------|-------------|------------------------------------------|----------------------------------|
| Market order                          | `MARKET`             | BUY or SELL | <code>40=1&#124;</code>                                 |                                  |
| Limit order                           | `LIMIT`             | BUY or SELL | <code>40=2&#124;</code>                                 |                                  |
| Limit maker order                     | `LIMIT_MAKER`       | BUY or SELL | <code>40=2&#124;18=6&#124;</code>                           |                                  |
| Buy stop loss order                   | `STOP_LOSS`         | BUY         | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102                             |
| Buy trailing stop loss order          | `STOP_LOSS`         | BUY         | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102,25009                        |
| Buy stop loss limit order             | `STOP_LOSS_LIMIT`   | BUY         | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102                             |
| Buy trailing stop loss limit order    | `STOP_LOSS_LIMIT`   | BUY         | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102,25009                        |
| Sell stop loss order                  | `STOP_LOSS`         | SELL        | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102                             |
| Sell trailing stop loss order         | `STOP_LOSS`         | SELL        | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102,25009                        |
| Sell stop loss limit order            | `STOP_LOSS_LIMIT`   | SELL        | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102                             |
| Sell trailing stop loss limit order   | `STOP_LOSS_LIMIT`   | SELL        | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102,25009                        |
| Buy take profit order                 | `TAKE_PROFIT`       | BUY         | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102                             |
| Buy trailing take profit order        | `TAKE_PROFIT`       | BUY         | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102,25009                        |
| Buy trailing take profit order        | `TAKE_PROFIT`       | BUY         | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;</code>         | 25009                             |
| Buy take profit order                 | `TAKE_PROFIT_LIMIT` | BUY         | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102                             |
| Buy trailing take profit limit order  | `TAKE_PROFIT_LIMIT` | BUY         | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=D&#124;</code> | 1102,25009                        |
| Buy trailing take profit limit order  | `TAKE_PROFIT_LIMIT` | BUY         | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;</code>         | 25009                             |
| Sell take profit order                | `TAKE_PROFIT`       | SELL        | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102                             |
| Sell trailing take profit order       | `TAKE_PROFIT`       | SELL        | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102,25009                        |
| Sell trailing take profit order       | `TAKE_PROFIT`       | SELL        | <code>40=3&#124;1100=4&#124;1101=1&#124;1107=2&#124;</code>         | 25009                             |
| Sell take profit limit order          | `TAKE_PROFIT_LIMIT` | SELL        | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102                             |
| Sell trailing take profit limit order | `TAKE_PROFIT_LIMIT` | SELL        | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;1109=U&#124;</code> | 1102,25009                        |
| Sell trailing take profit limit order | `TAKE_PROFIT_LIMIT` | SELL        | <code>40=4&#124;1100=4&#124;1101=1&#124;1107=2&#124;</code>         | 25009                             |

<a id="NewOrderSingle-required-fields"></a>

Required fields based on Binance OrderType:

| Binance OrderType   | Additional mandatory parameters | Additional Information   |
|---------------------|---------------------------------|--------------------------|
| `LIMIT`             | 38, 44, 59                      |                          |
| `MARKET`            | 38 OR 152                       | `MARKET` orders using the `OrderQty (38)` field specifies the amount of the `base asset` the user wants to buy or sell at the market price. <br/> E.g. `MARKET` order on BTCUSDT will specify how much BTC the user is buying or selling. <br/><br/> `MARKET` orders using `quoteOrderQty` specifies the amount the user wants to spend (when buying) or receive (when selling) the `quote` asset; the correct `quantity` will be determined based on the market liquidity and `quoteOrderQty`. <br/> E.g. Using the symbol BTCUSDT: <br/> `BUY` side, the order will buy as many BTC as `quoteOrderQty` USDT can. <br/> `SELL` side, the order will sell as much BTC needed to receive `CashOrderQty (152)` USDT. |
| `STOP_LOSS`         | 38, 1102 or 25009               | This will execute a `MARKET` order when the conditions are met. (e.g. `TriggerPrice (1102)` is met or `TriggerTrailingDeltaBips (25009)` is activated)  |
| `STOP_LOSS_LIMIT`   | 38, 44, 59, 1102 or 25009       |                           |
| `TAKE_PROFIT`       | 38, 1102 or 25009               | This will execute a `MARKET` order when the conditions are met. (e.g. `TriggerPrice (1102)` is met or `TriggerTrailingDeltaBips (25009)` is activated)   |
| `TAKE_PROFIT_LIMIT` | 38, 44, 59, 1102 or 25009       |
| `LIMIT_MAKER`       | 38, 44                          | This is a `LIMIT` order that will be rejected if the order immediately matches and trades as a taker. <br/> This is also known as a POST-ONLY order. |

<a id="executionreport"></a>

#### ExecutionReport `<8>`

Sent by the server whenever an order state changes.

> [!NOTE]
> * By default, ExecutionReport`<8>` is sent for all orders of an account, including those submitted in different connections. Please see [Response Mode](#responsemode) for other behavior options.
> * FIX API should give better performance for ExecutionReport<code>&lt;8&gt;</code> push.

| Tag   | Name                     | Type         | Required | Description                                                                                                                                                                                                                                                                                                                  |
|-------|--------------------------|--------------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 17    | ExecID                   | STRING       | N        | Omitted on rejected orders.                                                                                                                                                                                                                                                                                                  |
| 11    | ClOrdID                  | STRING       | N        | `ClOrdID` of the list as assigned on the request.                                                                                                                                                                                                                                                                            |
| 41    | OrigClOrdID              | STRING       | N        | Original `ClOrdID` of the order.                                                                                                                                                                                                                                                                                             |
| 37    | OrderID                  | INT          | N        | Assigned by exchange.                                                                                                                                                                                                                                                                                                        |
| 38    | OrderQty                 | QTY          | N        | Quantity of the order.                                                                                                                                                                                                                                                                                                       |
| 40    | OrdType                  | CHAR         | Y        | Possible values: <br></br> `1` - MARKET <br></br> `2` - LIMIT <br></br> `3` - STOP_LOSS <br></br> `4` - STOP_LIMIT <br></br> `P` - PEGGED                                                                                                                                                                                                                               |
| 54    | Side                     | CHAR         | Y        | Possible values: <br></br> `1` - BUY <br></br> `2` - SELL                                                                                                                                                                                                                                                                              |
| 55    | Symbol                   | STRING       | Y        | Symbol of the order.                                                                                                                                                                                                                                                                                                         |
| 18    | ExecInst                 | CHAR         | N        | Possible values: <br></br> `6` - PARTICIPATE_DONT_INITIATE                                                                                                                                                                                                                                                                        |
| 44    | Price                    | PRICE        | N        | Price of the order.                                                                                                                                                                                                                                                                                                          |
| 59    | TimeInForce              | CHAR         | N        | Possible values: <br></br> `1` - GOOD_TILL_CANCEL <br></br> `3` - IMMEDIATE_OR_CANCEL <br></br> `4` - FILL_OR_KILL                                                                                                                                                                                                                          |
| 60    | TransactTime             | UTCTIMESTAMP | N        | Timestamp when this event occurred.                                                                                                                                                                                                                                                                                          |
| 25018 | OrderCreationTime        | INT          | N        |                                                                                                                                                                                                                                                                                                                              |
| 111   | MaxFloor                 | QTY          | N        | Appears on iceberg orders.                                                                                                                                                                                                                                                                                                   |
| 66    | ListID                   | STRING       | N        | Appears on list orders.                                                                                                                                                                                                                                                                                                      |
| 152   | CashOrderQty             | QTY          | N        | OrderQty specified in the quote asset units.                                                                                                                                                                                                                                                                                 |
| 847   | TargetStrategy           | INT          | N        | `TargetStrategy (847)` from the order placement request.                                                                                                                                                                                                                                                                     |
| 7940  | StrategyID               | INT          | N        | `StrategyID (7940)` from the order placement request.                                                                                                                                                                                                                                                                        |
| 25001 | SelfTradePreventionMode  | CHAR         | N        | Possible values: <br></br> `1` - NONE <br></br> `2` - EXPIRE_TAKER <br></br> `3` - EXPIRE_MAKER <br></br>`4` - EXPIRE_BOTH  <br></br> `5` - DECREMENT  <br> `6` - TRANSFER                                                                                                                                                                                                                    |
| 150   | ExecType                 | CHAR         | Y        | **Note:** Field `PreventedMatchID(25024)` will be present if order has expired due to `SelfTradePreventionMode(25013)` <br></br> Possible values: <br></br> `0` - NEW <br></br> `4` - CANCELED <br></br> `5` - REPLACED <br></br> `8` - REJECTED <br></br> `F` - TRADE <br></br>`C` - EXPIRED                                                                   |
| 14    | CumQty                   | QTY          | Y        | Total number of base asset traded on this order.                                                                                                                                                                                                                                                                            |
| 151   | LeavesQty                | QTY          | N        | Quantity remaining for further execution.                                                                                                                                                                                                                                                                                    |
| 25017 | CumQuoteQty              | QTY          | N        | Total number of quote asset traded on this order.                                                                                                                                                                                                                                                                             |
| 1057  | AggressorIndicator       | BOOLEAN      | N        | Appears on trade execution reports. <br></br>Indicates whether the order was a taker in the trade.                                                                                                                                                                                                                                |
| 1003  | TradeID                  | STRING       | N        | Appears on trade execution reports.                                                                                                                                                                                                                                                                                          |
| 31    | LastPx                   | PRICE        | N        | The price of the last execution.                                                                                                                                                                                                                                                                                             |
| 32    | LastQty                  | QTY          | Y        | The quantity of the last execution.                                                                                                                                                                                                                                                                                          |
| 39    | OrdStatus                | CHAR         | Y        | Possible values: <br></br> `0` - NEW <br></br> `1` - PARTIALLY_FILLED <br></br> `2` - FILLED <br></br> `4` - CANCELED `6` - PENDING_CANCEL<br></br> `8` - REJECTED <br></br> `A` - PENDING_NEW <br></br> `C` - EXPIRED <br></br> Note that FIX does not support `EXPIRED_IN_MATCH` status, and get converted to `EXPIRED` in FIX.                                    |                                                                                                                                                                                                                                                                                           |
| 70 | AllocID                  | INT          | N        | Allocation ID as assigned by the exchange.                                                                                                                                                                                                                                                                                   |
| 574 | MatchType                | INT          | N        | Possible values:<br></br>`1` - ONE_PARTY_TRADE_REPORT<br></br>`4` - AUTO_MATCH                                                                                                                                                                                                                                                         |
| 25021 | WorkingFloor             | INT          | N        | Appears for orders that potentially have allocations.                                                                                                                                                                                                                                                                        |
| 25022 | TrailingTime             | UTCTIMESTAMP          | N        | Appears only for trailing stop orders.                                                                                                                                                                                                                                                                                       |
| 636   | WorkingIndicator         | BOOLEAN      | N        | Set to `Y` when this order enters order book.                                                                                                                                                                                                                                                                                |
| 25023 | WorkingTime              | UTCTIMESTAMP | N        | When this order appeared on the order book.                                                                                                                                                                                                                                                                                  |
| 25024 | PreventedMatchID         | INT          | N        | Appears only for orders that expired due to STP.                                                                                                                                                                                                                                                                             |
| 25025 | PreventedExecutionPrice  | PRICE        | N        | Appears only for orders that expired due to STP.                                                                                                                                                                                                                                                                             |
| 25026 | PreventedExecutionQty    | QTY          | N        | Appears only for orders that expired due to STP.                                                                                                                                                                                                                                                                             |
| 25027 | TradeGroupID             | INT          | N        | Appears only for orders that expired due to STP.                                                                                                                                                                                                                                                                             |
| 25028 | CounterSymbol            | STRING       | N        | Appears only for orders that expired due to STP.                                                                                                                                                                                                                                                                             |
| 25029 | CounterOrderID           | INT          | N        | Appears only for orders that expired due to STP.                                                                                                                                                                                                                                                                             |
| 25030 | PreventedQty             | QTY          | N        | Appears only for orders that expired due to STP.                                                                                                                                                                                                                                                                             |
| 25031 | LastPreventedQty         | QTY          | N        | Appears only for orders that expired due to STP.                                                                                                                                                                                                                                                                             |
| 25032 | SOR                      | BOOLEAN      | N        | Appears for orders that used SOR.                                                                                                                                                                                                                                                                                            |
| 25016 | ErrorCode                | INT          | N        | API error code (see [Error Codes](errors.md)).                                                                                                                                                                                                                                                                               |
| 58    | Text                     | STRING       | N        | Human-readable error message.                                                                                                                                                                                                                                                                                                |
| 136   | NoMiscFees               | NUMINGROUP   | N        | Number of repeating groups of miscellaneous fees.                                                                                                                                                                                                                                                                            |
| =>137 | MiscFeeAmt               | QTY          | Y        | Amount of fees denominated in `MiscFeeCurr(138)` asset                                                                                                                                                                                                                                                                       |
| =>138 | MiscFeeCurr              | STRING       | Y        | Currency of miscellaneous fee.                                                                                                                                                                                                                                                                                               |
| =>139 | MiscFeeType              | INT          | Y        | Possible values: <br></br>`4` - EXCHANGE_FEES                                                                                                                                                                                                                                                                                     |
| 1100  | TriggerType              | CHAR         | N        | Possible values: <br></br>`4` - PRICE_MOVEMENT                                                                                                                                                                                                                                                                                    |
| 1101  | TriggerAction            | CHAR         | N        | Possible values: <br></br> `1` - ACTIVATE                                                                                                                                                                                                                                                                                         |
| 1102  | TriggerPrice             | PRICE        | N        | Activation price for contingent orders. See [table](#ordertype)                                                                                                                                                                                                                                                              |
| 1107  | TriggerPriceType         | CHAR         | N        | Possible values: <br></br> `2` - LAST_TRADE                                                                                                                                                                                                                                                                                       |
| 1109  | TriggerPriceDirection    | CHAR         | N        | Used to differentiate between StopLoss and TakeProfit orders. See [table](#ordertype).<br></br>Possible values: <br></br> `U` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_UP_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE <br></br> `D` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_DOWN_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE |
| 25009 | TriggerTrailingDeltaBips | INT          | N        | Appears only for trailing stop orders. |
| 211 | PegOffsetValue | FLOAT | N | Amount added to the peg in the context of the PegOffsetType |
|1094 | PegPriceType | CHAR | N | Defines the type of peg <br> Possible values: <br> `4` - MARKET_PEG <br> `5` - PRIMARY_PEG|
|835 | PegMoveType | CHAR | N | Describes whether peg is fixed or floats. Required for Pegged Orders and must be set to `1` (FIXED) |
|836 | PegOffsetType | CHAR | N | Type of price peg offset. <br> Possible values: <br></br> `3`  - PRICE_TIER|
|839 | PeggedPrice  | PRICE  | N |  Current price the order is pegged at|

**Sample message:**

```
8=FIX.4.4|9=330|35=8|34=2|49=SPOT|52=20240611-09:01:46.228950|56=qNXO12fH|11=1718096506197867067|14=0.00000000|17=144|32=0.00000000|37=76|38=5.00000000|39=0|40=2|44=10.00000000|54=1|55=LTCBNB|59=4|60=20240611-09:01:46.228000|150=0|151=5.00000000|636=Y|1057=Y|25001=1|25017=0.00000000|25018=20240611-09:01:46.228000|25023=20240611-09:01:46.228000|10=095|
```

<a id="ordercancelrequest"></a>

#### OrderCancelRequest `<F>`

Sent by the client to cancel an order or an order list.

* To cancel an order either `OrderID (11)` or `OrigClOrdID (41)` are required.
  * If both `OrderID (37)` and `OrigClOrdID (41)` are provided, the `OrderID` is searched first, then the `OrigClOrdID` from that result is checked against that order. If both conditions are not met the request will be rejected.
* To cancel an order list either `ListID (66)` or `OrigClListID (25015)` are required.
  * If both `ListID (66)` and `OrigClListID (25015)` are provided, the `ListID` is searched first, then the `OrigClListID` from that result is checked against that order. If both conditions are not met the request will be rejected.

If the canceled order is part of an order list, the entire list will be canceled.

**Note:**

* The performance for canceling an order (single cancel or as part of a cancel-replace) is always better when only `orderId` is sent. Sending `origClientOrderId` or both `orderId` + `origClientOrderId` will be slower.

| Tag   | Name               | Type   | Required | Description                                                                                       |
|-------|--------------------|--------|----------|---------------------------------------------------------------------------------------------------|
| 11    | ClOrdID            | STRING | Y        | `ClOrdID` of this request.                                                                        |
| 41    | OrigClOrdID        | STRING | N        | `ClOrdID (11)` of the order to cancel.                                                            |
| 37    | OrderID            | INT    | N        | `OrderID (37)` of the order to cancel.                                                            |
| 25015 | OrigClListID       | STRING | N        | `ClListID (25014)` of the order list to cancel.                                                   |
| 66    | ListID             | STRING | N        | `ListID (66)` of the order list to cancel.                                                        |
| 55    | Symbol             | STRING | Y        | Symbol on which to cancel order.                                                                  |
| 25002 | CancelRestrictions | INT    | N        | Restrictions on the cancel. Possible values: <br></br> `1` - ONLY_NEW <br></br> `2` - ONLY_PARTIALLY_FILLED |

**Sample message:**

```
8=FIX.4.4|9=93|35=F|34=2|49=ieBwvCKy|52=20240613-01:11:13.784|56=SPOT|11=1718241073695674483|37=2|55=LTCBNB|10=210|
```

**Response:**

* [ExecutionReport`<8>`](#executionreport) with `ExecType (150)` value `CANCELED (4)` for each canceled order.
* [ListStatus`<N>`](#liststatus) if orders in an order list were canceled.
* [OrderCancelReject`<9>`](#ordercancelreject) if cancellation was rejected.
* [Reject`<3>`](#reject) if the message is rejected.

<a id="ordercancelreject"></a>

#### OrderCancelReject `<9>`

Sent by the server when [OrderCancelRequest`<F>`](#ordercancelrequest) has failed.

| Tag   | Name               | Type   | Required | Description                                                                                                               |
|-------|--------------------|--------|----------|---------------------------------------------------------------------------------------------------------------------------|
| 11    | ClOrdID            | STRING | Y        | `ClOrdID (11)` of the cancel request.                                                                                     |
| 41    | OrigClOrdID        | STRING | N        | `OrigClOrdID (41)` from the cancel request.                                                                               |
| 37    | OrderID            | INT    | N        | `OrderID (37)`  from the cancel request.                                                                                  |
| 25015 | OrigClListID       | STRING | N        | `OrigClListID (25015)` from the cancel request.                                                                           |
| 66    | ListID             | STRING | N        | `ListID (66)` from the cancel request.                                                                                    |
| 55    | Symbol             | STRING | Y        | `Symbol (55)` from the cancel request.                                                                                    |
| 25002 | CancelRestrictions | INT    | N        | `CancelRestrictions (25002)` from the cancel request.                                                                     |
| 434   | CxlRejResponseTo   | CHAR   | Y        | Type of request that this OrderCancelReject`<9>` is in response to. <br></br> Possible values: <br></br> `1` - ORDER_CANCEL_REQUEST |
| 25016 | ErrorCode          | INT    | Y        | API error code (see [Error Codes](errors.md)).                                                                            |
| 58    | Text               | STRING | Y        | Human-readable error message.                                                                                             |

**Sample message:**

```
8=FIX.4.4|9=137|35=9|34=2|49=SPOT|52=20240613-01:12:41.320869|56=OlZb8ht8|11=1718241161272843932|37=2|55=LTCBNB|58=Unknown order sent.|434=1|25016=-1013|10=087|
```

<a id="ordercancelrequestandnewordersingle"></a>

#### OrderCancelRequestAndNewOrderSingle `<XCN>`

Sent by the client to cancel an order and submit a new one for execution.
* To cancel an order either `OrderID (11)` or `OrigClOrdId (41)` are required.
* If both `OrderID (37)` and `OrigClOrdID (41)` are provided, the `OrderID` is searched first, then the `OrigClOrdID` from that result is checked against that order. If both conditions are not met the request will be rejected.

Filters and Order Count are evaluated before the processing of the cancellation and order placement occurs.

A new order that was not attempted (i.e. when `newOrderResult: NOT_ATTEMPTED`), will still increase the unfilled order count by 1.

**Unfilled Order Count:** 1

Please refer to [Supported Order Types](#ordertype) for supported field combinations when describing the new order.

> [!NOTE]
> Cancel is always processed first. Then immediately after that the new order is submitted.

| Tag   | Name                                    | Type   | Required | Description                                                                                                                                                                                                                                                                                                                  |
|-------|-----------------------------------------|--------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 25033 | OrderCancelRequestAndNewOrderSingleMode | INT    | Y        | What action should be taken if cancel fails. <br></br> Possible values: <br></br> `1` - STOP_ON_FAILURE <br></br> `2` - ALLOW_FAILURE                                                                                                                                                                                                       |
| 25038 | OrderRateLimitExceededMode              | INT    | N        | What should be done to the cancellation request if you exceed the unfilled order rate limit. <br></br>Possible values: `1` - DO_NOTHING <br></br> `2` - CANCEL_ONLY                                                                                                                                                                          |
| 37    | OrderID                                 | INT    | N        | `OrderID` of the order to cancel.                                                                                                                                                                                                                                                                                            |
| 25034 | CancelClOrdID                           | STRING | N        | `ClOrdID` of the cancel.                                                                                                                                                                                                                                                                                                     |
| 41    | OrigClOrdID                             | STRING | N        | `ClOrdID` of the order to cancel.                                                                                                                                                                                                                                                                                            |
| 11    | ClOrdID                                 | STRING | Y        | `ClOrdID` to be assigned to the new order.                                                                                                                                                                                                                                                                                   |
| 25002 | CancelRestrictions                      | INT    | N        | Restrictions on the cancel. Possible values: <br></br> `1` - ONLY_NEW <br></br> `2` - ONLY_PARTIALLY_FILLED                                                                                                                                                                                                                            |
| 38    | OrderQty                                | QTY    | N        | Quantity of the new order                                                                                                                                                                                                                                                                                                    |
| 40    | OrdType                                 | CHAR   | Y        | See the [table](#ordertype) to understand supported order types and the required fields to use them.<br></br>Possible values: <br></br> `1` - MARKET <br></br> `2` - LIMIT <br></br> `3` - STOP <br></br> `4` - STOP_LIMIT <br></br> `P` - PEGGED                                                                                      |
| 18    | ExecInst                                | CHAR   | N        | Possible values: <br></br> `6` - PARTICIPATE_DONT_INITIATE                                                                                                                                                                                                                                                                        |
| 44    | Price                                   | PRICE  | N        | Price of the new order                                                                                                                                                                                                                                                                                                       |
| 54    | Side                                    | CHAR   | Y        | Side of the order.<br></br>Possible values: <br></br> `1` - BUY <br></br> `2` - SELL                                                                                                                                                                                                                                                        |
| 55    | Symbol                                  | STRING | Y        | Symbol to cancel and place the order on.                                                                                                                                                                                                                                                                                     |
| 59    | TimeInForce                             | CHAR   | N        | Possible values: <br></br> `1` - GOOD_TILL_CANCEL <br></br> `3` - IMMEDIATE_OR_CANCEL <br></br> `4` - FILL_OR_KILL                                                                                                                                                                                                                          |
| 111   | MaxFloor                                | QTY    | N        | Used for iceberg orders, this specifies the visible quantity of the order on the book.                                                                                                                                                                                                                                       |
| 152   | CashOrderQty                            | QTY    | N        | Quantity of the order specified in the quote asset units, for reverse market orders.                                                                                                                                                                                                                                         |
| 847   | TargetStrategy                          | INT    | N        | The value cannot be less than `1000000`.                                                                                                                                                                                                                                                                                                                             |
| 7940  | StrategyID                              | INT    | N        |                                                                                                                                                                                                                                                                                      |
| 25001 | SelfTradePreventionMode                 | CHAR   | N        | Possible values: <br></br> `1` - NONE <br></br> `2` - EXPIRE_TAKER <br></br> `3` - EXPIRE_MAKER <br></br> `4` - EXPIRE_BOTH <br></br> `5` - DECREMENT  <br> `6` - TRANSFER                                                                                                                                                                                                                   |
|211 | PegOffsetValue | FLOAT | N | Amount added to the peg in the context of the PegOffsetType |
|1094 | PegPriceType | CHAR | N | Defines the type of peg <br> Possible values: <br> `4` - MARKET_PEG <br> `5` - PRIMARY_PEG|
|835 | PegMoveType | CHAR | N | Describes whether peg is fixed or floats. Required for Pegged Orders and must be set to `1` (FIXED) |
|836 | PegOffsetType | CHAR | N | Type of price peg offset. <br> Possible values: <br></br> `3`  - PRICE_TIER|
| 1100  | TriggerType                             | CHAR   | N        | Possible values: `4` - PRICE_MOVEMENT                                                                                                                                                                                                                                                                                        |
| 1101  | TriggerAction                           | CHAR   | N        | Possible values: <br></br> `1` - ACTIVATE                                                                                                                                                                                                                                                                                         |
| 1102  | TriggerPrice                            | PRICE  | N        | Activation price for contingent orders. See [table](#ordertype)                                                                                                                                                                                                                                                              |
| 1107  | TriggerPriceType                        | CHAR   | N        | Possible values: <br></br> `2` - LAST_TRADE                                                                                                                                                                                                                                                                                       |
| 1109  | TriggerPriceDirection                   | CHAR   | N        | Used to differentiate between StopLoss and TakeProfit orders. See [table](#ordertype).<br></br>Possible values: <br></br> `U` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_UP_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE <br></br> `D` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_DOWN_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE |
| 25009 | TriggerTrailingDeltaBips                | INT    | N        | Provide to create trailing orders.   |


**Sample message:**

```
8=FIX.4.4|9=160|35=XCN|34=2|49=JS8iiXK6|52=20240613-02:31:53.753|56=SPOT|11=1718245913721036458|37=8|38=5|40=2|44=4|54=1|55=LTCBNB|59=1|111=1|25033=1|25034=1718245913721036819|10=229|
```

**Response:**

* [ExecutionReport`<8>`](#executionreport) with `ExecType (150)` value `CANCELED (4)` for the canceled order.
* [ExecutionReport`<8>`](#executionreport) with `ExecType (150)` value `NEW (0)` for the new order.
* [ExecutionReport`<8>`](#executionreport) with `ExecType (150)` value `REJECTED (8)` if the new order was rejected.
* [OrderCancelReject`<9>`](#ordercancelreject) if the cancellation was rejected.
* [Reject`<3>`](#reject) if the message is rejected.

<a id="ordermasscancelrequest"></a>

#### OrderMassCancelRequest `<q>`

Sent by the client to cancel all open orders on a symbol.

> [!NOTE]
> All orders of the account will be canceled, including those placed in different connections.

| Tag | Name                  | Type   | Required | Description                                                                                                    |
|-----|-----------------------|--------|----------|----------------------------------------------------------------------------------------------------------------|
| 11  | ClOrdID               | STRING | Y        | `ClOrdId` of this mass cancel request.                                                                         |
| 55  | Symbol                | STRING | Y        | Symbol on which to cancel orders.                                                                              |
| 530 | MassCancelRequestType | CHAR   | Y        | Possible values: <br></br> `1` - CANCEL_SYMBOL_ORDERS                                                               |

**Sample message:**

```
8=FIX.4.4|9=95|35=q|34=2|49=dpYPesqv|52=20240613-01:24:36.948|56=SPOT|11=1718241876901971671|55=BTCUSDT|530=1|10=243|
```

**Responses:**

* [ExecutionReport`<8>`](#executionreport) with `ExecType (150)` value `CANCELED (4)` for the every order canceled.
* [OrderMassCancelReport`<r>`](#ordermasscancelreport) with `MassCancelResponse (531)` field indicating whether the message is accepted or rejected.
* [Reject`<3>`](#reject) if the message is rejected.

<a id="ordermasscancelreport"></a>

#### OrderMassCancelReport `<r>`

Sent by the server in response to [OrderMassCancelRequest`<q>`](#ordermasscancelrequest).

| Tag   | Name                   | Type   | Required | Description                                                                         |
|-------|------------------------|--------|----------|-------------------------------------------------------------------------------------|
| 55    | Symbol                 | STRING | Y        | `Symbol (55)` from the cancel request.                                              |
| 11    | ClOrdID                | STRING | Y        | `ClOrdID (11)` of the cancel request.                                               |
| 530   | MassCancelRequestType  | CHAR   | Y        | `MassCancelRequestType (530)` from the cancel request.                              |
| 531   | MassCancelResponse     | CHAR   | Y        | Possible values: <br></br> `0` - CANCEL_REQUEST_REJECTED <br></br> `1` - CANCEL_SYMBOL_ORDERS |
| 532   | MassCancelRejectReason | INT    | N        | Possible values: <br></br> `99` - OTHER                                                  |
| 533   | TotalAffectedOrders    | INT    | N        | How many orders were canceled.                                                      |
| 25016 | ErrorCode              | INT    | N        | API error code (see [Error Codes](errors.md)).                                      |
| 58    | Text                   | STRING | N        | Human-readable error message.                                                       |

**Sample message:**

```
8=FIX.4.4|9=109|35=r|34=2|49=SPOT|52=20240613-01:24:36.949763|56=dpYPesqv|11=1718241876901971671|55=LTCBNB|530=1|531=1|533=5|10=083|
```

<a id="neworderlist"></a>

#### NewOrderList `<E>`

Sent by the client to submit a list of orders for execution.

* OCOs or OTOs add **2 orders** to the `EXCHANGE_MAX_ORDERS` filter and the `MAX_NUM_ORDERS` filter.
* OTOCOs add **3 orders** to the `EXCHANGE_MAX_NUM_ORDERS` filter and `MAX_NUM_ORDERS` filter.

**Unfilled Order Count:**
* OCO: 2
* OTO: 2
* OTOCO: 3

Orders in an order list are contingent on one another.
Please refer to [Supported Order List Types](#order-list-types) for supported order types and triggering instructions.

| Tag      | Name                         | Type       | Required | Description                                                                                                                                                                                                                                                                                                                  |
|----------|------------------------------|------------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 25014    | ClListID                     | STRING     | Y        | `ClListID` to be assigned to the order list.                                                                                                                                                                                                                                                                                 |
| 1385     | ContingencyType              | INT        | N        | Possible values: <br></br> `1` - ONE_CANCELS_THE_OTHER <br></br> `2` - ONE_TRIGGERS_THE_OTHER   |
| 25046    | OPO                          | BOOLEAN    | N        | Sets this order list as an [OPO](../faqs/opo.md) when set to `true`. |
| 73       | NoOrders                     | NUMINGROUP | N        | The length of the array for Orders. Only 2 or 3 are allowed.             |
| =>11     | ClOrdID                      | STRING     | Y        | `ClOrdID` to be assigned to the order                                                                                                                                                                                                                                                                                        |
| =>38     | OrderQty                     | QTY        | N        | Quantity of the order                                                                                                                                                                                                                                                                                                        |
| =>40     | OrdType                      | CHAR       | Y        | See the [table](#ordertype) to understand supported order types and the required fields to use them.<br></br>Possible values: <br></br> `1` - MARKET <br></br> `2` - LIMIT <br></br> `3` - STOP <br></br> `4` - STOP_LIMIT  <br></br> `P`- PEGGED                                                                                       |
| =>18     | ExecInst                     | CHAR       | N        | Possible values: <br></br> `6` - PARTICIPATE_DONT_INITIATE                                                                                                                                                                                                                                                                        |
| =>44     | Price                        | PRICE      | N        | Price of the order                                                                                                                                                                                                                                                                                                           |
| =>54     | Side                         | CHAR       | Y        | Side of the order. Possible values: <br></br> `1` - BUY <br></br> `2` - SELL                                                                                                                                                                                                                                                           |
| =>55     | Symbol                       | STRING     | Y        | Symbol to place the order on.                                                                                                                                                                                                                                                                                                |
| =>59     | TimeInForce                  | CHAR       | N        | Possible values: <br></br> `1` - GOOD_TILL_CANCEL <br></br> `3` - IMMEDIATE_OR_CANCEL <br></br> `4` - FILL_OR_KILL                                                                                                                                                                                                                          |
| =>111    | MaxFloor                     | QTY        | N        | Used for iceberg orders, this specifies the visible quantity of the order on the book.                                                                                                                                                                                                                                       |
| =>152    | CashOrderQty                 | QTY        | N        | Quantity of the order specified in the quote asset units, for reverse market orders.                                                                                                                                                                                                                                         |
| =>847    | TargetStrategy               | INT        | N        | The value cannot be less than `1000000`.                                                                                                                                                                                                                                                                                                                             |
| =>7940   | StrategyID                   | INT        | N        |                                                                                                                                                                                                                                                                                      |
| =>25001  | SelfTradePreventionMode      | CHAR       | N        | Possible values: <br></br> `1` - NONE <br></br>`2` - EXPIRE_TAKER <br></br> `3` - EXPIRE_MAKER <br></br> `4` - EXPIRE_BOTH <br></br> `5` - DECREMENT  <br> `6` - TRANSFER                                                                                                                                                                                                                |
|=> 211 | PegOffsetValue | FLOAT | N | Amount added to the peg in the context of the PegOffsetType |
|=>1094 | PegPriceType | CHAR | N | Defines the type of peg <br> Possible values: <br> `4` - MARKET_PEG <br> `5` - PRIMARY_PEG|
|=>835 | PegMoveType | CHAR | N | Describes whether peg is fixed or floats. Required for Pegged Orders and must be set to `1` (FIXED) |
|=>836 | PegOffsetType | CHAR | N | Type of price peg offset. <br> Possible values: <br></br> `3`  - PRICE_TIER|
| =>1100   | TriggerType                  | CHAR       | N        | Possible values: <br></br> `4` - PRICE_MOVEMENT                                                                                                                                                                                                                                                                                   |
| =>1101   | TriggerAction                | CHAR       | N        | Possible values: <br></br> `1` - ACTIVATE                                                                                                                                                                                                                                                                                         |
| =>1102   | TriggerPrice                 | PRICE      | N        | Activation price for contingent orders. See [table](#ordertype)                                                                                                                                                                                                                                                              |
| =>1107   | TriggerPriceType             | CHAR       | N        | Possible values: <br></br> `2` - LAST_TRADE                                                                                                                                                                                                                                                                                       |
| =>1109   | TriggerPriceDirection        | CHAR       | N        | Used to differentiate between StopLoss and TakeProfit orders. See [table](#ordertype).<br></br>Possible values: <br></br> `U` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_UP_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE <br></br> `D` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_DOWN_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE |
| =>25009  | TriggerTrailingDeltaBips     | INT        | N        | Provide to create trailing orders.                                                                                                                                                                                                                                                                                           |
| =>25010  | NoListTriggeringInstructions | NUMINGROUP | N        | The length of the array for ListTriggeringInstructions.       |
| ==>25011 | ListTriggerType              | CHAR       | N        | What needs to happen to the order pointed to by ListTriggerTriggerIndex in order for the action to take place. <br></br> Possible values: <br></br> `1` - ACTIVATED <br></br> `2` - PARTIALLY_FILLED <br></br> `3` - FILLED                                                                                                                      |
| ==>25012 | ListTriggerTriggerIndex      | INT        | N        | Index of the trigger order: 0-indexed.                                                                                                                                                                                                                                                                                       |
| ==>25013 | ListTriggerAction            | CHAR       | N        | Action to take place on this order after the ListTriggerType has been fulfilled. <br></br> Possible values: <br></br> `1` - RELEASE <br></br> `2` - CANCEL|                                                                     |

**Sample message:**

```
8=FIX.4.4|9=236|35=E|34=2|49=Eg13pOvN|52=20240607-02:19:07.836|56=SPOT|73=2|11=w1717726747805308656|55=LTCBNB|54=2|38=1|40=2|44=0.25|59=1|11=p1717726747805308656|55=LTCBNB|54=2|38=1|40=1|25010=1|25011=3|25012=0|25013=1|1385=2|25014=1717726747805308656|10=171|
```

<a id="order-list-types"></a>

#### Supported Order List Types

> [!NOTE]
> Orders must be specified in the sequence indicated in the *Order Names* column in the table below.

| Order list name | Contingency Type (1385) | Order names                                                                                        | Order sides                                                                                                                      | Allowed Binance order types                                                                                                                                                                         | List Triggering Instructions                                                                                                                                                                                                                                                                                                                                                                                    |
|-----------------|-------------------------|----------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| OCO             | `1`                     | 1. below order<br></br><br></br>   2. above order                                                  | 1. below order=`SELL`<br></br><br></br>  2. above order=`SELL`                                                                   | 1. below order=`STOP_LOSS` or `STOP_LOSS_LIMIT`<br></br><br></br> 2. above order=`LIMIT_MAKER`                                                                                                      | 1. below order:  <br></br><code>25010=1&#124;25011=2&#124;25012=1&#124;25013=2&#124;</code><br></br><br></br>2. above order:        <br></br><code>25010=1&#124;25011=1&#124;25012=0&#124;25013=2&#124;</code>                                                                                                                                                                                                  |
| OCO             | `1`                     | 1. below order<br></br><br></br>   2. above order                                                  | 1. below order=`BUY` <br></br><br></br>  2. above order=`BUY`                                                                    | 1. below order=`LIMIT_MAKER`                   <br></br><br></br> 2. above order=`STOP_LOSS` or `STOP_LOSS_LIMIT`                                                                                   | 1. below order:  <br></br><code>25010=1&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code><br></br><br></br>2. above order:        <br></br><code>25010=1&#124;25011=2&#124;25012=0&#124;25013=2&#124;</code>                                                                                                                                                                                                  |
| OCO             | `1`                     | 1. below order<br></br><br></br>   2. above order                                                  | 1. below order=`SELL`<br></br><br></br>  2. above order=`SELL`                                                                   | 1. below order=`STOP_LOSS` or `STOP_LOSS_LIMIT`<br></br><br></br> 2. above order= `TAKE_PROFIT`                                                                                                     | 1. below order:  <br></br><code>25010=1&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code><br></br><br></br>2. above order:        <br></br><code>25010=1&#124;25011=1&#124;25012=0&#124;25013=2&#124;</code>                                                                                                                                                                                                  |
| OCO             | `1`                     | 1. below order<br></br><br></br>   2. above order                                                  | 1. below order=`BUY` <br></br><br></br>  2. above order=`BUY`                                                                    | 1. below order=`TAKE_PROFIT`                   <br></br><br></br> 2. above order = `STOP_LOSS` or `STOP_LOSS_LIMIT`                                                                                 | 1. below order:  <br></br><code>25010=1&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code><br></br><br></br>2. above order:        <br></br><code>25010=1&#124;25011=1&#124;25012=0&#124;25013=2&#124;</code>                                                                                                                                                                                                  |
| OTO             | `2`                     | 1. working order<br></br><br></br> 2. pending order                                                | 1. working order=`SELL` or `BUY`<br></br><br></br> 2. pending order=`SELL` or `BUY`                                              | 1. working order=`LIMIT` or `LIMIT_MAKER`      <br></br><br></br> 2. pending order=ANY                                                                                                              | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending order:      <br></br><code>25010=1&#124;25011=3&#124;25012=0&#124;25013=1&#124;</code>                                                                                                                                                                                                  |
| OTOCO           | `2`                     | 1. working order<br></br><br></br> 2. pending below order<br></br><br></br> 3. pending above order | 1. working order=`SELL` or `BUY`<br></br><br></br> 2. pending below order=`SELL`<br></br><br></br> 3. pending above order=`SELL` | 1. working order=`LIMIT` or `LIMIT_MAKER`      <br></br><br></br> 2. pending below order=`STOP_LOSS` or `STOP_LOSS_LIMIT`<br></br><br></br> 3. pending above order=`LIMIT_MAKER`                    | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending below order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=2&#124;25012=2&#124;25013=2&#124;</code><br></br><br></br>3. pending above order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code> |
| OTOCO           | `2`                     | 1. working order<br></br><br></br> 2. pending below order<br></br><br></br> 3. pending above order | 1. working order=`SELL` or `BUY`<br></br><br></br> 2. pending below order=`BUY`<br></br><br></br>  3. pending above order=`BUY`  | 1. working order=`LIMIT` or `LIMIT_MAKER`      <br></br><br></br> 2. pending below order=`LIMIT_MAKER`                   <br></br><br></br> 3. pending above order=`STOP_LOSS` or `STOP_LOSS_LIMIT` | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending below order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=2&#124;25013=2&#124;</code><br></br><br></br>3. pending above order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=2&#124;25012=1&#124;25013=2&#124;</code> |
| OTOCO           | `2`                     | 1. working order<br></br><br></br> 2. pending below order<br></br><br></br> 3. pending above order | 1. working order=`SELL` or `BUY`<br></br><br></br> 2. pending below order=`SELL`<br></br><br></br> 3. pending above order=`SELL` | 1. working order=`LIMIT` or `LIMIT_MAKER`      <br></br><br></br> 2. pending below order=`STOP_LOSS` or `STOP_LOSS_LIMIT`<br></br><br></br> 3. pending above order=`TAKE_PROFIT`                    | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending below order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=2&#124;25013=2&#124;</code><br></br><br></br>3. pending above order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code> |
| OTOCO           | `2`                     | 1. working order<br></br><br></br> 2. pending below order<br></br><br></br> 3. pending above order | 1. working order=`SELL` or `BUY`<br></br><br></br> 2. pending below order=`BUY`<br></br><br></br>  3. pending above order=`BUY`  | 1. working order=`LIMIT` or `LIMIT_MAKER`      <br></br><br></br> 2. pending below order=`TAKE_PROFIT`                   <br></br><br></br> 3. pending above order=`STOP_LOSS` or `STOP_LOSS_LIMIT` | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending below order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=2&#124;25013=2&#124;</code><br></br><br></br>3. pending above order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code> |
| OPO             | `2`                     | 1. working order<br></br><br></br> 2. pending order                                                | 1. working order=`BUY`<br></br><br></br> 2. pending order=`SELL`                                            | 1. working order=`LIMIT` or `LIMIT_MAKER`      <br></br><br></br> 2. pending order=ANY                                                                                                              | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending order:      <br></br><code>25010=1&#124;25011=3&#124;25012=0&#124;25013=1&#124;</code>
| OPOCO           | `2`                     | 1. working order<br></br><br></br> 2. pending below order<br></br><br></br> 3. pending above order | 1. working order=`BUY`<br></br><br></br> 2. pending below order=`SELL`<br></br><br></br> 3. pending above order=`SELL` | 1. working order=`LIMIT` or `LIMIT_MAKER`      <br></br><br></br> 2. pending below order=`STOP_LOSS` or `STOP_LOSS_LIMIT`<br></br><br></br> 3. pending above order=`LIMIT_MAKER`                    | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending below order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=2&#124;25012=2&#124;25013=2&#124;</code><br></br><br></br>3. pending above order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code> |
| OPOCO           | `2`                     | 1. working order<br></br><br></br> 2. pending below order<br></br><br></br> 3. pending above order | 1. working order=`BUY`<br><br> 2. pending below order=`SELL`<br></br><br></br> 3. pending above order=`SELL` | 1. working order=`LIMIT` or `LIMIT_MAKER`      <br></br><br></br> 2. pending below order=`STOP_LOSS` or `STOP_LOSS_LIMIT`<br></br><br></br> 3. pending above order=`TAKE_PROFIT` or `TAKE_PROFIT_LIMIT`  | 1. working order:<br></br>NONE<br></br><br></br>                                                             2. pending below order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=2&#124;25013=2&#124;</code><br></br><br></br>3. pending above order:<br></br><code>25010=2&#124;25011=3&#124;25012=0&#124;25013=2&#124;25011=1&#124;25012=1&#124;25013=2&#124;</code> |

<a id="liststatus"></a>

#### ListStatus `<N>`

Sent by the server whenever an order list state changes.

> [!NOTE]
> By default, ListStatus`<N>` is sent for all order lists of an account, including those submitted in different connections.
> Please see [Response Mode](#responsemode) for other behavior options.

| Tag      | Name                         | Type         | Required | Description                                                                                                                                             |
|----------|------------------------------|--------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| 55       | Symbol                       | STRING       | N        | Symbol of the order list.                                                                                                                               |
| 66       | ListID                       | STRING       | N        | `ListID` of the list as assigned by the exchange.                                                                                                       |
| 25014    | ClListID                     | STRING       | N        | `ClListID` of the list as assigned on the request.                                                                                                      |
| 25015    | OrigClListID                 | STRING       | N        |                                                                                                                                                         |
| 1385     | ContingencyType              | INT          | N        | Possible values: <br></br> `1` - ONE_CANCELS_THE_OTHER <br></br> `2` - ONE_TRIGGERS_THE_OTHER                                                                     |
| 429      | ListStatusType               | INT          | Y        | Possible values: <br></br> `2` - RESPONSE <br></br>`4` - EXEC_STARTED <br></br> `5` - ALL_DONE <br></br> `100` - UPDATED                                                                        |
| 431      | ListOrderStatus              | INT          | Y        | Possible values: <br></br> `3` - EXECUTING <br></br> `6` - ALL_DONE  <br></br> `7` - REJECT                                                                            |
| 1386     | ListRejectReason             | INT          | N        | Possible values: <br></br> `99` - OTHER                                                                                                                      |
| 103      | OrdRejReason                 | INT          | N        | Possible values: <br></br> `99` - OTHER                                                                                                                      |
| 60       | TransactTime                 | UTCTIMESTAMP | N        | Timestamp when this event occurred.                                                                                                                     |
| 25016    | ErrorCode                    | INT          | N        | API error code (see [Error Codes](errors.md)).                                                                                                          |
| 58       | Text                         | STRING       | N        | Human-readable error message.                                                                                                                           |
| 73       | NoOrders                     | NUMINGROUP   | N        | The length of the array for Orders.                                                                                                           |
| =>55     | Symbol                       | STRING       | Y        | Symbol of the order.                                                                                                                                    |
| =>37     | OrderID                      | INT          | Y        | `OrderID` of the order as assigned by the exchange.                                                                                                     |
| =>11     | ClOrdID                      | STRING       | Y        | `ClOrdID` of the order as assigned on the request.                                                                                                      |
| =>25010  | NoListTriggeringInstructions | NUMINGROUP   | N        | The length of the array for ListTriggeringInstructions.                                                                                          |
| ==>25011 | ListTriggerType              | CHAR         | N        | Possible values: <br></br> `1` - ACTIVATED <br></br> `2` - PARTIALLY_FILLED <br></br> `3` - FILLED                                                                     |
| ==>25012 | ListTriggerTriggerIndex      | INT          | N        |                                                                                                                                                         |
| ==>25013 | ListTriggerAction            | CHAR         | N        | Possible values: <br></br> `1` - RELEASE <br></br> `2` - CANCEL                                                                                                   |

**Sample message:**

```
8=FIX.4.4|9=293|35=N|34=2|49=SPOT|52=20240607-02:19:07.837191|56=Eg13pOvN|55=BTCUSDT|60=20240607-02:19:07.836000|66=25|73=2|55=BTCUSDT|37=52|11=w1717726747805308656|55=BTCUSDT|37=53|11=p1717726747805308656|25010=1|25011=3|25012=0|25013=1|429=4|431=3|1385=2|25014=1717726747805308656|25015=1717726747805308656|10=162|
```

<a id="orderamendkeeppriorityrequest"></a>

#### OrderAmendKeepPriorityRequest `<XAK>`

Sent by the client to reduce the original quantity of their order.

This adds 0 orders to the `EXCHANGE_MAX_ORDERS` filter and the `MAX_NUM_ORDERS` filter.

**Unfilled Order Count:** 0

Read [Order Amend Keep Priority FAQ](faqs/order_amend_keep_priority.md) to learn more.

**Notes:**

* The `ClOrdID (11)` is not required to be different from the `ClOrdID` of the order. When the `ClOrdID` of the request is the same as the `ClOrdID` of the order being amended, the `ClOrdID` will remain unchanged.
* If both `OrderID (37)` and `OrigClOrdID (41)` are provided, the `OrderID` is searched first, then the `OrigClOrdID (41)` from that result is checked against that order. If both conditions are not met the request will be rejected.

| Tag | Name | Type | Required | Description |
| :---- | :---- | :---- | :---- | ----- |
| 11 | ClOrdID | STRING | Y | The ClOrdID of this request.  |
| 41 | OrigClOrdID | STRING | N | `ClOrdID (11)` of the order to amend. Either `OrigClOrdID (41)` or `OrderId (37)` have to be specified. |
| 37 | OrderID | INT | N | `OrderID (37)` of the order to amend. Either `OrigClOrdID (41)` or `OrderId (37)` have to be specified. |
| 55 | Symbol  | STRING | Y | Symbol on which to amend the order. |
| 38 | OrderQty | QTY | N | New quantity of the order. Required to be smaller than the original OrderQty of the order. |

**Sample message:**

```
8=FIX.4.4|9=103|35=XAK|34=2|49=EXAMPLE|52=20250319-12:35:21.087|56=SPOT|11=O2EIAS01742387721086|37=0|38=0.9|55=BTCUSDT|10=254|
```

**Response:**

* [Reject `<3>`](#reject) if the incoming request is invalid either due to missing required fields, invalid fields, refers to an invalid symbol, or exceeds the message limit.
* [OrderAmendReject `<XAR>`](#orderamendreject) if failed due to insufficient order rate limits, pointing to a non-existent order, quantity is invalid, etc.
* [ExecutionReport `<8>`](#executionReport) if the request succeeded for amending a single order.
* [ExecutionReport `<8>`](#executionReport) \+ [ListStatus `<N>`](#liststatus) if the request succeeded for amending an order which is part of an Order list.

<a id="orderamendreject"></a>

### OrderAmendReject `<XAR>`

Sent by the server when the OrderAmendKeepPriorityRequest `<XAK>` has failed.

| Tag | Name | Type | Required | Description |
| :---- | :---- | :---- | :---- | :---- |
| 11 | ClOrdID | STRING | Y | `ClOrdId` of the amend request. |
| 41 | OrigClOrdID | STRING | N | `OrigClOrdId` (41) from the amend request. |
| 37 | OrderID | INT | N | `OrderId (37)` from the amend request. |
| 55 | Symbol | STRING | Y | `Symbol (55)` from the amend request. |
| 38 | OrderQty | QTY | Y |  |
| 25016 | ErrorCode | INT | Y | API error code (see [Error Codes](errors.md)). |
| 58 | Text | STRING | Y | Human-readable error message. |

**Sample message:**

```
8=FIX.4.4|9=0000176|35=XAR|49=SPOT|56=OE|34=2|52=20250319-14:27:32.751074|11=1WRGW5J1742394452749|37=0|55=BTCUSDT|38=1.000000|25016=-2038|58=The requested action would change no state; rejecting.|10=235|
```

### Limit Messages

<a id="limitquery"></a>

#### LimitQuery `<XLQ>`

Sent by the client to query current limits.

| Tag  | Name  | Type   | Required | Description        |
|------|-------|--------|----------|--------------------|
| 6136 | ReqID | STRING | Y        | ID of this request |

**Sample message:**

```
8=FIX.4.4|9=82|35=XLQ|34=2|49=7buKHZxZ|52=20240614-05:35:35.357|56=SPOT|6136=1718343335357229749|10=170|
```

<a id="limitresponse"></a>

#### LimitResponse `<XLR>`

Sent by the server in response to [LimitQuery`<XLQ>`](#limitquery).

| Tag     | Name                         | Type       | Required | Description                                                                                                            |
|---------|------------------------------|------------|----------|------------------------------------------------------------------------------------------------------------------------|
| 6136    | ReqID                        | STRING     | Y        | `ReqID` from the request.                                                                                              |
| 25003   | NoLimitIndicators            | NUMINGROUP | Y        | The length of the array for LimitIndicators.                                                                      |
| =>25004 | LimitType                    | CHAR       | Y        | Possible values: <br></br> `1` - ORDER_LIMIT <br></br> `2` - MESSAGE_LIMIT                                                       |
| =>25005 | LimitCount                   | INT        | Y        | The current use of this limit.                                                                                         |
| =>25006 | LimitMax                     | INT        | Y        | The maximum allowed for this limit.                                                                                    |
| =>25007 | LimitResetInterval           | INT        | N        | How often the limit resets.                                                                                            |
| =>25008 | LimitResetIntervalResolution | CHAR       | N        | Time unit of `LimitResetInterval`. Possible values: <br></br> `s` - SECOND <br></br> `m` - MINUTE <br></br> `h` - HOUR <br></br> `d` - DAY |

**Sample message:**

```
8=FIX.4.4|9=225|35=XLR|34=2|49=SPOT|52=20240614-05:42:42.724057|56=uGnG0ef8|6136=1718343762723730315|25003=3|25004=2|25005=1|25006=1000|25007=10|25008=s|25004=1|25005=0|25006=200|25007=10|25008=s|25004=1|25005=0|25006=200000|25007=1|25008=d|10=241|
```

### Market Data Messages

> [!NOTE]
> The messages below can only be used for the FIX Market Data.

<a id="instrumentlistrequest"></a>

#### InstrumentListRequest `<x>`

Sent by the client to query information about instruments.

| Tag | Name                      | Type   | Required | Description                                                                        |
|-----|---------------------------|--------|----------|------------------------------------------------------------------------------------|
| 320 | InstrumentReqID           | STRING | Y        | ID of this request                                                                 |
| 559 | InstrumentListRequestType | INT    | Y        | Possible values: <br></br> `0` - SINGLE_INSTRUMENT <br></br> `4` - ALL_INSTRUMENTS |
| 55  | Symbol                    | STRING | N        | Required when the `InstrumentListRequestType` is set to `SINGLE_INSTRUMENT(0)`     |

**Sample message:**

```
8=FIX.4.4|9=92|35=x|49=BMDWATCH|56=SPOT|34=2|52=20250114-08:46:56.096691|320=BTCUSDT_INFO|559=0|55=BTCUSDT|10=164|
```

<a id="instrumentlist"></a>

#### InstrumentList `<y>`

Sent by the server in a response to the [InstrumentListRequest`<x>`](#instrumentlistrequest).

> [!NOTE]
> More detailed symbol information is available through the [exchangeInfo](https://github.com/binance/binance-spot-api-docs/blob/master/testnet/rest-api.md#exchange-information) endpoint.


| Tag     | Name                  | Type       | Required | Description                                |
|---------|-----------------------|------------|----------|--------------------------------------------|
| 320     | InstrumentReqID       | STRING     | Y        | `InstrumentReqID` from the request.        |
| 146     | NoRelatedSym          | NUMINGROUP | Y        | Number of symbols                          |
| =>55    | Symbol                | STRING     | Y        |                                            |
| =>15    | Currency              | STRING     | Y        | Quote asset of this symbol                 |
| =>562   | MinTradeVol           | QTY        | N        | Corresponds to the [LOT_SIZE](filters.md#lot_size) filter|
| =>1140  | MaxTradeVol           | QTY        | N        | Corresponds to the [LOT_SIZE](filters.md#lot_size) filter|
| =>25039 | MinQtyIncrement       | QTY        | N        | Corresponds to the [LOT_SIZE](filters.md#lot_size) filter|
| =>25040 | MarketMinTradeVol     | QTY        | N        | Corresponds to the [MARKET_LOT_SIZE](filters.md#market_lot_size) filter|
| =>25041 | MarketMaxTradeVol     | QTY        | N        | Corresponds to the [MARKET_LOT_SIZE](filters.md#market_lot_size) filter|
| =>25042 | MarketMinQtyIncrement | QTY        | N        | Corresponds to the [MARKET_LOT_SIZE](filters.md#market_lot_size) filter|
| =>969   | MinPriceIncrement     | PRICE      | N        | Corresponds to the [PRICE](filters.md#price) filter|
| =>2551  | StartPriceRange       | PRICE      | N        | Corresponds to the [PRICE](filters.md#price) filter|
| =>2552  | EndPriceRange         | PRICE      | N        | Corresponds to the [PRICE](filters.md#price) filter|

**Sample message:**

```
8=FIX.4.4|9=218|35=y|49=SPOT|56=BMDWATCH|34=2|52=20250114-08:46:56.100147|320=BTCUSDT_INFO|146=1|55=BTCUSDT|15=USDT|562=0.00001000|1140=9000.00000000|25039=0.00001000|25040=0.00000001|25041=76.79001236|25042=0.00000001|969=0.01000000|10=093|
```

<a id="marketdatarequest"></a>

#### MarketDataRequest `<V>`

Sent by the client to subscribe to or unsubscribe from market data stream.

<a id="tradestream"></a>

**Trade Stream**

The Trade Streams push raw trade information; each trade has a unique buyer and seller.

**Fields required to subscribe:**

- `SubscriptionRequestType` present with value `SUBSCRIBE(1)`
- `MDEntryType` present with value `TRADE(2)`

**Update Speed:** Real-time

<a id="symbolbooktickerstream"></a>

**Individual Symbol Book Ticker Stream**

Pushes any update to the best bid or offers price or quantity in real-time for a specified symbol.

**Fields required to subscribe:**

- `SubscriptionRequestType` with value `SUBSCRIBE(1)`
- `MDEntryType` with value `BID(0)`
- `MDEntryType` with value `OFFER(1)`
- `MarketDepth` with value `1`

**Update Speed:** Real-time

> [!NOTE]
> In the [Individual Symbol Book Ticker Stream](#symbolbooktickerstream), when `MDUpdateAction` is set to `CHANGE(1)` in a
> [MarketDataIncrementalRefresh`<X>`](#marketdataincrementalrefresh) message sent from the server, it replaces the previous best quote.

<a id="diffdepthstream"></a>

**Diff. Depth Stream**

Order book price and quantity depth updates used to locally manage an order book.

**Fields required to subscribe:**

- `SubscriptionRequestType` with value `SUBSCRIBE(1)`
- `MDEntryType` with value `BID(0)`
- `MDEntryType` with value `OFFER(1)`
- `MarketDepth` with a value between `2` and `5000`, which controls the size of the initial snapshot and has no effect on subsequent [MarketDataIncrementalRefresh`<X>`](#marketdataincrementalrefresh) messages

**Update Speed:** 100ms

> [!NOTE]
> Since the [MarketDataSnapshot`<W>`](#marketdatasnapshot) have a limit on the number of price levels (5000 on each side maximum), you won't learn the quantities for the levels outside of the initial snapshot unless they change.
> So be careful when using the information for those levels, since they might not reflect the full view of the order book.
> However, for most use cases, seeing 5000 levels on each side is enough to understand the market and trade effectively.

| Tag   | Name                    | Type       | Required | Description                                                                                                                       |
|:------|-------------------------|------------|----------|-----------------------------------------------------------------------------------------------------------------------------------|
| 262   | MDReqID                 | STRING     | Y        | ID of this request                                                                                                                |
| 263   | SubscriptionRequestType | CHAR       | Y        | Subscription Request Type.  Possible values: <br></br> `1` - SUBSCRIBE <br></br> `2` - UNSUBSCRIBE                                |
| 264   | MarketDepth             | INT        | N        | Subscription depth. <br></br> Possible values: <br></br> `1` - Book Ticker subscription <br></br> `2`-`5000` - Diff. Depth Stream |
| 266   | AggregatedBook          | NUMINGROUP | N        | Possible values: <br></br> `Y` - one book entry per side per price                                                                |
| 146   | NoRelatedSym            | NUMINGROUP | N        | Number of symbols                                                                                                                 |
| =>55  | Symbol                  | STRING     | Y        |                                                                                                                                   |
| 267   | NoMDEntryTypes          | NUMINGROUP | N        | Number of entry types                                                                                                             |
| =>269 | MDEntryType             | CHAR       | Y        | Possible values: <br></br> `0` - BID <br></br> `1` - OFFER <br></br> `2` - TRADE                                                  |

**Sample message:**

```
# Subscriptions
# BOOK TICKER Stream
8=FIX.4.4|9=132|35=V|49=TRADER1|56=SPOT|34=4|52=20241122-06:17:14.183428|262=BOOK_TICKER_STREAM|263=1|264=1|266=Y|146=1|55=BTCUSDT|267=2|269=0|269=1|10=010|

# DEPTH Stream
8=FIX.4.4|9=127|35=V|49=TRADER1|56=SPOT|34=7|52=20241122-06:17:14.443822|262=DEPTH_STREAM|263=1|264=10|266=Y|146=1|55=BTCUSDT|267=2|269=0|269=1|10=111|

# TRADE Stream
8=FIX.4.4|9=120|35=V|49=TRADER1|56=SPOT|34=3|52=20241122-06:34:14.775606|262=TRADE_STREAM|263=1|264=1|266=Y|146=1|55=BTCUSDT|267=1|269=2|10=040|

# Unsubscription from TRADE Stream
8=FIX.4.4|9=79|35=V|49=TRADER1|56=SPOT|34=7|52=20241122-06:41:56.966969|262=TRADE_STREAM|263=2|264=1|10=113|
```

<a id="marketdatarequestreject"></a>

### MarketDataRequestReject `<Y>`

Sent by the server in a response to an invalid MarketDataRequest `<V>`.

| Tag   | Name           | Type   | Required | Description                                                                               |
|-------|----------------|--------|----------|-------------------------------------------------------------------------------------------|
| 262   | MDReqID        | STRING | Y        | ID of the invalid [MarketDataRequest`<V>`](#marketdatarequest)                            |
| 281   | MDReqRejReason | CHAR   | N        | Possible values: <br></br> `1` - DUPLICATE_MDREQID <br></br> `2` - TOO_MANY_SUBSCRIPTIONS |
| 25016 | ErrorCode      | INT    | N        | API Error code. See [Errors](errors.md)                                                   |
| 58    | Text           | STRING | N        | Human-readable error message.                                                             |

**Sample message:**

```
8=FIX.4.4|9=0000218|35=Y|49=SPOT|56=EXAMPLE|34=5|52=20241019-05:39:36.688964|262=BOOK_TICKER_2|281=2|25016=-1191|58=Similar subscription is already active on this connection. Symbol='BNBBUSD', active subscription id: 'BOOK_TICKER_1'.|10=137|
```

<a id="marketdatasnapshot"></a>

### MarketDataSnapshot `<W>`

Sent by the server in response to a [MarketDataRequest`<V>`](#marketdatarequest), activating [Individual Symbol Book Ticker Stream](#symbolbooktickerstream) or [Diff. Depth Stream](#diffdepthstream) subscriptions.

| Tag   | Name             | Type       | Required | Description                                                                             |
|-------|------------------|------------|----------|-----------------------------------------------------------------------------------------|
| 262   | MDReqID          | STRING     | Y        | ID of the [MarketDataRequest`<V>`](#marketdatarequest) that activated this subscription |
| 55    | Symbol           | STRING     | Y        |                                                                                         |
| 25044 | LastBookUpdateID | INT        | N        |                                                                                         |
| 268   | NoMDEntries      | NUMINGROUP | Y        | Number of entries                                                                       |
| =>269 | MDEntryType      | CHAR       | Y        | Possible values: <br></br> `0` - BID <br></br> `1` - OFFER <br></br> `2` - TRADE        |
| =>270 | MDEntryPx        | PRICE      | Y        | Price                                                                                   |
| =>271 | MDEntrySize      | QTY        | Y        | Quantity                                                                                |

**Sample message:**

```
8=FIX.4.4|9=0000107|35=W|49=SPOT|56=EXAMPLE|34=34|52=20241019-05:41:52.867164|262=BOOK_TICKER_1_2|55=BNBBUSD|25044=0|268=0|10=151|
```

<a id="marketdataincrementalrefresh"></a>

### MarketDataIncrementalRefresh `<X>`

Sent by the server when there is a change in a subscribed stream.

| Tag     | Name              | Type         | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                  |
|---------|-------------------|--------------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 262     | MDReqID           | STRING       | Y        | ID of the [MarketDataRequest`<V>`](#marketdatarequest) that activated this subscription                                                                                                                                                                                                                                                                                                                                      |
| 893     | LastFragment      | BOOLEAN      | N        | When present, this indicates that the message was fragmented. Fragmentation may occur when `NoMDEntry` would exceed 10000 in a single [MarketDataIncrementalRefresh`<X>`](#marketdataincrementalrefresh), in order to limit it to 10000. The fragments of a fragmented message are guaranteed to be consecutive in the stream. It can only appear in the [Trade Stream](#tradestream) and [Diff. Depth Stream](#diffdepthstream). |
| 268     | NoMDEntries       | NUMINGROUP   | Y        | Number of entries                                                                                                                                                                                                                                                                                                                                                                                                            |
| =>279   | MDUpdateAction    | CHAR         | Y        | Possible values: <br></br> `0` - NEW <br></br> `1` - CHANGE <br></br> `2` - DELETE                                                                                                                                                                                                                                                                                                                                           |
| =>270   | MDEntryPx         | PRICE        | Y        | Price                                                                                                                                                                                                                                                                                                                                                                                                                        |
| =>271   | MDEntrySize       | QTY          | N        | Quantity                                                                                                                                                                                                                                                                                                                                                                                                                     |
| =>269   | MDEntryType       | CHAR         | Y        | Possible values: <br></br> `0` - BID <br></br> `1` - OFFER <br></br> `2` - TRADE                                                                                                                                                                                                                                                                                                                                             |
| =>55    | Symbol            | STRING       | N        | Market Data Entry will default to the same `Symbol` of the previous Market Data Entry in the same Market Data message if `Symbol` is not specified                                                                                                                                                                                                                                                                           |
| =>60    | TransactTime      | UTCTIMESTAMP | N        |                                                                                                                                                                                                                                                                                                                                                                                                                              |
| =>1003  | TradeID           | INT          | N        |                                                                                                                                                                                                                                                                                                                                                                                                                              |
| =>2446  | AggressorSide     | CHAR         | N        | Possible values: <br></br> `1` - BUY <br></br> `2` - SELL |
| =>25043 | FirstBookUpdateID | INT          | N        | Only present in [Diff. Depth Stream](#diffdepthstream). <br></br> Market Data Entry will default to the same `FirstBookUpdateID` of the previous Market Data Entry in the same Market Data message if `FirstBookUpdateID` is not specified                                                                                                                                                                                   |
| =>25044 | LastBookUpdateID  | INT          | N        | Only present in [Diff. Depth Stream](#diffdepthstream) and [Individual Symbol Book Ticker Stream](#symbolbooktickerstream). <br></br> Market Data Entry will default to the same `LastBookUpdateID` of the previous Market Data Entry in the same Market Data message if `LastBookUpdateID` is not specified                                                                                                                 |

**Sample message:**

```
8=FIX.4.4|9=0000313|35=X|49=SPOT|56=EXAMPLE|34=16|52=20241019-05:40:11.466313|262=TRADE_3|893=N|268=3|279=0|269=2|270=10.00000|271=0.01000|55=BNBBUSD|1003=0|60=20241019-05:40:11.464000|279=0|269=2|270=10.00000|271=0.01000|1003=1|60=20241019-05:40:11.464000|279=0|269=2|270=10.00000|271=0.01000|1003=2|60=20241019-05:40:11.464000|10=125|
```

**Sample fragmented messages:**

> [!NOTE]
> Below are example messages, with `NoMDEntry` limited to *2*, In the real streams, the `NoMDEntry` is limited to *10000*.

[Trade Stream](#tradestream)
```
8=FIX.4.4|9=237|35=X|34=114|49=SPOT|52=20250116-19:36:44.544549|56=EXAMPLE|262=id|268=2|279=0|270=240.00|271=3.00000000|269=2|55=BNBBUSD|60=20250116-19:36:44.196569|1003=67|279=0|270=238.00|271=2.00000000|269=2|60=20250116-19:36:44.196569|1003=68|893=N|10=180|
8=FIX.4.4|9=163|35=X|34=115|49=SPOT|52=20250116-19:36:44.544659|56=EXAMPLE|262=id|268=1|279=0|270=233.00|271=1.00000000|269=2|55=BNBBUSD|60=20250116-19:36:44.196569|1003=69|893=Y|10=243|
```

[Diff. Depth Stream](#diffdepthstream)
```
8=FIX.4.4|9=156|35=X|34=12|49=SPOT|52=20250116-19:45:31.774162|56=EXAMPLE|262=id|268=2|279=2|270=362.00|269=0|55=BNBBUSD|25043=1143|25044=1145|279=2|270=313.00|269=0|893=N|10=047|
8=FIX.4.4|9=171|35=X|34=13|49=SPOT|52=20250116-19:45:31.774263|56=EXAMPLE|262=id|268=2|279=2|270=284.00|269=0|55=BNBBUSD|25043=1143|25044=1145|279=1|270=264.00|271=3.00000000|269=0|893=N|10=239|
8=FIX.4.4|9=149|35=X|34=14|49=SPOT|52=20250116-19:45:31.774281|56=EXAMPLE|262=id|268=1|279=1|270=395.00|271=19.00000000|269=1|55=BNBBUSD|25043=1143|25044=1145|893=Y|10=024|
```

## FIX SBE

FIX SBE (Simple Binary Encoding) can be used instead of FIX with the [spot_fix_testnet_latest.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_fix_testnet_latest.xml) schema file.

### SBE

Read the [SBE FAQ](../faqs/sbe_faq.md) for important information about using SBE with Binance APIs.

* Please review and understand the [SBE specification](https://www.fixtrading.org/standards/sbe-online/) before attempting to use FIX SBE
* When encoding and decoding SBE payloads, it is recommended to use code generated by [`SbeTool`](https://github.com/aeron-io/simple-binary-encoding) to ensure compliance with the FIX SBE specification.

### Endpoints

In addition to FIX encoding available on port 9000, two request/response encoding schemes are supported on additional TCP ports. See below endpoints for each API.

#### Order Entry

* `tcp+tls://fix-oe.testnet.binance.vision:9001`: Send FIX requests; receive FIX SBE responses
    * FIX `SbeSchemaId` tag (=25050) must be set to the FIX SBE schema ID (=1)
    * The FIX `SbeSchemaVersion` tag (=25051) must be set to the FIX SBE schema version (=0)
* `tcp+tls://fix-oe.testnet.binance.vision:9002`: Send FIX SBE requests; receive FIX SBE responses

#### Drop Copy

* `tcp+tls://fix-dc.testnet.binance.vision:9001`: Send FIX requests; receive FIX SBE responses
    * FIX `SbeSchemaId` tag (=25050) must be set to the FIX SBE schema ID (=1)
    * The FIX `SbeSchemaVersion` tag (=25051) must be set to the FIX SBE schema version (=0)
* `tcp+tls://fix-dc.testnet.binance.vision:9002`: Send FIX SBE requests; receive FIX SBE responses

#### Market data

* `tcp+tls://fix-md.testnet.binance.vision:9001`: Send FIX requests; receive FIX SBE responses
    * FIX `SbeSchemaId` tag (=25050) must be set to the FIX SBE schema ID (=1)
    * The FIX `SbeSchemaVersion` tag (=25051) must be set to the FIX SBE schema version (=0)
* `tcp+tls://fix-md.testnet.binance.vision:9002`: Send FIX SBE requests; receive FIX SBE responses

### FIX SBE encoding layout

FIX SBE request/response messages always come with a SOFH (Simple Open Framing Header) and message header. A given FIX SBE message of N bytes has the following wire format:

`<SOFH (6 bytes)> <message header (20 bytes)> <message (N bytes)>`

SOFH: This corresponds to the "sofh" composite type in the schema file. This acts as a framing header so that the FIX SBE servers/clients can know the length of SBE messages and ensure messages have been fully received prior to deserializing them

Notes:
- The two fields within the SOFH MUST be encoded in little-endian
- The FIX servers only support `0xEB50` for the encodingType field, i.e. only little-endian is supported for all fields

Message header: This corresponds to the "messageHeader" composite type in the schema file.

### Logon

The logon signature (RawData) is computed as documented in the [signature computation](#signaturecomputation) section.

#### Sample FIX SBE `Logon` request message

Please see below the hexdump of a sample FIX SBE `Logon` message obtained by following the above instructions.


| Bytes                                           | Description                 |
|-------------------------------------------------|-----------------------------|
| 0xd1, 0x00, 0x00, 0x00                          | sofh.messageLength          |
| 0x50, 0xeb                                      | sofh.encodingType           |
| 0x0e, 0x00                                      | messageHeader.blockLength   |
| 0x28, 0x4e                                      | messageHeader.templateId    |
| 0x01, 0x00                                      | messageHeader.schemaId      |
| 0x00, 0x00                                      | messageHeader.version       |
| 0x01, 0x00, 0x00, 0x00                          | messageHeader.seqNum        |
| 0x58, 0x7a, 0x5f, 0x99, 0xdb, 0x1b, 0x06, 0x00  | messageHeader.sendingTime   |
| 0x00                                            | Logon.EncryptMethod         |
| 0x1e, 0x00, 0x00, 0x00                          | Logon.HeartBtInt            |
| 0x01                                            | Logon.ResetSeqNumFlag       |
| 0x02                                            | Logon.MessageHandling       |
| 0xff                                            | Logon.ResponseMode          |
| 0xff                                            | Logon.ExecutionReportType   |
| 0xff                                            | Logon.DropCopyFlag          |
| 0xff, 0xff, 0xff, 0xff                          | Logon.RecvWindow            |
| 0x07                                            | Logon.SenderCompId.length   |
| 0x45, 0x58, 0x41, 0x4d, 0x50, 0x4c, 0x45        | Logon.SenderCompId.varData  |
| 0x04                                            | Logon.TargetCompId.length   |
| 0x53, 0x50, 0x4f, 0x54                          | Logon.TargetCompId.varData  |
| 0x58, 0x00                                      | Logon.RawData.length        |
| 0x34, 0x4d, 0x48, 0x58, 0x65, 0x6c, 0x56, 0x56  | Logon.RawData.varData       |
| 0x63, 0x70, 0x6b, 0x64, 0x77, 0x75, 0x4c, 0x62  | Logon.RawData.varData       |
| 0x6c, 0x36, 0x6e, 0x37, 0x33, 0x48, 0x51, 0x55  | Logon.RawData.varData       |
| 0x58, 0x55, 0x66, 0x31, 0x64, 0x73, 0x65, 0x32  | Logon.RawData.varData       |
| 0x50, 0x43, 0x67, 0x54, 0x31, 0x44, 0x59, 0x71  | Logon.RawData.varData       |
| 0x57, 0x39, 0x77, 0x38, 0x41, 0x56, 0x5a, 0x31  | Logon.RawData.varData       |
| 0x52, 0x41, 0x43, 0x46, 0x47, 0x4d, 0x2b, 0x35  | Logon.RawData.varData       |
| 0x55, 0x64, 0x6c, 0x47, 0x50, 0x72, 0x51, 0x48  | Logon.RawData.varData       |
| 0x72, 0x67, 0x74, 0x53, 0x33, 0x43, 0x76, 0x73  | Logon.RawData.varData       |
| 0x52, 0x55, 0x52, 0x43, 0x31, 0x6f, 0x6a, 0x37  | Logon.RawData.varData       |
| 0x33, 0x6a, 0x38, 0x67, 0x43, 0x41, 0x3d, 0x3d  | Logon.RawData.varData       |
| 0x40, 0x00                                      | Logon.Username.length       |
| 0x73, 0x42, 0x52, 0x58, 0x72, 0x4a, 0x78, 0x32  | Logon.Username.varData      |
| 0x44, 0x73, 0x4f, 0x72, 0x61, 0x4d, 0x58, 0x4f  | Logon.Username.varData      |
| 0x61, 0x55, 0x6f, 0x76, 0x45, 0x68, 0x67, 0x56  | Logon.Username.varData      |
| 0x52, 0x63, 0x6a, 0x4f, 0x76, 0x43, 0x74, 0x51  | Logon.Username.varData      |
| 0x77, 0x6e, 0x57, 0x6a, 0x38, 0x56, 0x78, 0x6b  | Logon.Username.varData      |
| 0x4f, 0x68, 0x31, 0x78, 0x71, 0x62, 0x6f, 0x53  | Logon.Username.varData      |
| 0x30, 0x32, 0x53, 0x50, 0x47, 0x66, 0x4b, 0x69  | Logon.Username.varData      |
| 0x32, 0x68, 0x38, 0x73, 0x70, 0x5a, 0x4a, 0x62  | Logon.Username.varData      |


### FIX vs. FIX SBE

General:
* The `sofh.messageLength` field _must_ include the size of the SOFH (6 bytes)
* FIX SBE has no `Checksum` field
* When sending FIX SBE requests on port 9002
    * All fields must be set in the payload
    * Optional fields that are not present must be set to the corresponding `nullValue`
        * The encoders generated by `SbeTool` handle this correctly
        * Please refer to the definition of `nullValue` in the [SBE specification](https://www.fixtrading.org/standards/sbe-online/) if encoding payloads manually

**Logon** message:
* The `SenderCompID`, `TargetCompID` and `RecvWindow` fields are provided in the `Logon` FIX SBE message instead of the message header
    * The `RecvWindow` field set in the `Logon` message applies to all trading request messages within the FIX SBE session
    * When set, the `RecvWindow` field is in microseconds
* When the `ResponseMode` field is set to `OnlyAcks`, the `ExecutionReportType` field can be set to `Mini` to receive `ExecutionReportAck` messages instead of `ExecutionReport`
    * Note: The `ExecutionReportType` field is only supported on port 9001 and port 9002 for the Order Entry and Drop Copy endpoints

**MarketDataIncrementalRefresh** message:
* This single message in the FIX schema is split into the following FIX SBE messages: `MarketDataIncrementalTrade`, `MarketDataIncrementalBookTicker` and `MarketDataIncrementalDepth`
* The `MDReqID` field is omitted from the market data snapshot and refresh messages as these messages can be tied to the subscription request using the `Symbol` field and the message's template ID
    * `MDReqID` is required in the `MarketDataRequest` message so that it may appear in `MarketDataRequestReject`
    * The value of `MDReqID` must be unique across subscriptions

**MarketDataIncrementalTrade** message:
* The MDUpdateAction field available in the FIX schema is omitted in FIX SBE since the value is always `NEW`.

**MarketDataIncrementalBookTicker** message:
* FIX SBE book ticker subscriptions use **auto-culling**: when the system is under high load, it may drop outdated events instead of queuing all events and delivering them with a delay.
    * For example, if a best bid/ask event is generated at time T2 when there is still an undelivered event queued at time T1 (where T1 < T2), the event for T1 is dropped, and the system will deliver only the event for T2. This is done on a per-symbol basis.
* The `MDUpdateAction` field available in the FIX schema is omitted in FIX SBE as its value may be derived from `MDEntrySize`.
    * When `MDEntrySize` is unset (`NullVal`), `MDUpdateAction` is `DELETE`.
    * When `MDEntrySize` is set,
        * if the price level exists in your local order book, `MDUpdateAction` is `CHANGE`
        * else `MDUpdateAction` is `NEW`.

**MarketDataIncrementalDepth** message:
* FIX SBE depth update speed: 50ms
* The `MDUpdateAction` field available in the FIX schema is omitted in FIX SBE as its value may be derived from `MDEntrySize`.
    * When `MDEntrySize` is unset (`NullVal`), `MDUpdateAction` is `DELETE`.
    * When `MDEntrySize` is set,
        * if the price level exists in your local order book, `MDUpdateAction` is `CHANGE`
        * else `MDUpdateAction` is `NEW`.

### Limits

Connection limits are shared between FIX and FIX SBE.

### Errors

The following FIX SBE-specific errors may be returned:

| Code    | Message                                         |  Description                                                       |
|---------|-------------------------------------------------|--------------------------------------------------------------------|
| -1152   | Invalid SBE message header.                     | Error when decoding `messageHeader` in FIX SBE request             |
| -1153   | Invalid SBE schema ID or version specified.     | Error when parsing/decoding FIX SBE schema ID/version              |
| -1177   | Invalid encodingType.                           | Error when decoding `encodingType` field in sofh composite type    |
| -1221   | Invalid/missing field(s) in SBE message.        | Invalid/missing field when decoding FIX SBE request                |

Note: Error codes returned for semantically equivalent FIX and FIX SBE requests may not be identical.

### FAQ

See the [SBE FAQ](../faqs/sbe_faq.md) for more information on generating SBE decoders and handling schema updates.
