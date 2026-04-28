# SDK Quick Start

The Paraflux SDK provides programmatic access to trading, LP operations, volatility data, and options Greeks across all power perp markets.

## Installation

**TypeScript**
```bash
npm install @paraflux/sdk
```

**Python**
```bash
pip install paraflux
```

> **Note**: The SDK is available on [GitHub](https://github.com/cmyocyte/paraflux-sdk). Check the repo for the latest version.

## Read-Only (No Wallet Required)

```typescript
import { ParafluxClient } from "@paraflux/sdk";

const client = new ParafluxClient({
  chain: 998, // testnet (999 for mainnet when live)
  rpc: "https://rpc.hyperliquid-testnet.xyz/evm",
});

// Get market data
const market = await client.getMarket("HYPE");
console.log(market.spot, market.fundingRate, market.openInterest);

// Get volatility surface
const vol = await client.getVolSurface("HYPE");
console.log(vol.vol1d, vol.vol7d, vol.vol30d);

// Get all vol surfaces at once
const allVol = await client.getAllVolSurfaces();
```

## Trading (Wallet Required)

```typescript
import { ParafluxClient } from "@paraflux/sdk";

const client = new ParafluxClient({
  chain: 998,
  rpc: "https://rpc.hyperliquid-testnet.xyz/evm",
  privateKey: process.env.PRIVATE_KEY,
});

// Open a 2x long on HYPE
const tx = await client.openPosition({
  asset: "HYPE",
  side: "long",
  sizeUsd: 1000,
  leverage: 2,
});

// Check your position
const pos = await client.getPosition("HYPE", "long");
console.log(pos.size, pos.health, pos.unrealizedPnl);

// Close position
await client.closePosition("HYPE", "long");
```

## LP Vault

```typescript
// Deposit 1000 USDC
await client.deposit(1000);

// Check share price and utilization
const vault = await client.getVaultStats();
console.log(vault.sharePrice, vault.utilization, vault.tvl);

// Withdraw (after 1-hour cooldown)
await client.withdraw(shares);
```

## Volatility Data

```typescript
// Single asset
const vol = await client.getVolSurface("BTC");
console.log(vol.realizedVol1d); // e.g. 0.35 (35%)
console.log(vol.forwardVariance); // 7d forward variance

// All assets
const surfaces = await client.getAllVolSurfaces();
for (const [asset, vol] of Object.entries(surfaces)) {
  console.log(`${asset}: ${(vol.vol30d * 100).toFixed(1)}%`);
}

// Historical
const history = await client.getVolHistory("ETH", { days: 30 });
```

## Options Greeks

```typescript
import { computeGreeks } from "@paraflux/sdk";

const greeks = computeGreeks({
  spot: 45.0,
  strike: 50.0,
  vol: 0.80,
  timeHorizon: 7, // days
  power: 2,
});

console.log(greeks.delta, greeks.gamma, greeks.theta);
```

## Advanced: Vol Arb Toolkit

```typescript
import { volArb } from "@paraflux/sdk";

// Compute breakeven vol for a position
const breakeven = volArb.breakevenVol({
  fundingRate: 0.0018, // 0.18%/day
  leverage: 2,
});

// Kelly-optimal sizing
const kelly = volArb.kellySize({
  realizedVol: 0.80,
  impliedVol: 0.65,
  bankroll: 10000,
});
```

## Chain Configuration

| Network | Chain ID | RPC |
|---------|----------|-----|
| Testnet | 998 | `https://rpc.hyperliquid-testnet.xyz/evm` |
| Mainnet | 999 | `https://rpc.hyperliquid.xyz/evm` |

## Further Reading

- [Volatility Data Guide](./volatility.md)
- [Fee Structure](./fees.md)
- [Markets](./markets.md)
- [GitHub: paraflux-sdk](https://github.com/cmyocyte/paraflux-sdk)
