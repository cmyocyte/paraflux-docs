# Fee Structure

All fees are charged on-chain by Paraflux contracts at trade execution. No off-chain fees, no payment for order flow, no hidden costs.

## Trading Fees

```
effectiveFee = tradingFee + impactCoefficient × notional / totalAssets
```

| Component | Default Value | Description |
|-----------|--------------|-------------|
| Router base fee | 0.15% (15 bps) | Fixed percentage on all Router trades |
| Compiled strategy fee | 0.12% (12 bps) | Flat fee via CompilerExecutor (discount vs Router) |
| Impact coefficient | 0.3 (30%) | Scales quadratically with position size |

### Compiled Strategy Discount

Multi-leg strategies executed via **CompilerExecutor** pay a flat 12 bps on total strategy notional instead of the Router's 15 bps per leg. Manual decomposition of a 3-leg strategy would cost ~45 bps effective (3 x 15 bps); the compiled path costs just 12 bps — a 73% discount. This incentivizes traders to use the compiler for complex payoffs.

## Impact Fee

Protects LPs from concentrated informed flow. Quadratic in position size relative to pool depth:

- $1,000 trade into $500K pool: ~0.06% impact (negligible)
- $50,000 trade into $500K pool: ~3% impact (significant)
- $100,000 trade into $500K pool: ~6% impact (very expensive)

Large trades pay proportionally more. This is the LP's primary defense against informed flow.

## Fee Split

```
Total Fee
  ├── 70% → LP Vault (immediate income)
  ├── 25% → Protocol Treasury
  └── 5%  → Insurance Fund (immutable — cannot be changed post-deploy)
```

When the insurance fund reaches its target size, the 5% slice redirects back to LPs (75% total).

## Funding

Continuous funding accrues every second on-chain. Longs pay shorts.

### Base Rate

```
dailyFunding = σ² / 365 × notional
```

Where σ² is realized variance from the RealizedVolOracle.

### Examples at Different Vol Levels

| Asset Vol | Daily per $10K notional | Annual Rate |
|-----------|------------------------|-------------|
| 30% | $2.47/day | 9.0% |
| 50% | $6.85/day | 25.0% |
| 80% | $17.53/day | 64.0% |
| 100% | $27.40/day | 100.0% |

### Skew Adjustment

Funding rate adjusts based on OI skew. When longs dominate, rate increases. When shorts dominate, rate decreases. This incentivizes balance.

### For Power = 2

The funding formula simplifies: `p*(p-1)/2 = 1`, so the baseVolatility parameter IS the underlying σ directly. No scaling factor needed.

## No Hidden Costs

- No deposit fees
- No withdrawal fees (1-hour cooldown only)
- No gas rebates or priority fee manipulation
- All parameters visible on-chain and immutable (or admin-configurable within bounds)
