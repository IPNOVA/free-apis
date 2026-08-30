<div align="center">

# IPNOVA Free APIs

**Six production JSON APIs. No key. No registration. CORS enabled.**

[ipnova.com](https://ipnova.com) · [Report an issue](https://github.com/IPNOVA/free-apis/issues) · [hello@ipnova.com](mailto:hello@ipnova.com)

</div>

---

Every API below runs on its own locally hosted dataset on our infrastructure. No request is proxied to a third party at lookup time, responses are cacheable, and nothing you look up is stored in any profile.

| API | Endpoint | Docs | Data source |
|---|---|---|---|
| [BIN / IIN Lookup](./bin-lookup/) | `GET cardbincheck.com/api/bin/{bin}` | [live docs](https://cardbincheck.com/bin-lookup-api) | 343k+ BIN dataset |
| [IP Intelligence](./ip-lookup/) | `GET ipsnapshot.com/api/ip/{ip}` | [live docs](https://ipsnapshot.com/ip-lookup-api) | DB-IP Lite (CC BY 4.0) + curated threat lists |
| [Bank Holidays](./bank-holidays/) | `GET bankholidaycheck.com/api/holidays/{cc}/{year}` | [live docs](https://bankholidaycheck.com/holiday-api) | python-holidays (MIT), 246 countries |
| [MAC Vendor Lookup](./mac-vendor/) | `GET macvendorcheck.com/api/mac/{mac}` | [live docs](https://macvendorcheck.com/mac-address-lookup-api) | IEEE MA-L / MA-M / MA-S registry |
| [Postal Code Lookup](./postal-codes/) | `GET postalcodecheck.com/api/postal/{cc}/{code}` | [live docs](https://postalcodecheck.com/postal-code-api) | GeoNames (CC BY 4.0), 887k codes |
| [Phone / Area Codes](./phone-area-codes/) | `GET areacodecheck.com/api/phone/{number}` | [live docs](https://areacodecheck.com/phone-api) | ITU / NANPA assignments, libphonenumber (Apache 2.0) |

## Quickstart

```bash
# Which bank issued this card prefix?
curl https://cardbincheck.com/api/bin/440066

# Where is this IP, and is it a VPN or datacenter?
curl https://ipsnapshot.com/api/ip/8.8.8.8

# When is the next bank holiday in the UK?
curl https://bankholidaycheck.com/api/holidays/next/GB

# Who manufactured this network device?
curl https://macvendorcheck.com/api/mac/00:1A:2B:3C:4D:5E

# Which city is ZIP 90210?
curl https://postalcodecheck.com/api/postal/US/90210

# Which region is this phone number from?
curl https://areacodecheck.com/api/phone/+14155552671
```

Each directory in this repository contains the full endpoint reference, an OpenAPI 3.1 specification, and copy-paste examples for curl, JavaScript and Python.

## Fair use

The same limits apply to every API:

- **500 requests per day** and **30 per minute**, per IP.
- Responses are cacheable (respect the `Cache-Control` headers and cache on your side where you can).
- Free for personal and commercial use. Attribution is appreciated: a link to the API's website.
- Need more volume? Write to **hello@ipnova.com**; we are happy to help genuine projects.

## Reliability

The APIs serve production traffic behind Cloudflare with local datasets refreshed on schedule (daily to monthly depending on the source). Breaking changes are avoided; if one is ever required it will be announced in this repository's releases first.

## Who is behind this

These APIs are built and operated by [IPNOVA SYSTEMS LTD](https://ipnova.com), a software company in Larnaca, Cyprus (Reg. HE 479674). We build free, data-driven utility platforms and keep them fast, honest and ad-free.

## License

The documentation and examples in this repository are [MIT licensed](./LICENSE). The underlying datasets keep their own licenses, credited per API above and in each response's `attribution` field.
