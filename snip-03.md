SNIP: 03
Title: Decentralized Mining Pool Accounting
Status: Draft
Type: Standard
Author: Decentralized Finance Working Group
Created: 2026-04-03
Dependencies: SNIP-00, NIP-01

This specification defines a public, cryptographically verifiable accounting pipeline for decentralized mining pools.

By migrating internal pool ledgers to the Nostr network (Kinds `35500`-`35505`), SNIP-03 allows mining pools to broadcast Pending Shares, Block Invoices, and On-chain Settlements directly to their workers. This architecture guarantees that miners can independently verify their unpaid balances and proportional payouts.

---

## 1. Pool Invoicing Logic

Before a pool can distribute rewards, it must declare the block it secured.

*   **Unpaid Invoice (`35500`)**: Emitted when a block is mined. Holds the total block reward but indicates miners are not yet paid.
*   **Settled Invoice (`35501`)**: Emitted when the on-chain payout transaction has been confirmed.

**Tag Structure:**

| Tag | Format | Required | Description |
| :-- | :----- | :------: | :---------- |
| `d` | `["d", "<HeightHash>"]` | Yes | Unique identifier. blake2b-256 hash of the block height integer. |
| `b` | `["b", "<BlockHash>"]` | Yes | 32-byte hex block hash. |
| `height` | `["height", "<int>"]` | Yes | Block height as a decimal integer. |
| `amount` | `["amount", "<int>"]` | Yes | Total block reward available in satoshis. |
| `workers` | `["workers", "<int>"]` | Yes | Number of participating workers. |
| `shares` | `["shares", "<Label>"]` | Yes | Pool's combined difficulty label. |
| `x` | `["x", "<Txid>"]` or `["x", "<Txid>", "<Height>", "<Hash>"]` | No | Settlement transaction. Short form for unconfirmed; full form with block height and block hash for confirmed. Required on Settled Invoices (`35501`). |

**Validation:**
- `HeightHash` MUST equal the blake2b-256 hash of the `height` value.
- `BlockHash` MUST be a valid 32-byte hex key.
- `height` MUST be greater than zero.

## 2. Miner Payout Slicing

Once the invoice is established, the pool publishes individual miner payout portions. Shares follow a three-stage lifecycle:

1. **Pending Share (`35503`)**: Unconfirmed per-worker shares. The `ShareID` is computed as `Hash(height, address, worker)`.
2. **Finalized Share (`35504`)**: Consolidated unpaid shares per address. The `ShareID` is recomputed as `Hash(height, address)`, merging all workers for that address.
3. **Share Payment (`35505`)**: Confirmed payout with on-chain transaction data. Includes the `chain` tag (defaults to `"15"` if absent).

When transitioning from Pending (`35503`) to Finalized (`35504`), the pool publishes deletion events (Kind `5`) referencing the pending share events being superseded.

**Tag Structure:**

| Tag | Format | Required | Description |
| :-- | :----- | :------: | :---------- |
| `d` | `["d", "<ShareID>"]` | Yes | Unique identifier. Computed hash (see lifecycle above). |
| `a` | `["a", "<Address>"]` | Yes | Miner's chain receiving address. |
| `h` | `["h", "<HeightHash>"]` | Yes | blake2b-256 hash of the block height. |
| `b` | `["b", "<BlockHash>"]` | Yes | 32-byte hex block hash. |
| `chain` | `["chain", "<ChainID_hex>"]` | No | Hex-encoded chain ID (e.g., `"15"` for chain 21). Required on Share Payment (`35505`), defaults to `"15"`. |
| `workers` | `["workers", "<name>", "<name>", ...]` | Yes | One or more worker names as separate values in the tag array. |
| `height` | `["height", "<int>"]` | Yes | Block height as a decimal integer. |
| `amount` | `["amount", "<int>"]` | Yes | Miner's payout amount in satoshis. |
| `shares` | `["shares", "<Label>", "<count>"]` | Yes | Miner's difficulty label and sharenote count. |
| `totalshares` | `["totalshares", "<Label>", "<count>"]` | Yes | Pool's aggregate difficulty label and total sharenote count. |
| `timestamp` | `["timestamp", "<unix_ms>"]` | Yes | Creation timestamp (milliseconds since epoch). |
| `fee` | `["fee", "<int>"]` | No | Transaction fee deducted from the miner's payout, in satoshis. |
| `eph` | `["eph", "<int>"]` | No | Estimated block height at which payment will occur. |
| `sn` | `["sn", "<event_id>", ...]` | No | References to Sharenote minting events (Kind `35510`). Multiple `sn` tags may be present. |
| `x` | `["x", "<Txid>"]` or `["x", "<Txid>", "<Height>", "<Hash>"]` | No | Settlement transaction. Short form for unconfirmed; full form for confirmed. Required on Share Payment (`35505`). |

**Validation:**
- `HeightHash` MUST equal the blake2b-256 hash of the `height` value.
- `BlockHash` MUST be a valid 32-byte hex key.
- At least one worker MUST be present.
- For pending shares (`35503`), `ShareID` MUST equal `Hash(height, address, first_worker)`.
- For paid shares (`35504`/`35505`), `ShareID` MUST equal `Hash(height, address)`.
- The sum of all individual `shares` difficulties across recipients MUST equal the declared `totalshares` when combined using SNIP-00 continuous arithmetic.

## 3. The Proportional Arithmetic Law

Dashboards verifying these events MUST implement linear continuous math (SNIP-00) to calculate the slice:
$$Miner\_Payout = \frac{\text{Miner Shares Difficulty}}{\text{Total Shares Difficulty}} \times \text{Invoice Amount}$$

Any dashboard or pool using string substitution or textual label truncation to perform payout slicing is non-compliant.
