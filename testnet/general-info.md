# Testnet General Info

<a id="faq-how-to-use"></a>
## How can I use the Spot Test Network?

Step 1: Log in on this [website](https://testnet.binance.vision/), and generate an API Key.

Step 2: Follow the [official documentation of the Spot API](https://developers.binance.com/docs/binance-spot-api-docs/CHANGELOG), replacing the URLs of the endpoints with the following values:

| Spot API URLs | Spot Test Network URLs |
| --- | --- |
| https://api.binance.com/api<br>https://api-gcp.binance.com/api<br>https://api1.binance.com/api<br>https://api2.binance.com/api<br>https://api3.binance.com/api<br>https://api4.binance.com/api | **https://testnet.binance.vision/api**<br>**https://api1.testnet.binance.vision/api** |
| wss://ws-api.binance.com/ws-api/v3<br>wss://ws-api.binance.com:9443/ws-api/v3 | **wss://ws-api.testnet.binance.vision/ws-api/v3**<br>**wss://ws-api.testnet.binance.vision:9443/ws-api/v3** |
| wss://stream.binance.com/ws<br>wss://stream.binance.com:9443/ws | **wss://stream.testnet.binance.vision/stream**<br>**wss://stream.testnet.binance.vision:9443/stream** |
| wss://stream.binance.com/stream<br>wss://stream.binance.com:9443/stream | **wss://stream.testnet.binance.vision/stream**<br>**wss://stream.testnet.binance.vision:9443/stream** |
| wss://stream-sbe.binance.com/ws<br>wss://stream-sbe.binance.com:9443/ws | **wss://stream-sbe.testnet.binance.vision/ws**<br>**wss://stream-sbe.testnet.binance.vision:9443/ws** |
| wss://stream-sbe.binance.com/stream<br>wss://stream-sbe.binance.com:9443/stream | **wss://stream-sbe.testnet.binance.vision/stream**<br>**wss://stream-sbe.testnet.binance.vision:9443/stream** |
| tcp+tls://fix-oe.binance.com:9000 | **tcp+tls://fix-oe.testnet.binance.vision:9000** |
| tcp+tls://fix-oe.binance.com:9001<br>(Send FIX requests; receive FIX SBE responses) | **tcp+tls://fix-oe.testnet.binance.vision:9001** |
| tcp+tls://fix-oe.binance.com:9002<br>(Send FIX SBE requests; receive FIX SBE responses) | **tcp+tls://fix-oe.testnet.binance.vision:9002** |
| tcp+tls://fix-dc.binance.com:9000 | **tcp+tls://fix-dc.testnet.binance.vision:9000** |
| tcp+tls://fix-dc.binance.com:9001<br>(Send FIX requests; receive FIX SBE responses) | **tcp+tls://fix-dc.testnet.binance.vision:9001** |
| tcp+tls://fix-dc.binance.com:9002<br>(Send FIX SBE requests; receive FIX SBE responses) | **tcp+tls://fix-dc.testnet.binance.vision:9002** |
| tcp+tls://fix-md.binance.com:9000 | **tcp+tls://fix-md.testnet.binance.vision:9000** |
| tcp+tls://fix-md.binance.com:9001<br>(Send FIX requests; receive FIX SBE responses) | **tcp+tls://fix-md.testnet.binance.vision:9001** |
| tcp+tls://fix-md.binance.com:9002<br>(Send FIX SBE requests; receive FIX SBE responses) | **tcp+tls://fix-md.testnet.binance.vision:9002** |

<a id="faq-supported-endpoints"></a>
## Can I use the `/sapi` endpoints on the Spot Test Network?

No, only the `/api` endpoints are available on the Spot Test Network:

* [Market Data Endpoints (REST API)](https://developers.binance.com/docs/binance-spot-api-docs/rest-api/market-data-endpoints)
* [Market Data Requests (WebSocket API)](https://developers.binance.com/docs/binance-spot-api-docs/websocket-api/market-data-requests)
* [WebSocket Market Streams](https://developers.binance.com/docs/binance-spot-api-docs/web-socket-streams)
* [Trading Endpoints (REST API)](https://developers.binance.com/docs/binance-spot-api-docs/rest-api/trading-endpoints)
* [Account Endpoints (REST API)](https://developers.binance.com/docs/binance-spot-api-docs/rest-api/account-endpoints)
* [Trading Requests (WebSocket API)](https://developers.binance.com/docs/binance-spot-api-docs/websocket-api/trading-requests)
* [Account Requests (WebSocket API)](https://developers.binance.com/docs/binance-spot-api-docs/websocket-api/account-requests)
* [User Data Streams](https://developers.binance.com/docs/binance-spot-api-docs/user-data-stream)

<a id="faq-funds-transfer"></a>
## How to get funds in/out of the Spot Test Network?

All users registering on the Spot Test Network automatically receive a balance in many different assets. Please note that these are not real assets and can be used only on the Spot Test Network itself.

All funds on the Spot Test Network are virtual, and can not be transferred in/out of the Spot Test Network.

<a id="faq-restrictions"></a>
## What are the restrictions on the Spot Test Network?

**IP Limits**, **Order Rate Limits**, **Exchange Filters** and **Symbol Filters** on the Spot Test Network are generally the same as on the Spot API.

All users are encouraged to regularly query the API to get the most up-to-date rate limits & filters, for example by doing:

```
curl "https://testnet.binance.vision/api/v3/exchangeInfo"
```

<a id="faq-periodic-reset"></a>
## All my data has disappeared! What happened?

The Spot Test Network is periodically reset to a blank state. That includes all pending and executed orders. During that reset procedure, all users automatically receive a fresh allowance of all assets.

These resets happen approximately **once per month**, and we do not offer prior notification for them.

Starting from August 2020, API Keys are preserved during resets. Users no longer need to re-register new API Keys after a reset.

<a id="ui-klines"></a>
## What is the difference between `klines` and `uiKlines`?

On the Spot Test Network, these 2 requests always return the same data.

<a id="faq-rsa-keys"></a>
## What are RSA API Keys?

RSA API Keys are an alternative to the typical HMAC-SHA-256 API Keys that are used to authenticate your requests on the Spot API.

Unlike HMAC-SHA-256 API Keys where we generate the secret signing key for you, with RSA API Keys, *you* generate a pair of public+private RSA keys, send us the public key, and sign your requests with your private key.

<a id="faq-rsa-types"></a>
## What type of RSA keys are supported?

We support RSA keys of any length from 2048 bits up to 4096 bits. We recommend **2048 bits keys** as a good balance between security and signature speed.

When generating the RSA signature, use the **PKCS#1 v1.5** signature scheme. This is the default when using OpenSSL. We currently do not support the PSS signature scheme.

<a id="faq-rsa-how-to"></a>
## How can I use RSA API Keys?

Step 1: Generate the private key `test-prv-key.pem`. **Do not share this file with anyone!**

```
openssl genrsa -out test-prv-key.pem 2048
```

Step 2: Generate the public key `test-pub-key.pem` from the private key.

```
openssl rsa -in test-prv-key.pem -pubout -outform PEM -out test-pub-key.pem
```

The public key should look something like this:

```
-----BEGIN PUBLIC KEY-----
bL4DUXwR3ijFSXzcecQtVFU1zVWcSQd0Meztl3DLX42l/8EALJx3LSz9YKS0PMQW
MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAv9ij99RAJM4JLl8Rg47b
dJXMrv84WL1OK/gid4hCnxo083LYLXUpIqMmL+O6fmXAvsvkyMyT520Cw0ZNCrUk
WoCjGE4JZZGF4wOkWdF37JFWbDnE/GF5mAykKj+OMaECBlZ207KleQqgVzHjKuCb
hPMuBVVD3IhjBfIc7EEM438LbtayMDx4dviPWwm127jwn8qd9H3kv5JBoDfsdYMB
3k39r724CljqlAfX33GpbV2LvEkL6Da3OFk+grfN98X2pCBRz5+1N95I2cRD7o+j
wtCr+65E+Gqjo4OI60F9Gq5GDcrnudnUw13a4zwlU6W+Cy8gJ4R0CcKTc4+VhYVX
5wW2tzLVnDqvjIN8hjhgtmUv8hr19Wn+42ev+5sNtO5QAS6sJMJG5D+cpxCNhei1
Xm+1zXliaA1fvVYRqon2MdHcedFeAjzVtX38+Xweytowydcq2V/9pUUNZIzUqX7t
Zr3F+Ao3QOb/CuWbUBpUcbXfGv7AI1ozP8LRByyu6O8Z1dZNdkdjWVt83maUrIJH
jjc7jlZY9JbH6EyYV5TenjJaupvdlx72vA7Fcgevx87seog2JALAJqZQNT+t9/tm
rTUSEp3t4aINKUC1QC0CYKECAwEAAQ==
-----END PUBLIC KEY-----
```

Step 3: Register your public key on the Spot Test Network.

During registration, we will generate an API Key for you that you will have to put in the `X-MBX-APIKEY` header of your requests, exactly the same way as you would do for HMAC-SHA-256 API Keys.

Step 4: When you send a request to the Spot Test Network, sign the payload using your private key.

Here is an example Bash script to post a new order and sign the request using OpenSSL. You can adapt it to your favorite programming language:

```bash
#!/usr/bin/env bash

# Set up authentication:
API_KEY="put your own API Key here"
PRIVATE_KEY_PATH="test-prv-key.pem"

# Set up the request:
API_METHOD="POST"
API_CALL="api/v3/order"
API_PARAMS="symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=0.2"

# Sign the request:
timestamp=$(date +%s000)
api_params_with_timestamp="$API_PARAMS&timestamp=$timestamp"
signature=$(echo -n "$api_params_with_timestamp" \
            | openssl dgst -sha256 -sign "$PRIVATE_KEY_PATH" \
            | openssl enc -base64 -A)

# Send the request:
curl -H "X-MBX-APIKEY: $API_KEY" -X "$API_METHOD" \
    "https://testnet.binance.vision/$API_CALL?$api_params_with_timestamp" \
    --data-urlencode "signature=$signature"
```

<a id="faq-ed25519-keys"></a>
## What are Ed25519 API keys?

Ed25519 API keys are an alternative to [RSA API keys](#faq-rsa-keys), using asymmetric cryptography to authenticate your requests on the Spot API.

Like RSA API keys, Ed25519 keys are *asymmetric*: you generate a keypair, share the public key with Binance, and use your private key to sign requests.

<a id="faq-ed25519-vs-rsa"></a>
## Why use Ed25519 instead of RSA API keys?

Ed25519 digital signature scheme provides security comparable to 3072-bit RSA keys, while having much smaller signatures that are faster to compute:

| API key type   | Signature size | Signature operation |
|----------------|----------------|---------------------|
| HMAC-SHA-256   | 64 bytes       | 0.00 ms             |
| Ed25519        | 88 bytes       | 0.03 ms             |
| RSA (2048-bit) | 344 bytes      | 0.55 ms             |
| RSA (4096-bit) | 684 bytes      | 3.42 ms             |

<a id="faq-ed25519-how-to"></a>
## How can I use Ed25519 API keys?

Step 1: Generate the private key `test-prv-key.pem`. **Do not share this file with anyone!**

```
openssl genpkey -algorithm ed25519 -out test-prv-key.pem
```

Step 2: Compute the public key `test-pub-key.pem` from the private key.

```
openssl pkey -pubout -in test-prv-key.pem -out test-pub-key.pem
```

The public key should look something like this:

```
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEACeCSz7VJkh3Bb+NF794hLMU8fLB9Zr+/tGMdVKCC2eo=
-----END PUBLIC KEY-----
```

Step 3: Register your public key on the Spot Test Network.

During registration, we will generate an API key for you. Please put it in the `X-MBX-APIKEY` header of your requests, exactly the same way as with other API key types.

Step 4: When you send a request to the Spot Test Network, sign the payload using your private key.

Here is an example in Python that posts a new order signed with Ed25519 key. You can adapt it to your favorite programming language.

```python
#!/usr/bin/env python3

import base64
import requests
import time
from cryptography.hazmat.primitives.serialization import load_pem_private_key

# Set up authentication
API_KEY='put your own API Key here'
PRIVATE_KEY_PATH='test-prv-key.pem'

# Load the private key.
# In this example the key is expected to be stored without encryption,
# but we recommend using a strong password for improved security.
with open(PRIVATE_KEY_PATH, 'rb') as f:
    private_key = load_pem_private_key(data=f.read(),
                                       password=None)

# Set up the request parameters
params = {
    'symbol':       'BTCUSDT',
    'side':         'SELL',
    'type':         'LIMIT',
    'timeInForce':  'GTC',
    'quantity':     '1.0000000',
    'price':        '0.20',
}

# Timestamp the request
timestamp = int(time.time() * 1000) # UNIX timestamp in milliseconds
params['timestamp'] = timestamp

# Sign the request
payload = '&'.join([f'{param}={value}' for param, value in params.items()])
signature = base64.b64encode(private_key.sign(payload.encode('ASCII')))
params['signature'] = signature

# Send the request
headers = {
    'X-MBX-APIKEY': API_KEY,
}
response = requests.post(
    'https://testnet.binance.vision/api/v3/order',
    headers=headers,
    data=params,
)
print(response.json())
```
