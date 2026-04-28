# Surge Vault (Coming Soon)

> **Coming soon** -- the Surge vault will launch after the core power perp protocol is stable.

Surge = 50% LP Vault + 50% leveraged long. Directional exposure with a yield buffer underneath.

## Strategy

1. 50% of capital goes to LP Vault (earns fees + funding)
2. 50% of capital goes to 3x leveraged long via CoreWriter
3. Daily rebalancing to maintain the 50/50 split

## Performance (Backtest)

| Metric | Surge | Hold HYPE | LP Vault |
|--------|-------|-----------|----------|
| Total Return | **253.5%** | 204.2% | 26.6% |
| Sharpe | 1.60 | 1.59 | 3.25 |
| Max DD | -75.9% | -64.2% | -7.1% |

In backtesting, Surge **outperformed buy-and-hold** because the LP Vault income buffer adds yield on top of the leveraged exposure. Past backtested performance does not guarantee future results.

## Who It's For

- Bullish on an asset but want yield income as a cushion
- Willing to accept high drawdowns for higher returns
- Want leveraged exposure without manual management

## Risk Factors

- **Leverage amplifies losses**: 3x on 50% = 1.5x effective exposure
- **Max drawdown**: -75.9% in the backtest
- **Rebalancing cost**: Daily rebalancing incurs slippage
- **LP Vault portion**: Subject to the same risks as the LP Vault (gamma, correlation)

## Current Alternative

While the Surge vault is not yet available, you can replicate the strategy manually by depositing into the LP vault and opening a leveraged long position on a native Hyperliquid perp via the Hyperliquid app.
