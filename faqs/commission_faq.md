# Commission Rates

**Disclaimer:**

* The commissions and prices used here are fictional, and do not imply anything about the actual setup on the live exchange.
* This applies only for the SPOT Exchange.

### What are Commission Rates?

These are the rates that determine the commission to be paid on trades when your order fills for any amount.

### What are the different types of rates?

There are 3 types:
* `standardCommission` - Standard commission rates on trades from the order.
* `taxCommission` - Tax commission rates on trades from the order.
* `specialCommission` - Extra commission that will be added in specific circumstances.

Standard commission rate may be reduced, depending on promotions for specific trading pairs, applicable discounts, etc.

### How do I know what the commission rates are?

You can find them using the following requests:

REST API: `GET /api/v3/account/commission`

WebSocket API: `account.commission`

You can also find out the commission rates to a trade from an order using the test order requests with `computeCommissionRates`.

<a id="test-order-diferences"></a>
### What is the difference between the response sending a test order with `computeCommissionRates` vs the response from querying commission rates?

A test order with `computeCommissionRates` returns detailed commission rates for that specific order:

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
Note: It does not show buyer/seller commissions separately, as these are already taken into account based on the order side.

In contrast, querying commission rates returns your current commission rates for the symbol on your account.

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

### How is the commission calculated?

Using an example commission configuration:

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

If you placed an order with the following parameters which took immediately and fully filled in a single trade:

|Parameter| Value|
|---      | ---  |
|symbol   |BTCUSDT|
|price    |35,000|
|quantity |0.49975|
|side     |SELL   |
|type     |MARKET |

Since you sold BTC for USDT, the commission will be paid either in USDT or BNB.

When standard commission is calculated, the received amount is multiplied with the sum of the rates.

Since this order is on the `SELL` side, the received amount is the notional value. (For orders on the `BUY` side, the received amount would be `quantity`.)
The order type was `MARKET`, making this the taker order for the trade.

```
Standard Commission = Notional value * (taker + seller)
                    = (35000 * 0.49975) * (0.00000020 + 0.00000040)
                    = 17491.25000000 * 0.00000060
                    = 0.01049475 USDT
```

Tax commission (if applicable) is calculated similarly:

```
Tax commission = Notional value * (taker + seller)
               = (35000 * 0.49975) * (0.00000114 + 0.00000116)
               = 17491.25000000 * 0.00000230
               = 0.04022988 USDT
```

Special commission (if applicable) is calculated as:

```
Special commission = Notional value * (taker + seller)
               = (35000 * 0.49975) * (0.02000000 + 0.04000000)
               = 17491.25000000 * 0.06000030
               = 1049.47500000 USDT
```

If not paying in BNB, the total commission are summed up and deducted from your received amount of `USDT`.

Since `enabledforAccount` and `enabledForSymbol` under `discount` is set to `true`, this means the commission will be paid in BNB assuming you have a sufficient balance.

If paying with BNB, then the standard commission will be reduced based on the `discount`.

First the standard commission and tax commission will be converted into BNB based on the exchange rate. For this example, assume that 1 BNB = 260 USDT.

```
Standard commission (Discounted and in BNB) = (Standard commission * BNB exchange rate) * discount
                                            = (0.01049475 * 1/260) * 0.25
                                            = 0.000040364 * 0.25
                                            = 0.000010091
```

Note that the discount **does not apply to tax commissions or special commissions**.

```
Tax Commission (in BNB) = Tax commission * BNB exchange rate
                        = 0.04022988 * (1/260)
                        = 0.00015473

Special Commission (in BNB) = Special commission * BNB exchange rate
                        = 1049.47500000 * (1/260)
                        = 4.036442308

```

```
Total Commission (in BNB) = Standard commission (Discounted) + Tax commission (in BNB) + Special commission (in BNB)
                          = 0.000010091 + 0.00015473 + 4.036442308
                          = 4.036607129
```

If you do not have enough BNB to pay the discounted commission, the full commission will be taken out of your received amount of USDT instead.
