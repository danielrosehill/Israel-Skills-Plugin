---
name: il-tech-search
description: Search the four mainstream Israeli tech retailers — Ivory, KSP, Bug, TMS — for a consumer-tech product (laptop, phone, peripheral, accessory, charger, cable, monitor, TV, headphones, etc.) and return prices ranked low-to-high. Use whenever the user asks to find, price, compare, or check availability of tech gear in Israel, or mentions any of those four retailers by name. Handles Hebrew-term resolution and falls back to Playwright when Tavily/WebFetch is blocked.
---

# IL Tech Search — Ivory / KSP / Bug / TMS

Finds a consumer-tech product across the four tier-1 Israeli tech chains and returns a ranked price table.

## Retailers (fixed set)

| Retailer | Search URL pattern | Notes |
|---|---|---|
| Ivory | `https://www.ivory.co.il/catalog.php?act=cat&q=<URL-encoded Hebrew>` | Usually scrapable via Tavily / WebFetch. |
| KSP | `https://ksp.co.il/web/cat/?search=<URL-encoded Hebrew>` | Bot-detected — Playwright required. Two prices (regular + Eilat); always quote the **regular** (higher) price unless the user specifies Eilat pickup. |
| Bug | `https://www.bug.co.il/` (homepage search) | Bot-detected — Playwright required. URL pattern not stable; use homepage search box. |
| TMS | `https://tms.co.il/index.php?route=product/search&search=<URL-encoded Hebrew>` | Scrapable via Tavily / WebFetch. |

**URL-encode the Hebrew query.** Example: `מחשב` → `%D7%9E%D7%97%D7%A9%D7%91`, so the Ivory URL for "computer" becomes `https://www.ivory.co.il/catalog.php?act=cat&q=%D7%9E%D7%97%D7%A9%D7%91`. Use any standard percent-encoder; don't paste raw Hebrew into the URL field.

If a documented pattern ever returns a homepage redirect or 404, fall back to Playwright homepage-search flow and flag it in the output notes.

Canonical store metadata lives in `~/repos/github/my-repos/Israel-Online-Stores/stores.json` (live — read directly when you need contact, delivery, or Eilat-door info).

## Preflight — Hebrew term resolution (mandatory)

Israeli retailer search **returns nothing useful with English category nouns**. Always build the query as:

```
<Hebrew category noun>  <brand in Latin>  <model in Latin>
```

Examples:
- Anker Prime A2343 charging tower → `עמדת טעינה Anker Prime A2343`
- Logitech MX Master 3S mouse → `עכבר Logitech MX Master 3S`
- LG C4 55" OLED TV → `טלוויזיה LG C4 55`

**Brand and model stay Latin** — retailers index SKUs in Latin. Only the category noun is translated.

If you don't know the Hebrew noun, derive it before searching:
1. Check the user's vocabulary — if they used a Hebrew word, use it.
2. Otherwise translate the category noun (e.g. "mouse" → `עכבר`, "charger" → `מטען`, "cable" → `כבל`, "monitor" → `מסך`, "headphones" → `אוזניות`, "laptop" → `מחשב נייד`, "keyboard" → `מקלדת`).
3. Have 1–2 Hebrew synonyms ready as fallback (e.g. `עמדת טעינה` / `תחנת טעינה` for a charging dock).

## Backend strategy

Try in this order per retailer. Don't ask the user — pick and proceed; note the backend used in the output.

1. **Tavily search** (`mcp__jungle-shared__tavily__tavily_search`) with `query: "<hebrew> <brand> <model> site:<domain>"`, `search_depth: "basic"`, `max_results: 5`. Fast. Works for Ivory and TMS.
2. **WebFetch** the retailer's search URL directly if you know the pattern (verify it returns real results, not a homepage redirect).
3. **Playwright fallback** (`mcp__plugin_playwright_playwright__browser_*`) when steps 1–2 return no results or the retailer is KSP/Bug:
   - `browser_navigate` to the retailer homepage.
   - `browser_snapshot` to locate the search input.
   - `browser_type` the Hebrew query into the input with `submit: true`.
   - `browser_snapshot` again, extract the top 5 product cards (title, price ILS, link, in-stock).
   - `browser_close` when done with that retailer.

If all three fail for a retailer, record "no results / blocked" in the output and continue — don't fail the whole run.

## Workflow

1. Resolve Hebrew category noun (above). State it in the first user-facing line.
2. For **each** of the four retailers, run the backend chain in parallel where possible. Cap at 5 results per retailer.
3. If a retailer returns zero hits, retry once with:
   - A Hebrew synonym, OR
   - Brand + model only (drop the Hebrew noun) — sometimes retailer search rejects the noun but accepts the SKU.
4. **KSP price rule:** quote the regular (non-Eilat) price. If both are shown, ignore the cheaper Eilat one unless the user has explicitly asked about Eilat pickup.
5. Rank all results low-to-high by VAT-inclusive ILS price.

## Output format

```
## <product> — Israeli tech stores
Hebrew query: <hebrew noun> <brand> <model>
Backends used: Ivory=<tavily|playwright|…>, KSP=<…>, Bug=<…>, TMS=<…>

| # | ₪ (inc. VAT) | Product | Retailer | Link |
|---|---|---|---|---|
| 1 | … | … | … | … |

### Notes
- Any retailer that returned nothing, and what fallbacks were tried.
- Any price outliers or stock warnings.
- KSP: regular price quoted (Eilat price, if seen: ₪X).
```

## International fallback — AliExpress & Amazon

If the local four come back empty, or the cheapest IL price looks inflated vs. international RRP, **mention AliExpress and Amazon (US/UK/DE)** as a cross-check:

- **AliExpress** — usually ships to Israel; factor in 3–6 week delivery and that purchases ≥ $75 attract Israeli VAT + customs.
- **Amazon US** — check the product page for "Ships to Israel" (many items don't). Amazon's own global shipping sometimes pre-bundles IL VAT/customs into checkout; third-party sellers usually don't ship IL.
- **Amazon DE / UK** — often better for EU-sourced gear; shipping to IL is inconsistent per seller.

Don't price these automatically in the main table — call them out in **Notes** as "worth checking abroad" with a rough international RRP if known. Dedicated international-price tools can be added later; for now this is a pointer, not a fetch.

## Rules

- Never invent a price. If you can't extract it, say so.
- Never quote USD on these retailers — they list in ILS inc. VAT.
- Never silently substitute a similar-but-different model — if the exact SKU isn't carried, name the closest match and flag it.
- If the user asks for retailers outside this set (Ace, Home Center, audio specialists, etc.), tell them this skill covers the tier-1 tech four only and suggest `/israel-search-major-retailers` or `/israel-search-zap`.
