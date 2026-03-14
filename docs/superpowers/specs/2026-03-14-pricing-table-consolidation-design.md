# Pricing Table Consolidation: 5 Cards to 3

## Problem

The current pricing section has 5 cards across two rows (Starter, Pro, Scale monthly + Pro Lifetime, Scale Lifetime). Feedback says 5 is too many — it creates decision fatigue and splits attention between monthly and lifetime options.

## Design

Consolidate to **3 cards in a single row**: Starter, Pro, Scale. Pro and Scale each combine their monthly and lifetime options into one card, with lifetime as the primary offer.

### Card: Starter (monthly only)

- Price: $27/mo
- Feature list (unchanged): 10k terms, 1-2 accounts, triage + NMM, AI clustering, savings tracking, iOS
- CTA: ghost button "Get started ->"
- Nothing below the button (upgrade-to-lifetime note removed)
- Purpose: low price anchor that makes lifetime deals look like obvious value

### Card: Pro (lifetime-primary, combined)

- Card name: "PRO" (not "Pro Lifetime" — drop the suffix)
- "⚡ 50 SPOTS ONLY" pill badge (retain emoji)
- Green border/glow treatment (`ltd` card style)
- Price: $297, cadence "one time - yours forever"
- Feature list (6 bullets, monthly-style — no lifetime-specific bullets like "No monthly bill" or "All future updates"): 100k terms, 10-20 accounts, triage + NMM, AI clustering, savings tracking, iOS
- CTA: green "Get lifetime access ->" button
- Below button: small text link "or subscribe monthly for $57/mo"

### Card: Scale (lifetime-primary, combined)

- Card name: "SCALE" (not "Scale Lifetime")
- "⚡ 50 SPOTS ONLY" pill badge (retain emoji)
- Green border/glow treatment (`ltd` card style)
- Price: $497, cadence "one time - yours forever"
- Feature list (6 bullets): 200k terms, +$30/mo per additional 100k, triage + NMM, AI clustering, savings tracking, iOS
- CTA: green "Get lifetime access ->" button
- Below button: small text link "or subscribe monthly for $87/mo"

### Layout

- Single `pricing-grid` with `grid-template-columns: repeat(3, 1fr)`
- Remove `pricing-row-2` div entirely
- Responsive breakpoints unchanged: 2-col at 860px, 1-col at 560px

### What's removed

- The 3 monthly-only cards (Starter stays, Pro/Scale monthly absorbed into combined cards)
- The `pricing-row-2` second row
- The "Upgrade to lifetime within 30 days" notes on all cards

### Monthly subscribe link style

- Reuse the `.p-upgrade` class (same size/color/alignment)
- Wrap the monthly price in an `<a>` tag with `href="#"` (placeholder) styled to inherit color with underline on hover
- Not a button — just a subtle link so lifetime stays dominant

### Scope boundary

- Final CTA section ("50 spots. One price. No recurring bill.") is untouched
- Update the stale CSS comment on line ~177 referencing "5-card layout"
- At 860px responsive, 3 cards in a 2-col grid means the 3rd card wraps alone — this is acceptable

## File

`keywords-landing-v4.html` — pricing section starting at line ~408
