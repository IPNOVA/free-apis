# IPNOVA Free APIs for Postman

A Postman collection covering all twelve IPNOVA free APIs: 25 requests, one folder per API, each with a working example value.

## Import

Run in Postman: [PLACEHOLDER: paste the Run in Postman button here after the collection is published]

Or import the files by hand:

1. In Postman choose Import and select `IPNOVA-Free-APIs.postman_collection.json`.
2. Import `IPNOVA.postman_environment.json` the same way and select the IPNOVA environment in the top right.
3. Send any request. No key is needed to try it.

## API key

Keys are optional. Without one, each API allows 100 requests a day and 30 a minute per IP. A free account at https://api.ipnova.com gives you one key for all twelve APIs with 5,000 lookups a month and 60 a minute, no card. Paste it into the `IPNOVA_API_KEY` variable (stored as a secret) and the collection sends it as the `X-API-Key` header on every request. Leave it empty to stay keyless. Never commit a filled-in environment file.

Batch requests cost one credit per ten items, rounded up. The collection Tests tab checks for status 200 and a JSON body.

## Folders

| Folder | Requests | What it does |
|---|---|---|
| BIN / IIN Lookup | 2 | Card scheme, funding and issuer from a BIN, single and batch |
| IP Intelligence | 1 | Geolocation, ASN and VPN or datacenter flags for an IP |
| SWIFT / BIC Lookup | 2 | Bank and branch behind a SWIFT code, single and batch |
| IBAN Validation | 2 | IBAN validation across 78 national formats, single and batch |
| Bank Holidays | 4 | Holidays by year, next holiday, date check, business days |
| MAC Vendor Lookup | 1 | Manufacturer from a MAC address |
| Postal Code Lookup | 2 | Place and coordinates for a postal code, plus country list |
| Phone / Area Codes | 1 | Region and line type of a phone number |
| Airport Codes and Routes | 5 | Airport details, routes, connections, search, countries |
| HS / Tariff Codes | 3 | HS code lookup, search and batch |
| Disposable Email Domains | 1 | Whether an email domain is disposable |
| VIN Decoder | 1 | Vehicle details, recalls and fuel economy from a VIN |

Full reference and OpenAPI specs: https://github.com/IPNOVA/free-apis
