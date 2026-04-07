SNIP: 00
Title: Core Z-Arithmetic & Hashrate Convergence
Status: Stable
Type: Standard
Created: 2026-04-03

This specification defines the core mathematical rules for calculating and aggregating Sharenote Proof-of-Work.

## Terminology

*   **Sharenote**: A cryptographically verifiable unit of proof-of-work.
*   **Z-Bits**: The base unit of measurement. It represents the logarithmic difficulty of a share.
*   **Discrete Labels**: Human-readable text strings (e.g., `34Z10`) used to route and sort Sharenotes. They declare the minimum difficulty floor of a share.
*   **Continuous Difficulty**: The exact mathematical value of a hash. Used by software to calculate true miner rewards.

Sharenote guarantees exact accounting by separating text labels from actual mathematical difficulty. This ensures every hash on the network is measured with continuous precision.

## Continuous Hash Density (Difficulty)

The metric of any proof-of-work share is its Continuous Hash Density (Difficulty), denoted as $D$. Difficulty is inversely proportional to the mathematical target.

For a given 256-bit cryptographic target $T$:
$$D = \frac{2^{256}}{T}$$

The fractional Z-bits represented by this physical target is:
$$B = \log_2(D)$$

When computing the value of a submitted share, systems **MUST** use the Continuous Hash Density ($D$).

> **Continuous Math Execution**
> All software implementations MUST accumulate shares using exact continuous difficulty. Adding two shares using their discrete labels instead of exact math causes precision loss and allows miners to game the system by halting hardware early.

## Discrete Denomination Class (Labels)

To allow humans and UI applications to route notes, Sharenote utilizes the Discrete Denomination Class, denoted `XZYY` where `X` is the integer bits and `YY` is the truncated centi-bits.

For a physical share with Z-bit difficulty $B$:
1. Compute the integer bits: $Z = \lfloor B \rfloor$
2. Compute the cents, applying the pricing floor ($+ 10^{-9}$): 
   $$C_{raw} = \lfloor ((B - Z) \times 100) + 10^{-9} \rfloor$$
3. Format the label as `Z` followed by `C_{raw}` clamped between $00$ and $99$.

This text string (e.g., `34Z10`) functions as a **Pricing Bucket Floor**. It indicates to downstream clients that the note contains *at least* $34.10$ bits of entropy. A cryptographic validator **MUST NOT** use this label to verify the target.

## Arithmetic in Sharenote Space

Arithmetic operations between Sharenotes MUST respect the logarithmic nature of the Z-unit system. 

Adding two Sharenotes by summing their Z components does **not** produce a correct combined note (e.g., `1Z00` + `1Z00` $\neq$ `2Z00`). To aggregate notes, developers **MUST** sum the work in linear difficulty space:

1. Convert notes to valid Continuous Density: $D_1 = 2^{B_1}$ , $D_2 = 2^{B_2}$
2. Sum linear difficulty: $D_{total} = D_1 + D_2$
3. Recover Z-bits: $B_{total} = \log_2(D_{total})$

> **IEEE-754 Precision Loss**
> While mathematical weight allows aggregation, software implementations MUST NOT use standard 64-bit floats as sequential accumulators over extended periods. When summing millions of micro-shares (e.g., `0Z01`) against a pool total, floating-point precision loss will drop fractional hashes. 
> To accumulate mining shares, pools MUST utilize rational number libraries or specialized fixed-point arithmetic variables.
