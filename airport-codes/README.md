# Airport Code & Routes API

Look up any IATA or ICAO airport code, list its direct-flight destinations, find the nonstop or best one-stop connection between two airports, search by name or city, and list countries by airport count. Airport records are served from a locally hosted OurAirports dataset; route data is adapted from Wikipedia and refreshed monthly on our own servers, so nothing is queried from third parties at lookup time.

**Base URL** `https://airportcodecheck.com` · **Live docs** [airportcodecheck.com/airport-api](https://airportcodecheck.com/airport-api) · [OpenAPI spec](./openapi.yaml)

## Endpoints

```
GET /api/airport/{code}            # airport by IATA or ICAO code
GET /api/routes/{code}             # direct destinations from that airport
GET /api/connections/{from}/{to}   # nonstop route, or best one-stop options
GET /api/search?q=...              # airport search by name, city or IATA code
GET /api/countries                 # countries with airport counts
```

| Parameter | Type | Description |
|---|---|---|
| `code` | path, required | IATA (3-letter) or ICAO (4-letter) airport code, 2 to 12 characters |
| `from`, `to` | path, required | IATA or ICAO airport code, 2 to 12 characters |
| `q` | query, required for search | 2 or more characters, matched against name, city and IATA code |

## Example

```bash
curl https://airportcodecheck.com/api/airport/FNC
```

```json
{
  "airport": {
    "slug": "fnc",
    "name": "Cristiano Ronaldo International Airport",
    "city": "Funchal",
    "country": "PT",
    "country_name": "Portugal",
    "iata": "FNC",
    "icao": "LPMA",
    "type": "large_airport",
    "scheduled_service": true,
    "lat": 32.69781,
    "lon": -16.77461,
    "elevation_ft": 192,
    "url": "https://airportcodecheck.com/airport/fnc"
  },
  "source": "OurAirports (public domain) + Wikipedia (CC BY-SA) via airportcodecheck.com"
}
```

### JavaScript

```js
const res = await fetch('https://airportcodecheck.com/api/routes/FNC');
const j = await res.json();
for (const d of j.destinations) {
  console.log(`${d.iata} ${d.city}: ${d.airlines.join(', ')} (${d.distance_km} km)`);
}
```

### Python

```python
import requests

j = requests.get("https://airportcodecheck.com/api/connections/FNC/JFK", timeout=10).json()
if j["nonstop"]:
    print("Nonstop:", ", ".join(j["nonstop"]["airlines"]))
else:
    best = j["one_stop"][0]
    print(f"Best one-stop via {best['via']['iata']}: {best['total_km']} km")
```

## Errors

| Status | Meaning |
|---|---|
| `400` | Search query (`q`) shorter than 2 characters |
| `404` | Unknown airport code, on a lookup, a routes request, or either side of a connection |
| `429` | Rate limit hit: 30/minute or 500/day per IP |

## Notes

- Airport records (names, codes, coordinates, elevation) come from [OurAirports](https://ourairports.com), a public-domain dataset. Direct-flight destination lists are adapted from Wikipedia's airline and destination tables, available under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/); adaptations of that route data stay under the same license.
- A route entry means a nonstop service exists on that pair, with the airlines flying it and a seasonal flag; it is not a live schedule, so confirm timings with the airline.
- `/api/connections` returns the nonstop service when one exists; otherwise up to 8 one-stop options via a third airport, ranked by total great-circle distance.
- On every endpoint except search, `country` is the ISO 3166-1 alpha-2 code and `country_name` is the full name. On search, `country` is the full country name, there is no separate code field.
- `distance_km` and `total_km` are great-circle (straight-line) distances, not flown distances.
- Responses are cacheable (`Cache-Control: public, max-age=86400`, and `max-age=3600` for search).
