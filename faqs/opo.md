# One Pays the Other (OPO)

## What is One Pays the Other?

This is a special behavior of OTO and OTOCO where the received amount from the working order is used for the quantity of the pending order(s). Thus the only balance requirement on order placement is that of the working order.

The received funds from the working order are *locked* for use by the pending order(s) and not available for trading or withdrawal. If the order list is canceled before the pending order is placed on the Matching Engine then these locked funds are unlocked.

OPO is almost identical to `OTO`, with the exception of the absence of the `quantity` of the pending order(s).

## How can I use this? 

Please refer to the following table:

| API | Request |
| ----- | ----- |
| REST API | `POST /api/v3/orderList/opo` <br> `POST /api/v3/orderList/opoco` |
| WebSocket API | `orderList.place.opo` <br>`orderList.place.opoco`  |
| FIX API | NewOrderList `<E>` where OPO `(25046)`=`true` |

## What is the difference with this order list from other order lists?

* The pending order(s) are placed into the Matching Engine without quantity; the quantity will be based on the received quantity from the working order once it fully fills.  
* The received quantity will have commission deducted as appropriate. The commission is taken from the available funds instead (i.e. `free` balances) if the received asset is not BNB and there are enough available funds.  
* The quantity of the pending order(s) are evaluated (e.g. filters) after the working order fully fills.  
* If a symbol has `LOT_SIZE` and/or `MARKET_LOT_SIZE` filters configured, the quantity of the pending order(s) are adjusted to meet them. Any of the locked quantity not used in the pending order(s) will be unlocked and returned to the `free` balances.  
* A pending OPO order's quantity may not be amended until the working order has been fully filled.  
* Only working orders on the `BUY` side and pending order(s) on the `SELL` side are accepted.

## Which symbols allow OPO orders? 

| Order | Reqired in Exchange Information |
| ----- | ----- |
| OPO   | `otoAllowed` and `opoAllowed`
| OPOCO  |`otoAllowed`, `opoAllowed`, and `ocoAllowed`


