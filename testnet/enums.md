# ENUM Definitions

This will apply for both Rest API and WebSocket API.

## ENUM definitions

#### Symbol status (status):

* `PRE_TRADING`
* `TRADING`
* `POST_TRADING`
* `END_OF_DAY`
* `HALT`
* `AUCTION_MATCH`
* `BREAK`

<a id="account-and-symbol-permissions"></a>

#### Account and Symbol Permissions (permissions):

* `SPOT`

#### Order status (status):

Status | Description
-----------| --------------
`NEW` | The order has been accepted by the engine.
`PENDING_NEW`|The order is in a pending phase until the working order of an order list has been fully filled.
`PARTIALLY_FILLED`| A part of the order has been filled.
`FILLED` | The order has been completed.
`CANCELED` | The order has been canceled by the user.
`PENDING_CANCEL` | Currently unused
`REJECTED`       | The order was not accepted by the engine and not processed.
`EXPIRED` | The order was canceled according to the order type's rules (e.g. LIMIT FOK orders with no fill, LIMIT IOC or MARKET orders that partially fill) <br/> or by the exchange, (e.g. orders canceled during liquidation, orders canceled during maintenance)
`EXPIRED_IN_MATCH` | The order was expired by the exchange due to STP. (e.g. an order with `EXPIRE_TAKER` will match with existing orders on the book with the same account or same `tradeGroupId`)

#### Order List Status (listStatusType):

Status | Description
-----------| --------------
`RESPONSE` | This is used when the ListStatus is responding to a failed action. (E.g. order list placement or cancellation)
`EXEC_STARTED`| The order list has been placed or there is an update to the order list status.
`ALL_DONE` | The order list has finished executing and thus is no longer active.

#### Order List Order Status (listOrderStatus):

Status | Description
-----------| --------------
`EXECUTING` | Either an order list has been placed or there is an update to the status of the list.
`ALL_DONE`| An order list has completed execution and thus no longer active.
`REJECT` | The List Status is responding to a failed action either during order placement or order canceled.

#### ContingencyType:

* `OCO`
* `OTO`

<a id="allocationtype"></a>

#### AllocationType:

* `SOR`

<a id="ordertypes"></a>

#### Order types (orderTypes, type):

* `LIMIT`
* `MARKET`
* `STOP_LOSS`
* `STOP_LOSS_LIMIT`
* `TAKE_PROFIT`
* `TAKE_PROFIT_LIMIT`
* `LIMIT_MAKER`

<a id="orderresponsetype"></a>

#### Order Response Type (newOrderRespType):

* `ACK`
* `RESULT`
* `FULL`

#### Working Floor:

* `EXCHANGE`
* `SOR`

<a id="side"></a>

#### Order side (side):

* `BUY`
* `SELL`

<a id="timeinforce"></a>

#### Time in force (timeInForce):

This sets how long an order will be active before expiration.

Status | Description
-----------| --------------
`GTC` | Good Til Canceled <br/> An order will be on the book unless the order is canceled.
`IOC` | Immediate Or Cancel <br/> An order will try to fill the order as much as it can before the order expires.
`FOK`| Fill or Kill <br/> An order will expire if the full order cannot be filled upon execution.


#### Rate limiters (rateLimitType):

* REQUEST_WEIGHT

```json
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalNum": 1,
      "limit": 6000
    }
```

* ORDERS

```json
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalNum": 1,
      "limit": 10
    }
```
* RAW_REQUESTS

```json
    {
      "rateLimitType": "RAW_REQUESTS",
      "interval": "MINUTE",
      "intervalNum": 5,
      "limit": 61000
    }
```

#### Rate limit intervals (interval):

* SECOND
* MINUTE
* DAY

<a id="stpmodes"></a>

#### STP Modes:

* `NONE`
* `EXPIRE_MAKER`
* `EXPIRE_TAKER`
* `EXPIRE_BOTH`
