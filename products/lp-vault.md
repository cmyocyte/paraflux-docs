# LP Vault (Earn)

> [Launch App](https://app.paraflux.xyz)

The LP Vault is Paraflux's default product. Deposit USDC, earn yield from being the counterparty to all power perp traders across 10 markets.

## How It Works

1. **Deposit USDC** -- receive ERC-4626 LP shares
2. **Vault earns** from three sources:
   - Trading fees (75% of all fees)
   - Funding income from leveraged traders
   - Net trader losses (most leveraged traders lose over time)
3. **Protocol hedges** delta exposure via CoreWriter on Hyperliquid's native order book
4. **Withdraw** USDC anytime (1-hour cooldown after deposit)

## Performance (Backtest)

365 days of real Hyperliquid data (HYPE, ETH, BTC, SOL):

| Metric | LP Vault | Covered Calls | Hold HYPE |
|--------|----------|-------------------------|-----------|
| Total Return | **26.6%** | 132.8% | 204.2% |
| Sharpe Ratio | **3.25** | 1.33 | 1.59 |
| Max Drawdown | **-7.1%** | -56.9% | -64.2% |
| Win Rate | **68%** | 51% | 50% |

The LP Vault has the **best risk-adjusted returns** (Sharpe 3.25) with the **lowest drawdown** (-7.1%).

## Revenue Attribution

| Source | Annual ($1M TVL) | % of Total |
|--------|-----------------|-----------|
| Trading Fees | $1,011K | 80% |
| Hedge P&L | $128K net | 10% |
| Funding Income | $25K | 2% |
| Hedge Cost | -$435K | -- |
| **Net** | **$266K** | -- |

Fee income is the dominant driver. This is why the vault works even in trending markets where gamma drag is high.

## Risk Factors

- **Gamma risk**: Large spot moves cause losses (partially offset by hedge)
- **Correlation risk**: Multi-asset means diversification, but correlated crashes hurt all positions
- **Smart contract risk**: Immutable contracts, 700+ tests, 4 audit reports -- but bugs are always possible
- **Hedge lag**: CoreWriter hedge rebalances every ~10 seconds, not every block

## SDK

```typescript
// Deposit
await client.deposit(10000);  // $10,000 USDC

// Check state
const vault = await client.getVaultState();
console.log(`Share price: $${vault.sharePrice.toFixed(4)}`);
console.log(`APY estimate: ${(vault.sharePrice - 1) * 100}%`);

// Withdraw
await client.withdraw(5000);
```
