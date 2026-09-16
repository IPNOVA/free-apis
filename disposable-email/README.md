# Disposable Email Domain API

Tell whether an email domain belongs to a disposable or throwaway mail service. The verdict is two-tier, drawn from four community blocklists, with a separate free-provider flag, so you can validate signups without ever collecting or storing an address. Domain-only by design: send the domain, not the address.

**Base URL** `https://emaildomaincheck.com` · **Live docs** [emaildomaincheck.com/email-api](https://emaildomaincheck.com/email-api) · [OpenAPI spec](./openapi.yaml)

## Endpoint

```
GET /api/email-domain/{domain}
```

| Parameter | Type | Description |
|---|---|---|
| `domain` | path, required | A domain name, not a full address. If a full address is sent anyway, only the part after `@` is used and nothing is stored |

## Example

```bash
curl https://emaildomaincheck.com/api/email-domain/mailinator.com
```

```json
{
  "success": true,
  "domain": "mailinator.com",
  "disposable": "confirmed",
  "matched_domain": "mailinator.com",
  "blocklists": ["disposable-email-domains blocklist", "disposable/disposable daily aggregate", "FGRibreau/mailchecker"],
  "blocklists_tracked": 4,
  "free_provider": false,
  "first_seen": null,
  "mx_active": null,
  "checked_at": "2026-09-16T05:58:20+00:00",
  "docs": "https://emaildomaincheck.com/email-api"
}
```

### JavaScript

```js
const res = await fetch('https://emaildomaincheck.com/api/email-domain/mailinator.com');
const j = await res.json();
if (j.disposable === 'confirmed') console.log('Block signup: confirmed disposable domain');
```

### Python

```python
import requests

j = requests.get("https://emaildomaincheck.com/api/email-domain/mailinator.com", timeout=10).json()
if j["disposable"] == "confirmed":
    print("Block signup: confirmed disposable domain")
```

## Errors

| Status | `reason` | Meaning |
|---|---|---|
| `400` | `invalid_input` | Missing domain in the request, or the domain is not a syntactically valid hostname. |
| `401` | `invalid_key` | The API key sent is malformed, unknown, revoked or expired. Keyless calls are never `401`. |
| `429` | `rate_limit` | Too many requests: 30/minute per IP. Back off for the seconds in `Retry-After`. |
| `429` | `daily_limit` | The keyless per-IP daily allowance (500/day) is used up. Resets at midnight UTC. |
| `429` | `credits_exhausted` | Keyed request whose monthly plan allowance is used up. |

Every error uses the shared envelope described in the [repository README](../README.md#errors-and-api-keys):

```bash
curl https://emaildomaincheck.com/api/email-domain/abc
```

```json
{
  "success": false,
  "error": true,
  "reason": "invalid_input",
  "message": "Not a valid domain.",
  "docs": "https://emaildomaincheck.com/email-api"
}
```

## Notes

- `disposable` is one of three values: `confirmed` (on the manually vetted list or 2 or more sources agree), `reported` (a single list only, soft-verify but do not hard-block on this alone) or `no` (not found on any tracked list). A `no` answer means "not on the lists we track", not a guarantee that the domain is legitimate. Treat every verdict as strong guidance for your own decision, never as a final judgement.
- The check is list-based against four community blocklists: [disposable-email-domains](https://github.com/disposable-email-domains/disposable-email-domains) (CC0, manually vetted, every entry requires proof of a working throwaway inbox), [disposable/disposable](https://github.com/disposable/disposable) (MIT, an aggregate regenerated every 24 hours), [FGRibreau/mailchecker](https://github.com/FGRibreau/mailchecker) (MIT, the largest single list) and [groundcat/disposable-email-domain-list](https://github.com/groundcat/disposable-email-domain-list) (MIT, MX-validated entries). `free_provider` comes from [Kikobeats/free-email-domains](https://github.com/Kikobeats/free-email-domains) (MIT).
- No SMTP probing or mailbox verification is ever performed, on this endpoint or anywhere on the site. Only domain-level list membership and DNS are checked.
- `matched_domain` walks up to the parent when you query a subdomain of a listed service (`x.mailinator.com` matches `mailinator.com`); it is `null` when nothing matched.
- `mx_active` reflects our own rolling DNS sampling (MX, with an A/AAAA implicit-MX fallback), not any upstream list; `null` means the domain has not been sampled yet.
- Responses are sent with `Cache-Control: no-store`. Nothing about the input is logged or cached server-side.
- For checking many addresses at once, use the browser-side [bulk checker](https://emaildomaincheck.com/bulk-email-checker) or download the [full list](https://emaildomaincheck.com/disposable-email-domains) (rebuilt daily, free for commercial use) instead of looping this endpoint.
- About 76,800 domains tracked across the four lists, rebuilt daily.
- Optional API key: send `X-Api-Key` or `?key=` from a free api.ipnova.com account and your plan allowance applies instead of the per-IP limits; keyed responses carry `X-Credits-Remaining` and `X-Credits-Used`.
