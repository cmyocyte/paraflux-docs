# Protocol Overview

Paraflux is a power perpetual protocol on HyperEVM. Traders get squared exposure (S^2) to crypto, commodities, and equities -- no strikes, no expiry, no options complexity.

## How It Works

Power perps are perpetual futures where the payoff is a power of the spot price. A 10% spot move produces roughly a 21% index move. Returns accelerate -- the bigger the move, the more you make.

Instead of an order book, the **LP vault** is the counterparty to every trade. Traders open and close positions instantly at oracle price plus fees. No slippage from thin order books, no waiting for a match.

## Architecture

```
Trader --> PowerPerpRouter --> PositionEngine (open/close)
                           --> FundingEngine  (continuous funding)
                           --> LPVault        (liquidity + settlement)
                           --> OracleModule   (EMA-smoothed price)
```

- **PowerPerpRouter**: Entry point for all trades. Handles fee calculation, collateral transfer, and position creation.
- **PositionEngine**: Stores positions, enforces margin requirements, tracks P&L.
- **FundingEngine**: Computes and accrues funding based on realized variance and OI skew. Funding is continuous -- per-second, on-chain.
- **LPVault**: ERC-4626 vault. Receives USDC deposits, provides liquidity, earns fees. Delta-hedged via CoreWriter on Hyperliquid's native order book.
- **OracleModule**: Reads spot prices from HyperEVM precompiles (0x0807 for core perps, 0x080e for HIP-3 assets). Applies EMA smoothing with circuit breaker protection.
- **LiquidationEngine**: Liquidates undercollateralized positions. 5% liquidator bonus. Insurance fund absorbs bad debt.

## For Traders

- Deposit USDC collateral, open long or short positions
- Up to 3x leverage
- No expiry -- hold as long as you want
- Pay continuous funding (based on realized variance)
- Close anytime at current oracle price
- Liquidation at 15% maintenance margin

## For LPs

- Deposit USDC into the vault, receive ERC-4626 shares
- Earn trading fees (70% of all fees), funding income, and net trader losses
- Delta risk is automatically hedged via CoreWriter on Hyperliquid's native perp order book
- 1-hour withdrawal cooldown after deposit
- Backtested performance: +141.4% annual return, Sharpe 12.08, -2.5% max drawdown (365 days of real Hyperliquid data across 14 markets, walk-forward validated)

## Markets

Paraflux launches with BTC and ETH power perps, with HYPE, SOL, gold, and silver following shortly after. Phase 3 adds equities and forex (XYZ100/Nasdaq, TSLA, EUR/USD). Vol oracles are already deployed on mainnet for all 9 assets.

All contracts are immutable and non-upgradeable. The protocol is currently live on HyperEVM testnet (chain 998) with mainnet deployment (chain 999) imminent.

## Links

- **App**: [app.paraflux.xyz](https://app.paraflux.xyz)
- **Testnet**: Chain 998
- **Disclosures**: [app.paraflux.xyz/disclosures](https://app.paraflux.xyz/disclosures)
