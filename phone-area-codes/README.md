# Phone / Area Code API

Analyze any phone number's prefix: country, dialling code, and for North American numbers the area code, its location and timezone. Purely algorithmic against public ITU / NANPA assignments and libphonenumber data; numbers are never stored or logged.

**Base URL** `https://areacodecheck.com` · **Live docs** [areacodecheck.com/phone-api](https://areacodecheck.com/phone-api) · [OpenAPI spec](./openapi.yaml)

## Endpoint

```
GET /api/phone/{number}
```

| Parameter | Type | Description |
|---|---|---|
| `number` | path, required | 3 to 15 digits, with or without `+` / `00` international prefix. Send the `+` as-is or URL-encoded as `%2B`, both are accepted |

## Example

```bash
curl "https://areacodecheck.com/api/phone/+14155552671"
```

```json
{
  "success": true,
  "data": {
    "input": "+14155552671",
    "e164": "+14155552671",
    "country_code": "1",
    "country": "United States, Canada & NANP territories",
    "region_iso": "US",
    "shared_note": "Shared across 25 countries and territories of the North American Numbering Plan",
    "area_code": "415",
    "area_location": "California",
    "area_country": "United States",
    "timezone": "America/Los_Angeles",
    "nanp_valid": true
  },
  "page": "https://areacodecheck.com/area-code/415",
  "attribution": "Data by AreaCodeCheck.com - includes libphonenumber data (Apache 2.0). Numbers are never stored.",
  "docs": "https://areacodecheck.com/phone-api"
}
```

### JavaScript

```js
const res = await fetch('https://areacodecheck.com/api/phone/+35724656406');
const { data } = await res.json();
console.log(`${data.country} (+${data.country_code})`);
```

### Python

```python
import requests

j = requests.get("https://areacodecheck.com/api/phone/+442071234567", timeout=10).json()
print(j["data"]["country"])
```

## Errors

| Status | `reason` | Meaning |
|---|---|---|
| `400` | `invalid_input` | Fewer than 3 or more than 15 digits |
| `401` | `invalid_key` | The API key sent is malformed, unknown, revoked or expired. Keyless calls are never `401`. |
| `429` | `rate_limit` | Rate limit hit: 30/minute per IP. Back off for the seconds in `Retry-After`. |
| `429` | `daily_limit` | Rate limit hit: 500/day per IP. Resets at midnight UTC. |
| `429` | `credits_exhausted` | Keyed request whose monthly plan allowance is used up. |

Every error uses the shared envelope described in the [repository README](../README.md#errors-and-api-keys):

```bash
curl -s http://127.0.0.1:8105/api/phone/1-1
```

```json
{
  "success": false,
  "error": true,
  "reason": "invalid_input",
  "message": "Provide a phone number of 3 to 15 digits: /api/phone/+14155552671",
  "docs": "https://areacodecheck.com/phone-api"
}
```

## Notes

- This is prefix analysis, not line verification: it identifies where a number belongs, never whether it is active or who owns it.
- Area-code detail (`area_code`, `area_location`, `timezone`) applies to NANP (+1) numbers; other countries return the country-level fields.
- Privacy by design: the number is analyzed in memory and discarded; only anonymous rate-limit counters exist.
- Responses are sent with `Cache-Control: no-store`, consistent with the no-storage design; nothing is cached.
- Optional API key: send `X-Api-Key` or `?key=` from a free api.ipnova.com account and your plan allowance applies instead of the per-IP limits; keyed responses carry `X-Credits-Remaining` and `X-Credits-Used`.
