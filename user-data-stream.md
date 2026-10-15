<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [User Data Streams for Binance](#user-data-streams-for-binance)
  - [General information](#general-information)
  - [User Data Stream Events](#user-data-stream-events)
    - [Account Update](#account-update)
    - [Balance Update](#balance-update)
    - [Order Update](#order-update)
      - [Conditional Fields in Execution Report](#conditional-fields-in-execution-report)
      - [Order Reject Reason](#order-reject-reason)
      - [Order Expiry Reason](#order-expiry-reason)
  - [Event Stream Terminated](#event-stream-terminated)
  - [External Lock Update](#external-lock-update)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# User Data Streams for Binance

## General information

- Subscribe via the
  [WebSocket API](web-socket-api.md#subscribe-to-user-data-stream-user_stream)
  using an API Key.
- Both [SBE](faqs/sbe_faq.md) and JSON output are supported.
- Account events are pushed in **real-time**.
- All timestamps in JSON payloads are in **milliseconds by default**.
- Events may contain non-ASCII characters encoded in UTF-8 if you own or trade any assets or symbols
  whose names contain non-ASCII characters.

## User Data Stream Events

### Account Update

`outboundAccountPosition` is sent any time an account balance has changed and contains the assets
that were possibly changed by the event that generated the balance change.

```javascript
{
    "subscriptionId": 0,
    "event": {
        "e": "outboundAccountPosition", // Event type
        "E": 1564034571105,             // Event Time
        "u": 1564034571073,             // Time of last account update
        // Balances Array
        "B": [
            {
                "a": "ETH",          // Asset
                "f": "10000.000000", // Free
                "l": "0.000000"      // Locked
            }
        ]
    }
}
```

### Balance Update

Balance Update occurs during the following:

- Deposits or withdrawals from the account
- Transfer of funds between accounts (e.g. Spot to Margin)

**Payload**

```javascript
{
    "subscriptionId": 0,
    "event": {
        "e": "balanceUpdate", // Event Type
        "E": 1573200697110,   // Event Time
        "a": "BTC",           // Asset
        "d": "100.00000000",  // Balance Delta
        "T": 1573200697068    // Clear Time
    }
}
```

### Order Update

Orders are updated with the `executionReport` event.

**Payload:**

```javascript
{
    "subscriptionId": 0,
    "event": {
        "e": "executionReport",        // Event type
        "E": 1499405658658,            // Event time
        "s": "ETHBTC",                 // Symbol
        "c": "mUvoqJxFIILMdfAW5iGSOW", // Client order ID
        "S": "BUY",                    // Side
        "o": "LIMIT",                  // Order type
        "f": "GTC",                    // Time in force
        "q": "1.00000000",             // Order quantity
        "p": "0.10264410",             // Order price
        "P": "0.00000000",             // Stop price
        "F": "0.00000000",             // Iceberg quantity
        "g": -1,                       // OrderListId
        "C": "",                       // Original client order ID; This is the ID of the order being canceled
        "x": "NEW",                    // Current execution type
        "X": "NEW",                    // Current order status
        "r": "NONE",                   // Order reject reason; Please see Order Reject Reason for more information.
        "i": 4293153,                  // Order ID
        "l": "0.00000000",             // Last executed quantity
        "z": "0.00000000",             // Cumulative filled quantity
        "L": "0.00000000",             // Last executed price
        "n": "0",                      // Commission amount
        "N": null,                     // Commission asset
        "T": 1499405658657,            // Transaction time
        "t": -1,                       // Trade ID
        "v": 3,                        // Prevented Match Id; This is only visible if the order expired due to STP
        "I": 8641984,                  // Execution Id
        "w": true,                     // Is the order on the book?
        "m": false,                    // Is this trade the maker side?
        "M": false,                    // Ignore
        "O": 1499405658657,            // Order creation time
        "Z": "0.00000000",             // Cumulative quote asset transacted quantity
        "Y": "0.00000000",             // Last quote asset transacted quantity (i.e. lastPrice * lastQty)
        "Q": "0.00000000",             // Quote Order Quantity
        "W": 1499405658657,            // Working Time; This is only visible if the order has been placed on the book.
        "V": "NONE",                   // SelfTradePreventionMode
        "d": 4,                        // Trailing Delta
        "D": 1668680518494,            // Trailing Time
        "j": 1,                        // Strategy Id
        "J": 1000000,                  // Strategy Type
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
        "gP": "PRIMARY_PEG",           // Pegged Price Type
        "gOT": "PRICE_LEVEL",          // Pegged Offset Type
        "gOV": 5,                      // Pegged Offset Value
        "gp": "1.00000000",            // Pegged Price
        "eR": "INSUFFICIENT_LIQUIDITY" // Expiry Reason. Appears when the order has expired.
    }
}
```

**Note:** Average price can be found by doing `Z` divided by `z`.

#### Conditional Fields in Execution Report

These are fields that appear in the payload only if certain conditions are met.

For additional information on these parameters, please refer to the
[Spot Glossary](faqs/spot_glossary.md).

| Field | Name                          | Description                                                                     | Examples                         |
| ----- | ----------------------------- | ------------------------------------------------------------------------------- | -------------------------------- |
| `d`   | Trailing Delta                | Appears only for trailing stop orders.                                          | `"d": 4`                         |
| `D`   | Trailing Time                 | Appears only for trailing stop orders.                                          | `"D": 1668680518494`             |
| `j`   | Strategy Id                   | Appears only if the `strategyId` parameter was provided upon order placement.   | `"j": 1`                         |
| `J`   | Strategy Type                 | Appears only if the `strategyType` parameter was provided upon order placement. | `"J": 1000000`                   |
| `v`   | Prevented Match Id            | Appears only for orders that expired due to STP.                                | `"v": 3`                         |
| `A`   | Prevented Quantity            | Appears only for orders that expired due to STP.                                | `"A": "3.000000"`                |
| `B`   | Last Prevented Quantity       | Appears only for orders that expired due to STP.                                | `"B": "3.000000"`                |
| `u`   | Trade Group Id                | Appears only for orders that expired due to STP.                                | `"u": 1`                         |
| `U`   | Counter Order Id              | Appears only for orders that expired due to STP.                                | `"U": 37`                        |
| `Cs`  | Counter Symbol                | Appears only for orders that expired due to STP.                                | `"Cs": "BTCUSDT"`                |
| `pl`  | Prevented Execution Quantity  | Appears only for orders that expired due to STP.                                | `"pl": "2.123456"`               |
| `pL`  | Prevented Execution Price     | Appears only for orders that expired due to STP.                                | `"pL": "0.10000001"`             |
| `pY`  | Prevented Execution Quote Qty | Appears only for orders that expired due to STP.                                | `"pY": "0.21234562"`             |
| `W`   | Working Time                  | Appears when the order is working on the book                                   | `"W": 1668683798379`             |
| `b`   | Match Type                    | Appears for orders that have allocations                                        | `"b": "ONE_PARTY_TRADE_REPORT"`  |
| `a`   | Allocation ID                 | Appears for orders that have allocations                                        | `"a": 1234`                      |
| `k`   | Working Floor                 | Appears for orders that potentially have allocations                            | `"k": "SOR"`                     |
| `uS`  | UsedSor                       | Appears for orders that used SOR                                                | `"uS": true`                     |
| `gP`  | Pegged Price Type             | Appears only for Pegged Orders                                                  | `"gP": "PRIMARY_PEG"`            |
| `gOT` | Pegged Offset Type            | Appears only for Pegged Orders                                                  | `"gOT": "PRICE_LEVEL"`           |
| `gOV` | Pegged Offset Value           | Appears only for Pegged Orders                                                  | `"gOV": 5`                       |
| `gp`  | Pegged Price                  | Appears only for Pegged Orders                                                  | `"gp": "1.00000000"`             |
| `eR`  | Expiry Reason                 | Appears when the order has expired.                                             | `"eR": "INSUFFICIENT_LIQUIDITY"` |

#### Order Reject Reason

For additional details, look up the Error Message in the
[Errors](errors.md#messages-for--1010-error_msg_received--2010-new_order_rejected--2011-cancel_rejected-and--2038-order_amend_rejected)
documentation.

| Rejection Reason (`r`)                 | Error Message                                                   |
| -------------------------------------- | --------------------------------------------------------------- |
| `NONE`                                 | N/A (i.e. The order was not rejected.)                          |
| `INSUFFICIENT_BALANCES`                | "Account has insufficient balance for requested action."        |
| `STOP_PRICE_WOULD_TRIGGER_IMMEDIATELY` | "Order would trigger immediately."                              |
| `WOULD_MATCH_IMMEDIATELY`              | "Order would immediately match and take."                       |
| `OCO_BAD_PRICES`                       | "The relationship of the prices for the orders is not correct." |

If the order is an order list, an event named `ListStatus` will be sent in addition to the
`executionReport` event.

**Payload**

```javascript
{
    "subscriptionId": 0,
    "event": {
        "e": "listStatus",             // Event Type
        "E": 1564035303637,            // Event Time
        "s": "ETHBTC",                 // Symbol
        "g": 2,                        // OrderListId
        "c": "OCO",                    // Contingency Type
        "l": "EXEC_STARTED",           // List Status Type
        "L": "EXECUTING",              // List Order Status
        "r": "NONE",                   // List Reject Reason
        "C": "F4QN4G8DlFATFlIUQ0cjdD", // List Client Order ID
        "T": 1564035303625,            // Transaction Time
        // An array of objects
        "O": [
            {
                "s": "ETHBTC",                // Symbol
                "i": 17,                      // OrderId
                "c": "AJYsMjErWJesZvqlJCTUgL" // ClientOrderId
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

**Execution types:**

See [Execution types](enums.md#execution-types) on the Enums page for what each `x` /
`X` value means.

<a id="order-expiry-reason"></a>

#### Order Expiry Reason

See [Expiry Reasons](enums.md#expiry-reasons) on the Enums page for what each `eR` value
means.

Check the [Enums page](enums.md) for more relevant enum definitions.

## Event Stream Terminated

`eventStreamTerminated` is sent when:

- [A listen token subscription](https://developers.binance.com/en/docs/products/margin-trading/listen-token-data-stream#websocket-api-method--userdatastreamsubscribelistentoken)
  expires due to token expiration.
- A [logon subscription](web-socket-api.md#log-in-with-api-key-user_data) ends
  after sending
  [`session.logout`](web-socket-api.md#log-out-of-the-session) method.
- The subscription is stopped via the
  [`userDataStream.unsubscribe`](web-socket-api.md#unsubscribe-from-user-data-stream)
  method.

**Payload:**

```javascript
{
    "subscriptionId": 0,
    "event": {
        "e": "eventStreamTerminated", // Event Type
        "E": 1728973001334            // Event Time
    }
}
```

## External Lock Update

`externalLockUpdate` is sent when part of your spot wallet balance is locked/unlocked by an external
system, for example when used as margin collateral.

**Payload:**

```javascript
{
    "subscriptionId": 0,
    "event": {
        "e": "externalLockUpdate", // Event Type
        "E": 1581557507324,        // Event Time
        "a": "NEO",                // Asset
        "d": "10.00000000",        // Delta
        "T": 1581557507268         // Transaction Time
    }
}
```
