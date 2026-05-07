# BIT US Equity Authentication

All private REST requests require HMAC SHA256 signed requests.

## Base URL

```
https://mapi.matrixport.com
```

All stock endpoints are under the `/stock/v1/` prefix.

## Required Headers

All private requests must include the following headers:

| Header | Value |
|--------|-------|
| `X-MatrixPort-Access-Key` | Your API key |
| `X-Signature` | HMAC SHA256 signature of the prehash string |
| `X-Timestamp` | Current Unix time in milliseconds |
| `X-Auth-Version` | `v2` (fixed value) |

## Signing Process

### Step 1: Get Current Timestamp

Get the current Unix time in milliseconds:

```python
import time
timestamp = int(time.time() * 1000)
```

### Step 2: Build the Prehash String

The prehash string is constructed differently depending on the HTTP method:

**For GET requests:**
```
prehash = timestamp + METHOD + api_path + '&' + query_string
```

- `query_string`: URL-encoded query parameters (e.g. `currency=BTC&limit=50`)
- If no parameters, use an empty string: `timestamp + METHOD + api_path + '&'`

**For POST/PUT requests:**
```
prehash = timestamp + METHOD + api_path + '&' + json_body
```

- `json_body`: JSON-serialized request body string (e.g. `{"currency":"ETH","amount":"1"}`)
- If no body, use an empty string: `timestamp + METHOD + api_path + '&'`

**Format:**
```
{timestamp}{METHOD}{api_path}&{body_or_query_string}
```

Where:
- `{timestamp}` — millisecond epoch integer as a string
- `{METHOD}` — HTTP method in **UPPERCASE** (e.g. `GET`, `POST`, `PUT`, `DELETE`)
- `{api_path}` — the **full** request path including the `/stock/v1/` prefix (e.g. `/stock/v1/orders`)
- `&` — literal ampersand separator
- `{body_or_query_string}` — query string for GET, JSON body string for POST/PUT

### Step 3: Generate the Signature

Create an HMAC SHA256 signature of the prehash string using your secret key, and hex-encode the result:

```python
import hmac
import hashlib

def sign(message, secret_key):
    mac = hmac.new(
        bytes(secret_key, encoding='utf8'),
        bytes(message, encoding='utf-8'),
        digestmod=hashlib.sha256
    )
    return mac.hexdigest()
```

```bash
# bash equivalent
echo -n "$PREHASH" | openssl dgst -sha256 -hmac "$SECRET_KEY" | cut -d' ' -f2
```

### Step 4: Attach Headers

Add the four required headers to your HTTP request:

```
X-MatrixPort-Access-Key: <your_api_key>
X-Signature: <hex_signature>
X-Timestamp: <millisecond_timestamp>
X-Auth-Version: v2
```

## Complete Examples

### GET Request Example

Query order status by order ID:

```
timestamp  = 1731931956000
method     = GET
api_path   = /stock/v1/orders          ← full path including /stock/v1/ prefix
params     = order_id=1217311455238426624

prehash = "1731931956000GET/stock/v1/orders&order_id=1217311455238426624"
```

```python
import time, hmac, hashlib, requests

def get_timestamp():
    return int(time.time() * 1000)

def sign(message, secret_key):
    mac = hmac.new(bytes(secret_key, encoding='utf8'), bytes(message, encoding='utf-8'), digestmod=hashlib.sha256)
    return mac.hexdigest()

def pre_hash(timestamp, method, request_path, body):
    return str(timestamp) + str.upper(method) + request_path + '&' + body

secret_key = "your_secret_key"
api_key    = "your_api_key"
base_url   = "https://mapi.matrixport.com"
timestamp  = get_timestamp()

method    = "GET"
api_path  = "/stock/v1/orders"   # full path used for both prehash and URL
query_str = "order_id=1217311455238426624"

prehash   = pre_hash(timestamp, method, api_path, query_str)
signature = sign(prehash, secret_key)

headers = {
    "X-MatrixPort-Access-Key": api_key,
    "X-Signature":             signature,
    "X-Timestamp":             str(timestamp),
    "X-Auth-Version":          "v2",
}

response = requests.get(f"{base_url}{api_path}?{query_str}", headers=headers)
print(response.json())
```

```bash
API_KEY="your_api_key"
SECRET_KEY="your_secret_key"
BASE_URL="https://mapi.matrixport.com"
TIMESTAMP=$(date +%s000)

METHOD="GET"
API_PATH="/stock/v1/orders"   # full path used for both prehash and URL
QUERY="order_id=1217311455238426624"

PREHASH="${TIMESTAMP}${METHOD}${API_PATH}&${QUERY}"
SIGNATURE=$(echo -n "$PREHASH" | openssl dgst -sha256 -hmac "$SECRET_KEY" | cut -d' ' -f2)

curl -s -X GET "${BASE_URL}${API_PATH}?${QUERY}" \
  -H "X-MatrixPort-Access-Key: ${API_KEY}" \
  -H "X-Signature: ${SIGNATURE}" \
  -H "X-Timestamp: ${TIMESTAMP}" \
  -H "X-Auth-Version: v2"
```

### POST Request Example

Place a limit order:

```
timestamp = 1731931956000
method    = POST
api_path  = /stock/v1/place_order     ← full path including /stock/v1/ prefix
body      = {"symbol":"AAPL.US","side":"Buy","price":"89.0","qty":"10","remark":"test123"}

prehash   = 1731931956000POST/stock/v1/place_order&{"symbol":"AAPL.US","side":"Buy","price":"89.0","qty":"10","remark":"test123"}
```

```python
import time, hmac, hashlib, json, requests

def get_timestamp():
    return int(time.time() * 1000)

def sign(message, secret_key):
    mac = hmac.new(bytes(secret_key, encoding='utf8'), bytes(message, encoding='utf-8'), digestmod=hashlib.sha256)
    return mac.hexdigest()

def pre_hash(timestamp, method, request_path, body):
    return str(timestamp) + str.upper(method) + request_path + '&' + body

secret_key = "your_secret_key"
api_key    = "your_api_key"
base_url   = "https://mapi.matrixport.com"
timestamp  = get_timestamp()

method   = "POST"
api_path = "/stock/v1/place_order"   # full path used for both prehash and URL
params   = {
    "symbol": "AAPL.US",
    "side":   "Buy",
    "price":  "89.0",
    "qty":    "10",
    "remark": "test123",
}
# Serialize once — use the same string for both signing and the request body
body = json.dumps(params, separators=(',', ':'))

prehash   = pre_hash(timestamp, method, api_path, body)
signature = sign(prehash, secret_key)

headers = {
    "X-MatrixPort-Access-Key": api_key,
    "X-Signature":             signature,
    "X-Timestamp":             str(timestamp),
    "X-Auth-Version":          "v2",
    "Content-Type":            "application/json",
}

response = requests.post(f"{base_url}{api_path}", headers=headers, data=body)
print(response.json())
```

```bash
API_KEY="your_api_key"
SECRET_KEY="your_secret_key"
BASE_URL="https://mapi.matrixport.com"
TIMESTAMP=$(date +%s000)

METHOD="POST"
API_PATH="/stock/v1/place_order"   # full path used for both prehash and URL
BODY='{"symbol":"AAPL.US","side":"Buy","price":"89.0","qty":"10","remark":"test123"}'

PREHASH="${TIMESTAMP}${METHOD}${API_PATH}&${BODY}"
SIGNATURE=$(echo -n "$PREHASH" | openssl dgst -sha256 -hmac "$SECRET_KEY" | cut -d' ' -f2)

curl -s -X POST "${BASE_URL}${API_PATH}" \
  -H "X-MatrixPort-Access-Key: ${API_KEY}" \
  -H "X-Signature: ${SIGNATURE}" \
  -H "X-Timestamp: ${TIMESTAMP}" \
  -H "X-Auth-Version: v2" \
  -H "Content-Type: application/json" \
  -d "$BODY"
```

## Executing Requests

**Preferred: `requests` library (Python)**

Use `requests` rather than `urllib` — it handles SSL certificates correctly and always returns the response body regardless of HTTP status code:

```bash
pip install requests
```

```python
response = requests.get(url, headers=headers)
data = response.json()   # works for both 200 and error responses
```

**macOS SSL note:** If you see `ssl.SSLCertVerificationError`, run:
```bash
/Applications/Python\ 3.x/Install\ Certificates.command
# or: pip install certifi
```

**curl** is also a reliable option for quick testing — it handles SSL and non-2xx responses without extra setup:
```bash
curl -s -X GET "..." -H "..."   # -s suppresses progress output
```

## Error Handling

Always read the full response body — error details are in the JSON, not just the HTTP status code.

```python
response = requests.get(url, headers=headers)
result = response.json()
if result.get("code") != 0:
    print(f"API error {result['code']}: {result.get('message')}")
else:
    print(result["data"])
```

### Common Error Codes

| Code | Meaning | Likely cause |
|------|---------|--------------|
| `0` | Success | — |
| `12000013` | Internal Server Error | Signing error, malformed request, or server-side issue — double-check prehash format and timestamp |
| Auth errors | Invalid signature | Wrong prehash format, wrong secret key, or timestamp too far from server time (check clock sync) |

If you receive repeated auth errors, verify:
1. Timestamp is current (not hardcoded or stale)
2. Prehash format exactly matches: `{timestamp}{METHOD}{path}&{body_or_query}`
3. For POST: the JSON body string used for signing is byte-for-byte identical to the body sent in the request

## Security Notes

- Never share your secret key
- Store credentials securely; never hard-code them in source files
- Use IP allowlists in BIT API settings where available
- Enable only the required permissions for the API key (e.g. read-only vs. trading)
- Timestamps must be within a reasonable window of server time; if you receive authentication errors, check clock synchronization
