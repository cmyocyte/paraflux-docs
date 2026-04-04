# Strategies (Coming Soon)

> **Coming soon** -- one-click execution via CoreWriter. These strategies compose power perps with native Hyperliquid linear perps to create defined-risk payoffs.

## Overview

Each strategy is a recipe that combines a power perp position with a linear perp hedge on Hyperliquid's native order book. The protocol executes both legs atomically via CoreWriter. No options expertise required.

## Four Strategies

### Protect -- Downside Insurance

**What it does:** Long power perp + short linear perp at entry. Profits if the asset moves sharply in either direction, loses if it stays flat.

**Recipe:**
1. Open a long p=2 power perp (convex exposure)
2. Short the equivalent delta on Hyperliquid's native perp via CoreWriter
3. Net position is delta-neutral at entry, but gains convexity

**Payoff:** Profits on large moves (up or down). Bleeds funding (sigma^2) in calm markets.

**Cost:** sigma^2 per year minus linear perp funding received. At 80% vol, approximately 50-60% annualized net.

**Best for:** Portfolio insurance before catalysts, macro events, or uncertain periods.

**Example:** Protect $50K of HYPE exposure. If HYPE moves more than 5% in a day, the convexity gain exceeds the funding cost.

---

### Amplify -- Leveraged Convexity

**What it does:** Long power perp without a hedge. Pure squared exposure.

**Recipe:**
1. Open a long p=2 power perp at desired leverage (up to 3x)

**Payoff:** Amplified gains on moves in your direction. Amplified losses on moves against you. Plus funding cost.

**Cost:** sigma^2 per year in funding, plus any directional losses.

**Best for:** High-conviction directional bets where you want more than linear exposure.

**Example:** Go long HYPE^2 with $5K at 2x. If HYPE rallies 20%, your position gains approximately 40% (before funding).

---

### Straddle -- Trade Volatility

**What it does:** Long power perp + continuous delta hedge. Isolates pure gamma/vol exposure.

**Recipe:**
1. Open a long p=2 power perp
2. Continuously hedge delta via CoreWriter (rebalance every ~10 seconds)

**Payoff:** Profits when realized vol exceeds funding rate (sigma^2). Loses when markets are calm.

**Cost:** sigma^2 per year minus gamma scalping income. Net cost depends on realized vol vs funding.

**Best for:** Vol traders who think realized vol will exceed the current funding rate.

**Example:** Open a straddle on ETH^2. If ETH realizes 90% vol but funding is priced at 70% vol, you earn the 20% spread.

---

### Income -- Sell Volatility

**What it does:** Short power perp + long linear perp hedge. Earns funding income while staying delta-neutral.

**Recipe:**
1. Open a short p=2 power perp (receive funding)
2. Long the equivalent delta on Hyperliquid's native perp via CoreWriter
3. Rebalance hedge as delta drifts

**Payoff:** Earns sigma^2 per year in funding. Loses on large moves (gamma drag exceeds income).

**Cost:** Net positive in calm markets. Gamma drag in volatile markets.

**Best for:** Yield seekers who believe vol will stay at or below current levels. This is the Anchor vault strategy executed manually.

**Example:** Short HYPE^2, delta-hedged. At 80% vol, earn approximately 64% annualized minus hedge cost (~6%).

## Risk Disclosures

- **Protect and Straddle** bleed funding in calm markets. Your maximum loss is your collateral.
- **Amplify** has full directional risk plus funding cost. Liquidation is possible.
- **Income** has unlimited loss potential on large moves. A 20%+ daily move can wipe out weeks of income.
- **All strategies** require active hedge rebalancing. Hedge lag during volatile periods can cause slippage.
