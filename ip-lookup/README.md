# IP Intelligence API

Geolocation, ASN and network-risk signals for any IPv4 or IPv6 address: country, region, city, ISP, plus VPN, datacenter and Tor detection from curated threat lists. Served from locally hosted data (DB-IP Lite plus daily-refreshed lists), so lookups are instant and never leave our infrastructure.

**Base URL** `https://ipsnapshot.com` · **Live docs** [ipsnapshot.com/ip-lookup-api](https://ipsnapshot.com/ip-lookup-api) · [OpenAPI spec](./openapi.yaml)

## Endpoint

```
GET /api/ip/{ip}
GET /api/ip/self      # resolves the caller's own address
```

| Parameter | Type | Description |
|---|---|---|
| `ip` | path, required | An IPv4 or IPv6 address, or the literal `self` |

## Example

```bash
curl https://ipsnapshot.com/api/ip/8.8.8.8
```

```json
{
  "success": true,
  "ip": "8.8.8.8",
  "data": {
    "ip_version": 4,
    "country_code": "US",
    "country": "United States",
    "region": "California",
    "city": "Mountain View",
    "continent": "North America",
    "asn": 15169,
    "isp": "Google LLC",
    "is_vpn": false,
    "is_tor": false,
    "is_datacenter": true,
    "risk_level": "Clean",
    "risk_score": 15
  },
  "page": "https://ipsnapshot.com/ip/8.8.8.8",
  "attribution": "Data by IPSnapshot.com - free IP lookup API. Geo/ASN: DB-IP Lite (CC BY 4.0).",
  "docs": "https://ipsnapshot.com/ip-lookup-api"
}
```

### JavaScript

```js
const res = await fetch('https://ipsnapshot.com/api/ip/self');
const { ip, data } = await res.json();
console.log(`${ip} is in ${data.city ?? data.country}, VPN: ${data.is_vpn}`);
```

### Python

```python
import requests

r = requests.get("https://ipsnapshot.com/api/ip/1.1.1.1", timeout=10).json()
print(r["data"]["isp"], r["data"]["is_datacenter"])
```

## Errors

| Status | `reason` | Meaning |
|---|---|---|
| `400` | `invalid_input` | Not a valid IPv4 or IPv6 address |
| `401` | `invalid_key` | The API key sent is malformed, unknown, revoked or expired. Keyless calls are never `401`. |
| `429` | `rate_limit` | Rate limit hit: 30/minute per IP. Back off for the seconds in `Retry-After`. |
| `429` | `daily_limit` | Rate limit hit: 500/day per IP. Resets at midnight UTC. |
| `429` | `credits_exhausted` | Keyed request whose monthly plan allowance is used up. |

Every error uses the shared envelope described in the [repository README](../README.md#errors-and-api-keys):

```bash
curl -s http://127.0.0.1:8102/api/ip/999.999.999.999
```

```json
{
  "success": false,
  "error": true,
  "reason": "invalid_input",
  "message": "Invalid IP. Provide an IPv4 or IPv6 address: /api/ip/8.8.8.8 (or /api/ip/self)",
  "docs": "https://ipsnapshot.com/ip-lookup-api"
}
```

## Notes

- `risk_score` (0 to 100) and `risk_level` combine the VPN / datacenter / Tor signals into one number; treat them as advisory, not as a verdict.
- City-level accuracy varies by provider and region, as with all IP geolocation.
- Attribution for the geo/ASN layer: [DB-IP Lite](https://db-ip.com) under CC BY 4.0.
- Responses are cacheable for one hour (`Cache-Control: public, max-age=3600`).
- Optional API key: send `X-Api-Key` or `?key=` from a free api.ipnova.com account and your plan allowance applies instead of the per-IP limits; keyed responses carry `X-Credits-Remaining` and `X-Credits-Used`.
