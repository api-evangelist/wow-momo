---
name: Embed a WOW! Momo page and read its SEO graph
description: >-
  Get a conformant oEmbed 1.0 representation of any www.wowmomo.com URL, and read the page's schema.org
  JSON-LD graph as parsed JSON instead of scraping it out of the HTML head.
api: openapi/_ae-authored/wow-momo-content-api-openapi.yml
operations:
  - getOembed
  - getSeoHead
  - listPages
---

# Embed a WOW! Momo page and read its SEO graph

Base URL: `https://www.wowmomo.com/wp-json`

## 1. Find the permalink

`listPages` — `GET /wp/v2/pages?_fields=id,slug,link` returns the six permalinks on the site.

## 2. Embed it

`getOembed` — `GET /oembed/1.0/embed?url=<permalink>`. Returns a conformant oEmbed 1.0 object naming
`"provider_name":"WOW! Momo"` and `"provider_url":"https://www.wowmomo.com"`.

```
GET https://www.wowmomo.com/wp-json/oembed/1.0/embed?url=https%3A%2F%2Fwww.wowmomo.com%2F
```

URL-encode the `url` parameter. A URL that is not on www.wowmomo.com returns HTTP 404 with
`{"code":"oembed_invalid_url","message":"Not Found"}` — note the message is the bare string "Not Found",
so branch on `code`.

## 3. Read the structured data

`getSeoHead` — `GET /yoast/v1/get_head?url=<permalink>`. Returns the rendered `<head>` as `html` **and**
as a parsed `json` object, including the schema.org `@graph`: `Organization`, `WebSite`, `WebPage`,
`BreadcrumbList`, `ImageObject` and a `SearchAction`. Use the `json` branch; do not parse the `html`.

Omitting `url` returns HTTP 400 `rest_missing_callback_param` with `data.params: ["url"]`.

This is the only anonymously readable route in the `yoast/v1` namespace — every other route in it is
administrative and returns 401.

## Notes

- No authentication, no key, no registration for any of the above.
- The `Organization` node is the best machine-readable identity record WOW! Momo publishes anywhere.
- Send a full browser User-Agent; mod_security answers a truncated one with HTTP 406.
