# Design Decisions

## Next Steps
- All 11 tasks complete
- Funnel page is ready for review

## Decisions

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

### 2026-04-05 — Funnel Shell Architecture
- Single-page app with hash-based routing (8 views)
- Light theme using established color system from pricing-options.html
- Figtree as body font (matches web app), DM Serif Display for headings
- Mobile-first responsive: base mobile, 768px tablet, 1024px desktop
- Frosted-glass sticky header with backdrop-blur
- CSS class conventions: `.funnel-view`, `.funnel-card`, `.funnel-btn-*` prefix pattern
