# Changelog

Notable changes to the twelve APIs. Data refreshes run on schedule and are not listed here.

## 2026-09-16

**One error envelope on every API, and optional API keys.**

- Every non-`200` response now has the same body on all twelve APIs: `success`, `error`, `reason`, `message`, `docs`, followed by any API-specific extra fields. The `reason` codes are listed in the root README under Errors and API keys.
- Every `200` response now starts with `"success": true`. APIs that already returned it are unchanged; the others gained the field as the first key and kept every existing field in place.
- Status codes were aligned: malformed input is `400` everywhere (the VIN decoder and disposable email APIs returned `422`), and a valid query with no record is `404` everywhere (the MAC vendor API returned `200` with `success: false` for unregistered prefixes).
- API keys from [api.ipnova.com](https://api.ipnova.com) are accepted on every API as the `X-Api-Key` header or the `?key=` query parameter. Keyed responses carry `X-Credits-Remaining`, `X-Credits-Used` and `X-Plan`. Keyless access keeps working within the per-IP limits.
- The `?key=` parameter is passed through on every `/api/...` route.

**What to check in existing integrations.** Success payloads are unchanged. If your code parses error bodies, note that `error` is now the boolean `true` and the text moved to `message`. If your code relied on `422` or on the MAC API's `200` for unknown prefixes, update the status checks.

## 2026-09-16 (earlier)

- Documentation for the six APIs launched since August added, twelve in total. Consistency pass across every folder: real limits, real status codes, real example responses.

## 2026-08-30

- Repository opened with six APIs: BIN lookup, IP intelligence, bank holidays, MAC vendor, postal codes, phone and area codes.
