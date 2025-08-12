# 挂钩订单（Pegged orders）

**声明**：

* 此术语表只适用于现货 （`SPOT`） 交易所。
* 此处使用的交易对和价格是虚构的，并不反映实际交易所的设置。
* 为简单起见，本文档中的示例不包括佣金。

## 什么是挂钩订单（pegged orders）？

挂钩订单本质上是**限价订单**，其价格来自订单簿。

例如，您可以通过发送“以最佳卖价卖出 1 BTC”这样的订单，而不是使用特定价格（例如，以至少 100,000 USDC 的价格卖出 1 BTC），来将您的订单排在订单簿中最高价的订单之后。或者通过“以 100,000 USDT 或最佳出价买入 1 BTC，立即成交否則取消”这样的订单，以最低价格（且仅以该价格）来挑选卖家。

挂钩订单为做市商提供了一种以最小延迟匹配最佳价格的方法，而散户则可以以最佳价格快速成交，并将滑价可能性降至最低。

挂钩订单也称为“最佳买卖价” （`best bid-offer`）或 BBO 订单。

## 如何下挂钩订单？

请参考以下列表：

<table border="1" cellpadding="5" cellspacing="0">
  <thead>
    <tr>
      <th>API</th>
      <th>请求</th>
      <th>参数</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3">REST API</td>
      <td><code>POST /api/v3/order</code></td>
      <td rowspan="6">
        <p><code>pegPriceType</code>:</p>
        <ul>
          <li><code>PRIMARY</code> — 在订单簿同一方向的最佳价格</li>
          <li><code>MARKET</code> — 在订单簿反方向的最佳价格</li>
        </ul>
        <p>
        <code>pegOffsetType</code> 和 <code>pegOffsetValue PRICE_LEVEL</code> — 抵消现有价格水平，深入订单簿内部</p>
        <p>关于订单列表：（预知详情， 请参考 API 文档。）</p>
        <ul>
          <li>OCO 使用 <code>above*</code> 和 <code>below*</code> 前缀。</li>
          <li>OTO 使用 <code>working*</code> 和 <code>pending*</code> 前缀。</li>
          <li>OTOCO 使用 <code>working*</code>, <code>pendingAbove*</code>， 和 <code>pendingBelow*</code> 前缀。</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td>
        <code>POST /api/v3/orderList/*</code><br>
      </td>
    </tr>
    <tr>
      <td><code>POST /api/v3/cancelReplace</code></td>
    </tr>
    <tr>
      <td rowspan="3">WebSocket API</td>
      <td><code>order.place</code></td>
    </tr>
    <tr>
      <td>
        <code>orderList.place.*</code><br>
      </td>
    </tr>
    <tr>
      <td><code>order.cancelReplace</code></td>
    </tr>
    <tr>
      <td rowspan="3">FIX API</td>
      <td>NewOrderSingle <code>&lt;D&gt;</code></td>
      <td rowspan="3"><code>OrdType=PEGGED</code>, <code>&lt;PegInstructions&gt;</code> 组件块， <code>PeggedPrice</code> 字段。</td>
    </tr>
    <tr>
      <td>NewOrderList <code>&lt;E&gt;</code></td>
    </tr>
      <td>OrderCancelRequestAndNewOrderSingle <code>&lt;XCN&gt;</code></td>
    </tr>
  </tbody>
</table>

目前， [智能指令路由 (SOR)](sor_faq_CN.md) 不支持挂钩订单。

此示例 REST API 响应显示，对于挂钩订单，`peggedPrice` 反映所选中的价格，而 `price` 为原始订单价格（如果未设置，则赋值为零）。

```json
{
  "symbol": "BTCUSDT",
  "orderId": 18,
  "orderListId": -1,
  "clientOrderId": "q1fKs4Y7wgE61WSFMYRFKo",
  "transactTime": 1750313780050,
  "price": "0.00000000",
  "pegPriceType": "PRIMARY_PEG",
  "peggedPrice": "0.04000000",
  "origQty": "1.00000000",
  "executedQty": "0.00000000",
  "origQuoteOrderQty": "0.00000000",
  "cummulativeQuoteQty": "0.00000000",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "workingTime": 1750313780050,
  "fills": [],
  "selfTradePreventionMode": "NONE"
}
```

## 哪些订单类型支持挂钩订单？

此功能支持除`市价单`（`MARKET`）以外的所有订单类型。

由于`止损单`（`STOP_LOSS`）和`获利单`（`TAKE_PROFIT` ）均会在满足止损条件后下达`市价单`（`MARKET`），因此这些订单无法使用挂钩类型。

### 限价单 `Limit orders`）

挂钩限价单立即以当前最佳价格进入市场：

* `LIMIT`
  * 使用 `pegPriceType=PRIMARY_PEG` 时候，只允许使用 `timeInForce=GTC`。
* `LIMIT_MAKER`
  * 只允许使用 `pegPriceType=PRIMARY_PEG`。

### 止损限价单（`Stop-limit orders`）

当价格变动触发止损单（通过止损价或追踪止损）时，挂钩止损限价单以最佳价格进入市场：

* `STOP_LOSS_LIMIT`
* `TAKE_PROFIT_LIMIT`

这意味着，止损单将采用被触发时的最佳价格，这会与止损单下单时的价格不同。并且这类订单只能绑定限价，不能绑定止损价。

### OCO

OCO 订单列表可以使用挂钩指令。

* OCO 中的任何订单均可使用挂钩：上方订单和下方订单，或仅其中之一。
* 挂钩订单在订单簿中下单时，将以最优价格成交：
  * `LIMIT_MAKER` 订单立即以当前最佳价格进入市场
  * `STOP_LOSS_LIMIT` 和 `TAKE_PROFIT_LIMIT` 会在被触发时，以最佳价格进入市场
* `STOP_LOSS` 和 `TAKE_PROFIT` 订单无法使用挂钩类型。

### OTO 和 OTOCO

OTO 订单列表也可以使用挂钩指令。

* OTO 中的任何订单均可使用挂钩：生效订单和待处理订单，或仅其中之一。
* 挂钩生效订单立即以当前最佳价格进入市场。
* 当生效订单完成后，挂钩待处理限价单会以最佳价格进入市场。
* 挂钩待处理止损限价单会在被触发时，以最佳价格进入市场。

OTOCO 订单列表也可以使用挂钩订单, 使用方法与 OTO 和 OCO 订单列表类似。

## 什么交易对支持挂钩订单？

请参阅交易所信息请求并查找 `pegInstructionsAllowed` 字段。如果这个字段设置为 true，那么在该交易上就可以使用挂钩订单。

## 哪些过滤器适用于挂钩订单？

挂钩订单需要通过选定价格的所有适用过滤器：

* `PRICE_FILTER`
* `PERCENT_PRICE` 和 `PERCENT_PRICE_BY_SIDE`
* `NOTIONAL` 和 `MIN_NOTIONAL` （请考虑 `quantity`)

如果挂钩订单指定了 `price`，那么这个订单就必须同时通过 `price` 和 `peggedPrice` 上的验证。

条件单以及 OTO 订单列表中的挂钩待处理订单在触发时会进行（重新）验证，之后可能会被拒绝。
