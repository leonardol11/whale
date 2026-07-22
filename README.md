# 🐋 Polymarket Whale Watch

A single-file, zero-install website that shows **large Polymarket trades in real time** — the idea
being to spot smart money and bet *with* the whale, on the theory they know something.

It polls Polymarket's **public Data API** (`data-api.polymarket.com/trades`) directly from your
browser. No account, no API key, no backend, no build step.

## Run it

Just open the file:

```bash
open index.html          # macOS — double-clicking works too
```

Or serve it (identical result, avoids any `file://` quirks):

```bash
python3 -m http.server 8777
# then visit http://localhost:8777
```

To put it online, drop `index.html` on any static host (GitHub Pages, Netlify, Vercel, Cloudflare
Pages). It's one static file — nothing to configure.

## What it shows

- **Live feed** of trades whose USD notional (`size × price`) is above your threshold, newest on top,
  new trades flash in. Auto-refreshes every 8–60s (your choice).
- Per trade: **BUY/SELL**, market + outcome, **entry price / implied odds**, USD size, the trader
  (links to their Polymarket profile), and a **Trade ↗** button straight to the market so you can
  follow the bet.
- **(avg $X, low $Y, high $Z)** next to each trader — their historical bet-size stats (average,
  smallest, largest), computed from that wallet's recent trades (Data API `?user=`, fetched once per
  wallet and cached). A trade near a trader's *high* — or far above their *average* — is a conviction
  outlier; hover the value to see the sample size.
- **🎯 LONGSHOT** flags big buys at long odds (≤20¢) — high-conviction / possible-edge bets.
  **🔒 FAVORITE** flags buys near certainty (≥90¢). Trades ≥ $250k get a gold glow.
- **Top wallets** leaderboard: who's moving the most size in the recent buffer, with net buy/sell.

## Controls

- **Minimum trade size** — preset buttons ($10k / $25k / $50k / $100k / $250k / $500k) plus a
  **custom** field for any exact value; default $50k. Changing it re-queries the API server-side.
- **Maximum (optional)** — cap the upper end to see a size band (e.g. only $40k–$100k trades).
- **Implied odds** — Any / Under 50% / Over 50% buttons. A trade's price
  *is* its implied probability, so **Under 50% + Buys** surfaces the contrarian/longshot bets. Filters
  compose with everything else.
- **Side** — All / Buys / Sells. (Note: `SELL @ 100¢` is usually someone cashing out a resolved
  position, not a fresh directional bet — filter to **Buys** to see new convictions.)
- **Market** — ✓ Active / ✓ Resolved checkmarks. Each trade's market is looked up on the Polymarket
  **Gamma API** (`closed` flag): uncheck **Resolved** to hide settled markets (the `SELL @ 100¢`
  redemptions), leaving only live, still-tradeable markets. Resolved rows carry an amber "Resolved" tag.
- **Trader → Near their high** — checkmark filter that keeps only trades at (or within 25% below) the
  trader's recent high bet size — i.e. their biggest bets, where they're going as big as they ever do.
  Needs the wallet's stats loaded (a few seconds), so rows fill in as those resolve.
- **Refresh** — poll interval. Click the **LIVE** pill to pause/resume.
- **Theme** — Light / Dark toggle in the header; your choice persists across visits (defaults to dark).

## Notes & caveats

- The feed covers **every market** — the only server-side filter is trade size (CASH notional).
  There's no market or category restriction, so all of Polymarket's large trades flow through.
- USD value = `size × price`. Size is share count; price is the 0–1 probability (also the cost in
  dollars per share and the implied odds).
- The API is CDN-cached ~5 min; each poll appends a cache-buster so you get fresh data.
- Categories are keyword/tag heuristics that now only set the **label** shown on each trade (they
  no longer filter the feed); tweak the `CAT_RULES` / `RE` regexes in `index.html` to relabel.
- **Not financial advice.** "Following the whale" assumes they know something. Sometimes they're just
  hedging, market-making, or wrong.
