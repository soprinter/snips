# Sharenote Specifications (SNIPs)

This directory contains the technical rules governing the Sharenote Proof-of-Work protocol.

| SNIP | Title | Description |
| :---: | :--- | :--- |
| **[00](./snip-00.md)** | [Core Z-Arithmetic & Hashrate Convergence](./snip-00.md) | Define Z-Bit difficulty labels, continuous difficulty math, and the mandatory aggregation rules for accumulating shares. |
| **[02](./snip-02.md)** | [Hashrate & Telemetry Event Specification](./snip-02.md) | Broadcast active miner hashrate and share telemetry on Nostr using `Kind 35502`. |
| **[03](./snip-03.md)** | [Decentralized Mining Pool Accounting](./snip-03.md) | Track pending miner shares, open pool invoices, and on-chain payouts using Nostr Kinds `35500-35505`. |
| **[04](./snip-04.md)** | [Raw Sharenote Minting (AuxPoW)](./snip-04.md) | Publish verifiable block headers as standalone Nostr events, including cross-chain Merged Mining (AuxPoW) data, using `Kind 35510`. |
| **[05](./snip-05.md)** | [Sharenote Identity (Miner & Pool)](./snip-05.md) | Register miner and pool identities over Nostr. Miners declare chain addresses (`Kind 10520`), pools declare profiles, fees, and payout schemes (`Kind 10521`). Both are replaceable events. |
