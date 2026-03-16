# Matrixport US Equity Authentication

All private REST requests require HMAC SHA256 signed requests.

## Base URL

```
https://api.matrixport.com
```

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
- `{api_path}` — the request path including leading `/` (e.g. `/v1/orders`)
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
api_path   = /v1/orders
params     = order_id=1217311455238426624

prehash = "1731931956000GET/v1/orders&order_id=1217311455238426624"
```

```python
import time, hmac, hashlib

def get_timestamp():
    return int(time.time() * 1000)

def sign(message, secret_key):
    mac = hmac.new(bytes(secret_key, encoding='utf8'), bytes(message, encoding='utf-8'), digestmod=hashlib.sha256)
    return mac.hexdigest()

def pre_hash(timestamp, method, request_path, body):
    return str(timestamp) + str.upper(method) + request_path + '&' + body

secret_key = "your_secret_key"
api_key    = "your_api_key"
timestamp  = get_timestamp()

method    = "GET"
api_path  = "/v1/orders"
query_str = "order_id=1217311455238426624"

prehash   = pre_hash(timestamp, method, api_path, query_str)
signature = sign(prehash, secret_key)

headers = {
    "X-MatrixPort-Access-Key": api_key,
    "X-Signature":             signature,
    "X-Timestamp":             str(timestamp),
    "X-Auth-Version":          "v2",
}
```

```bash
API_KEY="your_api_key"
SECRET_KEY="your_secret_key"
BASE_URL="https://api.matrixport.com"
TIMESTAMP=$(date +%s000)

METHOD="GET"
API_PATH="/v1/orders"
QUERY="order_id=1217311455238426624"

PREHASH="${TIMESTAMP}${METHOD}${API_PATH}&${QUERY}"
SIGNATURE=$(echo -n "$PREHASH" | openssl dgst -sha256 -hmac "$SECRET_KEY" | cut -d' ' -f2)

curl -X GET "${BASE_URL}${API_PATH}?${QUERY}" \
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
api_path  = /v1/place_order
body      = {"symbol":"AAPL.US","side":"Buy","price":"89.0","qty":"10","remark":"test123"}

prehash   = 1731931956000POST/v1/place_order&{"symbol":"AAPL.US","side":"Buy","price":"89.0","qty":"10","remark":"test123"}
```

```python
import time, hmac, hashlib, json

def get_timestamp():
    return int(time.time() * 1000)

def sign(message, secret_key):
    mac = hmac.new(bytes(secret_key, encoding='utf8'), bytes(message, encoding='utf-8'), digestmod=hashlib.sha256)
    return mac.hexdigest()

def pre_hash(timestamp, method, request_path, body):
    return str(timestamp) + str.upper(method) + request_path + '&' + body

secret_key = "your_secret_key"
api_key    = "your_api_key"
timestamp  = get_timestamp()

method   = "POST"
api_path = "/v1/place_order"
params   = {
    "symbol": "AAPL.US",
    "side":   "Buy",
    "price":  "89.0",
    "qty":    "10",
    "remark": "test123",
}
body = json.dumps(params)  # key order in body must match what is sent in the request

prehash   = pre_hash(timestamp, method, api_path, body)
signature = sign(prehash, secret_key)

headers = {
    "X-MatrixPort-Access-Key": api_key,
    "X-Signature":             signature,
    "X-Timestamp":             str(timestamp),
    "X-Auth-Version":          "v2",
    "Content-Type":            "application/json",
}
```

```bash
API_KEY="your_api_key"
SECRET_KEY="your_secret_key"
BASE_URL="https://api.matrixport.com"
TIMESTAMP=$(date +%s000)

METHOD="POST"
API_PATH="/v1/place_order"
BODY='{"symbol":"AAPL.US","side":"Buy","price":"89.0","qty":"10","remark":"test123"}'

PREHASH="${TIMESTAMP}${METHOD}${API_PATH}&${BODY}"
SIGNATURE=$(echo -n "$PREHASH" | openssl dgst -sha256 -hmac "$SECRET_KEY" | cut -d' ' -f2)

curl -X POST "${BASE_URL}${API_PATH}" \
  -H "X-MatrixPort-Access-Key: ${API_KEY}" \
  -H "X-Signature: ${SIGNATURE}" \
  -H "X-Timestamp: ${TIMESTAMP}" \
  -H "X-Auth-Version: v2" \
  -H "Content-Type: application/json" \
  -d "$BODY"
```

## Security Notes

- Never share your secret key
- Store credentials securely; never hard-code them in source files
- Use IP allowlists in Matrixport API settings where available
- Enable only the required permissions for the API key (e.g. read-only vs. trading)
- Timestamps must be within a reasonable window of server time; if you receive authentication errors, check clock synchronization
