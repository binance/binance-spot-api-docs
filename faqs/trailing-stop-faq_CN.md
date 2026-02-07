# 追踪止盈止损(Trailing Stop)订单常见问题

## 什么是追踪止盈止损订单?

追踪止盈止损是一种用于追踪市场价格变动的工具. 现货的API中多了一个新参数 `trailingDelta`, 用来定义订单触发时机, 其值为基点(BIPS).

显然追踪止盈止损订单允许价格在有利的方向变动,同时限制在不利方向上的价格变动.

买单: _价格低_ 是有利的. 持续的价格 _下跌_ 是被允许的. 但是如果市场价格基于下单后的最低点 _上涨_ 了预定义的价格差, 订单会被触发.

卖单: _价格高_ 是有利的. 持续的价格 _上涨_ 是被允许的. 但是如果市场价格基于下单后的最高点 _下跌_ 了预定义的价格差, 订单会被触发.

## 什么是基点(BIPs)?

基点,又称为bps或bips,是金融学中用来描述百分比变化.

基点数 | 百分比 | 小数形式
---- | ---------- | ----------
1 | 0.01% | 0.0001
10 | 0.1% | 0.001
100 | 1% | 0.01
1000 | 10% | 0.1

比如, 一个设置`trailingDelta=100`的止损(STOP_LOSS)卖单,则会构成一个追踪止损卖单, 此订单在提交后会跟踪价格变动,当价格从提交后的最高价下跌`1%`的时候会被触发为市价单卖掉.

## 什么样的订单可以成追踪止盈止损单?

`STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, 和 `TAKE_PROFIT_LIMIT` 支持追踪止盈止损功能.

限价止盈止损单订单(OCO)在其止盈止损单中支持追踪止盈止损功能. 如果追踪止盈止损被触发,其中的限价单(LIMIT)会被取消.

## 如果下追踪止盈止损订单?

追踪止盈止损订单的使用和`STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, 或者 `TAKE_PROFIT_LIMIT` 订单很类似,但是多了一个`trailingDelta`参数. 此参数必须在交易对的`TRAILING_DELTA`过滤器定义范围内. 具体值可以参见接口 `GET /api/v3/exchangeInfo`.

不同于一般的止盈止损单,追踪止盈止损订单中的`stopPrice`是可选的. 如果`stopPrice`被设置了,那么订单只有当这个`stopPrice`价格被触发了后,才开始追踪价格变动.如果`stopPrice`被忽略,则会立刻开始追踪价格变化.

## 什么样的价格变动会触发追踪止盈止损订单?

追踪的订单类型 | 方向 | 止盈止损价(Stop price)条件     | 触发所需的价格变动
------------------- | ---- | -------------------------- | ----------------------------------
`TAKE_PROFIT`       | 卖出 | 市场价 >= 止盈价 | 从最高价*回调*
`TAKE_PROFIT_LIMIT` | 卖出 | 市场价 >= 止盈价 | 从最高价*回调*
`STOP_LOSS`         | 卖出 | 市场价 <= 止损价 | 从最高价*回调*
`STOP_LOSS_LIMIT`   | 卖出 | 市场价 <= 止损价 | 从最高价*回调*
`STOP_LOSS`         | 买入  | 市场价 >= 止损价 | 从最低价*上涨*
`STOP_LOSS_LIMIT`   | 买入  | 市场价 >= 止损价 | 从最低价*上涨*
`TAKE_PROFIT`       | 买入  | 市场价 <= 止盈价 | 从最低价*上涨*
`TAKE_PROFIT_LIMIT` | 买入  | 市场价 <= 止盈价 | 从最低价*上涨*

## 如果使用 `TRAILING_DELTA` 过滤器?

对于 `STOP_LOSS` 买单, `STOP_LOSS_LIMIT` 买单, `TAKE_PROFIT` 卖单, 和 `TAKE_PROFIT_LIMIT` 卖单:

* `trailingDelta` >= `minTrailingAboveDelta`
* `trailingDelta` <= `maxTrailingAboveDelta`

对于 `STOP_LOSS` 卖单, `STOP_LOSS_LIMIT` 卖单, `TAKE_PROFIT` 买单, 和 `TAKE_PROFIT_LIMIT` 买单:

* `trailingDelta` >= `minTrailingBelowDelta`
* `trailingDelta` <= `maxTrailingBelowDelta`

## 追踪止盈止损订单用例

### 用例 A - 追踪止损限价买单

在 `12:01:00` 的时候,市场最新价为40,000. 这时候有一个限价止损买单(`STOP_LOSS_LIMIT`)进入交易所, 并且设置了止损价(`stopPrice`)为44,000, `trailingDelta`为500 (5%), 以及限价(`LIMIT`) 45,000.

在 `12:01:00` 到 `12:02:00` 之间市场上一系列的交易让最新价下跌到37,000. 到这里价格下跌了7.5%(750 BIPS), 超过了设置价格差值(`trailingDelta`).  但是因为还没有开始追踪市场, 所以价格变动被忽略.

在 `12:02:00` 到 `12:03:00` 之间市场上一系列的交易让价格开始上涨. 当一个交易(trade)价格等于,或者超过了止损价(`stopPrice`)(44,000),这时候订单立刻开始追踪价格变动. 满足条件的第一个交易会被看作"最低价", 在这个例子里就是44,000. 如果价格从44,000上涨了500个基点, 订单就会被触发. 
这时候市场持续交易, 并将最新价推高到45,000.

在 `12:03:00` 到 `12:04:00` 之间市场的交易让最新价上涨到46,000. 这时价格已经从之前标记的最低价(44,000)上涨了大约454个基点, 但是还不到触发订单的要求(500个基点).

在 `12:04:00` 到 `12:05:00` 之间市场上一系列的交易让最新价开始下跌到42,000. 这是从之前的标记的最低价开始的下跌, 最低价被标记为当前价(42,000). 如果市场从这里上涨500个基点, 订单会被触发.

在 `12:05:00` 到 `12:05:30` 之间市场交易让最新价上涨到44,100. 最新的交易价格到达或者超过了订单设置的500个基点要求(`44,100 = 42,000 * 1.05`). 这导致了订单被触发, 然后被以限价45,000的价格放到订单薄(order book)里面.

<img alt="image" src="https://user-images.githubusercontent.com/17701918/167370103-ab3b4c05-1e13-4a25-b99a-42f9e4d6adc8.png" />

### 用例 B - 追踪止损限价卖单

在 `12:01:00` 的时候, 市场最新价为40,000. 这时候有一个限价止损卖单(`STOP_LOSS_LIMIT`)进入交易所, 并且设置了止损价(`stopPrice`)为39,000, `trailingDelta`为1,000 (10%), 以及限价(LIMIT) 38,000.

在 `12:01:00` 到 `12:02:00` 之间市场上一系列的交易让最新价上涨到41,500. 

在 `12:02:00` 到 `12:03:00` 之间市场的交易让价格开始下跌. 当一个交易(trade)价格等于, 或者低于了止损价(`stopPrice`)(39,000), 这时候订单立刻开始追踪价格变动. 满足条件的第一个交易会被看作"最高价", 在这个例子里就是39,000. 如果价格从39,000下跌了1,000个基点, 订单就会被触发. 

在 `12:03:00` 到 `12:04:00` 之间市场上一系列的交易让最新价开始下跌到37,000. 这时价格已经从之前标记的最高价(39,000)回落了大约512个基点, 但是还不到触发订单的要求(1000个基点).

在 `12:04:00` 到 `12:05:00` 之间市场上一系列的交易让最新价开始上涨到41,000. 这是从之前的标记的最高价开始的上涨, 最高价被标记为当前价(41,000). 如果这时候有1000个基点下跌, 订单会被触发.

在 `12:05:00` 到 `12:05:30` 之间的一些市场交易让最新价下跌到36,900. 最新的交易价格到达或者超过了订单设置的1000个基点要求(`36,900 = 41,000 * 0.90`). 这导致了订单被触发, 然后被以限价38,000的价格放到订单薄(order book)里面.

<img alt="image" src="https://user-images.githubusercontent.com/17701918/167370383-eb813cc1-d9b8-4a94-896c-a1a29551e09d.png" />

### 用例 C - 追踪止盈限价买单

在 `12:01:00` 的时候, 市场最新价为40,000. 这时候有一个限价止盈买单(`TAKE_PROFIT_LIMIT`)进入交易所, 并且设置了止盈价(`stopPrice`)为38,000, `trailingDelta`为 850 (8.5%), 以及限价(LIMIT) 38,500.

在 `12:01:00` 到 `12:02:00` 之间市场上一系列的交易让最新价上涨到42,000.

在 `12:02:00` 到 `12:03:00` 之间市场交易让价格开始下跌. 当一个交易(trade)价格等于, 或者超过了止盈价(`stopPrice`)(38,000), 这时候订单立刻开始追踪价格变动. 满足条件的第一个交易会被看作"最低价", 在这个例子里就是38,000. 
市场的持续交易,让最新价下跌到37,000. 这时最低价被设为37,000. 如果价格从37,000上涨了850个基点, 订单就会被触发.

在 `12:03:00` 到 `12:04:00` 之间市场上一系列的交易让最新价开始上涨到39,000. 这时价格已经从之前标记的最低价(37,000)上涨了大约540个基点, 但是还不到触发订单的要求(850个基点).

在 `12:04:00` 到 `12:05:00` 之间市场上一系列的交易让最新价开始下跌到38,000, 没有低于之前的最低价, 所以不会有任何变化.

在 `12:05:00` 到 `12:05:30` 之间的一些市场交易让最新价上涨到40,145. 最新的交易价格到达或者超过了订单设置的850个基点要求(`40,145 = 37,000 * 1.085`). 这导致了订单被触发, 然后被以限价38,500的价格放到订单薄(order book)里面.

<img alt="image" src="https://user-images.githubusercontent.com/17701918/167370339-f1b83c76-790b-4108-8c9a-db2d89a4850f.png" />

### 用例 D - 追踪止盈限价卖单

在 `12:01:00` 的时候,市场最新价为40,000. 这时候有一个限价止盈卖单(`TAKE_PROFIT_LIMIT`)进入交易所, 并且设置了止盈价(`stopPrice`)为42,000, `trailingDelta`为 750 (7.5%), 以及限价(LIMIT) 41,000.

在 `12:01:00` 到 `12:02:00` 之间市场上一系列的交易让最新价上涨到41,500.

在 `12:02:00` 到 `12:03:00` 之间市场上一系列的交易让价格开始下跌到39,000.

在 `12:03:00` 到 `12:04:00` 之间市场上一系列的交易让最新价开始上涨. 
当最新的交易价格到达或者超过了订单设置的止盈价(`stopPrice`),订单就会立刻开始追踪. 第一个满足条件的交易价就被设为"最高价". 在这个例子中, 最高价是42,000. 如果从最高价下跌750个基点, 订单就会被触发.

这时市场价格持续上涨,最新价到了45,000. 追踪止盈订单的"最高价"也被设置成了45,000. 如果价格从这里下跌750个基点, 订单会被触发.

在 `12:04:00` 到 `12:05:00` 之间市场上一系列的交易让最新价开始下跌到44,000. 这相当于从之前的最高价(45,000)下跌了大约222个基点, 不过没有达到追踪止盈订单设置的750个基点要求, 所以不会触发.

在 `12:05:00` 到 `12:06:00` 之间的一些市场交易让最新价上涨到46,500. 因为市场上涨,之前的最高价被设置成当前价(46,500). 如果价格从这里下跌750个基点, 订单会被触发.

在 `12:06:00` 到 `12:06:50` 之间的一些市场交易让最新价下跌到43,012.5. 当前价格达到或者超过了追踪订单设定的750个基点(`43,012.5 = 46,500 * 0.925`), 导致订单被触发, 以限价(LIMIT)41,000的价格被放入订单薄.

<img alt="image" src="https://user-images.githubusercontent.com/17701918/167370298-172b227a-198d-46ee-a385-5cc267dc253b.png" />

### 用例 E - 不设置限价的追踪止盈止损订单

在 `12:01:00` 的时候, 市场最新价为40,000. 这时候有一个限价止损卖单(`STOP_LOSS_LIMIT`)进入交易所, 并且设置了`trailingDelta`为 700 (7%), 以及限价(LIMIT) 39,000, 但是没有设置限价(`stopPrice`). 订单创建后就开始追踪价格. 如果价格从40,000下跌了700个基点, 订单就会被触发.

在 `12:01:00` 到 `12:02:00` 之间市场上一系列的交易让价格开始上涨到42,000, 使追踪的"最高价"变成42,000. 如果价格从这里(42,000)开始下跌700个基点, 订单会被触发.

在 `12:02:00` 到 `12:03:00` 之间市场上一系列的交易让价格下跌到39,500, 这让市场从之前追踪的最高价（42,000)下跌了大约595个基点, 不过没有达到700个基点的要求, 订单不会被触发.

在 `12:03:00` 到 `12:04:00` 之间市场上一系列的交易让价格上涨到45,500. 市场的上涨, 让追踪的最高价变成当前价45,500. 如果价格从这里45,500下跌了700个基点, 订单就会被触发.

在 `12:04:00` 到 `12:04:45` 之间市场上一系列的交易让价格下跌到42,315. 当前价格达到或者超过了追踪订单设定的700个基点(`42,315 = 45,500 * 0.93`), 导致订单被触发, 以限价(LIMIT)39,000的价格被放入订单薄.

<img alt="image" src="https://user-images.githubusercontent.com/17701918/167370616-17d3295a-3e7c-4314-aa13-ad44e685a311.png" />

## 追踪止盈止损下单示例

假设当前最新价是40,000.

下一个追踪止损(`STOP_LOSS_LIMIT`)的买单, 设置价格为42,000, 追踪价差（`trailingDelta`)为5%.

```bash
# 不设置止损价(stop price)
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=BUY&type=STOP_LOSS_LIMIT&timeInForce=GTC&quantity=0.01&price=42000&trailingDelta=500&timestamp=<timestamp>&signature=<signature>'

# 设置止损价(stop price)43,000
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=BUY&type=STOP_LOSS_LIMIT&timeInForce=GTC&quantity=0.01&price=42000&stopPrice=43000&trailingDelta=500&timestamp=<timestamp>&signature=<signature>'
```

下一个追踪止损(`STOP_LOSS_LIMIT`)的卖单, 设置价格为37,500, 追踪价差（`trailingDelta`)为2.5%.

```bash
# 不设置止损价(stop price)
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=SELL&type=STOP_LOSS_LIMIT&timeInForce=GTC&quantity=0.01&price=37500&trailingDelta=250&timestamp=<timestamp>&signature=<signature>'

# 设置止损价(stop price)39,000
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=SELL&type=STOP_LOSS_LIMIT&timeInForce=GTC&quantity=0.01&price=37500&stopPrice=39000&trailingDelta=250&timestamp=<timestamp>&signature=<signature>'
```

下一个追踪止盈(`TAKE_PROFIT_LIMIT`)的买单, 设置价格为38,000, 追踪价差（`trailingDelta`)为5%.

```bash
# 不设置止盈价(stop price)
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=BUY&type=TAKE_PROFIT_LIMIT&timeInForce=GTC&quantity=0.01&price=38000&trailingDelta=500&timestamp=<timestamp>&signature=<signature>'

# 设置止盈价(stop price)36,000
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=BUY&type=TAKE_PROFIT_LIMIT&timeInForce=GTC&quantity=0.01&price=38000&stopPrice=36000&trailingDelta=500&timestamp=<timestamp>&signature=<signature>'
```

下一个追踪止盈(`TAKE_PROFIT_LIMIT`)的卖单, 设置价格为41,500, 追踪价差（`trailingDelta`)为1.75%.

```bash
# 不设置止盈价(stop price)
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=SELL&type=TAKE_PROFIT_LIMIT&timeInForce=GTC&quantity=0.01&price=41500&trailingDelta=175&timestamp=<timestamp>&signature=<signature>'

# 设置止盈价(stop price) 42,500
POST 'https://api.binance.com/api/v3/order?symbol=BTCUSDT&side=SELL&type=TAKE_PROFIT_LIMIT&timeInForce=GTC&quantity=0.01&price=41500&stopPrice=42500&trailingDelta=175&timestamp=<timestamp>&signature=<signature>'
```
