# Example Bots

Ready-to-run example bots demonstrating the SDK for power perp trading.

## 1. Index Arbitrage Bot

Trades deviations between the on-chain power index and fair value (spot^2/NORM).

```bash
cd sdk/typescript
RPC_URL=https://rpc.hyperliquid.xyz/evm \
PRIVATE_KEY=0x... \
THRESHOLD_BPS=30 \
POSITION_USD=1000 \
DRY_RUN=true \
npx tsx examples/arb-bot.ts
```

**Strategy**: When index deviates from fair value by >30bps, opens a position to capture the reversion.

## 2. Funding Harvester

Positions on the underpopulated side of the market to earn funding.

```bash
npx tsx examples/funding-harvester.ts
```

**Strategy**: Monitors OI skew. If longs dominate, goes short to earn elevated funding. Exits if direction flips.

## 3. Vol Regime Trading Bot

Trades based on on-chain realized volatility signals.

```bash
VOL_ASSET=HYPE \
POSITION_USD=1000 \
VOV_THRESHOLD=0.5 \
DRY_RUN=true \
npx tsx examples/vol-strategy.ts
```

**Signals**:
- **Vol-of-vol > 50%** -> REGIME_CAUTION: close positions
- **Forward contango** -> expect vol increase, adjust position sizing
- **High realized vol** -> funding rate will be expensive for longs

## 4. Multi-Asset Scanner

Not a trading bot -- a monitoring tool that scans all 10 vol oracles and prints actionable signals.

```bash
SCAN_INTERVAL=30 \
npx tsx examples/multi-asset-scanner.ts
```

**Output**:
- Per-asset vol z-scores (current vol vs 30-day mean)
- Term structure flags (contango/backwardation)
- Cross-asset dispersion level
- Funding rate extremes across all markets

## Python Equivalents

All bots have Python mirrors in `sdk/python/examples/`:
- `arb_bot.py`
- `funding_harvester.py`
- `vol_strategy.py`

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `RPC_URL` | testnet | HyperEVM JSON-RPC |
| `PRIVATE_KEY` | -- | Hex-encoded private key |
| `CHAIN_ID` | 998 | 998=testnet, 999=mainnet |
| `DRY_RUN` | true | Log without executing |
| `POSITION_USD` | 1000 | Collateral per trade |
| `LEVERAGE` | 2 | Position leverage |
| `POLL_INTERVAL` | 15 | Seconds between ticks |
