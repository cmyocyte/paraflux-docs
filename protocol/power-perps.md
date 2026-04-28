# Power Perpetuals

Power perpetuals are perpetual futures where the payoff is a **power of the spot price** rather than the spot price itself.

## What Makes Them Different

| Feature | Linear Perp | Power Perp (p=2) |
|---------|------------|------------------|
| Payoff | S (spot) | S²/NORM (spot squared) |
| Exposure | Linear | Convex (option-like) |
| Funding | Based on mark vs index | Based on σ² (variance) |
| Expiry | None | None |
| Strikes | N/A | N/A |
| Greeks | Delta=1, Gamma=0 | Delta=2S/NORM, Gamma=2/NORM (constant!) |

## The p=2 Payoff

For HYPE at $35:
- **Index price** = 35² / 100 = $12.25
- If HYPE goes to $40: index = 40² / 100 = $16.00 (+30.6% vs +14.3% spot)
- If HYPE drops to $30: index = 30² / 100 = $9.00 (-26.5% vs -14.3% spot)

The squared payoff amplifies moves in both directions — **more upside AND more downside** than a linear perp. This is convexity.

## Funding Rate

Longs pay shorts a continuous funding rate equal to **σ²** (annualized variance):

```
daily_funding = notional × σ² / 365
```

At 80% volatility (σ = 0.8):
- σ² = 0.64
- Daily funding per $10,000 notional = $10,000 × 0.64 / 365 = **$17.53/day**

This is the "price of convexity" — longs pay for their option-like exposure.

## Why Power Perps Over Options?

| Problem with Options | Power Perp Solution |
|---------------------|-------------------|
| Fragmented across 100+ strikes/expiries | **One instrument per asset** |
| Need professional MMs to quote | **LP vault prices automatically** |
| Exercise/settlement complexity | **Perpetual — no expiry** |
| Complex Greeks management | **Gamma is constant** |
| Cold-start liquidity problem | **Vault IS the market maker** |

## Position Mechanics

### Opening
1. Choose direction (long/short) and collateral amount
2. Router computes position size from collateral × leverage
3. Fee deducted (base + impact)
4. Position stored in PositionEngine

### Holding
- Funding accrues every second
- Health monitored: `equity / maintenance_margin > 1`
- No expiry — hold indefinitely

### Closing
- Collect P&L: `size × (currentIndex - entryIndex)` for longs
- Pay/receive accumulated funding
- Closing fee deducted
- Collateral + net P&L returned in USDC

### Liquidation
- Triggered when health drops below maintenance margin (15%)
- Liquidator receives 5% bonus
- Insurance fund absorbs bad debt
- Remaining shortfall: LP vault absorbs

## Greeks

For p=2 power perps, all Greeks have **closed-form** solutions:

```typescript
import { math } from '@paraflux/sdk';

const greeks = math.computeAllGreeks(size, spot, sigma, isLong);
```

| Greek | Formula | Meaning |
|-------|---------|---------|
| **Delta** | 2 × size × S / NORM | Dollar change per $1 spot move |
| **Gamma** | 2 × size / NORM | **Constant** — doesn't change with price |
| **Vega** | 2σ × notional / 365 per 1% | Sensitivity to vol changes |
| **Theta** | σ² × notional / 365 | Daily funding cost (longs) / income (shorts) |
| **Dollar Gamma** | gamma × S² / 200 | P&L from 1% spot move |

**Gamma being constant is unique to p=2.** It means risk is perfectly predictable — the LP vault knows exactly how much gamma exposure it has at all times.
