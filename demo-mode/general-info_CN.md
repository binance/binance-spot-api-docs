# 现货模拟交易

本页面说明如何通过 API 使用[模拟交易](https://www.binance.com/zh-CN/support/faq/detail/9be58f73e5e14338809e3b705b9687dd)。

## 如何通过 API 进行模拟交易？

1. 登录您的 Binance 账户后，点击 Binance 模拟交易，然后您可以在[API 密钥管理页面](https://demo.binance.com/en/my/settings/api-management)创建 API 密钥。
2. 按照现货 API 的官方文档操作，将接口/方法的 URL 替换为以下值：

<table>
    <thead>
    <tr>
        <th>服务</th>
        <th>现货 API URL</th>
        <th>模拟交易 URL</th>
    </tr>
    </thead>
    <body>
        <tr>
            <td>
                REST接口
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
                WebSocket行情推送
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
                WebSocket行情推送 (SBE)
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
                （发送 FIX 请求；接收 FIX 响应）
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
               （发送 FIX 请求；接收 FIX SBE 响应）
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
                （发送 FIX SBE 请求；接收 FIX SBE 响应）
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

## SPOT 测试网（Testnet）和 SPOT 模拟交易（Demo Mode）的区别是什么？

| SPOT 测试网（Testnet）                  | SPOT 模拟交易（Demo Mode）                      |
|---------------------------------------|----------------------------------------------|
| 余额每月重置一次。                     | 你可以通过界面随时重置余额。                   |
| 测试网有时会先于正式交易所推出新功能。 | 模拟交易始终拥有与正式交易所相同的功能。       |
| 测试网的价格和订单簿与正式交易所独立。 | 模拟交易的价格和订单簿与正式交易所相似。       |
| IP 限制、未成交订单数量、交易所过滤器通常与正式交易所相同。 | IP 限制、未成交订单数量、交易所过滤器与正式交易所完全相同。 |

**总结**：
* SPOT 测试网适合集成尚未在正式交易所上线的新功能。
* 模拟交易适合基于_类似真实的_市场数据进行测试。

> [!WARNING]
> 类似真实的市场数据不等同于“真实”市场数据。在模拟交易中有效的交易策略在正式交易所未必有效。

## 模拟交易维护期间会发生什么？

* 维护前会在 [更新日志](CHANGELOG_CN.md) 页面发布公告。
* 维护期间，您将无法下单或取消订单。
