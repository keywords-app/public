# Pricing Table Consolidation Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Consolidate the 5-card pricing table into 3 cards (Starter monthly, Pro lifetime-primary, Scale lifetime-primary) in `keywords-landing-v4.html`.

**Architecture:** Single-file edit. Remove the second pricing row, restructure the 3 remaining cards in one grid, add monthly subscribe links below the lifetime CTAs on Pro and Scale.

**Tech Stack:** Static HTML/CSS

**Spec:** `docs/superpowers/specs/2026-03-14-pricing-table-consolidation-design.md`

---

## Chunk 1: All Changes

### Task 1: Update CSS

**Files:**
- Modify: `keywords-landing-v4.html:177-193` (CSS grid styles)

- [ ] **Step 1: Update the CSS comment and remove `.pricing-row-2`**

Replace lines 177-192 (the comment, `.pricing-grid`, and `.pricing-row-2` rules) with:

```css
/* 3-card layout: Starter, Pro, Scale in one row */
.pricing-grid {
  margin-top: 48px;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}
```

Also remove the `.pricing-row-2` rules inside both responsive media queries:
- Inside `@media (max-width: 860px)`: remove `.pricing-row-2 { grid-template-columns: 1fr 1fr; max-width: 100%; }`
- Inside `@media (max-width: 560px)`: remove `.pricing-row-2 { grid-template-columns: 1fr; max-width: 100%; }`

### Task 2: Replace pricing HTML

**Files:**
- Modify: `keywords-landing-v4.html:413-500` (pricing cards HTML)

- [ ] **Step 2: Replace the pricing grid and row-2 HTML**

Replace everything from `<!-- Row 1: Starter, Pro, Scale -->` through the closing `</div>` of `.pricing-row-2` (lines 413-500) with:

```html
<div class="pricing-grid">
  <!-- Starter: monthly only, price anchor -->
  <div class="p-card">
    <div class="p-name">Starter</div>
    <div class="p-price"><sup>$</sup>27</div>
    <div class="p-cadence">per month</div>
    <div class="p-rule"></div>
    <ul class="p-features">
      <li>Up to 10,000 search terms/mo</li>
      <li>Good for 1–2 accounts</li>
      <li>Full triage + Negative Match Maker</li>
      <li>AI intent clustering</li>
      <li>Savings tracking</li>
      <li>iOS app included</li>
    </ul>
    <button class="p-btn p-btn-ghost">Get started →</button>
  </div>

  <!-- Pro: lifetime-primary, combined -->
  <div class="p-card ltd">
    <div class="ltd-pill">⚡ 50 SPOTS ONLY</div>
    <div class="p-name">Pro</div>
    <div class="p-price"><sup>$</sup>297</div>
    <div class="p-cadence">one time — yours forever</div>
    <div class="p-rule"></div>
    <ul class="p-features">
      <li>Up to 100,000 search terms/mo</li>
      <li>Good for 10–20 accounts</li>
      <li>Full triage + Negative Match Maker</li>
      <li>AI intent clustering</li>
      <li>Savings tracking</li>
      <li>iOS app included</li>
    </ul>
    <button class="p-btn p-btn-green">Get lifetime access →</button>
    <p class="p-upgrade"><a href="#" class="p-monthly-link">or subscribe monthly for $57/mo</a></p>
  </div>

  <!-- Scale: lifetime-primary, combined -->
  <div class="p-card ltd">
    <div class="ltd-pill">⚡ 50 SPOTS ONLY</div>
    <div class="p-name">Scale</div>
    <div class="p-price"><sup>$</sup>497</div>
    <div class="p-cadence">one time — yours forever</div>
    <div class="p-rule"></div>
    <ul class="p-features">
      <li>Up to 200,000 search terms/mo</li>
      <li>+$30/mo per additional 100k</li>
      <li>Full triage + Negative Match Maker</li>
      <li>AI intent clustering</li>
      <li>Savings tracking</li>
      <li>iOS app included</li>
    </ul>
    <button class="p-btn p-btn-green">Get lifetime access →</button>
    <p class="p-upgrade"><a href="#" class="p-monthly-link">or subscribe monthly for $87/mo</a></p>
  </div>
</div>
```

### Task 3: Add monthly link CSS

**Files:**
- Modify: `keywords-landing-v4.html` (CSS section, after the `.p-upgrade` rule)

- [ ] **Step 3: Add `.p-monthly-link` style**

Add after the `.p-upgrade` rule:

```css
.p-monthly-link { color: var(--text-dim); text-decoration: none; transition: color 0.2s; }
.p-monthly-link:hover { color: var(--text-mid); text-decoration: underline; }
```

### Task 4: Verify and commit

- [ ] **Step 4: Open in browser and verify**

Open `keywords-landing-v4.html` in a browser. Confirm:
- 3 cards in a single row at full width
- Starter is monthly-only with ghost button, no text below button
- Pro and Scale have green `ltd` treatment, lifetime prices, and "or subscribe monthly" links below CTA
- Responsive: 2-col at 860px (Scale wraps alone), 1-col at 560px
- Final CTA section unchanged

- [ ] **Step 5: Commit**

```bash
git add keywords-landing-v4.html
git commit -m "Consolidate pricing table from 5 cards to 3"
```
