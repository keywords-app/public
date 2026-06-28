# Design Decisions

## Next Steps
- **Public site is launch-ready.** Web-first home (#3), reconciled doc pages (#5/#6),
  and home referral capture (#7) are all merged and live; referral cross-surface
  flow verified on prod 2026-06-27. No open work blocking on this repo.
- **Confirm-before-publishing: ALL CLEARED for launch (2026-06-27).** Beta stats
  (10%+ / <15 min / 125+) confirmed current · "we never touch budgets" confirmed
  accurate vs OAuth scopes · Aug 1 Microsoft/Android dates confirmed · legal/privacy
  wording reviewed & cleared · Google Partner badge: **not adding at this time**
  (home keeps the empty `.partner-slot`, renders nothing — fill only if that changes).
- Remaining epic work is on other surfaces (not this repo): `/start` funnel re-skin
  (keywords-shared#13 Step 4, web-app) · launch-day polish/coordination (Step 5).
- Web app team: integrate funnel designs into app framework at /start route
- Web app team: wire up actual auth, Google Ads OAuth, Stripe, and scan flows

## Decisions

### 2026-06-28 — Mobile pricing/login discoverability on the home
- Mobile (<880px) hides the Pricing + Log in nav links, so the only header control
  was Get Started Free and pricing was a long scroll away. Two additions:
  - **"See pricing ↓"** link under the hero CTA (`.hero-pricing-link`, all viewports,
    smooth-scrolls to `#pricing`) — pricing affordance at the decision point.
  - **Mobile header CTA relabels on scroll:** at the top it reads **"Sign Up / Log In"**
    (the hero already shows the big Get Started button), then switches to
    **"Get Started Free"** once the hero CTA scrolls out of view (IntersectionObserver
    on `.hero .cta`, gated to `matchMedia('(max-width:880px)')`). Both states keep the
    same `app.keywords.app/start/signup` destination — the signup page (web-app) is
    being updated to make login easy from there. HTML default stays "Get Started Free"
    (no-JS safe). Scroll-flip verified by reasoning (proven IO pattern) + at-rest states
    in headless; needs a real-device eyeball.
- Still open (other surface): pricing/reassurance strip on the web-app signup page.

### 2026-06-27 — Home referral capture, ?refer= (public#7, epic keywords-shared#14)
- Added a tiny dependency-free inline script to `index.html` that captures
  `keywords.app/?refer=<code>`: validates `^[A-Za-z0-9]{1,32}$`, sets a shared
  cookie `referral_code=<code>; Domain=.keywords.app; Path=/; Max-Age=2592000;
  SameSite=Lax; Secure`, then — if an active code exists (URL param **or** existing
  cookie) — appends `?refer=<code>` to every `start/signup` CTA (cookie-blocked
  fallback) and shows the hero acknowledgment banner. Finally scrubs `?refer` from
  the URL via `history.replaceState`.
- Regex + cookie write string are **byte-identical to web-app#85** (`src/lib/
  referral.ts` `isValidReferralCode` / `buildReferralCookie`) so both surfaces agree.
  Cookie parse mirrors web-app's `parseReferralCookie` (split `;`, exact name match,
  `decodeURIComponent`, re-validate).
- Banner: `.refer-banner` styled to home tokens (tint bg / accent-dark text — green/
  red stay reserved to product UI), `hidden` by default, shown only on active code.
  Shown in two places: under the hero CTA and (added 2026-06-27) under the pricing
  card's Get Started button (`.refer-banner--card`, full-width to match the button).
  The script reveals **all** `.refer-banner` elements (querySelectorAll), not one by id.
- Local test (file://, so the Secure/Domain cookie can't persist) confirmed the
  URL-param path: valid code → banner + 4 CTAs carry the code + URL cleaned; invalid
  (`bad code`) → no-op (no banner/CTA) but URL still cleaned; no param → nothing.
  Sequenced after the web-app cookie-read. **Prod cross-surface flow (cookie set →
  signup on app.keywords.app reads it → cleared on success) tested & working 2026-06-27.**

### 2026-06-27 — Reconcile doc pages + fix data-accuracy copy (public#5)
- Restyled `help/support/privacy/terms.html` to the new web-first look via a shared
  **`doc.css`** (tokens, Inter, frosted sticky header with single Get Started Free
  CTA, matching footer, document typography, tables, badges, TOC, cards). Each page
  drops its old inline `<style>` and links `doc.css`; index.html keeps its own inline
  styles (home is self-contained). Header/footer markup normalized across all four.
- **Corrected the data-storage copy** to the real model (per the user): Google Ads
  data stays **on the device until the user syncs (slots) an account**; once synced,
  search terms + triage decisions + AI clusters are stored server-side to sync across
  devices and the web app. Removed the absolute "never on our servers / device-only"
  claim from privacy.html (§1 summary, §2.3, §5 retention), terms.html (§5), and
  support.html (FAQ "Is my data safe?"). help.html already described slotting/sync
  correctly — left as-is. Removing a synced account preserves server data for re-sync;
  deletion happens on account delete / deletion request.
- Legal wording (privacy/terms) was updated for accuracy. **Reviewed and cleared
  for launch by the user on 2026-06-27.**

### 2026-06-26 — Web-first marketing home (public#3, epic Step 2)
- Rebuilt `index.html` from the iOS-only page into the full 9-section web-first home
  by extending the locked Claude Design draft: sticky nav · hero · stat strip ·
  how-it-works (4 steps) · 6 alternating feature sections · "Also on iPhone" ·
  pricing · trust/privacy · final CTA · footer. Tokens used verbatim (Inter, accent
  `#0A66D6`, neutrals, 10/16px radii, two shadows).
- Single CTA everywhere → `https://app.keywords.app/start/signup` ("Get Started Free").
  Nav reconciled to real destinations only: Pricing (anchor) · Log in
  (`app.keywords.app/start`) · Get Started Free. Dropped the draft's invented
  Product/Customers/Resources links.
- Wired the 9 release-build screenshots as real `<img>` (local relative paths) into
  the browser-frame / phone-frame slots: hero `web-hero-noclusters`; features
  `web-clustering` · `web-negatives` · `web-push` · `web-savings` · `web-accounts`;
  cross-platform uses a logo/"Aug 1" pill lockup (no screenshot, per shot list); iOS
  `ios-search-terms` · `ios-performance` · `ios-negatives` in phone frames.
- App Store badge ships in the LIVE state with a documented launch-day FALLBACK
  (hidden `.appstore-fallback` line; toggle `[hidden]` if Apple review lags).
- **Privacy claim corrected:** dropped the old "data stays on your device / never on
  our servers" line (false for the web app). Trust section is web-accurate — limited
  OAuth scopes, never touches budgets, encrypted in transit and at rest.
- Carried over GTM (`GTM-MMPGCHP8`) + meta/OG; title/description updated to web-first.
- Kept `img-src/` PNG masters versioned (committed both webp + masters, per the user's
  correction — not gitignored).
- Retired stale variants: `index2.html`, `index3.html`, `keywords-landing-v4.html`,
  `funnel-start.html`. (`pricing-options.html` left as-is — out of scope.)
- **Tooling note:** headless Chrome (`--window-size=390`) imposes a fixed ~485px
  layout floor on *any* page (old `index.html` and the approved draft measure it too),
  so its mobile screenshots/clientWidth are misleading. The real responsiveness signal
  is `scrollWidth === clientWidth` (no horizontal overflow), which held at all widths.
  Verify mobile on a real device, not headless screenshots.

### 2026-06-21 — help.html rewritten for slots + referral model
- Replaced the subscription model (Free / Solo $27/mo / Pro $47/mo) with the unified **slots model**: free, 1 included account slot, earn more via referral, buy extra slots at **$10 one-time**. No platform split — web and iOS share the same go-to-market (verified against `keywords-shared/docs/contracts/slot-model-v1.md` + live `web-app`).
- Dropped the Solo/Pro plan table, "launch pricing," and all `$27` / `/mo` strings. Removed the "Settings → Subscription" framing → **Settings → Account Slots / Add Extra Slot — $10**. Dropped "upgrade" as an out-of-slots path (now: buy a slot or refer a friend).
- Added a **Refer a Friend** section (TOC + body): sign up via a friend's link → start with 2 slots; on connecting Google, both earn a slot; links are always web links (`app.keywords.app/r/<code>`). Per `referral-v1.md` the bonus fires on the referee's first Google login (OAuth connection), not on slotting a specific Ads account.
- Retired all free-scan language (AI Scan now follows the slot/account model). Web app reworded from "for paid subscribers" → **free for all signed-in users**.
- Slotting is an **explicit, opt-in action** — the first account is NOT slotted automatically (corrected per user). Removing a synced account returns its slot; triage data is preserved server-side and restored on re-sync.
- Kept "up to 1,000 terms per scan/fetch" — verified real: web-app `buildSearchTermsQuery` uses `LIMIT 1000 ORDER BY impressions DESC`; AI scan processes up to 1,000/run. Reframed away from being a free-tier-only cap.
- Documented (§6) **guided triage** and **predictive scoring** (Keep/Kill pills). Confirmed by the user 2026-06-21: both are **live on iOS**, so the iOS-framed copy stands.

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
