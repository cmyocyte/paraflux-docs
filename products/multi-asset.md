# 10 Power Perp Markets

Paraflux launches with 10 power perpetual markets across crypto, commodities, and energy -- all trading against the shared LP vault.

## Markets

| Market | Asset | Class | Oracle Source | Hedge Method |
|--------|-------|-------|--------------|-------------|
| HYPE^2 | HYPE | Crypto | Core perp precompile (0x0807) | CoreWriter |
| BTC^2 | BTC | Crypto | Core perp precompile | CoreWriter |
| ETH^2 | ETH | Crypto | Core perp precompile | CoreWriter |
| SOL^2 | SOL | Crypto | Core perp precompile | CoreWriter |
| XRP^2 | XRP | Crypto | Core perp precompile | CoreWriter |
| SUI^2 | SUI | Crypto | Core perp precompile | CoreWriter |
| DOGE^2 | DOGE | Crypto | Core perp precompile | CoreWriter |
| GOLD^2 | Gold | Commodity | HIP-3 BBO precompile (0x080e) | REST API (HIP-3) |
| SILVER^2 | Silver | Commodity | HIP-3 BBO precompile | REST API (HIP-3) |
| OIL^2 | Crude Oil | Energy | HIP-3 BBO precompile | REST API (HIP-3) |

## Two Oracle Paths

**Crypto markets** (HYPE, BTC, ETH, SOL, XRP, SUI, DOGE) use the core perp oracle precompile at `0x0807`. These are Hyperliquid's native perpetual markets with deep liquidity.

**Real-world asset markets** (GOLD, SILVER, OIL) use the HIP-3 BBO precompile at `0x080e`, which returns the best bid and offer from builder-deployed perpetual dexes. The mid-price serves as the oracle input, with the same trust model as core perps.

## What You Can Trade

### GOLD^2, SILVER^2, OIL^2

Squared exposure to traditional assets, 24/7, permissionless. This product category does not exist anywhere -- not in DeFi, not in TradFi, not on Deribit.

### Crypto Squared

HYPE^2, BTC^2, ETH^2, SOL^2, XRP^2, SUI^2, DOGE^2 -- convex exposure to the top crypto assets. Amplify your directional conviction, or short for vol income.

### Cross-Asset Strategies

With 10 markets sharing one vault, you can:
- Go long HYPE^2 and short BTC^2 for relative vol exposure
- Short GOLD^2 to earn low-vol funding while staying long crypto markets
- Diversify vol selling across uncorrelated assets

## Adding New Markets

New markets can be added via the **MarketFactory** — a single transaction deploys all contracts and wires them into the shared LP vault. The team manages market listings; future plans include expanding to 50+ assets.

> **For traders:** You do not need to add markets. Just trade the ones already listed above. New markets are added by the team and appear automatically in the app and SDK.

## SDK

```typescript
// Read vol data for all 10 markets
const surfaces = await client.getAllVolSurfaces();

// Open a position on any market
await client.openPosition({
  asset: 'GOLD',
  isLong: true,
  usdAmount: 5000,
  leverage: 2,
});
```
