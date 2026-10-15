<a id="market-data-only-urls"></a>

# 仅提供市场数据的URL

这些 URL 不需要任何身份验证（即不需要 API Key）并且仅提供公开市场数据。

### RESTful API

在 RESTful API 上，您可以在 `data-api.binance.vision` 上访问以下接口：

- [GET /api/v3/aggTrades](../rest-api_CN.md#compressedaggregate-trades-list)
- [GET /api/v3/avgPrice](../rest-api_CN.md#current-average-price)
- [GET /api/v3/depth](../rest-api_CN.md#order-book)
- [GET /api/v3/exchangeInfo](../rest-api_CN.md#exchange-information)
- [GET /api/v3/klines](../rest-api_CN.md#klinecandlestick-data)
- [GET /api/v3/ping](../rest-api_CN.md#test-connectivity)
- [GET /api/v3/ticker](../rest-api_CN.md#rolling-window-price-change-statistics)
- [GET /api/v3/ticker/24hr](../rest-api_CN.md#24hr-ticker-price-change-statistics)
- [GET /api/v3/ticker/bookTicker](../rest-api_CN.md#symbol-order-book-ticker)
- [GET /api/v3/ticker/price](../rest-api_CN.md#symbol-price-ticker)
- [GET /api/v3/time](../rest-api_CN.md#check-server-time)
- [GET /api/v3/trades](../rest-api_CN.md#recent-trades-list)
- [GET /api/v3/uiKlines](../rest-api_CN.md#uiklines)

请求示例:

```
curl -sX GET "https://data-api.binance.vision/api/v3/exchangeInfo?symbol=BTCUSDT"
```

### Websocket Streams

也可以通过 Websocket 市场数据的 URL `data-stream.binance.vision`
提取公共市场数据。此域名所提供的 stream 与 [WebSocket Market Streams_CN](../web-socket-streams_CN.md)
文档中的相同。请注意用户数据流**无法**从此 URL 获得。

请求示例:

```
wss://data-stream.binance.vision:443/ws/btcusdt@kline_1m
```
