---
name: rrp-check
description: Compare an Israeli price (ILS) against the international RRP/MSRP or a specific international retailer (Amazon, B&H, manufacturer). Reports markup % and a verdict (good deal / fair / above average / heavy markup) with import-cost realism.
---

# RRP Check

Given an Israeli price for a product, decide whether it's a good deal versus the international market. Lighter-weight than a full sourcing run — answers "is this a good price?".

> See `docs/search-strategies.md` § VAT and markup verdict scale for the thresholds and the 18% VAT rule.

## When to use

- User asks "is this a good price?" or "how does this compare internationally?"
- Cross-checking a shortlist against RRP before committing to buy.
- Input for `/freier-check`.

## Inputs

- Product name + model
- Israeli price (ILS, VAT-inclusive — confirm if ambiguous)
- Optional: comparison target (`vs Amazon US`, `vs B&H`, `vs RRP`). Default: manufacturer RRP.

## Backend choice

- **Tavily** — default for "what's the RRP" lookups (multi-source, fast).
- **Playwright** — when the user names a specific URL or retailer page.

## Workflow

1. **Get the international reference price.** Try in order:
   - Manufacturer site (most authoritative for RRP).
   - Amazon US / Amazon DE / Amazon UK (user preference or default US).
   - B&H Photo (camera/audio/computer gear).
   - Specialist retailer if the category demands.
2. **Convert to ILS** via `/convert-currency` or live rate. Always state the rate + date.
3. **Normalise VAT** — if the IL price is ex-VAT, add 18% first.
4. **Compute markup:** `(il_price - intl_price_ils) / intl_price_ils * 100`.
5. **Apply verdict scale** (from strategies doc):
   - ≤5% → GOOD DEAL
   - 5–15% → FAIR (typical IL import markup)
   - 15–30% → ABOVE AVERAGE
   - 30–40% → MILD MARKUP
   - \>40% → HEAVY MARKUP — consider importing
6. **Import realism.** Before recommending "import instead", factor landed cost:
   - Amazon: shipping ($15–40 small items) + 18% VAT + possible customs duty (imports >$75 USD de minimis).
   - AliExpress: shipping cheap/free, 2–6 week lead time, same VAT/customs rules.
   - A 30–40% IL markup is often still cheaper than importing once all-in costs are counted.

## Output

```
## {product} — Israel vs international

| Region | Price | Source |
|---|---|---|
| Israel | ₪{x} (inc. VAT) | {retailer} |
| {Region} | {ccy}{y} (≈ ₪{z}) | {source} |

**Markup:** {pct}%
**Verdict:** {GOOD DEAL | FAIR | ABOVE AVERAGE | MILD MARKUP | HEAVY MARKUP}
**Recommendation:** {buy locally | import via Amazon | AliExpress | wait}

### Notes
- Exchange rate: 1 {ccy} = {rate} ILS ({date})
- Whether shipping + import VAT/duty included on international side
- Warranty / local-support caveats
```

## Rules

- Always state the exchange rate used. Never silently apply an unstated rate.
- Be explicit about whether the international price includes shipping to Israel and import VAT/duty. Most Amazon US prices don't — flag it.
- If the user gives an ex-VAT IL price, normalise to inc-VAT before comparing.
- Don't recommend importing without factoring landed cost.
