---
name: convert-currency
description: Convert between ILS and the three common foreign currencies for Israeli shopping — USD, EUR, GBP — in either direction. Uses a live exchange-rate API with a fallback rate. Use whenever the user asks to convert a price or wants ILS↔USD/EUR/GBP figures.
---

# Convert Currency

Bidirectional conversion between ILS and USD/EUR/GBP. Supported pairs:

- USD ↔ ILS
- EUR ↔ ILS
- GBP ↔ ILS

(Cross-currency e.g. USD → EUR is out of scope — use a general FX tool for that.)

## Workflow

1. **Fetch the live rate** from `https://api.exchangerate-api.com/v4/latest/{base}` (use Bash + `curl` or the `WebFetch` tool). For example to get USD→ILS, fetch with base `USD` and read `rates.ILS`.
2. If the API fails, use these fallback rates (approximate, update if visibly wrong):
   - 1 USD ≈ 3.65 ILS
   - 1 EUR ≈ 3.95 ILS
   - 1 GBP ≈ 4.65 ILS
3. **Do the math** to the requested precision (default 2 decimals).
4. **Report** the result, the rate used, and whether it was live or fallback.

## Output

```
{amount} {from} = {result} {to}
Rate: 1 {from} = {rate} {to} ({live | fallback}, {date})
```

For multi-line conversions (e.g. converting a list of prices), use a small table.

## Rules

- Always show the rate you used. Never silently apply an unstated rate.
- If the user gives an ambiguous amount (e.g. just "100" without a currency), ask which direction.
- For Israeli shopping comparisons, prefer this skill over inline mental math.
