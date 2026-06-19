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
