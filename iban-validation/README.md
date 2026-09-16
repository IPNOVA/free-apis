# IBAN Validation API

Validate any IBAN and get back its country, length check, ISO 7064 mod-97 checksum result and domestic bank code. Validation is purely algorithmic against the ISO 13616 registry: nothing is proxied at request time, and IBANs are never stored or logged.

**Base URL** `https://ibancodecheck.com` · **Live docs** [ibancodecheck.com/iban-api](https://ibancodecheck.com/iban-api) · [OpenAPI spec](./openapi.yaml)

## Endpoint

```
GET /api/iban/{iban}
```

| Parameter | Type | Description |
|---|---|---|
| `iban` | path, required | An IBAN, letters and digits only in the URL; strip spaces and dashes before calling. 5 to 34 characters once normalized |

## Example

```bash
curl https://ibancodecheck.com/api/iban/DE89370400440532013000
```

```json
{
  "success": true,
  "data": {
    "iban": "DE89370400440532013000",
    "formatted": "DE89 3704 0044 0532 0130 00",
    "valid": true,
    "country_code": "DE",
    "country": "Germany",
    "sepa": true,
    "length_ok": true,
    "checksum_ok": true,
    "check_digits": "89",
    "bank_code": "37040044",
    "error": null
  },
  "page": "https://ibancodecheck.com/iban-format/DE",
  "attribution": "Validation by IbanCodeCheck.com - free IBAN checker API. IBANs are never stored or logged.",
  "docs": "https://ibancodecheck.com/iban-api"
}
```

A structurally wrong IBAN still returns `200`, with `valid: false` and a human-readable `error`, for example an IBAN with a typo in the check digits:

```json
{
  "success": true,
  "data": {
    "iban": "DE89370400440532013001",
    "formatted": "DE89 3704 0044 0532 0130 01",
    "valid": false,
    "country_code": "DE",
    "country": "Germany",
    "sepa": true,
    "length_ok": true,
    "checksum_ok": false,
    "check_digits": "89",
    "bank_code": "",
    "error": "The check digits do not match: there is a typo somewhere in this IBAN."
  },
  "page": "https://ibancodecheck.com/iban-format/DE",
  "attribution": "Validation by IbanCodeCheck.com - free IBAN checker API. IBANs are never stored or logged.",
  "docs": "https://ibancodecheck.com/iban-api"
}
```

### JavaScript

```js
const res = await fetch('https://ibancodecheck.com/api/iban/DE89370400440532013000');
const { data } = await res.json();
console.log(data.valid ? `Valid ${data.country} IBAN, bank code ${data.bank_code}` : data.error);
```

### Python

```python
import requests

data = requests.get("https://ibancodecheck.com/api/iban/DE89370400440532013000", timeout=10).json()["data"]
print(f"{data['formatted']} -> valid: {data['valid']}, SEPA: {data['sepa']}")
```

## Errors

| Status | `reason` | Meaning |
|---|---|---|
| `400` | `invalid_input` | Input is not 5 to 34 characters once normalized (the URL path only accepts letters and digits, so a dashed or spaced IBAN never reaches this check; it 404s at the route level instead). |
| `401` | `invalid_key` | The API key sent is malformed, unknown, revoked or expired. Keyless calls are never `401`. |
| `429` | `rate_limit` | Too many requests: 30/minute per IP. Back off for the seconds in `Retry-After`. |
| `429` | `daily_limit` | The keyless per-IP daily allowance (500/day) is used up. Resets at midnight UTC. |
| `429` | `credits_exhausted` | Keyed request whose monthly plan allowance is used up. |

Every error uses the shared envelope described in the [repository README](../README.md#errors-and-api-keys):

```bash
curl https://ibancodecheck.com/api/iban/XX00
```

```json
{
  "success": false,
  "error": true,
  "reason": "invalid_input",
  "message": "Provide an IBAN of 5 to 34 characters: /api/iban/DE89370400440532013000",
  "docs": "https://ibancodecheck.com/iban-api"
}
```

## Notes

- Validation is algorithmic only: country code and length against the ISO 13616 registry, then the ISO 7064 mod-97 checksum. It confirms the IBAN is structurally correct with no typos, never that the account exists or is open.
- An unknown country code, a wrong length for the country, or a failed checksum still return `200` with `valid: false` and the reason in `data.error`, so a form can show a helpful message. Only input outside the 5 to 34 character range returns `400`.
- `bank_code` is filled in only for countries where the registry defines a fixed position for it; it comes back empty otherwise.
- Responses are sent with `Cache-Control: no-store`. IBANs are never written to disk or logged.
- 78 country IBAN formats are covered by the registry.
- Optional API key: send `X-Api-Key` or `?key=` from a free api.ipnova.com account and your plan allowance applies instead of the per-IP limits; keyed responses carry `X-Credits-Remaining` and `X-Credits-Used`.
