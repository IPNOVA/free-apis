<div align="center">

# IPNOVA Free APIs

**Twelve production JSON APIs. No key. No registration. CORS enabled.**

[ipnova.com](https://ipnova.com) · [api.ipnova.com](https://api.ipnova.com) · [Report an issue](https://github.com/IPNOVA/free-apis/issues) · [hello@ipnova.com](mailto:hello@ipnova.com)

</div>

---

Every API below runs on its own locally hosted dataset on our infrastructure. No request is proxied to a third party at lookup time (the one exception, a first-time VIN decode, is explained on that API's page), responses are cacheable, and nothing you look up is stored in any profile.

| API | Endpoint | Docs | Data source |
|---|---|---|---|
| [BIN / IIN Lookup](./bin-lookup/) | `GET cardbincheck.com/api/bin/{bin}` | [live docs](https://cardbincheck.com/bin-lookup-api) | 343k+ BIN dataset |
| [IP Intelligence](./ip-lookup/) | `GET ipsnapshot.com/api/ip/{ip}` | [live docs](https://ipsnapshot.com/ip-lookup-api) | DB-IP Lite (CC BY 4.0) + curated threat lists |
| [SWIFT / BIC Lookup](./swift-codes/) | `GET swiftcodecheck.com/api/swift/{code}` | [live docs](https://swiftcodecheck.com/swift-code-api) | Open SWIFT/BIC dataset (MIT), 112k codes |
| [IBAN Validation](./iban-validation/) | `GET ibancodecheck.com/api/iban/{iban}` | [live docs](https://ibancodecheck.com/iban-api) | Algorithmic, 78 national formats |
| [Bank Holidays](./bank-holidays/) | `GET bankholidaycheck.com/api/holidays/{cc}/{year}` and 3 more | [live docs](https://bankholidaycheck.com/holiday-api) | python-holidays (MIT), 150+ countries |
| [MAC Vendor Lookup](./mac-vendor/) | `GET macvendorcheck.com/api/mac/{mac}` | [live docs](https://macvendorcheck.com/mac-address-lookup-api) | IEEE MA-L / MA-M / MA-S registry |
| [Postal Code Lookup](./postal-codes/) | `GET postalcodecheck.com/api/postal/{cc}/{code}` and 1 more | [live docs](https://postalcodecheck.com/postal-code-api) | GeoNames (CC BY 4.0), 887k codes |
| [Phone / Area Codes](./phone-area-codes/) | `GET areacodecheck.com/api/phone/{number}` | [live docs](https://areacodecheck.com/phone-api) | ITU / NANPA assignments, libphonenumber (Apache 2.0) |
| [Airport Codes and Routes](./airport-codes/) | `GET airportcodecheck.com/api/airport/{code}` and 4 more | [live docs](https://airportcodecheck.com/airport-api) | OurAirports (public domain), Wikipedia route tables (CC BY-SA) |
| [HS / Tariff Codes](./hs-tariff-codes/) | `GET tariffcodecheck.com/api/hs/{code}` and 1 more | [live docs](https://tariffcodecheck.com/hs-code-api) | US HTS (public domain), EU Combined Nomenclature (Eurostat), UK Tariff (OGL v3) |
| [Disposable Email Domains](./disposable-email/) | `GET emaildomaincheck.com/api/email-domain/{domain}` | [live docs](https://emaildomaincheck.com/email-api) | Four open blocklists (CC0 / MIT), 76k domains |
| [VIN Decoder](./vin-decoder/) | `GET vindecodercheck.com/api/vin/{vin}` | [live docs](https://vindecodercheck.com/vin-api) | NHTSA vPIC and recalls, EPA fuel economy (US public data) |

## Quickstart

```bash
# Which bank issued this card prefix?
curl https://cardbincheck.com/api/bin/440066

# Where is this IP, and is it a VPN or datacenter?
curl https://ipsnapshot.com/api/ip/8.8.8.8

# Which bank and branch is behind this SWIFT code?
curl https://swiftcodecheck.com/api/swift/DEUTDEFF

# Is this IBAN well formed?
curl https://ibancodecheck.com/api/iban/DE89370400440532013000

# When is the next bank holiday in the UK?
curl https://bankholidaycheck.com/api/holidays/next/GB

# Who manufactured this network device?
curl https://macvendorcheck.com/api/mac/00:1A:2B:3C:4D:5E

# Which city is ZIP 90210?
curl https://postalcodecheck.com/api/postal/US/90210

# Which region is this phone number from?
curl https://areacodecheck.com/api/phone/+14155552671

# Which airport is LHR, and where does it fly direct?
curl https://airportcodecheck.com/api/airport/LHR

# What is HS code 8504.40, and what duty applies?
curl https://tariffcodecheck.com/api/hs/850440

# Is this email domain disposable?
curl https://emaildomaincheck.com/api/email-domain/mailinator.com

# What vehicle is this VIN?
curl https://vindecodercheck.com/api/vin/1HGCM82633A004352
```

Each directory in this repository contains the full endpoint reference, an OpenAPI 3.1 specification, and copy-paste examples for curl, JavaScript and Python.

## Fair use

The same limits apply to every API unless its own page says otherwise:

- **500 requests per day** and **30 per minute**, per IP. The VIN decoder allows 300 per day and 20 per minute because first-time decodes reach the NHTSA service. The MAC vendor lookup allows 1,000 per day.
- Responses are cacheable (respect the `Cache-Control` headers and cache on your side where you can).
- Free for personal and commercial use. Attribution is appreciated: a link to the API's website.
- Need more volume? Plans with higher limits and one key for all twelve APIs open shortly at **[api.ipnova.com](https://api.ipnova.com)**; a free account gets you in first. Genuine open-source and research projects can write to **hello@ipnova.com**.

## Errors and API keys

Every API answers the same way, so you learn the pattern once.

- Every `200` body starts with `"success": true`. The data fields that follow are specific to each API and documented in its folder.
- Every non-`200` body is the same envelope, sometimes followed by one or two extra fields:

```json
{
  "success": false,
  "error": true,
  "reason": "not_found",
  "message": "BIN not found in the public dataset.",
  "docs": "https://cardbincheck.com/bin-lookup-api"
}
```

| Status | `reason` | Meaning |
|---|---|---|
| `400` | `invalid_input` | Malformed input. Fix the request. |
| `401` | `invalid_key` | The API key sent is malformed, unknown, revoked or expired. Keyless calls are never `401`. |
| `404` | `not_found` | Valid query, no record. |
| `405` | `method_not_allowed` | Wrong HTTP method for the endpoint. |
| `413` | `payload_too_large` | Batch body over the documented maximum. |
| `429` | `rate_limit` | Too many requests per minute. Back off for the seconds in `Retry-After`. |
| `429` | `daily_limit` | The keyless per-IP daily allowance is used up. Resets at midnight UTC. |
| `429` | `credits_exhausted` | A keyed request whose monthly plan allowance is used up. |
| `502` | `upstream_unavailable` | A source the API depends on did not answer. Retry with backoff. |

**API keys are optional.** A free account at [api.ipnova.com](https://api.ipnova.com) gives you one key for all twelve APIs and a usage dashboard. Send it as the `X-Api-Key` header or the `?key=` query parameter and your plan allowance applies instead of the per-IP limits. Keyed `200` responses carry `X-Credits-Remaining` and `X-Credits-Used`; only `200` responses consume credits. If the key service cannot be reached, the call is served under the keyless limits with `X-Credits-Remaining: unavailable`, so a key never makes an integration less reliable than no key.

## Reliability

The APIs serve production traffic behind Cloudflare with local datasets refreshed on schedule (daily to monthly depending on the source). Breaking changes are avoided; if one is ever required it will be announced in this repository's releases first.

## Who is behind this

These APIs are built and operated by [IPNOVA SYSTEMS LTD](https://ipnova.com), a software company in Larnaca, Cyprus (Reg. HE 479674). We build free, data-driven utility platforms and keep them fast, honest and ad-free.

## License

The documentation and examples in this repository are [MIT licensed](./LICENSE). The underlying datasets keep their own licenses, credited per API above and, where a response carries one, in its `attribution` or `source` field.
