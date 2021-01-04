# 更新日志 (2021-01-01)


## 2021-01-01

**USER DATA STREAM**

* 移除`outboundAccountInfo`事件.

---

## 2020-11-27

为了优化性能，除了当前的`api.binance.com`，新加了一些API的集群。如果访问`api.binance.com`有性能问题，也可以尝试访问:

* https://api1.binance.com/api/v3/*
* https://api2.binance.com/api/v3/*
* https://api3.binance.com/api/v3/*

## 2020-09-09

用户数据 STREAM

* `outboundAccountInfo`事件不再推荐使用。
* `outboundAccountInfo`事件以后会被删除(具体时间未定) **请使用 `outboundAccountPosition` 事件.**
* `outboundAccountInfo`只推送余额不为0，以及余额刚变成0的资产。

---

## 2020-05-01
* 从2020-05-01 UTC 00:00开始, 所有交易对都会有最多200个挂单的限制, 体现在过滤器[MAX_NUM_ORDERS](./rest-api_CN.md#max_num_orders-%E6%9C%80%E5%A4%9A%E8%AE%A2%E5%8D%95%E6%95%B0)上.
  * 已经存在的挂单不会被移除或者撤销。
  * 单交易对(`symbol`)的挂单数量达到或超过200的账号, 无法在此交易对上下新的订单, 除非挂单数量低于200。
  * OCO订单在被触发成`LIMIT`订单, 或者被触发成`STOP_LOSS`(或者`STOP_LOSS_LIMIT`)前, 被认为是2个挂单量. 一旦OCO订单被触发, 就只被算作一个挂单。

---

## 2020-04-23

WEB SOCKET 连接限制

* Websocket服务器每秒最多接受5个消息。消息包括:
	* PING帧
	* PONG帧
	* JSON格式的消息, 比如订阅, 断开订阅.
* 如果用户发送的消息超过限制，连接会被断开连接。反复被断开连接的IP有可能被服务器屏蔽。
* 单个连接最多可以订阅 **1024** 个Streams。

---
## 2020-03-24

* 添加过滤器 `MAX_POSITION`.
    * 这个过滤器定义账户允许的基于`base asset`的最大仓位。一个用户的仓位可以定义为如下资产的总和:
        * `base asset`的可用余额
        * `base asset`的锁定余额
        * 所有处于open的买单的数量总和

    * 如果用户的仓位大于最大的允许仓位，买单会被拒绝。

---

## 2018-11-13
### Rest API
  * 账户交易权限被禁时允许进行撤单操作。
  * 增加了新的过滤器: `PERCENT_PRICE`, `MARKET_LOT_SIZE`, `MAX_NUM_ICEBERG_ORDERS`。
  * 增加了 `RAW_REQUST` 频率限制. 该限制仅统计请求次数，不统计请求权重。
  * /api/v3/ticker/price 无symbol参数时，权重增加到2。
  * /api/v3/ticker/bookTicker 无symbol参数时，权重增加到2。
  * DELETE /api/v3/order 现在会返回订单撤销前所处的末次状态。
  * `MIN_NOTIONAL` 新增两个参数: `applyToMarket` (是否对市价单生效) and `avgPriceMins` (对市价单生效时，估算金额时使用过去几分钟的平均价格?). 
  *  /api/v1/exchangeInfo 中的限制增加了`intervalNum`. `intervalNum`表示该限制针对多少时间间隔进行统计. 例如: `intervalNum`= 5, `interval` = minute, 表示该限制对每5分钟内的行为进行统计。
  
#### 如何计算过去n分钟平均价格:
  1. [对过去n分钟所有订单的数量\*价格求和] / 过去n分钟订单数
  2. 如果过去n分钟没有交易发生，则继续向前追溯，直到找到第一个交易，以此价格作为过去n分钟平均价格。
  3. 如果该交易对之前从未发生过交易，则无平均价格，亦即无法在该交易对下市价单，必须至少有一个（双方均未限价单）的交易成交后才可以下市价单。
  4. 当前系统使用的平均价格可以通过接口 `https://api.binance.com/api/v3/avgPrice?symbol=<symbol>`查询
     例如
     https://api.binance.com/api/v3/avgPrice?symbol=BNBUSDT

### User data stream
  * 成交报告中增加了 `末次成交金额` (`Y`)，等于 `末次成交量` * `末次成交价格` (`L` * `l`).

## 2018-07-18
### Rest API
  *  新过滤器: `ICEBERG_PARTS`
  *  `POST api/v3/order` 中 `newOrderRespType` 参数的缺省值更改; `MARKET`  `LIMIT` 默认为 `FULL`, 其他默认为 `ACK`.
  *  POST api/v3/order `RESULT` 与 `FULL` 响应中增加 `cummulativeQuoteQty`
  *  GET api/v3/openOrders 无symbol调用权重下降为 40.
  *  GET api/v3/ticker/24hr 无symbol调用权重下降为 40.
  *  GET /api/v1/trades amount最大可取到1000.
  *  GET /api/v1/historicalTrades amount最大可取到1000.
  *  GET /api/v1/aggTrades amount最大可取到1000.
  *  GET /api/v1/klines amount最大可取到1000.
  *  订单查询结果返回中增加 `updateTime` 字段，代表该订单末次更新(创建、成交、过期、取消、拒绝等等)时间; `time` 仅表示创建时间.
  *  订单查询结果中增加 `cummulativeQuoteQty`字段. 负值表示尚无任何成交，该字段不可用.
  *  `REQUESTS` 限制更名为 `REQUEST_WEIGHT`. 避免名字造成的误解。

### User data stream
  *  订单报告与成交报告中增加`cummulativeQuoteQty` 字段 (`Z`). 表示已经成交的金额， 即已经花费的金额(买入订单)或已经收到的金额(卖出订单)，均未计算手续费. 此功能增加之前成交的订单在历史订单接口中查询到的该字段可能小于零. 
  *  `cummulativeQuoteQty`/`cummulativeQty` 可以用来计算该订单已经成交部分的平均价格。
  *  成交报告中增加了 `O`字段 (订单创建时间) 


## 2018-01-23
  * GET /api/v1/historicalTrades权重降为 5
  * GET /api/v1/aggTrades 权重降为 1
  * GET /api/v1/klines 权重降为 1
  * GET /api/v1/ticker/24hr 不带symbol参数的权重降为 symbols总数 / 2
  * GET /api/v3/allOrders 权重降为 5
  * GET /api/v3/myTrades 权重降为 5
  * GET /api/v3/account 权重降为 5
  * GET /api/v1/depth limit=500 权重降为 5
  * GET /api/v1/depth limit=1000 权重降为 10
  * websocket 用户增加 -1003 error code

## 2018-01-20
  * GET /api/v1/ticker/24hr 单symbol参数调用权重降为 1
  * GET /api/v3/openOrders 不带symbol参数的权重降为 symbols总数 / 2
  * GET /api/v3/allOrders  权重降为  15
  * GET /api/v3/myTrades  权重降为  15
  * GET /api/v3/order  权重降为  1
  * 自成交现在会在myTrades结果中有两条记录。

## 2018-01-14
  * GET /api/v1/aggTrades 权重改为 2
  * GET /api/v1/klines 权重改为 2
  * GET /api/v3/order 权重改为 2
  * GET /api/v3/allOrders 权重改为 20
  * GET /api/v3/account 权重改为 20
  * GET /api/v3/myTrades 权重改为 20
  * GET /api/v3/historicalTrades 权重改为 20
