---
name: davita-find-dialysis-center
description: Find DaVita dialysis centers and Kidney Smart education classes near a location using DaVita's public, unauthenticated REST API.
api: DaVita Web REST API
base_url: https://www.davita.com/wp-json
operations:
  - get_davita_v1_find_a_center
  - get_davita_v1_find_a_class
generated: '2026-09-05'
method: generated
source: openapi/davita-wp-rest.yml (derived from https://www.davita.com/wp-json/), verified live 2026-09-05
---

# Find a DaVita dialysis center or Kidney Smart class

DaVita's website locator is backed by two public endpoints that need no key and no account.

## Before you start

- Base URL: `https://www.davita.com/wp-json`
- No authentication. Send no `Authorization` header.
- This is an **undocumented** API. DaVita publishes no compatibility promise, so check the shape of
  every response rather than assuming it.
- These endpoints return **center and class listings only**. No patient data is on this surface.

## 1. Find centers near a location — `get_davita_v1_find_a_center`

```
GET /davita/v1/find-a-center?location=Denver%2C%20CO
```

Parameters: `location` (**required**), `lat`, `lng`, `modalities`.

`location` is required even when you have coordinates. Calling with only `lat` and `lng` returns:

```json
{"code":"rest_missing_callback_param","message":"Missing parameter(s): location","data":{"status":400,"params":["location"]}}
```

Pass `lat` and `lng` **alongside** `location` to disambiguate, not instead of it. Use `modalities`
to narrow to a treatment type (in-center hemodialysis, home hemodialysis, peritoneal dialysis).

The response uses the `davita/v1` envelope, which is different from the `wp/v2` collections:

```json
{"success": true, "data": {"locations": [...]}}
```

Always branch on `success` first, then read `data`. `data.locations` can be `null` for a location
with no match — that is an empty result, not an error.

## 2. Find Kidney Smart education classes — `get_davita_v1_find_a_class`

```
GET /davita/v1/find-a-class?location=80202
```

Parameters: `location`, `language`, `sort`, `classes`, `days`, `time`.

The response carries a `data.filters[]` block that enumerates every valid filter value **with live
counts**, e.g. language (English, Spanish), class location (in person, online), sort (date,
distance). Read `filters[]` to discover the legal values before you filter — do not hardcode them.

## Errors

| code | status | what to do |
|---|---|---|
| `rest_missing_callback_param` | 400 | `data.params` is the array of missing names. Supply them. |
| `rest_invalid_param` | 400 | `data.details.<param>.message` names the exact constraint. |
| `rest_no_route` | 404 | Re-read the route index at `https://www.davita.com/wp-json/`. |

Errors are the WordPress envelope (`code` / `message` / `data.status`), **not** RFC 9457. Key on
`code`. See `errors/davita-problem-types.yml`.

## Rate limits and retries

None are published and no rate-limit header is returned — you get no budget signal. Responses carry
`cache-control: max-age=60`. Be conservative: serialize requests, cache for at least 60 seconds, and
back off on any 5xx. See `rate-limits/davita-rate-limits.yml`.

## Do not

- Do not send credentials. There are none to send.
- Do not use `/wp/v2/dv-dialysis-center` — it is a registered but **empty** collection (X-WP-Total 0).
  `/davita/v1/find-a-center` is the real locator.
- Do not present results as clinical advice. Point people at
  https://davita.com/tools/find-a-dialysis-center/ to confirm.
