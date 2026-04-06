# Hero Animation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a ~18-second looping CSS/JS animation that replaces the hero image placeholder in the funnel landing page, showing the full Keywords.app workflow: connect → scan → cluster → sort → push → savings.

**Architecture:** Single-file modification to `funnel-start.html`. The animation is a self-contained HTML element with its own CSS keyframes and a JS sequencer that drives 6 beats. An IntersectionObserver pauses the animation when off-screen. Variant data (cluster labels, savings number) is defined as JS constants for easy A/B swapping.

**Tech Stack:** CSS @keyframes, CSS transitions, vanilla JS timing (setTimeout). No libraries.

**Spec:** `docs/superpowers/specs/2026-04-06-hero-animation-design.md`

---

## File Structure

One file modified:
- **Modify:** `funnel-start.html` — replace `.hero-image-placeholder` with animation element, add CSS and JS

---

## Chunk 1: Animation Container and Beat Sequencer

### Task 1: Create animation container and JS beat sequencer

**Files:**
- Modify: `funnel-start.html`

- [ ] **Step 1: Replace the hero image placeholder with the animation container**

Use the `frontend-design` skill. In `funnel-start.html`, find the element:
```html
<div class="hero-image-placeholder">
  [Screenshot: triage screen showing organized search term clusters with savings tracker]
</div>
```
(approximately line 1596 in `#view-landing`)

Replace it with:
```html
<div class="hero-anim" id="hero-anim">
  <div class="hero-anim-stage" id="hero-anim-stage">
    <!-- Beat content is injected/shown by JS sequencer -->
    <div class="ha-beat ha-beat-connect" id="ha-beat-1"></div>
    <div class="ha-beat ha-beat-scan" id="ha-beat-2"></div>
    <div class="ha-beat ha-beat-clusters" id="ha-beat-3"></div>
    <div class="ha-beat ha-beat-sort" id="ha-beat-4"></div>
    <div class="ha-beat ha-beat-push" id="ha-beat-5"></div>
    <div class="ha-beat ha-beat-endcard" id="ha-beat-6"></div>
  </div>
</div>
```

**CSS for the container** (add to end of `<style>` block):

```css
/* ===== Hero Animation ===== */
.hero-anim {
  width: 100%;
  max-width: 460px;
  aspect-ratio: 4 / 3;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  box-shadow: var(--shadow-md);
  overflow: hidden;
  position: relative;
}

.hero-anim-stage {
  width: 100%;
  height: 100%;
  position: relative;
}

.ha-beat {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 24px;
  opacity: 0;
  transition: opacity 0.4s ease;
  pointer-events: none;
}

.ha-beat.active {
  opacity: 1;
}
```

**JS for the beat sequencer** (add to end of `<script>` block):

```js
/* ===== Hero Animation Sequencer ===== */
(function() {
  // Variant-ready constants — swap these for A/B tests
  const HA_CLUSTERS = [
    { label: 'plumber near me', good: true },
    { label: 'emergency plumber', good: true },
    { label: '24hr plumber', good: true },
    { label: 'plumbing school', good: false }
  ];
  const HA_SAVINGS = 728;
  const HA_TIMINGS = [2000, 3000, 4000, 4000, 3000, 2000]; // ms per beat

  let haTimer = null;
  let haRunning = false;

  function startHeroAnim() {
    if (haRunning) return;
    haRunning = true;
    runBeatSequence();
  }

  function stopHeroAnim() {
    haRunning = false;
    if (haTimer) { clearTimeout(haTimer); haTimer = null; }
  }

  function runBeatSequence() {
    let beatIndex = 0;
    function nextBeat() {
      if (!haRunning) return;
      // Hide all beats
      document.querySelectorAll('.ha-beat').forEach(b => b.classList.remove('active'));
      // Show current beat
      const beat = document.getElementById('ha-beat-' + (beatIndex + 1));
      if (beat) beat.classList.add('active');
      // Trigger beat-specific animations
      if (window['ha_startBeat' + (beatIndex + 1)]) {
        window['ha_startBeat' + (beatIndex + 1)]();
      }
      // Schedule next
      haTimer = setTimeout(function() {
        beatIndex = (beatIndex + 1) % 6;
        nextBeat();
      }, HA_TIMINGS[beatIndex]);
    }
    nextBeat();
  }

  // IntersectionObserver: pause when off-screen
  var heroAnimEl = document.getElementById('hero-anim');
  if (heroAnimEl) {
    var haObserver = new IntersectionObserver(function(entries) {
      entries.forEach(function(entry) {
        if (entry.isIntersecting) { startHeroAnim(); }
        else { stopHeroAnim(); }
      });
    }, { threshold: 0.3 });
    haObserver.observe(heroAnimEl);
  }

  // Expose constants for beat implementations
  window.HA_CLUSTERS = HA_CLUSTERS;
  window.HA_SAVINGS = HA_SAVINGS;
})();
```

- [ ] **Step 2: Verify in browser**

Open `funnel-start.html`. The hero should show a white rounded card where the dashed placeholder was. The animation container is visible but beats are empty (blank white cycling). Confirm the sequencer runs by checking that `.ha-beat` elements cycle their `.active` class (use browser dev tools).

- [ ] **Step 3: Commit**

```bash
git add funnel-start.html
git commit -m "feat: hero animation container and beat sequencer with IntersectionObserver"
```

---

## Chunk 2: Beats 1-3 (Connect, Scan, Clusters)

### Task 2: Beat 1 — Connect Google Ads

**Files:**
- Modify: `funnel-start.html`

- [ ] **Step 1: Add Beat 1 HTML and CSS**

Use the `frontend-design` skill. Fill in `#ha-beat-1` with the Connect beat:

**HTML** inside `#ha-beat-1`:
- A button styled like `.funnel-btn-google` (white bg, Google G icon, border) but smaller to fit the animation stage. Text: "Connect Google Ads". Use a class like `.ha-google-btn`.
- A green checkmark element (`.ha-checkmark`) that is initially hidden (`opacity: 0; transform: scale(0);`)

**CSS:**
- `.ha-google-btn` — styled to look like the funnel's Google button but scaled down (~80% size, `font-size: 13px`, `padding: 10px 20px`)
- `.ha-google-btn.pressed` — `transform: scale(0.95);` transition
- `.ha-google-btn.done` — `opacity: 0; transform: scale(0.9);` transition
- `.ha-checkmark` — green circle (~40px), white checkmark inside (CSS-only, same technique as the `#view-ready` success circle but smaller)
- `.ha-checkmark.show` — `opacity: 1; transform: scale(1);` with a spring-like transition (`cubic-bezier(0.34, 1.56, 0.64, 1)`)

**JS** — define `window.ha_startBeat1`:
```js
window.ha_startBeat1 = function() {
  var btn = document.querySelector('.ha-google-btn');
  var check = document.querySelector('#ha-beat-1 .ha-checkmark');
  if (!btn || !check) return;
  // Reset
  btn.className = 'ha-google-btn';
  btn.style.display = '';
  check.classList.remove('show');
  // After 600ms: press
  setTimeout(function() { btn.classList.add('pressed'); }, 600);
  // After 900ms: release + fade out
  setTimeout(function() { btn.classList.remove('pressed'); btn.classList.add('done'); }, 900);
  // After 1200ms: show checkmark
  setTimeout(function() { check.classList.add('show'); }, 1200);
};
```

- [ ] **Step 2: Verify in browser**

Watch Beat 1 play: button appears → press → fades out → checkmark pops in. Should feel snappy and satisfying.

- [ ] **Step 3: Commit**

```bash
git add funnel-start.html
git commit -m "feat: hero animation beat 1 — connect Google Ads"
```

---

### Task 3: Beat 2 — AI Scan

**Files:**
- Modify: `funnel-start.html`

- [ ] **Step 1: Add Beat 2 HTML and CSS**

Use the `frontend-design` skill. Fill in `#ha-beat-2`:

**HTML** inside `#ha-beat-2`:
- A progress bar container (`.ha-progress-bar`) with a fill element (`.ha-progress-fill`)
- A pulsing dot (`.ha-pulse-dot`)
- Phase text element (`.ha-phase-text`) — starts with "Analyzing search terms..."

**CSS:**
- `.ha-progress-bar` — `width: 70%; height: 6px; background: var(--border); border-radius: 3px; overflow: hidden;`
- `.ha-progress-fill` — `height: 100%; background: var(--green); border-radius: 3px; width: 0%; transition: width 2.5s ease-out;`
- `.ha-progress-fill.filling` — `width: 85%;`
- `.ha-pulse-dot` — `width: 8px; height: 8px; background: var(--green); border-radius: 50%;` with a pulse keyframe animation
- `.ha-phase-text` — `font-size: 13px; color: var(--text-mid); margin-top: 14px; transition: opacity 0.3s;`

**JS** — define `window.ha_startBeat2`:
```js
window.ha_startBeat2 = function() {
  var fill = document.querySelector('.ha-progress-fill');
  var text = document.querySelector('.ha-phase-text');
  if (!fill || !text) return;
  // Reset
  fill.classList.remove('filling');
  fill.style.width = '0%';
  text.textContent = 'Analyzing search terms...';
  text.style.opacity = '1';
  // Start fill
  requestAnimationFrame(function() {
    requestAnimationFrame(function() { fill.classList.add('filling'); });
  });
  // Switch text at 1500ms
  setTimeout(function() {
    text.style.opacity = '0';
    setTimeout(function() {
      text.textContent = 'Building clusters...';
      text.style.opacity = '1';
    }, 300);
  }, 1500);
};
```

- [ ] **Step 2: Verify in browser**

Beat 2: progress bar fills, text cycles from "Analyzing..." to "Building clusters...". Pulsing dot animates.

- [ ] **Step 3: Commit**

```bash
git add funnel-start.html
git commit -m "feat: hero animation beat 2 — AI scan progress"
```

---

### Task 4: Beat 3 — Clusters Appear

**Files:**
- Modify: `funnel-start.html`

- [ ] **Step 1: Add Beat 3 HTML and CSS**

Use the `frontend-design` skill. Fill in `#ha-beat-3`:

**HTML** inside `#ha-beat-3`:
- A container `.ha-clusters` holding 4 cluster cards
- Each card (`.ha-cluster-card`) has:
  - A small text label (the search term cluster name)
  - The last one has an additional class `.bad` for the red highlight
- Cards are generated from `HA_CLUSTERS` but hardcode them in HTML for simplicity:
  ```html
  <div class="ha-cluster-card" style="animation-delay: 0s">plumber near me</div>
  <div class="ha-cluster-card" style="animation-delay: 0.15s">emergency plumber</div>
  <div class="ha-cluster-card" style="animation-delay: 0.3s">24hr plumber</div>
  <div class="ha-cluster-card bad" style="animation-delay: 0.45s">plumbing school</div>
  ```

**CSS:**
- `.ha-clusters` — `display: flex; flex-direction: column; gap: 8px; width: 80%;`
- `.ha-cluster-card` — `background: var(--card); border: 1px solid var(--border); border-radius: 8px; padding: 10px 16px; font-size: 13px; color: var(--text); font-weight: 500; opacity: 0; transform: translateY(12px);`
- When parent beat is `.active`: animate in using a keyframe `ha-slide-in` that goes from `opacity:0; translateY(12px)` to `opacity:1; translateY(0)`. Apply `animation: ha-slide-in 0.35s ease forwards;` with the staggered `animation-delay` from the inline style.
- `.ha-cluster-card.bad` — `border-color: #ea4335; background: rgba(234,67,53,0.05); color: #c62828;`

**JS** — define `window.ha_startBeat3`:
```js
window.ha_startBeat3 = function() {
  // Reset card animations by briefly removing and re-adding them
  var cards = document.querySelectorAll('#ha-beat-3 .ha-cluster-card');
  cards.forEach(function(card) {
    card.style.animation = 'none';
    card.offsetHeight; // force reflow
    card.style.animation = '';
  });
};
```

- [ ] **Step 2: Verify in browser**

Beat 3: 4 cards slide in staggered. First 3 have normal styling, last one ("plumbing school") is red-highlighted.

- [ ] **Step 3: Commit**

```bash
git add funnel-start.html
git commit -m "feat: hero animation beat 3 — clusters appear"
```

---

## Chunk 3: Beats 4-6 (Sort, Push, End Card)

### Task 5: Beat 4 — Sort into Keep/Block

**Files:**
- Modify: `funnel-start.html`

- [ ] **Step 1: Add Beat 4 HTML, CSS, and JS**

Use the `frontend-design` skill. Fill in `#ha-beat-4`:

This is the most complex beat. Two columns, cards animate into them, savings counter ticks up.

**HTML** inside `#ha-beat-4`:
```html
<div class="ha-sort-stage">
  <div class="ha-sort-columns">
    <div class="ha-sort-col ha-sort-keep">
      <div class="ha-sort-col-header">Keep</div>
      <div class="ha-sort-col-cards" id="ha-keep-cards"></div>
    </div>
    <div class="ha-sort-col ha-sort-block">
      <div class="ha-sort-col-header ha-sort-col-header-block">Block</div>
      <div class="ha-sort-col-cards" id="ha-block-cards"></div>
    </div>
  </div>
  <div class="ha-savings-counter">
    <span class="ha-savings-label">Savings:</span>
    <span class="ha-savings-value" id="ha-savings-value">$0/mo</span>
  </div>
</div>
```

**CSS:**
- `.ha-sort-stage` — `width: 90%; display: flex; flex-direction: column; gap: 12px;`
- `.ha-sort-columns` — `display: grid; grid-template-columns: 1fr 1fr; gap: 12px;`
- `.ha-sort-col` — `border-radius: 8px; padding: 8px; min-height: 100px;`
- `.ha-sort-keep` — `background: rgba(45,154,73,0.06); border: 1px solid rgba(45,154,73,0.18);`
- `.ha-sort-block` — `background: rgba(234,67,53,0.06); border: 1px solid rgba(234,67,53,0.18);`
- `.ha-sort-col-header` — `font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.08em; color: var(--green); margin-bottom: 8px;`
- `.ha-sort-col-header-block` — `color: #ea4335;`
- `.ha-sort-col-cards` — `display: flex; flex-direction: column; gap: 4px;`
- `.ha-sort-card` — `background: var(--card); border: 1px solid var(--border); border-radius: 6px; padding: 6px 10px; font-size: 11px; color: var(--text); opacity: 0; transform: translateX(20px); transition: opacity 0.3s, transform 0.3s;` (cards slide in from the side)
- `.ha-sort-card.placed` — `opacity: 1; transform: translateX(0);`
- `.ha-sort-card.bad` — `border-color: rgba(234,67,53,0.3);`
- `.ha-savings-counter` — `text-align: center; margin-top: 8px;`
- `.ha-savings-label` — `font-size: 12px; color: var(--text-dim);`
- `.ha-savings-value` — `font-family: 'DM Serif Display', serif; font-size: 22px; color: var(--green); margin-left: 6px;`

**JS** — define `window.ha_startBeat4`:
```js
window.ha_startBeat4 = function() {
  var keepCol = document.getElementById('ha-keep-cards');
  var blockCol = document.getElementById('ha-block-cards');
  var savingsEl = document.getElementById('ha-savings-value');
  if (!keepCol || !blockCol || !savingsEl) return;

  // Clear columns
  keepCol.innerHTML = '';
  blockCol.innerHTML = '';
  savingsEl.textContent = '$0/mo';

  var clusters = window.HA_CLUSTERS;
  var delay = 400;

  clusters.forEach(function(cluster, i) {
    setTimeout(function() {
      var card = document.createElement('div');
      card.className = 'ha-sort-card' + (cluster.good ? '' : ' bad');
      card.textContent = cluster.label;
      if (cluster.good) { keepCol.appendChild(card); }
      else { blockCol.appendChild(card); }
      // Trigger slide-in
      requestAnimationFrame(function() {
        requestAnimationFrame(function() { card.classList.add('placed'); });
      });
    }, delay * (i + 1));
  });

  // Savings counter: tick from 0 to ~500 during this beat (rest in beat 5)
  var target = Math.round(window.HA_SAVINGS * 0.7); // 70% of total during sort
  var current = 0;
  var steps = 30;
  var interval = 3000 / steps; // spread over ~3s
  var increment = target / steps;
  var counterInterval = setInterval(function() {
    current += increment;
    if (current >= target) { current = target; clearInterval(counterInterval); }
    savingsEl.textContent = '$' + Math.round(current) + '/mo';
  }, interval);

  // Store interval for cleanup
  window._haSavingsInterval = counterInterval;
  window._haSavingsCurrent = function() { return current; };
};
```

- [ ] **Step 2: Verify in browser**

Beat 4: Two columns appear. Cards slide in staggered — 3 to Keep, 1 to Block. Savings counter ticks from $0 to ~$510.

- [ ] **Step 3: Commit**

```bash
git add funnel-start.html
git commit -m "feat: hero animation beat 4 — sort into keep/block with savings counter"
```

---

### Task 6: Beat 5 — Push to Google

**Files:**
- Modify: `funnel-start.html`

- [ ] **Step 1: Add Beat 5 HTML, CSS, and JS**

Use the `frontend-design` skill. Fill in `#ha-beat-5`:

**HTML** inside `#ha-beat-5`:
```html
<div class="ha-push-stage">
  <button class="ha-push-btn" id="ha-push-btn">Push to Google</button>
  <div class="ha-push-check" id="ha-push-check"></div>
  <div class="ha-savings-counter">
    <span class="ha-savings-label">Savings:</span>
    <span class="ha-savings-value" id="ha-savings-value-2">$510/mo</span>
  </div>
</div>
```

**CSS:**
- `.ha-push-stage` — `display: flex; flex-direction: column; align-items: center; gap: 16px;`
- `.ha-push-btn` — styled like `.funnel-btn-primary` but smaller: `background: var(--green); color: #fff; font-family: 'Figtree', sans-serif; font-weight: 600; font-size: 14px; padding: 11px 24px; border-radius: 8px; border: none; cursor: default; transition: transform 0.15s, box-shadow 0.15s; box-shadow: 0 2px 12px rgba(45,154,73,0.2);`
- `.ha-push-btn.pressed` — `transform: scale(0.95); box-shadow: 0 1px 6px rgba(45,154,73,0.15);`
- `.ha-push-btn.done` — `opacity: 0; transform: scale(0.9); transition: opacity 0.3s, transform 0.3s;`
- `.ha-push-check` — same as Beat 1's `.ha-checkmark` style (green circle, white check, initially hidden). Reuse the same CSS class `.ha-checkmark` if possible, or duplicate the styles.
- `.ha-push-check.show` — `opacity: 1; transform: scale(1);`

**JS** — define `window.ha_startBeat5`:
```js
window.ha_startBeat5 = function() {
  var btn = document.getElementById('ha-push-btn');
  var check = document.getElementById('ha-push-check');
  var savingsEl = document.getElementById('ha-savings-value-2');
  if (!btn || !check || !savingsEl) return;

  // Reset
  btn.className = 'ha-push-btn';
  btn.style.display = '';
  check.classList.remove('show');

  // Start savings from where beat 4 left off
  var startVal = window._haSavingsCurrent ? window._haSavingsCurrent() : Math.round(window.HA_SAVINGS * 0.7);
  savingsEl.textContent = '$' + Math.round(startVal) + '/mo';

  // Press at 600ms
  setTimeout(function() { btn.classList.add('pressed'); }, 600);
  // Release + fade at 900ms
  setTimeout(function() { btn.classList.remove('pressed'); btn.classList.add('done'); }, 900);
  // Checkmark at 1200ms
  setTimeout(function() { check.classList.add('show'); }, 1200);

  // Continue savings counter to final value
  var target = window.HA_SAVINGS;
  var current = startVal;
  var steps = 15;
  var interval = 1800 / steps;
  var increment = (target - current) / steps;
  var counterInterval = setInterval(function() {
    current += increment;
    if (current >= target) { current = target; clearInterval(counterInterval); }
    savingsEl.textContent = '$' + Math.round(current) + '/mo';
  }, interval);
};
```

- [ ] **Step 2: Verify in browser**

Beat 5: "Push to Google" button appears → press → fade → checkmark. Savings counter ticks from ~$510 to $728.

- [ ] **Step 3: Commit**

```bash
git add funnel-start.html
git commit -m "feat: hero animation beat 5 — push to Google"
```

---

### Task 7: Beat 6 — End Card

**Files:**
- Modify: `funnel-start.html`

- [ ] **Step 1: Add Beat 6 HTML, CSS, and JS**

Use the `frontend-design` skill. Fill in `#ha-beat-6`:

**HTML** inside `#ha-beat-6`:
```html
<div class="ha-endcard">
  <div class="ha-endcard-time">
    <span class="ha-endcard-label">Time</span>
    <span class="ha-endcard-value">10 minutes</span>
  </div>
  <div class="ha-endcard-divider"></div>
  <div class="ha-endcard-savings">
    <span class="ha-endcard-label">Savings</span>
    <span class="ha-endcard-value ha-endcard-green" id="ha-endcard-amount">$728/mo</span>
  </div>
</div>
```

**CSS:**
- `.ha-endcard` — `display: flex; align-items: center; justify-content: center; gap: 28px; opacity: 0; transform: scale(0.95); transition: opacity 0.5s ease, transform 0.5s ease;`
- `.ha-endcard.show` — `opacity: 1; transform: scale(1);`
- `.ha-endcard-label` — `display: block; font-size: 11px; color: var(--text-dim); text-transform: uppercase; letter-spacing: 0.08em; font-weight: 600; margin-bottom: 4px;`
- `.ha-endcard-value` — `display: block; font-family: 'DM Serif Display', serif; font-size: 24px; color: var(--text-dark);`
- `.ha-endcard-green` — `color: var(--green);`
- `.ha-endcard-divider` — `width: 1px; height: 44px; background: var(--border);`

**JS** — define `window.ha_startBeat6`:
```js
window.ha_startBeat6 = function() {
  var endcard = document.querySelector('#ha-beat-6 .ha-endcard');
  var amountEl = document.getElementById('ha-endcard-amount');
  if (!endcard) return;
  endcard.classList.remove('show');
  if (amountEl) amountEl.textContent = '$' + window.HA_SAVINGS + '/mo';
  // Pop in after a short delay
  setTimeout(function() { endcard.classList.add('show'); }, 100);
};
```

- [ ] **Step 2: Verify in browser**

Beat 6: Clean card fades in showing "Time: 10 minutes · Savings: $728/mo". Holds for 2 seconds, then loops back to Beat 1 seamlessly.

- [ ] **Step 3: Commit**

```bash
git add funnel-start.html
git commit -m "feat: hero animation beat 6 — end card with time and savings"
```

---

### Task 8: Final polish and loop smoothness

**Files:**
- Modify: `funnel-start.html`

- [ ] **Step 1: Polish the animation**

- **Loop transition:** Ensure the fade from Beat 6 → Beat 1 on loop restart is smooth. The beat sequencer's `opacity` transition on `.ha-beat` (0.4s) should handle this — Beat 6 fades out while Beat 1 fades in. Verify this looks good and adjust timing if needed.
- **Reset states:** Ensure each `ha_startBeatN` function properly resets its elements so the loop looks identical every time. Walk through 2-3 full loops checking for stale state.
- **Savings counter cleanup:** Make sure `clearInterval` is called for savings counters when leaving beats (the beat transition hides the element, but the interval should be cleaned up). Add cleanup in `ha_startBeat5` for beat 4's interval, and in `ha_startBeat6` for beat 5's interval.
- **Mobile sizing:** At `<768px` the animation container should be `max-width: 100%` and maintain its aspect ratio. Verify font sizes inside the animation are readable at small sizes — if not, reduce them slightly for mobile.
- **Reduce motion:** Add `@media (prefers-reduced-motion: reduce)` that disables the animation and shows a static version (just the end card content visible).

- [ ] **Step 2: Full loop verification**

Watch the animation loop 3 full times. Verify:
- Beat transitions are smooth (no flicker, no jump)
- Savings counter goes 0 → ~510 → 728 consistently
- Cards slide in correctly every loop
- No stale state after multiple loops
- Looks good at both desktop and mobile widths

- [ ] **Step 3: Commit and push**

```bash
git add funnel-start.html
git commit -m "feat: hero animation polish — loop smoothness, cleanup, reduced motion"
git push origin main
```
