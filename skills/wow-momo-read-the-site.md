---
name: Read the WOW! Momo site through its API
description: >-
  Find and read anything published on www.wowmomo.com without scraping HTML, using the site's live
  WordPress REST API. Anonymous, no key, no registration.
api: openapi/_ae-authored/wow-momo-content-api-openapi.yml
operations:
  - listSearchResults
  - listPages
  - getPage
  - getApiIndex
---

# Read the WOW! Momo site through its API

Base URL: `https://www.wowmomo.com/wp-json`

This surface is tiny and completely open. Six pages, no posts. Do not scrape the HTML — every page is
Elementor-built and the markup is noise.

## 1. Start with search, not with a collection

`listSearchResults` — `GET /wp/v2/search?search=<term>` — is the only operation that crosses post-type
boundaries. It returns `id`, `title`, `url`, `type` and `subtype` for every match. Seven objects are
indexed in total, so an unfiltered call returns the whole site.

```
GET https://www.wowmomo.com/wp-json/wp/v2/search?per_page=20
```

Use `subtype` to decide where to go next: `page` means the pages collection; `elementor_library` is a
template, not content, and should be ignored.

## 2. Read the page

`getPage` — `GET /wp/v2/pages/{id}`. Ask for only what you need with the `_fields` global parameter,
which this install supports (confirmed live) even though it is not declared in the route document:

```
GET https://www.wowmomo.com/wp-json/wp/v2/pages/7456?_fields=id,slug,link,title,content
```

Add `_embed=1` to inline the author record under `_embedded`.

## 3. Or list them all

`listPages` — `GET /wp/v2/pages`. Six published pages: the home page, `franchise-form`,
`events-and-bulk-orders`, `privacy`, `terms-conditions`, and a legacy `blog` page that renders empty.
Pagination is `page` / `per_page`, capped at 100; read `X-WP-Total` and `X-WP-TotalPages` from the
response headers, and `Link: rel="next"` for the next page.

## 4. When you need the route table

`getApiIndex` — `GET /` (that is `https://www.wowmomo.com/wp-json/`). Returns the site name, the 36
declared namespaces, all 396 routes, and the authentication block. Everything in this contract was read
from it.

## Rules that will save you a retry

- **Errors are not RFC 9457.** The body is `{"code":"...","message":"...","data":{"status":404}}`. Branch
  on `code`, not on the message string. See `errors/wow-momo-problem-types.yml`.
- **Validation errors are worth reading.** A bad `per_page` or `orderby` returns HTTP 400
  `rest_invalid_param` with `data.details.<param>.message` stating the accepted range or enum verbatim.
- **Send a real User-Agent.** The host runs mod_security and answered a request sent with the
  User-Agent `Mozilla/5.0` with HTTP 406.
- **This is read-only for you.** Writes exist on the same routes but need a WordPress application
  password issued from inside wp-admin. There is no idempotency mechanism and no dry-run mode.
- **No rate limit is published or signalled.** Be conservative anyway; nothing tells you when you are
  close.
