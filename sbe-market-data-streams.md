# SBE Market Data Streams

## General Information

* The base endpoint is **stream-sbe.binance.com** or **stream-sbe.binance.com:9443**.
* To retrieve market data in JSON format, please refer to [this page](web-socket-streams.md).
* SBE schema used for decoding the streams can be found [here](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/stream_1_0.xml).
* All symbols in stream names are **lowercase**.
* You can subscribe to a single stream at **/ws/\<streamName\>**.
* You can subscribe to multiple streams at **/stream?streams=\<streamName1\>/\<streamName2\>/\<streamName3\>**.
* A single connection to **stream-sbe.binance.com** is **only valid for 24 hours**; expect to be disconnected at the 24 hour mark.
* All time and timestamp fields are in **microseconds**.
* **An API Key is necessary for access**.
  * Only Ed25519 keys are allowed.
  * Please put your API Key in the `X-MBX-APIKEY` header when opening the connection. Timestamp and signature are not necessary.
  * No extra API key permissions are necessary to access public market data. Symbol whitelist also does not affect access to SBE Market Data Streams.
  * However, if you use an IP whitelist for the API key, only specified IP addresses are allowed to use the API key.
* The server sends a `ping frame` every 20 seconds.
  * If the server does not receive a `pong frame` back from you within a minute, the connection will be closed.
  * When you receive a ping, you must send a pong with a copy of ping's payload as soon as possible.
  * Unsolicited `pong frames` are allowed, but will not prevent disconnection. **It is recommended that the payload for these pong frames are empty.**
* [Live Subscribing and Unsubscribing](web-socket-streams.md#live-subscribingunsubscribing-to-streams) is also supported.
  * You must send the subscription requests in JSON, and will receive the subscription response also in JSON.
  * You can differentiate subscription responses from market data events by looking at the WebSocket frame type: subscription responses are always sent in text frames (containing JSON), and events are always sent in binary frames (containing SBE).
* If your request contains a symbol name containing non-ASCII characters, then the stream events may contain non-ASCII characters encoded in UTF-8.

## WebSocket Limits

* WebSocket connections have a rate limit of **5 requests per second**.
  * Only messages from your client are considered:
    * `PING frame`
    * `PONG frame`
    * `Text frame` with JSON control request
  * Events pushed by the server are not rate-limited.
  * Connections that go beyond the limit will be closed. Repeatedly disconnected IP addresses may be banned.
* A single connection can listen to a maximum of 1024 streams.
* There is a limit of **300 connection attempts every 5 minutes per IP address**.

## Available Streams

### Trades Streams

Raw trade information, pushed in real-time.

**SBE Message Name:** `TradesStreamEvent`

**Stream Name**: \<symbol\>@trade

**Update Speed**: Real time

### Best Bid/Ask Streams

The best bid and ask price and quantity, pushed in real-time when the order book changes.

> [!NOTE]
> Best bid/ask streams in SBE are the equivalent of bookTicker streams in JSON, except they support auto-culling, and also include the `eventTime` field.

**SBE Message Name:** `BestBidAskStreamEvent`

**Stream Name**: \<symbol\>@bestBidAsk

**Update Speed**: Real time

<a id="auto-culling"></a>
SBE best bid/ask streams use **auto-culling**: when the system is under high load, it may drop outdated events instead of queuing all events and delivering them with a delay.

For example, if a best bid/ask event is generated at time T2 when there is still an undelivered event queued at time T1 (where T1 < T2), the event for T1 is dropped, and the system will deliver only the event for T2. This is done on a per-symbol basis.

### Diff. Depth Streams

Incremental updates to the order book, pushed at regular intervals. Use this stream to maintain a local order book.

[How to manage a local order book.](web-socket-streams.md#how-to-manage-a-local-order-book-correctly)

**SBE Message Name:** `DepthDiffStreamEvent`

**Stream Name**: \<symbol\>@depth

**Update Speed:**  50ms

### Partial Book Depth Streams

Snapshots of the top 20 levels of the order book, pushed at regular intervals.

**SBE Message Name:** `DepthSnapshotStreamEvent`

**Stream Name**: \<symbol\>@depth20

**Update Speed:** 50ms
