<a id="websocket"></a>

<a id="websocket-账户接口"></a>

<a id="user-data-streams-for-binance"></a>

# 用户数据流

<a id="general-information"></a>

## 一般信息

- 通过使用 API Key 订阅
  [WebSocket API](web-socket-api_CN.md#subscribe-to-user-data-stream-user_stream)。
- 支持 [SBE](faqs/sbe_faq_CN.md) 和 JSON 输出格式。
- 账户事件以 **实时** 方式推送。
- JSON 数据中的所有时间戳默认均为 **毫秒**。
- 如果您持有或交易任何名称包含非 ASCII 字符的资产或交易对，那么事件中可能包含以 UTF-8 编码的非 ASCII 字符。

<a id="user-data-stream-events"></a>

## 用户数据流事件

<a id="account-update"></a>

### 账户更新

每当帐户余额发生更改时，都会发送一个事件`outboundAccountPosition`，其中包含可能由生成余额变动的事件而变动的资产。

```javascript
{
    "subscriptionId": 0,
    "event": {
        "e": "outboundAccountPosition", // 事件类型
        "E": 1564034571105,             // 事件时间
        "u": 1564034571073,             // 账户末次更新时间戳
        // 余额
        "B": [
            {
                "a": "ETH",          // 资产名称
                "f": "10000.000000", // 可用余额
                "l": "0.000000"      // 冻结余额
            }
        ]
    }
}
```

<a id="balance-update"></a>

### 余额更新

当下列情形发生时更新:

- 账户发生充值或提取
- 交易账户之间发生划转(例如 现货向杠杆账户划转)

**Payload**

```javascript
{
    "subscriptionId": 0,
    "event": {
        "e": "balanceUpdate", // 事件类型
        "E": 1573200697110,   // 事件时间
        "a": "BTC",           // 资产名称
        "d": "100.00000000",  // 余额增量
        "T": 1573200697068    // 清算时间
    }
}
```

<a id="executionReport"></a>

<a id="order-update"></a>

### 订单更新

订单通过`executionReport`事件进行更新。

**Payload:**

```javascript
{
    "subscriptionId": 0,
    "event": {
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
        "F": "0.00000000",             // 冰山订单数量
        "g": -1,                       // OCO订单 OrderListId
        "C": "",                       // 原始订单自定义ID（原始订单，指撤单操作的对象。撤单本身被视为另一个订单）
        "x": "NEW",                    // 本次事件的具体执行类型
        "X": "NEW",                    // 订单的当前状态
        "r": "NONE",                   // 订单被拒绝的原因；请参阅订单被拒绝的原因了解更多信息
        "i": 4293153,                  // 系统的订单ID
        "l": "0.00000000",             // 订单末次成交量
        "z": "0.00000000",             // 订单累计已成交量
        "L": "0.00000000",             // 订单末次成交价格
        "n": "0",                      // 佣金数量
        "N": null,                     // 佣金资产类别
        "T": 1499405658657,            // 成交时间
        "t": -1,                       // 交易ID
        "v": 3,                        // 被阻止的交易Id；仅在订单因为STP被阻止时显示
        "I": 8641984,                  // Execution ID
        "w": true,                     // 订单是否在订单簿上？
        "m": false,                    // 该成交是作为挂单成交吗？
        "M": false,                    // 请忽略
        "O": 1499405658657,            // 订单创建时间
        "Z": "0.00000000",             // 订单累计已成交金额
        "Y": "0.00000000",             // 订单末次成交金额
        "Q": "0.00000000",             // 报价订单数量
        "W": 1499405658657,            // Working Time; 订单被添加到 order book 的时间
        "V": "NONE",                   // SelfTradePreventionMode
        "d": 4,                        // 追踪止损Delta
        "D": 1668680518494,            // 追踪止损时间
        "j": 1,                        // 策略ID
        "J": 1000000,                  // 策略类型
        "A": "3.000000",               // Prevented Quantity
        "B": "3.000000",               // Last Prevented Quantity
        "u": 1,                        // Trade Group Id
        "U": 37,                       // Counter Order Id
        "Cs": "BTCUSDT",               // Counter Symbol
        "pl": "2.123456",              // Prevented Execution Quantity
        "pL": "0.10000001",            // Prevented Execution Price
        "pY": "0.21234562",            // Prevented Execution Quote Qty
        "b": "ONE_PARTY_TRADE_REPORT", // Match Type
        "a": 1234,                     // Allocation ID
        "k": "SOR",                    // Working Floor
        "uS": true,                    // UsedSor
        "gP": "PRIMARY_PEG",           // 挂钩价格类型
        "gOT": "PRICE_LEVEL",          // 挂钩偏移类型
        "gOV": 5,                      // 挂钩偏移值
        "gp": "1.00000000",            // 挂钩价格
        "eR": "INSUFFICIENT_LIQUIDITY" // 过期原因。当订单已过期时出现。
    }
}
```

**备注:** 通过将`Z`除以`z`可以找到平均价格。

<a id="conditional-fields-in-execution-report"></a>

#### `executionReport` 中的特定条件时才会出现的字段

这些字段仅在满足特定条件时才会出现。有关这些参数的更多信息，请参阅
[现货交易API术语表](faqs/spot_glossary_CN.md)。

| 字段  | 名称                          | 描述                                             | 示例                             |
| ----- | ----------------------------- | ------------------------------------------------ | -------------------------------- |
| `d`   | Trailing Delta                | 出现在追踪止损订单中。                           | `"d": 4`                         |
| `D`   | Trailing Time                 | 出现在追踪止损订单中。                           | `"D": 1668680518494`             |
| `j`   | Strategy Id                   | 如果在请求中添加了`strategyId`参数，则会出现。   | `"j": 1`                         |
| `J`   | Strategy Type                 | 如果在请求中添加了`strategyType`参数，则会出现。 | `"J": 1000000`                   |
| `v`   | Prevented Match Id            | 只有在因为 STP 导致订单失效时可见。              | `"v": 3`                         |
| `A`   | Prevented Quantity            | 只有在因为 STP 导致订单失效时可见。              | `"A": "3.000000"`                |
| `B`   | Last Prevented Quantity       | 只有在因为 STP 导致订单失效时可见。              | `"B": "3.000000"`                |
| `u`   | Trade Group Id                | 只有在因为 STP 导致订单失效时可见。              | `"u": 1`                         |
| `U`   | Counter Order Id              | 只有在因为 STP 导致订单失效时可见。              | `"U": 37`                        |
| `Cs`  | Counter Symbol                | 只有在因为 STP 导致订单失效时可见。              | `"Cs": "BTCUSDT"`                |
| `pl`  | Prevented Execution Quantity  | 只有在因为 STP 导致订单失效时可见。              | `"pl": "2.123456"`               |
| `pL`  | Prevented Execution Price     | 只有在因为 STP 导致订单失效时可见。              | `"pL": "0.10000001"`             |
| `pY`  | Prevented Execution Quote Qty | 只有在因为 STP 导致订单失效时可见。              | `"pY": "0.21234562"`             |
| `W`   | Working Time                  | 只有在订单在订单簿上时可见                       | `"W": 1668683798379`             |
| `b`   | Match Type                    | 只有在订单有分配时可见                           | `"b": "ONE_PARTY_TRADE_REPORT"`  |
| `a`   | Allocation ID                 | 只有在订单有分配时可见                           | `"a": 1234`                      |
| `k`   | Working Floor                 | 只有在订单可能有分配时可见                       | `"k": "SOR"`                     |
| `uS`  | UsedSor                       | 只有在订单使用 SOR 时可见                        | `"uS": true`                     |
| `gP`  | Pegged Price Type             | 仅出现在挂钩订单中                               | `"gP": "PRIMARY_PEG"`            |
| `gOT` | Pegged Offset Type            | 仅出现在挂钩订单中                               | `"gOT": "PRICE_LEVEL"`           |
| `gOV` | Pegged Offset Value           | 仅出现在挂钩订单中                               | `"gOV": 5`                       |
| `gp`  | Pegged Price                  | 仅出现在挂钩订单中                               | `"gp": "1.00000000"`             |
| `eR`  | Expiry Reason                 | 出现在订单已过期时。                             | `"eR": "INSUFFICIENT_LIQUIDITY"` |

<a id="order-reject-reason"></a>

#### 订单拒绝原因

有关更多详细信息，请查阅
[错误代码汇总](errors_CN.md#messages-for--1010-error_msg_received--2010-new_order_rejected--2011-cancel_rejected-and--2038-order_amend_rejected)
文档中的错误消息。

| 拒绝原因 (`r`)                         | 错误信息                                                        |
| -------------------------------------- | --------------------------------------------------------------- |
| `NONE`                                 | N/A (i.e. The order was not rejected.)                          |
| `INSUFFICIENT_BALANCES`                | "Account has insufficient balance for requested action."        |
| `STOP_PRICE_WOULD_TRIGGER_IMMEDIATELY` | "Order would trigger immediately."                              |
| `WOULD_MATCH_IMMEDIATELY`              | "Order would immediately match and take."                       |
| `OCO_BAD_PRICES`                       | "The relationship of the prices for the orders is not correct." |

如果是一个订单组，则除了显示 `executionReport` 事件外，还将显示一个名为 `ListStatus` 的事件。

**Payload**

```javascript
{
    "subscriptionId": 0,
    "event": {
        "e": "listStatus",             // 事件类型
        "E": 1564035303637,            // 事件时间
        "s": "ETHBTC",                 // 交易对
        "g": 2,                        // OrderListId
        "c": "OCO",                    // Contingency 类型
        "l": "EXEC_STARTED",           // List 状态类型
        "L": "EXECUTING",              // List 订单类型
        "r": "NONE",                   // List 被拒绝的原因
        "C": "F4QN4G8DlFATFlIUQ0cjdD", // List Client Order ID
        "T": 1564035303625,            // 成交时间
        // 对象数组
        "O": [
            {
                "s": "ETHBTC",                // 交易对
                "i": 17,                      // 系统的订单ID
                "c": "AJYsMjErWJesZvqlJCTUgL" // clientOrderId
            },
            {
                "s": "ETHBTC",
                "i": 18,
                "c": "bfYPSQdLoqAJeNrOr9adzq"
            }
        ]
    }
}
```

**可能的执行类型:**

每个 `x` / `X` 值的含义请参阅枚举定义页面的 [可能的执行类型](enums_CN.md#execution-types)。

<a id="order-expiry-reason"></a>

#### 订单过期原因

每个 `eR` 值的含义请参阅枚举定义页面的 [订单过期原因](enums_CN.md#expiry-reasons)。

请查阅 [枚举定义](enums_CN.md) 文档获取更多枚举定义。

<a id="event-stream-terminated"></a>

## 事件流已终止

`eventStreamTerminated` 会在以下情况下发送：

- 当 [Listen Token 订阅](https://developers.binance.com/zh-CN/docs/products/margin-trading/listen-token-data-stream) 因 Token 过期而失效时。
- 在发送 [`session.logout`](web-socket-api_CN.md#log-out-of-the-session)
  方法后，[登录订阅](web-socket-api_CN.md#log-in-with-api-key-user_data) 结束时。
- 通过
  [`userDataStream.unsubscribe`](web-socket-api_CN.md#unsubscribe-from-user-data-stream)
  方法终止订阅时。

**Payload:**

```javascript
{
    "subscriptionId": 0,
    "event": {
        "e": "eventStreamTerminated", // 事件类型
        "E": 1728973001334            // 事件时间
    }
}
```

<a id="external-lock-update"></a>

## 外部锁定更新

当您的现货钱包余额被外部系统锁定/解锁时 （例如，当用作保证金抵押品时），新事件 `externalLockUpdate`
将会被发送。

**Payload:**

```javascript
{
    "subscriptionId": 0,
    "event": {
        "e": "externalLockUpdate", // 事件类型
        "E": 1581557507324,        // 事件时间
        "a": "NEO",                // 资产
        "d": "10.00000000",        // 余额变动量
        "T": 1581557507268         // 交易时间
    }
}
```
