# Options (Roadmap)

> **This product is on the roadmap and not yet available.** Options will be introduced in a future release, trading bilaterally on a CLOB -- separate from the LP vault.

## Planned Design

Options on Paraflux will be **bilateral** -- traders trade with each other and with professional market makers, not against the LP vault. This keeps the vault focused on power perps while options scale independently.

Market makers will publish **parametric volatility surfaces** (SVI), and the system will compute prices for any strike and expiry from those parameters. This approach solves the liquidity fragmentation problem that has plagued previous on-chain options protocols.

## Why Power Perps First

Power perps solve the core problem that makes on-chain options hard:

| Problem with On-Chain Options | Power Perp Solution |
|-------------------------------|-------------------|
| Fragmented across 100+ strikes/expiries | **One instrument per asset** |
| Need professional MMs to quote | **LP vault prices automatically** |
| Cold-start liquidity problem | **Vault IS the market maker** |
| Exercise/settlement complexity | **Perpetual -- no expiry** |

Options will complement power perps when the protocol has enough volume and market maker interest to support bilateral trading.

## Current Alternative

Power perps replicate many options payoffs:

- **Long power perp** = perpetual ATM straddle (convex exposure, pays funding)
- **Short power perp** = perpetual short straddle (earns funding, short gamma)
- **Long power perp + delta hedge** = isolated gamma/vol trade
- **Short power perp + delta hedge** = delta-neutral vol selling (Anchor strategy)

See [Power Perpetuals](power-perps.md) and [Strategies](../products/protection.md) for details.
