# 错误代码汇总 (2023-03-13)
币安Rest接口(包括wapi)返回的错误包含两部分，错误码与错误信息. 错误码是大类，一个错误码可能对应多个不同的错误信息。
以下是一个完整错误码实例
```javascript
{
  "code":-1121,
  "msg":"Invalid symbol."
}
```


## 10xx - 服务器或网络问题
#### -1000 未知错误
 * 未知错误

#### -1001 连接断开
 * 通常是一个内部错误，一般重试即可解决。

#### -1002 未授权
 * 请检查你的(API)权限

#### -1003 请求过多
 * 排队的请求过多。
 * 请求权重过多； 当前限制是 %s 每 %s 的请求权重。 请使用 Websocket Streams 进行实时更新，以避免轮询API。
 * 请求权重过多； IP被禁止，直到％s。 请使用 Websocket Streams 进行实时更新，以免被禁。

#### -1006 非常规响应
 * （从内部）接收到了不符合预设格式的消息，下单状态未知。

#### -1007 超时
 * 后端服务超时，下单状态未知。

### -1008 SERVER_BUSY
  * 现货交易服务器当前因其他请求而过载。 请在几分钟后重试。

#### -1014 不支持的订单参数(组合)
 * 不支持的订单参数组合. 

#### -1015 订单太多
 * 下单(撤单)太多

#### -1016 服务器下线
 * 服务器下线

#### -1020 不支持的操作
 * 不支持的操作

#### -1021 时间同步问题
 * 时延过大，服务器根据接请求中的时间戳判定耗时已经超出了recevWindow。请改善网络条件或者增大recevWindow。
 * 时间偏移过大，服务器根据请求中的时间戳判定客户端时间比服务器时间提前了1秒钟以上。(该参数不可由客户端调节)

#### -1022 签名不正确
 * 请求中携带的signature与服务器根据规则计算得到的signature不一致。通常是因为客户端代码中使用的apisecret错误。


## 11xx - 请求内容中的问题
#### -1100 非法字符
 * Illegal characters found in a parameter.
 * Illegal characters found in parameter '%s'; legal range is '%s'.

#### -1101 参数太多
 * Too many parameters sent for this endpoint.
 * Too many parameters; expected '%s' and received '%s'.
 * Duplicate values for a parameter detected.

#### -1102 缺少必须参数
 * A mandatory parameter was not sent, was empty/null, or malformed.
 * Mandatory parameter '%s' was not sent, was empty/null, or malformed.
 * Param '%s' or '%s' must be sent, but both were empty/null!

#### -1103 无法识别的参数
 * An unknown parameter was sent.

#### -1104 冗余参数
 * Not all sent parameters were read.
 * Not all sent parameters were read; read '%s' parameter(s) but was sent '%s'.

#### -1105 空参数(仅有参数名)
 * A parameter was empty.
 * Parameter '%s' was empty.

#### -1106 非必需参数
 * A parameter was sent when not required.
 * Parameter '%s' sent when not required.

#### -1108 参数溢出
 * Parameter '%s' overflowed.

#### -1111 精度过高
 * Precision is over the maximum defined for this asset.

#### -1112 空白的orderbook
 * No orders on book for symbol.

#### -1114 错误地发送了不需要的TIF参数
 * TimeInForce parameter sent when not required.

#### -1115 无效的TIF参数
 * Invalid timeInForce.

#### -1116 无效的订单类型
 * Invalid orderType.

#### -1117 无效的订单方向
 * Invalid side.

#### -1118 空白的newClientOrderId
 * New client order ID was empty.

#### -1119 空白的originalClientOrderId
 * Original client order ID was empty.

#### -1120 无效的间隔(interval)
 * Invalid interval.

#### -1121 无效的交易对
 * Invalid symbol.

#### -1125 无效的listenKey
 * This listenKey does not exist.

#### -1127 查询间隔过长
 * Lookup interval is too big.
 * More than %s hours between startTime and endTime.

#### -1128 无效的可选参数组合
 * Combination of optional parameters invalid.

#### -1130 无效参数(值)
 * Invalid data sent for a parameter.
 * Data sent for parameter '%s' is not valid.

#### -1134 strategyType不符合需求
 * `strategyType` was less than 1000000. 

#### -1135 无效的JSON
 * Invalid JSON Request
 * JSON sent for parameter '%s' is not valid

#### -1145 无效的取消限制
 * `cancelRestrictions` has to be either `ONLY_NEW` or `ONLY_PARTIALLY_FILLED`.

#### -2010 新订单被拒绝
 * NEW_ORDER_REJECTED

#### -2011 订单取消被拒绝
 * CANCEL_REJECTED

#### -2013 不存在的订单
 * Order does not exist.

#### -2014 API Key格式无效
 * API-key format invalid.

#### -2015 API Key权限，例如该Key不存在、请求并非来自允许的IP范围、或者该接口对应权限未开放
 * Invalid API-key, IP, or permissions for action.

#### -2016 非交易窗口
 * No trading window could be found for the symbol. Try ticker/24hrs instead.

#### -2026 交易被归档
  * Order was canceled or expired with no executed qty over 90 days ago and has been archived.

## -1010 收到了错误消息
这个错误代码是由撮合引擎抛出的，引擎还会抛出2010和2011，具体原因需要参考下面列出的具体消息

错误消息                                                          | 描述
------------                                                     | ------------
"Unknown order sent."                                            | 找不到订单， (根据请求里发送的 `orderId`, `clOrdId`, `origClOrdId`) 
"Duplicate order sent."                                          | 客户自定义的订单号重复了
"Market is closed."                                              | 该交易对交易关闭了
"Account has insufficient balance for requested action."         | 账户金额不足。
"Market orders are not supported for this symbol."               | 该交易对无法发起市价单
"Iceberg orders are not supported for this symbol."              | 该交易对无法发起冰山订单
"Stop loss orders are not supported for this symbol."            | 该交易对无法发起止损单
"Stop loss limit orders are not supported for this symbol."      | 该交易对无法发起止损限价单
"Take profit orders are not supported for this symbol."          | 该交易对无法发起止盈单
"Take profit limit orders are not supported for this symbol."    | 该交易对无法发起止盈限价单
"Price * QTY is zero or less."                                   | 订单金额必须大于0
"IcebergQty exceeds QTY."                                        | 冰山订单中小订单的Quantity必须小于总的Quantity
"This action is disabled on this account."                       | 联系客户支持； 该账户已禁用了某些操作。
"This account may not place or cancel orders."                   | 联系客户支持： 该账户已被禁用了交易操作。
"Unsupported order combination"                                  | `orderType`, `timeInForce`, `stopPrice`, `icebergQty` 某些参数取某些值的时候另一些参数必须/不得提供。
"Order would trigger immediately."                               | 止盈、止损单必须在未来触发，如果条件太弱现在的市场行情就可以触发（通常是误操作填错了条件），就会报这个错误。
"Cancel order is invalid. Check origClOrdId and orderId."        | 撤销订单必须提供 `origClOrdId` 或者 `orderId` 中的一个。 
"Order would immediately match and take."                        | `LIMIT_MAKER` 订单如果按照规则会成为Taker，就会报此错。
"The relationship of the prices for the orders is not correct."  | `OCO`订单中设置的价格不符合报价规则：<br/> The rules are: <br/> `SELL Orders`: Limit Price > Last Price > Stop Price <br/>`BUY Orders`: Limit Price < Last Price < Stop Price
"OCO orders are not supported for this symbol"                   | `OCO`订单不支持该交易对。
"Quote order qty market orders are not support for this symbol." | 这个交易对，市价单不支持参数 `quoteOrderQty`
"Trailing stop orders are not supported for this symbol."        | 此symbol不支持 `trailingDelta`
"Order cancel-replace is not supported for this symbol."         | 此symbol不支持 `POST /api/v3/order/cancelReplace` 
"This symbol is not permitted for this account."                 | 账户没有权限在此交易对交易。
"This symbol is restricted for this account."                    | 账户没有权限在此交易对交易。
"Order was not canceled due to cancel restrictions."             | `cancelRestrictions` 设置为 `ONLY_NEW` 但订单状态不是 `NEW` <br/> 或 <br/> `cancelRestrictions` 设置为 `ONLY_PARTIALLY_FILLED` 但订单状态不是 `PARTIALLY_FILLED`。

## 关于 POST /api/v3/order/cancelReplace 的错误

### -2021 Order cancel-replace partially failed
收到该错误码代表撤单**或者**下单失败。

### -2022 Order cancel-replace failed.
收到该错误码代表撤单**和**下单都失败。

## 订单未能通过过滤器
错误信息 | 描述
------------ | ------------
"Filter failure: PRICE_FILTER" | 检查价格的上限、下限、步进间隔。
"Filter failure: PERCENT_PRICE" |检查订单中价格是否相对于过去N分钟的平均价格变动超过了百分之X。
"Filter failure: LOT_SIZE" | 检查订单中数量的上限、下线、步进间隔。
"Filter failure: MIN_NOTIONAL" | `price` * `quantity`，也就是订单金额，是否超过了最小值。
"Filter failure: ICEBERG_PARTS" | 冰山订单只能被分割成有限个小订单。
"Filter failure: MARKET_LOT_SIZE" | 与LOT_SIZE含义一致，只是对市价单生效。
"Filter failure: MAX_NUM_ORDERS" | 账户在该交易对下最多挂单数。
"Filter failure: MAX_NUM_ALGO_ORDERS" | 账户在该交易对下最多的止盈/止损挂单数。
"Filter failure: MAX_NUM_ICEBERG_ORDERS" | 账户在该交易对下最多的冰山订单数。
"Filter failure: EXCHANGE_MAX_NUM_ORDERS" | 账户在全交易所最多的挂单数。
"Filter failure: EXCHANGE_MAX_NUM_ALGO_ORDERS" | 账户在全交易所最多的止盈/止损挂单数。


