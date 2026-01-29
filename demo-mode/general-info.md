# Demo Mode for SPOT Trading

This page explains how to use [Demo Mode Trading](https://www.binance.com/en/support/faq/detail/9be58f73e5e14338809e3b705b9687dd) via the API.

## How can I trade on Demo Mode using the API?

1. After logging into your Binance account, click on Binance Demo Trading and then you can create an API key in the [API Key Management page](https://demo.binance.com/en/my/settings/api-management).
2. Follow the official documentation of the SPOT API, replacing the URLs of the endpoints/methods with the following values:

<table>
    <thead>
    <tr>
        <th>Service</th>
        <th>Spot API URLs</th>
        <th>Demo Mode URLs</th>
    </tr>
    </thead>
    <body>
        <tr>
            <td>
                REST API
            </td>
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
                    <li><strong>https://demo-api.binance.com/api</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                WebSocket API
            </td>
            <td>
                <ul>
                    <li>wss://ws-api.binance.com/ws-api/v3</li>
                    <li>wss://ws-api.binance.com:9443/ws-api/v3</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>wss://demo-ws-api.binance.com/ws-api/v3</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td rowspan="2">
                WebSocket Market Streams
            </td>
            <td>
                <ul>
                    <li>wss://stream.binance.com/ws</li>
                    <li>wss://stream.binance.com:9443/ws</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>wss://demo-stream.binance.com/ws</strong></li>
                    <li><strong>wss://demo-stream.binance.com:9443/ws</strong></li>
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
                    <li><strong>wss://demo-stream.binance.com/stream</strong></li>
                    <li><strong>wss://demo-stream.binance.com:9443/stream</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td rowspan="2">
                WebSocket Market Streams (SBE)
            </td>
            <td>
                <ul>
                    <li>wss://stream-sbe.binance.com/ws</li>
                    <li>wss://stream-sbe.binance.com:9443/ws</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>wss://demo-stream-sbe.binance.com/ws</strong></li>
                    <li><strong>wss://demo-stream-sbe.binance.com:9443/ws</strong></li>
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
                    <li><strong>wss://demo-stream-sbe.binance.com/stream</strong></li>
                    <li><strong>wss://demo-stream-sbe.binance.com:9443/stream</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td rowspan="3">
                FIX <br>
                (Send FIX requests; receive FIX responses)
            </td>
            <td>
                <ul>
                    <li>tcp+tls://fix-oe.binance.com:9000</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>tcp+tls://demo-fix-oe.binance.com:9000</strong></li>
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
                    <li><strong>tcp+tls://demo-fix-dc.binance.com:9000</strong></li>
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
                    <li><strong>tcp+tls://demo-fix-md.binance.com:9000</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td rowspan="3">
                FIX SBE <br>
               (Send FIX requests; receive FIX SBE responses)
            </td>
            <td>
                <ul>
                    <li>tcp+tls://fix-oe.binance.com:9001</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>tcp+tls://demo-fix-oe.binance.com:9001</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>tcp+tls://fix-dc.binance.com:9001</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>tcp+tls://demo-fix-dc.binance.com:9001</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>tcp+tls://fix-md.binance.com:9001</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>tcp+tls://demo-fix-md.binance.com:9001</strong></li>
                </ul>
            </td>
        </tr>
            <td rowspan="3">
                FIX SBE <br>
                (Send FIX SBE requests; receive FIX SBE responses)
            </td>
            <td>
                <ul>
                    <li>tcp+tls://fix-oe.binance.com:9002</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>tcp+tls://demo-fix-oe.binance.com:9002</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>tcp+tls://fix-dc.binance.com:9002</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>tcp+tls://demo-fix-dc.binance.com:9002</strong></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>tcp+tls://fix-md.binance.com:9002</li>
                </ul>
            </td>
            <td>
                <ul>
                    <li><strong>tcp+tls://demo-fix-md.binance.com:9002</strong></li>
                </ul>
            </td>
        </tr>
        </body>
        </table>
</p>

## What is the difference between SPOT Testnet and SPOT Demo Mode?

|SPOT Testnet| SPOT Demo Mode|
|----        |----|
|Balances are reset every month.| You can reset your balance whenever you want via the UI.|
|SPOT Testnet sometimes has new features before the live exchange. |Demo Mode always has the same features as the live exchange.|
|Testnet's prices and order books are independent from the live exchange.| Demo Mode's prices and order books are similar to the live exchange.|
|IP Limits, Unfilled Order Count, Exchange Filters are generally the same as the live exchange.|IP Limits, Unfilled Order Count, Exchange Filters are exactly the same as the live exchange.|

**In summary**:
* SPOT Testnet is useful to integrate with upcoming features not yet available on the live exchange.
* Demo Mode is useful to test against _realistic_ market data.

> [!WARNING]
> Realistic market data is not equal to "real" market data. Do not assume trading strategies that work in Demo Mode will work in the live exchange.

## What happens when Demo Mode is under maintenance?

* There will be an announcement on the [CHANGELOG](CHANGELOG.md) page prior to downtime.
* During maintenance, you will not be able to place or cancel orders.
