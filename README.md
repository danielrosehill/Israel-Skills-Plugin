# Israel-Shopping

![banner](banner.png)

Israeli shopping workflows for Claude Code — tech retailers, Zap price comparison, Hebrew term resolution, ILS conversion, and RRP/market checks.

> Renamed from `Israel-Skills` on 2026-04-20. The scope was always shopping; the new name reflects it. Non-shopping Israel utilities will live in a separate plugin.

## Skills (auto-invoked)

- `il-tech-search` — search Ivory, KSP, Bug, TMS in Hebrew with Playwright fallback for bot-detected retailers.

More skills to come: grocery search (Shufersal, Rami Levy), pharmacy search (Clalit, Maccabi), RRP check, ILS conversion. Upstream candidates tracked in [`INTEGRATIONS.md`](INTEGRATIONS.md).

## Commands (slash)

Retail & shopping:

- `israel-search-zap` — query zap.co.il, the canonical Israeli price-comparison aggregator
- `israel-search-google-il` — `site:.il` / Hebrew-keyword Google discovery
- `israel-search-main-tech-stores` — KSP, Ivory, Bug, TMS
- `israel-search-major-retailers` — Ace, Home Center, Office Depot, Audioline
- `israel-search-by-category` — dispatch to stores matching a category slug from the store DB
- `israel-discover-hebrew-term` — reverse-lookup the canonical Hebrew noun for a product class
- `israel-compare-to-international` — compare an Israeli price against international RRP
- `israel-convert-currency` — ILS ↔ USD/EUR/GBP conversion
- `israel-market-check` — quick IL vs international RRP sanity check
- `israel-source` — apply the Israeli sourcing waterfall (tier-1 tech → major retailers → Zap)
- `israel-add-store` — append a vendor to the Israeli store database with auto-dedup

## Data

Store metadata is read live from [`danielrosehill/Israel-Online-Stores`](https://github.com/danielrosehill/Israel-Online-Stores) (`stores.json`) — 800+ Israeli retailers with tier, delivery, Eilat-door, and Zap-profile metadata.

## Installation

```bash
claude plugins marketplace add danielrosehill/Claude-Code-Plugins
claude plugins install israel-shopping@danielrosehill
```

## License

MIT
