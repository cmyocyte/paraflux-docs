# TypeScript SDK

TypeScript SDK for the Paraflux Power Perpetual Protocol on HyperEVM. Built on [viem](https://viem.sh/) for type-safe EVM interaction.

## Install

```bash
npm install @paraflux/sdk
```

## Quick Start

```typescript
import { ParaFluxClient } from '@paraflux/sdk';

const client = new ParaFluxClient({
  chainId: 999,
  rpcUrl: 'https://rpc.hyperliquid.xyz/evm',
  privateKey: '0x...',
});

// Read market data
const market = await client.getMarketData('HYPE');
console.log('HYPE spot:', market.hypeSpot);
console.log('Funding rate:', market.fundingRate);

// Open a 2x long with $500 collateral
await client.openPosition({
  asset: 'HYPE',
  isLong: true,
  usdAmount: 500,
  leverage: 2,
});
```

## API Reference

### Read Methods (no signer needed)

| Method | Returns | Description |
|--------|---------|-------------|
| `getMarketData(asset?)` | `MarketData` | Index, spot, EMA, funding rate, OI, skew (multicall batched) |
| `getVaultState()` | `VaultState` | TVL, share price, utilization, available liquidity |
| `getPosition(trader, asset, isLong)` | `PositionDetails \| null` | Full position with P&L, health, liquidation price |
| `estimateFee({ asset, size?, usdAmount?, leverage? })` | `FeeQuote` | Preview base + impact fees before trading |
| `isLiquidatable(trader, asset, isLong)` | `{ liquidatable, remainingMargin }` | Health check |
| `getLiquidationPrice(trader, asset, isLong)` | `number` | Spot price threshold |
| `getInsuranceState()` | `InsuranceState` | Fund balance, target, fill ratio |

### Protocol Analytics

| Method | Returns | Description |
|--------|---------|-------------|
| `getTotalCollateralLocked()` | `bigint` | Total collateral locked (USDC 6-dec) |
| `getTotalTraderPnL()` | `bigint` | Aggregate trader P&L (WAD, signed) |
| `getAccumulatedFees()` | `bigint` | Total fees collected (USDC 6-dec) |
| `getTotalBadDebt()` | `bigint` | Bad debt absorbed by vault (USDC 6-dec) |
| `getTotalPositions()` | `bigint` | Total active position count |
| `getCumulativeFunding()` | `bigint` | Current funding accumulator (WAD) |
| `getCachedEmaIndex()` | `bigint` | Cached EMA index (WAD) |
| `getLastOracleUpdateBlock()` | `bigint` | Block number of last oracle update |
| `getLastDepositTime(depositor)` | `bigint` | Unix timestamp of last deposit (cooldown tracking) |

### Write Methods (require `privateKey`)

| Method | Returns | Description |
|--------|---------|-------------|
| `openPosition(params)` | `TxResult` | Open long or short (handles approval, slippage, fees) |
| `closePosition(asset, isLong, minPayoutUsd?)` | `TxResult` | Close a position |
| `addCollateral(asset, isLong, usdAmount)` | `TxResult` | Add collateral to existing position |
| `removeCollateral(asset, isLong, usdAmount)` | `TxResult` | Remove collateral (must stay above 20% margin) |
| `deposit(usdAmount)` | `TxResult` | Deposit USDC into LP vault |
| `withdraw(usdAmount)` | `TxResult` | Withdraw USDC from LP vault (1hr cooldown) |
| `redeem(shares)` | `TxResult` | Redeem LP shares for USDC |
| `liquidate(trader, asset, isLong)` | `TxResult` | Liquidate an unhealthy position (permissionless, earns 5% bonus) |

### Admin Methods (require owner key)

| Method | Returns | Description |
|--------|---------|-------------|
| `pause()` | `TxResult` | Pause the Router (no new trades) |
| `unpause()` | `TxResult` | Unpause the Router |
| `setTradingFee(feeWad)` | `TxResult` | Update base trading fee rate |
| `setImpactCoefficient(coeffWad)` | `TxResult` | Update impact fee coefficient |
| `setProtocolFeeBps(bps)` | `TxResult` | Update protocol fee share |
| `setMinPositionSize(sizeWad)` | `TxResult` | Update minimum position size |
| `setMaxPositionUtilizationBps(bps)` | `TxResult` | Update per-position utilization cap |

### Event Watchers

```typescript
// Watch liquidation events in real-time
const unwatch = client.watchLiquidations((log) => {
  console.log('Liquidation:', log.args);
});

// Also available: watchPositions(), watchVault(), watchFunding()
// Call unwatch() to stop listening
```

### Math Utilities

```typescript
import { math } from '@paraflux/sdk';

math.wadToNumber(index)              // WAD bigint -> float
math.numberToUsdc(100.5)             // float -> 6-dec USDC bigint
math.computeIndex(spotWad)           // spot^2 / NORM
math.computePnL(size, entry, current, isLong)
math.estimateFee(notional, tvl, tradingFee, impactCoeff)
math.computeHealth(collateral, pnl, fundingOwed, notional)
math.forecastFundingRate(longOI, shortOI, additionalSize, isLong, baseVol, skewSens)
```

## Example Bots

### Arb Bot

Trades index vs fair value (spot^2 / NORM). Opens when deviation exceeds threshold, closes when it narrows.

```bash
PRIVATE_KEY=0x... THRESHOLD_BPS=30 POSITION_USD=1000 npx tsx examples/arb-bot.ts
```

### Funding Harvester

Positions on the underpopulated side to collect funding payments.

```bash
PRIVATE_KEY=0x... MIN_FUNDING_ANNUAL=0.10 POSITION_USD=5000 npx tsx examples/funding-harvester.ts
```

Both bots support `DRY_RUN=true` to log what they *would* do without submitting transactions.

## Contract Addresses

Contract addresses are configured per-chain. Override via the `contracts` config option:

```typescript
const client = new ParaFluxClient({
  chainId: 999,
  rpcUrl: '...',
  contracts: {
    router: '0x...',
    oracle: '0x...',
    fundingEngine: '0x...',
    positionEngine: '0x...',
    lpVault: '0x...',
    liquidationEngine: '0x...',
    insuranceFund: '0x...',
    usdc: '0xb88339CB7199b77E23DB6E890353E22632Ba630f',
  },
});
```
