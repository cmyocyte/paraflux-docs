# Greeks & Risk Analytics

Power perps with p=2 have **closed-form Greeks** -- no Black-Scholes, no simulation, no approximation. Gamma is constant (unique to p=2), making risk perfectly predictable.

## Quick Reference

| Greek | Formula | p=2 Property |
|-------|---------|-------------|
| **Delta** | `2 * size * S / NORM` | Linear in spot |
| **Gamma** | `2 * size / NORM` | **Constant** (does not change with price) |
| **Vega** | `2 * sigma * notional / 365` per 1% | Linear in sigma |
| **Theta** | `sigma^2 * notional / 365` daily | Quadratic in sigma |
| **Dollar Gamma** | `gamma * S^2 / 200` | P&L from 1% move |

## Usage

### Compute Greeks

```typescript
import { math } from '@paraflux/sdk';

const greeks = math.computeAllGreeks(
  math.numberToWad(10),   // 10 units
  math.numberToWad(35),   // $35 spot
  0.8,                    // 80% vol
  true,                   // long
);

// greeks = {
//   delta: 7.0,          // +$7 per $1 spot move
//   gamma: 0.2,          // constant
//   vega: 0.005,         // per 1% vol change
//   theta: -175.2,       // -$175/day funding cost
//   dollarGamma: 1.225,  // +$1.23 per 1% move
//   notional: 122.5,     // $122.50 notional
// }
```

### Position Greeks from Chain

```typescript
const greeks = await client.getPositionGreeks(traderAddress, 'HYPE', true);
```

### Individual Functions

```typescript
math.computeDelta(size, spot)        // -> number
math.computeGamma(size)              // -> number (constant!)
math.computeVega(notional, sigma)    // -> number (daily dollar per 1%)
math.computeTheta(notional, sigma, isLong)  // -> number (daily $)
math.computeDollarGamma(size, spot)  // -> number (P&L per 1% move)
```

## Vol Arb Toolkit

### Breakeven Vol

The realized vol at which a short vol position breaks even:

```typescript
const breakeven = math.computeBreakevenVol(0.8, 10);
// 80% realized vol, 10bps daily hedge cost -> breakeven at 52.4%
// If realized vol < 52.4%, short position is profitable
```

### Kelly Criterion

Optimal position sizing for the vol risk premium trade:

```typescript
const kelly = math.computeKellyFraction(
  0.05,   // 5% expected VRP
  0.10,   // 10% VRP standard deviation
  3,      // max leverage cap
);
// -> 2.5x (half-Kelly, capped at 3x)
```

### Expected P&L

```typescript
const pnl = math.computeExpectedVolPnL(
  100000,  // $100K notional
  0.64,    // current variance (80% vol)
  0.49,    // expected realized variance (70% vol)
  30,      // 30-day holding period
);
```

## Cross-Asset Theta Table

```typescript
const surfaces = await client.getAllVolSurfaces();
for (const [asset, s] of surfaces) {
  const theta = math.computeTheta(10000, s.realizedVol1d, false);
  console.log(`${asset}: earn $${theta.toFixed(2)}/day per $10K short`);
}
```

## Python

```python
from paraflux.math import (
    compute_all_greeks,
    compute_breakeven_vol,
    compute_kelly_fraction,
    compute_expected_vol_pnl,
)

greeks = compute_all_greeks(size_wad, spot_wad, 0.8, True)
breakeven = compute_breakeven_vol(0.8, 10)
kelly = compute_kelly_fraction(0.05, 0.1)
```
