# FAQ

## General

**What is Paraflux?**
Paraflux is a power perpetual protocol on HyperEVM. You trade squared exposure (S^2) to 10 markets -- 7 crypto and 3 real-world assets. The LP vault is the counterparty to every trade, hedged automatically.

**What are power perps?**
Power perpetuals are perpetual futures where the payoff is a power of the spot price. For p=2, your position tracks spot^2 -- giving convex exposure similar to a perpetual ATM straddle, but with no strikes, no expiry, and no fragmentation. One instrument per market.

**How is this different from regular perps?**
Regular perps give linear exposure (1x spot). Power perps give squared exposure -- moves are amplified in both directions. A 10% spot move produces roughly a 20% index move. You also pay funding based on variance (sigma^2) instead of mark vs index spread.

**What markets are available?**
10 markets at launch: HYPE^2, BTC^2, ETH^2, SOL^2, XRP^2, SUI^2, DOGE^2 (crypto) + GOLD^2, SILVER^2, OIL^2 (commodities/energy via HIP-3).

**Is Paraflux live?**
Yes. Mainnet on HyperEVM (chain 999). Trade at [app.paraflux.xyz](https://app.paraflux.xyz).

## Trading

**What's the maximum leverage?**
3x. This is enforced on-chain. Higher leverage increases liquidation risk.

**How much does it cost to trade?**
Base fee is 7 bps (0.07%). Impact fee scales with trade size relative to pool depth. A $20K trade in a $3M pool pays approximately 30 bps effective.

**Can I get liquidated?**
Yes, if your position health drops below 1.0 (equity < 15% of notional). A 5% liquidation bonus incentivizes liquidators. The insurance fund absorbs bad debt first.

**How does funding work?**
Longs pay shorts sigma^2 per year, continuously. At 80% vol, that is 64% annually. Funding is deducted from your margin every second. The rate is based on realized variance from the on-chain oracle, plus a skew adjustment based on OI imbalance.

**What is the funding rate based on?**
Realized variance (sigma^2) from the on-chain RealizedVolOracle, not implied volatility. This is an honest design choice -- we measure what actually happened, not what a market consensus predicts.

## LP Vault

**How does the LP vault work?**
Deposit USDC, receive ERC-4626 LP shares. The vault is the counterparty to all power perp trades. It earns 75% of trading fees, funding income from leveraged traders, and net trader losses. Delta exposure is hedged via CoreWriter on Hyperliquid's native order book.

**What are the risks of LPing?**
The vault is short gamma -- it loses on large spot moves. The CoreWriter hedge removes approximately 80% of directional risk, but residual gamma exposure remains. Fee income is the primary income source that offsets gamma drag.

**What's the expected return?**
Backtest shows 26.6% annual with Sharpe 3.25 on 365 days of real data. Past performance does not guarantee future results.

**Can I withdraw anytime?**
Yes, after a 1-hour cooldown from your last deposit. This prevents flash loan attacks.

## Volatility Oracle

**Is the vol data really on-chain?**
Yes. The RealizedVolOracle contract computes realized variance from price observations entirely on-chain. No off-chain computation, no trusted relayers. Any smart contract can read it via a `view` call for free.

**Is this an implied vol oracle?**
No. It measures realized variance -- what actually happened. It does not produce implied volatility. We are upfront about this distinction. For implied vol data, we plan to integrate Volmex BVIV in a future release.

**How often is it updated?**
Every 5 minutes (288 observations per day per asset). A keeper calls `recordObservation()` which is permissionless -- anyone can call it.

**What assets are tracked?**
All 10 launch markets: HYPE, BTC, ETH, SOL, XRP, SUI, DOGE, GOLD, SILVER, OIL.

## Technical

**What chain is this on?**
HyperEVM -- the EVM layer of Hyperliquid. Mainnet is chain 999, testnet is chain 998. Sub-cent gas, 2-second blocks.

**Are the contracts upgradeable?**
No. All contracts are immutable. Security through simplicity.

**Where can I find the code?**
BSL 1.1 (open source). Contracts in `contracts/src/`, SDKs in `sdk/typescript/` and `sdk/python/`.

**How many tests?**
700+ tests covering unit, fuzz (80K+ runs), integration, and invariant testing.

## Roadmap

**What about options, variance swaps, VIX token?**
These are on the roadmap but not yet available. Power perps are the core product at launch. Options and variance swaps will be introduced when there is sufficient volume and market maker interest. Vol index trading will integrate Volmex BVIV for implied vol data. See the [Roadmap section](../SUMMARY.md) in the docs for details.
