# Phone / Area Code API

Analyze any phone number's prefix: country, dialling code, and for North American numbers the area code, its location and timezone. Purely algorithmic against public ITU / NANPA assignments and libphonenumber data; numbers are never stored or logged.

**Base URL** `https://areacodecheck.com` · **Live docs** [areacodecheck.com/phone-api](https://areacodecheck.com/phone-api) · [OpenAPI spec](./openapi.yaml)

## Endpoint

```
GET /api/phone/{number}
```

| Parameter | Type | Description |
|---|---|---|
| `number` | path, required | 3 to 15 digits, with or without `+` / `00` international prefix. URL-encode the `+` as `%2B` |

## Example

```bash
curl "https://areacodecheck.com/api/phone/%2B14155552671"
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
const res = await fetch('https://areacodecheck.com/api/phone/%2B35724656406');
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

| Status | Meaning |
|---|---|
| `400` | Fewer than 3 or more than 15 digits |
| `429` | Rate limit hit: 30/minute or 500/day per IP |

## Notes

- This is prefix analysis, not line verification: it identifies where a number belongs, never whether it is active or who owns it.
- Area-code detail (`area_code`, `area_location`, `timezone`) applies to NANP (+1) numbers; other countries return the country-level fields.
- Privacy by design: the number is analyzed in memory and discarded; only anonymous rate-limit counters exist.
