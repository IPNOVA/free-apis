# SWIFT / BIC Lookup API

Identify the bank, branch, city, country and ISO 9362 code breakdown behind any SWIFT/BIC code. Served from a locally hosted dataset of 112,000+ codes across 232 countries and territories; nothing is proxied at request time.

**Base URL** `https://swiftcodecheck.com` · **Live docs** [swiftcodecheck.com/swift-code-api](https://swiftcodecheck.com/swift-code-api) · [OpenAPI spec](./openapi.yaml)

## Endpoint

```
GET /api/swift/{code}
```

| Parameter | Type | Description |
|---|---|---|
| `code` | path, required | 8 or 11-character SWIFT/BIC code (case-insensitive, spaces and dashes are stripped before validation) |

## Example

```bash
curl https://swiftcodecheck.com/api/swift/DEUTDEFF
```

```json
{
  "success": true,
  "data": {
    "swift_code": "DEUTDEFF",
    "bank": "Deutsche Bank AG",
    "city": "Frankfurt Am Main",
    "branch": "",
    "country_code": "DE",
    "country": "Germany",
    "bank_code": "DEUT",
    "location_code": "FF",
    "branch_code": "XXX",
    "head_office": true
  },
  "page": "https://swiftcodecheck.com/swift-code/DEUTDEFF",
  "attribution": "Data by SwiftCodeCheck.com - free SWIFT/BIC lookup API",
  "docs": "https://swiftcodecheck.com/swift-code-api"
}
```

### JavaScript

```js
const res = await fetch('https://swiftcodecheck.com/api/swift/DEUTDEFF');
const { data } = await res.json();
console.log(`${data.bank} in ${data.city}, ${data.country}`);
```

### Python

```python
import requests

data = requests.get("https://swiftcodecheck.com/api/swift/DEUTDEFF", timeout=10).json()["data"]
print(f"{data['bank']} in {data['city']}, {data['country']}")
```

## Errors

| Status | `reason` | Meaning |
|---|---|---|
| `400` | `invalid_input` | Input is not a valid 8 or 11-character SWIFT/BIC format. |
| `401` | `invalid_key` | The API key sent is malformed, unknown, revoked or expired. Keyless calls are never `401`. |
| `404` | `not_found` | Code has a valid format but is not in the public dataset (response includes `code` and `valid_format: true`). |
| `429` | `rate_limit` | Too many requests: 30/minute per IP. Back off for the seconds in `Retry-After`. |
| `429` | `daily_limit` | The keyless per-IP daily allowance (500/day) is used up. Resets at midnight UTC. |
| `429` | `credits_exhausted` | Keyed request whose monthly plan allowance is used up. |

Every error uses the shared envelope described in the [repository README](../README.md#errors-and-api-keys):

```bash
curl https://swiftcodecheck.com/api/swift/ZZZZUS00XXX
```

```json
{
  "success": false,
  "error": true,
  "reason": "not_found",
  "message": "SWIFT code not found in the public dataset. The format is valid - verify the code with the bank.",
  "docs": "https://swiftcodecheck.com/swift-code-api",
  "code": "ZZZZUS00XXX",
  "valid_format": true
}
```

## Notes

- A SWIFT/BIC code identifies a bank or branch, never an individual account. Always advise verifying a code with the receiving bank before a transfer.
- A `404` with `valid_format: true` means the code is structurally valid ISO 9362 but absent from our directory, not proof that the code does not exist.
- Data comes from an openly licensed (MIT) dataset of 112,000+ codes; fields the dataset does not disclose come back as an empty string, never invented.
- Responses are cacheable for 24 hours (`Cache-Control: public, max-age=86400`).
- Optional API key: send `X-Api-Key` or `?key=` from a free api.ipnova.com account and your plan allowance applies instead of the per-IP limits; keyed responses carry `X-Credits-Remaining` and `X-Credits-Used`.
