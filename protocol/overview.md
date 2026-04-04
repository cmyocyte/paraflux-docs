# Protocol Overview

> **Status:** Live on HyperEVM mainnet (chain 999). All contracts immutable (no upgrades, no proxies).

Paraflux is a **power perpetual protocol** on HyperEVM. Traders get squared exposure (S^2) to 10 markets. The LP vault is the counterparty to every trade, delta-hedged automatically via CoreWriter.

## Architecture

```
Trader -> PowerPerpRouter -> PositionEngine (positions, collateral)
                          -> FundingEngine (funding accrual, vol oracle reads)
                          -> LPVault (fee income, P&L settlement, collateral lock)
                          -> HedgeManager -> CoreWriter -> Hyperliquid L1 order book
```

## Pool-to-Peer Model

Power perpetuals trade against the **LP vault**. There is no order book for power perps -- the vault provides instant liquidity at oracle price plus fees. This eliminates the cold-start liquidity problem that plagues order-book-based options protocols.

| Component | Role |
|-----------|------|
| PowerPerpRouter | Entry point for all trades. Handles fees, slippage, USDC conversion. |
| PositionEngine | Stores positions, tracks collateral, computes health. |
| FundingEngine | Accrues funding based on realized variance (sigma^2) and OI skew. |
| LPVault | ERC-4626 vault. Counterparty to all traders. Earns fees, bears gamma. |
| HedgeManager | Hedges the vault's delta exposure on Hyperliquid's native order book via CoreWriter. |
| LiquidationEngine | Permissionless liquidation with pessimistic index pricing. |
| InsuranceFund | First-loss buffer (5% of fees, immutable). |
| RealizedVolOracle | On-chain realized variance from spot price log returns. |

## How It Works

### For Traders

1. **Deposit USDC collateral** via the Router
2. **Open a position** (long or short, up to 3x leverage) on any of the 10 markets
3. **Pay/receive funding** every second based on sigma^2 (realized variance)
4. **Close anytime** -- no expiry, no exercise, no strikes

The payoff of a long p=2 position is proportional to `spot^2`. This gives convex exposure -- like a perpetual ATM straddle, but simpler.

### For LPs

1. **Deposit USDC** into the LP Vault (ERC-4626)
2. **Earn trading fees** (75% of all power perp fees), funding income, and net trader losses
3. **Protocol hedges delta** automatically via CoreWriter on Hyperliquid's native order book
4. **Withdraw anytime** (1-hour cooldown after deposit)

### Delta Hedging via CoreWriter

The LP vault's directional risk is hedged on Hyperliquid's native perpetual order book using the CoreWriter precompile (0x3333...3). This is an asynchronous fire-and-forget mechanism -- the HedgeManager sends hedge orders to HyperCore, which executes them on the L1 order book.

- **Crypto markets** (HYPE, BTC, ETH, SOL, XRP, SUI, DOGE): hedged via CoreWriter on core perps
- **Commodity/energy markets** (GOLD, SILVER, OIL): hedged via REST API on HIP-3 dexes

Hedge ratio: ~80%. Rebalance threshold: ~10% delta drift. Hedge cost: approximately 6% of LP fees.

## Key Properties

- **Immutable contracts** -- no upgrades, no proxy patterns
- **Permissionless** -- funding accrual and liquidations can be called by anyone
- **ERC-4626** -- LP vault follows the standard tokenized vault interface
- **10 markets** -- 7 crypto + 3 real-world assets (gold, silver, crude oil)

## Coming Soon

- **Anchor Vault** -- delta-neutral yield from short power perps + hedge
- **Surge Vault** -- leveraged long with LP yield buffer
- **Strategies** -- Protect, Amplify, Straddle, Income -- one-click compositions

## Future Roadmap

- Vol index trading via Volmex BVIV integration
- Variance swaps
- Options
- 50+ markets via MarketFactory
