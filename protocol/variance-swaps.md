# Variance Swaps (Roadmap)

> **This product is on the roadmap and not yet available.** Variance swaps will be introduced after the core power perp protocol is live.

Perpetual variance swaps will let you trade **realized volatility directly** -- no delta, no gamma, no strikes, no expiry. Pure vol exposure on a single asset.

## What Is a Variance Swap?

A variance swap is a bet on how much an asset's price moves over time. You agree on a "strike" (the expected variance), and your P&L is the difference between what actually happened (realized variance) and what was expected.

- **Long**: You profit when the asset moves **more** than expected.
- **Short**: You profit when the asset stays **calm**. You earn the "volatility risk premium."

The key insight: **you don't need to predict price direction**. A long var swap profits whether the asset crashes or rallies -- as long as the moves are big.

## How It Will Work

When you open a variance swap, the current windowed variance becomes your **strike**:

```
Long P&L  = notional * (realized variance - strike variance)
Short P&L = notional * (strike variance - realized variance)
```

Settlement is against the on-chain RealizedVolOracle -- trustless, continuous, verifiable.

## The Perpetual Mechanism

Unlike TradFi variance swaps that have fixed expiry dates, ours will be **perpetual** -- no expiry, hold as long as you want. A funding rate anchors the swap to fair value.

## Risk Warning

**Long variance** bleeds funding in calm markets. Only hold when you have a specific view on upcoming volatility.

**Short variance** has **unlimited loss potential**. A flash crash or market cascade can spike realized variance far above your strike. The loss on a single bad day can exceed months of accumulated premium.

## Current Alternative

While variance swaps are not yet available, you can get similar exposure through power perps:

- **Long vol**: Long power perp + delta hedge = profits from large moves (similar to long variance)
- **Short vol**: Short power perp + delta hedge = earns funding income (similar to short variance)

See [Strategies](../products/protection.md) for more on these compositions.
