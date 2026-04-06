# Ad Funnel Design: Facebook Ads → Free Sign-Up → Paid Conversion

## Overview

A straight-line funnel for acquiring users via Facebook ads. Traffic lands on a landing page integrated into the web app at a single URL (`/start`). The user signs up free, connects Google Ads, sees a pricing offer, and experiences the "wow moment" AI scan — all without leaving the page. Hash-based routing (`#signup`, `#connect`, etc.) gives each step a trackable URL with browser back-button support, while keeping the experience seamless.

**Target audience:** Google Ads managers spending 2+ hours/week on hands-on account work.

**Ad angle:** "Managing Search Terms Has Never Been Easier"

**Goal:** Acquire customers at net-zero ad spend (same-day recovery). Test until CAC is stable, then scale.

---

## Pricing Context

This funnel uses **launch pricing** (April discount — "lock in these rates for life"):

| | Free | Solo | Pro |
|---|---|---|---|
| Price | $0 | $27/mo | $47/mo |
| Regular price (public site) | $0 | $47/mo | $77/mo |

The funnel pricing cards frame these as "Launch Discount — April only. Lock in these rates for life. 30-day money-back guarantee."

Feature reference: `docs/keywords-app-pricing-spec.md`

Key corrections from the pricing spec's search term pull numbers:
- **Solo:** 50k/week automated (not 100k)
- **Pro:** 250k/week automated (not 100k)

---

## Architecture

**Single URL, hash-based routing.** All funnel views live at one URL (`/start`). JavaScript swaps views with transitions. Each step gets a hash fragment for analytics tracking and browser back-button support.

**Reusable funnel component.** The funnel logic (steps 3–8) must work both at `/start` and when triggered from the Accounts screen in the web app. If a user skips connecting Google Ads during the funnel but connects later from the app, the remaining funnel steps (pricing → select → discovery → scan → handoff) should pick up automatically since it's their first Google Ads connection.

**Light theme.** Consistent with the user's preference for readability.

**Responsive.** Must work well on mobile and desktop. Reference: cascader.io for responsive design patterns.

**Event tracking.** Facebook pixel events and PostHog events fire at each step transition. Specific event names and parameters are an implementation concern, not specified here.

---

## Funnel Views

### View 1: Landing Page (`/start`)

The entry point from Facebook ads. No nav bar — this is a funnel page, not a website. Logo top-left, subtle "Log in" link top-right for existing users.

**Hero section:**
- Headline: "Managing Search Terms Has Never Been Easier"
- Subhead: focused on the 10-minute, end-to-end promise
- CTA: "Get Started Free"
- No credit card required messaging

**Social proof strip:**
- 10%+ average spend saved in month 1
- <15 minutes per account, entire first month
- 125+ beta users

**How it works (the core story):**
1. Connect your Google Ads account → AI fetches and groups your search terms into clusters by meaning and intent
2. Tag good and bad search terms and clusters → negative keyword suggestions (live now) make it even faster
3. Negative Match Maker blocks the bad ones, pushes negatives to Google, locking in your savings for life
4. Estimated savings (typically >10%) shows live in the app

End-to-end: ~10 minutes.

**Pricing preview:**
- Launch discount banner: "Launch Discount — April only. Lock in these rates for life."
- 3-tier cards: Free / Solo ~~$47~~ $27/mo / Pro ~~$77~~ $47/mo (strikethrough regular prices)
- All CTAs say "Get Started Free" — not selling here, just showing the value ladder
- Feature lists per the pricing spec
- 30-day money-back guarantee below the cards
- Pricing framing matches the in-funnel pricing offer view (#pricing) for consistency

**Final CTA:** Repeat "Get Started Free"

**Image placeholders needed:**
- Hero: animated loop (MP4 or GIF) showing the full workflow — connect Google Ads → AI scan → clusters sorting into keep/block → savings counter ticking up → "Push to Google" → end card: "Time: 10 minutes. Savings: $728/mo"
- Phone mockup for mobile app
- Before/after: messy search terms vs organized clusters

All CTA buttons trigger transition to `#signup`.

---

### View 2: Sign Up (`#signup`)

Centered card layout. Matches existing web app auth UI style but reordered for the funnel.

- Headline: "Create your free account"
- Subhead: "No credit card required. Get your first scan in under 10 minutes."
- Auth options in order:
  1. Continue with Google (primary — largest button)
  2. Continue with Apple
  3. Email + password (form fields below)
- "Already have an account? Log in" link at bottom
- On successful sign-up → auto-advance to `#connect`

---

### View 3: Connect Google Ads (`#connect`)

Centered card layout.

- Headline: "Connect your Google Ads account"
- Subhead: "Keywords.app needs read access to your search terms and campaigns. We never modify your ads or spend."
- "Connect Google Ads" button with Google icon → triggers separate OAuth flow (not the same as sign-in Google OAuth — different scopes, likely different Google account)
- Trust signals below button: "Read-only access · Your data stays on your device · Disconnect anytime"
- "Skip for now" link at bottom → drops user to the Accounts screen in the web app (logged in). The funnel resumes when they connect Google Ads from the app for the first time.
- On successful OAuth → account list loads in background → auto-advance to `#pricing`

---

### View 4: Pricing Offer (`#pricing`)

Shown after Google Ads connects and we've loaded their account list (so we know how many accounts they have access to).

- **Banner/header:** "Launch Discount — April only. Lock in these rates for life."
- **3 cards:** Free / Solo $27/mo / Pro $47/mo
- **Dynamic emphasis:** If user has access to 1 Ads account, Solo card is highlighted ("Most popular" or similar). If multiple accounts, Pro card is highlighted. Both prices and a "Stay free" option always visible regardless.
- **Feature lists** from pricing spec. Key features:
  - Free: 1 account, 1,000 on-demand search terms, search term clusters, negative keyword suggestions, iOS app, Android (May 1)
  - Solo: 1 synced+AI account, 50k/wk automated, search term clusters ✨, negative keyword suggestions ✨, weekly scans, sync, push notifications, iOS, Android (May 1)
  - Pro: 5 synced+AI accounts, 250k/wk automated, search term clusters ✨, negative keyword suggestions ✨, weekly scans, sync, push notifications, iOS, Android (May 1)
  - ✨ = AI sparkle indicator on Solo/Pro for clustering and negative keyword features
  - Extra accounts: $27 one-time each (Solo and Pro)
- **"30-day money-back guarantee"** below the cards
- **CTAs:**
  - Free: "Stay free" → advances to `#select`
  - Solo: "Start Solo" → Stripe modal → on success, advance to `#select`
  - Pro: "Start Pro" → Stripe modal → on success, advance to `#select`
- **Stripe payment** handled via in-app modal (no redirect to stripe.com)

---

### View 5: Select Account (`#select`)

User picks which Ads account to scan first.

- Headline: "Which account should we scan first?"
- List of all Ads accounts from all connected Google logins (account name + customer ID)
- Always show the list, even if only 1 account — user must actively select
- "Connect another Google account" option at the bottom of the list (adds another Google OAuth login, then refreshes the account list)
- Pick one → advance to `#discovery`
- "Skip for now" link → drops to Accounts screen in the web app

---

### View 6: Discovery Confirmation (`#discovery`)

After selecting an account, the app runs discovery (~12 seconds): reads ads, keywords, landing pages, geographic targets.

- **Loading state:** Existing web app modal showing progress (reading ads, landing pages, keywords, checking geographic targeting). The web app team adapts the existing discovery modal to fit the funnel's visual frame. We design the container; the modal internals are adapted from existing code.
- **Confirmation view:** Shows what we found — business name, geographic targets, campaign count, keyword count, landing pages detected. User confirms or corrects.
- "Looks good — start the scan" button → advances to `#scanning`

---

### View 7: AI Scan (`#scanning`)

~3 minute wait. User is captive and excited.

- **Progress indicator:** Phased animation (not a percentage). Stages:
  - "Analyzing search terms..."
  - "Building clusters..."
  - "Scoring relevance..."
  - "Preparing your results..."
- **Auto-advancing slideshow** behind/below the progress indicator. Not swipeable — just auto-rotates. Slides:
  1. How clustering works and why it saves time
  2. What the savings tracker does
  3. How negative keyword suggestions work
  4. The mobile app — review terms from your phone
  5. Social proof: 10%+ savings in month 1
  6. Need help? Email and in-app support available
- On scan complete → auto-advance to `#ready`

---

### View 8: Scan Complete → Handoff (`#ready`)

The wow moment has been prepared. Time to hand off.

- Headline: "Your scan is ready!"
- Preview teaser: "We found X search terms across Y clusters"
- **Desktop:** "Start reviewing" button → drops into triage screen in the web app
- **Mobile (iOS detected):** "Download the app for the best experience" (App Store link) + "or continue in browser" secondary link. User will need to log in again in the iOS app (2-tap Google/Apple login).
- **Mobile (Android detected):** "For the best experience, open Keywords.app on a desktop browser" + "continue in browser" secondary link. (Android app coming May 1.)

---

## Supporting Systems (Bare-Bones Outline)

### Email Recapture

Trigger: user signed up but dropped off before scan complete.

3 emails over 7 days:
1. **1 hour after drop-off:** "Your free scan is waiting" — deep link back to where they left off
2. **24 hours:** Value reminder — the 10-minute, 10%+ savings pitch
3. **7 days:** Last nudge, testimonial or case study

### Nurture / Educate (post-scan, all users)

Weekly emails with tips, each tied to an in-app exercise:
- How to read clusters effectively
- When to block vs keep search terms
- Using savings data with clients
- Mobile app tips and tricks

### Post-Sale (paid users)

- Onboarding email: what to do next, how to add more accounts
- Agency users: how to pitch savings tracking to clients
- Solo users: how to get more clients using Keywords.app as a differentiator

### Retargeting

Facebook custom audiences segmented by drop-off point:
- Signed up but didn't connect Google Ads
- Connected but didn't complete scan
- Scanned but didn't purchase

Ad creative matches where they dropped off in the funnel.

---

## Out of Scope

- LTD (lifetime deal) offer — this is an in-app persistent banner + modal, not part of this funnel's design
- Email sequence copywriting — outlined above but detailed copy is separate work
- Retargeting ad creative — outlined but creative is separate work
- Triage screen design — the user lands in the existing triage UI after handoff
- Specific Facebook pixel event names and PostHog event schemas — implementation concern
