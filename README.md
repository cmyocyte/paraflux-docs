# What is Paraflux?

**Power perpetuals on HyperEVM. Squared returns. No options complexity.**

Paraflux is a power perpetual protocol on HyperEVM. Trade squared exposure (S^2) to crypto, commodities, and equities -- no strikes, no expiry, no fragmentation. The LP vault is the counterparty, delta-hedged via CoreWriter.

> **Status:** Live on HyperEVM testnet (chain 998). Mainnet (chain 999) imminent.
>
> [Launch App](https://app.paraflux.xyz) | [View Contracts](contracts/addresses.md) | [SDK Quick Start](sdk/quickstart.md)

---

## Why Paraflux?

- **Power Perpetuals** -- Squared exposure to any asset. No strikes, no expiry, no fragmentation. One instrument per market.
- **LP Vault** -- Deposit USDC. Earn trading fees, funding income, and hedged market-making yield. ERC-4626.
- **Realized Vol Oracle** -- On-chain realized variance computed natively from HyperEVM precompiles. Any protocol can consume it.

## Markets

### Phase 1 -- Launch

| Market | Asset | Class | Oracle |
|--------|-------|-------|--------|
| BTC^2 | Bitcoin | Crypto | Core perp (0x0807) |
| ETH^2 | Ethereum | Crypto | Core perp (0x0807) |

### Phase 2 -- Multi-Asset

| Market | Asset | Class | Oracle |
|--------|-------|-------|--------|
| HYPE^2 | Hyperliquid | Crypto | Core perp (0x0807) |
| SOL^2 | Solana | Crypto | Core perp (0x0807) |
| GOLD^2 | Gold | Commodity | HIP-3 BBO (0x080e) |
| SILVER^2 | Silver | Commodity | HIP-3 BBO (0x080e) |

### Phase 3 -- Expansion

| Market | Asset | Class | Oracle |
|--------|-------|-------|--------|
| XYZ100^2 | Nasdaq | Equities Index | HIP-3 BBO (0x080e) |
| TSLA^2 | Tesla | Equities | HIP-3 BBO (0x080e) |
| EUR/USD^2 | Euro/Dollar | Forex | HIP-3 BBO (0x080e) |

Vol oracles are already deployed on mainnet for all 9 assets. Power perp markets roll out in phases.

## Quick Start

### Read market data (no wallet needed)

```typescript
import { ParaFluxClient } from '@paraflux/sdk';

const client = new ParaFluxClient({
  chainId: 998,  // testnet (999 for mainnet)
  rpcUrl: 'https://rpc.hyperliquid-testnet.xyz/evm',
});

// Read volatility
const surfaces = await client.getAllVolSurfaces();
for (const [asset, s] of surfaces) {
  console.log(`${asset}: vol=${(s.realizedVol1d * 100).toFixed(1)}%`);
}
```

### Trade

```typescript
const client = new ParaFluxClient({
  chainId: 998,
  rpcUrl: 'https://rpc.hyperliquid-testnet.xyz/evm',
  privateKey: '0x...',
});

// Open a long power perp
await client.openPosition({
  asset: 'BTC',
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
| **See all markets** | [Markets](products/multi-asset.md) |
| **Build with the SDK** | [Quick Start](sdk/quickstart.md) |
| **Read vol data** | [Volatility Data](sdk/volatility.md) |
| **Common questions** | [FAQ](resources/faq.md) |
