# Anchor Vault (Coming Soon)

> **Coming soon** -- the Anchor vault will launch after the core power perp protocol is stable.

The Anchor Vault is a structured product that earns pure volatility funding income (sigma^2 per year) by being short p=2 and delta-hedging via CoreWriter.

## Strategy

1. Open a short p=2 power perp position (1.5x leverage)
2. Delta-hedge on Hyperliquid's native perp via CoreWriter
3. Earn the funding rate (sigma^2 per year) while staying delta-neutral
4. Rebalance when delta drifts > 10%

## Income Source

Short p=2 positions receive funding from longs at a rate of **sigma^2 per year**:

| Asset Vol | Annual Funding Rate | Daily per $10K |
|-----------|-------------------|---------------|
| 50% | 25% | $6.85 |
| 80% | 64% | $17.53 |
| 100% | 100% | $27.40 |

## When Anchor Works Best

| Market Condition | Anchor Performance | Why |
|-----------------|-------------------|-----|
| **Ranging (< 2% daily moves)** | **Best** -- high win rate | Funding >> gamma drag |
| **Crash** | **Positive** | Short benefits, hedge profits |
| **Bull** | **Negative** | Gamma drag exceeds funding |

Anchor is a **ranging/crash product**, not a bull product. For bull exposure, use Surge.

## Risk Factors

- **Gamma drag**: Large daily moves (>3%) cost more than funding earns
- **Hedge cost**: CoreWriter execution slippage on rebalancing
- **Linear perp funding**: The long hedge position pays linear perp funding
- **Single-asset risk**: Concentrated in one asset's vol (LP Vault diversifies across 9 markets)

## Multi-Asset Opportunity

Anchor on lower-vol assets would have ideal conditions -- low gamma drag, steady funding:

| Asset | Avg Vol | Expected Anchor APY |
|-------|---------|-------------------|
| Gold | 15-25% | 5-15% |
| Silver | 20-35% | 10-25% |
| HYPE | 50-100% | 25-50% (when ranging) |

## Current Alternative

While the Anchor vault is not yet available, you can replicate the strategy manually using the [Income strategy](protection.md) described in the Strategies page.
