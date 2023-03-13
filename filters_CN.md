# 过滤器
过滤器，即Filter，定义了一系列交易规则。
共有两类，分别是针对交易对的过滤器`symbol filters`，和针对整个交易所的过滤器`exchange filters`

## 交易对过滤器
### PRICE_FILTER 价格过滤器
价格过滤器用于检测order订单中price参数的合法性
* `minPrice` 定义了 `price`/`stopPrice` 允许的最小值; `minPrice` == 0 的时候则失效。
* `maxPrice` 定义了 `price`/`stopPrice` 允许的最大值; `maxPrice` == 0 的时候则失效。
* `tickSize` 定义了 `price`/`stopPrice` 的步进间隔; `tickSize` == 0 的时候则失效。

以上每一项均可为0，为0时代表这一项不再做限制。

逻辑伪代码如下：
* `price` >= `minPrice`
* `price` <= `maxPrice`
* `price` % `tickSize` == 0

**/exchangeInfo 响应中的格式:**
```javascript
  {
    "filterType": "PRICE_FILTER",
    "minPrice": "0.00000100",
    "maxPrice": "100000.00000000",
    "tickSize": "0.00000100"
  }
```

### PERCENT_PRICE 价格振幅过滤器
可以理解为一个瞬时的涨跌停限制，不允许价格瞬间剧烈浮动。
`avgPriceMins` 指用过去几分钟的平均价格来计算价格基准. 0 表示用最新成交价格作为价格计准。

逻辑伪代码如下：
* `price` <= `weightedAveragePrice` * `multiplierUp`
* `price` >= `weightedAveragePrice` * `multiplierDown`

**/exchangeInfo 响应中的格式:**
```javascript
  {
    "filterType": "PERCENT_PRICE",
    "multiplierUp": "1.3000",
    "multiplierDown": "0.7000",
    "avgPriceMins": 5
  }
```

#### PERCENT_PRICE_BY_SIDE
`PERCENT_PRICE_BY_SIDE` 过滤器定义了基于交易对平均价格的合法价格范围. 取决于`BUY`或者`SELL`, 价格范围可能有所不同.<br/>

`avgPriceMins` 是用来计算平均价格的分钟数. 0 表示用最新价(last price).<br/>

买向订单需要满足:

* `Order price` <= `weightedAveragePrice` * `bidMultiplierUp`
* `Order price` >= `weightedAveragePrice` * `bidMultiplierDown`

卖向订单需要满足:

* `Order Price` <= `weightedAveragePrice` * `askMultiplierUp`
* `Order Price` >= `weightedAveragePrice` * `askMultiplierDown`


**/exchangeInfo 响应中的格式:**
```javascript
  {
    "filterType": "PERCENT_PRICE_BY_SIDE",
    "bidMultiplierUp": "1.2",
    "bidMultiplierDown": "0.2",
    "askMultiplierUp": "5",
    "askMultiplierDown": "0.8",
    "avgPriceMins": 1
  }
```

### LOT_SIZE 订单尺寸
"lots" 是拍卖术语，这个过滤器对订单中的 `quantity` 也就是数量参数进行合法性检查。包含三个部分：

* `minQty` 表示 `quantity`/`icebergQty` 允许的最小值.
* `maxQty` 表示 `quantity`/`icebergQty` 允许的最大值
* `stepSize` 表示 `quantity`/`icebergQty` 允许的步进值。

逻辑伪代码如下：

* `quantity` >= `minQty`
* `quantity` <= `maxQty`
* `quantity` % `stepSize` == 0

**/exchangeInfo 响应中的格式:**
```javascript
  {
    "filterType": "LOT_SIZE",
    "minQty": "0.00100000",
    "maxQty": "100000.00000000",
    "stepSize": "0.00100000"
  }
```

### MIN_NOTIONAL 最小金额
MIN_NOTIONAL过滤器定义了交易对订单所允许的最小名义价值(成交额)。
订单的名义价值是`价格`*`数量`。
如果是高级订单(比如止盈止损订单`STOP_LOSS_LIMIT`)，名义价值会按照`stopPrice` * `quantity`来计算。
如果是冰山订单，名义价值会按照`price` * `icebergQty`来计算。
`applyToMarket`确定 `MIN_NOTIONAL`过滤器是否也将应用于`MARKET`订单。   
由于`MARKET`订单没有价格，因此会在最后`avgPriceMins`分钟内使用平均价格。   
`avgPriceMins`是计算平均价格的分钟数。 0表示使用最后的价格。 


**/exchangeInfo 响应中的格式:**
```javascript
  {
    "filterType": "MIN_NOTIONAL",
    "minNotional": "0.00100000",
    "applyToMarket": true,
    "avgPriceMins": 5
  }
```

### NOTIONAL 名义价值

**/exchangeInfo 响应中的格式:**

```javascript
{
   "filterType": "NOTIONAL",
   "minNotional": "10.00000000",
   "applyMinToMarket": false,
   "maxNotional": "10000.00000000",
   "applyMaxToMarket": false,
   "avgPriceMins": 5
}
```

名义价值过滤器(`NOTIONAL`)定义了订单在一个交易对上可以下单的名义价值区间.<br/><br/>
`applyMinToMarket` 定义了 `minNotional` 是否适用于市价单(`MARKET`)  <br/>
`applyMaxToMarket` 定义了 `maxNotional` 是否适用于市价单(`MARKET`).

要通过此过滤器, 订单的名义价值 (单价 x 数量, `price * quantity`) 需要满足如下条件:

* `price * quantity` <= `maxNotional`
* `price * quantity` >= `minNotional`

对于市价单(`MARKET`), 用于计算的价格采用的是在 `avgPriceMins` 定义的时间之内的平均价.<br/>
如果 `avgPriceMins` 为 0, 则采用最新的价格.

### ICEBERG_PARTS 冰山订单拆分数
`ICEBERG_PARTS` 代表冰山订单最多可以拆分成多少个小订单。
计算方法为 `向上取整(qty / icebergQty)`.

**/exchangeInfo 响应中的格式:**
```javascript
  {
    "filterType": "ICEBERG_PARTS",
    "limit": 10
  }
```

### MARKET_LOT_SIZE 市价订单尺寸
`MARKET_LOT_SIZE`过滤器为交易对上的`MARKET`订单定义了`数量`(即拍卖中的"手数")规则。 共有3部分：

* `minQty`定义了允许的最小`quantity`。
* `maxQty`定义了允许的最大数量。
* `stepSize`定义了可以增加/减少数量的间隔。

为了通过 `market lot size`，`quantity` 必须满足以下条件：

* `quantity` >= `minQty`
* `quantity` <= `maxQty`
* `quantity` % `stepSize` == 0

**/exchangeInfo 响应中的格式:**
```javascript
  {
    "filterType": "MARKET_LOT_SIZE",
    "minQty": "0.00100000",
    "maxQty": "100000.00000000",
    "stepSize": "0.00100000"
  }
```

### MAX_NUM_ORDERS 最多订单数
定义了某个交易对最多允许的挂单数量（不包括已关闭的订单）
普通订单与条件订单均计算在内

**/exchangeInfo 响应中的格式:**
```javascript
  {
    "filterType": "MAX_NUM_ORDERS",
    "maxNumOrders": 25
  }
```

### MAX_NUM_ALGO_ORDERS 最多条件单数
`MAX_NUM_ALGO_ORDERS`过滤器定义允许账户在交易对上开设的"algo"订单的最大数量。    
"Algo"订单是`STOP_LOSS`，`STOP_LOSS_LIMIT`，`TAKE_PROFIT`和`TAKE_PROFIT_LIMIT`止盈止损单。

**/exchangeInfo 响应中的格式:**
```javascript
  {
    "filterType": "MAX_NUM_ALGO_ORDERS",
    "maxNumAlgoOrders": 5
  }
```

### MAX_NUM_ICEBERG_ORDERS 最多冰山单数
`MAX_NUM_ICEBERG_ORDERS`过滤器定义了允许在交易对上开设账户的`ICEBERG`订单的最大数量。     
`ICEBERG`订单是icebergQty大于0的任何订单。

**/exchangeInfo 响应中的格式:**
```javascript
  {
    "filterType": "MAX_NUM_ICEBERG_ORDERS",
    "maxNumIcebergOrders": 5
  }
```

### MAX_POSITION 过滤器

这个过滤器定义账户允许的基于`base asset`的最大仓位。一个用户的仓位可以定义为如下资产的总和:
1. `base asset`的可用余额
1. `base asset`的锁定余额
1. 所有处于open的买单的数量总和

如果用户的仓位大于最大的允许仓位，买单会被拒绝。

如果一个订单的数量(`quantity`) 可能导致持有仓位溢出, 会触发过滤器 `MAX_POSITION`.

**/exchangeInfo 响应中的格式:**
```javascript
{
  "filterType": "MAX_POSITION",
  "maxPosition": "10.00000000"
}
```

### TRAILING_DELTA 过滤器


此过滤器定义了参数`trailingDelta`的最大和最小值.

下追踪止损订单, 需要满足条件:

对于 `STOP_LOSS BUY`, `STOP_LOSS_LIMIT_BUY`, `TAKE_PROFIT SELL` 和 `TAKE_PROFIT_LIMIT SELL` 订单:

* `trailingDelta` >= `minTrailingAboveDelta`
* `trailingDelta` <= `maxTrailingAboveDelta`

对于 `STOP_LOSS SELL`, `STOP_LOSS_LIMIT SELL`, `TAKE_PROFIT BUY`, 和 `TAKE_PROFIT_LIMIT BUY` 订单:

* `trailingDelta` >= `minTrailingBelowDelta`
* `trailingDelta` <= `maxTrailingBelowDelta`

 **/exchangeInfo format:**
```javascript
    {
          "filterType": "TRAILING_DELTA",
          "minTrailingAboveDelta": 10,
          "maxTrailingAboveDelta": 2000,
          "minTrailingBelowDelta": 10,
          "maxTrailingBelowDelta": 2000
   }
```

## 交易所级别过滤器
### EXCHANGE_MAX_NUM_ORDERS 最多订单数
`EXCHANGE_MAX_NUM_ORDERS`过滤器定义了允许在交易对上开设账户的最大订单数。    
请注意，此过滤器同时计算"algo"订单和常规订单。

**/exchangeInfo 响应中的格式:**
```javascript
  {
    "filterType": "EXCHANGE_MAX_NUM_ORDERS",
    "maxNumOrders": 1000
  }
```

### EXCHANGE_MAX_NUM_ALGO_ORDERS 最多条件单数
`EXCHANGE_MAX_ALGO_ORDERS`过滤器定义了允许在交易上开设账户的"algo"订单的最大数量。    
"Algo"订单是`STOP_LOSS`，`STOP_LOSS_LIMIT`，`TAKE_PROFIT`和`TAKE_PROFIT_LIMIT`订单。

**/exchangeInfo 响应中的格式:**
```javascript
  {
    "filterType": "EXCHANGE_MAX_NUM_ALGO_ORDERS",
    "maxNumAlgoOrders": 200
  }
```

### EXCHANGE_MAX_NUM_ICEBERG_ORDERS 冰山订单的最大订单数

此过滤器定义了允许账号持有的最大冰山订单数量.


**/exchangeInfo 响应中的格式:**

```javascript
{
  "filterType": "EXCHANGE_MAX_NUM_ICEBERG_ORDERS",
  "maxNumIcebergOrders": 10000
}
```