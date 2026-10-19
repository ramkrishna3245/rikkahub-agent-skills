---
name: price-compare
description: Compare a product's price and availability across e-commerce sites (Amazon, Flipkart, Croma, Meesho, etc.) and report the best real deal considering delivery, seller trust, and variants. Load when the user asks about prices, deals, "where is X cheapest", or whether something is worth buying.
---

# Price Compare

Find the genuinely best deal — not just the lowest number on the page.

## Workflow

1. **Pin the exact product.** Brand + model + variant (size, colour, RAM/storage,
   pack count). Comparing a 128 GB against a 256 GB phone is not a comparison.
2. **Search across sites.** For each site:
   ```
   open_url("<site search URL for the product>", package_name="com.android.chrome")
   ```
   Typical patterns:
   - Amazon:  `https://www.amazon.in/s?k=<query>`
   - Flipkart: `https://www.flipkart.com/search?q=<query>`
   - Croma:    `https://www.croma.com/searchB?q=<query>`
   - Meesho:   `https://www.meesho.com/search?q=<query>`
3. **For each listing capture:**
   - price (and MRP / discount)
   - seller name and rating
   - delivery date & charge
   - variant confirmation (does it match?)
   - stock status
   - return/replacement policy
4. **Normalise** to a single comparable number:
   `effective price = price + delivery − cashback (only if certain)`
5. **Rank** by effective price, then tie-break on seller trust and delivery speed.
6. **Report** and save.

## Report format

```
write_text_file path="<workspace>/prices/<YYYY-MM-DD>-<slug>.md"
```

```markdown
# <Product>

**Variant compared:** <exact variant>
**Date:** <YYYY-MM-DD>

| Site | Price | Delivery | Seller rating | Effective | Notes |
|---|---|---|---|---|---|
| Amazon | ₹X | ₹0, 2 days | 4.4 (Acme) | ₹X | ... |
| Flipkart | ₹Y | Free, 4 days | 4.1 | ₹Y | ... |

**Best:** <site> at ₹<effective price> — <one-line why>
**Second best:** ...
**Verdict:** buy now / wait — <reason>
```

## Deal-quality checks

Call these out explicitly when found:

- **Fake discount** — MRP inflated so the "60% off" is meaningless.
- **Different variant** — cheaper because it is a smaller pack or older model.
- **Third-party seller risk** — much cheaper but a new/low-rated seller; flag it.
- **No-cost EMI / bank offers** — mention real card offers if shown on the page.
- **Out of stock in the user's pincode** — check delivery availability, not just price.
- **Refurbished/open-box** — say so loudly if the listing is not new.

## Guardrails

- Never add an item to cart, checkout, or buy anything. Comparison only.
- Do not print a price you did not actually read on the page.
- Prices move constantly — always timestamp the report and say when it was read.
- Note that prices can differ by pincode; mention the pincode you compared against.
