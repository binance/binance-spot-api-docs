# Updates to the Spot Order Count Limit Rules

To reward efficient traders on the Spot market, we have upgraded the order count algorithm to decrement the used 10-second and 24-hour order counts when an order is first partially filled or fully filled.

When any order is filled for the first time (partially or fully), the order count will be decremented by 1 for the current 10-second and 24-hour order count windows. The net effect is that any (partially or fully) filled orders will not count towards the 10-second or 24-hour order limits.

## How it works in detail:
Order placement requests (e.g. `POST /api/v3/order`) return the `X-MBX-ORDER-COUNT-24H` header which indicates how many orders have been placed in the current 24-hour UTC time window.
Below is an example of orders being placed and the `X-MBX-ORDER-COUNT-24H` header value:


|Time| Action                       | X-MBX-ORDER-COUNT-24H|
|--- | ---                          | ----                 |
|T1  | Place LIMIT Order A          | 1 (LIMIT Order A counted) |
|T2  | Place LIMIT Order B          | 2 (LIMIT Order B counted) |
|T3  | Order A partially filled     | 1 (LIMIT Order A decremented) |
|T4  | Order A partially filled     | 1 (No decrement; already decremented) |
|T5  | Place LIMIT Order C          | 2 (LIMIT Order C counted) |
|T5  | Place LIMIT Order D          | 3 (LIMIT Order D counted) |
|T6  | Order C filled               | 2 (LIMIT Order C decremented) |
|T7  | Order B filled               | 1 (LIMIT Order B decremented) |
|T8  | Place MARKET Order E         | 2 (MARKET Order E counted*) (See below) |
|T9  | Place MARKET Order E (FILLS) | 1 (MARKET Order E decremented*) (See below) |
|T10  | Place MARKET Order F (No Fills) | 2 (MARKET Order F counted)  |

Note that this will also decrement the 10-second order count window as well; the example is using the 24-hour window for reference.

*  The decrement is an asynchronous process and there will be a short delay between an order being filled and the order count being decremented. Because of this, orders that immediately fill (such as a MARKET order that takes immediately) will return the order count **at the time of order placement** and the returned count **will not include the decrement**. Order decrements should occur within 1000ms.

## Which timezone is this 24-hour window working on?

UTC

## Which window will be decremented if an order is placed in one window and filled in another window?

At UTC 00:00:00 the 24-hour window will reset. If an order placed on the previous UTC day (e.g. UTC 23:59:00) had a fill on the current day (e.g. UTC 00:01:00), the order count will be decremented in the current window. The same logic applies to the 10-second windows.
