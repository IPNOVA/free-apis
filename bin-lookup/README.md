# BIN / IIN Lookup API

Identify the network, issuer, country and card type behind the first 6 to 8 digits of any payment card. Served from a locally hosted dataset of 343,000+ BINs; nothing is proxied at request time and full card numbers are never accepted.

**Base URL** `https://cardbincheck.com` · **Live docs** [cardbincheck.com/bin-lookup-api](https://cardbincheck.com/bin-lookup-api) · [OpenAPI spec](./openapi.yaml)

## Endpoint

```
GET /api/bin/{bin}
```

| Parameter | Type | Description |
|---|---|---|
| `bin` | path, required | 6 to 8 digits, the start of a card number |

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

| Status | Meaning |
|---|---|
| `400` | Input is not 6 to 8 digits |
| `404` | BIN not present in the public dataset |
| `429` | Rate limit hit: 30/minute or 500/day per IP |

## Notes

- BIN data identifies card *ranges*, never an individual card or cardholder. It is the same public industry data merchants use for routing and risk.
- Fields the dataset does not disclose come back as `"Unknown"` or an empty string, never invented.
- Responses are cacheable for 24 hours (`Cache-Control: public, max-age=86400`).
