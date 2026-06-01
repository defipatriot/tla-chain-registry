# tla-chain-registry

**Layer 0 of the TLA chain-native data pipeline.**
Reads the canonical contract registry + pool registry directly from the
Eris ve3 contracts on Terra (phoenix-1). Replaces the hardcoded contract
addresses scattered across the previous cron generation.

## What this repo holds

```
2026/
├── current.json    — latest snapshot (the dashboard reads this)
├── heartbeat.json  — freshness signal for the cron-health widget
└── daily/          — per-day archives (one file per UTC day)
    ├── 2026-05-31.json
    └── ...
```

When 2027 arrives, a new `2027/` folder will be added. `2026/` stays
frozen as historical record.

## What's captured

Per run (daily at 00:05 UTC):

- **Contract directory** — `global-config.all_addresses` query result
  (every ve3 contract by role: asset-gauge, voting-escrow, bribe-manager,
  asset-compounder, 4× asset-staking, 4× connector-alliance, etc.)
- **Pool registry** — `asset-gauge.distributions` query result
  (every pool with its canonical `gauge_pool_id`, bucket, reward share %)
- **Canonical epoch** — `asset-gauge.last_distribution_period`
  (the authoritative epoch number, settles the off-by-one issues)

## Why this exists

The previous generation of TLA crons each held hardcoded contract addresses
and reconstructed pool lists from various indirect sources. This was the
source of most "variant collision" bugs (the same pool appearing under
multiple keys, dead post-migration pools being mistaken for active ones).

The chain itself knows the truth. This layer reads it.

## Cadence + cost

- Runs once daily, ~10 chain queries per run
- Cost: effectively zero
- Source: Terra LCD endpoints (publicnode primary + fallback)

## Source code

The cron writing here lives at:
`defipatriot/cron-scripts/tla-chain-registry/`

## Part of a layered pipeline

- **Layer 0 — this repo** — discovery/bootstrap
- Layer 1 — pricing (planned)
- Layer 2 — entities (planned)
- Layer 3 — participants (planned)
- Layer 4 — rollups (planned)

See the cron-fixes brief (kept off-repo) for the full architecture plan.
