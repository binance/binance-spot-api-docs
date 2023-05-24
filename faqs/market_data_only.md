# Market Data Only URLs

These URLs do not require any authentication (i.e. The API key is not necessary) and serve only public market data.

## RESTful API

On the RESTful API, these are the endpoints you can request on `data-api.binance.vision`: 

* [GET /api/v3/aggTrades](../rest-api.md#compressedaggregate-trades-list)
* [GET /api/v3/avgPrice](../rest-api.md#current-average-price)
* [GET /api/v3/depth](../rest-api.md#order-book)
* [GET /api/v3/exchangeInfo](../rest-api.md#exchange-information)
* [GET /api/v3/klines](../rest-api.md#klines)
* [GET /api/v3/ping](../rest-api.md#ping)
* [GET /api/v3/ticker](../rest-api.md#rolling-window-price-change-statistics)
* [GET /api/v3/ticker/24hr](../rest-api.md#24hr-ticker-price-change-statistics)
* [GET /api/v3/ticker/bookTicker](../rest-api.md#symbol-order-book-ticker)
* [GET /api/v3/ticker/price](../rest-api.md#symbol-price-ticker)
* [GET /api/v3/time](../rest-api.md#time)
* [GET /api/v3/trades](../rest-api.md#recent-trades-list)
* [GET /api/v3/uiKlines](../rest-api.md#uiKlines)

Sample request:

```
curl -sX GET "https://data-api.binance.vision/api/v3/exchangeInfo?symbol=BTCUSDT" 
```

## Websocket Streams

Public market data can also be retrieved through the websocket market data using the URL `data-stream.binance.vision`.
The streams available through this domain are the same that can be found in the [Websocket Market Streams](../web-socket-streams.md) documentation.

Note that User Data Streams **cannot** be accessed through this URL.

Sample request:

```
wss://data-stream.binance.vision:443/ws/btcusdt@kline_1m
```


