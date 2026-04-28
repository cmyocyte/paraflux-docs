# Realized Vol Oracle

The RealizedVolOracle computes **realized variance from spot price log returns** -- entirely on-chain, with zero off-chain infrastructure. This is NOT an implied vol oracle. Pool-to-peer protocols cannot produce implied volatility because there is no bilateral price discovery. We are honest about what this measures: historical realized variance.

## Why It Matters

As of April 2026, no major oracle provider (Chainlink, Pyth) offers dedicated on-chain volatility feeds. Most DeFi lending protocols rely on off-chain vol analysis pushed through periodic governance votes. Paraflux computes realized variance natively on-chain across 9 markets — any smart contract can read it via a free `view` call.

## How It Works

### Cumulative Variance Accumulator

Every 5 minutes, a keeper calls `recordObservation()`:

```
logReturn = ln(spot_new / spot_old)
instantVariance = logReturn^2 * SECONDS_PER_YEAR / dt
cumulativeVariance += instantVariance
```

This is the **Uniswap TWAP pattern applied to variance**. Any consumer can diff two snapshots to get realized variance over any window.

### Multi-Window Term Structure

Three ring-buffer checkpoints provide windowed averages:

| Window | Purpose | Observations |
|--------|---------|-------------|
| 1-day | Short-term vol | ~288 |
| 7-day | Medium-term vol | ~2,016 |
| 30-day | Long-term vol | ~8,640 |

Plus a micro window (1-hour) for high-frequency applications.

Reading any window is O(1): `(cumulativeVariance - checkpoint) / n`

### Vol-of-Vol

A second-order accumulator tracks `cumulativeVarianceSquared`, enabling:

```
volOfVol = sqrt(E[var^2] - E[var]^2)
```

High vol-of-vol = regime transition (calm to volatile or volatile to calm). Useful for risk management and strategy signals.

### Forward Variance

```
forwardVar = (var7d * n7d - var1d * n1d) / (n7d - n1d)
```

- **Contango** (forward > spot): market is calm now, expects higher vol
- **Backwardation** (forward < spot): market is volatile, expects mean reversion

## What This Is NOT

This oracle measures **realized variance** -- what actually happened in the past. It does not produce implied volatility (what the market expects to happen in the future). The funding rate on power perps is set by realized variance, not by market consensus on future vol.

For implied volatility data, we plan to integrate **Volmex BVIV/EVIV** feeds in a future release, which will enable vol index trading and more sophisticated products.

## SDK Usage

### Read realized vol

```typescript
const surface = await client.getVolSurface('HYPE');
console.log(`Realized vol (1d): ${(surface.realizedVol1d * 100).toFixed(1)}%`);
console.log(`Vol-of-vol: ${(math.varianceToVol(surface.termStructure.volOfVol) * 100).toFixed(1)}%`);
```

### Read all 9 markets at once

```typescript
const surfaces = await client.getAllVolSurfaces();
for (const [asset, s] of surfaces) {
  console.log(`${asset}: ${(s.realizedVol1d * 100).toFixed(1)}%`);
}
```

### Historical vol data

```typescript
const history = await client.getVolHistory('HYPE', fromBlock);
// Returns: VolObservation[] with timestamp, spotPrice, variance, vol
```

### Stream real-time observations

```typescript
client.watchAllVolSurfaces((asset, obs) => {
  console.log(`${asset}: vol=${(obs.realizedVol * 100).toFixed(1)}%`);
});
```

## Integration Guide

### For Lending Protocols

```solidity
uint256 variance = volOracle.getCurrentVariance();
uint256 ltv = variance < 0.25e18 ? 8000 : variance < 1e18 ? 7000 : 5000;
```

### For Strategy Vaults

```solidity
uint256 volOfVol = volOracle.getVolOfVol();
if (volOfVol > 2e18) {
    // Regime transition -- deleverage
}
```

### For Risk Engines

```solidity
(uint256 var1d, uint256 var7d, uint256 var30d, uint256 vov) = volOracle.getVolTermStructure();
// Full term structure in one call
```

## Gas Costs

All `view` functions (reading vol data) are **free**. Only `recordObservation()` costs gas, running on HyperEVM's low-cost infrastructure.
