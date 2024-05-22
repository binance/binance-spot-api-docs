# 佣金率

**免责声明：**

* 本文所用的佣金和价格都是虚构的，并不代表现实交易中的设置。
* 本内容只适用于现货交易所。

## 什么是佣金率？

这些比率是用来决定当您的任何金额订单成交后，您所需要支付的佣金金额数目。

## 佣金率有哪些不同的类型？

有以下3种类型：

* 标准佣金（`standardCommission`） - 来自订单的标准交易佣金率。
* 税务佣金（`taxCommission`） - 来自订单的税费佣金率。
* 折扣（`discount`） - 如果使用`BNB`支付佣金，在标准佣金基础上可以得到的折扣率。

## 我怎么才能知道佣金率是多少？

您可以通过以下请求找到它们：

REST API: `GET /api/v3/account/commission`

WebSocket API: `account.commission`

您也可以通过在测试订单请求中使用 `computeCommissionRates` 来找出订单交易的佣金比率。

## 在发送测试订单中使用`computeCommissionRates`得到的响应与查询佣金率的响应之间有什么不同？

以下是有关前者的一个例子：


```json
{
  "standardCommissionForOrder": {
    "maker": "0.00000050",
    "taker": "0.00000060"
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

当使用带有`computeCommissionRates`的订单测试请求时，`standardCommissionForOrder` 和 `taxCommissionForOrder` 显示了该订单交易的实际佣金率。无论订单参数如何设置，响应均会返回`maker`和`taker`的费率。

而查询佣金率的响应则提供了针对于您的帐户中该交易对的当前佣金率。


## 佣金是怎么计算的？

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

请注意，折扣（`discount`）**不适用于税务佣金（`税务佣金`）**。

```
税务佣金 (以BNB支付) = 税务佣金 * BNB 汇率
                        = 0.04022988 * (1/260)
                        = 0.00015473
```

```
佣金总数 (以BNB支付) = 标准佣金(打折后) + 税务佣金 (以BNB支付)
                          = 0.000010091 + 0.00015473
                          = 0.00016482
```

如果您的`BNB`余额不足以支付折扣后的佣金，那么全部佣金将从您所接收的`USDT`金额中扣除。




