# What is Paraflux?

**Power perpetuals on HyperEVM. 10 markets. Squared returns.**

Paraflux is a power perpetual protocol on HyperEVM. Trade squared exposure (S^2) to crypto, commodities, and energy -- with no strikes, no expiry, and no fragmentation. The LP vault is the counterparty, delta-hedged via CoreWriter.

> **Status:** Live on HyperEVM testnet (chain 998). Mainnet (chain 999) imminent.
>
> [Launch App](https://app.paraflux.xyz) | [View Contracts](contracts/addresses.md) | [SDK Quick Start](sdk/quickstart.md)

---

## Why Paraflux?

- **Power Perpetuals** -- Squared exposure to any asset. No strikes, no expiry, no fragmentation. One instrument per market.
- **10 Markets** -- 7 crypto + 3 real-world assets (gold, silver, crude oil) via HIP-3. All tradeable 24/7.
- **LP Vault** -- Deposit USDC. Earn trading fees, funding income, and hedged market-making yield. ERC-4626.
- **Realized Vol Oracle** -- On-chain realized variance across all 10 markets. Any protocol can consume it.

## Markets

| Market | Asset | Class |
|--------|-------|-------|
| HYPE^2 | HYPE | Crypto |
| BTC^2 | BTC | Crypto |
| ETH^2 | ETH | Crypto |
| SOL^2 | SOL | Crypto |
| XRP^2 | XRP | Crypto |
| SUI^2 | SUI | Crypto |
| DOGE^2 | DOGE | Crypto |
| GOLD^2 | Gold | Commodity |
| SILVER^2 | Silver | Commodity |
| OIL^2 | Crude Oil | Energy |

## Quick Start

### Read market data (no wallet needed)

```typescript
import { ParaFluxClient } from '@paraflux/sdk';

const client = new ParaFluxClient({
  chainId: 999,
  rpcUrl: 'https://rpc.hyperliquid.xyz/evm',
});

// Read volatility across all 10 markets
const surfaces = await client.getAllVolSurfaces();
for (const [asset, s] of surfaces) {
  console.log(`${asset}: vol=${(s.realizedVol1d * 100).toFixed(1)}%`);
}
```

### Trade

```typescript
const client = new ParaFluxClient({
  chainId: 999,
  rpcUrl: 'https://rpc.hyperliquid.xyz/evm',
  privateKey: '0x...',
});

// Open a long power perp
await client.openPosition({
  asset: 'HYPE',
  isLong: true,
  usdAmount: 1000,
  leverage: 2,
});
```

## Get Started

| | Link |
|---|---|
| **Trade now** | [Launch App](https://app.paraflux.xyz) |
| **Understand the protocol** | [Protocol Overview](protocol/overview.md) |
| **See all 10 markets** | [Markets](products/multi-asset.md) |
| **Build with the SDK** | [Quick Start](sdk/quickstart.md) |
| **Read vol data** | [Volatility Data](sdk/volatility.md) |
| **Common questions** | [FAQ](resources/faq.md) |
