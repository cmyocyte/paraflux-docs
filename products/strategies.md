# Strategies

The Strategy Builder lets you replicate options-like payoffs using power perps. No strikes to pick, no expiry to manage, no options infrastructure. Just select a strategy, set your parameters, and execute.

## How It Works

Each strategy combines power perp positions (and optionally Hyperliquid native perp hedges) to create a specific payoff shape. The system computes the payoff curve, liquidation prices, and funding costs in real time as you adjust parameters.

## Available Strategies

### Directional

| Strategy | Bias | Side | Description |
|----------|------|------|-------------|
| **Amplify** | Bullish | Long | Squared upside. Gains accelerate with the move -- a 10% move gives 21%, a 30% move gives 69%. Pure convexity. |
| **Call** | Bullish | Long | Pick a strike above spot. Below it you pay only funding. Above it, convex profits accelerate. |
| **Put** | Bearish | Short | Pick a strike below spot. Profits accelerate the further price drops. |
| **Spread** | Bullish | Long | Two strikes. Max loss capped below lower, max gain capped above upper. Cheaper than naked call. |

### Hedge

| Strategy | Bias | Side | Description |
|----------|------|------|-------------|
| **Protect** | Neutral | Long | Already hold the asset? Set a floor -- losses stop there, upside stays open. Portfolio insurance. |
| **Collar** | Neutral | Long | Floor + cap. Cap generates income that offsets floor cost. Can be zero-cost. |
| **Covered Call** | Neutral | Short | Hold asset + short power perp. Earn daily funding. Trade-off: miss big rallies past cap. |

### Yield

| Strategy | Bias | Side | Description |
|----------|------|------|-------------|
| **Income** | Neutral | Short | Sell vol, collect daily funding. Steady income in calm markets. Risk: big moves hurt. |
| **Range** | Neutral | Both | Earn while price stays in your range. Breakout = losses. Tighter range = more income, more risk. |

### Advanced

| Strategy | Bias | Side | Description |
|----------|------|------|-------------|
| **Straddle** | Neutral | Both | Delta-neutral at entry. Profit from big moves in either direction. You're long volatility. |
| **Draw** | Any | Any | Draw any payoff shape. The compiler decomposes it into executable power perp positions. |

## Risk Presets

Every strategy offers three quick presets:

| Preset | Size | Leverage | Best For |
|--------|------|----------|----------|
| Safe | $500 | 1x | Learning, small exposure |
| Mid | $1,000 | 2x | Active trading |
| Degen | $5,000 | 3x | High conviction |

## Strategy Parameters

Each strategy has adjustable parameters that change the payoff shape:

- **Floor %** (Protect, Collar): How far below spot your losses are capped
- **Cap %** (Collar, Covered Call): How far above spot your gains are capped
- **Strike %** (Call, Put): Distance from spot where the payoff kicks in
- **Range bounds** (Range, Spread): Lower and upper boundaries

The payoff chart updates in real time as you drag sliders. Hover over the curve to see exact P&L at any price point.

## Liquidation on the Chart

The payoff chart shows liquidation zones:
- **Red shaded area**: Region where your position would be liquidated
- **LIQ marker**: Exact price where liquidation triggers
- **Curve cutoff**: Beyond the liq price, the position is closed -- the curve ends

Higher leverage moves the liq price closer to entry. The chart shows this visually.

## Executing a Strategy

1. **Pick a strategy** from the sidebar
2. **Select asset** (HYPE, BTC, ETH, etc.)
3. **Set size and leverage** (or use a preset)
4. **Adjust parameters** (floor, cap, strike)
5. **Review** the payoff chart, stats, and funding cost
6. **Execute** -- the position opens instantly against the LP vault

Strategies that combine multiple legs (Collar, Spread, Straddle) are executed atomically -- all positions open in a single transaction.

## Strategy Guide

First-time users see a guided walkthrough that highlights each section of the interface. You can re-trigger the guide anytime via the "Strategy Guide" button in the sidebar.
