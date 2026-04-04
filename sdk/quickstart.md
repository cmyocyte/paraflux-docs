# SDK Quick Start

The Paraflux SDK provides full access to the protocol -- trading, LP operations, volatility data, and Greeks across 10 power perp markets.

## Install

**TypeScript** (recommended)
```bash
npm install @paraflux/sdk
```

**Python**
```bash
pip install paraflux
```

## Read-Only (No Wallet Needed)

### Connect

```typescript
import { ParaFluxClient } from '@paraflux/sdk';

const client = new ParaFluxClient({
  chainId: 999,
  rpcUrl: 'https://rpc.hyperliquid.xyz/evm',
});
```

### Market Data

```typescript
const market = await client.getMarketData('HYPE');
console.log(`HYPE spot: $${(Number(market.hypeSpot) / 1e18).toFixed(2)}`);
console.log(`Funding rate: ${(Number(market.fundingRate) / 1e18 * 100).toFixed(4)}%`);
```

### Realized Volatility (10 Markets)

```typescript
const surfaces = await client.getAllVolSurfaces();
for (const [asset, s] of surfaces) {
  console.log(`${asset}: vol=${(s.realizedVol1d * 100).toFixed(1)}%`);
}
```

### Greeks

```typescript
import { math } from '@paraflux/sdk';

const greeks = math.computeAllGreeks(
  math.numberToWad(1),   // 1 unit size
  math.numberToWad(35),  // $35 HYPE spot
  0.8,                   // 80% vol
  true,                  // long
);

console.log(`Delta: ${greeks.delta.toFixed(3)}`);      // 0.700
console.log(`Gamma: ${greeks.gamma.toFixed(4)}`);      // 0.0200 (constant!)
console.log(`Theta: $${greeks.theta.toFixed(2)}/day`); // -$17.52/day
```

## Trading (Wallet Required)

### Connect with Wallet

```typescript
const client = new ParaFluxClient({
  chainId: 999,
  rpcUrl: 'https://rpc.hyperliquid.xyz/evm',
  privateKey: '0x...',
});
```

### Open Position

```typescript
// Long HYPE^2: $1,000 collateral at 2x leverage
const tx = await client.openPosition({
  asset: 'HYPE',
  isLong: true,
  usdAmount: 1000,
  leverage: 2,
  slippageBps: 100,  // 1% slippage tolerance
});
console.log(`Opened: ${tx.hash}`);
```

### Check Position

```typescript
const pos = await client.getPosition(client.getAccount(), 'HYPE', true);
if (pos) {
  console.log(`Size: ${math.wadToNumber(pos.position.size)}`);
  console.log(`Health: ${pos.health.toFixed(2)}`);
  console.log(`P&L: $${math.wadToNumber(pos.unrealizedPnL).toFixed(2)}`);
}
```

### Close Position

```typescript
await client.closePosition('HYPE', true);  // close HYPE long
```

## LP Vault

```typescript
// Deposit $10,000 USDC
await client.deposit(10000);

// Check vault state
const vault = await client.getVaultState();
console.log(`Share price: $${vault.sharePrice.toFixed(4)}`);
console.log(`Utilization: ${(vault.utilization * 100).toFixed(1)}%`);

// Withdraw (after 1-hour cooldown)
await client.withdraw(5000);
```

## Vol Arb Toolkit

```typescript
// Breakeven vol for short position with 10bps hedge cost
const breakeven = math.computeBreakevenVol(0.8, 10);
console.log(`Breakeven: ${(breakeven * 100).toFixed(1)}%`); // 52.4%

// Kelly sizing for VRP trade
const kelly = math.computeKellyFraction(0.05, 0.1);
console.log(`Optimal size: ${kelly.toFixed(2)}x`); // 2.50x
```

## Python

```python
from paraflux import ParaFluxClient

client = ParaFluxClient(chain_id=999, rpc_url='https://rpc.hyperliquid.xyz/evm')

# Realized volatility
surface = client.get_vol_surface('HYPE')
print(f"HYPE realized vol: {surface.realized_vol_1d * 100:.1f}%")

# Greeks
from paraflux.math import compute_all_greeks
greeks = compute_all_greeks(size, spot, 0.8, True)
```

## Next Steps

- [TypeScript SDK Reference](typescript.md)
- [Python SDK Reference](python.md)
- [Volatility Data Guide](volatility.md)
- [Example Bots](examples.md)
