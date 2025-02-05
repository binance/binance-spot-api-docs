# FIX API

> [!NOTE]
> This API can only be used with the SPOT Exchange.

## General Information

FIX connections require TLS encryption. Please either use native TCP+TLS connection or set up a local proxy such as [stunnel](https://www.stunnel.org/) to handle TLS encryption.

**FIX sessions only support Ed25519 keys.** </br>
You can setup and configure your API key permissions on [Spot Test Network](https://testnet.binance.vision/).

### FIX API order entry sessions

- Endpoint is: `tcp+tls://fix-oe.testnet.binance.vision:9000`
- Supports placing orders, canceling orders, and querying current limit usage.
- Supports receiving all of the account's [ExecutionReport`<8>`](#executionreport) and [List Status`<N>`](#liststatus).
- Only API keys with `FIX_API` are allowed to connect.
- QuickFix Schema can be found [here](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml).

### FIX API Drop Copy sessions

- Endpoint is: `tcp+tls://fix-dc.testnet.binance.vision:9000`
- Supports receiving all of the account's [ExecutionReport`<8>`](#executionreport) and [List Status`<N>`](#liststatus).
- Only API keys with `FIX_API` or `FIX_API_READ_ONLY` are allowed to connect.
- QuickFix Schema can be found [here](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-oe.xml).

### Fix Market Data sessions

* Endpoint is: `tcp+tls://fix-md.testnet.binance.vision:9000`
* Supports market data streams and active instruments queries.  
* Does not support placing or canceling orders.   
* Only API keys with `FIX_API` or `FIX_API_READ_ONLY` are allowed to connect.
* QuickFix Schema can be found [here](https://github.com/binance/binance-spot-api-docs/blob/master/fix/schemas/spot-fix-md.xml).


<a id="orderedmode"></a>

### On message processing order

The `MessageHandling (25035)` field required in the initial [Logon`<A>`](#logon-request) message controls whether the
messages may get reordered before they are processed by the engine.

- `UNORDERED(1)` messages from client are allowed to be sent to the engine out of order.
- `SEQUENTIAL(2)` messages from client are always sent to the engine in the `MsgSeqNum (34)` order.

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

<a id="signaturecomputation"></a>

### How to sign Logon<code>&lt;A&gt;</code> request

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
* FIX Market Data sessions have a limit of 10,000 messages every 60 seconds. 

<a id="connection-limits"></a>

### Connection Limits

* Each Account has a limit on how many TCP connections can be established at the same time.
* The limit is reduced when the TCP connection is closed. If the reduction of connections is not immediate, please wait up to twice the value of `HeartBtInt (108)` for the change to take effect.
  For example, if the current value of `HeartBtInt` is 5, please wait up to 10 seconds.
* Upon breaching the limit a [Reject `<3>`](#reject) will be sent containing information about the connection limit
  breach and the current limit.
* The limit is 5 concurrent TCP connections per account for the order entry sessions.
* The limit is 10 concurrent TCP connections per account for the Drop Copy sessions.
* The limit is 100 concurrent TCP connections per account for Market Data sessions.

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

| Type           | Description                                                     |
|----------------|-----------------------------------------------------------------|
| `BOOLEAN`      | Enum: `Y` or `N`.                                               |
| `CHAR`         | Single character.                                               |
| `INT`          | Signed 64-bit integer.                                          |
| `LENGTH`       | Unsigned 64-bit integer.                                        |
| `NUMINGROUP`   | Unsigned 64-bit integer.                                        |
| `PRICE`        | Fixed-point number. Precision depends on the symbol definition. |
| `QTY`          | Fixed-point number. Precision depends on the symbol definition. |
| `SEQNUM`       | Unsigned 64-bit integer.                                        |
| `STRING`       | Sequence of printable ASCII characters.                         |
| `UTCTIMESTAMP` | String representing datetime in UTC.                            |

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
| 35    | MsgType      | STRING       | Y        | Must be the third field in the message. <br></br> Possible values: <br></br>`0` - [HEARTBEAT](#heartbeat) <br></br>`1` - [TEST_REQUEST](#testrequest) <br></br>`3` - [REJECT](#reject) <br></br>`5` - [LOGOUT](#logout) <br></br>`8` - [EXECUTION_REPORT](#executionreport) <br></br> `9` - [ORDER_CANCEL_REJECT](#ordercancelreject) <br></br> `A` - [LOGON](#logon-main) <br></br> `D` - [NEW_ORDER_SINGLE](#newordersingle) <br></br> `E` - [NEW_ORDER_LIST](#neworderlist) <br></br> `F` - [ORDER_CANCEL_REQUEST](#ordercancelrequest) <br></br> `N` - [LIST_STATUS](#liststatus) <br></br> `q` - [ORDER_MASS_CANCEL_REQUEST](#ordermasscancelrequest) <br></br> `r` - [ORDER_MASS_CANCEL_REPORT](#ordermasscancelreport) <br></br> `XCN` - [ORDER_CANCEL_REQUEST_AND_NEW_ORDER_SINGLE](#ordercancelrequestandnewordersingle) <br></br> `XLQ` - [LIMIT_QUERY](#limitquery) <br></br> `XLR` - [LIMIT_RESPONSE](#limitresponse) <br></br> `B` - [NEWS](#news) <br></br> `x`- [INSTRUMENT_LIST_REQUEST](#instrumentlistrequest) <br></br> `y` - [INSTRUMENT_LIST](#instrumentlist)  <br></br>`V` - [MARKET_DATA_REQUEST](#marketdatarequest) <br></br> `Y` - [MARKET_DATA_REQUEST_REJECT](#marketdatarequestreject) <br></br>`W` - [MARKET_DATA_SNAPSHOT](#marketdatasnapshot) <br></br>`X` - [MARKET_DATA_INCREMENTAL_REFRESH](#marketdataincrementalrefresh) |
| 49    | SenderCompID | STRING       | Y        | Must be unique across an account's active sessions.  <br></br> Must obey regex: `^[a-zA-Z0-9-_]{1,8}$`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| 56    | TargetCompID | STRING       | Y        | A string identifying this TCP connection.<br></br>On messages from client required to be set to `SPOT`. <br></br>Must be unique across TCP connections. <br></br> Must conform to the regex: `^[a-zA-Z0-9-_]{1,8}$`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |                                                                                                                                      |
| 34    | MsgSeqNum    | SEQNUM       | Y        | Integer message sequence number. <br></br> Values that will cause a gap will be rejected.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| 52    | SendingTime  | UTCTIMESTAMP | Y        | Time of message transmission (always expressed in UTC).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| 25000 | RecvWindow   | INT          | N        | Number of milliseconds after `SendingTime (52)` the request is valid for. <br></br> Defaults to `5000` milliseconds in [Logon`<A>`](#logon-request) and has a max value of `60000` milliseconds.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

<a id="trailer"></a>

### Trailer

Appears at the end of every message.

| Tag | Name     | Type   | Required | Description                                                                                                                                                                                                                                                                                                                                                      |
|-----|----------|--------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 10  | CheckSum | STRING | Y        | Always three-character numeric string, calculated by summing the ASCII values of each preceding character in the message, including start-of-header (SOH) characters. <br></br> The resultant sum is divided by 256, with the remainder forming the CheckSum value. <br></br> To maintain a fixed length, the CheckSum field is right-justified and zero-padded as needed. |

## Administrative Messages

<a id="heartbeat"></a>

### Heartbeat<code>&lt;0&gt;</code>

Sent by the server if there is no outgoing traffic during the heartbeat interval (`HeartBtInt (108)` in [Logon`<A>`](#logon-main)).

Sent by the client to indicate that the session is healthy.

Sent by the client or the server in response to a [TestRequest`<1>`](#testrequest) message.

| Tag | Name      | Type   | Required | Description                                                                                              |
|-----|-----------|--------|----------|----------------------------------------------------------------------------------------------------------|
| 112 | TestReqID | STRING | N        | When Heartbeat`<35>` is sent in response to TestRequest`<1>`, must mirror the value in TestRequest`<1>`. |

<a id="testrequest"></a>

### TestRequest<code>&lt;1&gt;</code>

Sent by the server if there is no incoming traffic during the heartbeat interval (`HeartBtInt (108)` in [Logon`<A>`](#logon-main)).

Sent by the client to request a [Heartbeat`<0>`](#heartbeat) response.

> [!NOTE]
> If the client does not respond to TestRequest`<1>` with Heartbeat`<0>` with a correct `TestReqID (112)`  within timeout, the connection will be dropped.

| Tag | Name      | Type   | Required | Description                                                            |
|-----|-----------|--------|----------|------------------------------------------------------------------------|
| 112 | TestReqID | STRING | Y        | Arbitrary string that must be included in the Heartbeat`<0>` response. |

<a id="reject"></a>

### Reject<code>&lt;3&gt;</code>

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

### Logon<code>&lt;A&gt;</code>

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

### Logout<code>&lt;5&gt;</code>

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

### News <code>&lt;B&gt;</code>

Sent by the server when the connection is about to be closed.

| Tag | Name | Type | Required | Description |
| :---- | :---- | :---- | :---- | :---- |
| 148 | Headline | STRING | Y |    |

**Sample message:**

```
8=FIX.4.4|9=0000113|35=B|49=SPOT|56=OE|34=4|52=20240924-21:07:35.773537|148=Your connection is about to be closed. Please reconnect.|10=165|
```

## Application Messages

### Order Entry Messages 

> [!NOTE]
> The messages below can only be used for the FIX Order Entry and FIX Drop Copy Sessions.

<a id="newordersingle"></a>

#### NewOrderSingle<code>&lt;D&gt;</code>

Sent by the client to submit a new order for execution.

Please refer to [Supported Order Types](#ordertype) for supported field combinations.

> [!NOTE]
> Many fields become required based on the order type.
> Please refer to [Supported Order Types](#NewOrderSingle-required-fields).

| Tag   | Name                     | Type    | Required | Description                                                                                                                                                                                                                                                                                                                  |
|-------|--------------------------|---------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 11    | ClOrdID                  | STRING  | Y        | `ClOrdID` to be assigned to the order.                                                                                                                                                                                                                                                                                       |
| 38    | OrderQty                 | QTY     | N        | Quantity of the order                                                                                                                                                                                                                                                                                                        |
| 40    | OrdType                  | CHAR    | Y        | See the [table](#ordertype) to understand supported order types and the required fields to use them.<br></br>Possible values: <br></br> `1` - MARKET <br></br> `2` - LIMIT <br></br> `3` - STOP <br></br> `4` - STOP_LIMIT                                                                                        |
| 18    | ExecInst                 | CHAR    | N        | Possible values: <br></br> `6` - PARTICIPATE_DONT_INITIATE                                                                                                                                                                                                                                                                        |
| 44    | Price                    | PRICE   | N        | Price of the order                                                                                                                                                                                                                                                                                                           |
| 54    | Side                     | CHAR    | Y        | Side of the order.<br></br>Possible values: <br></br> `1` - BUY <br></br> `2` - SELL                                                                                                                                                                                                                                                        |
| 55    | Symbol                   | STRING  | Y        | Symbol to place the order on.                                                                                                                                                                                                                                                                                                |
| 59    | TimeInForce              | CHAR    | N        | Possible values: <br></br> `1` - GOOD_TILL_CANCEL <br></br> `3` - IMMEDIATE_OR_CANCEL <br></br> `4` - FILL_OR_KILL                                                                                                                                                                                                                          |
| 111   | MaxFloor                 | QTY     | N        | Used for iceberg orders, this specifies the visible quantity of the order on the book.                                                                                                                                                                                                                                       |
| 152   | CashOrderQty             | QTY     | N        | Quantity of the order specified in the quote asset units, for reverse market orders.                                                                                                                                                                                                                                         |
| 847   | TargetStrategy           | INT     | N        | The value cannot be less than `1000000`.                                                                                                                                                                                                                                                                                                                             |
| 7940  | StrategyID               | INT     | N        |                                                                                                                                                                                                                                                                                      |
| 25001 | SelfTradePreventionMode  | CHAR    | N        | Possible values: <br></br> `1` - NONE <br></br> `2` - EXPIRE_TAKER <br></br> `3` - EXPIRE_MAKER <br></br> `4` - EXPIRE_BOTH                                                                                                                                                                                                                      |
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

#### ExecutionReport<code>&lt;8&gt;</code>

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
| 40    | OrdType                  | CHAR         | Y        | Possible values: <br></br> `1` - MARKET <br></br> `2` - LIMIT <br></br> `3` - STOP_LOSS <br></br> `4` - STOP_LIMIT                                                                                                                                                                                                                               |
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
| 25001 | SelfTradePreventionMode  | CHAR         | N        | Possible values: <br></br> `1` - NONE <br></br> `2` - EXPIRE_TAKER <br></br> `3` - EXPIRE_MAKER <br></br>`4` - EXPIRE_BOTH                                                                                                                                                                                                                       |
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
| 25009 | TriggerTrailingDeltaBips | INT          | N        | Appears only for trailing stop orders.                                                                                                                                                                                                                                                                                       |

**Sample message:**

```
8=FIX.4.4|9=330|35=8|34=2|49=SPOT|52=20240611-09:01:46.228950|56=qNXO12fH|11=1718096506197867067|14=0.00000000|17=144|32=0.00000000|37=76|38=5.00000000|39=0|40=2|44=10.00000000|54=1|55=LTCBNB|59=4|60=20240611-09:01:46.228000|150=0|151=5.00000000|636=Y|1057=Y|25001=1|25017=0.00000000|25018=20240611-09:01:46.228000|25023=20240611-09:01:46.228000|10=095|
```

<a id="ordercancelrequest"></a>

#### OrderCancelRequest<code>&lt;F&gt;</code>

Sent by the client to cancel an order or an order list.

* To cancel an order either `OrderID (11)` or `OrigClOrdID (41)` are required.
* To cancel an order list either `ListID (66)` or `OrigClListID (25015)` are required.

If the canceled order is part of an order list, the entire list will be canceled.

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

#### OrderCancelReject<code>&lt;9&gt;</code>

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

#### OrderCancelRequestAndNewOrderSingle<code>&lt;XCN&gt;</code>

Sent by the client to cancel an order and submit a new one for execution.

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
| 40    | OrdType                                 | CHAR   | Y        | See the [table](#ordertype) to understand supported order types and the required fields to use them.<br></br>Possible values: <br></br> `1` - MARKET <br></br> `2` - LIMIT <br></br> `3` - STOP <br></br> `4` - STOP_LIMIT                                                                                        |
| 18    | ExecInst                                | CHAR   | N        | Possible values: <br></br> `6` - PARTICIPATE_DONT_INITIATE                                                                                                                                                                                                                                                                        |
| 44    | Price                                   | PRICE  | N        | Price of the new order                                                                                                                                                                                                                                                                                                       |
| 54    | Side                                    | CHAR   | Y        | Side of the order.<br></br>Possible values: <br></br> `1` - BUY <br></br> `2` - SELL                                                                                                                                                                                                                                                        |
| 55    | Symbol                                  | STRING | Y        | Symbol to cancel and place the order on.                                                                                                                                                                                                                                                                                     |
| 59    | TimeInForce                             | CHAR   | N        | Possible values: <br></br> `1` - GOOD_TILL_CANCEL <br></br> `3` - IMMEDIATE_OR_CANCEL <br></br> `4` - FILL_OR_KILL                                                                                                                                                                                                                          |
| 111   | MaxFloor                                | QTY    | N        | Used for iceberg orders, this specifies the visible quantity of the order on the book.                                                                                                                                                                                                                                       |
| 152   | CashOrderQty                            | QTY    | N        | Quantity of the order specified in the quote asset units, for reverse market orders.                                                                                                                                                                                                                                         |
| 847   | TargetStrategy                          | INT    | N        | The value cannot be less than `1000000`.                                                                                                                                                                                                                                                                                                                             |
| 7940  | StrategyID                              | INT    | N        |                                                                                                                                                                                                                                                                                      |
| 25001 | SelfTradePreventionMode                 | CHAR   | N        | Possible values: <br></br> `1` - NONE <br></br> `2` - EXPIRE_TAKER <br></br> `3` - EXPIRE_MAKER <br></br> `4` - EXPIRE_BOTH                                                                                                                                                                                                                      |
| 1100  | TriggerType                             | CHAR   | N        | Possible values: `4` - PRICE_MOVEMENT                                                                                                                                                                                                                                                                                        |
| 1101  | TriggerAction                           | CHAR   | N        | Possible values: <br></br> `1` - ACTIVATE                                                                                                                                                                                                                                                                                         |
| 1102  | TriggerPrice                            | PRICE  | N        | Activation price for contingent orders. See [table](#ordertype)                                                                                                                                                                                                                                                              |
| 1107  | TriggerPriceType                        | CHAR   | N        | Possible values: <br></br> `2` - LAST_TRADE                                                                                                                                                                                                                                                                                       |
| 1109  | TriggerPriceDirection                   | CHAR   | N        | Used to differentiate between StopLoss and TakeProfit orders. See [table](#ordertype).<br></br>Possible values: <br></br> `U` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_UP_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE <br></br> `D` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_DOWN_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE |
| 25009 | TriggerTrailingDeltaBips                | INT    | N        | Provide to create trailing orders.                                                                                                                                                                                                                                                                                           |

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

#### OrderMassCancelRequest<code>&lt;q&gt;</code>

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
8=FIX.4.4|9=94|35=q|34=2|49=dpYPesqv|52=20240613-01:24:36.948|56=SPOT|11=1718241876901971671|55=ABCDEF|530=1|10=110|
```

**Responses:**

* [ExecutionReport`<8>`](#executionreport) with `ExecType (150)` value `CANCELED (4)` for the every order canceled.
* [OrderMassCancelReport`<r>`](#ordermasscancelreport) with `MassCancelResponse (531)` field indicating whether the message is accepted or rejected.
* [Reject`<3>`](#reject) if the message is rejected.

<a id="ordermasscancelreport"></a>

#### OrderMassCancelReport<code>&lt;r&gt;</code>

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

#### NewOrderList<code>&lt;E&gt;</code>

Sent by the client to submit a list of orders for execution.

Orders in an order list are contingent on one another.
Please refer to [Supported Order List Types](#order-list-types) for supported order types and triggering instructions.

| Tag      | Name                         | Type       | Required | Description                                                                                                                                                                                                                                                                                                                  |
|----------|------------------------------|------------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 25014    | ClListID                     | STRING     | Y        | `ClListID` to be assigned to the order list.                                                                                                                                                                                                                                                                                 |
| 1385     | ContingencyType              | INT        | N        | Possible values: <br></br> `1` - ONE_CANCELS_THE_OTHER <br></br> `2` - ONE_TRIGGERS_THE_OTHER                                                                                                                                                                                                                                          |
| 73       | NoOrders                     | NUMINGROUP | N        | The length of the array for Orders. Only 2 or 3 are allowed.             |
| =>11     | ClOrdID                      | STRING     | Y        | `ClOrdID` to be assigned to the order                                                                                                                                                                                                                                                                                        |
| =>38     | OrderQty                     | QTY        | N        | Quantity of the order                                                                                                                                                                                                                                                                                                        |
| =>40     | OrdType                      | CHAR       | Y        | See the [table](#ordertype) to understand supported order types and the required fields to use them.<br></br>Possible values: <br></br> `1` - MARKET <br></br> `2` - LIMIT <br></br> `3` - STOP <br></br> `4` - STOP_LIMIT                                                                                        |
| =>18     | ExecInst                     | CHAR       | N        | Possible values: <br></br> `6` - PARTICIPATE_DONT_INITIATE                                                                                                                                                                                                                                                                        |
| =>44     | Price                        | PRICE      | N        | Price of the order                                                                                                                                                                                                                                                                                                           |
| =>54     | Side                         | CHAR       | Y        | Side of the order. Possible values: <br></br> `1` - BUY <br></br> `2` - SELL                                                                                                                                                                                                                                                           |
| =>55     | Symbol                       | STRING     | Y        | Symbol to place the order on.                                                                                                                                                                                                                                                                                                |
| =>59     | TimeInForce                  | CHAR       | N        | Possible values: <br></br> `1` - GOOD_TILL_CANCEL <br></br> `3` - IMMEDIATE_OR_CANCEL <br></br> `4` - FILL_OR_KILL                                                                                                                                                                                                                          |
| =>111    | MaxFloor                     | QTY        | N        | Used for iceberg orders, this specifies the visible quantity of the order on the book.                                                                                                                                                                                                                                       |
| =>152    | CashOrderQty                 | QTY        | N        | Quantity of the order specified in the quote asset units, for reverse market orders.                                                                                                                                                                                                                                         |
| =>847    | TargetStrategy               | INT        | N        | The value cannot be less than `1000000`.                                                                                                                                                                                                                                                                                                                             |
| =>7940   | StrategyID                   | INT        | N        |                                                                                                                                                                                                                                                                                      |
| =>25001  | SelfTradePreventionMode      | CHAR       | N        | Possible values: <br></br> `1` - NONE <br></br>`2` - EXPIRE_TAKER <br></br> `3` - EXPIRE_MAKER <br></br> `4` - EXPIRE_BOTH                                                                                                                                                                                                                       |
| =>1100   | TriggerType                  | CHAR       | N        | Possible values: <br></br> `4` - PRICE_MOVEMENT                                                                                                                                                                                                                                                                                   |
| =>1101   | TriggerAction                | CHAR       | N        | Possible values: <br></br> `1` - ACTIVATE                                                                                                                                                                                                                                                                                         |
| =>1102   | TriggerPrice                 | PRICE      | N        | Activation price for contingent orders. See [table](#ordertype)                                                                                                                                                                                                                                                              |
| =>1107   | TriggerPriceType             | CHAR       | N        | Possible values: <br></br> `2` - LAST_TRADE                                                                                                                                                                                                                                                                                       |
| =>1109   | TriggerPriceDirection        | CHAR       | N        | Used to differentiate between StopLoss and TakeProfit orders. See [table](#ordertype).<br></br>Possible values: <br></br> `U` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_UP_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE <br></br> `D` - TRIGGER_IF_THE_PRICE_OF_THE_SPECIFIED_TYPE_GOES_DOWN_TO_OR_THROUGH_THE_SPECIFIED_TRIGGER_PRICE |
| =>25009  | TriggerTrailingDeltaBips     | INT        | N        | Provide to create trailing orders.                                                                                                                                                                                                                                                                                           |
| =>25010  | NoListTriggeringInstructions | NUMINGROUP | N        | The length of the array for ListTriggeringInstructions.       |
| ==>25011 | ListTriggerType              | CHAR       | N        | What needs to happen to the order pointed to by ListTriggerTriggerIndex in order for the action to take place. <br></br> Possible values: <br></br> `1` - ACTIVATED <br></br> `2` - PARTIALLY_FILLED <br></br> `3` - FILLED                                                                                                                      |
| ==>25012 | ListTriggerTriggerIndex      | INT        | N        | Index of the trigger order: 0-indexed.                                                                                                                                                                                                                                                                                       |
| ==>25013 | ListTriggerAction            | CHAR       | N        | Action to take place on this order after the ListTriggerType has been fulfilled. <br></br> Possible values: <br></br> `1` - RELEASE <br></br> `2` - CANCEL                                                                                                                                                                                  |

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


<a id="liststatus"></a>

#### ListStatus<code>&lt;N&gt;</code>

Sent by the server whenever an order list state changes.

> [!NOTE]
> By default, ListStatus`<N>` is sent for all order lists of an account, including those submitted in different connections.
> Please see [Response Mode](#responsemode) for other behavior options.

| Tag      | Name                         | Type         | Required | Description                                                                                                                                             |
|----------|------------------------------|--------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| 55       | Symbol                       | STRING       | Y        | Symbol of the order list.                                                                                                                               |
| 66       | ListID                       | STRING       | N        | `ListID` of the list as assigned by the exchange.                                                                                                       |
| 25014    | ClListID                     | STRING       | N        | `ClListID` of the list as assigned on the request.                                                                                                      |
| 25015    | OrigClListID                 | STRING       | N        |                                                                                                                                                         |
| 1385     | ContingencyType              | INT          | N        | Possible values: <br></br> `1` - ONE_CANCELS_THE_OTHER <br></br> `2` - ONE_TRIGGERS_THE_OTHER                                                                     |
| 429      | ListStatusType               | INT          | Y        | Possible values: <br></br> `2` - RESPONSE <br></br>`4` - EXEC_STARTED <br></br> `5` - ALL_DONE                                                                         |
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
8=FIX.4.4|9=290|35=N|34=2|49=SPOT|52=20240607-02:19:07.837191|56=Eg13pOvN|55=ABCDEF|60=20240607-02:19:07.836000|66=25|73=2|55=LTCBNB|37=52|11=w1717726747805308656|55=ABCDEF|37=53|11=p1717726747805308656|25010=1|25011=3|25012=0|25013=1|429=4|431=3|1385=2|25014=1717726747805308656|25015=1717726747805308656|10=019|
```

### Limit Messages

<a id="limitquery"></a>

#### LimitQuery<code>&lt;XLQ&gt;</code>

Sent by the client to query current limits.

| Tag  | Name  | Type   | Required | Description        |
|------|-------|--------|----------|--------------------|
| 6136 | ReqID | STRING | Y        | ID of this request | 

**Sample message:**

```
8=FIX.4.4|9=82|35=XLQ|34=2|49=7buKHZxZ|52=20240614-05:35:35.357|56=SPOT|6136=1718343335357229749|10=170|
```

<a id="limitresponse"></a>

#### LimitResponse<code>&lt;XLR&gt;</code>

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

#### InstrumentListRequest<code>&lt;x&gt;</code>

Sent by the client to query information about active instruments (i.e., those that have the TRADING status). If used for an inactive instrument, it will be responded to with a [Reject`<3>`](#reject).

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

#### InstrumentList<code>&lt;y&gt;</code>

Sent by the server in a response to the [InstrumentListRequest`<x>`](#instrumentlistrequest).

> [!NOTE]
> More detailed symbol information is available through the [exchangeInfo](https://github.com/binance/binance-spot-api-docs/blob/master/testnet/rest-api.md#exchange-information) endpoint.


| Tag     | Name                  | Type       | Required | Description                                |
|---------|-----------------------|------------|----------|--------------------------------------------|
| 320     | InstrumentReqID       | STRING     | Y        | `InstrumentReqID` from the request.        |
| 146     | NoRelatedSym          | NUMINGROUP | Y        | Number of symbols                          |
| =>55    | Symbol                | STRING     | Y        |                                            |
| =>15    | Currency              | STRING     | Y        | Quote asset of this symbol                 |
| =>562   | MinTradeVol           | QTY        | Y        | The minimum trading quantity               |
| =>1140  | MaxTradeVol           | QTY        | Y        | The maximum trading quantity               |
| =>25039 | MinQtyIncrement       | QTY        | Y        | The minimum quantity increase              |
| =>25040 | MarketMinTradeVol     | QTY        | Y        | The minimum market order trading quantity  |
| =>25041 | MarketMaxTradeVol     | QTY        | Y        | The maximum market order trading quantity  |
| =>25042 | MarketMinQtyIncrement | QTY        | Y        | The minimum market order quantity increase |
| =>969   | MinPriceIncrement     | PRICE      | Y        | The minimum price increase                 |

**Sample message:**

```
8=FIX.4.4|9=218|35=y|49=SPOT|56=BMDWATCH|34=2|52=20250114-08:46:56.100147|320=BTCUSDT_INFO|146=1|55=BTCUSDT|15=USDT|562=0.00001000|1140=9000.00000000|25039=0.00001000|25040=0.00000001|25041=76.79001236|25042=0.00000001|969=0.01000000|10=093|
```

<a id="marketdatarequest"></a>

#### MarketDataRequest<code>&lt;V&gt;</code>

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

### MarketDataRequestReject<code>&lt;Y&gt;</code>

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

### MarketDataSnapshot<code>&lt;W&gt;</code>

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

### MarketDataIncrementalRefresh<code>&lt;X&gt;</code>

Sent by the server when there is a change in a subscribed stream.

| Tag     | Name              | Type         | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                  |
|---------|-------------------|--------------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 262     | MDReqID           | STRING       | Y        | ID of the [MarketDataRequest`<V>`](#marketdatarequest) that activated this subscription                                                                                                                                                                                                                                                                                                                                      |
| 893     | LastFragment      | BOOLEAN      | N        | When present, this indicates that the message was fragmented. Fragmentation occurs when `NoMDEntry` would exceed 10000 in a single [MarketDataIncrementalRefresh`<X>`](#marketdataincrementalrefresh), in order to limit it to 10000. The fragments of a fragmented message are guaranteed to be consecutive in the stream. It can only appear in the [Trade Stream](#tradestream) and [Diff. Depth Stream](#diffdepthstream). |
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
