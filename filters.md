# Filters
Filters define trading rules on a symbol or an exchange.
Filters come in three forms: `symbol filters`, `exchange filters` and `asset filters`.

## Symbol filters
### PRICE_FILTER
The `PRICE_FILTER` defines the `price` rules for a symbol. There are 3 parts:

* `minPrice` defines the minimum `price`/`stopPrice` allowed; disabled on `minPrice` == 0.
* `maxPrice` defines the maximum `price`/`stopPrice` allowed; disabled on `maxPrice` == 0.
* `tickSize` defines the intervals that a `price`/`stopPrice` can be increased/decreased by; disabled on `tickSize` == 0.

Any of the above variables can be set to 0, which disables that rule in the `price filter`. In order to pass the `price filter`, the following must be true for `price`/`stopPrice` of the enabled rules:

* `price` >= `minPrice`
* `price` <= `maxPrice`
* `price` % `tickSize` == 0

**/exchangeInfo format:**
```javascript
{
    "filterType": "PRICE_FILTER",
    "minPrice": "0.00000100",
    "maxPrice": "100000.00000000",
    "tickSize": "0.00000100"
}
```

### PERCENT_PRICE
The `PERCENT_PRICE` filter defines the valid range for an order `price` based on an `average of previous trade prices`.

* When a non-null [reference price](./faqs/price_range_execution_rules.md) for the symbol exists, it is used in the filter evaluation.
* When a non-null reference price for the symbol does not exist, then the volume weighted average price over the preceding `avgPriceMins` minutes is used in the filter evaluation.
  * If `avgPriceMins` is 0, then the last price is used in the filter evaluation.

An order will pass this filter evaluation if:
* `price` <= `average of previous trade prices` * `multiplierUp`
* `price` >= `average of previous trade prices` * `multiplierDown`

**/exchangeInfo format:**
```javascript
{
    "filterType": "PERCENT_PRICE",
    "multiplierUp": "1.3000",
    "multiplierDown": "0.7000",
    "avgPriceMins": 5
}
```

### PERCENT_PRICE_BY_SIDE
The `PERCENT_PRICE_BY_SIDE` filter defines the valid range for an order `price` based on an `average of previous trade prices`.

* When a non-null [reference price](./faqs/price_range_execution_rules.md) for the symbol exists, it is used in the filter evaluation.
* When a non-null reference price for the symbol does not exist, then the volume weighted average price over the preceding `avgPriceMins` minutes is used in the filter evaluation.
  * If `avgPriceMins` is 0, then the last price is used in the filter evaluation.

There is a different range depending on whether an order is placed on the `BUY` side or the `SELL` side.

A `BUY` order will pass this filter evaluation if:
* `price` <= `average of previous trade prices` * `bidMultiplierUp`
* `price` >= `average of previous trade prices` * `bidMultiplierDown`

A `SELL` order will pass this filter evaluation if:
* `price` <= `average of previous trade prices` * `askMultiplierUp`
* `price` >= `average of previous trade prices` * `askMultiplierDown`

**/exchangeInfo format:**
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


### LOT_SIZE
The `LOT_SIZE` filter defines the `quantity` (aka "lots" in auction terms) rules for a symbol. There are 3 parts:

* `minQty` defines the minimum `quantity`/`icebergQty` allowed.
* `maxQty` defines the maximum `quantity`/`icebergQty` allowed.
* `stepSize` defines the intervals that a `quantity`/`icebergQty` can be increased/decreased by.

In order to pass the `lot size`, the following must be true for `quantity`/`icebergQty`:

* `quantity` >= `minQty`
* `quantity` <= `maxQty`
* `quantity` % `stepSize` == 0

**/exchangeInfo format:**
```javascript
{
    "filterType": "LOT_SIZE",
    "minQty": "0.00100000",
    "maxQty": "100000.00000000",
    "stepSize": "0.00100000"
}
```

### MIN_NOTIONAL
The `MIN_NOTIONAL` filter defines the minimum notional value allowed for an order on a symbol.

* An order's notional value is the `price` * `quantity`.
* `applyToMarket` determines whether or not the `MIN_NOTIONAL` filter will also be applied to `MARKET` orders.
  * Since `MARKET` orders have no `price`, an `average of previous trade prices` is used instead.
    * When a non-null [reference price](./faqs/price_range_execution_rules.md) for the symbol exists, it is used as `price`.
    * When a non-null reference price for the symbol does not exist, then the volume weighted average price over the preceding `avgPriceMins` minutes is used as `price`.
      * If `avgPriceMins` is 0, then the last price is used as `price`.

An order will pass this filter evaluation if:
* `price` * `quantity` >= `minNotional`

**/exchangeInfo format:**
```javascript
{
    "filterType": "MIN_NOTIONAL",
    "minNotional": "0.00100000",
    "applyToMarket": true,
    "avgPriceMins": 5
}
```

### NOTIONAL
The `NOTIONAL` filter defines the acceptable notional range allowed for an order on a symbol.

* `applyMinToMarket` determines whether `minNotional` will be applied to `MARKET` orders.
* `applyMaxToMarket` determines whether `maxNotional` will be applied to `MARKET` orders.
  * Since `MARKET` orders have no `price`, an `average of previous trade prices` is used instead.
    * When a non-null [reference price](./faqs/price_range_execution_rules.md) for the symbol exists, it is used as `price`.
    * When a non-null reference price for the symbol does not exist, then the volume weighted average price over the preceding `avgPriceMins` minutes is used as `price`.
      * If `avgPriceMins` is 0, then the last price is used as `price`.

An order will pass this filter evaluation if:
* `price` * `quantity` <= `maxNotional`
* `price` * `quantity` >= `minNotional`

**/exchangeInfo format:**
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

### ICEBERG_PARTS
The `ICEBERG_PARTS` filter defines the maximum parts an iceberg order can have. The number of `ICEBERG_PARTS` is defined as `CEIL(qty / icebergQty)`.

**/exchangeInfo format:**
```javascript
{
    "filterType": "ICEBERG_PARTS",
    "limit": 10
}
```

### MARKET_LOT_SIZE
The `MARKET_LOT_SIZE` filter defines the `quantity` (aka "lots" in auction terms) rules for `MARKET` orders on a symbol. There are 3 parts:

* `minQty` defines the minimum `quantity` allowed.
* `maxQty` defines the maximum `quantity` allowed.
* `stepSize` defines the intervals that a `quantity` can be increased/decreased by.

In order to pass the `market lot size`, the following must be true for `quantity`:

* `quantity` >= `minQty`
* `quantity` <= `maxQty`
* `quantity` % `stepSize` == 0

**/exchangeInfo format:**
```javascript
{
    "filterType": "MARKET_LOT_SIZE",
    "minQty": "0.00100000",
    "maxQty": "100000.00000000",
    "stepSize": "0.00100000"
}
```

### MAX_NUM_ORDERS
The `MAX_NUM_ORDERS` filter defines the maximum number of orders an account is allowed to have open on a symbol.
Note that both "algo" orders and normal orders are counted for this filter.

**/exchangeInfo format:**
```javascript
{
    "filterType": "MAX_NUM_ORDERS",
    "maxNumOrders": 25
}
```

### MAX_NUM_ALGO_ORDERS
The `MAX_NUM_ALGO_ORDERS` filter defines the maximum number of "algo" orders an account is allowed to have open on a symbol.
"Algo" orders are `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders.

**/exchangeInfo format:**
```javascript
{
    "filterType": "MAX_NUM_ALGO_ORDERS",
    "maxNumAlgoOrders": 5
}
```

### MAX_NUM_ICEBERG_ORDERS
The `MAX_NUM_ICEBERG_ORDERS` filter defines the maximum number of `ICEBERG` orders an account is allowed to have open on a symbol.
An `ICEBERG` order is any order where the `icebergQty` is > 0.

**/exchangeInfo format:**
```javascript
{
    "filterType": "MAX_NUM_ICEBERG_ORDERS",
    "maxNumIcebergOrders": 5
}
```

### MAX_POSITION

The `MAX_POSITION` filter defines the allowed maximum position an account can have on the base asset of a symbol. An account's position defined as the sum of the account's:
1. free balance of the base asset
1. locked balance of the base asset
1. sum of the qty of all open BUY orders

`BUY` orders will be rejected if the account's position is greater than the maximum position allowed.

If an order's `quantity` can cause the position to overflow, this will also fail the `MAX_POSITION` filter.

**/exchangeInfo format:**
```javascript
{
    "filterType": "MAX_POSITION",
    "maxPosition": "10.00000000"
}
```

### TRAILING_DELTA

The `TRAILING_DELTA` filter defines the minimum and maximum value for the parameter [`trailingDelta`](faqs/trailing-stop-faq.md).

In order for a trailing stop order to pass this filter, the following must be true:

For `STOP_LOSS BUY`, `STOP_LOSS_LIMIT_BUY`,`TAKE_PROFIT SELL` and `TAKE_PROFIT_LIMIT SELL` orders:

* `trailingDelta` >= `minTrailingAboveDelta`
* `trailingDelta` <= `maxTrailingAboveDelta`

For `STOP_LOSS SELL`, `STOP_LOSS_LIMIT SELL`, `TAKE_PROFIT BUY`, and `TAKE_PROFIT_LIMIT BUY` orders:

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

### MAX_NUM_ORDER_AMENDS

The `MAX_NUM_ORDER_AMENDS` filter defines the maximum number of times an order can be amended on the given symbol.

If there are too many order amendments made on a single order, you will receive the `-2038` error code.

**/exchangeInfo format:**

```javascript
{
    "filterType": "MAX_NUM_ORDER_AMENDS",
    "maxNumOrderAmends": 10
}
```

### MAX_NUM_ORDER_LISTS

The `MAX_NUM_ORDER_LISTS` filter defines the maximum number of open order lists an account can have on a symbol. Note that OTOCOs count as one order list.

**/exchangeInfo format:**

```javascript
{
    "filterType": "MAX_NUM_ORDER_LISTS",
    "maxNumOrderLists": 20
}
```


## Exchange Filters
### EXCHANGE_MAX_NUM_ORDERS
The `EXCHANGE_MAX_NUM_ORDERS` filter defines the maximum number of orders an account is allowed to have open on the exchange.
Note that both "algo" orders and normal orders are counted for this filter.

**/exchangeInfo format:**
```javascript
{
    "filterType": "EXCHANGE_MAX_NUM_ORDERS",
    "maxNumOrders": 1000
}
```

### EXCHANGE_MAX_NUM_ALGO_ORDERS
The `EXCHANGE_MAX_NUM_ALGO_ORDERS` filter defines the maximum number of "algo" orders an account is allowed to have open on the exchange.
"Algo" orders are `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders.

**/exchangeInfo format:**
```javascript
{
    "filterType": "EXCHANGE_MAX_NUM_ALGO_ORDERS",
    "maxNumAlgoOrders": 200
}
```

### EXCHANGE_MAX_NUM_ICEBERG_ORDERS
The `EXCHANGE_MAX_NUM_ICEBERG_ORDERS` filter defines the maximum number of iceberg orders an account is allowed to have open on the exchange.

**/exchangeInfo format:**
```javascript
{
    "filterType": "EXCHANGE_MAX_NUM_ICEBERG_ORDERS",
    "maxNumIcebergOrders": 10000
}
```

### EXCHANGE_MAX_NUM_ORDER_LISTS

The `EXCHANGE_MAX_NUM_ORDER_LISTS` filter defines the maximum number of order lists an account is allowed to have open on the exchange. Note that OTOCOs count as one order list.

**/exchangeInfo format:**

```javascript
{
    "filterType": "EXCHANGE_MAX_NUM_ORDER_LISTS",
    "maxNumOrderLists": 20
}
```


## Asset Filters
### MAX_ASSET

The `MAX_ASSET` filter defines the maximum quantity of an asset that an account is allowed to transact in a single order.

* When the asset is a symbol's base asset, the limit applies to the order's quantity.
* When the asset is a symbol's quote asset, the limit applies to the order's notional value.
* For example, a MAX_ASSET filter for USDC applies to all symbols that have USDC as either a base or quote asset, such as:
  * USDCBNB
  * BNBUSDC

**/myFilters format:**

```javascript
{
    "filterType": "MAX_ASSET",
    "asset": "USDC",
    "limit": "42.00000000"
}
```
