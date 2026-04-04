# Funding Rate

Power perp funding is the "price of convexity" — longs pay shorts for their squared exposure.

## Key Terms

| Term | Meaning |
|------|---------|
| **σ² (sigma squared)** | Variance — the square of annualized volatility. If HYPE vol is 80%, σ² = 0.64 |
| **σ (sigma)** | Annualized volatility. 80% = the asset's price typically moves ~80%/√365 ≈ 4.2% per day |
| **Skew** | The imbalance between long and short open interest. +1 = all longs, -1 = all shorts, 0 = balanced |

## Formula

For p=2 power perps:

```
funding_rate_per_second = σ² + skew × skewSensitivity
```

Where:
- **σ²** = the current realized variance from the on-chain RealizedVolOracle
- **skew** = (longOI - shortOI) / totalOI — market directional imbalance
- **skewSensitivity** = how much skew affects the rate (higher = more responsive to imbalance)

## Accrual

Funding accrues **every second** via a cumulative accumulator:

```
cumulativeFunding += fundingRatePerSecond × elapsed
```

Each position tracks its entry accumulator value. The funding owed is:

```
fundingOwed = size × (currentCumulativeFunding - entryAccumulatorValue)
```

## Examples

At 80% vol (σ = 0.8):
- σ² = 0.64
- Annual funding: **64% of notional**
- Daily: **$17.53 per $10,000 notional**

At 100% vol (σ = 1.0):
- σ² = 1.0
- Annual: **100% of notional**
- Daily: **$27.40 per $10,000 notional**

## Who Pays / Receives

| Side | Balanced Market | Long-Heavy | Short-Heavy |
|------|----------------|-----------|------------|
| Longs | Pay σ² | Pay σ² + skew | Pay σ² - skew |
| Shorts | Receive σ² | Receive σ² + skew | Receive σ² - skew |

## Key Properties

- **Permissionless accrual**: Anyone can call `accrueGlobalFunding()`
- **Reads vol oracle on-chain**: No keeper calibration lag when `setVolOracle()` is wired
- **Single accumulator**: O(1) for any number of positions
- **Falls back to baseVolatility**: If vol oracle not set, uses admin-configured σ

## SDK

```typescript
const market = await client.getMarketData();
const annualRate = Number(market.fundingRate) / 1e18;
console.log(`Annual funding rate: ${(annualRate * 100).toFixed(1)}%`);

// Forecast with additional position
const forecast = math.forecastFundingRate(longOI, shortOI, size, true, baseVol, skewSens);
```
