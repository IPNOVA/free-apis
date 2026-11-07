# HS / Tariff Code API

Look up a Harmonized System subheading and see how the United States and the United Kingdom treat it, side by side: US HTS lines with duty rates and statistical suffixes, UK declarable commodity codes, plus the EU Combined Nomenclature line. A second endpoint offers alias-aware search by product name or code prefix. Served from a locally hosted dataset refreshed weekly; nothing is proxied at request time.

**Base URL** `https://tariffcodecheck.com` · **Live docs** [tariffcodecheck.com/hs-code-api](https://tariffcodecheck.com/hs-code-api) · [OpenAPI spec](./openapi.yaml)

## Endpoints

```
GET /api/hs/{code}      # one HS6 subheading: description, US HTS, EU CN, UK commodities
GET /api/search?q=...   # alias-aware search, up to 25 results
POST /api/hs-batch      # check up to 100 HS codes in one call
```

| Parameter | Type | Description |
|---|---|---|
| `code` | path, required | HS digits, 2 to 13 characters, dots allowed (e.g. `851713` or `8517.13`). Codes longer than 6 digits resolve to their HS6 parent; fewer than 6 digits after stripping non-digits returns 404 |
| `q` | query, required on `/api/search` | Product name or code prefix, up to 80 characters |
| `codes` | body, required (batch only) | JSON array of 1 to 100 HS code strings, 6 or more digits each, dots allowed: `{"codes": ["851713", "8471.30"]}` |

## Example

```bash
curl https://tariffcodecheck.com/api/hs/851713
```

```json
{
  "success": true,
  "hs6": "851713",
  "code": "8517.13",
  "description": "Smartphones",
  "chapter": {"number": "85", "title": "Electrical machinery; electronics; sound and TV equipment"},
  "us_hts": [
    {
      "hts": "85171300",
      "description": "Smartphones",
      "context": "",
      "duty_general": "Free",
      "duty_special": "",
      "duty_col2": "35%",
      "units": ["No."],
      "statistical": []
    }
  ],
  "eu_cn": [
    {"cn_code": "85171300", "description": "Smartphones", "taric_url": "https://ec.europa.eu/taxation_customs/dds2/taric/measures.jsp?Lang=en&Taric=8517130000"}
  ],
  "uk_commodities": [
    {"commodity_code": "8517130000", "description": "Smartphones"}
  ],
  "page": "https://tariffcodecheck.com/hs-code/851713",
  "disclaimer": "Reference data from official schedules; not a binding classification ruling.",
  "attribution": "Data: USITC Harmonized Tariff Schedule (public domain) + UK Integrated Online Tariff (OGL v3.0) via TariffCodeCheck.com. Reference only, not a customs ruling.",
  "docs": "https://tariffcodecheck.com/hs-code-api"
}
```

```bash
curl https://tariffcodecheck.com/api/search?q=laptop
```

```json
{
  "success": true,
  "query": "laptop",
  "count": 1,
  "data": [
    {
      "hs6": "847130",
      "code": "8471.30",
      "description": "Portable automatic data processing machines, weighing not more than 10 kg, consisting of at least a central processing unit, a keyboard and a display",
      "page": "https://tariffcodecheck.com/hs-code/847130"
    }
  ],
  "attribution": "Data: USITC Harmonized Tariff Schedule (public domain) + UK Integrated Online Tariff (OGL v3.0) via TariffCodeCheck.com. Reference only, not a customs ruling."
}
```

### Batch

```bash
curl -X POST https://tariffcodecheck.com/api/hs-batch \
  -H "Content-Type: application/json" \
  -d '{"codes": ["851713", "8471.30"]}'
```

```json
{
  "success": true,
  "count": 2,
  "found": 2,
  "results": [
    {
      "input": "851713",
      "hs6": "851713",
      "code": "8517.13",
      "found": true,
      "description": "Smartphones",
      "chapter": {
        "number": "85",
        "title": "Electrical machinery; electronics; sound and TV equipment"
      },
      "page": "https://tariffcodecheck.com/hs-code/851713"
    },
    {
      "input": "8471.30",
      "hs6": "847130",
      "code": "8471.30",
      "found": true,
      "description": "Portable automatic data processing machines, weighing not more than 10 kg, consisting of at least a central processing unit, a keyboard and a display",
      "chapter": {
        "number": "84",
        "title": "Machinery and mechanical appliances; computers"
      },
      "page": "https://tariffcodecheck.com/hs-code/847130"
    }
  ],
  "disclaimer": "Reference data from official schedules; not a binding classification ruling.",
  "attribution": "Data: USITC Harmonized Tariff Schedule (public domain) + UK Integrated Online Tariff (OGL v3.0) via TariffCodeCheck.com. Reference only, not a customs ruling.",
  "docs": "https://tariffcodecheck.com/hs-code-api"
}
```

Each code is resolved to its HS6 parent, as on `/api/hs/{code}`. A code that cannot be resolved does not fail the call: it comes back as a row with `found: false` and `error: "invalid format"`. The batch rows carry the HS6 description and chapter only; call `/api/hs/{code}` for the US HTS, EU CN and UK detail.

### JavaScript

```js
const res = await fetch('https://tariffcodecheck.com/api/hs/851713');
const j = await res.json();
console.log(`${j.code} ${j.description}: US general duty ${j.us_hts[0].duty_general}`);
```

Batch:

```js
const res = await fetch('https://tariffcodecheck.com/api/hs-batch', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ codes: ['851713', '8471.30'] }),
});
const { results } = await res.json();
for (const r of results) console.log(r.input, r.found ? `${r.code} ${r.description}` : r.error);
```

### Python

```python
import requests

data = requests.get("https://tariffcodecheck.com/api/hs/851713", timeout=10).json()
print(f"{data['code']} {data['description']}: US general duty {data['us_hts'][0]['duty_general']}")
```

Batch:

```python
import requests

res = requests.post(
    "https://tariffcodecheck.com/api/hs-batch",
    json={"codes": ["851713", "8471.30"]},
    timeout=10,
)
for row in res.json()["results"]:
    print(row["input"], row.get("description") or row.get("error"))
```

## Errors

| Status | `reason` | Meaning |
|---|---|---|
| `400` | `invalid_input` | `/api/search` called without a `q` parameter, or `q` is empty; or, on the batch endpoint, `codes` missing, empty, not an array, or more than 100 items. |
| `401` | `invalid_key` | The API key sent is malformed, unknown, revoked or expired. Keyless calls are never `401`. |
| `404` | `not_found` | Code is not a resolvable HS6 subheading (fewer than 6 digits after stripping non-digits, or not in the dataset). |
| `405` | `method_not_allowed` | `GET` (or any non-`POST` method) on `/api/hs-batch`. |
| `413` | `payload_too_large` | Batch request body over 64 KB. |
| `429` | `rate_limit` | Too many requests: 30/minute per IP. Back off for the seconds in `Retry-After`. |
| `429` | `daily_limit` | The keyless per-IP daily allowance (100/day) is used up. Resets at midnight UTC. |
| `429` | `credits_exhausted` | Keyed request whose monthly plan allowance is used up. |

Every error uses the shared envelope described in the [repository README](../README.md#errors-and-api-keys):

```bash
curl https://tariffcodecheck.com/api/hs/0000.00
```

```json
{
  "success": false,
  "error": true,
  "reason": "not_found",
  "message": "Unknown HS code. Provide 6 digits: /api/hs/851713 (longer codes are truncated to their HS6 parent).",
  "docs": "https://tariffcodecheck.com/hs-code-api"
}
```

## Notes

- US HTS codes, descriptions and duty rates: U.S. International Trade Commission, Harmonized Tariff Schedule, public domain (US government work).
- EU Combined Nomenclature codes and descriptions: Eurostat CN vocabulary via data.europa.eu (© European Union), reuse under Commission Decision 2011/833/EU with attribution; EU duty rates are consulted on the official TARIC service.
- UK commodity codes and descriptions: UK Integrated Online Tariff (HMRC), contains public sector information licensed under the Open Government Licence v3.0.
- The dataset is reference information from the official schedules, refreshed weekly. It is not a binding classification ruling; the legally binding classification is the one made by the customs authority of the importing country. Duty surcharges (for example US Section 301) may apply on top of the listed rates.
- Keep the `attribution` field when you republish results.
- Responses are cacheable for an hour (`Cache-Control: public, max-age=3600`).
- Batch responses are sent with `Cache-Control: no-store` and `Content-Type: application/json`, since each is call-specific.
- A batch call costs one credit per ten codes, rounded up (`ceil(n / 10)`, based on the size of the submitted `codes` array), and counts against the per-IP daily limit the same way, so a 100-code batch costs 10, not 100. Codes are not de-duplicated: `results` has one row per submitted code, in order.
- Optional API key: send `X-Api-Key` or `?key=` from a free api.ipnova.com account and your plan allowance applies instead of the per-IP limits; keyed responses carry `X-Credits-Remaining` and `X-Credits-Used`.
