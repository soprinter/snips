SNIP: 02
Title: Hashrate & Telemetry Event Specification
Status: Stable
Type: Standard
Author: Ops Team
Created: 2026-04-03
Dependencies: NIP-01 (Nostr Basic Protocol), SNIP-00 (Core Math)

This specification defines the standard event schema for streaming live Sharenote hashrate metrics.

By establishing a unified telemetry format over Nostr (Kind 35502), SNIP-02 enables seamless interoperability between mining hardware, pool monitors, and global hashrate aggregators.

### The Telemetry Payload

A telemetry broadcast (mapped to Nostr `Kind 35502`) requires an empty content field. All data is transported via tags to minimize payload size during high-frequency broadcasts.

**Identity Tag:**
*   `["a", "<address>"]` -> The chain address of the mining operation. Mandatory.

**Aggregate Mode (multi-worker):**
*   `["all", "<hashrate>", "msn:<label>"]` -> The total EMA-adjusted hashrate across all reporting workers (H/s), with the inline Mean Sharenote label. Mandatory when broadcasting aggregate farm telemetry.

**Single-Worker Mode:**
*   `["h", "<hashrate>", "msn:<label>"]` -> A single worker's hashrate and mean sharenote. Used when there is no aggregate total.

One of `all` or `h` MUST be present.

**Connected Workers:**
Individual ASICs or machines are appended to the broadcast using `w:` prefixed tags. Each worker tag carries a series of `key:value` sub-fields:

`["w:<worker_name>", "h:<hashrate>", "sn:<label>", "msn:<mean_label>", "csn:<count>", "crsn:<rejected>", "mt:<seconds>", "lsn:<unix>", "ua:<agent>"]`

*   `h:<hashrate>`: (Required) Worker's EMA-adjusted hashrate.
*   `sn:<label>`: Minimum difficulty actively assigned to the worker.
*   `msn:<mean_label>`: Mean Sharenote difficulty achieved by this worker.
*   `csn:<count>`: Total count of accepted sharenotes.
*   `crsn:<rejected>`: Count of rejected sharenotes. Omitted when zero.
*   `mt:<seconds>`: Mean time in seconds between accepted shares.
*   `lsn:<unix>`: Unix timestamp of the last submitted valid share.
*   `ua:<agent>`: User-agent string identifying the mining software.

Workers are sorted alphabetically by name.

### Privacy Recommendations

Third parties may monitor telemetry to identify ASIC models and hardware efficiency of operations based on the `Mean Time` (`mt:`) and `Mean Sharenote` (`msn:`) of individual workers. 

If hardware privacy is a concern, operators should rotate the `<worker_name>` identifier string rather than using static MAC addresses.

### Validation Constraints

Relays storing telemetry MUST enforce:
1. Presence of exactly one `a` tag.
2. An `all` or `h` tag containing a valid hashrate value.
3. Valid signatures (to prevent hashrate spoofing).
