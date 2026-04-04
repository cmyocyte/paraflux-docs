# Glossary

| Term | Definition |
|------|-----------|
| **Power Perp** | Perpetual futures where payoff = S^p / NORM. For p=2: squared exposure to spot price. |
| **sigma (sigma)** | Annualized volatility. 80% vol means the asset typically moves ~4.2% per day. |
| **sigma^2 (sigma squared)** | Variance -- sigma squared. If vol is 80%, variance is 0.64 (64%). This is what the funding rate tracks. |
| **WAD** | 1e18 fixed-point precision used on-chain. `1.0 = 1000000000000000000`. The SDK handles conversion. |
| **NORM** | Normalization constant (100). Index = spot^2 / 100. Makes the index price readable: HYPE at $35 -> index = $12.25. |
| **Index Price** | S^2/NORM -- the power perp mark price. HYPE at $35 -> index = $12.25. |
| **Funding Rate** | sigma^2 per year -- the continuous payment from longs to shorts for convexity. Based on realized variance. |
| **Realized Variance** | On-chain computed from `ln(S_new/S_old)^2` observations. The RealizedVolOracle's core output. |
| **Vol-of-Vol (VoV)** | Standard deviation of variance itself. High VoV = regime transition. |
| **Forward Variance** | Expected future variance derived from term structure. Contango/backwardation signal. |
| **Term Structure** | 1-day, 7-day, 30-day windowed variance averages. |
| **Cumulative Variance** | Monotonically increasing sum of annualized instant variances. The Uniswap TWAP for vol. |
| **LP Vault** | ERC-4626 vault. Counterparty to all traders. Earns fees, bears gamma, hedges delta. |
| **Anchor Vault** | Structured product (coming soon): short p=2, delta-neutral, earns sigma^2 funding. |
| **Surge Vault** | Structured product (coming soon): 50% LP Vault + 50% leveraged long. Bull exposure + yield buffer. |
| **CoreWriter** | HyperEVM precompile (0x3333...3) for sending transactions to HyperCore from EVM contracts. Used for delta hedging. |
| **HIP-3** | Hyperliquid Improvement Proposal 3: builder-deployed perpetuals. Source of GOLD, SILVER, OIL price feeds. |
| **BBO Precompile** | 0x080e -- returns best bid/offer for any asset including HIP-3 markets. |
| **Insurance Fund** | First-loss buffer absorbing bad debt. 5% of fees (immutable). |
| **Maintenance Margin** | 5% of notional. Position liquidated if equity drops below this. |
| **Liquidation Bonus** | 5% reward paid to liquidators from remaining collateral. |
| **Impact Fee** | Quadratic fee that scales with trade size relative to pool depth. LP protection. |
| **Kelly Criterion** | Optimal position sizing formula: `f = E[R] / Var[R]`. Half-Kelly used for safety. |
| **Dollar Gamma** | P&L from a 1% spot move. `gamma * S^2 / 200`. |
| **Theta** | Daily funding cost (longs) or income (shorts). `sigma^2 * notional / 365`. |
| **Partial Liquidation** | Only closing enough of a position to restore margin, instead of wiping the whole thing. |
| **szDecimals** | Size decimals for Hyperliquid assets. Determines price encoding in the oracle precompile. |
| **Pool-to-Peer** | Trading model where the LP vault is the counterparty to all trades. No order book needed. |
| **Pessimistic Index** | min(EMA, raw) for longs, max(EMA, raw) for shorts -- prevents stale EMA from blocking liquidation. |
