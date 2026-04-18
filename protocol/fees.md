# Fee Structure

All fees are charged on-chain by Paraflux smart contracts at the time of trade execution. There are no off-chain fees, no payments for order flow, and no hidden costs.

## Trading Fees

Every trade pays a fee composed of two parts:

```
effectiveFee = tradingFee + impactCoefficient x notional / totalAssets
```

| Component | Default Value | Description |
|-----------|--------------|-------------|
| Trading Fee | 0.30% (30 bps) | Fixed percentage of notional |
| Impact Coefficient | 0.3 (30%) | Scales quadratically with trade size relative to pool |
| Protocol Split | 20% of total fees | Directed to protocol treasury |
| Insurance Split | 5% of LP fees | First-loss buffer (immutable, set at deploy) |

### Impact Fee

The impact fee protects LPs from concentrated informed flow. It's quadratic in position size:

- A $1,000 trade into a $500K pool pays ~0.06% impact (negligible)
- A $50,000 trade into a $500K pool pays ~3% impact (significant)
- A $100,000 trade into a $500K pool pays ~6% impact (very expensive)

This ensures large trades pay proportionally more, reflecting their outsized impact on the vault.

### Fee Split

```
Total Fee
  |-- 20% --> Protocol Treasury
  |-- 80% --> LP Pool
        |-- 95% --> LPs
        |-- 5%  --> Insurance Fund (immutable)
```

When the insurance fund reaches its target size, its 5% slice redirects back to LPs.

## Funding

Funding is continuous -- it accrues every second, on-chain. Longs pay shorts.

### Base Rate

```
dailyFunding = sigma^2 / 365 x notional
```

Where sigma^2 is the realized variance from the on-chain RealizedVolOracle. When the ImpliedVolOracle is set, it uses live implied variance instead.

### Examples at Different Vol Levels

| Asset Vol | Daily Funding (per $1,000 notional) | Annual Rate |
|-----------|-------------------------------------|-------------|
| 30% | $0.25/day | 9.0% |
| 50% | $0.68/day | 25.0% |
| 80% | $1.75/day | 64.0% |
| 100% | $2.74/day | 100.0% |

### Skew Adjustment

Funding rate adjusts based on OI skew (long OI vs short OI). When longs dominate, the rate increases. When shorts dominate, it decreases. This incentivizes balance.

### For Power = 2

Because p=2, the funding formula simplifies: `p*(p-1)/2 = 1`, so the base volatility parameter IS the underlying sigma directly. No sqrt(2) scaling needed.

## No Hidden Costs

- No deposit fees
- No withdrawal fees (1-hour cooldown only)
- No gas rebates or priority fee manipulation
- All fee parameters are visible on-chain and immutable (or admin-configurable within defined bounds)
