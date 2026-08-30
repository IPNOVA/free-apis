# Bank Holidays API

Bank and public holiday calendars for 246 countries, four years of data (previous year through two years ahead), refreshed annually. Three endpoints: a full year calendar, the next upcoming holiday, and a specific-date check with a banks-closed verdict. Holiday names are available in five languages.

**Base URL** `https://bankholidaycheck.com` · **Live docs** [bankholidaycheck.com/holiday-api](https://bankholidaycheck.com/holiday-api) · [OpenAPI spec](./openapi.yaml)

## Endpoints

```
GET /api/holidays/{cc}/{year}         # full calendar for a year
GET /api/holidays/next/{cc}           # the next upcoming holiday
GET /api/holidays/check/{cc}/{date}   # is this date a holiday / weekend?
```

| Parameter | Type | Description |
|---|---|---|
| `cc` | path, required | ISO 3166-1 alpha-2 country code, e.g. `GB` |
| `year` | path | Four-digit year within the published window |
| `date` | path | `YYYY-MM-DD` |
| `lang` | query, optional | `en` (default), `es`, `fr`, `de`, `pt` for localized holiday names |

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

| Status | Meaning |
|---|---|
| `400` | Malformed country code, year or date |
| `404` | Country or year not in the dataset |
| `429` | Rate limit hit: 30/minute or 500/day per IP |

## Notes

- The date check treats weekends as banks-closed days too and says which reason applies.
- Dates are generated from the actively maintained [python-holidays](https://github.com/vacanza/holidays) library (MIT) and refreshed annually; for contractual deadlines confirm with official sources.
- Transfers timing tip: for international payments the holidays of *both* countries matter.
