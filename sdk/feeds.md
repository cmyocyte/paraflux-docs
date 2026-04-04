# Real-Time Feeds & Execution

Stream live data and execute trades with precision.

## Real-Time Feeds

Subscribe to live market data instead of polling.

```typescript
import { ParaFluxFeed } from '@paraflux/sdk';

const feed = new ParaFluxFeed({ rpcUrl: 'https://rpc.hyperliquid.xyz/evm' });

feed.onPrice((tick) => console.log(`HYPE: $${tick.spot.toFixed(2)}`));
feed.onFunding((tick) => console.log(`Funding: ${(tick.rate * 100).toFixed(1)}%`));
feed.onLiquidation((event) => console.log(`Liquidated: ${event.trader}`));
feed.onVolSurface((tick) => {
  tick.assets.forEach(a => console.log(`${a.asset}: ${(a.vol7d * 100).toFixed(0)}% vol`));
});

await feed.start();
```

### Available Streams

| Stream | Data | Interval |
|--------|------|----------|
| `onPrice` | Spot, index, EMA | Every block (~2s) |
| `onFunding` | Rate, OI, skew, cumulative | Every block |
| `onVolatility` | Per-asset realized vol (1d/7d/30d), vol-of-vol | Every block |
| `onVolSurface` | All 10 markets at once | Every block |
| `onPositionOpened` | Trader, size, collateral, direction | Event-driven |
| `onPositionClosed` | Trader, PnL, size | Event-driven |
| `onLiquidation` | Trader, size, bonus | Event-driven |
| `onVaultDeposit` | Depositor, amount, shares | Event-driven |

## Execution Helpers

### Fee Estimation

```typescript
import { estimateEffectiveFee } from '@paraflux/sdk';

const fee = estimateEffectiveFee({
  notionalUsd: 10_000,
  vaultTvl: 3_000_000,
});
// -> { totalFeeBps: 37, baseFeeBps: 7, impactFeeBps: 30, feeUsd: 37 }
```

### Optimal Sizing

Find the largest position that stays within risk constraints:

```typescript
import { optimalSize } from '@paraflux/sdk';

const size = optimalSize({
  capitalUsd: 10_000,
  maxLeverage: 3,
  maxUtilizationPct: 5,
  vaultTvl: 3_000_000,
  maxFeeBps: 50,
});
// -> { sizeUsd: 28_500, leverage: 2.85, effectiveFeeBps: 48 }
```

### Order Splitting

Split large orders to minimize quadratic impact fees:

```typescript
import { splitOrder } from '@paraflux/sdk';

const plan = splitOrder({
  totalSizeUsd: 100_000,
  vaultTvl: 3_000_000,
  maxFeeBps: 50,
});
// -> { chunks: [25000, 25000, 25000, 25000], totalFeeSaved: 1200, avgFeeBps: 38 }
```

### Health Simulation

Project what happens to your position at a target price:

```typescript
import { simulateHealth } from '@paraflux/sdk';

const sim = simulateHealth({
  collateralUsd: 5_000,
  notionalUsd: 15_000,
  isLong: true,
  entrySpot: 25,
  targetSpot: 20,    // what if HYPE drops to $20?
  fundingRateAnn: 0.64,
  holdDays: 7,
});
// -> { health: 0.31, pnlUsd: -2400, wouldBeLiquidated: false }
```

### Funding Scanner

Scan all 10 markets for mispricing opportunities:

```typescript
import { scanFundingOpportunities, ParaFluxClient } from '@paraflux/sdk';

const opps = await scanFundingOpportunities(client);
opps.forEach(o => {
  console.log(`${o.asset}: RV=${(o.realizedVol*100).toFixed(0)}% -> ${o.signal}`);
});
```
