---
name: fetch-listing-api
description: Use when the user wants AliExpress product data fetched via the **official AliExpress Affiliate API** (HMAC-SHA256 signed requests against `api-sg.aliexpress.com/sync`) rather than the no-auth Puppeteer scraper. More reliable long-term, higher rate limits, structured response — but requires an approved AliExpress Affiliate app (App Key + App Secret + Access Token). Same output shape as `fetch-listing` so they are swappable. Trigger phrases — "use the aliexpress api", "fetch via affiliate api", "scrape is broken use the api".
---

# Fetch AliExpress Listing (Official API)

Authenticated counterpart to `fetch-listing`. Uses the AliExpress Affiliate Open Platform API instead of HTML scraping.

## Status: stub (awaiting credential validation)

This skill's auth scaffolding is wired but the request signing has not been validated end-to-end against a live, approved app. The first successful round-trip will be against Daniel's `DSR Holdings Purchasing` app.

## When to use

- The Puppeteer-based `fetch-listing` is failing (DOM rotation, anti-bot challenges, or rate-limited)
- You need higher throughput or more reliable structured data
- You have an approved AliExpress Affiliate app and credentials configured

For the no-auth path (default, no setup), use `fetch-listing` instead.

## API endpoints

Two relevant calls (Affiliate API, not Dropshipping):

| Endpoint                                        | Purpose                                         |
|-------------------------------------------------|-------------------------------------------------|
| `aliexpress.affiliate.productdetail.get`        | Fetch full detail for one or more product IDs   |
| `aliexpress.affiliate.product.query`            | Search the product catalogue (paginated)        |

Base URL: `https://api-sg.aliexpress.com/sync`

## Credential storage

Per the canonical plugin-data-storage convention, credentials live at:

```
${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/israel-shopping/config.json
```

with `0600` perms. Schema:

```json
{
  "aliexpress_affiliate": {
    "app_key": "...",
    "app_secret": "...",
    "access_token": "...",
    "refresh_token": "...",
    "expires_at": "2026-..."
  }
}
```

The plugin repo never contains credentials. The skill resolves the path at runtime and refuses to operate if the file is missing.

## Setup

1. Apply for an AliExpress Affiliate app at https://openservice.aliexpress.com/ — pick **Affiliates (individual)** as the collaborator type.
2. After approval, create an app under **App Console → Create App → Affiliates API**.
3. Complete the OAuth flow to obtain an access token (the callback URL set during app creation receives the `code` param).
4. Write credentials to the config path above.

## Request signing (HMAC-SHA256)

AliExpress signs requests by:

1. Sorting all parameters (system + business) alphabetically by key
2. Concatenating `key1value1key2value2...` into a single string
3. Computing `HMAC-SHA256(string, app_secret).hexdigest().upper()`
4. Appending as the `sign` parameter

Required system params per request:

- `app_key`
- `timestamp` — Unix milliseconds
- `sign_method` — `hmac-sha256`
- `format` — `json`
- `v` — `2.0`
- `method` — e.g. `aliexpress.affiliate.productdetail.get`

Reference implementation: see [FindMyDeal index.js](https://github.com/AdirCohen333/FindMyDeal/blob/main/index.js) — uses the same signing pattern against the DS endpoints (sub `aliexpress.affiliate.*` for `aliexpress.ds.*`).

## Output format

Same as `fetch-listing` — title, price, ship-to-IL fee, lead time, ratings, store, landed cost (subtotal + optional 18% VAT band). The skill normalises the API response into this shape so downstream skills don't care which path was used.

## Out of scope

- **Search** — eventually `search-aliexpress-api` (sibling to the current Playwright-based `search-aliexpress`).
- **Order placement** — DS API only (we registered as Affiliate, not DS).

## Validation checklist (when wired)

1. `config.json` resolved from `CLAUDE_USER_DATA` / XDG path.
2. `app_key`, `app_secret`, `access_token` all non-empty.
3. Signature computed matches AliExpress's expected format (verify with their sandbox).
4. Response status `0` or `00` indicates success — log `code` + `msg` on any other.
5. Output structure matches `fetch-listing` so consumers are interchangeable.
