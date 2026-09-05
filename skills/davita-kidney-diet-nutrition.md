---
name: davita-kidney-diet-nutrition
description: Look up kidney-relevant nutrients for a food and find kidney-friendly recipes using DaVita's public, unauthenticated REST API.
api: DaVita Web REST API
base_url: https://www.davita.com/wp-json
operations:
  - get_davita_v1_usda_search
  - get_davita_v1_usda_food_id
  - get_wp_v2_dv_recipe
  - get_wp_v2_dv_recipe_id
  - get_wp_v2_dv_recipe_diet_tag
  - get_wp_v2_dv_recipe_category
generated: '2026-09-05'
method: generated
source: openapi/davita-wp-rest.yml (derived from https://www.davita.com/wp-json/), verified live 2026-09-05
---

# Kidney diet: nutrients and recipes

Two public, keyless surfaces: a nutrient lookup that DaVita reshapes from USDA FoodData Central for
the kidney diet, and a 1,258-record kidney-friendly recipe library.

## 1. Search a food — `get_davita_v1_usda_search`

```
GET /davita/v1/usda/search?food=apple&page=1
```

Parameters: `food` (**required**), `page`.

```json
{"success":true,"data":{"totalHits":25661,"currentPage":1,"totalPages":2567,
 "foods":[{"fdcId":174988,"fullDescription":"Croissants, apple",
 "nutrients":{"calories":254,"protein":"7.4 g","sodium":"274 mg","potassium":"90 mg","phosphorus":"58 mg"}}]}}
```

The nutrient panel is the point: **sodium, potassium and phosphorus** are the three DaVita surfaces
for kidney patients, alongside calories and protein. Note that nutrient values are **strings with
units** while `calories` is a number — parse accordingly.

Paginate with `page` against `data.totalPages`, not with `per_page` (this route has no page-size
parameter).

## 2. Get one food — `get_davita_v1_usda_food_id`

```
GET /davita/v1/usda/food/174988
```

`id` must be an `fdcId` you got from the search above. An unknown id returns HTTP 404 with a
namespace-specific code, not the usual WordPress one:

```json
{"code":"dv_usda_api_fail_error","message":"Something went wrong while getting data from the USDA API.","data":{"status":404}}
```

That code also covers an upstream USDA failure, so a 404 here does not always mean "no such food".
Retry once before concluding the food is missing.

## 3. Find kidney-friendly recipes — `get_wp_v2_dv_recipe`

```
GET /wp/v2/dv-recipe?search=chicken&per_page=20&_fields=id,title,link,dv-recipe-diet-tag
```

Parameters: `search`, `page`, `per_page` (1-100), `offset`, `order`, `orderby`, `_fields`, `_embed`.

This is a `wp/v2` collection, so the shape is **different from `davita/v1`**: a bare JSON array, with
paging in the headers — `X-WP-Total` (1258 observed 2026-09-05), `X-WP-TotalPages`, and a `Link`
header carrying `rel="next"`.

Always send `_fields` to trim the payload; a full recipe record includes rendered HTML content.

Filter by taxonomy id, which you resolve first:

- `get_wp_v2_dv_recipe_diet_tag` — 9 diet tags
- `get_wp_v2_dv_recipe_category` — 14 categories
- plus cuisine tag (19), dish type (19), holiday tag (10), method tag (10)

Then pass the term ids on the recipe query, e.g. `?dv-recipe-diet-tag=<id>`.

## 4. Get one recipe — `get_wp_v2_dv_recipe_id`

```
GET /wp/v2/dv-recipe/55203?_embed
```

`_embed` inlines the featured image and terms through the `_links` graph so you avoid a second
round trip.

## Errors

| code | status | what to do |
|---|---|---|
| `rest_missing_callback_param` | 400 | Missing `food`. `data.params` lists it. |
| `rest_invalid_param` | 400 | Usually `per_page` outside 1-100, or an `orderby` not in the enum. `data.details` names the constraint. |
| `rest_post_invalid_id` | 404 | Recipe id does not exist. List the collection first. |
| `dv_usda_api_fail_error` | 404 | Unknown `fdcId` **or** an upstream USDA failure. Retry once. |

## Do not

- Do not treat this as clinical guidance. Nutrient targets are individual; the API returns data, not
  a diet plan. Point people at https://davita.com/tools/food-analyzer/ and their care team.
- Do not credit DaVita with the nutrient values — the source is USDA FoodData Central.
- Do not write. Every write route requires WordPress credentials that are not offered to the public
  and returns `rest_cannot_create` (401) without them.
