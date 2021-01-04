# Websocket账户接口 (2021-01-01)

# 基本信息
* 本篇所列出REST接口的baseurl **https://api.binance.com**
* 用于订阅账户数据的 `listenKey` 从创建时刻起有效期为60分钟
* 可以通过`PUT`一个`listenKey`延长60分钟有效期
* `DELETE`一个 `listenKey` 立即关闭当前数据流
* 本篇所列出的websocket接口baseurl: **wss://stream.binance.com:9443**
* 订阅账户数据流的stream名称为 **/ws/\<listenKey\>**
* 每个到stream.binance.com的链接有效期不超过24小时，请妥善处理断线重连。
* 账户数据流的消息**不保证**严格时间序; **请使用 E 字段进行排序**

# 与Websocket账户接口相关的REST接口

## 生成listenKey
```
POST /api/v1/userDataStream
```
创建一个新的user data stream，返回值为一个listenKey，即websocket订阅的stream名称。

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

## 延长listenKey有效期
```
PUT /api/v1/userDataStream
```
有效期延长至本次调用后60分钟

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

## 关闭listenKey
```
DELETE /api/v1/userDataStream
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

# websocket推送事件

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

## 订单/交易 更新
当有新订单创建、订单有新成交或者新的状态变化时会推送此类事件

event type统一为 `executionReport`

具体内容需要读取 `x`字段 判断执行类型


**Payload:**
```javascript
{
  "e": "executionReport",        // 事件类型
  "E": 1499405658658,            // 事件时间
  "s": "ETHBTC",                 // 交易对
  "c": "mUvoqJxFIILMdfAW5iGSOW", // 客户端自定订单ID
  "S": "BUY",                    // 订单方向
  "o": "LIMIT",                  // 订单类型
  "f": "GTC",                    // Time in force
  "q": "1.00000000",             // 订单原始数量
  "p": "0.10264410",             // 订单原始价格
  "P": "0.00000000",             // 止盈止损单触发价格
  "F": "0.00000000",             // 冰山订单数量
  "g": -1,                       // 忽略
  "C": "null",                   // 原始订单自定义ID(原始订单，指撤单操作的对象。撤单本身被视为另一个订单)
  "x": "NEW",                    // 本次事件的具体执行类型
  "X": "NEW",                    // 订单的当前状态
  "r": "NONE",                   // 订单被拒绝的原因
  "i": 4293153,                  // 订单ID
  "l": "0.00000000",             // 订单末次成交数量
  "z": "0.00000000",             // 订单累计已成交数量
  "L": "0.00000000",             // 订单末次成交价格
  "n": "0",                      // 手续费数量
  "N": null,                     // 手续费资产类别
  "T": 1499405658657,            // 成交事件
  "t": -1,                       // 成交ID
  "I": 8641984,                  // 请忽略
  "w": true,                     // 订单是否仍然有效(对止盈/止损生效)
  "m": false,                    // 该成交是作为挂单成交吗？
  "M": false,                    // Ignore
  "O": 1499405658657,            // 订单创建时间
  "Z": "0.00000000",             // 订单累计已成交金额
  "Y": "0.00000000"              // 订单末次成交金额
}
```

**可能的执行类型:**

* NEW 新订单
* CANCELED 订单被取消
* REPLACED (保留字段，当前未使用)
* REJECTED 新订单被拒绝
* TRADE 订单有新成交
* EXPIRED 订单失效（根据订单的Time In Force参数）
