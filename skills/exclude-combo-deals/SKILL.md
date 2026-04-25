---
name: exclude-combo-deals
description: Use when the user wants to search AliExpress and **hide products that carry the `Max Combo` badge** — i.e. exclude bundle/combo deals from the result set so only standalone listings remain. AliExpress provides no server-side filter for this on the listing page (validated — see `research/ui-selectors/listing-page-filters.md` §3); the skill therefore runs the standard search via `search-aliexpress`, then post-filters the visible product cards by reading the `Max Combo` badge text in the card DOM and dropping any card that has it. Trigger phrases — "search aliexpress without combo", "exclude max combo", "no bundle deals aliexpress", "aliexpress single items only", "hide combo deals".
---

# Exclude Combo Deals

Wrapper around `search-aliexpress` that drops `Max Combo` bundle listings from the results.

## Why this is a separate skill

`Max Combo` is a **per-product badge**, not a server-side filter — there is no `filterCode:*` toggle for it on the listing page. The only way to "exclude combo" is to fetch the full result set and drop badged cards client-side. This skill encapsulates that pattern so callers don't have to know about the badge mechanics.

## Inputs

Same as `search-aliexpress`, plus:

- `query` (required)
- `filters` (optional) — same shape: `{ choice, freeshipping, rating4plus, premium, ship_from }`
- `max_results` (optional, default 20) — applied **after** combo filtering, so to surface 20 non-combo cards the skill will scrape more than 20 raw cards.
- `over_fetch_factor` (optional, default 2) — multiplier for how much to over-fetch. With `max_results=20` and factor `2`, scrape up to 40 raw cards then drop combos and trim to 20.

## How

1. Delegate the search to the same flow as `search-aliexpress` (locale handshake → URL → filter toggles).
2. Scrape raw product cards (more than `max_results`, see over_fetch_factor).
3. For each card, detect the `Max Combo` badge.
4. Drop combos. Keep order.
5. Trim to `max_results` and return.

## Detecting the `Max Combo` badge

Validated in `research/ui-selectors/listing-page-filters.md` §3:

```html
<span class="lw_an">Max Combo</span>
```

The class `lw_an` is hashed and **will rotate** between deploys — anchor on the **text** instead, scoped to product card containers.

```js
// On the listing page, after results render:
const badged = new Set(
  [...document.querySelectorAll('span')]
    .filter(el => /^max combo$/i.test(el.textContent.trim()))
    .map(el => el.closest('a[href*="/item/"]'))
    .filter(Boolean)
);
```

Then when extracting product cards, drop any whose anchor element is in `badged`.

### Locale wrinkle

When `b_locale=iw_IL`, AliExpress may render the badge in Hebrew. Capture both:

```js
/^(max combo|מקס קומבו|קומבו)$/i.test(el.textContent.trim())
```

If the Hebrew rendering is ever observed and differs from the regex above, update the research notes and this skill.

## Output format

Same shape as `search-aliexpress`, with one extra line in the header:

```
Combo cards dropped: <N>   (raw scraped: <R>, returned: <max_results or fewer>)
```

If `R - N < max_results`, return what's available and note that the over-fetch wasn't enough — the user can re-run with a higher `over_fetch_factor`.

## Edge cases

- **Result page has fewer raw cards than `max_results × over_fetch_factor`**: just process what's there; report the actual counts.
- **Page has zero combos**: return `max_results` cards normally; report `Combo cards dropped: 0`.
- **All visible cards are combos**: report and return an empty list — the user will want to know that's what AliExpress is currently surfacing for this query.
- **Badge text rendered as part of a longer string** (e.g. `"Max Combo deal"`): the regex above is anchored, so it won't match; this is intentional. If observed in the wild, relax to `/\bmax combo\b/i` and document the change.

## Out of scope

- Does **not** detect generic "bundle" or "set" listings that aren't badged. AliExpress has many multi-pack listings (`Pack of 3`, `Set of 5`) with no Max Combo badge — these are not filtered. Add a separate skill if needed.
- Does **not** alter pagination behaviour beyond over-fetch.

## Validation checklist

1. The same locale + filter validation as `search-aliexpress` passes.
2. Raw-scrape count and combo-dropped count are both reported.
3. No card in the returned list has a descendant matching the Max Combo regex.
4. Returned count ≤ `max_results`.
