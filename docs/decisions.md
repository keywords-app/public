# Design Decisions

## Next Steps
- Produce remaining image assets for placeholders in funnel-start.html
- Web app team: integrate funnel designs into app framework at /start route
- Web app team: wire up actual auth, Google Ads OAuth, Stripe, and scan flows

## Decisions

### 2026-04-06 — Hero Animation Density & Discovery Waterfall
- Reduced `.ha-beat` padding from 24px to 16px for tighter framing
- Widened content areas: scan/clusters to 92%, sort to 94%
- Replaced Beat 2 plain progress bar with discovery-style data waterfall cycling through Ad Copy, Keywords, Landing Pages, and Geo Targeting categories as animated pill tags
- Added "Acme Plumbing Co." account name reveal after Beat 1 checkmark
- Added two more cluster cards to Beat 3 for fuller layout
- Updated reduced-motion media query for new elements

### 2026-04-05 — Hero Animation Beats 4-6 + Polish
- Beat 4 (Sort): Keywords sort into Keep/Block columns with staggered card animations; savings counter counts up to 70% of target ($728) over 3s
- Beat 5 (Push): Green "Push to Google" button with press/done animation, reuses checkmark from Beat 1; savings counter continues from Beat 4 value to full target
- Beat 6 (End Card): Two-stat summary card ("10 minutes" / "$728/mo") fades in with scale transition
- Polish: Added `prefers-reduced-motion` media query disabling all transitions/animations; 360px breakpoint reduces end card font size; savings interval cleared on animation stop to prevent stale counters across loops

### 2026-04-05 — Hero Animation Beats 1-3
- Beat 1 (Connect): Google-styled button with press animation, then fade to green checkmark
- Beat 2 (AI Scan): Progress bar fills to 85% over 2.5s, phase text swaps from "Analyzing search terms..." to "Building clusters..."
- Beat 3 (Clusters): Four cards slide in with staggered delays (0/0.15/0.3/0.45s); last card styled red as a "bad" cluster (plumbing school)
- All beat start functions are global (`window.ha_startBeatN`) called by the existing sequencer
- CSS added at end of `<style>` block; JS added before the IIFE closing in `<script>` block

### 2026-04-05 — View Transitions, Scroll Animations, and Polish
- View fade transitions already implemented (opacity 0→1, 0.3s ease) via `.funnel-view` / `.fade-in` classes
- Scroll-to-top already fires on view change via `window.scrollTo(0, 0)` in `showView()`
- Added fade-up-on-scroll animations for landing page sections (social proof, how-it-works, pricing preview, final CTA) using IntersectionObserver with 15% threshold
- Added `:focus-visible` outlines (2px green, 2px offset) on all interactive elements: buttons, links, account rows, scan dots
- Fixed duplicate `margin-top` on `.progress-step-connector`
- Scanning slideshow JS verified: intervals start on view entry, stop on exit, dot clicks reset interval

### 2026-04-05 — Scan Complete View (#view-ready)
- Success indicator: CSS-only green circle (80px) with ::after pseudo-element checkmark, no images
- Numbers in teaser ("2,847 search terms", "143 clusters") use `var(--green)` via `<strong>` inside `.ready-teaser`
- Desktop CTA is an `<a>` tag with `.funnel-btn-primary` enlarged to 17px / 16px padding — feels more prominent than other views
- Mobile variants shown below a text separator for web app team reference; iOS uses dark rounded "App Store" style button with  icon, Android shows text-only message
- All contained in `.ready-wrapper` max-width 560px centered — slightly wider than standard `.funnel-card` (440px) to accommodate two-column mobile cards if needed later

### 2026-04-05 — Landing Page Pricing Must Match Funnel Pricing
- Landing page pricing preview now shows launch discount banner, strikethrough prices (~~$47~~ $27 Solo, ~~$77~~ $47 Pro), and 30-day MBG — same treatment as the #pricing offer view
- Both the landing page and the in-funnel pricing view should always show consistent pricing/discount framing

### 2026-04-05 — Funnel Shell Architecture
- Single-page app with hash-based routing (8 views)
- Light theme using established color system from pricing-options.html
- Figtree as body font (matches web app), DM Serif Display for headings
- Mobile-first responsive: base mobile, 768px tablet, 1024px desktop
- Frosted-glass sticky header with backdrop-blur
- CSS class conventions: `.funnel-view`, `.funnel-card`, `.funnel-btn-*` prefix pattern
