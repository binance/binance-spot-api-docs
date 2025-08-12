# 佣金率

**免责声明：**

* 本文所用的佣金和价格都是虚构的，并不代表现实交易中的设置。
* 本内容只适用于现货交易所。

### 什么是佣金率？

这些比率是用来决定当您的任何金额订单成交后，您所需要支付的佣金金额数目。

### 佣金率有哪些不同的类型？

有以下3种类型：
* 标准佣金（`standardCommission`） - 来自订单的标准交易佣金率。
* 税务佣金（`taxCommission`） - 来自订单的税费佣金率。
* 特殊佣金（`specialCommission`） - 在特定情况下，将会被收取的额外佣金。

标准佣金率可能会被降低，具体情况取决于特定交易对的促销活动、可适用的折扣等。

### 我怎么才能知道佣金率是多少？

您可以通过以下请求找到它们：

REST API: `GET /api/v3/account/commission`

WebSocket API: `account.commission`

您也可以通过在测试订单请求中使用 `computeCommissionRates` 来找出订单交易的佣金比率。

<a id="test-order-diferences"></a>
### 在发送测试订单中使用`computeCommissionRates`得到的响应与查询佣金率的响应之间有什么不同？

使用 `computeCommissionRates` 的测试订单会返回该特定订单的详细佣金率：


```json
{
  "standardCommissionForOrder": {
    "maker": "0.00000050",
    "taker": "0.00000060"
  },
  "specialCommissionForOrder": {
    "maker": "0.05000000",
    "taker": "0.06000000"
  },
  "taxCommissionForOrder": {
    "maker": "0.00000228",
    "taker": "0.00000230"
  },
  "discount": {
    "enabledForAccount": true,
    "enabledForSymbol": true,
    "discountAsset": "BNB",
    "discount": "0.25000000"
  }
}
```
注意：因为买方/卖方佣金已根据订单方向计算在内，所以买方/卖方佣金不会被单独显示出来。

相反，查询佣金率的响应则提供了针对于您的帐户中该交易对的当前佣金率。

```json
{
  "symbol": "BTCUSDT",
  "standardCommission": {
    "maker": "0.00000040",
    "taker": "0.00000050",
    "buyer": "0.00000010",
    "seller": "0.00000010"
  },
  "specialCommission": {
    "maker": "0.04000000",
    "taker": "0.05000000",
    "buyer": "0.01000000",
    "seller": "0.01000000"
  },
  "taxCommission": {
    "maker": "0.00000128",
    "taker": "0.00000130",
    "buyer": "0.00000100",
    "seller": "0.00000100"
  },
  "discount": {
    "enabledForAccount": true,
    "enabledForSymbol": true,
    "discountAsset": "BNB",
    "discount": "0.25000000"
  }
}
```


### 佣金是怎么计算的？

以下面的这个佣金配置为例：

```json
{
  "symbol": "BTCUSDT",
  "standardCommission": {
    "maker": "0.00000010",
    "taker": "0.00000020",
    "buyer": "0.00000030",
    "seller": "0.00000040"
  },
  "specialCommission": {
    "maker": "0.01000000",
    "taker": "0.02000000",
    "buyer": "0.03000000",
    "seller": "0.04000000"
  },
  "taxCommission": {
    "maker": "0.00000112",
    "taker": "0.00000114",
    "buyer": "0.00000118",
    "seller": "0.00000116"
  },
  "discount": {
    "enabledForAccount": true,
    "enabledForSymbol": true,
    "discountAsset": "BNB",
    "discount": "0.25000000"
  }
}
```

如果您使用了下列参数来下一个订单，该订单立即执行并通过一次交易就全部成交：

|参数      | 取值 |
|---      | ---  |
|symbol   |BTCUSDT|
|price    |35,000|
|quantity |0.49975|
|side     |SELL   |
|type     |MARKET |

由于您卖出了 `BTC` 来换取 `USDT` ，佣金将以 `USDT` 或 `BNB` 形式支付。

在计算标准佣金（`standard commission`）时，所接收的金额会与费率之和相乘。

由于此订单在`卖方` （`SELL`）一侧，所接收的金额是`名义价值` （`notional value`）。 对于`买方` （`BUY`）一侧的订单，所接收的金额则是`数量` （`quantity`）。
因为订单类型为`市价` （`MARKET`）的关系，使得此订单成为交易的`吃单方` （`taker`）。

```
标准佣金 = 名义价值 * (taker + seller)
                    = (35000 * 0.49975) * (0.00000020 + 0.00000040)
                    = 17491.25000000 * 0.00000060
                    = 0.01049475 USDT
```

如果适用，税务佣金（`tax commission`）的计算方式类似于标准佣金：

```
税务佣金 = 名义价值 * (taker + seller)
               = (35000 * 0.49975) * (0.00000114 + 0.00000116)
               = 17491.25000000 * 0.00000230
               = 0.04022988 USDT
```

如果适用，特殊佣金（`special commission`）的计算方式如下：

```
特殊佣金 = 名义价值 * (taker + seller)
               = (35000 * 0.49975) * (0.02000000 + 0.04000000)
               = 17491.25000000 * 0.06000030
               = 1049.47500000 USDT
```

如果您不使用`BNB`支付佣金，佣金总数会被加起来并从您所接收的`USDT`金额中扣除。

由于`discount`下的`enabledforAccount`和`enabledForSymbol`被设置为`true`，这意味着如果您所持有的余额足够，那么佣金将用`BNB`支付。

如果用`BNB`支付，那么基于折扣（`discount`），您所需要支付的标准佣金将会减少。

首先，标准佣金和税务佣金将根据汇率转换成`BNB`。在这个例子中，假设1 BNB = 260 USDT。

```
标准佣金(打折后以BNB支付) = (标准佣金 * BNB 汇率) * 折扣
                                            = (0.01049475 * 1/260) * 0.25
                                            = 0.000040364 * 0.25
                                            = 0.000010091
```

请注意，折扣（`discount`）**不适用于税务佣金（`taxCommission`）或特殊佣金（`special commission`）**。

```
税务佣金 (以BNB支付) = 税务佣金 * BNB 汇率
                        = 0.04022988 * (1/260)
                        = 0.00015473

特殊佣金 (以BNB支付) = 特殊佣金 * BNB 汇率
                        = 1049.47500000 * (1/260)
                        = 4.036442308

```

```
佣金总数 (以BNB支付) = 标准佣金(打折后) + 税务佣金 (以BNB支付) + 特殊佣金 (以BNB支付)
                        = 0.000010091 + 0.00015473 + 4.036442308
                        = 4.036607129
```

如果您的`BNB`余额不足以支付折扣后的佣金，那么全部佣金将从您所接收的`USDT`金额中扣除。
