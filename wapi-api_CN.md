# REST接口(账户与资金) (2018-07-18)
# 基本信息
* 本篇列出REST接口的baseurl **https://api.binance.com**
* 所有接口的响应都是JSON格式
* 响应中如有数组，数组元素以时间升序排列，越早的数据越提前。
* 所有时间、时间戳均为UNIX时间，单位为毫秒
* HTTP `4XX` 错误码用于指示错误的请求内容、行为、格式。
* HTTP `429` 错误码表示警告访问频次超限，即将被封IP
* HTTP `418` 表示收到429后继续访问，于是被封了。
* HTTP `5XX` 错误码用于指示Binance服务侧的问题。 
* HTTP `504` 表示API服务端已经向业务核心提交了请求但未能获取响应，特别需要注意的是`504`代码不代表请求失败，而是未知。很可能已经得到了执行，也有可能执行失败，需要做进一步确认。
* 每个接口都有可能抛出异常，异常响应格式如下：
```javascript
{
  "code": -1121,
  "msg": "Invalid symbol."
}
```


* 具体的错误码及其解释在[错误代码汇总.md](./错误代码汇总.md)
* 本篇列出的接口,所有参数必须在`query string`中发送.
* `POST`, `PUT`, 和 `DELETE` 方法的接口, 参数可以在 `query string`中发送，也可以在 `request body`中发送(content type `application/x-www-form-urlencoded`。允许混合这两种方式发送参数。但如果同一个参数名在query string和request body中都有，query string中的会被优先采用。
* 对参数的顺序不做要求。


# 访问限制
* 在 `/api/v1/exchangeInfo`接口中`rateLimits`数组里包含有REST接口(不限于本篇的REST接口)的访问限制。包括带权重的访问频次限制、下单速率限制。
* 违反上述任何一个访问限制都会收到HTTP 429，这是一个警告.
* 每一个接口均有一个相应的权重(weight)，有的接口根据参数不同可能拥有不同的权重。越消耗资源的接口权重就会越大。
* 当收到429告警时，调用者应当降低访问频率或者停止访问。
* **收到429后仍然继续违反访问限制，会被封禁IP，并收到418错误码**
* 频繁违反限制，封禁时间会逐渐延长，**从最短2分钟到最长3天**.

# 接口鉴权类型
* 每个接口都有自己的鉴权类型，鉴权类型决定了访问时应当进行何种鉴权
* 如果需要 API-key，应当在HTTP头中以`X-MBX-APIKEY`字段传递
* API-key 与 API-secret 是大小写敏感的
* 可以在网页用户中心修改API-key 所具有的权限，例如读取账户信息、发送交易指令、发送提现指令

鉴权类型 | 描述
------------ | ------------
NONE | 不需要鉴权的接口
TRADE | 需要有效的API-KEY和签名
USER_DATA | 需要有效的API-KEY和签名
USER_STREAM | 需要有效的API-KEY
MARKET_DATA | 需要有效的API-KEY


# 需要签名的接口 (TRADE 与 USER_DATA)
* 调用这些接口时，除了接口本身所需的参数外，还需要传递`signature`即签名参数。
* 签名使用`HMAC SHA256`算法. API-KEY所对应的API-Secret作为 `HMAC SHA256` 的密钥，其他所有参数作为`HMAC SHA256`的操作对象，得到的输出即为签名。
* 签名大小写不敏感。
* 当同时使用query string和request body时，`HMAC SHA256`的输入query string在前，request body在后

## 时间同步安全
* 签名接口均需要传递`timestamp`参数, 其值应当是请求发送时刻的unix时间戳（毫秒）
* 服务器收到请求时会判断请求中的时间戳，如果是5000毫秒之前发出的，则请求会被认为无效。这个时间窗口值可以通过发送可选参数`recvWindow`来自定义。
* 另外，如果服务器计算得出客户端时间戳在服务器时间的‘未来’一秒以上，也会拒绝请求。
* 逻辑伪代码：
  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```

**关于交易时效性** 
互联网状况并不100%可靠，不可完全依赖,因此你的程序本地到币安服务器的时延会有抖动.
这是我们设置`recvWindow`的目的所在，如果你从事高频交易，对交易时效性有较高的要求，可以灵活设置recvWindow以达到你的要求。
**不推荐使用5秒以上的recvWindow**



## POST /wapi/v3/withdraw.html 的示例
以下是在linux bash环境下使用 echo openssl 和curl工具实现的一个调用接口下单的示例
apikey、secret仅供示范

Key | Value
------------ | ------------
apiKey | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


参数 | 取值
------------ | ------------
asset | ETH
address  |0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b
addressTag | 1 (Secondary address identifier for coins like XRP,XMR etc.)
amount | 1
recvWindow | 5000
name | addressName (Description of the address)
timestamp | 1508396497000
signature  | 157fb937ec848b5f802daa4d9f62bea08becbf4f311203bda2bd34cd9853e320


### Example: 
* **queryString:** asset=ETH&address=0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b&amount=1&recvWindow=5000&name=test&timestamp=1510903211000
* **HMAC SHA256 签名:**

    ```
    [linux]$ echo -n "asset=ETH&address=0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b&amount=1&recvWindow=5000&name=test&timestamp=1510903211000" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= 157fb937ec848b5f802daa4d9f62bea08becbf4f311203bda2bd34cd9853e320
    ```


* **curl 命令:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://www.binance.com/wapi/v3/withdraw.html?asset=ETH&address=0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b&amount=1&recvWindow=5000&name=test&timestamp=1510903211000&signature=157fb937ec848b5f802daa4d9f62bea08becbf4f311203bda2bd34cd9853e320'
    ```

### 提现
```
POST /wapi/v3/withdraw.html (HMAC SHA256)
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset	|  STRING |	YES	
address	 | STRING | YES	
addressTag | STRING | NO | 某些币种例如 XRP,XMR 允许填写备注
amount | DECIMAL | YES	
name | STRING | NO | 地址的备注，填写该参数后会加入该币种的提现地址簿。地址簿上限为20，超出后会造成提现失败。
recvWindow | LONG | NO	
timestamp | LONG | YES	
**响应:**
```javascript
{
    "msg": "success",
    "success": true,
    "id":"7213fea8e94b4a5593d507237e5a555b"
}
```


### 充值历史 (USER_DATA)
```
GET /wapi/v3/depositHistory.html (HMAC SHA256)
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | NO	
status | INT | NO | 0(0:充值进行中,1:充值已经成功)
startTime | LONG | NO	
endTime | LONG | NO	
recvWindow | LONG | NO	
timestamp | LONG | YES	


**响应:**
```javascript
{
    "depositList": [
        {
            "insertTime": 1508198532000, //币安系统记录该笔充值的时间
            "amount": 0.04670582, //充值金额
            "asset": "ETH", //充值资产
            "address": "0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b", //充值来源地址
            "txId": "0xdf33b22bdb2b28b1f75ccd201a4a4m6e7g83jy5fc5d5a9d1340961598cfcb0a1", //充值交易id
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

### 提现历史 (USER_DATA)
```
GET /wapi/v3/withdrawHistory.html (HMAC SHA256)
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | NO	
status | INT | NO | 0(0:已发送确认Email,1:已被用户取消 2:等待确认 3:被拒绝 4:处理中 5:提现交易失败 6 提现完成)
startTime | LONG | NO	
endTime | LONG | NO	
recvWindow | LONG | NO	
timestamp | LONG | YES	


**响应:**
```javascript
{
    "withdrawList": [
        {
            "id":"7213fea8e94b4a5593d507237e5a555b",
            "amount": 1,
            "address": "0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b",
            "asset": "ETH",
            "txId": "0xdf33b22bdb2b28b1f75ccd201a4a4m6e7g83jy5fc5d5a9d1340961598cfcb0a1",
            "applyTime": 1508198532000,
            "status": 4
        },
        {
            "id":"7213fea8e94b4a5534ggsd237e5a555b",//该笔提现在币安的id
            "amount": 1000, //提现金额
            "address": "463tWEBn5XZJSxLU34r6g7h8jtxuNcDbjLSjkn3XAXHCbLrTTErJrBWYgHJQyrCwkNgYvyV3z8zctJLPCZy24jvb3NiTcTJ", //提现目的地址
            "addressTag": "342341222", //提现备注 只对某些币种存在
            "txId": "b3c6219639c8ae3f9cf010cdc24fw7f7yt8j1e063f9b4bd1a05cb44c4b6e2509", //提现交易id
            "asset": "XMR",//提现资产类别
            "applyTime": 1508198532000, //提现申请发起时间
            "status": 4 //提现状态
        }
    ],
    "success": true
}
```




### 获取充值地址(USER_DATA)
```
GET  /wapi/v3/depositAddress.html (HMAC SHA256)
```


**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | YES	
status | Boolean |NO
recvWindow | LONG | NO	
timestamp | LONG | YES	

**响应:**
```javascript
{
    "address": "0x6915f16f8791d0a1cc2bf47c13a6b2a92000504b",
    "success": true,
    "addressTag": "1231212",
    "asset": "BNB"
}

```

### 账户状态 (USER_DATA)
```
GET /wapi/v3/accountStatus.html
```

**权重:**
1

**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO	
timestamp | LONG | YES	

**响应:**
```javascript
{
    "msg": "Order failed:Low Order fill rate! Will be reactivated after 5 minutes.", //msg中给出了账户状态，左边的例子表示由于成交率过低被暂停交易权限。
    "success": true,
    "objs": [
        "5"
    ]
}
```

### 系统状态 (System)
```
GET /wapi/v3/systemStatus.html
```
**响应:**
```javascript
{ 
    "status": 0,              // 0: 正常，1：系统维护
    "msg": "normal"           // normal|system maintenance
}
```

### 小额资产转换历史 (USER_DATA)
```
GET /wapi/v3/userAssetDribbletLog.html   (HMAC SHA256)
```

**权重:**
1


**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO  
timestamp | LONG | YES  

**响应:**
```javascript
{
    "success": true, 
    "results": {
        "total": 2,   //共计发生过的转换笔数
        "rows": [
            {
                "transfered_total": "0.00132256",//本次转换所得BNB
                "service_charge_total": "0.00002699",   //本次转换手续费(BNB)
                "tran_id": 4359321,
                "logs": [           //本次转换的细节 下面的参数太简单了，不翻译了
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

### 交易手续费率查询 (USER_DATA)
```
GET  /wapi/v3/tradeFee.html (HMAC SHA256)
```

**权重:**
1


**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO  
timestamp | LONG | YES  
symbol | STRING | NO

**响应:**
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


### 上架资产详情 (USER_DATA)
```
GET  /wapi/v3/assetDetail.html (HMAC SHA256)
```

**权重:**
1


**参数:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO  
timestamp | LONG | YES  

**响应:**
```javascript
{
    "success": true,
    "assetDetail": {
        "CTR": { //资产代号
            "minWithdrawAmount": "70.00000000", //最小提现数量
            "depositStatus": false,//是否可以充值
            "withdrawFee": 35, // 提现手续费
            "withdrawStatus": true, //是否开放提现
            "depositTip": "Delisted, Deposit Suspended" //暂停充值的原因(如果暂停才有这一项)
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
