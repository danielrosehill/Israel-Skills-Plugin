---
name: search-zap
description: Search zap.co.il — the canonical Israeli price-comparison aggregator. Zap indexes prices across all tier-1 and tier-2 IL retailers plus many niche specialists, so one Zap search often beats querying each retailer individually. Use as the default first-pass for any cross-retailer price comparison in Israel. Fall back to per-retailer skills only for SKUs Zap doesn't index.
---

# Search Zap

Zap is the canonical Israeli price-comparison aggregator. One Zap search returns a ranked table of all vendors carrying the product, with prices, so it's materially more efficient than hitting each retailer's search endpoint independently.

**Make this the default first-pass** for any cross-retailer price comparison. Only fall through to `/tech-product-search` or `/general-search` when Zap misses (new SKU, niche specialist product, or Zap search returns nothing).

> See `docs/search-strategies.md` § Zap for full URL patterns and workflow, § Hebrew-term resolution for the preflight, and § store-metadata merge convention for the merged-list rule used when flagging vendors.

## Backend

Playwright MCP. Zap has bot detection; Tavily's indexed snapshots are often stale and miss the price table.

## Preflight — Hebrew term resolution

Zap search is dramatically better with Hebrew queries. Build: `<hebrew term> <brand> <model>` — brand + model stay Latin.

Example: `עמדת טעינה Anker Prime A2343`, not `"Anker Prime A2343 desktop charging station"`.

Full resolution procedure (including reverse-lookup from listings when the term is unknown) is in `docs/search-strategies.md` § Hebrew-term resolution.

## URL patterns

### Keyword search (start here)

```
https://www.zap.co.il/search.aspx?keyword=<encoded query>
```

Returns a list of matching products and/or model pages.

### Model page (comparison table)

Once you've identified the model, Zap has a dedicated page with the full vendor price table:

```
https://www.zap.co.il/model.aspx?modelid=<id>
```

The `modelid` is visible in the search results URL. This is the page you actually want — it has all vendors side-by-side.

### Category browse (fallback)

If keyword search is unhelpful, browse by category:

```
https://www.zap.co.il/models.aspx?sog=<category>
```

Less precise but useful when you don't know the exact product name.

## Workflow

1. **Resolve Hebrew term** (preflight above).
2. **Navigate to Zap keyword search** with the Hebrew query.
3. **Find the right model.** Zap groups by SKU — pick the result that matches the specific model the user asked about. If unsure, list the top 3 model pages and confirm with user.
4. **Open the model page** (`model.aspx?modelid=...`). This has the price-comparison table.
5. **Extract the vendor table.** Each row has:
   - Vendor name
   - Price (ILS, usually VAT-inclusive — verify)
   - Delivery info
   - Link through to the vendor listing
6. **Check for Eilat / ex-VAT pricing.** Apply rules from `data/shopping-rules.md` — if a vendor shows an unusually low price, check for an ex-VAT or Eilat badge and use the higher regular price.
7. **Rank low-to-high** by VAT-inclusive price.
8. **Note vendors not in `data/israeli-stores.json`** — Zap often surfaces small specialists worth adding.

## Output

```
## {product} — Zap price comparison
Hebrew query: {hebrew}
Zap model page: {url}
Vendors found: {count}

| # | Price (₪, inc. VAT) | Vendor | Delivery | Link |
|---|---------------------|--------|----------|------|
| 1 | ...                 | ...    | ...      | ...  |

### Notes
- Any VAT adjustments applied
- Vendors not in our curated store list: ...
- Delivery cost differences worth flagging
```

## When Zap misses

If the keyword search returns no model page, or the vendor table is empty/very short, fall through in this order:

1. **`/general-search`** — Google IL discovery + category-dispatched search catches specialists Zap doesn't index.
2. **`/tech-product-search`** — direct queries to KSP/Ivory/Bug/TMS.

Tell the user why Zap didn't work (new product, niche, SKU variant not indexed) so they know whether the fallback results are representative.

## Rules

- Never quote Zap prices without visiting the vendor page — Zap's cached price can lag reality by 24+ hours.
- Always verify VAT treatment on the vendor page — Zap sometimes lists ex-VAT prices without a clear badge.
- If Zap has its own "Zap price" (recommendation), show it alongside the cheapest vendor — it reflects Zap's own deal arrangements.
- Cap output at 10 vendors unless the user explicitly asks for more.
