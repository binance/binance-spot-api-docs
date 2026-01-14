<h3 id="faq-how-to-use">
        How can I use the Spot Test Network?
</h3>

<p>
        Step 1: Log in on this <a href="https://testnet.binance.vision/">website</a>, and generate an API Key.
</p>

<p>
        Step 2: Follow the <a href="https://developers.binance.com/docs/binance-spot-api-docs/CHANGELOG" target="_blank">official documentation of the Spot API</a>,
        replacing the URLs of the endpoints with the following values:
<table>
    <thead>
    <tr>
        <th>Spot API URLs</th>
        <th>Spot Test Network URLs</th>
    </tr>
    </thead>
    <body>
        <tr>
            <td>
                <ul>
                    <li>https://api.binance.com/api</li>
                    <li>https://api-gcp.binance.com/api</li>
                    <li>https://api1.binance.com/api</li>
                    <li>https://api2.binance.com/api</li>
                    <li>https://api3.binance.com/api</li>
                    <li>https://api4.binance.com/api</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>https://testnet.binance.vision/api</strong></li>
                    <li><strong>https://api1.testnet.binance.vision/api</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>wss://ws-api.binance.com/ws-api/v3</li>
                    <li>wss://ws-api.binance.com:9443/ws-api/v3</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>wss://ws-api.testnet.binance.vision/ws-api/v3</strong></li>
                    <li><strong>wss://ws-api.testnet.binance.vision:9443/ws-api/v3</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>wss://stream.binance.com/ws</li>
                    <li>wss://stream.binance.com:9443/ws</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>wss://stream.testnet.binance.vision/stream</strong></li>
                    <li><strong>wss://stream.testnet.binance.vision:9443/stream</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>wss://stream.binance.com/stream</li>
                    <li>wss://stream.binance.com:9443/stream</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>wss://stream.testnet.binance.vision/stream</strong></li>
                    <li><strong>wss://stream.testnet.binance.vision:9443/stream</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>wss://stream-sbe.binance.com/ws</li>
                    <li>wss://stream-sbe.binance.com:9443/ws</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>wss://stream-sbe.testnet.binance.vision/ws</strong></li>
                    <li><strong>wss://stream-sbe.testnet.binance.vision:9443/ws</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>wss://stream-sbe.binance.com/stream</li>
                    <li>wss://stream-sbe.binance.com:9443/stream</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>wss://stream-sbe.testnet.binance.vision/stream</strong></li>
                    <li><strong>wss://stream-sbe.testnet.binance.vision:9443/stream</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>tcp+tls://fix-oe.binance.com:9000</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>tcp+tls://fix-oe.testnet.binance.vision:9000</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>tcp+tls://fix-dc.binance.com:9000</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>tcp+tls://fix-dc.testnet.binance.vision:9000</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>tcp+tls://fix-md.binance.com:9000</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>tcp+tls://fix-md.testnet.binance.vision:9000</strong></li>
                </ul>
            </td>
        </tr>
        </body>
        </table>
</p>

<hr>

<h3 id="faq-supported-endpoints">
        Can I use the <code>/sapi</code> endpoints on the Spot Test Network?
</h3>

<p>
        No, only the <code>/api</code> endpoints are available on the Spot Test Network:
    <ul>
            <li><a href="https://developers.binance.com/docs/binance-spot-api-docs/rest-api/market-data-endpoints" target="_blank">Market Data Endpoints (REST API)</a></li>
            <li><a href="https://developers.binance.com/docs/binance-spot-api-docs/websocket-api/market-data-requests" target="_blank">Market Data Requests (WebSocket API)</a></li>
            <li><a href="https://developers.binance.com/docs/binance-spot-api-docs/web-socket-streams" target="_blank">Websocket Market Streams</a></li>
            <li><a href="https://developers.binance.com/docs/binance-spot-api-docs/rest-api/trading-endpoints" target="_blank">Trading Endpoints (REST API)</a></li>
            <li><a href="https://developers.binance.com/docs/binance-spot-api-docs/rest-api/account-endpoints" target="_blank">Account Endpoints (REST API)</a></li>
            <li><a href="https://developers.binance.com/docs/binance-spot-api-docs/websocket-api/trading-requests" target="_blank">Trading Requests (WebSocket API)</a></li>
            <li><a href="https://developers.binance.com/docs/binance-spot-api-docs/websocket-api/account-requests" target="_blank">Account Requests (WebSocket API)</a></li>
            <li><a href="https://developers.binance.com/docs/binance-spot-api-docs/user-data-stream" target="_blank">User Data Streams</a></li>
        </ul>
</p>

<hr>

<h3 id="faq-funds-transfer">
        How to get funds in/out of the Spot Test Network?
</h3>

<p>
        All users registering on the Spot Test Network automatically receive a balance in many different assets.
        Please note that these are not real assets and can be used only on the Spot Test Network itself.
</p>

<p>
        All funds on the Spot Test Network are virtual, and can not be transferred in/out of the Spot Test Network.
</p>

<hr>

<h3 id="faq-restrictions">
        What are the restrictions on the Spot Test Network?
</h3>

<p>
        <strong>IP Limits</strong>, <strong>Order Rate Limits</strong>, <strong>Exchange Filters</strong> and <strong>Symbol Filters</strong>
        on the Spot Test Network are generally the same as on the Spot API.
</p>

<p>
        All users are encouraged to regularly query the API to get the most up-to-date rate limits &amp; filters, for
        example by doing:
</p>

<pre>curl "https://testnet.binance.vision/api/v3/exchangeInfo"</pre>

<hr>

<h3 id="faq-periodic-reset">
        All my data has disappeared! What happened?
</h3>

<p>
        The Spot Test Network is periodically reset to a blank state. That includes all pending and executed orders.
        During that reset procedure, all users automatically receive a fresh allowance of all assets.
</p>

<p>
        These resets happen approximately <strong>once per month</strong>, and we do not offer prior notification
        for them.
</p>

<p>
        Starting from August 2020, API Keys are preserved during resets. Users no longer need to re-register new API
        Keys after a reset.
</p>

<hr>

<h3 id="ui-klines">
        What is the difference between <code>klines</code> and <code>uiKlines</code>?
</h3>

<p>
        On the Spot Test Network, these 2 requests always return the same data.
</p>

<hr>

<h3 id="faq-rsa-keys">
        What are RSA API Keys?
</h3>

<p>
        RSA API Keys are an alternative to the typical HMAC-SHA-256 API Keys that are used to authenticate your requests
        on the Spot API.
</p>

<p>
        Unlike HMAC-SHA-256 API Keys where we generate the secret signing key for you, with RSA API Keys, *you* generate a
        pair of public+private RSA keys, send us the public key, and sign your requests with your private key.
</p>

<hr>

<h3 id="faq-rsa-types">
        What type of RSA keys are supported?
</h3>

<p>
        We support RSA keys of any length from 2048 bits up to 4096 bits. We recommend <strong>2048 bits keys</strong>
        as a good balance between security and signature speed.
</p>

<p>
        When generating the RSA signature, use the <strong>PKCS#1 v1.5</strong> signature scheme. This is the default when
        using OpenSSL. We currently do not support the PSS signature scheme.
</p>

<hr>

<h3 id="faq-rsa-how-to">
        How can I use RSA API Keys?
</h3>

<p>
        Step 1: Generate the private key <code>test-prv-key.pem</code>.
        <strong>Do not share this file with anyone!</strong>
</p>

<pre>openssl genrsa -out test-prv-key.pem 2048</pre>

<p>
        Step 2: Generate the public key <code>test-pub-key.pem</code> from the private key.
</p>

<pre>openssl rsa -in test-prv-key.pem -pubout -outform PEM -out test-pub-key.pem</pre>

<p>
        The public key should look something like this:
</p>

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

<p>
        Step 3:
        Register your public key
        on the Spot Test Network.
</p>

<p>
        During registration, we will generate an API Key for you that you will have to put in the <code>X-MBX-APIKEY</code>
        header of your requests, exactly the same way as you would do for HMAC-SHA-256 API Keys.
</p>

<p>
        Step 4: When you send a request to the Spot Test Network, sign the payload using your private key.
</p>

<p>
        Here is an example Bash script to post a new order and sign the request using OpenSSL. You can adapt it to your
        favorite programming language:
</p>

````bash
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
````
<hr>

<h3 id="faq-ed25519-keys">
        What are Ed25519 API keys?
</h3>

<p>
        Ed25519 API keys are an alternative to <a href="#faq-rsa-keys">RSA API keys</a>,
        using asymmetric cryptography to authenticate your requests on the Spot API.
</p>

<p>
        Like RSA API keys, Ed25519 keys are <em>asymmetric</em>:
        you generate a keypair, share the public key with Binance, and use your private key to sign requests.
</p>

<hr>

<h3 id="faq-ed25519-vs-rsa">
        Why use Ed25519 instead of RSA API keys?
</h3>

<p>
        Ed25519 digital signature scheme provides security comparable to 3072-bit RSA keys,
        while having much smaller signatures that are faster to compute:
</p>

<table>
    <thead>
    <tr>
        <th>API key type</th>
        <th>Signature size</th>
        <th>Signature operation</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>HMAC-SHA-256</td>
        <td>64 bytes</td>
        <td>0.00 ms</td>
    </tr>
    <tr>
        <td>Ed25519</td>
        <td>88 bytes</td>
        <td>0.03 ms</td>
    </tr>
    <tr>
        <td>RSA (2048-bit)</td>
        <td>344 bytes</td>
        <td>0.55 ms</td>
    </tr>
    <tr>
        <td>RSA (4096-bit)</td>
        <td>684 bytes</td>
        <td>3.42 ms</td>
    </tr>
    </tbody>
</table>

<hr>

<h3 id="faq-ed25519-how-to">
        How can I use Ed25519 API keys?
</h3>

<p>
        Step 1: Generate the private key <code>test-prv-key.pem</code>.
        <strong>Do not share this file with anyone!</strong>
</p>

<pre>openssl genpkey -algorithm ed25519 -out test-prv-key.pem</pre>

<p>
        Step 2: Compute the public key <code>test-pub-key.pem</code> from the private key.
</p>

<pre>openssl pkey -pubout -in test-prv-key.pem -out test-pub-key.pem</pre>

<p>
        The public key should look something like this:
</p>

```
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEACeCSz7VJkh3Bb+NF794hLMU8fLB9Zr+/tGMdVKCC2eo=
-----END PUBLIC KEY-----
```

<p>
        Step 3:
        Register your public key
        on the Spot Test Network.
</p>

<p>
        During registration, we will generate an API key for you.
        Please put it in the <code>X-MBX-APIKEY</code> header of your requests,
        exactly the same way as with other API key types.
</p>

<p>
        Step 4: When you send a request to the Spot Test Network, sign the payload using your private key.
</p>

<p>
        Here is an example in Python that posts a new order signed with Ed25519 key.
        You can adapt it to your favorite programming language.
</p>

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
</div>
