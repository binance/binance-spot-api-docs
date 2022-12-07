# Spot Trailing Stop order FAQ

## What is a trailing stop order?

Trailing stop is a type of contingent order with a dynamic trigger price influenced by price changes in the market. For the SPOT API, the change required to trigger order entry is specified in the `trailingDelta` parameter, and is defined in BIPS.

Intuitively, trailing stop orders allow unlimited price movement in a direction that is beneficial for the order, and limited movement in a detrimental direction.

Buy orders: _low_ prices are good. Unlimited price _decreases_ are allowed but the order will trigger after a price _increase_ of the supplied delta, relative to the _lowest_ trade price since submission.

Sell orders: _high_ prices are good. Unlimited price _increases_ are allowed but the order will trigger after a price _decrease_ of the supplied delta, relative to the _highest_ trade price since submission.

## What are BIPs?

Basis Points, also known as BIP or BIPS, are used to indicate a percentage change.

BIPS conversion reference:

BIPS | Percentage | Multiplier
---- | ---------- | ----------
1 | 0.01% | 0.0001
10 | 0.1% | 0.001
100 | 1% | 0.01
1000 | 10% | 0.1

For example, a `STOP_LOSS` `SELL` order with a `trailingDelta` of 100 is a trailing stop order which will be triggered after a price decrease of 1% from the highest price after placing the order.

## What order types can be trailing stop orders?

Trailing stop orders are supported for contingent orders such as `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT`.

OCO orders also support trailing stop orders in the contingent leg. In this scenario if the trailing stop condition is triggered, the limit leg of the OCO order will be canceled.

## How do I place a trailing stop order?

Trailing stop orders are entered the same way as regular `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, or `TAKE_PROFIT_LIMIT` orders, but with an additional `trailingDelta` parameter. This parameter must be within the range of the `TRAILING_DELTA` filter for that symbol.

Unlike regular contingent orders, the `stopPrice` parameter is optional for trailing stop orders. If it is provided then the order will only start tracking price changes after the `stopPrice` condition is met. If the `stopPrice` parameter is omitted then the order starts tracking price changes from the next trade.

## What kind of price changes will trigger my trailing stop order?

Trailing order type | Side | Stop price condition       | Market price movement required to trigger
------------------- | ---- | -------------------------- | ----------------------------------
`TAKE_PROFIT`       | SELL | market price >= stop price | *decrease* from maximum
`TAKE_PROFIT_LIMIT` | SELL | market price >= stop price | *decrease* from maximum
`STOP_LOSS`         | SELL | market price <= stop price | *decrease* from maximum
`STOP_LOSS_LIMIT`   | SELL | market price <= stop price | *decrease* from maximum
`STOP_LOSS`         | BUY  | market price >= stop price | *increase* from minimum
`STOP_LOSS_LIMIT`   | BUY  | market price >= stop price | *increase* from minimum
`TAKE_PROFIT`       | BUY  | market price <= stop price | *increase* from minimum
`TAKE_PROFIT_LIMIT` | BUY  | market price <= stop price | *increase* from minimum

## How do I pass the `TRAILING_DELTA` filter?

For `STOP_LOSS` `BUY`, `STOP_LOSS_LIMIT` `BUY`, `TAKE_PROFIT` `SELL`, and `TAKE_PROFIT_LIMIT` `SELL` orders:

* `trailingDelta` >= `minTrailingAboveDelta`
* `trailingDelta` <= `maxTrailingAboveDelta`

For `STOP_LOSS` `SELL`, `STOP_LOSS_LIMIT` `SELL`, `TAKE_PROFIT` `BUY`, and `TAKE_PROFIT_LIMIT` `BUY` orders:

* `trailingDelta` >= `minTrailingBelowDelta`
* `trailingDelta` <= `maxTrailingBelowDelta`

## Trailing Stop Order Scenarios

### Scenario A - Trailing Stop Loss Limit Buy Order

At `12:01:00` there is a trade at a price of 40,000 and a `STOP_LOSS_LIMIT` order is placed on the `BUY` side of the exchange. The order has of a `stopPrice` of 44,000, a `trailingDelta` of 500 (5%), and a limit `price` of 45,000.

Between `12:01:00` and `12:02:00` a series of linear trades lead to a decrease in last price, ending at 37,000. This is a price decrease of 7.5% or 750 BIPS, well exceeding the order's `trailingDelta`. However since the order has not started price tracking, the price movement is ignored and the order remains contingent.

Between `12:02:00` and `12:03:00` a series of linear trades lead to an increase in last price. When a trade is equal to, or surpasses, the `stopPrice` the order starts tracking price changes immediately; the first trade that meets this condition sets the "lowest price". In this case, the lowest price is 44,000 and if there is a 500 BIPS increase from 44,000 then the order will trigger. The series of linear trades continue to increase the last price, ending at 45,000.

Between `12:03:00` and `12:04:00` a series of linear trades lead to an increase in last price, ending at 46,000. This is an increase of ~454 BIPS from the order's previously noted lowest price, but it's not large enough to trigger the order.

Between `12:04:00` and `12:05:00` a series of linear trades lead to a decrease in last price, ending at 42,000. This is a decrease from the order's previously noted lowest price. If there is a 500 BIPS increase from 42,000 then the order will trigger.

Between `12:05:00` and `12:05:30` a series of linear trades lead to an increase in last price to 44,100. This trade is equal to, or surpasses, the order's requirement of 500 BIPS, as `44,100 = 42,000 * 1.05`. This causes the order to trigger and start working against the order book at its limit price of 45,000.

<img alt="image" src="https://user-images.githubusercontent.com/17701918/167370103-ab3b4c05-1e13-4a25-b99a-42f9e4d6adc8.png" />

### Scenario B - Trailing Stop Loss Limit Sell Order

At `12:01:00` there is a trade at a price of 40,000 and a `STOP_LOSS_LIMIT` order is placed on the `SELL` side of the exchange. The order has of a `stopPrice` of 39,000, a `trailingDelta` of 1000 (10%), and a limit `price` of 38,000.

Between `12:01:00` and `12:02:00` a series of linear trades lead to an increase in last price, ending at 41,500.

Between `12:02:00` and `12:03:00` a series of linear trades lead to a decrease in last price. When a trade is equal to, or surpasses, the `stopPrice` the order starts tracking price changes immediately; the first trade that meets this condition sets the "highest price". In this case, the highest price is 39,000 and if there is a 1000 BIPS decrease from 39,000 then the order will trigger.

Between `12:03:00` and `12:04:00` a series of linear trades lead to a decrease in last price, ending at 37,000. This is a decrease of ~512 BIPS from the order's previously noted highest price, but it's not large enough to trigger the order.

Between `12:04:00` and `12:05:00` a series of linear trades lead to an increase in last price, ending at 41,000. This is an increase from the order's previously noted highest price. If there is a 1000 BIPS decrease from 41,000 then the order will trigger.

Between `12:05:00` and `12:05:30` a series of linear trades lead to a decrease in last price to 36,900. This trade is equal to, or surpasses, the order's requirement of 1000 BIPS, as `36,900 = 41,000 * 0.90`. This causes the order to trigger and start working against the order book at its limit price of 38,000.

<img alt="image" src="https://user-images.githubusercontent.com/17701918/167370383-eb813cc1-d9b8-4a94-896c-a1a29551e09d.png" />

### Scenario C - Trailing Take Profit Limit Buy Order

At `12:01:00` there is a trade at a price of 40,000 and a `TAKE_PROFIT_LIMIT` order is placed on the `BUY` side of the exchange. The order has of a `stopPrice` of 38,000, a `trailingDelta` of 850 (8.5%), and a limit `price` of 38,500.

Between `12:01:00` and `12:02:00` a series of linear trades lead to an increase in last price, ending at 42,000.

Between `12:02:00` and `12:03:00` a series of linear trades lead to a decrease in last price. When a trade is equal to, or surpasses, the `stopPrice` the order starts tracking price changes immediately; the first trade that meets this condition sets the "lowest price". In this case, the lowest price is 38,000 and if there is a 850 BIPS increase from 38,000 then the order will trigger. 

The series of linear trades continues to decrease the last price, ending at 37,000. If there is a 850 BIPS increase from 37,000 then the order will trigger. 

Between `12:03:00` and `12:04:00` a series of linear trades lead to an increase in last price, ending at 39,000. This is an increase of ~540 BIPS from the order's previously noted lowest price, but it's not large enough to trigger the order.

Between `12:04:00` and `12:05:00` a series of linear trades lead to a decrease in last price, ending at 38,000. It does not surpass the order's previously noted lowest price, resulting in no change to the order's trigger price.

Between `12:05:00` and `12:05:30` a series of linear trades lead to an increase in last price to 40,145. This trade is equal to, or surpasses, the order's requirement of 850 BIPS, as `40,145 = 37,000 * 1.085`. This causes the order to trigger and start working against the order book at its limit price of 38,500.

<img alt="image" src="https://user-images.githubusercontent.com/17701918/167370339-f1b83c76-790b-4108-8c9a-db2d89a4850f.png" />

### Scenario D - Trailing Take Profit Limit Sell Order

At `12:01:00` there is a trade at a price of 40,000 and a `TAKE_PROFIT_LIMIT` order is placed on the `SELL` side of the exchange. The order has of a `stopPrice` of 42,000, a `trailingDelta` of 750 (7.5%), and a limit `price` of 41,000.

Between `12:01:00` and `12:02:00` a series of linear trades lead to an increase in last price, ending at 41,500.

Between `12:02:00` and `12:03:00` a series of linear trades lead to a decrease in last price, ending at 39,000.

Between `12:03:00` and `12:04:00` a series of linear trades lead to an increase in last price. When a trade is equal to, or surpasses, the `stopPrice` the order starts tracking price changes immediately; the first trade that meets this condition sets the "highest price". In this case, the highest price is 42,000 and if there is a 750 BIPS decrease from 42,000 then the order will trigger. 

The series of linear trades continues to increase the last price, ending at 45,000. If there is a 750 BIPS decrease from 45,000 then the order will trigger.

Between `12:04:00` and `12:05:00` a series of linear trades lead to a decrease in last price, ending at 44,000. This is a decrease of ~222 BIPS from the order's previously noted highest price, but it's not large enough to trigger the order.

Between `12:05:00` and `12:06:00` a series of linear trades lead to an increase in last price, ending at 46,500. This is an increase from the order's previously noted highest price. If there is a 750 BIPS decrease from 46,500 then the order will trigger.

Between `12:06:00` and `12:06:50` a series of linear trades lead to a decrease in last price to 43,012.5. This trade is equal to, or surpasses, the order's requirement of 750 BIPS, as `43,012.5 = 46,500 * 0.925`. This causes the order to trigger and start working against the order book at its limit price of 41,000.

<img alt="image" src="https://user-images.githubusercontent.com/17701918/167370298-172b227a-198d-46ee-a385-5cc267dc253b.png" />

### Scenario E - Trailing Stop Order Without A Stop Price

At `12:01:00` there is a trade at a price of 40,000 and a `STOP_LOSS_LIMIT` order is placed on the `SELL` side of the exchange. The order has a `trailingDelta` of 700 (7%), a limit `price` of 39,000 and no `stopPrice`. The order starts tracking price changes once placed. If there is a 700 BIPS decrease from 40,000 then the order will trigger.

Between `12:01:00` and `12:02:00` a series of linear trades lead to an increase in last price, ending at 42,000. This is an increase from the order's previously noted highest price. If there is a 700 BIPS decrease from 42,000 then the order will trigger.

Between `12:02:00` and `12:03:00` a series of linear trades lead to a decrease in last price, ending at 39,500. This is a decrease of ~595 BIPS from the order's previously noted highest price, but it's not large enough to trigger the order.

Between `12:03:00` and `12:04:00` a series of linear trades lead to an increase in last price, ending at 45,500. This is an increase from the order's previously noted highest price. If there is a 700 BIPS decrease from 45,500 then the order will trigger.

Between `12:04:00` and `12:04:45` a series of linear trades lead to a decrease in last price to 42,315. This trade is equal to, or surpasses, the order's requirement of 700 BIPS, as `42,315 = 45,500 * 0.93`. This causes the order to trigger and start working against the order book at its limit price of 39,000.

<img alt="image" src="https://user-images.githubusercontent.com/17701918/167370616-17d3295a-3e7c-4314-aa13-ad44e685a311.png" />

## Trailing Stop Order Examples

Assuming a last price of 40,000.

Placing a trailing stop `STOP_LOSS_LIMIT BUY` order, with a price of 42,000.0 and a trailing stop of 5%.
```bash
# Excluding stop price
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=BUY&type=STOP_LOSS_LIMIT&timeInForce=GTC&quantity=0.01&price=42000&trailingDelta=500&timestamp=<timestamp>&signature=<signature>'

# Including stop price of 43,000
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=BUY&type=STOP_LOSS_LIMIT&timeInForce=GTC&quantity=0.01&price=42000&stopPrice=43000&trailingDelta=500&timestamp=<timestamp>&signature=<signature>'
```

Placing a trailing stop `STOP_LOSS_LIMIT SELL` order, with a price of 37,500.0 and a trailing stop of 2.5%.
```bash
# Excluding stop price
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=SELL&type=STOP_LOSS_LIMIT&timeInForce=GTC&quantity=0.01&price=37500&trailingDelta=250&timestamp=<timestamp>&signature=<signature>'

# Including stop price of 39,000
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=SELL&type=STOP_LOSS_LIMIT&timeInForce=GTC&quantity=0.01&price=37500&stopPrice=39000&trailingDelta=250&timestamp=<timestamp>&signature=<signature>'
```

Placing a trailing stop `TAKE_PROFIT_LIMIT BUY` order, with a price of 38,000.0 and a trailing stop of 5%.

```bash
# Excluding stop price
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=BUY&type=TAKE_PROFIT_LIMIT&timeInForce=GTC&quantity=0.01&price=38000&trailingDelta=500&timestamp=<timestamp>&signature=<signature>'

# Including stop price of 36,000
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=BUY&type=TAKE_PROFIT_LIMIT&timeInForce=GTC&quantity=0.01&price=38000&stopPrice=36000&trailingDelta=500&timestamp=<timestamp>&signature=<signature>'
```

Placing a trailing stop `TAKE_PROFIT_LIMIT SELL` order, with a price of 41,500.0 and a trailing stop of 1.75%.
```bash
# Excluding stop price
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=SELL&type=TAKE_PROFIT_LIMIT&timeInForce=GTC&quantity=0.01&price=41500&trailingDelta=175&timestamp=<timestamp>&signature=<signature>'

# Including stop price of 42,500
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=SELL&type=TAKE_PROFIT_LIMIT&timeInForce=GTC&quantity=0.01&price=41500&stopPrice=42500&trailingDelta=175&timestamp=<timestamp>&signature=<signature>'
```