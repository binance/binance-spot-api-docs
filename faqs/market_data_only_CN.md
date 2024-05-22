# 仅提供市场数据的URL

这些 URL 不需要任何身份验证（即不需要 API Key）并且仅提供公开市场数据。

## RESTful API

在 RESTful API 上，您可以在 `data-api.binance.vision` 上访问以下接口：

* [GET /api/v3/aggTrades](../rest-api_CN.md#近期成交归集)
* [GET /api/v3/avgPrice](../rest-api_CN.md#当前平均价格)
* [GET /api/v3/depth](../rest-api_CN.md#深度信息)
* [GET /api/v3/exchangeInfo](../rest-api_CN.md#交易规范信息)
* [GET /api/v3/klines](../rest-api_CN.md#k线数据)
* [GET /api/v3/ping](../rest-api_CN.md#测试服务器连通性-ping)
* [GET /api/v3/ticker](../rest-api_CN.md#滚动窗口价格变动统计)
* [GET /api/v3/ticker/24hr](../rest-api_CN.md#24hr价格变动情况)
* [GET /api/v3/ticker/bookTicker](../rest-api_CN.md#最优挂单接口)
* [GET /api/v3/ticker/price](../rest-api_CN.md#最新价格接口)
* [GET /api/v3/time](../rest-api_CN.md#获取服务器时间)
* [GET /api/v3/trades](../rest-api_CN.md#近期成交)
* [GET /api/v3/uiKlines](../rest-api_CN.md#uik线数据)

请求示例:

```
curl -sX GET "https://data-api.binance.vision/api/v3/exchangeInfo?symbol=BTCUSDT" 
```

## Websocket Streams

也可以通过 Websocket 市场数据的 URL `data-stream.binance.vision` 提取公共市场数据。
此域名所提供的 stream 与 [Websocket Market Streams_CN](../web-socket-streams_CN.md) 文档中的相同。
请注意账户信息推送**无法**从此 URL 获得。

请求示例:

```
wss://data-stream.binance.vision:443/ws/btcusdt@kline_1m
```

