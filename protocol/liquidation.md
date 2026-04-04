# Liquidation

Positions are liquidated when their health ratio drops below the maintenance margin requirement.

## Health Ratio

```
health = (collateral + unrealizedPnL - fundingOwed) / maintenanceMargin
```

- **Healthy**: health > 1.0
- **Liquidatable**: health ≤ 1.0
- **Maintenance margin**: 5% of notional

## Liquidation Process

1. Anyone can call `liquidate(trader, isLong)` (permissionless)
2. LiquidationEngine verifies health < 1.0 using pessimistic index
3. Position closed at current market price
4. **Liquidator receives 5% bonus** on remaining collateral
5. If position is underwater (bad debt):
   - Insurance fund absorbs first
   - Remaining shortfall → LP vault (`absorbBadDebt`)

## Pessimistic Index

To prevent stale EMA from blocking liquidations, the LiquidationEngine uses:

- For **longs**: `min(emaIndex, rawIndex)` — uses the worse price
- For **shorts**: `max(emaIndex, rawIndex)` — uses the worse price

Multi-step EMA catch-up handles gaps: `computeMultiStepEMA()` uses `rpow((1-α), blockGap, WAD)` for O(log n) convergence.

## SDK

```typescript
// Check if a position is liquidatable
const { liquidatable, remainingMargin } = await client.isLiquidatable(trader, true);

// Get liquidation price
const liqPrice = await client.getLiquidationPrice(trader, true);
console.log(`Liquidation at HYPE = $${math.wadToNumber(liqPrice).toFixed(2)}`);

// Execute liquidation (earn 5% bonus)
if (liquidatable) {
  await client.liquidate(trader, true);
}
```
