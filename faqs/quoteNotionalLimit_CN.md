<a id="using-quotenotionallimit"></a>

# 使用报价资产名义价值上限（`quoteNotionalLimit`）

**免责声明:**

- 此说明仅适用于现货交易所。
- 此处使用的交易对和数值均为虚构，不代表实际交易所的配置。
- 为简单起见，本文档中的示例不包含佣金。

<a id="why-use-quotenotionallimit"></a>

## 为什么使用报价资产名义价值上限 `quoteNotionalLimit`?

现有的订单类型存在一些限制。

`MARKET` 买单的含义是“以订单簿提供的任何价格买入 `quantity` X”。如果流动性不足，订单可能会吃掉多档订单簿，花费比预期更多的报价资产。

`LIMIT` 买单更可预测，但余额使用效率较低：账户必须按照整个订单都挂在订单簿上的最坏情况锁定 `price * quantity`。如果您对订单簿有准确的判断并能估算滑点，那么锁定的名义价值可能会大于您实际打算花费的金额。

`quoteNotionalLimit` 是买单上的一个可选参数，用于限制订单将消耗的报价资产。订单在下单时锁定名义价值上限，而不是根据订单簿或 `price * quantity` 计算锁定金额；一旦名义价值上限用完，即使还有剩余的 `quantity`，订单也会停止执行。

与 `quoteOrderQty`（由撮合引擎为您计算数量）不同，`quoteNotionalLimit` 保留请求的 `quantity`，并将名义价值上限视为严格的花费上限。这两个参数不能同时使用。

<a id="how-do-i-send-an-order-with-a-notional-limit"></a>

## 如何发送带有名义价值上限的订单?

名义价值上限是以报价资产计价的 `DECIMAL` 金额。提供时必须大于零，并且仅适用于买单（即 `BUY` 方向的订单）。

例如，通过 REST 发送 `quoteNotionalLimit=0` 会返回:

```json
{
  "code": -1102,
  "msg": "Mandatory parameter 'quoteNotionalLimit' was not sent, was empty/null, or malformed."
}
```

名义价值上限对撮合引擎而言过大时会被拒绝，并返回 `-1013`:

```json
{
  "code": -1013,
  "msg": "Quote notional limit is over the symbol's maximum notional."
}
```

这与账户资金不足以支付名义价值上限的情况不同。订单在下单时会锁定名义价值上限，因此如果账户没有足够的可用报价资产，订单会改为被拒绝并返回常规的余额不足错误。例如，可用余额为 `1,000 USDT` 的账户发送一个 `quoteNotionalLimit=5000` 的 `BTCUSDT` 订单，会收到:

```json
{
  "code": -2010,
  "msg": "Account has insufficient balance for requested action."
}
```

订单列表接口会在参数名称前添加前缀，以标识该参数适用于哪个订单:

- OCO: `aboveQuoteNotionalLimit`, `belowQuoteNotionalLimit`
- OTO: `workingQuoteNotionalLimit`, `pendingQuoteNotionalLimit`
- OTOCO: `workingQuoteNotionalLimit`, `pendingAboveQuoteNotionalLimit`, `pendingBelowQuoteNotionalLimit`
- OPO, OPOCO: `workingQuoteNotionalLimit`（待处理订单只能是卖单，不能设置名义价值上限）

请参考下表:

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
      <td><code>POST /api/v3/order</code>, <code>POST /api/v3/order/test</code></td>
      <td rowspan="6">单一订单接口使用 <code>quoteNotionalLimit</code>；订单列表使用带 <code>above*</code>/<code>below*</code>、<code>working*</code>/<code>pending*</code> 前缀的变体（见上方列表）。</td>
    </tr>
    <tr>
      <td><code>POST /api/v3/orderList/oco</code>, <code>/oto</code>, <code>/otoco</code>, <code>/opo</code>, <code>/opoco</code></td>
    </tr>
    <tr>
      <td><code>POST /api/v3/order/cancelReplace</code></td>
    </tr>
    <tr>
      <td rowspan="3">WebSocket API</td>
      <td><code>order.place</code>, <code>order.test</code></td>
    </tr>
    <tr>
      <td><code>orderList.place.oco</code>, <code>.oto</code>, <code>.otoco</code>, <code>.opo</code>, <code>.opoco</code></td>
    </tr>
    <tr>
      <td><code>order.cancelReplace</code></td>
    </tr>
    <tr>
      <td rowspan="3">FIX API</td>
      <td>NewOrderSingle <code>&lt;D&gt;</code></td>
      <td rowspan="3">自定义标签 <code>25057</code> <code>QuoteNotionalLimit</code>。适用与 REST/WebSocket 相同的订单类型规则。</td>
    </tr>
    <tr>
      <td>NewOrderList <code>&lt;E&gt;</code></td>
    </tr>
    <tr>
      <td>OrderCancelRequestAndNewOrderSingle <code>&lt;XCN&gt;</code></td>
    </tr>
  </tbody>
</table>

> [!NOTE]
>
> 旧版 OCO 接口（`POST /api/v3/order/oco`、WebSocket `orderList.place`）不接受这些新参数。
> `POST /api/v3/order/test` 和 `newOrderRespType=ACK` 下单也不会显示 `quoteNotionalLimit`，因为这些形式不包含订单属性。
>

[智能订单路由 (SOR)](sor_faq_CN.md) 不支持 `quoteNotionalLimit`：SOR 接口会拒绝带有此参数的请求，并返回 `-1106`。

以下 REST API 响应示例展示了一个 `LIMIT` 买单上的 `quoteNotionalLimit`：以 `30` 的价格买入 `10`，`quoteNotionalLimit=250`。将全部 `10` 挂在订单簿上需要 `10 * 30 = 300`，超过了名义价值上限，因此如果下单时没有任何成交，订单将过期而不会挂单。此处订单在下单时以 `20` 的价格成交了 `5`，花费 `100`。挂出剩余的 `5` 需要 `5 * 30 = 150`，而 `100 + 150 = 250` 没有超过名义价值上限，因此剩余的 `5` 挂在订单簿上:

```json
{
  "symbol": "BTCUSDT",
  "orderId": 42,
  "orderListId": -1,
  "clientOrderId": "P8hJx3kN9rT2vL6qY4wB1z",
  "transactTime": 1750313780050,
  "price": "30.00000000",
  "origQty": "10.00000000",
  "executedQty": "5.00000000",
  "origQuoteOrderQty": "0.00000000",
  "quoteNotionalLimit": "250.00000000",
  "cummulativeQuoteQty": "100.00000000",
  "status": "PARTIALLY_FILLED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "workingTime": 1750313780050,
  "fills": [
    {
      "price": "20.00000000",
      "qty": "5.00000000",
      "commission": "0.00000000",
      "commissionAsset": "BTC",
      "tradeId": 7
    }
  ],
  "selfTradePreventionMode": "NONE"
}
```

请注意，只有当订单下单时带有该参数，响应中才会出现 `quoteNotionalLimit`。

它会出现在下单（`RESULT`/`FULL`）、订单状态、当前挂单、所有订单、撤单、撤消挂单再下单和保留优先级的修改订单的响应中，以及订单列表的 `orderReports` 条目中。

<a id="example-market-buy"></a>

### 市价买单示例

发送一个签名的 REST `POST /api/v3/order` 请求，带上以下参数，以及常规的 `timestamp` 和 `signature`。使用 `X-MBX-TIME-UNIT: MICROSECOND` 以获得下方所示的时间戳单位。

| 参数                 | 值                 |
| -------------------- | ------------------ |
| `symbol`             | `BTCUSDT`          |
| `side`               | `BUY`              |
| `type`               | `MARKET`           |
| `quantity`           | `1`                |
| `quoteNotionalLimit` | `60000`            |
| `newOrderRespType`   | `FULL`             |
| `newClientOrderId`   | `notional_example` |

如果订单簿上的最优卖单是价格为 `100,000 USDT`、数量为 `1 BTC` 的卖单，该订单买入 `0.6 BTC`，花费 `60,000 USDT`，并因剩余的 `0.4 BTC` 会超过名义价值上限而过期。

响应:

```json
{
  "symbol": "BTCUSDT",
  "orderId": 1,
  "orderListId": -1,
  "clientOrderId": "notional_example",
  "transactTime": 1792059013310530,
  "price": "0.0000",
  "origQty": "1.0000",
  "executedQty": "0.6000",
  "origQuoteOrderQty": "0.0000",
  "quoteNotionalLimit": "60000.0000",
  "cummulativeQuoteQty": "60000.0000",
  "status": "EXPIRED",
  "expiryReason": "INSUFFICIENT_QUOTE_NOTIONAL_LIMIT",
  "timeInForce": "GTC",
  "type": "MARKET",
  "side": "BUY",
  "workingTime": 1792059013310530,
  "fills": [
    {
      "price": "100000.0000",
      "qty": "0.6000",
      "commission": "0.0000",
      "commissionAsset": "BTC",
      "tradeId": 0
    }
  ],
  "selfTradePreventionMode": "NONE"
}
```

`origQty` 保持为请求的数量。`executedQty` 和 `cummulativeQuoteQty` 显示订单实际买入和花费的数量；`quoteNotionalLimit` 保留提供的上限。

<a id="what-order-types-support-a-notional-limit"></a>

## 哪些订单类型支持名义价值上限?

只有买单接受 `quoteNotionalLimit`。SELL 方向的订单会拒绝该参数，并返回 `-1013` "Quote notional limit is not allowed for this order."

<a id="market-orders"></a>

### 市价单

- `MARKET`, `STOP_LOSS`, `TAKE_PROFIT`
- 当交易对的 `marketQuoteNotionalLimitAllowed` 为 `true` 时支持。该字段显示在交易规范信息响应中（REST API `GET /api/v3/exchangeInfo`、WebSocket API `exchangeInfo`）。
- 订单在下单时锁定提供的名义价值上限，而不是根据订单簿估算锁定金额。
- `STOP_LOSS` 和 `TAKE_PROFIT` 在等待阶段和订单触发时都保留名义价值上限。触发不需要额外资金，因此永远不会因资金不足而失败。
- 不能与 `quoteOrderQty` 同时使用：同时提供会被拒绝，并返回 `-1106`。

<a id="limit-orders"></a>

### 限价单

- `LIMIT`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT_LIMIT`
- 当交易对的 `limitQuoteNotionalLimitAllowed` 为 `true` 时支持。该字段显示在交易规范信息响应中（REST API `GET /api/v3/exchangeInfo`、WebSocket API `exchangeInfo`）。
- 订单锁定提供的名义价值上限，而不是 `price * quantity`。名义价值上限必须严格小于 `price * quantity`；否则请求会被拒绝，并返回 `-1013` "Quote notional limit must be less than the notional of the order at its limit price."
- `LIMIT_MAKER` 订单不能设置名义价值上限，因为这会与其挂单的名义价值相矛盾。这同样适用于任何可能出现 `LIMIT_MAKER` 的地方，包括订单列表。
- 挂钩 `LIMIT` 订单不受"名义价值上限低于 `price * quantity`"规则的限制，锁定名义价值上限与 `quantity * peg price` 中较小的一个。

<a id="order-lists"></a>

### 订单列表

- `OCO`：每个条件订单都有各自可选的名义价值上限，使用上述订单特定的前缀。`LIMIT_MAKER` 订单仍然不能设置。
- `OTO`, `OTOCO`：生效的 `LIMIT` 订单和任何符合条件的待处理买单（单个订单或 OCO）都可以设置名义价值上限。如果生效订单以 `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT` 过期，待处理订单将以 `OTO_PHASE_ONE_EXPIRED` 过期，且永远不会被下单。
- `OPO`, `OPOCO`：只有生效的 `LIMIT` 订单可以设置名义价值上限。待处理订单只能是卖单，不能设置。

<a id="what-happens-when-the-notional-limit-is-reached"></a>

## 达到名义价值上限时会发生什么?

每次成交前，撮合引擎会将已花费的名义价值加上候选成交与名义价值上限进行比较。如果某笔成交会超过名义价值上限，撮合引擎会将该笔交易成交的数量缩减为不超过上限，且为 `LOT_SIZE` 步进间隔的整数倍的报价金额对应的最大数量。任何未成交的数量都会以新的过期原因 `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT` 过期（参见[枚举](../enums_CN.md#expiry-reasons)）。如果名义价值上限不足以支付一个 `LOT_SIZE` 步进间隔，则不会发生交易，挂单者(maker)保留其数量和优先级。

对于市价单，如果吃单后订单簿上没有剩余的卖单，即使最后一笔成交用完了全部名义价值上限，过期原因也会是 `INSUFFICIENT_LIQUIDITY`。

对于 `GTC` `LIMIT` 买单，只有当 `spent + remaining quantity * price` 仍在名义价值上限内时，剩余部分才会挂在订单簿上。否则，即使是订单簿（而非名义价值上限）结束了吃单，剩余部分也会以 `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT` 过期。

对于 `IOC`，吃单后任何剩余数量都会过期。当名义价值上限停止了吃单，且订单簿上仍有价格等于或低于限价的卖单时，过期原因为 `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT`。否则，即使最后一笔成交用完了全部名义价值上限，过期原因也是 `UNFILLED_IOC_QUANTITY_EXPIRED`。

对于 `FOK`，撮合引擎首先检查全部数量是否在名义价值上限内。如果不在，订单将以 `UNFILLED_FOK_ORDER_EXPIRED` 过期，且没有任何成交。

[自我交易预防 (STP)](stp_faq_CN.md) 对名义价值上限的影响取决于应用于被阻止撮合的 STP 模式:

- `TRANSFER`：被阻止数量的名义价值被转移，并像交易一样计入名义价值上限。它不包含在 `cummulativeQuoteQty` 中，后者只反映交易。在上述 `GTC` 挂单检查中，转移的名义价值计为已花费。
- `DECREMENT`：被阻止的数量会减少订单的剩余数量，但不计入名义价值上限。当订单指定了 `TRANSFER` 但实际应用的是 `DECREMENT` 时（例如两个订单来自同一账户），同样适用。
- `EXPIRE_TAKER`, `EXPIRE_MAKER`, `EXPIRE_BOTH`：行为与没有名义价值上限的订单相同。

<a id="example-buy-10-market-with-quotenotionallimit90-book-ask-3-at-30-then-5-at-40"></a>

### 示例：`MARKET` 买单 `10`，`quoteNotionalLimit=90`，订单簿卖单依次为 `30` 价格的 `3` 和 `40` 价格的 `5`

用户数据流 `executionReport` 事件:

| 事件 | `x`       | `X`                | `l` | `L` | `z` | `Z` | `eR`                                |
| ---- | --------- | ------------------ | --- | --- | --- | --- | ----------------------------------- |
| 1    | `NEW`     | `NEW`              | 0   | 0   | 0   | 0   | 无                                  |
| 2    | `TRADE`   | `PARTIALLY_FILLED` | 3   | 30  | 3   | 90  | 无                                  |
| 3    | `EXPIRED` | `EXPIRED`          | 0   | 0   | 3   | 90  | `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT` |

每个事件都保留 `q=10`、`Qq=90` 和 `Q=0`。报价资产锁定在下单时为 `90`，过期后为 `0`。全部名义价值上限已用完。挂单者(maker)的交易事件在过期事件之前。如果订单恰好在名义价值上限处完全成交，则只有一个 `FILLED` 事件，没有过期事件。

<a id="example-buy-10-at-30-gtc-with-quotenotionallimit250"></a>

### 示例：`GTC` 买单，以 `30` 的价格买入 `10`，`quoteNotionalLimit=250`

| 下单时成交        | 已花费 | 剩余 | 已花费 + 剩余 \* 限价 | 结果                                                               |
| ----------------- | ------ | ---- | --------------------- | ------------------------------------------------------------------ |
| 无                | 0      | 10   | 300                   | 以 `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT` 过期；释放 `250`            |
| 以 `20` 成交 `2`  | 40     | 8    | 280                   | 剩余的 `8` 以 `INSUFFICIENT_QUOTE_NOTIONAL_LIMIT` 过期；释放 `210` |
| 以 `20` 成交 `5`  | 100    | 5    | 250                   | 挂出 `5`，锁定 `150`；无需释放                                     |
| 以 `20` 成交 `8`  | 160    | 2    | 220                   | 挂出 `2`，锁定 `60`；释放 `30`                                     |
| 以 `20` 成交 `10` | 200    | 0    | 200                   | `FILLED`；释放 `50`                                                |

在"以 `20` 成交 `2`"的情况下，事件依次为 `NEW`、`z=2`/`Z=40` 的 `TRADE`，然后是总计相同且 `eR=INSUFFICIENT_QUOTE_NOTIONAL_LIMIT` 的 `EXPIRED`。剩余部分永远不会挂在订单簿上，也不会出现在 `openOrders` 中。

<a id="which-symbols-allow-orders-with-a-notional-limit"></a>

## 哪些交易对允许名义价值上限?

请参考交易规范信息响应（REST API `GET /api/v3/exchangeInfo`、WebSocket API
`exchangeInfo`），并查看两个新的交易对字段:

- `marketQuoteNotionalLimitAllowed` — 如果为 `true`，`MARKET`、`STOP_LOSS` 和 `TAKE_PROFIT` 买单可以设置名义价值上限。
- `limitQuoteNotionalLimitAllowed` — 如果为 `true`，`LIMIT`、`STOP_LOSS_LIMIT` 和 `TAKE_PROFIT_LIMIT` 买单可以设置名义价值上限。

例如，以下响应允许市价和限价订单类型设置名义价值上限。

```json
{
  "symbol": "BTCUSDT",
  "orderTypes": [
    "LIMIT",
    "LIMIT_MAKER",
    "MARKET",
    "STOP_LOSS",
    "STOP_LOSS_LIMIT",
    "TAKE_PROFIT",
    "TAKE_PROFIT_LIMIT"
  ],
  "marketQuoteNotionalLimitAllowed": true,
  "limitQuoteNotionalLimitAllowed": true
}
```

这两个字段彼此独立，也与 `quoteOrderQtyMarketAllowed` 无关。当对应的字段为 `false`
时，发送该参数会被拒绝，并返回 `-1013`:

```json
{
  "code": -1013,
  "msg": "Quote notional limit is not supported for this order type on this symbol."
}
```

<a id="which-filters-are-applicable-to-orders-with-a-notional-limit"></a>

## 哪些过滤器适用于带有名义价值上限的订单?

带有名义价值上限的订单与没有名义价值上限的订单使用相同的过滤器检查，但有以下调整:

- `MIN_NOTIONAL`、`NOTIONAL`（包括冰山订单分片检查）和报价资产 `MAX_ASSET` 过滤器使用名义价值上限与计算出的名义价值中较小的一个。
- `MARKET_LOT_SIZE`、`LOT_SIZE`、`PRICE_FILTER` 和 `MAX_POSITION` 检查保持不变。`MAX_POSITION` 在下单时根据提供的 `quantity` 检查（与按报价资产数量下的市价单不同，后者会延后检查）。
- 名义价值上限本身除了报价资产的精度外，不受任何步进间隔或精度过滤器的检查。
- 对于在没有最新价格的交易对上带有名义价值上限的 `MARKET` 买单，启用 `applyToMarket` 的 `MIN_NOTIONAL` 和启用 `applyMinToMarket` 的 `NOTIONAL` 仍会拒绝该订单，因为名义价值上限只是一个上限，交易仍可能低于最小值。最大值方向的市价单过滤器（启用 `applyMaxToMarket` 的 `NOTIONAL` 和报价资产 `MAX_ASSET`）会将名义价值上限与最大值进行比较。
