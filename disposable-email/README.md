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

| Status | Meaning |
|---|---|
| `400` | Missing domain in the request |
| `422` | Domain is not a syntactically valid hostname |
| `429` | Rate limit hit: 30/minute or 500/day per IP |

## Notes

- `disposable` is one of three values: `confirmed` (on the manually vetted list or 2 or more sources agree), `reported` (a single list only, soft-verify but do not hard-block on this alone) or `no` (not found on any tracked list). A `no` answer means "not on the lists we track", not a guarantee that the domain is legitimate. Treat every verdict as strong guidance for your own decision, never as a final judgement.
- The check is list-based against four community blocklists: [disposable-email-domains](https://github.com/disposable-email-domains/disposable-email-domains) (CC0, manually vetted, every entry requires proof of a working throwaway inbox), [disposable/disposable](https://github.com/disposable/disposable) (MIT, an aggregate regenerated every 24 hours), [FGRibreau/mailchecker](https://github.com/FGRibreau/mailchecker) (MIT, the largest single list) and [groundcat/disposable-email-domain-list](https://github.com/groundcat/disposable-email-domain-list) (MIT, MX-validated entries). `free_provider` comes from [Kikobeats/free-email-domains](https://github.com/Kikobeats/free-email-domains) (MIT).
- No SMTP probing or mailbox verification is ever performed, on this endpoint or anywhere on the site. Only domain-level list membership and DNS are checked.
- `matched_domain` walks up to the parent when you query a subdomain of a listed service (`x.mailinator.com` matches `mailinator.com`); it is `null` when nothing matched.
- `mx_active` reflects our own rolling DNS sampling (MX, with an A/AAAA implicit-MX fallback), not any upstream list; `null` means the domain has not been sampled yet.
- Responses are sent with `Cache-Control: no-store`. Nothing about the input is logged or cached server-side.
- For checking many addresses at once, use the browser-side [bulk checker](https://emaildomaincheck.com/bulk-email-checker) or download the [full list](https://emaildomaincheck.com/disposable-email-domains) (rebuilt daily, free for commercial use) instead of looping this endpoint.
