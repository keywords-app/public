# Hero Animation: Product Hype Reel

## Overview

A short, looping CSS/JS animation embedded inline in the hero section of the ad funnel landing page (`funnel-start.html`). Replaces the current dashed image placeholder. Tells the full Keywords.app story in ~18 seconds using real UI elements, choreographed for impact — a hype reel, not a demo.

## Format

- **Technology:** CSS animations + JS timing, inline HTML element. No video files, no external assets.
- **Duration:** ~18 seconds per loop, autoplaying, infinite loop
- **Audio:** None
- **Container:** Replaces the `.hero-image-placeholder` in `#view-landing`. Responsive — scales down on mobile.
- **Visual style:** Real UI elements from the funnel/app (buttons, cards, color system), stylized and choreographed. Uses the app's green accent, card styles, and button designs. Not a screen recording or walkthrough — each beat is a designed moment.

## Storyboard: 6 Beats

### Beat 1: Connect (~2s)
- "Connect Google Ads" button fades in, styled like the real `.funnel-btn-google` button
- Button gets a press/click animation (scale down slightly, then back up)
- Button transforms into a green checkmark (fade/morph transition)

### Beat 2: Scan (~3s)
- Progress bar appears and fills
- Text below cycles: "Analyzing search terms..." → "Building clusters..."
- Subtle pulsing dot alongside the text (same pattern as the `#scanning` view)

### Beat 3: Clusters Appear (~4s)
- 3-4 cluster cards slide in from the bottom or fade in staggered
- Each card has a label:
  - "plumber near me"
  - "emergency plumber"
  - "24hr plumber"
  - "plumbing school" — this one has a red/warning border or highlight (bad cluster)
- Cards use the app's card styling (white bg, border, rounded) at small scale

### Beat 4: Sort (~4s)
- Two columns appear: green "Keep" on the left, red "Block" on the right
- Good clusters (first three) slide left into the Keep column
- Bad cluster ("plumbing school") slides right into the Block column
- Savings counter appears and starts ticking up from $0
- Counter uses a number-rolling animation, accelerating as it goes

### Beat 5: Push (~3s)
- "Push to Google" button appears, styled like the real app's push button
- Button gets the click animation
- Confirmation: green pulse/checkmark animation on the button
- Savings counter continues ticking, approaching $728

### Beat 6: End Card (~2s)
- Clean centered display: **"Time: 10 minutes · Savings: $728/mo"**
- Uses DM Serif Display for the numbers, green color for $728
- Holds for ~2 seconds
- Fades out and loops seamlessly back to Beat 1

## Savings Counter Behavior

- Appears at the start of Beat 4
- Ticks from $0 to $728 across Beats 4 and 5 (~7 seconds total)
- Uses an easing curve — starts slow, accelerates in the middle, decelerates to land precisely on $728
- Format: "$XXX/mo" with the dollar amount rolling/counting up
- Lands on $728 as Beat 6 begins

## Variant-Ready Design

The animation is designed so these elements are easy to swap for A/B testing or audience-targeted ad variants:
- **Cluster labels:** Change industry (dental, pest control, HVAC, etc.)
- **Savings number:** Adjust to match the industry or account size being targeted
- **Bad cluster label:** Change to match the industry (e.g., "dental school" for dentists)

These values should be defined as JS constants at the top of the animation code, not buried in the HTML.

## Responsive Behavior

- **Desktop (1024px+):** Full size, sits to the right of the hero text
- **Tablet (768px):** Slightly smaller, still beside or below the text
- **Mobile (<768px):** Below the hero text, full width of the container, slightly reduced scale

## Technical Notes

- All animation timing via CSS `@keyframes` and `animation-delay` where possible, JS `setTimeout`/`setInterval` only for sequencing beats
- The animation should pause when not visible (IntersectionObserver) to avoid wasting CPU
- No external dependencies — pure CSS/JS
- Container should have a fixed aspect ratio to prevent layout shift

## File

Modify: `funnel-start.html` — replace `.hero-image-placeholder` in `#view-landing` with the animation element and add CSS/JS.
