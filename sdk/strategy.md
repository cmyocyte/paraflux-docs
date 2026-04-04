# Strategy Framework

Build automated trading strategies by extending `BaseStrategy`. You implement `onTick()` -- the framework handles position management, risk limits, PnL tracking, execution, and graceful shutdown.

## Quick Start

```typescript
import { BaseStrategy, TickContext, ParaFluxClient } from '@paraflux/sdk';

class MyBot extends BaseStrategy {
  async onTick(ctx: TickContext) {
    if (ctx.fundingRate > 0.5 && !ctx.position) {
      await this.openShort({ asset: 'HYPE', usdAmount: 5000, leverage: 2 });
    }
    if (ctx.position && ctx.fundingRate < 0.1) {
      await this.closePosition();
    }
  }
}

const client = new ParaFluxClient({ chainId: 999, rpcUrl: '...', privateKey: '0x...' });
new MyBot({ client, name: 'FundingBot', tickInterval: 10_000 }).run();
```

## TickContext

Every tick, your strategy receives a complete market snapshot:

| Field | Type | Description |
|-------|------|-------------|
| `spot` | number | Current spot price |
| `index` | number | S^2/NORM |
| `fundingRate` | number | Annualized funding rate |
| `longOI` / `shortOI` | number | Open interest by side |
| `skew` | number | OI imbalance |
| `vaultTvl` | number | LP vault TVL |
| `position` | PositionSnapshot \| null | Your current position |
| `sessionPnl` | number | PnL since strategy started |
| `dailyPnl` | number | PnL today |
| `drawdown` | number | Current drawdown from peak (%) |

## Risk Management

Built-in risk limits that auto-trigger `emergencyClose()`:

```typescript
new MyBot({
  client,
  name: 'SafeBot',
  tickInterval: 10_000,
  maxPositionUsd: 50_000,     // max position size
  maxDrawdownPct: 15,          // stop if 15% drawdown
  maxDailyLossUsd: 2_000,     // stop if $2K daily loss
  dryRun: true,                // log only, no execution
}).run();
```

## Lifecycle Hooks

| Hook | When | Use case |
|------|------|----------|
| `onTick(ctx)` | Every tick (required) | Your signal logic |
| `onStart()` | Once on startup | Initialize state, load config |
| `onStop()` | On shutdown | Save state, log final PnL |
| `onFill(fill)` | After a trade executes | Track fills, update models |
| `onError(err)` | On errors | Alert, retry logic |

## Backtesting (Python)

Run any strategy against historical Hyperliquid data. Same code works live and backtested.

```python
from paraflux.strategy import BaseStrategy, TickContext
from paraflux.backtest import Backtester, BacktestConfig

class FundingBot(BaseStrategy):
    async def on_tick(self, ctx: TickContext):
        if ctx.funding_rate > 0.50 and not ctx.position:
            await self.open_short(usd_amount=5000, leverage=2)
        if ctx.position and abs(ctx.funding_rate) < 0.10:
            await self.close_position()

bt = Backtester(BacktestConfig(
    strategy_class=FundingBot,
    asset='HYPE',
    start_date='2025-06-01',
    end_date='2026-03-01',
    initial_capital=10_000,
))
result = await bt.run()
result.print_summary()
result.plot()
```

## Example Strategies

### Funding Harvester
Positions on the underpopulated side when funding is extreme (>50% annualized).

### Multi-Asset Scanner
Standalone tool that scans all 10 vol oracles and flags opportunities: vol z-scores, term structure inversions, cross-asset dispersion.

See `sdk/typescript/examples/` and `sdk/python/examples/` for full source code.
