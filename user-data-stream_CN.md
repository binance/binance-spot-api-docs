# WebSocket 账户接口(2022-12-05)

# 基本信息
* 本篇所列出API接口的base url : **https://api.binance.com**
* 用于订阅账户数据的 `listenKey` 从创建时刻起有效期为60分钟
* 可以通过 `PUT` 一个 `listenKey` 延长60分钟有效期
* 可以通过`DELETE`一个 `listenKey` 立即关闭当前数据流，并使该`listenKey` 无效
* 在具有有效`listenKey`的帐户上执行`POST`将返回当前有效的`listenKey`并将其有效期延长60分钟
* websocket接口的baseurl: **wss://stream.binance.com:9443**
* U订阅账户数据流的stream名称为 **/ws/\<listenKey\>** 或 **/stream?streams=\<listenKey\>**
* 每个链接有效期不超过24小时，请妥善处理断线重连。
* 账户数据流的消息不保证严格时间序; **请使用 E 字段进行排序**

# 与Websocket账户接口相关的REST接口

## 生成 Listen Key (USER_STREAM)
```
POST /api/v3/userDataStream
```
开始一个新的数据流。除非发送 keepalive，否则数据流于60分钟后关闭。如果该帐户具有有效的`listenKey`，则将返回该`listenKey`并将其有效期延长60分钟。

**权重:**
1

**参数:**
NONE

**响应:**
```javascript
{
  "listenKey": "pqia91ma19a5s61cv6a81va65sdf19v8a65a1a5s61cv6a81va65sdf19v8a65a1"
}
```

## 延长 Listen Key 有效期 (USER_STREAM)
```
PUT /api/v3/userDataStream
```
有效期延长至本次调用后60分钟, 建议每30分钟发送一个 ping。

**权重:**
1

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES

**响应:**
```javascript
{}
```

## 关闭 Listen Key (USER_STREAM)
```
DELETE /api/v3/userDataStream
```
关闭某账户数据流

**权重:**
1

**参数:**

名称 | 类型 | 是否必须 | 描述
------------ | ------------ | ------------ | ------------
listenKey | STRING | YES

**响应:**
```javascript
{}
```

# Websocket推送事件

## 账户更新

每当帐户余额发生更改时，都会发送一个事件`outboundAccountPosition`，其中包含可能由生成余额变动的事件而变动的资产。

**Payload**

```javascript
{
  "e": "outboundAccountPosition", // 事件类型
  "E": 1564034571105,             // 事件时间
  "u": 1564034571073,             // 账户末次更新时间戳
  "B": [                          // 余额
    {
      "a": "ETH",                 // 资产名称
      "f": "10000.000000",        // 可用余额
      "l": "0.000000"             // 冻结余额
    }
  ]
}
```

## 余额更新

当下列情形发生时更新:

* 账户发生充值或提取
* 交易账户之间发生划转(例如 现货向杠杆账户划转)

**Payload**

```javascript
{
  "e": "balanceUpdate",         //Event Type
  "E": 1573200697110,           //Event Time
  "a": "ABC",                   //Asset
  "d": "100.00000000",          //Balance Delta
  "T": 1573200697068            //Clear Time
}
```

  
## 订单更新
订单通过`executionReport`事件进行更新。 

请查阅文档[公开API参数](#public-api-definitions)以及以下文档，以获取相关的枚举定义。

通过将`Z`除以`z`可以找到平均价格。


**Payload:**
```javascript
{
  "e": "executionReport",        // 事件类型
  "E": 1499405658658,            // 事件时间
  "s": "ETHBTC",                 // 交易对
  "c": "mUvoqJxFIILMdfAW5iGSOW", // clientOrderId
  "S": "BUY",                    // 订单方向
  "o": "LIMIT",                  // 订单类型
  "f": "GTC",                    // 有效方式
  "q": "1.00000000",             // 订单原始数量
  "p": "0.10264410",             // 订单原始价格
  "P": "0.00000000",             // 止盈止损单触发价格
  "d": 4,                        // 追踪止损(Trailing Delta) 只有在追踪止损订单中才会推送.
  "F": "0.00000000",             // 冰山订单数量
  "g": -1,                       // OCO订单 OrderListId
  "C": "",                       // 原始订单自定义ID(原始订单，指撤单操作的对象。撤单本身被视为另一个订单)
  "x": "NEW",                    // 本次事件的具体执行类型
  "X": "NEW",                    // 订单的当前状态
  "r": "NONE",                   // 订单被拒绝的原因
  "i": 4293153,                  // orderId
  "l": "0.00000000",             // 订单末次成交量
  "z": "0.00000000",             // 订单累计已成交量
  "L": "0.00000000",             // 订单末次成交价格
  "n": "0",                      // 手续费数量
  "N": null,                     // 手续费资产类别
  "T": 1499405658657,            // 成交时间
  "t": -1,                       // 成交ID
  "I": 8641984,                  // 请忽略
  "w": true,                     // 订单是否在订单簿上？
  "m": false,                    // 该成交是作为挂单成交吗？
  "M": false,                    // 请忽略
  "O": 1499405658657,            // 订单创建时间
  "Z": "0.00000000",             // 订单累计已成交金额
  "Y": "0.00000000",             // 订单末次成交金额
  "Q": "0.00000000"              // Quote Order Qty
  "D": 1668680518494,            // 追踪时间; 这仅在追踪止损订单已被激活时可见
  "j": 1,                        // Strategy ID; 下单时填上字段才会返回
  "J": 1000000                   // Strategy Type; 下单时填上字段才会返回
  "W": 1499405658657,            // Working Time; 订单被添加到 order book 的时间
  "V": "NONE"                    // SelfTradePreventionMode
}
```

**可能的执行类型:**

* NEW 新订单
* CANCELED 订单被取消
* REPLACED (保留字段，当前未使用)
* REJECTED 新订单被拒绝 （这信息只会在撤消挂单再下单中发生，下新订单被拒绝但撤消挂单请求成功。）
* TRADE 订单有新成交
* EXPIRED 订单失效（根据订单的Time In Force参数）

如果订单是OCO，则除了显示"executionReport"事件外，还将显示一个名为"ListStatus"的事件。

> **Payload**

```javascript
{
  "e": "listStatus",                // 事件类型
  "E": 1564035303637,               // 事件时间
  "s": "ETHBTC",                    // 交易对
  "g": 2,                           // OrderListId
  "c": "OCO",                       // Contingency Type
  "l": "EXEC_STARTED",              // List Status Type
  "L": "EXECUTING",                 // List Order Status
  "r": "NONE",                      // List 被拒绝的原因
  "C": "F4QN4G8DlFATFlIUQ0cjdD",    // List Client Order ID
  "T": 1564035303625,               // 成交时间
  "O": [                           
    {
      "s": "ETHBTC",                // 交易对
      "i": 17,                      // orderId
      "c": "AJYsMjErWJesZvqlJCTUgL" // clientOrderId
    },
    {
      "s": "ETHBTC",
      "i": 18,
      "c": "bfYPSQdLoqAJeNrOr9adzq"
    }
  ]
}
```