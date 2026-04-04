# Fee Model

Every trade pays a fee that protects LPs from informed flow and funds the protocol.

## Fee Formula

```
effectiveFee = tradingFee + impactCoefficient × notional / totalAssets
```

| Component | Default | Purpose |
|-----------|---------|---------|
| Trading fee | 7 bps (0.07%) | Base fee floor |
| Impact coefficient | 0.3e18 (30%) | Scales quadratically with size |
| Protocol fee | 20% of total fees | Treasury |
| Insurance fee | 5% of total fees | First-loss buffer (immutable) |
| LP share | 75% of total fees | LP vault income |

## Impact Fee (LP Protection)

The impact fee is **quadratic in position size relative to pool depth**:

| Trade Size | $1M Pool | $3M Pool | $10M Pool |
|-----------|----------|----------|-----------|
| $1,000 | 37 bps | 17 bps | 10 bps |
| $10,000 | 307 bps | 107 bps | 37 bps |
| $50,000 | 1,507 bps | 507 bps | 157 bps |

Large trades pay exponentially more. This protects LPs from informed flow.

## Fee Split

```
Total Fee
  ├── 75% → LP Vault (immediate income)
  ├── 20% → Protocol Treasury
  └── 5%  → Insurance Fund (immutable — cannot be changed)
```

When the insurance fund reaches its target, the 5% redirects back to LPs (80% total).

## SDK

```typescript
const fee = await client.estimateFee({
  isLong: true,
  usdAmount: 10000,
  leverage: 2,
});
console.log(`Total fee: ${math.wadToNumber(fee.totalFee).toFixed(2)} (${fee.effectiveBps} bps)`);
```
