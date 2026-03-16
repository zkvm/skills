---
name: us-equity
description: Matrixport US Equity trading via the Matrixport API. Use this skill whenever the user wants to trade US stocks or equities on Matrixport — including placing orders, checking order status, viewing account balance, or querying positions. Trigger even if the user doesn't say "Matrixport" explicitly, e.g. "buy 10 shares of AAPL", "sell my TSLA position", "what's my cash balance", "show my positions", "did my order fill". Requires API key and secret key.
metadata:
  version: 1.0.0
  author: Matrixport
license: MIT
---

# Matrixport US Equity Skill

Place and manage US equity orders on Matrixport. All endpoints require authentication via API key and secret key.

## Base URL

```
https://api.matrixport.com
```

## Authentication

All private endpoints require four request headers. See [Authentication Reference](references/authentication.md) for full details.

| Header | Description |
|--------|-------------|
| `X-MatrixPort-Access-Key` | Your API key |
| `X-Signature` | HMAC SHA256 hex signature of the prehash string |
| `X-Timestamp` | Current Unix timestamp in milliseconds |
| `X-Auth-Version` | Always `v2` |

### Prehash Format

```
{timestamp}{METHOD}{api_path}&{body_or_query_string}
```

- **GET**: `body_or_query_string` is the raw query string (e.g. `symbol=AAPL&limit=10`)
- **POST/PUT**: `body_or_query_string` is the JSON-serialized request body
- If there are no parameters/body, use an empty string after `&`

Sign using HMAC SHA256 over the prehash string with the secret key, hex-encoded. See [Authentication Reference](references/authentication.md) for complete code.

## Endpoints — Quick Reference

For full request/response details, see [Endpoint Reference](references/endpoints.md).

| Endpoint | Method | Description | Auth |
|----------|--------|-------------|------|
| `/v1/place_order` | POST | Place a limit or market order | Yes |
| `/v1/orders` | GET | Query order status by `order_id` | Yes |
| `/v1/balance` | GET | Account cash balances | Yes |
| `/v1/positions` | GET | All current equity positions | Yes |

**Symbol format:** `<TICKER>.<MARKET>` — e.g. `AAPL.US`, `TSLA.US`

**Order side values:** `Buy` / `Sell` (capitalized)

**All numeric values** (price, qty, cash) are returned and submitted as **strings**.

## Credentials

| Parameter | Type | Description |
|-----------|------|-------------|
| `apiKey` | string | Matrixport API key — display only first 5 + last 4 characters |
| `secretKey` | string | Matrixport secret key — never display, used only for signing |

### Storage

Store credentials in a local file (default `~/.matrixport/credentials`):
- Line 1: API key
- Line 2: Secret key

Only display masked versions to the user:
- API key: show first 5 and last 4 characters, mask the rest with `*`
- Secret key: show only last 5 characters, mask the rest with `*`

## Typical Workflow

```
1. User wants to place an order → ask for CONFIRM, then call POST /v1/place_order
2. Follow up → call GET /v1/orders to confirm order was received
3. Check balance or positions as needed via GET /v1/balance or GET /v1/positions
```

## Agent Behavior

### Credential Handling

- **Never log, echo, or display the secret key**
- When the user asks to show credentials, display masked versions only
- If credentials are not yet set, prompt the user to provide them and store them in the credentials file

### Making Requests

1. Read API key and secret key from the credentials file
2. Get the current millisecond timestamp
3. Build the prehash string using the format above
4. Generate the HMAC SHA256 hex signature
5. Attach all four required headers to the HTTP request
6. For POST/PUT requests, also set `Content-Type: application/json`

### Confirmations

- For **order placement** (POST requests), ask the user to type `CONFIRM` before proceeding — orders have financial consequences and cannot always be reversed
- For read-only (GET) requests, proceed without confirmation

### Error Handling

- On authentication errors (e.g. signature mismatch, timestamp out of range), report the error and suggest checking:
  1. That the API key and secret key are correct
  2. That the system clock is synchronized (timestamp tolerance is typically ±30 seconds)
- On business errors, return the raw error code and message from the API response

### Response Format

Return raw JSON responses. Do not modify or summarize response data unless the user explicitly requests it.

## Signing Requests

See [Authentication Reference](references/authentication.md) for the full signing process, Python and bash examples, and worked prehash examples for each endpoint type.

## Security

- Never share or expose your secret key
- Use IP allowlists in Matrixport API settings where available
- Enable only the permissions required for your use case
- Rotate credentials if you suspect compromise
