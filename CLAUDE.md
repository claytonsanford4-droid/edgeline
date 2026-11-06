# Edgeline — sports betting research site

Edgeline is Clayton's personal sports-betting research and tracking site. It is a static site:
`index.html` renders everything from **`data/edgeline.json`**. Your job is to keep that data file
accurate and current. Never fabricate odds, scores, injuries or results — if you can't verify
something, leave it out or mark it as missing.

## Data file

`data/edgeline.json` = `{ "updatedAt": ISO time, "collections": { name: { docId: doc } } }`.
Always set top-level `updatedAt` to now when you change anything, and give every doc you
create or edit its own `updatedAt` (ISO). The page prefers whichever copy (file vs. the user's
browser edits) has the newer timestamp.

Collections (look at existing docs for exact shapes before writing new ones):

| collection | doc id pattern | what it is |
|---|---|---|
| `games` | `nhl-2026-11-05-vgk-det` (sport-date-away-home) | schedule + status (`scheduled`/`live`/`final`), `start` (UTC ISO), team names/nicknames/abbrs, `score` {away,home}, pregame `wp` {away,home} |
| `markets` | `m-…` moneyline/spread/total, `p-…` player props, `c-…` NCAAF, `b-…` NCAAB | the prediction-engine input: `sides[]` (pick, books[{book,odds}], optional `boardId`), `sources[]` (name, p, side, sd, verified), `adjustments[]` (side, delta, sd, label, evidence, priced), optional `h2h`, `warnings[]`, `news[]`, `explain`, `capturedAt`; props add `player`, `team`, `opp`, `line`, `stats` |
| `board` | `2026-11-05-k1` | ratings shown in Edge Finder: `verdict` card/lean/trigger/pass, `pick`, `game`, `odds`, `books`, `p`, `data[]`, `signals[]`, `note` |
| `review` | `2026-11-05` | the daily two-pass review: `headline`, `counts` {candidates, ontario, pass1, pass2}, `rule1`, `rule2`, `final[]`, `cuts[]` |
| `intel` | `2026-11-05` | news feed: `headline`, `items[]` {tag, title, detail, impact} |
| `bets` | Claude: `YYYY-MM-DD-NN`; user: `me-…` | the ledger: `owner` (`claude`/`me`), date, sport, type (single/parlay/flyer), pick, game, odds (American int), units, result (pending/win/loss/push), optional `p`, `close`, `legs[]`, `signals[]`, `why`, `final` |
| `meta` | `settings`, `playbook` | bankroll/unit settings and the playbook rules |

## The engine (what the site computes from `markets`)

1. Convert each book's odds to implied probability; where a book prices both sides, remove its margin proportionally (no-vig), then average across books.
2. Blend independent probabilities weighted by 1/σ²: single external model σ=5, multi-book no-vig consensus σ=3, single book σ=4. Use σ 6–9 for crude projections (e.g. score projections converted with σ=11 for college hoops margins, σ=14 for college football, σ=16 for basketball totals).
3. Add documented adjustments (rest, travel, lineups, goalies, matchup history), each with its own σ. Don't double-count something the source model already includes.
4. 80% interval = mean ± 1.28σ. EV = p × net profit − (1 − p) × stake.
5. A bet is only recommended if EV stays positive at the low end of the interval. Rank by EV at the low end × liquidity (ML 1.0, totals/spreads 0.9, props 0.6, exotics 0.4).

## Hard rules

- **Ontario only.** Bettable prices must come from Ontario-licensed books: FanDuel, DraftKings, BetMGM, bet365, theScore Bet, Caesars, PointsBet, BetRivers, Betway, 888sport, Sports Interaction, BET99, BetVictor, TonyBet, NorthStar Bets, PROLINE+, Hard Rock Bet, Betano, LeoVegas, Rivalry, Pinnacle. Offshore books (Bovada, SportsBetting, 1xBet, 22Bet), Novig, Polymarket and Kalshi may only appear as information, never as the price.
- **Playbook:** 3–6 plays a night max, never more than 5u at risk; minimum 3 points of edge; 1u standard, 0.5u for plus-money/thin edges; one parlay max; long shots ≤ 0.25u total. Never force picks — "0 bets tonight" is a valid card.
- **Two-pass review** every day: Review 1 = Ontario price + positive EV at the mean + at least one independent model. Review 2 = re-check your own reasoning: survives the low end? double-counting? does matchup history or a recent trend contradict it? do sources actually agree? Record every cut and why.
- **Matchup history** is a small-weight warning flag, never the main driver.
- **User bets** (`owner: "me"`): never change them except to grade results or when the user asks.
- Prefer props and college markets for diversity; sides/totals in the big leagues are the sharpest markets.

## Daily update procedure (`/daily-update`)

1. **Grade:** every bet with `result: "pending"` dated before today → look up final scores/box scores, set win/loss/push and a short `final` note. Set yesterday's `games` to `final` with scores.
2. **Slate:** add today's and tomorrow's games (NHL, NBA, NFL, NCAAF, NCAAB, plus soccer/MLB in season) to `games`.
3. **Prices + models:** gather Ontario book prices and independent probabilities. Good sources: DraftKings Network game previews (DraftKings prices), Covers, Dimers, DRatings, MoneyPuck, numberFire, Action Network projections, official injury reports, team news.
4. **Markets:** write today's `markets` docs (both sides' prices where available, sources, adjustments, h2h, warnings, news) and link `boardId`s to `board` docs with verdicts.
5. **Review:** write today's `review` doc from the engine numbers (compute them yourself with the formulas above).
6. **Card:** add Claude's bets for the final picks to `bets` (`owner: "claude"`, `p` = model probability). Leans go in at 0 units.
7. **Intel:** write today's `intel` doc.
8. Bump `updatedAt`, then `git add -A && git commit -m "Daily update YYYY-MM-DD" && git push`.

## Other commands

- `/log-bet` — add a bet for the user from their description.
- `/import-changes` — merge an `edgeline-changes.json` the user downloaded from the site (Settings → "Download my changes") into the data file, keeping whichever version of each doc is newer; drop entries with `_deleted`.
