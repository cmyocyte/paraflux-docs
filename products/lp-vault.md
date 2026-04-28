# LP Vault (Earn)

> [Launch App](https://app.paraflux.xyz)

The LP Vault is Paraflux's default product. Deposit USDC, earn yield from being the counterparty to all power perp traders across multiple markets.

## How It Works

1. **Deposit USDC** → receive ERC-4626 LP shares
2. **Vault earns** from three sources:
   - Trading fees (70% of all fees)
   - Funding income from leveraged traders
   - Net trader losses (most leveraged traders lose over time)
3. **Protocol hedges** delta exposure via CoreWriter on Hyperliquid's native order book
4. **Withdraw** USDC anytime (1-hour cooldown after deposit)

## Performance (Backtest)

365 days of real Hyperliquid data across 14 markets (HYPE, ETH, BTC, SOL, Gold, Silver, Oil — p=2 and p=0.5):

| Metric | Value |
|--------|-------|
| Total Return | **+141.4%** |
| Annualized Return | **+88.5%** |
| Sharpe Ratio | **12.08** |
| Sortino Ratio | 16.23 |
| Max Drawdown | **-2.5%** |
| Calmar Ratio | 34.76 |
| Win Rate | 81.4% |
| Worst Day | -1.5% |
| Best Day | +1.4% |

### Volume Sensitivity

Returns scale with trading volume. Even at conservative assumptions, the vault generates strong risk-adjusted returns:

| Scenario | Vol/TVL | Ann. Return | Sharpe | Max DD |
|----------|---------|------------|--------|--------|
| Very Conservative (25%) | 25% | +53.6% | 10.36 | -1.8% |
| Conservative (35%) | 35% | +68.6% | 10.54 | -2.2% |
| Base Case (50%) | 50% | +88.5% | 12.27 | -2.0% |
| Active (100%) | 100% | +130.3% | 16.80 | -1.6% |

## Revenue Attribution

At $1M TVL, base case (50% vol/TVL):

| Source | Annual | Daily Avg |
|--------|--------|-----------|
| Fee Income | +$1,945K | +$5,330 |
| Funding Income | +$43K | +$118 |
| Counterparty P&L | -$2,102K | -$5,760 |
| Hedge P&L | +$1,549K | +$4,246 |
| Hedge Cost | -$64K | -$176 |
| **Net** | **+$1,414K** | **+$3,874** |

Fee income is the dominant revenue driver. The hedge converts counterparty losses (gamma drag from large moves) into manageable net P&L. This is why the vault works even in trending markets.

## Multi-Market Diversification

The vault serves as counterparty across multiple assets simultaneously. p=0.5 (sqrt) markets provide a natural gamma offset — the vault is *long* gamma on sqrt positions, partially hedging its short gamma on p=2 positions.

Low or negative correlation with major crypto assets (ETH: -0.51, BTC: -0.37) means LP vault returns provide diversification in a crypto portfolio.

## Risk Factors

- **Gamma risk**: Large spot moves cause losses (partially offset by hedge). Worst day in backtest: -1.5%.
- **Hedge lag**: CoreWriter hedge rebalances every ~10 seconds, not every block. In a gap move, unhedged exposure is absorbed by the vault.
- **Correlation risk**: Multi-asset diversification helps, but correlated crashes hurt all positions simultaneously.
- **Smart contract risk**: Immutable contracts, 700+ tests, 4 audit passes — but bugs are always possible.
- **Concentration risk**: Single pool — one exploit drains everyone. Insurance fund absorbs bad debt first.

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
