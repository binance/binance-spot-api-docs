# Commission Rates

**Disclaimer:** 

* The commissions and prices used here are fictional, and do not imply anything about the actual setup on the live exchange. 
* This applies only for the SPOT Exchange.

## What are Commission Rates?

These are the rates that determine the commission to be paid on trades when your order fills for any amount.

## What are the different types of rates?

There are 3 types:

* `standardCommission` - Standard commission rates on trades from the order.
* `taxCommission` - Tax commission rates on trades from the order.
* `discount` - Discount rate on standard commission if paying the commission with BNB.

## How do I know what the commission rates are?

You can find them using the following requests:

REST API: `GET /api/v3/account/commission`

WebSocket API: `account.commission`

You can also find out the commission rates to a trade from an order using the test order requests with `computeCommissionRates`.

## What is the difference between the response sending a test order with `computeCommissionRates` vs the response from querying commission rates?

This is an example of the former:

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

When using the order test request using `computeCommissionRates`, the `standardCommissionForOrder`  and `taxCommissionForOrder` shows the actual commission rate for trades from that order. Both `maker` and `taker` rates are returned regardless of the order parameters. 

The response for query commission rates gives the current commission rate for that symbol for your account.

## How is the commission calculated?

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

Note that the discount **does not apply to tax commissions**.

```
Tax Commission (in BNB) = Tax commission * BNB exchange rate
                        = 0.04022988 * (1/260)
                        = 0.00015473
```

```
Total Commission (in BNB) = Standard commission (Discounted) + Tax commission (in BNB)
                          = 0.000010091 + 0.00015473
                          = 0.00016482
```

If you do not have enough BNB to pay the discounted commission, the full commission will be taken out of your received amount of USDT instead.




