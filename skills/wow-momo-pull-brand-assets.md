---
name: Pull WOW! Momo brand and campaign imagery
description: >-
  Enumerate the WOW! Momo media library — brand logos and campaign artwork for WOW! Momo, WOW! China and
  WOW! Chicken — with real source URLs and dimensions, from the site's live WordPress REST API.
api: openapi/_ae-authored/wow-momo-content-api-openapi.yml
operations:
  - listMediaItems
  - getMediaItem
---

# Pull WOW! Momo brand and campaign imagery

Base URL: `https://www.wowmomo.com/wp-json`

98 media items were published at capture (2026-09-04), read from the `X-WP-Total` header. This is the
only WOW! Momo dataset that is both public and machine-readable.

## 1. Enumerate

`listMediaItems` — `GET /wp/v2/media`. Ask for a narrow projection; the full object is large:

```
GET https://www.wowmomo.com/wp-json/wp/v2/media?per_page=100&_fields=id,slug,source_url,mime_type,alt_text,date
```

`per_page` is capped at 100 — over it you get HTTP 400 `rest_invalid_param` with the bound stated in
`data.details.per_page.message`. With 98 items the whole library fits in one page.

## 2. Filter

Declared parameters that actually help here: `search`, `after` / `before` and `modified_after` /
`modified_before` (ISO 8601), `media_type`, `mime_type`, `order` and `orderby`. Slugs are descriptive —
`Wow-Momo-Logo`, `wow-china`, `wow-chicken`, `Wow-Kulfi-mango-_1920X799` — so a `search` on the brand name
is usually enough.

## 3. Get the renditions

`getMediaItem` — `GET /wp/v2/media/{id}`. `media_details.sizes` carries every generated rendition with its
own `source_url`, `width` and `height`. Pick the size you need rather than downloading the original and
resizing.

## What is NOT here

There is no menu, no outlet list, no nutrition data and no pricing on this API. The store locator is a
vendor-operated microsite at `restaurants.wowmomo.com`, and the ordering backend at `api.wowmomo.com`
answers every anonymous request with `{"message":"NO_AUTH"}` — including paths that do not exist, so a
200 there means nothing.

## Licensing

These are WOW! Momo's trademarks and campaign artwork. The API being open is not a licence to use them;
the site's Terms & Conditions (https://www.wowmomo.com/terms-conditions/) are the only governing
document, and they are not an API licence.
