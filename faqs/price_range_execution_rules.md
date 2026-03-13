# Price Range Execution Rule

**Disclaimer:**

* The symbols and values used here are fictional and do not imply anything about the actual configuration of the live exchange.

## What are execution rules?

Execution rules are trading rules that are enforced at the time of order execution. The only execution rule currently available is the Price Range rule.

## What does the Price Range Execution Rule do?

This rule ensures that trades may only be executed at prices within and equal to a price range around a reference price.

## How can I query the execution price range allowed for a symbol?

Refer to the following endpoints/methods:

| API | Request |
| ---- | ---- |
| REST API | `GET /api/v3/executionRules` |
| WebSocket API | `executionRules` |

## How can I query the reference price?

Refer to the following endpoints/methods:

| API | Request |
| ---- | ---- |
| REST API | `GET /api/v3/referencePrice` |
| WebSocket API | `referencePrice` |
|WebSocket Streams|`<symbol>@referencePrice`|

Note that the **reference price is continually changing**, so it is recommended to monitor the reference price via WebSocket Streams.

## How does the Price Range Execution Rule work?

As an example, given the hypothetical execution rule for this symbol:

```json
{
  "symbolRules": [
    {
      "symbol": "BAZUSD",
      "rules": [
        {
          "ruleType": "PRICE_RANGE",
          "bidMultiplierUp": "2.0000",
          "bidMultiplierDown": "0.5000",
          "askMultiplierUp": "2.0000",
          "askMultiplierDown": "0.5000"
        }
      ]
    }
  ]
}
```

If the reference price for the symbol is:

```json
{
  "symbol": "BAZUSD",
  "referencePrice": "10.00",
  "timestamp": 1770736694138
}
```

This means that at time `1770736694138`:
1. an order to `BUY` may not execute at a price more than twice the reference price or less than half the reference price and
2. an order to `SELL` may not execute at a price more than twice the reference price or less than half the reference price.

## What happens if an order attempts to execute at a price outside of the allowed price range?

If a taker order attempts to execute at a price outside of the allowed price range, it will be expired (i.e. status: `EXPIRED`) with the expiry reason `EXECUTION_RULE_PRICE_RANGE_EXCEEDED`.

| Service | Reference |
| ---- | ---- |
| Non-FIX APIs | `expiryReason` |
| FIX APIs | `ExpiryReason <25056>` |
| User Data Stream | `"eR"` |

## How is the reference price calculated?

If the reference price is being calculated by the Matching Engine, then a query for the reference price calculation returns `"calculationType": "ARITHMETIC_MEAN"`.

If the reference price is being calculated outside the matching engine, then a query for the reference price calculation returns `"calculationType": "EXTERNAL"`. See below for more details.

<a id="matching-engine-calculation"></a>

## How does the Matching Engine calculate the reference price?

The matching engine calculates the reference price as a simple moving average of trade prices over a time window. The calculation is configured with a bucket width in milliseconds (`bucketWidthMs`) and the number of buckets (`bucketCount`). The bucket width multiplied by the number of buckets defines the size of the time window.

When a trade occurs, the matching engine captures the trade price and adds it to the current bucket. Each bucket has
* an open time, which is aligned to engine time modulo the bucket width
* a trade count, which is a fixed-point integer with four decimal places of precision
* a sum of all the trade prices represented in that bucket, which is a fixed-point integer with an extra four decimal places of precision over the quote asset precision.

The matching engine calculates the average of a particular bucket by dividing the sum by the trade count.
The first trade for a given open time creates a bucket and the matching engine gradually accumulates buckets as trades happen. The matching engine drops a bucket when its close time is outside the time window. This means that:
* The oldest bucket at any given time likely has an open time outside of the time window and a close time inside of the time window.
* The maximum number of buckets tracked by the engine is actually 1 more than the configured `bucketCount`.

The remainder of this explanation refers to the oldest time in the time window as the "cutoff time".

When the oldest bucket straddles the cutoff time, its contents are *prorated*:
* The fraction of the bucket outside the moving window is: (cutoff time - the bucket's open time) divided by the bucket width. Call this the "expired fraction."
* The bucket's trade count is reduced by the expired fraction.
* The bucket's sum is reduced by the expired fraction.
* The open time is set to the cutoff time.

The reference price is the total of the sum in each bucket divided by the total of the trade count in each bucket. Division is truncating integer division.

## How are reference prices calculated outside the matching engine?

If the reference price is being calculated outside the matching engine, then a query for the reference price calculation returns `"externalCalculationId":` followed by an integer number. Each of these numbers indicates a different calculation method.

## External Reference Price Calculation Method 0

The reference price was set manually by a human operator. This calculation method will only be used in situations when algorithmic calculation of the reference price has been deemed unsuitable.
