---
name: compare-to-local
description: Use when the user wants to check whether a product they've found on AliExpress (or are considering importing) is also sold by Israeli retailers — and at what price. Runs two separate web searches scoped to Israeli sources — one with `site:il` (broad — Ksp, Bug, Ivory, Zap-listed shops, manufacturer .co.il pages, etc.) and one with `site:zap.co.il` (Zap, Israel's main price-comparison aggregator) — then summarises the cheapest local options in ILS for side-by-side comparison with the AliExpress listing. Trigger phrases — "compare to local", "is this sold in israel", "check zap for X", "site:il price for X", "local price israel", "compare aliexpress to local".
---

# Compare to Local (Israel)

Given a product (name, model, or AliExpress listing), find out whether it's available locally in Israel and at what price, by running **two separate scoped web searches**:

1. `<query> site:il` — broad sweep across the `.il` TLD (catches `.co.il`, `.org.il`, `.net.il`, etc. — Ksp, Bug, Ivory, Lastprice, manufacturer pages, smaller shops, marketplaces)
2. `<query> site:zap.co.il` — Zap is Israel's dominant price-comparison aggregator; this surfaces aggregated listings with multi-vendor pricing

The two searches are run **independently** and reported separately. Do not collapse them — they answer different questions:
- `site:il` = "who sells this in Israel?"
- `site:zap.co.il` = "what's the comparison-shopping price?"

## Inputs

- `query` (required) — the product to look up. Prefer a precise identifier (brand + model number, e.g. `Anker 737 PowerCore`, `Logitech MX Master 3S`) over a generic description.
- `aliexpress_url` (optional) — the AliExpress listing the user is comparing against. If supplied, also extract the AliExpress title and ILS price for the final side-by-side block.
- `aliexpress_price_ils` (optional) — landed-cost figure if the user has already computed it; otherwise just the listed AE price.

## How to query

Use whichever web search tool is available (WebSearch, Tavily, etc.). Fire two distinct searches:

### Search 1 — broad `.il` TLD

```
<query> site:il
```

Notes:
- `site:il` matches **any** `.il` second-level (`co.il`, `org.il`, `net.il`, `ac.il`, `gov.il`, plain `.il`). Don't narrow to `site:co.il` — you'll miss legitimate retailers.
- If the query is an English brand/model, also try a second variant with the Hebrew transliteration if obvious (e.g. `Logitech` → `לוג'יטק`). Mark Hebrew variants as a separate sub-search in the report.

### Search 2 — Zap aggregator

```
<query> site:zap.co.il
```

Notes:
- Zap product pages live at `zap.co.il/model.aspx?modelid=...` (aggregated model page) and `zap.co.il/models.aspx?...` (search). The model page is the gold result — it shows a price range across vendors.
- If the top hit is a Zap **category** page rather than a model page, drill into the model page before reporting prices.

## What to extract from each result

For every promising hit (top ~5 per search):

- `vendor` — retailer name (e.g. Ksp, Bug, Ivory, Lastprice, Zap, manufacturer)
- `url` — direct product URL
- `title` — product title as shown on the page
- `price_ils` — listed price in ILS. If a range, capture both ends.
- `in_stock` — best-effort: in-stock / out-of-stock / unknown
- `notes` — any flags worth surfacing (refurbished, parallel import / יבוא מקביל, kit-only, used)

If the search snippet contains the price, you can use it directly. Otherwise fetch the page and parse — but only for the top 2–3 results to avoid wasted fetches.

## Israel-specific gotchas

- **Parallel import (יבוא מקביל)**: common on AliExpress-adjacent items. Often cheaper than the official importer (יבואן רשמי) but warranty is shorter and not honoured by the brand's IL service centre. Surface this distinction if visible.
- **VAT (מע"מ)**: prices on Israeli retail sites are normally VAT-inclusive (currently 18% as of 2026; verify if the comparison hinges on it). AliExpress prices to IL are pre-customs/VAT for orders ≥ $75 USD, so always compare *landed* cost, not sticker.
- **Zap "starting from" prices**: the headline price on a Zap model page is the cheapest single vendor; click through to see whether that vendor is reputable (Zap shows a star rating). Don't treat the headline price as the realistic floor without verifying the vendor.
- **Hebrew vs. English titles**: many Israeli retailers list the product in Hebrew but with the English model number embedded. Match on the model number, not the title text.

## Output format

Report the two searches separately, then a comparison block if `aliexpress_url` / `aliexpress_price_ils` were supplied.

```
Query: <query>

──────── Search 1: site:il ────────
Top results:
1. <vendor> — ₪<price>   [in stock | OOS | ?]
   <title>
   <url>
   notes: <parallel import / official / refurb / …>
2. …

──────── Search 2: site:zap.co.il ────────
Zap model page: <URL or "no model page found">
Price range on Zap: ₪<low> – ₪<high>   (across <N> vendors)
Cheapest vendor: <name> @ ₪<price>   <url>
Notable vendors: <list>

──────── Comparison (if AliExpress supplied) ────────
AliExpress: ₪<ae_price>   <ae_url>
Cheapest local (site:il): ₪<best_il>   <vendor>
Cheapest on Zap:          ₪<best_zap>  <vendor>
Delta vs AliExpress:      <±%> vs site:il, <±%> vs Zap

Caveats:
- AliExpress price is sticker — add VAT/customs above ₪… threshold for true landed cost.
- Local prices include 18% VAT.
- Warranty: AE = none / seller dispute only; local official importer = 1y+; parallel import = vendor-dependent.
```

If a search returns nothing useful (genuine zero results, or only irrelevant hits), say so explicitly. Do not invent vendors or prices.

## Failure modes / what to do

- **No results on either search**: report that the product appears not to be sold in Israel under that name; suggest the user try the Hebrew transliteration or the brand-only query.
- **Zap returns category page only**: report the category URL and note that no aggregated model page exists yet — common for very new or niche products.
- **Price visible only after login or in cart**: skip silently; Israeli retailers occasionally hide prices for B2B items. Note in the report.
- **Captcha / blocked search result**: don't loop. Report which source is blocked and continue with the other.

## Out of scope

- Does **not** place orders or check live cart prices.
- Does **not** evaluate the AliExpress side beyond the inputs given — pair this with `evaluate-listing` (downstream skill) if landed cost is needed.
- Does **not** rank vendors by trust/reputation beyond what's visible in the search snippet — that's a separate concern.

## Validation checklist (before declaring success)

1. Two separate searches were issued: one with `site:il`, one with `site:zap.co.il`.
2. Each search has its own results section in the output (not merged).
3. All prices are in ILS and labelled with `₪`.
4. If `aliexpress_url` was supplied, the comparison block is present and prices are aligned to the same currency.
5. Vendor names and URLs are real (no fabricated hits).
