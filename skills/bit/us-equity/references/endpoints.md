# BIT US Equity — Endpoint Reference

All endpoints require authentication headers. See `authentication.md` for signing details.

## Base URL

```
https://mapi.matrixport.com
```

All stock endpoints are under the `/stock/v1/` prefix.

## Response Envelope

Every response wraps data in a standard envelope:

```json
{"code": 0, "message": "success", "data": { ... }}
```

- `code: 0` — success
- `code: <non-zero>` — error; `message` contains a human-readable description

---

## POST /stock/v1/place_order

Place a new order (limit or market).

**Requires CONFIRM from user before proceeding.**

### Request Body (JSON)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `symbol` | string | Yes | Stock symbol in `<TICKER>.<MARKET>` format, e.g. `AAPL.US` |
| `side` | string | Yes | `Buy` or `Sell` (case-insensitive) |
| `qty` | string | Yes | Order quantity — positive integer string, e.g. `"100"` |
| `order_type` | string | No | `LO` (limit) or `MO` (market). Defaults to `LO` |
| `price` | string | Conditional | Limit price as a decimal string, e.g. `"150.50"`. Required when `order_type=LO` |
| `time_in_force` | string | No | `Day` (default) — valid for current trading day; `GTC` — until cancelled; `GTD` — until expiration date |
| `expire_date` | string | Conditional | `YYYY-MM-DD`. Required when `time_in_force=GTD` |
| `outside_rth` | string | No | Outside regular trading hours. See Appendix C |
| `remark` | string | No | Optional order note |

### Response

| Field | Type | Description |
|-------|------|-------------|
| `order_id` | string | Assigned order ID |

### Example

```bash
curl -X POST 'https://mapi.matrixport.com/stock/v1/place_order' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2' \
  -H 'Content-Type: application/json' \
  -d '{"symbol":"AAPL.US","side":"Buy","qty":"100","order_type":"LO","price":"150.50","remark":"test"}'
```

```json
{"code": 0, "message": "success", "data": {"order_id": "701276261045858304"}}
```

### Signing note (POST)

```
prehash = "{timestamp}POST/stock/v1/place_order&{json_body}"
```

> **Important:** The `json_body` used to compute the signature must be byte-for-byte identical to the body sent in the HTTP request. Construct the JSON string once (e.g. `body = json.dumps(params)`) and use that same string for both signing and the request body. Do not re-serialize.

---

## POST /stock/v1/edit_order

Modify the price and quantity of an existing open order.

**Requires CONFIRM from user before proceeding.**

### Request Body (JSON)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order_id` | string | Yes | Order ID to modify |
| `qty` | string | Yes | New quantity — positive integer string |
| `price` | string | Yes | New limit price — decimal string |

> Both `qty` and `price` are required. To change only one, resend the existing value for the other.

### Response

`data` is empty on success — use `code == 0` to confirm acceptance.

```json
{"code": 0, "message": "success", "data": {}}
```

### Example

```bash
curl -X POST 'https://mapi.matrixport.com/stock/v1/edit_order' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2' \
  -H 'Content-Type: application/json' \
  -d '{"order_id":"701276261045858304","qty":"200","price":"148.00"}'
```

### Signing note (POST)

```
prehash = "{timestamp}POST/stock/v1/edit_order&{json_body}"
```

---

## POST /stock/v1/cancel_order

Cancel an existing open order.

**Requires CONFIRM from user before proceeding.**

### Request Body (JSON)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order_id` | string | Yes | Order ID to cancel |

### Response

```json
{"code": 0, "message": "success", "data": {}}
```

### Example

```bash
curl -X POST 'https://mapi.matrixport.com/stock/v1/cancel_order' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2' \
  -H 'Content-Type: application/json' \
  -d '{"order_id":"701276261045858304"}'
```

### Signing note (POST)

```
prehash = "{timestamp}POST/stock/v1/cancel_order&{json_body}"
```

---

## GET /stock/v1/orders

Query orders matching the given filters. Returns up to 1000 records as an array.

> At least one query parameter must be provided. Otherwise the server returns HTTP 400 with `{"code":20080400,"message":"missing query parameters"}`.

### Query Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `symbol` | string | No | e.g. `AAPL.US` |
| `side` | string | No | `Buy` / `Sell` (case-insensitive) |
| `start_at` | string | No | Unix timestamp (seconds) — query window start |
| `end_at` | string | No | Unix timestamp (seconds) — query window end |
| `order_id` | string | No | If provided, all other params are ignored |

### Response Fields

`data` is an array of order objects:

| Field | Type | Description |
|-------|------|-------------|
| `order_id` | string | Order ID |
| `status` | string | See Appendix B |
| `symbol` | string | e.g. `AAPL.US` |
| `qty` | string | Submitted quantity |
| `price` | string | Submitted price |
| `executed_qty` | string | Filled quantity |
| `executed_price` | string | Average fill price (`""` if unfilled) |
| `side` | string | `Buy` / `Sell` |
| `order_type` | string | See Appendix A |
| `time_in_force` | string | `Day` / `GTC` / `GTD` |
| `outside_rth` | string | See Appendix C |
| `remark` | string | Order note |
| `submitted_at` | string | Unix timestamp (seconds) — submission time |
| `updated_at` | string | Unix timestamp (seconds) — last update |

### Example

```bash
curl -X GET 'https://mapi.matrixport.com/stock/v1/orders?symbol=AAPL.US&start_at=1680000000&end_at=1680856800' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2'
```

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "order_id": "701276261045858304",
      "status": "FilledStatus",
      "symbol": "AAPL.US",
      "qty": "100",
      "price": "150.50",
      "executed_qty": "100",
      "executed_price": "150.48",
      "side": "Buy",
      "order_type": "LO",
      "time_in_force": "Day",
      "outside_rth": "RTH_ONLY",
      "remark": "test",
      "submitted_at": "1680856500",
      "updated_at": "1680856746"
    }
  ]
}
```

### Signing note (GET)

```
prehash = "{timestamp}GET/stock/v1/orders&order_id=701276261045858304"
```

---

## GET /stock/v1/order_detail

Query the detail of a single order, including the fee breakdown.

### Query Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order_id` | string | Yes | Order ID |

### Response Fields

All standard order fields (same as `/stock/v1/orders` items) plus:

| Field | Type | Description |
|-------|------|-------------|
| `charge_detail` | object | Fee breakdown. May be `null` if the order has not been filled |

`charge_detail`:

| Field | Type | Description |
|-------|------|-------------|
| `currency` | string | Fee currency |
| `total_amount` | string | Total fee amount |
| `items` | array | Fee categories |
| `items[].code` | string | Category code (e.g. `BROKER_FEES`, `THIRD_FEES`) |
| `items[].fees` | array | Fee items in this category |
| `items[].fees[].code` | string | Fee item code (e.g. `PlatformFee`, `ClearingFee`) |
| `items[].fees[].amount` | string | Fee amount |
| `items[].fees[].currency` | string | Fee currency |

### Example

```bash
curl -X GET 'https://mapi.matrixport.com/stock/v1/order_detail?order_id=701276261045858304' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2'
```

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "order_id": "701276261045858304",
    "status": "FilledStatus",
    "symbol": "AAPL.US",
    "side": "Buy",
    "order_type": "LO",
    "time_in_force": "Day",
    "qty": "100",
    "price": "150.50",
    "executed_qty": "100",
    "executed_price": "150.48",
    "submitted_at": "1680856500",
    "updated_at": "1680856746",
    "outside_rth": "RTH_ONLY",
    "remark": "test",
    "charge_detail": {
      "currency": "USD",
      "total_amount": "1.00",
      "items": [
        {"code": "BROKER_FEES", "fees": [{"code": "PlatformFee", "amount": "0.99", "currency": "USD"}]},
        {"code": "THIRD_FEES", "fees": [{"code": "ClearingFee", "amount": "0.01", "currency": "USD"}]}
      ]
    }
  }
}
```

### Signing note (GET)

```
prehash = "{timestamp}GET/stock/v1/order_detail&order_id=701276261045858304"
```

---

## GET /stock/v1/open_orders

Query all currently open (non-terminal) orders for the current trading day. Returns up to 1000 records.

### Query Parameters

None.

### Response

`data` is an array of order objects with the same structure as `/stock/v1/orders` items.

### Example

```bash
curl -X GET 'https://mapi.matrixport.com/stock/v1/open_orders' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2'
```

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "order_id": "701276261045858304",
      "status": "NewStatus",
      "symbol": "AAPL.US",
      "qty": "100",
      "price": "150.50",
      "executed_qty": "0",
      "executed_price": "",
      "side": "Buy",
      "order_type": "LO",
      "time_in_force": "Day",
      "outside_rth": "RTH_ONLY",
      "remark": "",
      "submitted_at": "1680856500",
      "updated_at": "1680856500"
    }
  ]
}
```

### Signing note (GET, no params)

```
prehash = "{timestamp}GET/stock/v1/open_orders&"
```

Note the trailing `&` with empty string — required even when there are no query parameters.

---

## GET /stock/v1/balance

Query account cash balance. Currency is fixed to USD.

### Query Parameters

None.

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `total_cash` | string | Total cash (USD) |
| `total_available_cash` | string | Cash available to trade (USD) |
| `total_withdraw_cash` | string | Cash available to withdraw (USD) |

### Example

```bash
curl -X GET 'https://mapi.matrixport.com/stock/v1/balance' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2'
```

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "total_cash": "100000.00",
    "total_available_cash": "85000.00",
    "total_withdraw_cash": "80000.00"
  }
}
```

### Signing note (GET, no params)

```
prehash = "{timestamp}GET/stock/v1/balance&"
```

---

## GET /stock/v1/positions

Query all current stock positions.

### Query Parameters

None.

### Response

`data` is an object containing a `positions` array (empty `[]` if no positions).

| Field | Type | Description |
|-------|------|-------------|
| `positions` | array | List of position objects |

Position fields:

| Field | Type | Description |
|-------|------|-------------|
| `symbol` | string | e.g. `AAPL.US` |
| `market` | string | e.g. `US` |
| `ccy` | string | Currency, currently `USD` |
| `total_qty` | string | Total position size (held + reserved) |
| `available_qty` | string | Available (sellable) quantity |
| `pre_close` | string | Previous session closing price |
| `last_done` | string | Latest price |
| `average_cost_price` | string | Average cost basis |

### Example

```bash
curl -X GET 'https://mapi.matrixport.com/stock/v1/positions' \
  -H 'X-MatrixPort-Access-Key: <api_key>' \
  -H 'X-Signature: <signature>' \
  -H 'X-Timestamp: <timestamp_ms>' \
  -H 'X-Auth-Version: v2'
```

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "positions": [
      {
        "symbol": "AAPL.US",
        "market": "US",
        "ccy": "USD",
        "total_qty": "200",
        "available_qty": "200",
        "pre_close": "151.00",
        "last_done": "152.30",
        "average_cost_price": "149.75"
      }
    ]
  }
}
```

### Signing note (GET, no params)

```
prehash = "{timestamp}GET/stock/v1/positions&"
```

---

## Appendix A: Order Types (`order_type`)

| Value | Description |
|-------|-------------|
| `LO` | Limit Order |
| `MO` | Market Order |

---

## Appendix B: Order Statuses (`status`)

`Active = ✅` means the order is still live (non-terminal) and may still transition or be cancelled.

| Value | Description | Active |
|-------|-------------|--------|
| `NotReported` | Not reported | ✅ |
| `ProtectedNotReported` | Protected not reported | ✅ |
| `VarietiesNotReported` | Varieties not reported | ✅ |
| `ReplacedNotReported` | Replaced not reported | ✅ |
| `WaitToNew` | Waiting to submit | ✅ |
| `NewStatus` | Submitted, pending fill | ✅ |
| `WaitToReplace` | Waiting to modify | ✅ |
| `PendingReplaceStatus` | Modification in progress | ✅ |
| `ReplacedStatus` | Modified | ✅ |
| `WaitToCancel` | Waiting to cancel | ✅ |
| `PendingCancelStatus` | Cancellation in progress | ✅ |
| `PartialFilledStatus` | Partially filled | ✅ |
| `FilledStatus` | Fully filled | ❌ |
| `CanceledStatus` | Cancelled | ❌ |
| `ExpiredStatus` | Expired | ❌ |
| `RejectedStatus` | Rejected | ❌ |
| `PartialWithdrawal` | Partially withdrawn | ❌ |

---

## Appendix C: Outside Regular Trading Hours (`outside_rth`)

| Value | Description |
|-------|-------------|
| `RTH_ONLY` | Regular trading hours only |
| `ANY_TIME` | Allow pre-market and post-market trading |
| `OVERNIGHT` | Overnight session only |
