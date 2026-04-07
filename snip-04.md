SNIP: 04
Title: Raw Sharenote Minting & AuxPoW
Status: Draft
Type: Standard
Author: Core Cryptography Team
Created: 2026-04-03

This specification defines the standard formatting for packaging Merged Mining (AuxPoW) block headers into standalone, verifiable payloads.

By embedding proof-of-work Merkle branch data into Nostr objects (Kind 35510), SNIP-04 provides validators with direct cryptographic access to the raw execution logic required to authenticate a minted Sharenote.

## Cryptographic Header Payloads

When a note is minted, the broadcasting agent provides the unparsed header encoded in hex via the `dd` tag. Validators hash this hex to confirm it results in `d` (HeaderHash) and satisfies the leading-zero bounds dictated by `z` (Sharenote denomination).

**Tag Structure:**

| Tag | Format | Required | Description |
| :-- | :----- | :------: | :---------- |
| `d` | `["d", "<HeaderHash>"]` | Yes | Block header hash. Unique identifier for the sharenote. |
| `a` | `["a", "<address>", "<worker>", "<agent>"]` | Yes | Miner attribution: chain address, worker name, and user-agent string. |
| `z` | `["z", "<label>"]` | Yes | Sharenote denomination label, always lowercase (e.g., `"34z10"`). |
| `w` | `["w", "<BlockHash>", "<ChainID>", "<Height>", "<Solved>", "<Label>"]` | Yes | Auxiliary block entries (see below). One or more required. |
| `dd` | `["dd", "<HeaderHex>"]` | Yes | Full block header encoded as hex. Always the last tag. |

**Example:**
```json
[
  ["d", "0000000000000000abcd1234..."],
  ["a", "fc1qxyz...", "antminer-s19", "bmminer/2.0.0"],
  ["z", "34z10"],
  ["w", "000000ab...", "15", "843000", "true", "40z00"],
  ["w", "000001cc...", "2a", "110000", "false", "34z10"],
  ["dd", "0100000081cd02ab..."]
]
```

## Merged Mining (AuxPoW) Subsidiaries

A single hash execution can solve multiple independent blockchains if their targets are met concurrently. To map these subsidiary proofs, the event includes an array of `w` tags. 

The **primary chain** `w` tag MUST appear first, followed by auxiliary chain entries.

```json
[
  ["w", "<BlockHash>", "<ChainID>", "<Height>", "<Solved>", "<Label>"],
  ["w", "000000ab...", "15", "843000", "true", "40z00"],
  ["w", "000001cc...", "2a", "110000", "false", "34z10"]
]
```

**Tag Definitions:**
**`[1] BlockHash`**: The auxiliary target hash.
**`[2] ChainID`**: Hex-encoded integer network ID (e.g., `"15"` for decimal 21).
**`[3] Height`**: Chain block height as a decimal integer.
**`[4] Solved`**: Boolean string (`"true"` or `"false"`) indicating whether this hash reached the network subsidy target (`true`), or only passed the lower Sharenote boundary (`false`).
**`[5] Label`**: The Sharenote difficulty label for this branch (lowercase).
