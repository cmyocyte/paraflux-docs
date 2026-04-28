# Liquidations

Positions that fall below the maintenance margin are liquidated automatically by third-party keepers. The system protects LPs through a layered defense: maintenance margin, liquidation bonus, and insurance fund.

## When Does Liquidation Happen?

A position is liquidated when:

```
equity < maintenanceMargin x notional
```

Where:
- **equity** = collateral + unrealized P&L - accrued funding
- **maintenanceMargin** = 15% of notional
- **notional** = position size x current index

### Example

You open a 2x long on HYPE at $45 with $500 collateral ($1,000 notional):
- Maintenance margin = 15% x $1,000 = $150
- You get liquidated when equity drops to $150
- That's a loss of $350 on $500 collateral (70% loss)
- At 2x leverage, this happens after roughly a ~18% adverse spot move (accounting for the squared payoff)

## Liquidation Price

For power perps (p=2), the liquidation price depends on leverage and the squared payoff curve:

| Leverage | Approx. Adverse Move to Liq (Long) |
|----------|-------------------------------------|
| 1x | ~54% drop |
| 1.5x | ~38% drop |
| 2x | ~28% drop |
| 2.5x | ~23% drop |
| 3x | ~19% drop |

These are approximate -- actual liquidation depends on accumulated funding and the squared index dynamics.

### Pessimistic Index

The LiquidationEngine uses a **pessimistic index** for safety:
- For longs: `min(EMA index, raw index)` -- uses whichever is worse
- For shorts: `max(EMA index, raw index)` -- uses whichever is worse

This prevents stale EMA prices from blocking valid liquidations.

## What Happens During Liquidation

1. **Keeper calls `liquidate()`** on the LiquidationEngine
2. **Position is closed** at current pessimistic index
3. **Remaining equity** is distributed:
   - Trader receives: equity minus liquidation bonus (if any equity remains)
   - Liquidator receives: 5% bonus (from trader's remaining equity)
   - If the position is underwater (negative equity): insurance fund covers the shortfall

## Insurance Fund

The insurance fund is the first-loss buffer before bad debt reaches LPs:

1. **Funded by**: 5% of LP trading fees (immutable, cannot be changed post-deploy)
2. **Used when**: A liquidated position has negative equity (bad debt)
3. **Flow**: InsuranceFund pays the vault to cover the shortfall
4. **Cap**: When the fund reaches its target size, the 5% fee slice redirects back to LPs
5. **Last resort**: If the insurance fund is empty, bad debt is absorbed by the vault via `absorbBadDebt()`

## Liquidation Keepers

Liquidations are executed by third-party keepers, not by Paraflux. Anyone can run a liquidation keeper:

- Keepers monitor position health across all markets
- Profitable liquidations earn the 5% bonus
- The LiquidationExecutor contract simplifies the keeper interface
- Keepers share infrastructure with the funding and vol oracle keepers

## Avoiding Liquidation

- **Use lower leverage**: 1x-1.5x gives you significantly more room
- **Monitor position health**: The UI shows health percentage and liquidation price
- **Add collateral**: You can top up your margin at any time
- **Close early**: Close your position before health drops too low
- **Watch funding**: Continuous funding erodes your collateral over time -- factor it into your holding period

## Key Parameters

| Parameter | Value | Mutable? |
|-----------|-------|----------|
| Maintenance margin | 15% of notional | Admin-configurable |
| Liquidation bonus | 5% of remaining equity | Admin-configurable |
| Insurance fee | 5% of LP fees | Immutable |
| Max position utilization | 20% of pool (hard cap 50%) | Admin-configurable |
