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

| Status | Meaning |
|---|---|
| `400` | Fewer than 6 hex digits provided |
| `429` | Rate limit hit: 30/minute or 1,000/day per IP |

## Notes

- Separators and case are ignored; `00:1a:2b`, `00-1A-2B`, `001a.2b3c.4d5e` and `001A2B` are the same query.
- Longer prefixes are matched most-specific-first (MA-S 36-bit, then MA-M 28-bit, then MA-L 24-bit), matching how the IEEE allocates blocks.
- An unregistered prefix is not a `404`: the API answers `200` with `success: false` and an explanatory `error`, for example `curl https://macvendorcheck.com/api/mac/020000` returns `{"success":false,"mac":"02:00:00","error":"No vendor found for this MAC in the IEEE registry."}`. Randomized/private MAC addresses (second hex digit 2, 6, A or E) are intentionally unregistered and come back this way.
- This API allows 1,000 requests a day, higher than the 500/day default on the other APIs in this repository.
- No `Cache-Control` header is sent (`cf-cache-status: DYNAMIC`); treat responses as uncached and poll only as often as you need to.
