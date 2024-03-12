# CHANGELOG for Binance SPOT Testnet (2024-03-13)

**Note:** All features here will only apply to the [SPOT Testnet](https://testnet.binance.vision/). These features will be released to the live exchange at a later date.

## 2024-03-13

General changes:

* `GET /api/v3/account` has a new optional parameter `omitZeroBalances`, allowing to hide all non-zero balances
* `account.status` has a new optional parameter `omitZeroBalances` allowing to hide all non-zero balances.


User Data Stream:

* New event `listenKeyExpired` is now emitted when a `listenKey` expires.

