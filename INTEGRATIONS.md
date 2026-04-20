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

## Pharmacy

Pharmacy is shopping-adjacent — OTC prices, product availability, and kupat holim discount tiers all matter. Folded into this plugin rather than a health plugin.

- [`tomron/agent-skill-clalit-pharm-search`](https://github.com/tomron/agent-skill-clalit-pharm-search) — Clalit pharmacy search.
- [`alexpolonsky/agent-skill-maccabi-pharm-search`](https://github.com/alexpolonsky/agent-skill-maccabi-pharm-search) — Maccabi pharmacy search.
- [`skills-il/health-services`](https://github.com/skills-il/health-services) — broader health services. Check for pharmacy/drug-evaluation sub-skills worth lifting; the non-pharmacy parts belong in a future `Claude-Israel-Services` plugin.

## Status

None integrated yet. Next step: clone, read each manifest, decide:
- Depend on upstream MCP directly (preferred when it exists and is published).
- Wrap as a local skill that calls the MCP.
- Re-implement minimal logic if upstream is unmaintained or over-scoped.
