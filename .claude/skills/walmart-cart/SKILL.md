---
name: walmart-cart
description: Add items to a Walmart shopping cart using browser automation. Use this skill whenever the user wants to add groceries or products to their Walmart cart, do Walmart shopping, buy items on Walmart, or replenish supplies via Walmart. Trigger this skill even if the user just says something like "add these to my Walmart cart" or "do my Walmart shopping". Assumes the user is already logged in to walmart.com (session persists via CloakBrowser profile).
---

# Walmart Cart Skill

Add items to a Walmart cart using `walmart_cart.py`, which drives a stealth CloakBrowser (Playwright-based) window. Login session persists via `~/.walmart-cloak-profile`.

## Script Location

```
/Users/guy/dev/automation/claude-chrome-walmart/walmart_cart.py
```

## How to Run

Pass each item as a separate CLI argument:

```bash
python3 /Users/guy/dev/automation/claude-chrome-walmart/walmart_cart.py "hawaiian king rolls" "2% milk"
```

The script will:
1. Launch a stealth Chromium window using CloakBrowser (bot-detection bypass)
2. Use the persistent profile at `~/.walmart-cloak-profile` (preserves login session)
3. If not logged in, prompt the user to log in manually, then press Enter
4. Search for each item and pick the product with the highest "Bought X+ times" badge (falls back to first "Add to cart" button)
5. Click "Add to cart" via JS injection

## First Run

On first run, a browser window opens and the script prompts:
```
Not logged in — please log in to walmart.com in the browser window, then press Enter.
```
Log in to Walmart, then press Enter. Session is saved and subsequent runs skip this step.

## Reporting

After running the script, report back using its stdout output in this format:

```
Added to cart:
- [item] — [product name or label shown]

Could not add:
- [item] — [reason from script output]
```

If the script exits with an error or prints an `ERROR:` line, report that to the user and do not retry automatically.

## Item Preferences

Some items have preferred specific product names to search for:

- **Yoplait yogurt / Yoplait protein yogurt**: Use `"Yoplait Protein Yogurt Cultured Dairy Snack Cup, Mixed Berry Flavored, 5.6 oz"` or `"Yoplait Protein Yogurt Cultured Dairy Snack Cup, Strawberry Flavored, 5.6 oz"` as the search query instead of a generic term.

## Notes

- Do NOT use MCP browser tools (`mcp__claude-in-chrome__*`) for this task - use the script instead.
- If a CAPTCHA or sign-in wall is reported in the script output, stop and notify the user.
- The script selects based on "Bought X+ times" count; no manual product selection is needed.
- CloakBrowser uses a patched Chromium binary with fingerprint spoofing — helps avoid Walmart bot detection.
