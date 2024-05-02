# CHANGELOG for Binance SPOT Testnet (2024-05-02)

**Note:** All features here will only apply to the [SPOT Testnet](https://testnet.binance.vision/). 
This is not always synced with the live exchange. 

## 2024-05-02

* One-Triggers-the-Other (OTO) orders and One-Triggers-a-One-Cancels-The-Other (OTOCO) orders are now enabled.
* New requests have been added:
    * REST API:
        * `POST /api/v3/orderList/oto`
        * `POST /api/v3/orderList/otoco`
    * WebSocket API:
        * `orderList.place.oto`
        * `orderList.place.otoco`

---

## 2024-04-04 

General changes:

* Symbol permission information in Exchange Information responses has moved from field `permissions` to field `permissionSets`.
* Field `permissions` will be empty and will be removed in a future release.
* Previously, `"permissions":["SPOT","MARGIN"]` meant that you could place an order on the symbol if your account had `SPOT` or `MARGIN` permissions. The equivalent is `"permissionSets":[["SPOT","MARGIN"]]`. (Note the extra set of square brackets.) Each array of permissions inside the `permissionSets` array is called a "permission set".
* Symbol permissions can now be more complex. `"permissionSets":[["SPOT","MARGIN"],["TRD_GRP_004","TRD_GRP_005"]]` means that you may place an order on the symbol if your account has SPOT or MARGIN permissions **and** `TRD_GRP_004` or `TRD_GRP_005` permissions. There may be an arbitrary number of permission sets in a symbol's `permissionSets`.
* **The weight of the following requests has increased from 10 to 25**: 
	* `GET /api/v3/trades`
	* `GET /api/v3/historicalTrades`
	* `trades.recent`
	* `trades.historical`

REST API

* The `POST /api/v3/order/oco` endpoint is now deprecated on the REST API. You should use the new `POST /api/v3/orderList/oco` endpoint instead. Note that this new endpoint uses different parameters.
* `POST /api/v3/order/oco` has been removed from the Rest API documentation for SPOT Testnet.
* `otoAllowed` will now appear on `GET /api/v3/exchangeInfo`, that indicates if One-Triggers-the-Other (OTO) orders are supported on that symbol.

WebSocket API

* The `orderList.place` request is now deprecated on the WebSocket API. You should now use the new `orderList.place.oco` request instead. Note that this new request uses different parameters.
* `orderList.place` has been removed from the WebSocket API documentation for SPOT Testnet.
* `otoAllowed` will now appear on `exchangeInfo`, that indicates if One-Triggers-the-Other (OTO) orders are supported on that symbol.

SBE

* A new schema 2:0 [spot_2_0.xml](https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_0.xml) has been released for SPOT Testnet. The current schema, 1:0 [spot_1_0.xml](https://github.com/binance/binance-spot-api-docs/blob/becd4d44a09d94821d2dc761ba9197aae8b495c3/sbe/schemas/spot_1_0.xml), will thus be deprecated and retired from the Testnet APIs in 6 months as per our schema deprecation policy.
* When using schema 1:0 on REST API or WebSocket API, group "permissions" in message "ExchangeInfoResponse" will always be empty. Upgrade to schema 2:0 to find permission information in group "permissionSets". See General changes above for more details.
* Responses for deprecated OCO requests are supported by both schema 1:0 and 2:0



---

## 2024-03-13

General changes:

* `GET /api/v3/account` has a new optional parameter `omitZeroBalances`, allowing to hide all zero balances
* `account.status` has a new optional parameter `omitZeroBalances` allowing to hide all zero balances.


User Data Stream:

* New event `listenKeyExpired` is now emitted when a `listenKey` expires.

