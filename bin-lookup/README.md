# BIN / IIN Lookup API

Identify the network, issuer, country and card type behind the first 6 to 8 digits of any payment card. Served from a locally hosted dataset of 343,000+ BINs; nothing is proxied at request time and full card numbers are never accepted.

**Base URL** `https://cardbincheck.com` · **Live docs** [cardbincheck.com/bin-lookup-api](https://cardbincheck.com/bin-lookup-api) · [OpenAPI spec](./openapi.yaml)

## Endpoints

```
GET /api/bin/{bin}          # look up one BIN
POST /api/bin-batch         # look up up to 100 BINs in one call
```

| Parameter | Type | Description |
|---|---|---|
| `bin` | path, required | 6 to 8 digits, the start of a card number |
| `bins` | body, required (batch only) | JSON array of 1 to 100 BIN strings, each 6 to 8 digits: `{"bins": ["424242", "510510"]}` |

## Example

```bash
curl https://cardbincheck.com/api/bin/440066
```

```json
{
  "success": true,
  "data": {
    "bin": "440066",
    "scheme": "Visa",
    "funding": "Credit",
    "segment": "Signature",
    "issuer": "Unknown",
    "country_code": "US",
    "country_name": "United States",
    "bank_url": "",
    "bank_phone": ""
  },
  "page": "https://cardbincheck.com/bin/440066",
  "attribution": "Data by CardBinCheck.com - free BIN lookup API",
  "docs": "https://cardbincheck.com/bin-lookup-api"
}
```

### Batch

```bash
curl -X POST https://cardbincheck.com/api/bin-batch \
  -H "Content-Type: application/json" \
  -d '{"bins": ["440066", "510510"]}'
```

```json
{
  "success": true,
  "count": 2,
  "found": 2,
  "results": [
    {
      "found": true,
      "bin": "440066",
      "scheme": "Visa",
      "funding": "Credit",
      "segment": "Signature",
      "issuer": "Unknown",
      "country_code": "US",
      "country_name": "United States",
      "bank_url": "",
      "bank_phone": ""
    },
    {
      "found": true,
      "bin": "510510",
      "scheme": "Mastercard",
      "funding": "Credit",
      "segment": "Unknown",
      "issuer": "Bank Of Hawaii",
      "country_code": "US",
      "country_name": "United States",
      "bank_url": "www.boh.com/personal/",
      "bank_phone": "643-3888"
    }
  ],
  "attribution": "Data by CardBinCheck.com - free BIN lookup API",
  "docs": "https://cardbincheck.com/bin-lookup-api"
}
```

### JavaScript

```js
const res = await fetch('https://cardbincheck.com/api/bin/440066');
const { data } = await res.json();
console.log(`${data.scheme} ${data.funding} card from ${data.country_name}`);
```

### Python

```python
import requests

data = requests.get("https://cardbincheck.com/api/bin/440066", timeout=10).json()["data"]
print(f"{data['scheme']} {data['funding']} card from {data['country_name']}")
```

## Errors

| Status | `reason` | Meaning |
|---|---|---|
| `400` | `invalid_input` | Input is not 6 to 8 digits; or, on the batch endpoint, `bins` missing/empty/not an array, or more than 100 items |
| `401` | `invalid_key` | The API key sent is malformed, unknown, revoked or expired. Keyless calls are never `401`. |
| `404` | `not_found` | BIN not present in the public dataset |
| `405` | `method_not_allowed` | `GET` on `/api/bin-batch`, or a non-`POST`/`GET` method on either endpoint |
| `413` | `payload_too_large` | Batch request body over 64 KB |
| `429` | `rate_limit` | Rate limit hit: 30/minute per IP. Back off for the seconds in `Retry-After`. |
| `429` | `daily_limit` | Rate limit hit: 500/day per IP. Resets at midnight UTC. |
| `429` | `credits_exhausted` | Keyed request whose monthly plan allowance is used up. |

Every error uses the shared envelope described in the [repository README](../README.md#errors-and-api-keys):

```bash
curl -s http://127.0.0.1:8101/api/bin/999999
```

```json
{
  "success": false,
  "error": true,
  "reason": "not_found",
  "message": "BIN not found in the public dataset.",
  "docs": "https://cardbincheck.com/bin-lookup-api",
  "bin": "999999"
}
```

## Notes

- BIN data identifies card *ranges*, never an individual card or cardholder. It is the same public industry data merchants use for routing and risk.
- Fields the dataset does not disclose come back as `"Unknown"` or an empty string, never invented.
- Responses are cacheable for 24 hours (`Cache-Control: public, max-age=86400`); batch responses are sent `no-store` since each is call-specific.
- A batch call counts against the daily limit as `ceil(n / 10)` requests (based on the size of the submitted `bins` array), so a 100-BIN batch costs 10, not 100. Duplicate BINs within one batch are looked up once; the extra occurrences are dropped, so `results` and `count` can be shorter than the input array.
- Optional API key: send `X-Api-Key` or `?key=` from a free api.ipnova.com account and your plan allowance applies instead of the per-IP limits; keyed responses carry `X-Credits-Remaining` and `X-Credits-Used`.
