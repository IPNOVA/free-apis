# Postal Code Lookup API

City, region and coordinates for any postal code, ZIP code or postcode: 887,000+ codes across 95 countries from the GeoNames open dataset, hosted locally and refreshed monthly.

**Base URL** `https://postalcodecheck.com` · **Live docs** [postalcodecheck.com/postal-code-api](https://postalcodecheck.com/postal-code-api) · [OpenAPI spec](./openapi.yaml)

## Endpoints

```
GET /api/postal/{cc}/{code}   # look up one code
GET /api/countries            # supported countries with counts and formats
```

| Parameter | Type | Description |
|---|---|---|
| `cc` | path, required | ISO 3166-1 alpha-2 country code, e.g. `US` |
| `code` | path, required | The postal code; case, spaces and hyphens are ignored |

## Example

```bash
curl https://postalcodecheck.com/api/postal/US/90210
```

```json
{
  "success": true,
  "country": "US",
  "country_name": "United States",
  "code": "90210",
  "places": [
    {
      "place": "Beverly Hills",
      "admin1": "California",
      "admin2": "Los Angeles",
      "lat": "34.0901",
      "lng": "-118.4065"
    }
  ],
  "url": "https://postalcodecheck.com/postal-code/US/90210",
  "source": "GeoNames (CC BY 4.0) via postalcodecheck.com"
}
```

### JavaScript

```js
const res = await fetch('https://postalcodecheck.com/api/postal/DE/10115');
const j = await res.json();
console.log(`${j.code} is ${j.places[0].place}, ${j.places[0].admin1}`);
```

### Python

```python
import requests

j = requests.get("https://postalcodecheck.com/api/postal/GB/SW1A", timeout=10).json()
print(j["places"][0]["place"])
```

## Errors

| Status | `reason` | Meaning |
|---|---|---|
| `400` | `invalid_input` | Country code is not two letters |
| `401` | `invalid_key` | The API key sent is malformed, unknown, revoked or expired. Keyless calls are never `401`. |
| `404` | `not_found` | Country not covered, or code not in the dataset |
| `429` | `rate_limit` | Rate limit hit: 30/minute per IP. Back off for the seconds in `Retry-After`. |
| `429` | `daily_limit` | Rate limit hit: 500/day per IP. Resets at midnight UTC. |
| `429` | `credits_exhausted` | Keyed request whose monthly plan allowance is used up. |

Every error uses the shared envelope described in the [repository README](../README.md#errors-and-api-keys):

```bash
curl -s http://127.0.0.1:8108/api/postal/US/00000
```

```json
{
  "success": false,
  "error": true,
  "reason": "not_found",
  "message": "Code not found for US.",
  "docs": "https://postalcodecheck.com/postal-code-api"
}
```

## Notes

- One code can cover several places (common in rural areas); `places` lists them all, primary first.
- UK coverage is at outward-code level (`SW1A`), which is what the open dataset provides.
- Data: [GeoNames postal dataset](https://www.geonames.org) under CC BY 4.0; the `source` field carries the attribution.
- Responses are cacheable for 24 hours (`Cache-Control: public, max-age=86400`).
- Optional API key: send `X-Api-Key` or `?key=` from a free api.ipnova.com account and your plan allowance applies instead of the per-IP limits; keyed responses carry `X-Credits-Remaining` and `X-Credits-Used`.
