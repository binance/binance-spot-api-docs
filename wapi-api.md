# Public WAPI for Binance (2018-07-18)
# General API Information
* The base endpoint is: **https://api.binance.com**
* All endpoints return either a JSON object or array.
* Data is returned in **ascending** order. Oldest first, newest last.
* All time and timestamp related fields are in milliseconds.
* HTTP `4XX` return codes are used for for malformed requests;
  the issue is on the sender's side.
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `418` return code is used when an IP has been auto-banned for continuing to send requests after receiving `429` codes.
* HTTP `5XX` return codes are used for internal errors; the issue is on
  Binance's side.
* HTTP `504` return code is used when the API successfully sent the message
but not get a response within the timeout period.
It is important to **NOT** treat this as a failure; the execution status is
**UNKNOWN** and could have been a success.
* Any endpoint can retun an ERROR; the error payload is as follows:
```javascript
{
  "success": false,
  "msg": "Invalid symbol."
}
```

* Specific error codes and messages defined in another document.
* For `GET` endpoints, parameters must be sent as a `query string`.
* For `POST`, `PUT`, and `DELETE` endpoints, the parameters may be sent as a
  `query string` or in the `request body` with content type
  `application/x-www-form-urlencoded`. You may mix parameters between both the
  `query string` and `request body` if you wish to do so.
* Parameters may be sent in any order.
* If a parameter sent in both the `query string` and `request body`, the
  `query string` parameter will be used.

# LIMITS
* The `/wapi/v3` `rateLimits` array contains objects related to the exchange's `REQUESTS` and `ORDER` rate limits.
* A 429 will be returned when either rate limit is violated.
* Each route has a `weight` which determines for the number of requests each endpoint counts for. Heavier endpoints and endpoints that do operations on multiple symbols will have a heavier `weight`.
* When a 429 is recieved, it's your obligation as an API to back off and not spam the API.
* **Repeatedly violating rate limits and/or failing to back off after receiving 429s will result in an automated IP ban (http status 418).**
* IP bans are tracked and **scale in duration** for repeat offenders, **from 2 minutes to 3 days**.

# Endpoint security type
* Each endpoint has a security type that determines the how you will
  interact with it.
* API-keys are passed into the Rest API via the `X-MBX-APIKEY`
  header.
* API-keys and secret-keys **are case sensitive**.
* API-keys can be configured to only access certain types of secure endpoints.
 For example, one API-key could be used for TRADE only, while another API-key
 can access everything except for TRADE routes.
* By default, API-keys can access all secure routes.

Security Type | Description
------------ | ------------
NONE | Endpoint can be accessed freely.
TRADE | Endpoint requires sending a valid API-Key and signature.
USER_DATA | Endpoint requires sending a valid API-Key and signature.
USER_STREAM | Endpoint requires sending a valid API-Key.
MARKET_DATA | Endpoint requires sending a valid API-Key.


* `TRADE` and `USER_DATA` endpoints are `SIGNED` endpoints.

# SIGNED (TRADE and USER_DATA) Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, to be
  sent in the  `query string` or `request body`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* The `signature` is **not case sensitive**, but it should come last in the list or params!
* `totalParams` is defined as the `query string` concatenated with the
  `request body`.

## Timing security
* A `SIGNED` endpoint also requires a parameter, `timestamp`, to be sent which
  should be the millisecond timestamp of when the request was created and sent.
* An additional parameter, `recvWindow`, may be sent to specific the number of
  milliseconds after `timestamp` the request is valid for. If `recvWindow`
  is not sent, **it defaults to 5000**.
* The logic is as follows:
  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```

**Serious trading is about timing.** Networks can be unstable and unreliable,
which can lead to requests taking varying amounts of time to reach the
servers. With `recvWindow`, you can specify that the request must be
processed within a certain number of milliseconds or be rejected by the
server.


**Tt recommended to use a small recvWindow of 5000 or less!**


## SIGNED Endpoint Examples for POST /wapi/v3/withdraw.html
Here is a step-by-step example of how to send a vaild signed payload from the
Linux command line using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
apiKey | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


Parameter | Value
------------ | ------------
asset | ETH
address  |0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b
addressTag | 1 (Secondary address identifier for coins like XRP,XMR etc.)
amount | 1
recvWindow | 5000
name | addressName (Description of the address)
timestamp | 1508396497000
signature  | 157fb937ec848b5f802daa4d9f62bea08becbf4f311203bda2bd34cd9853e320


### Example 1: As a query string
* **queryString:** asset=ETH&address=0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b&amount=1&recvWindow=5000&name=test&timestamp=1510903211000
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "asset=ETH&address=0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b&amount=1&recvWindow=5000&name=test&timestamp=1510903211000" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= 157fb937ec848b5f802daa4d9f62bea08becbf4f311203bda2bd34cd9853e320
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://www.binance.com/wapi/v3/withdraw.html?asset=ETH&address=0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b&amount=1&recvWindow=5000&name=test&timestamp=1510903211000&signature=157fb937ec848b5f802daa4d9f62bea08becbf4f311203bda2bd34cd9853e320'
    ```

Note that for wapi, parameters must be sent in query strings.

### Withdraw
```
POST /wapi/v3/withdraw.html (HMAC SHA256)
```
Submit a withdraw request.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset	|  STRING |	YES
network | STRING | NO
address	 | STRING | YES	
addressTag | STRING | NO | Secondary address identifier for coins like XRP,XMR etc.
amount | DECIMAL | YES	
name | STRING | NO | Description of the address
recvWindow | LONG | NO	
timestamp | LONG | YES	
**Response:**
```javascript
{
    "msg": "success",
    "success": true,
    "id":"7213fea8e94b4a5593d507237e5a555b"
}
```


### Deposit History (USER_DATA)
```
GET /wapi/v3/depositHistory.html (HMAC SHA256)
```
Fetch deposit history.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | NO	
status | INT | NO | 0(0:pending,6: credited but cannot withdraw, 1:success)
startTime | LONG | NO	
endTime | LONG | NO	
recvWindow | LONG | NO	
timestamp | LONG | YES	


**Response:**
```javascript
{
    "depositList": [
        {
            "insertTime": 1508198532000,
            "amount": 0.04670582,
            "asset": "ETH",
            "address": "0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b",
            "txId": "0xdf33b22bdb2b28b1f75ccd201a4a4m6e7g83jy5fc5d5a9d1340961598cfcb0a1",
            "status": 1
        },
        {
            "insertTime": 1508298532000,
            "amount": 1000,
            "asset": "XMR",
            "address": "463tWEBn5XZJSxLU34r6g7h8jtxuNcDbjLSjkn3XAXHCbLrTTErJrBWYgHJQyrCwkNgYvyV3z8zctJLPCZy24jvb3NiTcTJ",
            "addressTag": "342341222",
            "txId": "b3c6219639c8ae3f9cf010cdc24fw7f7yt8j1e063f9b4bd1a05cb44c4b6e2509",
            "status": 1
        }
    ],
    "success": true
}
```

### Withdraw History (USER_DATA)
```
GET /wapi/v3/withdrawHistory.html (HMAC SHA256)
```
Fetch withdraw history.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | NO	
status | INT | NO | 0(0:Email Sent,1:Cancelled 2:Awaiting Approval 3:Rejected 4:Processing 5:Failure 6Completed)
startTime | LONG | NO	
endTime | LONG | NO	
recvWindow | LONG | NO	
timestamp | LONG | YES	


**Response:**
```javascript
{
    "withdrawList": [
        {
            "id":"7213fea8e94b4a5593d507237e5a555b",
            "amount": 0.99,
	    "transactionFee": 0.01,
            "address": "0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b",
            "asset": "ETH",
            "txId": "0xdf33b22bdb2b28b1f75ccd201a4a4m6e7g83jy5fc5d5a9d1340961598cfcb0a1",
            "applyTime": 1508198532000,
            "status": 4
        },
        {
            "id":"7213fea8e94b4a5534ggsd237e5a555b",
            "amount": 999.9999,
	    "transactionFee": 0.0001,
            "address": "463tWEBn5XZJSxLU34r6g7h8jtxuNcDbjLSjkn3XAXHCbLrTTErJrBWYgHJQyrCwkNgYvyV3z8zctJLPCZy24jvb3NiTcTJ",
            "addressTag": "342341222",
            "txId": "b3c6219639c8ae3f9cf010cdc24fw7f7yt8j1e063f9b4bd1a05cb44c4b6e2509",
            "asset": "XMR",
            "applyTime": 1508198532000,
            "status": 4
        }
    ],
    "success": true
}
```




### Deposit Address (USER_DATA)
```
GET  /wapi/v3/depositAddress.html (HMAC SHA256)
```
Fetch deposit address.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | YES	
status | Boolean |NO
recvWindow | LONG | NO	
timestamp | LONG | YES	

**Response:**
```javascript
{
    "address": "0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b",
    "success": true,
    "addressTag": "1231212",
    "asset": "BNB"
}

```

### Account Status (USER_DATA)
```
GET /wapi/v3/accountStatus.html
```
Fetch account status detail.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO	
timestamp | LONG | YES	

**Response:**
```javascript
{
    "msg": "Order failed:Low Order fill rate! Will be reactivated after 5 minutes.",
    "success": true,
    "objs": [
        "5"
    ]
}
```

### System Status (System)
```
GET /wapi/v3/systemStatus.html
```

Fetch system status.
**Response:**
```javascript
{ 
    "status": 0,              // 0: normal，1：system maintenance
    "msg": "normal"           // normal or system maintenance
}
```

### Account API Trading Status (USER_DATA)
```
GET /wapi/v3/apiTradingStatus.html
```
Fetch account api trading status detail.

For more details about our api trading rules, please refer to the link:https://support.binance.com/hc/en-us/articles/115003235691


**Weight:**
1


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO  
timestamp | LONG | YES  

**Response:**
```javascript
{
    "success": true,     // Query result
    "status": {          // API trading status detail
        "isLocked": false,   // API trading function is locked or not
        "plannedRecoverTime": 0,  // If API trading function is locked, this is the planned recover time
        "triggerCondition": { 
            "GCR": 150,  // Number of GTC orders
            "IFER": 150, // Number of FOK/IOC orders
            "UFR": 300   // Number of orders
        },
        "indicators": {  // The indicators updated every 30 seconds
           "BTCUSDT": [  // The symbol
            {
            "i": "UFR",  // Unfilled Ratio (UFR)
            "c": 20,     // Count of all orders
            "v": 0.05,   // Current UFR value
            "t": 0.995   // Trigger UFR value
            },
            {
            "i": "IFER", // IOC/FOK Expiration Ratio (IFER)
            "c": 20,     // Count of FOK/IOC orders
            "v": 0.99,   // Current IFER value
            "t": 0.99    // Trigger IFER value
            },
            {
            "i": "GCR",  // GTC Cancellation Ratio (GCR)
            "c": 20,     // Count of GTC orders
            "v": 0.99,   // Current GCR value
            "t": 0.99    // Trigger GCR value
            }
            ],
            "ETHUSDT": [ 
            {
            "i": "UFR",
            "c": 20,
            "v": 0.05,
            "t": 0.995
            },
            {
            "i": "IFER",
            "c": 20,
            "v": 0.99,
            "t": 0.99
            },
            {
            "i": "GCR",
            "c": 20,
            "v": 0.99,
            "t": 0.99
            }
            ]
        },
        "updateTime": 1547630471725   // The query result return time
    }
}
```

### DustLog (USER_DATA)
```
GET /wapi/v3/userAssetDribbletLog.html   (HMAC SHA256)
```
Fetch small amounts of assets exchanged BNB records.


**Weight:**
1


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO  
timestamp | LONG | YES  

**Response:**
```javascript
{
    "success": true, 
    "results": {
        "total": 2,   //Total counts of exchange
        "rows": [
            {
                "transfered_total": "0.00132256",//Total transfered BNB amount for this exchange.
                "service_charge_total": "0.00002699",   //Total service charge amount for this exchange.
                "tran_id": 4359321,
                "logs": [           //Details of  this exchange.
                    {
                        "tranId": 4359321,
                        "serviceChargeAmount": "0.000009",
                        "uid": "10000015",
                        "amount": "0.0009",
                        "operateTime": "2018-05-03 17:07:04",
                        "transferedAmount": "0.000441",
                        "fromAsset": "USDT"
                    },
                    {
                        "tranId": 4359321,
                        "serviceChargeAmount": "0.00001799",
                        "uid": "10000015",
                        "amount": "0.0009",
                        "operateTime": "2018-05-03 17:07:04",
                        "transferedAmount": "0.00088156",
                        "fromAsset": "ETH"
                    }
                ],
                "operate_time": "2018-05-03 17:07:04" //The time of this exchange.
            },
            {
                "transfered_total": "0.00058795",
                "service_charge_total": "0.000012",
                "tran_id": 4357015,
                "logs": [       // Details of  this exchange.
                    {
                        "tranId": 4357015,
                        "serviceChargeAmount": "0.00001",
                        "uid": "10000015",
                        "amount": "0.001",
                        "operateTime": "2018-05-02 13:52:24",
                        "transferedAmount": "0.00049",
                        "fromAsset": "USDT"
                    },
                    {
                        "tranId": 4357015,
                        "serviceChargeAmount": "0.000002",
                        "uid": "10000015",
                        "amount": "0.0001",
                        "operateTime": "2018-05-02 13:51:11",
                        "transferedAmount": "0.00009795",
                        "fromAsset": "ETH"
                    }
                ],
                "operate_time": "2018-05-02 13:51:11"
            }
        ]
    }
}
```

### Trade Fee (USER_DATA)
```
GET  /wapi/v3/tradeFee.html (HMAC SHA256)
```
Fetch trade fee.


**Weight:**
1


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO  
timestamp | LONG | YES  
symbol | STRING | NO

**Response:**
```javascript
{
	"tradeFee": [{
		"symbol": "ADABNB",
		"maker": 0.9000,
		"taker": 1.0000
	}, {
		"symbol": "BNBBTC",
		"maker": 0.3000,
		"taker": 0.3000
	}],
	"success": true
}
```


### Asset Detail (USER_DATA)
```
GET  /wapi/v3/assetDetail.html (HMAC SHA256)
```
Fetch asset detail.


**Weight:**
1


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO  
timestamp | LONG | YES  

**Response:**
```javascript
{
    "success": true,
    "assetDetail": {
        "CTR": {
            "minWithdrawAmount": "70.00000000", //min withdraw amount
            "depositStatus": false,//deposit status
            "withdrawFee": 35, // withdraw fee
            "withdrawStatus": true, //withdraw status
            "depositTip": "Delisted, Deposit Suspended" //reason
        },
        "SKY": {
            "minWithdrawAmount": "0.02000000",
            "depositStatus": true,
            "withdrawFee": 0.01,
            "withdrawStatus": true
        }	
    }
}
```


### Query Sub-account List(For Master Account)
```
GET   /wapi/v3/sub-account/list.html (HMAC SHA256)
```
Fetch sub account list.


**Weight:**
1


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
email | STRING | NO | Sub-account email
status | STRING | NO | Sub-account status: enabled or disabled
page | INT | NO | Default value: 1
limit | INT | NO | Default value: 500
recvWindow | LONG | NO  
timestamp | LONG | YES  

**Response:**
```javascript
{
    "success":true,
    "subAccounts":[
        {
            "email":"123@test.com",
            "status":"enabled",
            "activated":true,
            "mobile":"91605290",
            "gAuth":true,
            "createTime":1544433328000
        },
        {
            "email":"321@test.com",
            "status":"disabled",
            "activated":true,
            "mobile":"22501238",
            "gAuth":true,
            "createTime":1544433328000
        }
    ]
}
```

### Query Sub-account Transfer History(For Master Account)
```
GET   /wapi/v3/sub-account/transfer/history.html (HMAC SHA256)
```
Fetch transfer history list


**Weight:**
1


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
email | STRING | YES | Sub-account email
startTime | LONG | NO | Default return the history with in 100 days
endTime | LONG | NO | Default return the history with in 100 days
page | INT | NO | Default value: 1
limit | INT | NO | Default value: 500
recvWindow | LONG | NO  
timestamp | LONG | YES  

**Response:**
```javascript
{
    "success":true,
    "transfers":[
        {
            "from":"aaa@test.com",
            "to":"bbb@test.com",
            "asset":"BTC",
            "qty":"1",
            "time":1544433328000
        },
        {
            "from":"bbb@test.com",
            "to":"ccc@test.com",
            "asset":"ETH",
            "qty":"2",
            "time":1544433328000
        }
    ]
}
```

### Sub-account Transfer(For Master Account)
```
POST   /wapi/v3/sub-account/transfer.html (HMAC SHA256)
```
Execute sub-account transfer


**Weight:**
1


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
fromEmail | STRING | YES | Sender email
toEmail | STRING | YES | Recipient email
asset | STRING | YES
amount | DECIMAL | YES
recvWindow | LONG | NO  
timestamp | LONG | YES  

**Response:**
```javascript
{
    "success":true,
    "txnId":"2966662589"
}

```


### Query Sub-account Assets(For Master Account)
```
GET   /wapi/v3/sub-account/assets.html (HMAC SHA256)
```
Fetch sub-account assets

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
email | STRING | YES  | Sub account email
recvWindow | LONG | NO  
timestamp | LONG | YES  
symbol | STRING | NO

**Response:**
```javascript
{

    "success":true,
    "balances":[
        {
            "asset":"ADA",
            "free":10000,
            "locked":0
        },
        {
            "asset":"BNB",
            "free":10003,
            "locked":0
        },
        {
            "asset":"BTC",
            "free":11467.6399,
            "locked":0
        },
        {
            "asset":"ETH",
            "free":10004.995,
            "locked":0
        },
        {
            "asset":"USDT",
            "free":11652.14213,
            "locked":0
        }
    ]
}
```

### Dust Transfer (USER_DATA)
```
Post /sapi/v1/asset/dust (HMAC SHA256)
```
Convert dust assets to BNB.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | ARRAY | YES | The asset being converted. For example: asset=BTC&asset=USDT
recvWindow | LONG | NO |
timestamp | LONG | YES |


**Response:**
```javascript
{
    "totalServiceCharge":"0.02102542",
    "totalTransfered":"1.05127099",
    "transferResult":[
        {
            "amount":"0.03000000",
            "fromAsset":"ETH",
            "operateTime":1563368549307,
            "serviceChargeAmount":"0.00500000",
            "tranId":2970932918,
            "transferedAmount":"0.25000000"
        },
        {
            "amount":"0.09000000",
            "fromAsset":"LTC",
            "operateTime":1563368549404,
            "serviceChargeAmount":"0.01548000",
            "tranId":2970932918,
            "transferedAmount":"0.77400000"
        },
        {
            "amount":"248.61878453",
            "fromAsset":"TRX",
            "operateTime":1563368549489,
            "serviceChargeAmount":"0.00054542",
            "tranId":2970932918,
            "transferedAmount":"0.02727099"
        }
    ]
}
```

### Asset dividend record (USER_DATA)
```
Get /sapi/v1/asset/assetDividend (HMAC SHA256)
```
Query asset dividend record.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | NO |
startTime | LONG | NO |
endTime | LONG | NO |
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
{
    "rows":[
        {
            "amount":"10.00000000",
            "asset":"BHFT",
            "divTime":1563189166000,
            "enInfo":"BHFT distribution",
            "tranId":2968885920
        },
        {
            "amount":"10.00000000",
            "asset":"BHFT",
            "divTime":1563189165000,
            "enInfo":"BHFT distribution",
            "tranId":2968885920
        }
    ],
    "total":2
}
```
