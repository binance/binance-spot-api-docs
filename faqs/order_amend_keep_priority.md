# Order Amend Keep Priority

**Disclaimer**:

* The symbols and values used here are fictional and do not imply anything about the actual setup on the live exchange.
* For simplicity, the examples in this document do not include commission.

## What is Order Amend Keep Priority?

Order Amend Keep Priority request is used to modify (amend) an existing order **without losing order book priority**.

The following order modifications are allowed:

* reduce the quantity of the order

## How can I amend the quantity of my order?

Use the following requests:

| API | Request |
| :---- | :---- |
| REST API | `PUT /api/v3/order/amend/keepPriority` |
| WebSocket API | `order.amend.keepPriority` |
| FIX API | OrderAmendKeepPriorityRequest `<XAK>` |

## What is the difference between "Cancel an Existing Order and Send a New Order" (cancel-replace) and "Order Amend Keep Priority"?

**Cancel an Existing Order and Send a New Order** request cancels the old order and places a new order.<br> Time priority is lost. The new order executes after existing orders at the same price.

**Order Amend Keep Priority** request modifies an existing order in-place. <br>The amended order keeps its time priority among existing orders at the same price.

For example, consider the following order book:

| User | Order ID | Side | Order price | quantity |
| :---- | ----: | :---- | ----: | ----: |
| User A | 10 | BUY | 87,000 | 1.00 |
| ⭐️ YOU | 15 | BUY | 87,000 | 5.50 |
| User B | 20 | BUY | 87,000 | 4.00 |
| User C | 21 | BUY | 86,999 | 2.00 |

Your order 15 is the second one in the queue based on price and time.

You want to reduce the quantity from 5.50 down to 5.00.

If you use **cancel-replace** to cancel `orderId=15` and place a new order with `qty=5.00`, the order book will look like this:

| User | Order ID | Side | Order price | quantity |
| :---- | ----: | :---- | ----: | ----: |
| User A | 10 | BUY | 87,000 | 1.00 |
| ~~⭐️ YOU~~ | ~~11~~ | ~~BUY~~ | ~~87,000~~ | ~~5.50~~ |
| User B | 20 | BUY | 87,000 | 4.00 |
| ⭐️ YOU | (new) 22 | BUY | 87,000 | 5.00 |
| User C | 21 | BUY | 86,999 | 2.00 |

Note that the new order gets a new order ID and you lose time priority: order 22 will trade after the order 20\.

If instead you use **Order Amend Keep Priority** to reduce the quantity of `orderId=15` down to `qty=5.00`, the order book will look like this:

| User | Order ID | Side | Order price | quantity |
| :---- | ----: | :---- | ----: | ----: |
| User A | 10 | BUY | 87,000 | 1.00 |
| ⭐️ YOU | 15 | BUY | 87,000 | (amended) **5.00** |
| User B | 20 | BUY | 87,000 | 4.00 |
| User C | 21 | BUY | 86,999 | 2.00 |

Note that the order ID stays the same and the order keeps its priority in the queue. Only the quantity of the order changes.

## Does Order Amend Keep Priority affect unfilled order count (rate limits)?

Currently, Order Amend Keep Priority requests charge 0 for unfilled order count.

## How do I know if my order has been amended?

If the order was amended successfully, the API response contains your order with the updated quantity.

On User Data Stream, you will receive an `"executionReport"` event with execution type `"x": "REPLACED"`.

If the amended order belongs to an order list and the client order ID has changed, you will also receive a "listStatus" event with list status type `"l": "UPDATED"`.

You can also use the following requests to query order modification history:

| API | Request |
| :---- | :---- |
| REST API | `GET /api/v3/order/amendments` |
| WebSocket API | `order.amendments` |

## What happens if my amend request does not succeed?

If the request fails for any reason (e.g. fails the filters, permissions, account restrictions, etc), then the order amend request is rejected and the order remains unchanged.

## Is it possible to reuse the current clientOrderId for my amended order?

Yes.

By default, amended orders get a random new client order ID, but you can pass the current client order ID in the `newClientOrderId` parameter if you wish to keep it.

## Can Iceberg Orders be amended?

Yes.

Note that an iceberg order's visible quantity will only change if `newQty` is below the pre-amended visible quantity.

## Can Order lists be amended?

Orders in an order list can be amended.

Note that OCO order pairs must have the same quantity, since only one of the orders can ever be executed. This means that amending either order affects both orders.

For OTO orders, the working and pending orders can be amended individually.

## Which symbols allow Order Amend Keep Priority?

This information is available in Exchange Information.
Symbols that allow Order Amend Keep Priority requests have `amendAllowed` set to `true`.
