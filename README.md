# Token Radar

![token-radar-01](https://make.fun/badge/token-radar-01.svg)
![rank](https://make.fun/badge/token-radar-01/reydenim/token-radar-01/rank.svg)
![reward](https://make.fun/badge/token-radar-01/reydenim/token-radar-01/reward.svg)

An entry for the [token-radar-01 bounty](https://make.fun/bounty/token-radar-01) on make.fun: a dashboard that finds the protocols whose token does not exist yet, ranks them by an implied value, and shows a reader how to farm each one.

## What it does

The radar reads a sealed snapshot of 758 DefiLlama protocols (fetched 2026-11-14) and scores every rankable pre-token protocol. The board holds **198 rows**: all 40 protocols on the snapshot's watchlist (at least $50M of TVL, no token on record) plus 158 more rankable protocols below the floor. Protocols whose token already trades are set aside in a separate Live view, where the market's own market cap is shown instead of a model number. Wrapped, staked and bridged products of tokens that already trade never rank: they are not protocols waiting on a launch.

## The model (radar-2)

Each protocol's implied value is one function of the snapshot's own figures, never a hardcoded list of names:

```
implied = round( tvl_usd
  x category_multiple
  x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
  x (0.6 + min(age_days, 1095) / 1095 x 0.4)
  x (1 + 0.05 x min(chains - 1, 3)) )
```

- **category_multiple** is the market's own price for the category: the median market-cap-per-TVL of the live-token protocols in the same category (at least two comps; otherwise the global median across all 129 live-token comps, x0.21). A category the market pays more for scores higher.
- **growth** is the protocol's seven-day TVL change, clamped to -60%..+120%, weighted by half.
- **maturity** is how long the protocol has run with no token, from the day DefiLlama began tracking it; an unlisted age counts as 365 days.
- **breadth** rewards protocols live on more chains, up to three extra.

The dashboard shows the full breakdown for every row and recomputes the number from the row's own factors, so the page and `board.json` agree to the dollar.

## Farm plans

Every row carries a seven-step farm plan derived from the protocol's category, chains, TVL and age: a fresh wallet, a first action specific to what the protocol does (supply, provide liquidity, bridge, stake, deposit into a vault, mint, buy cover), a second action of a different kind, a weekly cadence, a hold period, and how to check and claim. Capital is $50, $100, $150 or $250 by TVL; gas is priced by chain tier (Ethereum about $8, Solana about $0.05, L2s about $0.30, other chains about $1).

## The dashboard

`dashboard/` is a static page with no framework and no network requests: a sortable board (rank, protocol, TVL, 7-day growth, age, implied value), filters for category and chain, search, a pre-token / live-token switch, a detail view for each protocol with the model's factor breakdown and its farm plan, and a model panel that states the formula and the category multiples. Screenshots in `screenshots/` are real renders of it at 1440x900: the board, a detail view, and a filtered view.

## How to run it

Requires Node.js 22 or later. Standard library only; nothing is installed.

```sh
mkdir -p out
MAKEFUN_OUTPUT_DIR=out node main.ts --input path/to/snapshot.json
```

That writes `out/board.json` (the ranked board), `out/data.js` (the dashboard's data), `out/dashboard/` and `out/screenshots/` (copies of this repository's), and `out/dossier.json` (the summary the judge reads). Serve `out/` and open `dashboard/index.html`, or open the page straight from the built output.
