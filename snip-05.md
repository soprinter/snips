SNIP: 05
Title: Sharenote Identity (Miner & Pool)
Status: Draft
Type: Standard
Author: Ops Team
Created: 2026-04-07
Dependencies: NIP-01 (Nostr Basic Protocol), SNIP-00

This specification defines identity events for participants in the Sharenote mining ecosystem. Two distinct event kinds allow miners and pool operators to publish their profiles, chain addresses, and operational parameters over Nostr.

Both kinds fall within the NIP-01 Replaceable range (10000–19999). For each combination of pubkey and kind, relays store only the latest event. Publishing a new event replaces the previous one automatically — no `d` tag is needed.

---

## 1. Miner Identity (Kind 10520)

A miner publishes a single replaceable event declaring which chain addresses they control. This allows pools, dashboards, and settlement systems to discover where to send payouts.

**Content:** Empty string.

**Tag Structure:**

| Tag | Format | Required | Description |
| :-- | :----- | :------: | :---------- |
| `a` | `["a", "<chain_id>", "<address>"]` | Yes | Chain address binding. Hex-encoded chain ID and the miner's receiving address. One or more `a` tags allowed. |
| `payout` | `["payout", "<scheme>"]` | No | Miner's preferred payout scheme (e.g., `"pplns"`, `"pps"`). Informational hint to pools. |

### Miner Examples

**A) Single-chain miner:**

A miner operating on chain 21 (hex `"15"`), receiving payouts to one address.

```json
{
  "kind": 10520,
  "content": "",
  "tags": [
    ["a", "15", "fc1q0vw9cmzqqc3vxfhd7m4zcqf8n82s9gsmagr02a"]
  ]
}
```

**B) Multi-chain miner:**

A miner with addresses on two chains, preferring PPLNS payout.

```json
{
  "kind": 10520,
  "content": "",
  "tags": [
    ["a", "15", "fc1q0vw9cmzqqc3vxfhd7m4zcqf8n82s9gsmagr02a"],
    ["a", "01", "bc1qar0srrr7xfkvy5l643lydnw9re59gtzzwf5mdq"],
    ["payout", "pplns"]
  ]
}
```

**C) Miner updating their address:**

The miner publishes a new Kind 10520 event. Because it is replaceable, the relay overwrites the previous identity. Old addresses are no longer valid.

```json
{
  "kind": 10520,
  "content": "",
  "tags": [
    ["a", "15", "fc1qNEWADDRESS789xyz..."]
  ]
}
```

---

## 2. Pool Identity (Kind 10521)

A pool operator publishes a single replaceable event declaring their pool profile, accepted chains with fee schedules, supported payout schemes, and operational parameters.

**Content:** JSON object following the Kind 0 (NIP-01 `set_metadata`) convention:

```json
{
  "name": "<pool_name>",
  "about": "<description>",
  "picture": "<logo_url>",
  "website": "<pool_url>"
}
```

All fields are optional. An empty content string (`""`) is valid.

**Tag Structure:**

| Tag | Format | Required | Description |
| :-- | :----- | :------: | :---------- |
| `a` | `["a", "<chain_id>", "<address>", "<fee_bps>"]` | Yes | Chain address with pool fee. Fee is in basis points (1 bps = 0.01%). One or more `a` tags. |
| `payout` | `["payout", "<scheme>", "<key:value>", ...]` | Yes | Supported payout scheme. One or more `payout` tags. |
| `sharenote` | `["sharenote", "<min_label>"]` | No | Minimum accepted sharenote denomination floor (e.g., `"30z00"`). |
| `threshold` | `["threshold", "<chain_id>", "<amount>"]` | No | Minimum payout threshold in satoshis for the given chain. |

### Chain Address Tag (`a`)

```
["a", "<chain_id>", "<address>", "<fee_bps>"]
```

- **`chain_id`**: Hex-encoded chain ID (e.g., `"15"` for chain 21).
- **`address`**: The pool's hot-wallet or payout address for this chain.
- **`fee_bps`**: Pool fee in basis points as an integer string. `"200"` = 2.00%, `"50"` = 0.50%, `"0"` = no fee.

### Payout Scheme Tag (`payout`)

```
["payout", "<scheme>", "<key:value>", ...]
```

The scheme identifier is a lowercase string. Scheme-specific parameters follow as `key:value` pairs in tail positions (same convention as SNIP-02 worker tags).

**Defined Schemes:**

| Scheme | ID | Description | Parameters |
| :----- | :- | :---------- | :--------- |
| Pay Per Share | `pps` | Fixed payment per valid share. Pool absorbs block-finding variance. | — |
| Full Pay Per Share | `fpps` | PPS plus a share of transaction fee revenue. | — |
| Pay Per Last N Shares | `pplns` | Payment proportional to shares in a sliding window of the last N shares when a block is found. | `n:<window_size>` (required) |
| Proportional | `prop` | Simple proportional split of the block reward among all shares submitted in the round. | — |
| Solo | `solo` | Miner receives the full block reward. Pool only provides infrastructure. | — |

A pool MAY offer multiple schemes simultaneously by including multiple `payout` tags. The miner selects their preferred scheme via their Miner Identity `payout` tag or through an out-of-band configuration.

### Pool Examples

**D) Simple PPS pool, single chain:**

A pool accepting chain 21 with a 2% fee, paying per share.

```json
{
  "kind": 10521,
  "content": "{\"name\":\"AlphaPool\",\"about\":\"Pay-per-share mining pool\",\"website\":\"https://alphapool.io\"}",
  "tags": [
    ["a", "15", "fc1qpoolalpha...", "200"],
    ["payout", "pps"],
    ["sharenote", "30z00"]
  ]
}
```

**E) Multi-chain PPLNS pool:**

A pool accepting two chains with different fees, using PPLNS with a window of 10,000 shares.

```json
{
  "kind": 10521,
  "content": "{\"name\":\"BetaPool\",\"about\":\"PPLNS mining across multiple chains\",\"website\":\"https://betapool.io\"}",
  "tags": [
    ["a", "15", "fc1qpoolbeta...", "150"],
    ["a", "01", "bc1qpoolbeta...", "200"],
    ["payout", "pplns", "n:10000"],
    ["sharenote", "32z00"],
    ["threshold", "15", "100000"],
    ["threshold", "01", "50000"]
  ]
}
```

**F) FPPS pool with low fees:**

A pool offering Full PPS at 0.5% fee, with a 33Z00 minimum sharenote floor.

```json
{
  "kind": 10521,
  "content": "{\"name\":\"GammaPool\",\"about\":\"Full Pay-per-share with transaction fee revenue sharing\",\"picture\":\"https://gammapool.io/logo.png\",\"website\":\"https://gammapool.io\"}",
  "tags": [
    ["a", "15", "fc1qpoolgamma...", "50"],
    ["payout", "fpps"],
    ["sharenote", "33z00"]
  ]
}
```

**G) Solo mining pool (infrastructure only):**

A pool providing stratum infrastructure but passing the full block reward to the miner. 1% infrastructure fee.

```json
{
  "kind": 10521,
  "content": "{\"name\":\"SoloForge\",\"about\":\"Solo mining infrastructure\",\"website\":\"https://soloforge.io\"}",
  "tags": [
    ["a", "15", "fc1qpoolsolo...", "100"],
    ["payout", "solo"],
    ["sharenote", "34z00"]
  ]
}
```

**H) Multi-scheme pool (miner chooses PPS or PPLNS):**

A pool offering both PPS and PPLNS. PPS has a higher fee (3%) because the pool absorbs variance. PPLNS has a lower fee (1.5%) because miners share the risk.

```json
{
  "kind": 10521,
  "content": "{\"name\":\"DeltaPool\",\"about\":\"Choose your payout: PPS or PPLNS\",\"website\":\"https://deltapool.io\"}",
  "tags": [
    ["a", "15", "fc1qpooldelta...", "150"],
    ["payout", "pps", "fee:300"],
    ["payout", "pplns", "fee:150", "n:5000"],
    ["sharenote", "30z00"]
  ]
}
```

When a pool offers multiple schemes with different fees, each `payout` tag MAY include a `fee:<bps>` parameter that overrides the base fee declared in the `a` tag for miners enrolled in that scheme. The `a` tag fee serves as the default.

**I) Proportional pool, no minimum, zero fee:**

A community-operated pool with no fees and no minimum sharenote requirement.

```json
{
  "kind": 10521,
  "content": "{\"name\":\"CommunityPool\",\"about\":\"Free proportional mining pool\"}",
  "tags": [
    ["a", "15", "fc1qpoolcommunity...", "0"],
    ["payout", "prop"]
  ]
}
```

---

## 3. Validation Constraints

### Miner Identity (Kind 10520)

1. At least one `a` tag MUST be present.
2. Each `a` tag MUST contain exactly 3 elements: `["a", "<chain_id>", "<address>"]`.
3. Chain IDs MUST be valid hex strings.
4. Duplicate chain IDs within a single event are NOT allowed.
5. The `payout` tag is optional and limited to one.

### Pool Identity (Kind 10521)

1. At least one `a` tag MUST be present.
2. Each `a` tag MUST contain exactly 4 elements: `["a", "<chain_id>", "<address>", "<fee_bps>"]`.
3. `fee_bps` MUST be a non-negative integer string.
4. At least one `payout` tag MUST be present.
5. PPLNS payout tags MUST include the `n:<window_size>` parameter.
6. `content`, if non-empty, MUST be valid JSON.
7. Chain IDs MUST be valid hex strings.
8. Duplicate chain IDs within a single event are NOT allowed.
