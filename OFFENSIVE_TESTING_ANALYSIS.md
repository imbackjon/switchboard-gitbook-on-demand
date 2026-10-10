# Documentation Security Review — Switchboard On-Demand Integration Guides

Scope: review of the integration guidance in this repository for patterns that lead
consumers to build insecure integrations. No live deployment, RPC endpoint, or signing
material was used. Every finding below is sourced from repository content.

## Findings

### 1. Feed account validation shown commented out (High)
Files:
- `switchboard/readme/links-and-technical-documentation/integrating-on-chain-svm.md:45-49`
- `product-documentation/data-feeds/solana-svm/part-3-integrating-your-feed/integrating-your-feed-on-chain.md:48-51`

The examples parse the feed account with the public-key check present only as a comment:

```rust
// if ctx.accounts.feed.key != &specific_pubkey {
//     throwSomeError
// }
```

Impact: `feed` is declared as `AccountInfo<'info>` (unchecked). A caller can substitute an
arbitrary account, and a consumer following the example as written trusts whatever data is
passed. This is an account-substitution vector, and it is presented as the normal pattern
rather than as an omission to fix.

Fix: make the check live code in every example. Prefer an `address = ...` constraint, or
`Account<'info, ...>` with an explicit owner/type check, where the feed is fixed.

### 2. `unwrap()` on account parse (Medium)
Files:
- `switchboard/readme/links-and-technical-documentation/developers-build-and-use.md:52`
- `switchboard/readme/links-and-technical-documentation/integrating-on-chain-svm.md:52`
- `switchboard/readme/migrating-to-switchboard/using-pyth/with-solana.md:98`
- `product-documentation/data-feeds/solana-svm/part-3-integrating-your-feed/integrating-your-feed-on-chain.md:54`

`PullFeedAccountData::parse(feed_account).unwrap()` panics on malformed data. Return a
program error instead. On an instruction other code depends on, an unconditional panic is a
griefing surface.

### 3. Staleness and variance defaults undocumented (Medium)
Files:
- `switchboard/readme/links-and-technical-documentation/creating-a-solana-feed.md:115` (`maxStaleness: 60`)
- `switchboard/readme/links-and-technical-documentation/creating-a-solana-feed.md:244` (`maxStaleness: 300`)
- `product-documentation/data-feeds/starknet.md:56-58` (`maxStaleness: 100`, `maxVariance: 5e9`)
- `switchboard/readme/on-sui/getting-started.md:43-46`
- `switchboard/readme/on-aptos/getting-started.md:142-145`
- `product-documentation/data-feeds/movement.md:181-184`

These parameters decide whether stale or divergent data is accepted. The guides give values
with one-line comments and no discussion of the tradeoff. A consumer copying
`maxStaleness: 300` accepts roughly two minutes of staleness at ~400ms slots. `maxVariance`
ranges from `1.0` (Solana examples) to `1e9` (Sui/Aptos) to `5e9` (Starknet) across guides,
with no explanation of the unit or the consequence of the spread.

### 4. No read-time staleness enforcement shown (Low/Medium)
The `value(&clock)` / `get_value` read path does not itself bound staleness, and the
integration examples do not show passing an explicit tolerance at read time. Consumers may
reasonably assume the read is freshness-checked when it is not.

## Note on method
Read-only review of repository files. No live systems were contacted and no credentials or
signing material were used or requested.
