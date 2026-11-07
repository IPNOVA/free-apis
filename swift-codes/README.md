# SWIFT / BIC Lookup API

Identify the bank, branch, city, country and ISO 9362 code breakdown behind any SWIFT/BIC code. Served from a locally hosted dataset of 112,000+ codes across 232 countries and territories; nothing is proxied at request time.

**Base URL** `https://swiftcodecheck.com` · **Live docs** [swiftcodecheck.com/swift-code-api](https://swiftcodecheck.com/swift-code-api) · [OpenAPI spec](./openapi.yaml)

## Endpoints

```
GET /api/swift/{code}        # look up one code
POST /api/swift-batch       # look up up to 100 codes in one call
```

| Parameter | Type | Description |
|---|---|---|
| `code` | path, required | 8 or 11-character SWIFT/BIC code (case-insensitive, spaces and dashes are stripped before validation) |
| `codes` | body, required (batch only) | JSON array of 1 to 100 SWIFT/BIC strings, each 8 or 11 characters: `{"codes": ["DEUTDEFF", "CHASUS33"]}` |

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

### Batch

```bash
curl -X POST https://swiftcodecheck.com/api/swift-batch \
  -H "Content-Type: application/json" \
  -d '{"codes": ["DEUTDEFF", "CHASUS33"]}'
```

```json
{
  "success": true,
  "count": 2,
  "found": 2,
  "results": [
    {
      "code": "DEUTDEFF",
      "found": true,
      "valid_format": true,
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
    {
      "code": "CHASUS33",
      "found": true,
      "valid_format": true,
      "swift_code": "CHASUS33",
      "bank": "Jpmorgan Chase Bank, N.A.",
      "city": "New York,Ny",
      "branch": "",
      "country_code": "US",
      "country": "United States",
      "bank_code": "CHAS",
      "location_code": "33",
      "branch_code": "XXX",
      "head_office": true
    }
  ],
  "attribution": "Data by SwiftCodeCheck.com - free SWIFT/BIC lookup API",
  "docs": "https://swiftcodecheck.com/swift-code-api"
}
```

A malformed item does not fail the call: it comes back as a row with `found: false`, `valid_format: false` and `error: "invalid format"`. A well-formed code that is not in the dataset comes back as `found: false` with `valid_format: true`.

### JavaScript

```js
const res = await fetch('https://swiftcodecheck.com/api/swift/DEUTDEFF');
const { data } = await res.json();
console.log(`${data.bank} in ${data.city}, ${data.country}`);
```

Batch:

```js
const res = await fetch('https://swiftcodecheck.com/api/swift-batch', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ codes: ['DEUTDEFF', 'CHASUS33'] }),
});
const { results } = await res.json();
for (const r of results) console.log(r.code, r.found ? r.bank : r.error ?? 'not in dataset');
```

### Python

```python
import requests

data = requests.get("https://swiftcodecheck.com/api/swift/DEUTDEFF", timeout=10).json()["data"]
print(f"{data['bank']} in {data['city']}, {data['country']}")
```

Batch:

```python
import requests

res = requests.post(
    "https://swiftcodecheck.com/api/swift-batch",
    json={"codes": ["DEUTDEFF", "CHASUS33"]},
    timeout=10,
)
for row in res.json()["results"]:
    print(row["code"], row.get("bank") or row.get("error", "not in dataset"))
```

## Errors

| Status | `reason` | Meaning |
|---|---|---|
| `400` | `invalid_input` | Input is not a valid 8 or 11-character SWIFT/BIC format; or, on the batch endpoint, `codes` missing, empty, not an array, or more than 100 items. |
| `401` | `invalid_key` | The API key sent is malformed, unknown, revoked or expired. Keyless calls are never `401`. |
| `404` | `not_found` | Code has a valid format but is not in the public dataset (response includes `code` and `valid_format: true`). |
| `405` | `method_not_allowed` | `GET` (or any non-`POST` method) on `/api/swift-batch`. |
| `413` | `payload_too_large` | Batch request body over 64 KB. |
| `429` | `rate_limit` | Too many requests: 30/minute per IP. Back off for the seconds in `Retry-After`. |
| `429` | `daily_limit` | The keyless per-IP daily allowance (100/day) is used up. Resets at midnight UTC. |
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
- Batch responses are sent with `Cache-Control: no-store` and `Content-Type: application/json`, since each is call-specific.
- A batch call costs one credit per ten codes, rounded up (`ceil(n / 10)`, based on the size of the submitted `codes` array), and counts against the per-IP daily limit the same way, so a 100-code batch costs 10, not 100. Codes are normalised and de-duplicated before lookup, so `results` and `count` can be shorter than the input array.
- Optional API key: send `X-Api-Key` or `?key=` from a free api.ipnova.com account and your plan allowance applies instead of the per-IP limits; keyed responses carry `X-Credits-Remaining` and `X-Credits-Used`.
