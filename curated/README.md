# curated/

This folder holds **hand-curated catalog data** that supplements the
auto-discovered data the `tla-registry` cron pulls from chain + APIs.

## How this works

1. The cron reads these files from THIS repo's `curated/` folder daily.
2. It MERGES them with auto-discovered data (chain queries, Cosmos Chain
   Registry, Eris/Astroport/SS APIs).
3. Output goes to `../2026/current.json` with the merged catalog.

To add a new entry: edit the relevant file directly on GitHub
(web UI is fine, no clone needed). The next cron run picks it up.

## Files

| File | Purpose |
|---|---|
| `categories.json` | Taxonomy definition. Edit to add new category subtypes. |
| `wallets.json` | Known wallet labels — treasuries, members, bribers, allied DAOs. |
| `protocols.json` | Protocol metadata — Eris, Astroport, SS, etc. |
| `known_contracts.json` | Smart contract labels and metadata. Seeded from aDAO registry. |
| `token_overrides.json` | Per-token display name preferences, custom warnings. |
| `acquisition_guides.json` | How to safely acquire each token — bridge paths, warnings. |

## Editing tips

- Each file has `_meta.schema` showing the expected shape per entry.
- Each file has a `_curation_queue` section at the bottom — these are
  things we know we want to add but haven't filled in the address for yet.
  Move entries from there into the main list as you confirm.
- Underscore-prefixed keys (like `_example_wBTCatom_disabled`) are
  ignored by the cron. Remove the leading underscore to "activate" them.
- The cron is forgiving: missing files don't break it, malformed entries
  are skipped, and the catalog still publishes with whatever's valid.

## What goes where

Some judgment calls:

- **Wallets**: any EOA — a person, a DAO member, a briber, a treasury.
- **Contracts**: smart contracts that DO things — a DAO module, a staking
  contract, a marketplace contract, a bribe distributor.
- **Tokens**: fungible single-asset tokens (LUNA, USDC, ampLUNA). Each is
  one entry per Terra address, even if the same symbol shows up via
  multiple bridges (those are different entries with the same base symbol).
- **LP tokens**: catalogued automatically from gauge `distributions` —
  rarely needs curation.
- **amplp tokens**: catalogued automatically from `asset-compounder.asset_configs`.
- **Token overrides**: when Eris/Astroport name something poorly, override here.
- **Acquisition guides**: prose-style "how do I actually acquire this safely"
  — especially important for IBC tokens (USDC, wBTC variants) where Keplr
  search returns dozens of false matches.

## Confusion flagging — the wBTC case

If you spot multiple Terra addresses for what looks like the same token
(e.g. wBTC.atom vs wBTC.axl vs wBTC.osmo), you should:

1. Add ALL of them to `token_overrides.json` with their bridge details
2. Add acquisition guides showing the difference
3. The cron will auto-detect the shared base symbol and add the
   `shared_base_symbol_with` flag — this triggers the warning UI on
   the catalog view page.
