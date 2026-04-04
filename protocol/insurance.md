# Insurance Fund

The insurance fund is a first-loss buffer that absorbs bad debt before LPs are affected.

## How It Works

- **5% of all trading fees** flow to the insurance fund (immutable — cannot be changed by admin)
- When a liquidation results in bad debt, insurance fund absorbs it first
- Only if insurance is depleted does the LP vault absorb remaining bad debt
- When the fund reaches its target size, the 5% redirects back to LPs

## Fee Flow

```
Trader pays fee
  → 75% to LP Vault
  → 20% to Protocol Treasury
  → 5% to Insurance Fund (immutable)
```

## Bad Debt Flow

```
Liquidation with negative equity
  → Insurance Fund covers first
  → If insufficient → vault.absorbBadDebt(remainder)
```

## Key Properties

- `insuranceFeeBps` is set at deploy and **has no setter** — cannot be changed post-deploy
- Target size is configurable by admin
- When `isFull()`, insurance fee redirects to LPs (80% total)
- USDC flow: Router → vault.transferOut(insuranceFund, amt); Fund → usdc.safeTransfer(vault, amt)

## SDK

```typescript
const ins = await client.getInsuranceState();
console.log(`Balance: $${math.usdcToNumber(ins.balance).toFixed(2)}`);
console.log(`Target:  $${math.usdcToNumber(ins.targetSize).toFixed(2)}`);
console.log(`Full:    ${ins.isFull}`);
console.log(`Fill:    ${(ins.fundingRatio * 100).toFixed(1)}%`);
```
