# tla-chain-registry

**The TLA ecosystem catalog data repo.** Produced by the `tla-registry` cron, consumed by every TLA-related page on `thealliancedao.com`.

This is a **data-only repo** — no scripts run here. The cron source lives in `defipatriot/cron-scripts` at `chain/tla-registry/`.

## What this repo holds

```
2026/
├── current.json    — latest snapshot (~500 KB, the canonical catalog)
├── heartbeat.json  — freshness signal for cron-health monitoring
└── daily/          — per-day archives (one file per UTC day)
    ├── 2026-06-01.json
    └── ...

curated/            — manually-maintained inputs the cron merges in
├── categories.json          — taxonomy
├── wallets.json             — known wallet labels
├── protocols.json           — protocol metadata
├── known_contracts.json     — contract labels (seeded from aDAO registry)
├── token_overrides.json     — display name / logo preferences per token
├── acquisition_guides.json  — how to safely acquire each token
└── curation-candidates.json — tooling (top unnamed TLA member wallets)
```

When 2027 arrives, a new `2027/` folder will be added. `2026/` stays frozen as historical record.

## What's in current.json

The catalog merges three classes of source for every TLA-relevant token, pool, contract, and wallet:

1. **On-chain queries** (Q1-Q6: global-config, asset-gauge, voting-escrow, asset-compounder) + per-pool `pair{}` and cw2 `contract_info` raw-storage reads — **~200+ LCD queries per run**
2. **External registries** (Cosmos Chain Registry, Eris prices, Astroport pools, Skeleton Swap pools, CoinGecko)
3. **Curated overrides** from this repo's `curated/` folder

Output schema (top-level keys):

| Key | Contents |
|---|---|
| `capturedAt`, `canonicalEpoch` | When + which epoch |
| `directory` | Master ve3 contract registry by role |
| `pools[]` | TLA-gauged LPs with bucket, distribution_pct, total_vp, **on-chain architecture** (contract, version, pair_type, dex) |
| `buckets{}` | Per-bucket totals |
| `tokens{}` | Every token in scope, keyed by Terra address — name, logo, decimals, CoinGecko match, source coverage |
| `amplp_mappings{}` | The 65 Eris asset-compounder vaults + their underlying LPs |
| `lp_to_amplp{}` | Reverse map (LP → amplp) |
| `contracts_catalog{}` | Labeled contracts (treasury, multisigs, ve3 components, DEX pairs) |
| `wallets_catalog{}` | Discovered wallets (TLA lockers, DAODAO stakers, NFT holders) with PFPK names + avatars where available |
| `scope.lp_to_underlyings{}` | LP address → [token addresses it holds] (from on-chain `pair{}` queries) |
| `scope.lp_to_architecture{}` | LP address → architecture object (from cw2 `contract_info` raw storage) |
| `source_coverage` | Per-source fetch status (which external sources succeeded) |
| `_errors[]` | Any non-fatal failures during the run |

## What's captured per run

Currently ~6 chain queries to bootstrap + ~200 follow-on LCD queries (pair{} per LP, cw2 contract_info per pair, minter{} per cw20 LP) + 5 external HTTP fetches + 7 curated file reads. Total runtime ~60-90s.

## Curated files

Edit via the GitHub web UI under `curated/`. The cron pulls them via SHA-pinned URLs to bypass Fastly cache (changes visible within minutes of push, not waiting for the 5-minute CDN cache to expire).

| File | Purpose |
|---|---|
| `categories.json` | Bucket / type taxonomy used by the catalog page |
| `wallets.json` | Known wallet labels (aDAO Treasury, aDAO Council Multi-Sig, council members) |
| `protocols.json` | Protocol metadata (Astroport, Eris, Skeleton Swap, etc.) |
| `known_contracts.json` | Contract address labels (seeded from the legacy aDAO contract registry) |
| `token_overrides.json` | Display-name and logo-URL overrides where Eris/chain registry disagree or are missing |
| `acquisition_guides.json` | Per-token "how to safely acquire" routes — manually verified by council members |

## Cadence + cost

- Runs once daily at 00:05 UTC
- ~60-90 seconds per run
- Source: Terra LCD endpoints (publicnode primary + fallback)
- Cost: effectively zero

## Why this exists

The previous generation of TLA tooling held hardcoded contract addresses and reconstructed pool lists from various indirect sources. This was the source of most "variant collision" bugs — the same pool appearing under multiple keys, dead post-migration pools mistaken for active ones, `(S)` Skeleton Swap variants conflated with their Astroport namesakes, etc.

The chain itself knows the truth. This catalog reads the chain and merges in external metadata + curation to produce a single source-of-truth artifact that every TLA-related page can consume.

## Source code

The cron writing here lives at:
**`defipatriot/cron-scripts/chain/tla-registry/`**

(Note: source folder is `chain/tla-registry/` for brevity; data repo is `tla-chain-registry` for description. Deliberately different names.)

Documentation for cron internals: `defipatriot/cron-scripts/chain/tla-registry/README.md`
Documentation for the catalog system + change history: `defipatriot/website-adao-core/catalog-log.md` and `PROJECT_KNOWLEDGE.md`.

## Status

**Phase 0 (data foundation): LOCKED IN as of 2026-06-06 (Rev 0.16).** Every category fully populated, zero data-quality issues, zero log noise.

Subsequent work builds on this foundation without changes to the catalog schema or contents:
- Phase 1: TLA Stats migration (existing `tla-stats.html` consumes catalog data)
- Phase 2: Member Stats `dao-tla.html` (new page using catalog as foundation)
- Phase 3: Portfolio Tracker (time-series + P&L)
- Phase 4: LP Health Scoring
- Phase 5: Bribes Tracking
- Phase 6: Vote Intelligence

Full Rev-by-Rev history in `catalog-log.md`.
