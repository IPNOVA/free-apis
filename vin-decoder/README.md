# VIN Decoder API

Decode a Vehicle Identification Number into make, model, year, engine, plant and body facts, sourced from the NHTSA vPIC database (US public domain). Each VIN is decoded upstream once, ever, then served from a permanent local cache. The response links to the site's specs and recalls page for the exact model when one exists.

**Base URL** `https://vindecodercheck.com` · **Live docs** [vindecodercheck.com/vin-api](https://vindecodercheck.com/vin-api) · [OpenAPI spec](./openapi.yaml)

## Endpoint

```
GET /api/vin/{vin}
```

| Parameter | Type | Description |
|---|---|---|
| `vin` | path, required | 11 to 17 characters, letters and digits only, case-insensitive. VINs never contain the letters I, O or Q. A full 17-character VIN decodes completely; an 11 to 16-character partial VIN decodes to the extent the World Manufacturer Identifier and descriptor allow. |

## Example

```bash
curl https://vindecodercheck.com/api/vin/1HGCM82633A004352
```

```json
{
  "vin": "1HGCM82633A004352",
  "make": "HONDA",
  "model": "Accord",
  "model_year": "2003",
  "manufacturer": "AMERICAN HONDA MOTOR CO., INC.",
  "plant_city": "MARYSVILLE",
  "plant_country": "UNITED STATES (USA)",
  "vehicle_type": "PASSENGER CAR",
  "body_class": "Coupe",
  "engine_cylinders": "6",
  "displacement_l": "2.998832712",
  "fuel_type_primary": "Gasoline",
  "transmission_style": "Automatic",
  "doors": "2",
  "g_v_w_r": "Class 1C: 4,001 - 5,000 lb (1,814 - 2,268 kg)",
  "trim": "EX-V6",
  "error_text": "0 - VIN decoded clean. Check Digit (9th position) is correct",
  "specs_page": "https://vindecodercheck.com/car/2003-honda-accord",
  "recalls_on_record": 15,
  "source": "NHTSA vPIC (US public domain)",
  "docs": "https://vindecodercheck.com/vin-api"
}
```

### JavaScript

```js
const res = await fetch('https://vindecodercheck.com/api/vin/1HGCM82633A004352');
const car = await res.json();
console.log(`${car.model_year} ${car.make} ${car.model}`);
```

### Python

```python
import requests

car = requests.get("https://vindecodercheck.com/api/vin/1HGCM82633A004352", timeout=10).json()
print(f"{car['model_year']} {car['make']} {car['model']}")
```

## Errors

| Status | Meaning |
|---|---|
| `422` | VIN is not 11 to 17 characters, or contains I, O or Q |
| `429` | Rate limit hit: 20/minute or 300/day per IP |
| `502` | Upstream vPIC decoder unavailable (only on an uncached VIN's first decode), try again shortly |

## Notes

- Each VIN is decoded against NHTSA vPIC upstream once, ever. After that first decode it serves from our permanent cache, so repeated integration calls are fast and place no further load on the government source.
- This API's limits are lower than IPNOVA's other free APIs (20/minute, 300/day) because a VIN that has never been seen before still calls NHTSA vPIC live at request time; the cap protects that upstream government service from uncached traffic.
- Responses are `no-store` and VINs are never logged. Fields the decode does not return (a "Not Applicable" upstream value) are omitted, never invented.
- Data sources: NHTSA vPIC (US public domain) for the decode itself. The linked `specs_page`, when present, adds EPA fueleconomy.gov figures and the NHTSA recalls database (both US public domain), refreshed monthly.
- Build facts only: no ownership, title or accident data, ever.
