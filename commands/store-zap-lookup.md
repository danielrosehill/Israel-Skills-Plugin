---
name: store-zap-lookup
description: Given an Israeli retailer name (Hebrew or English), find its Zap.co.il profile URL and retrieve the seller rating and review count. Useful for vetting an unfamiliar retailer before buying.
---

# Store Zap Lookup

Resolve an Israeli retailer to its Zap seller profile and pull the rating.

## When to use

- User found a low price on an unfamiliar retailer and wants to check reputation.
- Comparing retailers on Zap and wants the rating surfaced alongside price.
- Adding a new store via `/add-store` and wants to record the Zap profile.

## Backend

Playwright (Zap has bot detection). WebFetch may work for the profile page itself once the URL is known.

## Workflow

1. **Load the merged store list** — check if the retailer already has a `zap_profile` URL recorded (upstream `stores.json` + `<plugin-data-dir>/user-stores.json (resolved via $CLAUDE_USER_DATA — see docs/search-strategies.md)`; see `docs/search-strategies.md`).
2. **If known,** navigate directly to the profile URL (`https://www.zap.co.il/shop.aspx?sog=shop-<id>`).
3. **If unknown,** search Zap for the retailer:
   - Navigate `https://www.zap.co.il/` and use the shop search, OR
   - Google `site:zap.co.il "<retailer name>"` and look for a `shop.aspx?sog=shop-<id>` URL.
4. **On the profile page, extract:**
   - Seller rating (usually out of 5 or as a percentage)
   - Review count
   - Years active on Zap (if shown)
   - Any flags (verified seller, "Zap recommended", etc.)

## Output

```
## {retailer} — Zap profile

- **Zap URL:** {url}
- **Rating:** {rating} ({review_count} reviews)
- **Years on Zap:** {years or "not shown"}
- **Flags:** {verified / recommended / none}

### Notes
- {anything unusual — low rating, recent negative reviews, no Zap presence}
```

If the retailer has no Zap profile at all, say so — it's a soft signal about smaller or very new shops. Don't invent a rating.

## Rules

- Never fabricate a rating. If the page didn't load or the rating wasn't visible, say so.
- Cross-check the profile matches the retailer (Zap has some duplicate / similarly-named profiles).
