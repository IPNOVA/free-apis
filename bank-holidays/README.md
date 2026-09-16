# Bank Holidays API

Bank and public holiday calendars for 150+ countries, four years of data (previous year through two years ahead), refreshed annually. Four endpoints: a full year calendar, the next upcoming holiday, a specific-date check with a banks-closed verdict, and a business-days calculator between two dates. Holiday names are available in eight languages.

**Base URL** `https://bankholidaycheck.com` · **Live docs** [bankholidaycheck.com/holiday-api](https://bankholidaycheck.com/holiday-api) · [OpenAPI spec](./openapi.yaml)

## Endpoints

```
GET /api/holidays/{cc}/{year}                  # full calendar for a year
GET /api/holidays/next/{cc}                    # the next upcoming holiday
GET /api/holidays/check/{cc}/{date}            # is this date a holiday / weekend?
GET /api/business-days/{cc}/{from}/{to}        # business days between two dates, holidays excluded
```

| Parameter | Type | Description |
|---|---|---|
| `cc` | path, required | ISO 3166-1 alpha-2 country code, e.g. `GB` |
| `year` | path | Four-digit year within the published window |
| `date` | path | `YYYY-MM-DD` |
| `from`, `to` | path | `YYYY-MM-DD`, business-days endpoint only; inclusive, any order, up to 5 years apart |
| `lang` | query, optional | `en` (default), `es`, `fr`, `de`, `pt`, `it`, `nl` or `pl` for localized holiday names |

## Example

```bash
curl https://bankholidaycheck.com/api/holidays/next/GB
```

```json
{
  "success": true,
  "country": "United Kingdom",
  "country_code": "GB",
  "data": {
    "date": "2026-08-31",
    "name": "Late Summer Bank Holiday",
    "days": 1
  },
  "page": "https://bankholidaycheck.com/bank-holidays/united-kingdom",
  "attribution": "Data by BankHolidayCheck.com - generated with the python-holidays library (MIT).",
  "docs": "https://bankholidaycheck.com/holiday-api"
}
```

### Business days

```bash
curl https://bankholidaycheck.com/api/business-days/DE/2026-01-01/2026-03-31
```

```json
{
  "success": true,
  "country": "Germany",
  "country_code": "DE",
  "from": "2026-01-01",
  "to": "2026-03-31",
  "inclusive": true,
  "data": {
    "business_days": 63,
    "total_days": 90,
    "weekend_days": 26,
    "holidays_skipped": [
      { "date": "2026-01-01", "name": "New Year's Day" }
    ],
    "weekend": "Sat-Sun weekend"
  },
  "page": "https://bankholidaycheck.com/business-days-calculator",
  "attribution": "Data by BankHolidayCheck.com - generated with the python-holidays library (MIT).",
  "docs": "https://bankholidaycheck.com/holiday-api"
}
```

### JavaScript

```js
const res = await fetch('https://bankholidaycheck.com/api/holidays/check/DE/2026-10-03');
const j = await res.json();
console.log(j.data.banks_closed ? `Banks closed: ${j.data.holiday_name ?? j.data.weekday}` : 'Banks open');
```

### Python

```python
import requests

year = requests.get("https://bankholidaycheck.com/api/holidays/US/2026", timeout=10).json()
for day in year["data"]:
    print(day["date"], day["name"])
```

## Errors

| Status | `reason` | Meaning |
|---|---|---|
| `400` | `invalid_input` | Malformed country code, year, date, or `from`/`to` range (invalid dates, or over 5 years apart) |
| `401` | `invalid_key` | The API key sent is malformed, unknown, revoked or expired. Keyless calls are never `401`. |
| `404` | `not_found` | Country or year not in the dataset, or (on the `next` endpoint) no upcoming holiday data for the country |
| `429` | `rate_limit` | Rate limit hit: 30/minute per IP. Back off for the seconds in `Retry-After`. |
| `429` | `daily_limit` | Rate limit hit: 500/day per IP. Resets at midnight UTC. |
| `429` | `credits_exhausted` | Keyed request whose monthly plan allowance is used up. |

Every error uses the shared envelope described in the [repository README](../README.md#errors-and-api-keys):

```bash
curl -s http://127.0.0.1:8106/api/holidays/GB/2030
```

```json
{
  "success": false,
  "error": true,
  "reason": "not_found",
  "message": "No data for United Kingdom in 2030. Years available: 2025, 2026, 2027, 2028.",
  "docs": "https://bankholidaycheck.com/holiday-api"
}
```

## Notes

- The date check treats weekends as banks-closed days too and says which reason applies.
- Dates are generated from the actively maintained [python-holidays](https://github.com/vacanza/holidays) library (MIT) and refreshed annually; for contractual deadlines confirm with official sources.
- Transfers timing tip: for international payments the holidays of *both* countries matter.
- The business-days endpoint counts weekdays between `from` and `to` inclusive, minus any holiday in `data.holidays_skipped`; the date range accepts either order and is capped at 5 years.
- Responses are cacheable (`Cache-Control: public, max-age=3600`).
- Optional API key: send `X-Api-Key` or `?key=` from a free api.ipnova.com account and your plan allowance applies instead of the per-IP limits; keyed responses carry `X-Credits-Remaining` and `X-Credits-Used`.
