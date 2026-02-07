# Spot Unfilled Order Count Rules

To ensure a fair and orderly Spot market, we limit the rate at which new orders may be placed.

The rate limit applies to the number of new, *unfilled* orders placed within a time interval. That is, orders which are partially or fully filled do not count against the rate limit.

> [!NOTE]  
> Unfilled order rate limit rewards efficient traders.
> 
>**So long as your orders trade, you can keep trading.**
>
> More information: [How do filled orders affect the rate limit?](#filled-orders-rate-limit)

## What are the current rate limits?

You can query current rate limits using the "exchange information" request.

The `"rateLimitType": "ORDERS"` indicates the current unfilled order rate limit.

Please refer to the API documentation:

| API           | Request                                                    |
|:--------------|:-----------------------------------------------------------|
| FIX API       | [LimitQuery`<XLQ>`](../fix-api.md#limitquery)             |
| REST API      | [`GET /api/v3/exchangeInfo`](../rest-api.md#exchangeInfo) |
| WebSocket API | [`exchangeInfo`](../web-socket-api.md#exchangeInfo)        |

> [!IMPORTANT]  
> Order placement requests are also affected by the general request rate limits on REST and WebSocket API and the message limits on FIX API.
>
> If you send too many requests at a high rate, you will be blocked by the API.

<a id="order-rate-limit"></a>

## How does the unfilled `ORDERS` rate limit work?

Every successful request to place an order adds to the unfilled order count for the current time interval. If too many unfilled orders accumulate during the interval, subsequent requests will be rejected.

For example, if the unfilled order rate limit is 100 per 10 seconds:

```javascript
{
  "rateLimitType": "ORDERS",
  "interval": "SECOND",
  "intervalNum": 10,
  "limit": 100
}
```

then you can place at most 100 new orders between 12:34:00 and 12:34:10, then 100 more from 12:34:10 to 12:34:20, and so on.

>[!TIP]  
>If the newly placed orders receive fills, your unfilled order count decreases and you may place more orders during the time interval.
>
>More information: [How do filled orders affect the rate limit?](#filled-orders-rate-limit)

When an order is rejected by the system due to the unfilled order rate limit, the HTTP status code is set to `429 Too Many Requests` and the error code is `-1015 "Too many new orders"`.

If you encounter these errors, please stop sending orders until the affected rate limit interval expires.

Please refer to the API documentation:

| API           | Documentation                                                    |
|:--------------|:-----------------------------------------------------------------|
| FIX API       | [Unfilled Order Count](../fix-api.md#unfilled-order-count)       |
| REST API      | [Unfilled Order Count](../rest-api.md#unfilled-order-count)      |
| WebSocket API | [Unfilled Order Count](../web-socket-api.md#unfilled-order-count) |

## Is the unfilled order count tracked by IP address?

Unfilled order count is tracked **by (sub)account**.

Unfilled order count is shared across all IP addresses, all API keys, and all APIs.

<a id="filled-orders-rate-limit"></a>

## How do filled orders affect the unfilled order count?

When an order is filled for the first time (partially or fully), your unfilled order count is decremented by one order for all intervals of the `ORDERS` rate limit. Effectively, orders that trade do not count towards the rate limit, allowing efficient traders to keep placing new orders. 

Certain orders provide additional incentive:

* **Orders that do not fill immediately (that is, first fill in the maker phase).**   
* Orders that fill large quantities.

In these cases the unfilled order count may be decremented by more than one order for each order that starts trading.

**Notes:**

* **The examples only give a general idea of the behavior.** The 10-second interval is used for simplicity. The actual configuration on the live exchange may be different.  
* There is a short delay between the order being filled and the unfilled order count update. Please be careful when your unfilled order count is close to the limit.  
* Please refer to [How does unfilled `ORDERS` rate limit work?](#order-rate-limit) to see how you can monitor the unfilled order count depending on the API.

**Example 1** — taker:

| Time     | Action                     | Unfilled order count         |
|:---------|:---------------------------|:-----------------------------|
| 00:00:00 |                            | 0                            |
| 00:00:01 | Place LIMIT order A        | 1 — new order (+1)           |
| 00:00:02 | Place LIMIT order B        | 2 — new order (+1)           |
|          | (order B partially filled) | 1 — first fill as taker (−1) |
| 00:00:03 | Place LIMIT order C        | 2 — new order (+1)           |
| 00:00:04 | (order B partially filled) | 2                            |
| 00:00:04 | (order B filled)           | 2                            |
| 00:00:05 | Place MARKET order D       | 3 — new order (+1)           |
|          | (order D fully filled)     | 2 — first fill as taker (−1) |

Note how for every taker order that immediately trades, the unfilled order count is decremented later, allowing you to keep placing orders.

**Example 2** — maker:

| Time     | Action                     | Unfilled order count         |
|:---------|:---------------------------|:-----------------------------|
| 00:00:00 |                            | 0                            |
| 00:00:01 | Place LIMIT order A        | 1 — new order (+1)           |
| 00:00:01 | Place LIMIT order B        | 2 — new order (+1)           |
| 00:00:02 | Place LIMIT order C        | 3 — new order (+1)           |
| 00:00:02 | Place LIMIT order D        | 4 — new order (+1)           |
| 00:00:02 | Place LIMIT order E        | 5 — new order (+1)           |
| 00:00:03 | (order A partially filled) | 0 — first fill as maker (−5) |
| 00:00:04 | Place LIMIT order F        | 1 — new order (+1)           |
| 00:00:04 | Place LIMIT order G        | 2 — new order (+1)           |
| 00:00:05 | (order A partially filled) | 2                            |
| 00:00:05 | (order A filled)           | 2                            |
| 00:00:05 | (order B partially filled) | 0 — first fill as maker (−5) |
| 00:00:06 | Place LIMIT order H        | 1 — new order (+1)           |

Note how for every maker order that is filled later, the unfilled order count is decremented by a higher amount, allowing you to place more orders.

## How do canceled or expired orders affect the unfilled order count?

Canceling an order does not change the unfilled order count.

Expired orders also do not change the unfilled order count.

**Example:**

| Time     | Action                         | Unfilled order count |
|:---------|:-------------------------------|:---------------------|
| 00:00:00 |                                | 0                    |
| 00:00:01 | Place LIMIT order A            | 1 — new order (+1)   |
| 00:00:02 | Cancel order A                 | 1                    |
| 00:00:02 | Place LIMIT order B            | 2 — new order (+1)   |
| 00:00:03 | Place LIMIT FOK order C        | 3 — new order (+1)   |
|          | (order C is fully filled)      | 2 — fill (−1)        |
| 00:00:05 | Place LIMIT order D            | 3 — new order (+1)   |
| 00:00:06 | Place LIMIT FOK order E        | 4 — new order (+1)   |
|          | (order E expires with no fill) | 4                    |
| 00:00:07 | Cancel order D                 | 4                    |
| 00:00:07 | Place LIMIT order F            | 5 — new order (+1)   |

## Which time zone does `"interval":"DAY"` use?

UTC

## What happens if I placed an order yesterday but it is filled the next day?

New order fills decrease your *current* unfilled order count regardless of when the orders were placed.

**Example:**

| Time             | Action                      | Unfilled order count |
|:-----------------|:----------------------------|:---------------------|
| 2024-01-01 09:00 | Place 5 orders: 1..5        | 5                    |
| 2024-01-02 00:00 | (rate limit interval reset) | 0                    |
| 2024-01-02 09:00 | Place 10 orders: 6..15      | 10                   |
| 2024-01-02 12:00 | (orders 1..5 are filled)    | 5                    |
| 2024-01-02 13:00 | (orders 6..10 are filled)   | 0                    |
| 2024-01-02 14:00 | Place 2 orders: 16, 17      | 2                    |
| 2024-01-02 15:00 | (orders 11..15 are filled)  | 0                    |

**Note:** You do not get credit for order fills. That is, once the unfilled order count is down to zero, additional fills will not decrease it further. New orders will increase the count as usual.
