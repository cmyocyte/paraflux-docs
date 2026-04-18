# FAQ

## General

### What is Paraflux?
A power perpetual protocol on HyperEVM. Trade squared exposure (S^2) across crypto, commodities, and equities. The LP vault is the automated, hedged counterparty -- no order book needed.

### What are power perps?
Perpetual futures where the payoff is a power of the spot price. With p=2, a 10% spot move produces roughly a 21% index move. You get convex exposure -- like an ATM straddle -- without strikes, expiry, or rolling.

### How is this different from regular perps?
Regular perps give you linear exposure. Power perps give you **squared** exposure -- gains accelerate on larger moves. Funding is based on realized variance (sigma^2) rather than mark-index price spreads.

### What markets are available?
**Launch (Phase 1):** BTC, ETH
**Phase 2:** HYPE, GOLD, SILVER
**Phase 3:** SOL, XRP, and real-world assets (TSLA, NVDA, OIL)

All markets share a single LP vault.

### Is Paraflux live?
Currently live on HyperEVM testnet (chain 998) at [app.paraflux.xyz](https://app.paraflux.xyz). Mainnet deployment (chain 999) is imminent.

---

## Trading

### What's the maximum leverage?
3x. Higher leverage tightens your liquidation price -- at 3x, a long gets liquidated after roughly a 31% adverse move.

### How much does it cost to trade?

| Fee | Rate | Description |
|-----|------|-------------|
| Trading fee | 0.30% of notional | Fixed per-trade fee |
| Impact fee | 0.3 x notional / poolSize | Quadratic fee protecting LPs from large orders |
| Protocol split | 20% of total fees | Directed to protocol treasury |
| Insurance split | 5% of LP fees | First-loss buffer (immutable) |

Effective fee formula: `effectiveFee = tradingFee + impactCoeff x notional / totalAssets`

### Can I get liquidated?
Yes. Liquidation triggers when your equity drops below 5% of notional (maintenance margin). A 5% liquidator bonus incentivizes keepers. The insurance fund absorbs any bad debt before it reaches LPs.

### How does funding work?
Longs pay shorts continuously -- per-second, on-chain. The rate is based on realized variance:

`dailyFunding ~ sigma^2 / 365 x notional`

At 80% annualized vol, that's roughly 0.18% of notional per day. Funding accrues automatically -- no manual claims or settlements.

### What is the funding rate based on?
Realized variance (sigma^2) from the on-chain RealizedVolOracle, adjusted by OI skew. When the vol oracle is set, funding uses live implied variance from ImpliedVolOracle. This is **realized** variance -- what actually happened -- not implied volatility.

---

## LP Vault

### How does the LP vault work?
Deposit USDC, receive ERC-4626 shares. The vault is the counterparty to all power perp trades. It earns:
- **75%** of all trading fees
- Funding income (when longs pay shorts)
- Net trader losses

Delta risk is automatically hedged via CoreWriter on Hyperliquid's native perp order book (80% hedge ratio, 5x leverage, grid orders).

### What are the risks of LPing?
The vault is short gamma -- it loses on large, sudden spot moves. Hedging mitigates ~96% of directional risk but doesn't eliminate it. In backtesting, the vault survived a simulated -60% crash with only -0.9% impact. Across 500 Monte Carlo paths, the vault showed 0% probability of loss.

### What's the expected return?
Walk-forward backtesting on 365 days of real data: **68% annual return, Sharpe ratio 5.1**. This includes grid hedging costs, keeper latency, withdrawal impact, and Greeks-based risk management.

### Can I withdraw anytime?
Yes, after a 1-hour cooldown following your last deposit. Withdrawals are processed instantly at the current share price.

### Fee split breakdown

| Recipient | Share | Source |
|-----------|-------|--------|
| LPs | 75% | Trading fees + funding |
| Protocol treasury | 20% | Trading fees |
| Insurance fund | 5% | LP fee portion (immutable) |

---

## Volatility Oracle

### Is the vol data really on-chain?
Yes. The RealizedVolOracle computes variance entirely on-chain using HyperEVM precompile price feeds. No Chainlink, no Pyth, no trusted relayers.

### Is this an implied vol oracle?
No. It measures **realized variance** -- what actually happened. The ImpliedVolOracle (when enabled) derives implied vol from funding utilization, but the core oracle is realized.

### How often is it updated?
Every 5 minutes (288 observations per day per asset) via permissionless keeper calls.

### What assets are tracked?
All launched markets receive real-time vol coverage.

---

## Technical

### What chain is this on?
HyperEVM -- testnet (chain 998), mainnet (chain 999). Sub-cent gas, ~2-second blocks.

### Are the contracts upgradeable?
No. All contracts are **immutable** -- once deployed, they cannot be modified. Security through simplicity and thorough testing.

### Is the code open source?
No. All deployed contracts are independently verifiable on-chain via the HyperEVM block explorer.

### How is the protocol secured?
Three internal audit passes covering access control, funding manipulation, liquidation logic, vault-market-maker interactions, and economic invariants. The OracleModule includes a circuit breaker that rejects price updates deviating beyond a configurable threshold from the EMA.

---

## Roadmap

### What about options, variance swaps, VIX token?
These are on the roadmap:
- **Phase 2**: Multi-asset expansion (HYPE, gold, silver), Anchor vault (delta-neutral yield)
- **Phase 3**: Variance swaps, structured vaults (Anchor + Surge), vol index perpetual
- **Phase 4**: 50+ markets, portfolio margining, LP shares as collateral on HyperLend

Power perps are the foundation -- additional instruments build on top of the same vault and oracle infrastructure.
