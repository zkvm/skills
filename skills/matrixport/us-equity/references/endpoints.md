# Matrixport US Equity — Endpoint Reference

All endpoints require authentication headers. See `authentication.md` for signing details.

## Response Envelope

Every response wraps data in a standard envelope:

```json
{"code": 0, "data": { ... }}
```

- `code: 0` — success
- `code: <non-zero>` — error; check for an accompanying `message` field

---

## POST /v1/place_order

Place a new order (limit or market) for a US equity.

**Requires CONFIRM from user before proceeding.**

### Request Body (JSON)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `symbol` | string | Yes | Equity symbol in `<TICKER>.<MARKET>` format, e.g. `AAPL.US` |
| `side` | string | Yes | `Buy` or `Sell` |
| `order_type` | string | No | `LO` (limit order) or `MO` (market order). Defaults to `LO` if omitted |
| `price` | string | Conditional | Limit price as a decimal string, e.g. `"89.0"`. Required for `LO`, omit for `MO` |
| `qty` | string | Yes | Order quantity as a decimal string, e.g. `"10"` |
| `remark` | string | No | Optional order label / note |

### Response

| Field | Type | Description |
|-------|------|-------------|
| `order_id` | string | Assigned order ID |

### Example

```bash
curl -X POST 'https://mapi.matrixport.com/skopenapi/v1/place_order' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2' \
  -H 'Content-Type: application/json' \
  -d '{"symbol":"AAPL.US","side":"Buy","price":"89.0","qty":"10","remark":"test123"}'
```

```json
{"code": 0, "data": {"order_id": "1217311455238426624"}}
```

### Signing note (POST)

```
prehash = "{timestamp}POST/skopenapi/v1/place_order&{json_body}"
```

> **Important:** The `json_body` used to compute the signature must be byte-for-byte identical to the body sent in the HTTP request. Construct the JSON string once (e.g. `body = json.dumps(params)`) and use that same string for both signing and the request body. Do not re-serialize.

---

## GET /v1/orders

Query the status and details of a single order.

### Query Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order_id` | string | Yes | The order ID returned by `place_order` |

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `order_id` | string | Order ID |
| `status` | string | Order status (see Status Values below) |
| `symbol` | string | e.g. `AAPL.US` |
| `side` | string | `Buy` or `Sell` |
| `order_type` | string | e.g. `LO` (Limit Order) |
| `price` | string | Submitted limit price |
| `qty` | string | Submitted quantity |
| `executed_qty` | string | Filled quantity so far |
| `executed_price` | string | Average fill price (0 if unfilled) |
| `time_in_force` | string | e.g. `Day` |
| `remark` | string | Order label |

### `order_type` Values

| Value | Description |
|-------|-------------|
| `LO` | Limit Order |
| `ELO` | Enhanced Limit Order |
| `MO` | Market Order |
| `AO` | At-auction Order |
| `ALO` | At-auction Limit Order |
| `ODD` | Odd Lots Order |
| `LIT` | Limit If Touched |
| `MIT` | Market If Touched |
| `TSLPAMT` | Trailing Limit If Touched (Trailing Amount) |
| `TSLPPCT` | Trailing Limit If Touched (Trailing Percent) |
| `TSMAMT` | Trailing Market If Touched (Trailing Amount) |
| `TSMPCT` | Trailing Market If Touched (Trailing Percent) |
| `SLO` | SLO Order |

### `status` Values

| Value | Meaning |
|-------|---------|
| `NotReported` | Submitted, not yet acknowledged by exchange |
| `ReplacedNotReported` | Replaced order, not yet acknowledged |
| `ProtectedNotReported` | Protected order, not yet acknowledged |
| `VarietiesNotReported` | Varieties order, not yet acknowledged |
| `WaitToNew` | Waiting to be submitted |
| `NewStatus` | Acknowledged by exchange, live |
| `WaitToReplace` | Waiting for replace to be processed |
| `PendingReplaceStatus` | Replace request pending at exchange |
| `ReplacedStatus` | Successfully replaced |
| `PartialFilledStatus` | Partially filled, still live |
| `WaitToCancel` | Waiting for cancel to be processed |
| `PendingCancelStatus` | Cancel request pending at exchange |
| `FilledStatus` | Fully filled |
| `RejectedStatus` | Rejected by exchange |
| `CanceledStatus` | Cancelled |
| `ExpiredStatus` | Expired |
| `PartialWithdrawal` | Partially withdrawn |

### `time_in_force` Values

| Value | Description |
|-------|-------------|
| `Day` | Day Order — expires at end of trading session |
| `GTC` | Good Til Canceled — remains until filled or manually cancelled |
| `GTD` | Good Til Date — remains until a specified date |

### Example

```bash
curl -X GET 'https://mapi.matrixport.com/skopenapi/v1/orders?order_id=1217311455238426624' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2'
```

```json
{
  "code": 0,
  "data": {
    "order_id": "1217311455238426624",
    "status": "NotReported",
    "symbol": "AAPL.US",
    "qty": "10",
    "price": "89.00",
    "executed_qty": "0",
    "executed_price": "0",
    "side": "Buy",
    "order_type": "LO",
    "time_in_force": "Day",
    "remark": "test123"
  }
}
```

### Signing note (GET)

```
prehash = "{timestamp}GET/skopenapi/v1/orders&order_id=1217311455238426624"
```

---

## GET /v1/balance

Query account cash balances.

### Query Parameters

None.

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `total_cash` | string | Total cash in account |
| `total_available_cash` | string | Cash available to trade |
| `total_withdraw_cash` | string | Cash available to withdraw |

### Example

```bash
curl -X GET 'https://mapi.matrixport.com/skopenapi/v1/balance' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2'
```

```json
{
  "code": 0,
  "data": {
    "total_cash": "128479.47",
    "total_available_cash": "128169.47",
    "total_withdraw_cash": "128169.47"
  }
}
```

### Signing note (GET, no params)

```
prehash = "{timestamp}GET/skopenapi/v1/balance&"
```

Note the trailing `&` with an empty string — required even when there are no query parameters.

---

## GET /v1/positions

Query all current equity positions.

### Query Parameters

None.

### Response

Returns an array of position objects under `data`.

| Field | Type | Description |
|-------|------|-------------|
| `symbol` | string | e.g. `AAPL.US` |
| `ccy` | string | Settlement currency — currently `USD` only |
| `total_qty` | string | Total position size (held + reserved) |
| `available_qty` | string | Available quantity (can be sold) |
| `pre_close` | string | Previous session closing price |
| `average_cost_price` | string | Average cost basis price |

### Example

```bash
curl -X GET 'https://mapi.matrixport.com/skopenapi/v1/positions' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2'
```

```json
{
  "code": 0,
  "data": [
    {
      "symbol": "AAPL.US",
      "ccy": "USD",
      "total_qty": "10",
      "available_qty": "10",
      "pre_close": "88.50",
      "average_cost_price": "89.00"
    }
  ]
}
```

### Signing note (GET, no params)

```
prehash = "{timestamp}GET/skopenapi/v1/positions&"
```

---

## POST /v1/edit_order

Modify the price and/or quantity of an existing open order.

**Requires CONFIRM from user before proceeding.**

### Request Body (JSON)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order_id` | string | Yes | The order ID to modify |
| `qty` | string | Yes | New order quantity |
| `price` | string | No | New limit price. Omit to keep existing price |

### Response

`data` is empty on success — use `code == 0` to confirm the edit was accepted.

```json
{"code": 0, "data": {}}
```

### Examples

Change both qty and price:
```bash
curl -s -X POST 'https://mapi.matrixport.com/skopenapi/v1/edit_order' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2' \
  -H 'Content-Type: application/json' \
  -d '{"order_id":"1217311455238426624","qty":"5","price":"7.5"}'
```

Change qty only (omit `price` key entirely — do not send `null` or `""`):
```bash
  -d '{"order_id":"1217311455238426624","qty":"5"}'
```

### Signing note (POST)

```
prehash = "{timestamp}POST/skopenapi/v1/edit_order&{json_body}"
```

---

## POST /v1/cancel_order

Cancel an existing open order.

**Requires CONFIRM from user before proceeding.**

### Request Body (JSON)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order_id` | string | Yes | The order ID to cancel |

### Response

`data` is empty on success — use `code == 0` to confirm the cancellation was accepted.

```json
{"code": 0, "data": {}}
```

### Example

```bash
curl -s -X POST 'https://mapi.matrixport.com/skopenapi/v1/cancel_order' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2' \
  -H 'Content-Type: application/json' \
  -d '{"order_id":"1217311455238426624"}'
```

### Signing note (POST)

```
prehash = "{timestamp}POST/skopenapi/v1/cancel_order&{json_body}"
```

---

## GET /v1/open_orders

Query all currently open orders for the account.

### Query Parameters

None.

### Response

Returns an array of order objects under `data`. Each object has the same fields as the `/v1/orders` response.

| Field | Type | Description |
|-------|------|-------------|
| `order_id` | string | Order ID |
| `status` | string | Order status (see Status Values under GET /v1/orders) |
| `symbol` | string | e.g. `AAPL.US` |
| `side` | string | `Buy` or `Sell` |
| `order_type` | string | e.g. `LO` |
| `price` | string | Submitted limit price |
| `qty` | string | Submitted quantity |
| `executed_qty` | string | Filled quantity so far |
| `executed_price` | string | Average fill price (0 if unfilled) |
| `time_in_force` | string | e.g. `Day` |
| `remark` | string | Order label |

### Example

```bash
curl -s -X GET 'https://mapi.matrixport.com/skopenapi/v1/open_orders' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2'
```

```json
{
  "code": 0,
  "data": [
    {
      "order_id": "1217311455238426624",
      "status": "NewStatus",
      "symbol": "BTDR.US",
      "side": "Buy",
      "order_type": "LO",
      "price": "7.80",
      "qty": "10",
      "executed_qty": "0",
      "executed_price": "0",
      "time_in_force": "Day",
      "remark": ""
    }
  ]
}
```

### Signing note (GET, no params)

```
prehash = "{timestamp}GET/skopenapi/v1/open_orders&"
```
