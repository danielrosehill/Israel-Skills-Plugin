# Integrations — upstream repos to fold in

Queue of community skills/MCPs to evaluate and wrap as skills in this plugin.

## Grocery

- [`matipojo/shufersal-mcp`](https://github.com/matipojo/shufersal-mcp) — MCP server for Shufersal (Israel's largest supermarket chain). Primary candidate for a grocery-search skill.
- [`aloncarmel/supermeskill`](https://github.com/aloncarmel/supermeskill) — "SuperMe" skill. Evaluate scope before wrapping.
- [`OpenIsraeliSupermarkets/israeli-supermarket-scarpers`](https://github.com/OpenIsraeliSupermarkets/israeli-supermarket-scarpers) — Python scraper library covering Shufersal, Rami Levy, Victory, etc. Lower-level; useful if the MCPs don't cover a chain we need.
- [`danielJL-altius/israelgrocerymcp`](https://github.com/danielJL-altius/israelgrocerymcp) — another grocery MCP. Compare with shufersal-mcp before picking.
- [`asafr93-rosa/shopping-list-il`](https://github.com/asafr93-rosa/shopping-list-il) — shopping-list tool. Complementary to price search.

## International import / cross-border

- [`rootsbymenda/iherbchecker`](https://github.com/rootsbymenda/iherbchecker) — iHerb checker. iHerb is a go-to for supplements shipping to Israel; a skill wrapping this would complement the domestic pharmacy search and the existing AliExpress/Amazon callout in `il-tech-search`.
- [`justinritchie/aliexpress-mcp-server`](https://github.com/justinritchie/aliexpress-mcp-server) — AliExpress MCP. High priority: AliExpress is the dominant cross-border vendor for Israeli tech/home buyers. Wrap as a skill for product lookup + price-to-ILS comparison against domestic retailers.
- **Amazon (no candidate yet)** — need an MCP covering amazon.com product data **and** Israel shipping eligibility (ships-to-IL flag, import fee deposit estimate). Evaluate existing Amazon MCPs; if none handle IL shipping, scope a minimal wrapper around the PA-API or a scraper that checks the "Deliver to Israel" availability on the product page.
- **Newegg (no candidate yet)** — frequent reference point for ROW tech pricing. Lower priority than Amazon (Newegg's IL shipping is patchier), but worth a lightweight product-lookup skill for price comparison.
- **Currency conversion (no candidate yet)** — load-bearing for every cross-border comparison (USD/EUR/GBP → ILS). Find or wrap an FX-rate MCP (e.g. exchangerate.host, Frankfurter, open.er-api.com). Must expose mid-market rate + timestamp so comparisons are reproducible. The existing `israel-convert-currency` skill currently handles this inline — an MCP would let it pull live rates instead of hardcoded or web-scraped values.

## Pharmacy

Pharmacy is shopping-adjacent — OTC prices, product availability, and kupat holim discount tiers all matter. Folded into this plugin rather than a health plugin.

- [`tomron/agent-skill-clalit-pharm-search`](https://github.com/tomron/agent-skill-clalit-pharm-search) — Clalit pharmacy search.
- [`alexpolonsky/agent-skill-maccabi-pharm-search`](https://github.com/alexpolonsky/agent-skill-maccabi-pharm-search) — Maccabi pharmacy search.
- [`skills-il/health-services`](https://github.com/skills-il/health-services) — broader health services. Check for pharmacy/drug-evaluation sub-skills worth lifting; the non-pharmacy parts belong in a future `Claude-Israel-Services` plugin.

## Consumer rights / legal (optional reference)

- [`Ansvar-Systems/israel-law-mcp`](https://github.com/Ansvar-Systems/israel-law-mcp) — hosted MCP exposing 66 Israeli statutes (`https://mcp.ansvar.eu/law-il/mcp`). Tangential to shopping but useful for return-rights / cooling-off-period / price-display / warranty questions **if** Consumer Protection Law (חוק הגנת הצרכן) is covered — the README highlights privacy/corporate/penal, so verify coverage before wrapping. Candidate for a future `israel-consumer-rights` skill.

## Status

None integrated yet. Next step: clone, read each manifest, decide:
- Depend on upstream MCP directly (preferred when it exists and is published).
- Wrap as a local skill that calls the MCP.
- Re-implement minimal logic if upstream is unmaintained or over-scoped.
