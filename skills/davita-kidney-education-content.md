---
name: davita-kidney-education-content
description: Retrieve DaVita's kidney education articles, vocabulary terms, videos and site-wide search results through its public, unauthenticated REST API.
api: DaVita Web REST API
base_url: https://www.davita.com/wp-json
operations:
  - get_wp_v2_search
  - get_wp_v2_dv_article
  - get_wp_v2_dv_article_id
  - get_wp_v2_dv_glossary_term
  - get_wp_v2_dv_topic
  - get_wp_v2_dv_video
  - get_wp_v2_dv_cookbook
generated: '2026-09-05'
method: generated
source: openapi/davita-wp-rest.yml (derived from https://www.davita.com/wp-json/), verified live 2026-09-05
---

# DaVita kidney education content

DaVita's education library is readable anonymously as structured JSON. Counts below were observed
live on 2026-09-05 from the `X-WP-Total` header.

## Start with site-wide search — `get_wp_v2_search`

```
GET /wp/v2/search?search=hospital%20partners&per_page=10
```

Returns `{id, title, url, type, subtype, _links}` per hit across every content type. This is the
fastest way to resolve a topic to a canonical `davita.com` URL, and it is how you should find a page
rather than guessing a path — many plausible paths on davita.com return a themed 404.

## The collections

| Operation | Collection | Records |
|---|---|---|
| `get_wp_v2_dv_article` | Kidney education articles | 570 |
| `get_wp_v2_dv_video` | Videos | 126 |
| `get_wp_v2_dv_topic` | Education topic taxonomy | 52 |
| `get_wp_v2_dv_cookbook` | Cookbooks | 51 |
| `get_wp_v2_dv_glossary_term` | Kidney vocabulary terms | 33 |

All are standard `wp/v2` collections:

```
GET /wp/v2/dv-glossary-term?search=glomeruli&_fields=id,title,content,link
GET /wp/v2/dv-article?per_page=50&orderby=modified&order=desc&_fields=id,title,link,modified
```

## Paging and field selection

- `page`, `per_page` (1-100), `offset`.
- Totals arrive in headers: `X-WP-Total`, `X-WP-TotalPages`. A `Link` header carries `rel="next"`.
- `_fields` trims the payload — always use it. Full records embed rendered HTML.
- `_embed` inlines the featured image and taxonomy terms via `_links`.
- `orderby` is enum-validated; an invalid value returns `rest_invalid_param` whose
  `data.details.orderby.message` lists every legal value. Read it rather than guessing.

## Detecting change

There is no changelog and no webhook. The only change signal is per record:

```
GET /wp/v2/dv-article?modified_after=2026-08-01T00:00:00&orderby=modified&order=desc
```

Poll with `modified_after` and store the high-water `modified_gmt`. Respect
`cache-control: max-age=60` — do not poll faster than once a minute.

## Errors

WordPress envelope, `{code, message, data.status}`, not RFC 9457. `rest_post_invalid_id` (404) for a
missing record, `rest_invalid_param` (400) for a bad parameter, `rest_no_route` (404) for a path that
is not registered. See `errors/davita-problem-types.yml`.

## Attribution and limits

- Content is DaVita's, under https://davita.com/terms-of-use/. Link back to the record's `link`
  field rather than republishing rendered HTML wholesale.
- This is patient **education** material. It is not individualized medical advice, and nothing on
  this API is clinical data.
- The API is undocumented and carries no compatibility promise. Handle schema drift.
