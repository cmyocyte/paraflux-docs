# FAQ

## General

**What is Paraflux?**

Power perpetual protocol on HyperEVM. Trade squared exposure (S²) across crypto, commodities, and equities. LP vault is the automated, hedged counterparty — no order book needed. Build any option payoff shape via the strategy builder and payoff compiler.

**What are power perps?**

Perpetual futures where the payoff is a power of the spot price. With p=2, a 10% spot move produces roughly a 21% index move. Convex exposure — like an ATM straddle — without strikes, expiry, or rolling.

**How is this different from regular perps?**

Regular perps give linear exposure. Power perps give **squared** exposure — gains accelerate on larger moves. Funding is based on realized variance (σ²) rather than mark-index price spreads.

**What about options, variance swaps, vol index?**

The strategy builder already lets you replicate any option payoff shape (calls, puts, spreads, straddles, collars) using compiled power perp positions. Variance swaps and a vol index token are on the roadmap — they build on the same vault and oracle infrastructure once power perp markets are live and stable.

**What markets are available?**

**Launch (Phase 1):** BTC, ETH
**Phase 2:** HYPE, GOLD, SILVER
**Phase 3:** SOL and additional real-world assets

All markets share a single LP vault.

---

## Trading

**What's the maximum leverage?**

3x. Higher leverage tightens liquidation price — at 3x, a long gets liquidated after roughly a 19% adverse move.

**How much does it cost to trade?**

| Fee | Rate | Description |
|-----|------|-------------|
| Router base fee | 0.15% (15 bps) | Fixed per-trade fee |
| Compiled strategy fee | 0.12% (12 bps) | Flat fee via CompilerExecutor |
| Impact fee | 0.3 × notional / poolSize | Quadratic, protecting LPs |

Multi-leg strategies via CompilerExecutor cost 12 bps total, not 15 bps per leg — a 73% discount for a 3-leg strategy.

**Can I get liquidated?**

Yes. Liquidation triggers when equity drops below 15% of notional (maintenance margin). A 5% liquidator bonus incentivizes keepers. The insurance fund absorbs any bad debt before it reaches LPs.

**How does funding work?**

Longs pay shorts continuously — per-second, on-chain. Rate based on realized variance:

`dailyFunding ≈ σ² / 365 × notional`

At 80% annualized vol, roughly $17.53 per $10K notional per day. Funding accrues automatically — no manual claims or settlements.

**What happens when I'm wrong?**

Unlike options, your position doesn't expire worthless. A compiled call below the strike evaluates to zero P&L — your collateral minus accumulated funding is returned. The tradeoff: you must decide when to close. Set a trigger order at entry to auto-close if funding exceeds your threshold.

---

## LP Vault

**How does the LP vault work?**

Deposit USDC, receive ERC-4626 shares. Vault is counterparty to all power perp trades across all markets. Earns:
- 70% of all trading fees
- Funding income (when longs pay shorts)
- Net trader losses

Delta risk automatically hedged via CoreWriter on Hyperliquid's native perp order book.

**What are the risks of LPing?**

Vault is short gamma — loses on large, sudden spot moves. Hedging mitigates ~96% of directional risk but doesn't eliminate it. In backtesting, vault survived simulated -60% crash with only -0.9% impact. Across 500 Monte Carlo paths, 0% probability of total loss.

**What's the expected return?**

Walk-forward backtest across 14 markets on 365 days of real data: **+141.4% annual return, Sharpe 12.08, -2.5% max drawdown**. Even at the most conservative scenario (25% vol/TVL), the vault returns +53.6% with Sharpe 10.36. Past performance doesn't guarantee future results.

**Can I withdraw anytime?**

Yes, after 1-hour cooldown following last deposit. Withdrawals process instantly at current share price. A utilization-based exit fee applies when the vault is heavily utilized — 0% when utilization is below 50%, ramping up to 3% max at full utilization. Most withdrawals pay zero exit fee. See [Fee Structure](../protocol/fees.md) for details.

**Fee split breakdown:**

| Recipient | Share |
|-----------|-------|
| LPs | 70% |
| Protocol treasury | 25% |
| Insurance fund | 5% (immutable) |

---

## Volatility Oracle

**Is the vol data really on-chain?**

Yes. The RealizedVolOracle computes variance entirely on-chain using HyperEVM precompile price feeds. No external oracle dependencies, no trusted relayers.

**Is this an implied vol oracle?**

No. It measures **realized variance** — what actually happened. The ImpliedVolOracle (when enabled) derives implied vol from funding utilization, but the core oracle is realized.

**How often is it updated?**

Every 5 minutes (288 observations per day per asset) via permissionless keeper calls. The protocol funds keeper operations.

**What assets are tracked?**

9 assets across 5 classes: BTC, ETH, SOL, HYPE (crypto), XYZ100/Nasdaq, TSLA (equities), Gold, Silver (metals), EUR/USD (forex).

---

## Technical

**What chain is this on?**

HyperEVM — testnet (chain 998), mainnet (chain 999). Sub-cent gas, ~2-second blocks.

**Are the contracts upgradeable?**

No. All contracts are **immutable** — once deployed, cannot be modified.

**What can the admin change?**

Fee parameters (trading fee rate, impact coefficient, protocol fee share), position limits (min size, max utilization per position), hedge parameters (via keeper), and emergency pause. The admin **cannot** change the insurance fee (immutable at deploy), upgrade contracts, or modify the funding rate formula.

**Is the code open source?**

BSL 1.1. Contracts in `contracts/src/`, SDKs in `sdk/typescript/` and `sdk/python/`.

**How is the protocol secured?**

Four internal audit passes covering access control, funding manipulation, liquidation logic, vault interactions, and economic invariants. 700+ tests (unit, fuzz, integration, invariant). OracleModule includes circuit breaker rejecting price updates deviating beyond configurable threshold from EMA.
