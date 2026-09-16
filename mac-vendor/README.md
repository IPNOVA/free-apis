# MAC Vendor Lookup API

Identify the manufacturer behind any MAC address or OUI prefix, from a locally hosted copy of the complete IEEE registry (MA-L, MA-M and MA-S, 53,000+ assignments), refreshed monthly.

**Base URL** `https://macvendorcheck.com` · **Live docs** [macvendorcheck.com/mac-address-lookup-api](https://macvendorcheck.com/mac-address-lookup-api) · [OpenAPI spec](./openapi.yaml)

## Endpoint

```
GET /api/mac/{mac}
```

| Parameter | Type | Description |
|---|---|---|
| `mac` | path, required | A full MAC address or an OUI prefix, any common format: `00:1A:2B:3C:4D:5E`, `00-1A-2B`, `001a.2b3c.4d5e`, `001A2B` |

## Example

```bash
curl https://macvendorcheck.com/api/mac/00:1A:2B:3C:4D:5E
```

```json
{
  "success": true,
  "mac": "00:1A:2B:3C:4D:5E",
  "oui": "001A2B",
  "vendor": "Ayecom Technology Co., Ltd.",
  "registry": "MA-L",
  "country": "TW",
  "page": "https://macvendorcheck.com/mac/001A2B",
  "attribution": "Data from the IEEE MA-L/MA-M/MA-S registry, via MacVendorCheck.com"
}
```

### JavaScript

```js
const res = await fetch('https://macvendorcheck.com/api/mac/B8:27:EB:00:00:01');
const j = await res.json();
console.log(j.vendor); // Raspberry Pi Foundation
```

### Python

```python
import requests

j = requests.get("https://macvendorcheck.com/api/mac/001A2B", timeout=10).json()
print(j["vendor"], j["registry"])
```

## Errors

| Status | `reason` | Meaning |
|---|---|---|
| `400` | `invalid_input` | Fewer than 6 hex digits provided |
| `401` | `invalid_key` | The API key sent is malformed, unknown, revoked or expired. Keyless calls are never `401`. |
| `404` | `not_found` | No vendor registered for this MAC or OUI prefix |
| `429` | `rate_limit` | Rate limit hit: 30/minute per IP. Back off for the seconds in `Retry-After`. |
| `429` | `daily_limit` | Rate limit hit: 1,000/day per IP. Resets at midnight UTC. |
| `429` | `credits_exhausted` | Keyed request whose monthly plan allowance is used up. |

Every error uses the shared envelope described in the [repository README](../README.md#errors-and-api-keys):

```bash
curl -s http://127.0.0.1:8107/api/mac/020000
```

```json
{
  "success": false,
  "error": true,
  "reason": "not_found",
  "message": "No vendor found for this MAC in the IEEE registry.",
  "docs": "https://macvendorcheck.com/mac-address-lookup-api",
  "mac": "02:00:00"
}
```

## Notes

- Separators and case are ignored; `00:1a:2b`, `00-1A-2B`, `001a.2b3c.4d5e` and `001A2B` are the same query.
- Longer prefixes are matched most-specific-first (MA-S 36-bit, then MA-M 28-bit, then MA-L 24-bit), matching how the IEEE allocates blocks.
- An unregistered prefix answers `404` (`not_found`), keeping the `mac` field in the error body so you can see what was parsed, for example `curl https://macvendorcheck.com/api/mac/020000` above. Randomized/private MAC addresses (second hex digit 2, 6, A or E) are intentionally unregistered and come back this way.
- This API allows 1,000 requests a day, higher than the 500/day default on the other APIs in this repository.
- No `Cache-Control` header is sent (`cf-cache-status: DYNAMIC`); treat responses as uncached and poll only as often as you need to.
- Optional API key: send `X-Api-Key` or `?key=` from a free api.ipnova.com account and your plan allowance applies instead of the per-IP limits; keyed responses carry `X-Credits-Remaining` and `X-Credits-Used`.
