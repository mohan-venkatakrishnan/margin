# Margin

A Manifest V3 Chrome extension that opens in the browser **side panel** and shows a
swipeable, InShorts-style stack of short news cards. Pick your topics on first run;
each card shows an image, a title, a ~60-word summary, and the source. Tapping a card
opens the original article in a new tab.

- **Fully client-side** — no backend. Feeds are fetched and summarized on the user's
  machine.
- **On-device summaries** via Chrome's built-in **Summarizer API** (Gemini Nano).
  Mode is set in Settings → AI summaries: **Auto** (default — used automatically
  when the device already supports the model, never triggers a download, so
  ineligible hardware stays quiet with no provisioning errors), **On** (also
  downloads the model if supported), or **Off**. A silent feed-snippet fallback
  is shown whenever a summary isn't available.
- **Free** = 3 topics + light theme. **Plus** = unlimited topics + all themes +
  custom colors + **live feed** (always-latest on open) + **Insights** (a reading
  analytics dashboard). Free users can open Insights but see a blurred sample
  preview behind an upgrade CTA.
- **25 curated, location-neutral topics** (World, Politics, Business, Money, Tech,
  AI, Science, Space, Environment, Health, Education, Sports, Football, Formula 1,
  Gaming, Entertainment, TV, Film, Music, Books, Art & Design, Fashion, Food,
  Lifestyle, Travel) backed by **~100 publisher feeds** (BBC, The Guardian, NYT,
  Wired, and more) — all verified live by the regression gate.
- **Images** come from the feed (`media:*`, `enclosure`, inline `<img>` in
  RSS `content:encoded` / Atom `<content>`), falling back to the article's
  `og:image` resolved client-side, then a topic placeholder — never a broken image.

## Develop

```bash
npm install
npm run dev        # Vite dev server with HMR (load dist as unpacked, see below)
npm run build      # typecheck + production build into dist/
npm test           # unit tests (offline)
npm run test:net   # live feed integration test (hits the network)
npm run regression # SANITY GATE: every topic + 50 sampled articles load content & images
npm run validate:feeds   # confirm every configured feed returns live XML
npm run probe:feeds      # probe a wider candidate list (for adding feeds)
npm run package          # build + zip dist/ into margin-<version>.zip for the Web Store
npm run icons      # regenerate brand PNG icons
```

### Sanity gate (`npm run regression`)

Run this before shipping a feed/parser change or a Web Store release. It fetches
every default topic, samples **10 articles per topic**, and asserts:

- every topic returns a full sample of 10 cards,
- ≥95% of sampled cards have readable body text,
- each topic clears a **70% image rate** and the overall image rate is ≥85% —
  counting the feed image **or** the `og:image` fetched from the article
  (the same fallback the extension uses).

It is network-gated (only runs with `MARGIN_NET=1`, which the script sets) so it
never destabilises the offline unit suite. It exists because feeds drift: e.g.
TechCrunch dropped inline images from its RSS, so images now come from the
`og:image` fallback — exactly the kind of regression this gate catches.

## Load the extension in Chrome

1. `npm run build`
2. Go to `chrome://extensions`, enable **Developer mode**.
3. **Load unpacked** → select the `dist/` folder.
4. Click the Margin toolbar icon to open the side panel.

The on-device Summarizer requires desktop Chrome with hardware support (~22 GB free
disk; GPU ≥4 GB VRAM or ≥16 GB RAM; a one-time ~4 GB model download). Where it's
unavailable or still downloading, Margin shows the cleaned feed snippet instead — never
a blank card.

## Architecture

```
manifest.config.ts      # MV3 manifest (consumed by @crxjs/vite-plugin)
vite.config.ts          # build (crxjs + preact)
vitest.config.ts        # tests (preact only; faithful XML parser via tests/setup.ts)
src/
  background.ts         # opens side panel on action click; refresh alarm; ExtensionPay init
  sidepanel/
    App.tsx             # prefs load, theme application, view routing
    Onboarding.tsx      # topic picker + free 3-topic cap
    Feed.tsx            # scroll-snap card stack, keyboard nav, topic filter
    Card.tsx            # one card: image, title, summary, source, tap-to-source
    Settings.tsx        # topics, theme picker (gated), subscription
    Analytics.tsx       # Insights dashboard (Plus; blurred sample for free)
    charts.tsx          # dependency-free SVG/flex charts
    topicArt.tsx        # per-category fallback art for missing images
    useFeed.ts          # load cache, refresh-when-stale, prefetch summaries + images
    icons.tsx           # inline stroke icons
  lib/
    types.ts            # canonical shapes + tier constants
    store.ts            # chrome.storage wrappers + gating helpers
    topics.ts           # curated topic resolution + feed-map builder
    feeds.ts            # fetch + parse RSS/Atom -> NewsCard[]
    images.ts           # image extraction chain + og:image fallback
    dedupe.ts           # dedupe / sort / retention / pool merge
    summarize.ts        # Summarizer API wrapper + prefetch + fallback
    payments.ts         # ExtensionPay abstraction (dev fallback when unconfigured)
    analytics.ts        # local reading stats (storage.local) + chart derivation
    text.ts             # pure text helpers
  config/
    topics.json         # built-in topics (location-neutral defaults)
    feeds.json          # topic -> feed URL mapping
  styles/theme.css      # design system (CSS variables per theme)
```

## Going live with payments

Margin uses [ExtensionPay](https://extensionpay.com) for licensing. Until configured it
runs a **dev fallback**: the upgrade button grants Plus locally so all gating is
testable. To enable real Stripe checkout:

1. Register the extension at extensionpay.com.
2. Set `EXTPAY_ID` in `src/lib/payments.ts`.
3. `startBackgroundPayments()` (already called in `background.ts`) wires the rest.

## Legal

Margin only ever shows a short summary/snippet and **always** links out to the
publisher. It never mirrors or iframes full article content. The feed list in
`src/config/feeds.json` is intentionally easy to edit so any publisher can be dropped
quickly. Review each major source's RSS terms before a paid launch.
