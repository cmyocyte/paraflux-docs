# Options Chain & Payoff Compiler

The Options tab provides a pro-grade interface for building custom options-like payoffs from power perps.

## Options Chain

The options chain displays a familiar strike-by-strike grid showing:

- **Calls and Puts** at each strike price
- **Price, Delta, Gamma** for each option
- **IV and Greeks** computed from the on-chain realized variance oracle
- **Bid/Ask spreads** implied by funding costs

Select any cell to add it to your position builder. Combine multiple strikes to create spreads, straddles, condors, or any multi-leg strategy.

## How It's Different

Traditional options require:
- Picking a strike price
- Choosing an expiry date
- Finding a counterparty at that strike/expiry
- Rolling positions before expiry

Power perp options have:
- **No expiry** -- hold as long as you want
- **No strike selection needed** -- the compiler handles it
- **Instant fill** from the LP vault
- **Daily funding** instead of upfront premium

## Payoff Compiler

The compiler takes a target payoff shape and decomposes it into executable power perp positions.

### How It Works

The **Basis Efficiency Theorem** proves that any smooth payoff curve can be approximated by a weighted combination of power perps. The compiler:

1. Takes your target payoff (from the options chain selection or a hand-drawn curve)
2. Finds the optimal weights for each power perp component
3. Computes total funding cost
4. Executes all legs atomically via the CompilerExecutor contract

### Supported Payoff Shapes

- **Standard options**: Calls, puts, spreads, straddles, strangles
- **Exotic structures**: Iron condors, butterflies, ratio spreads
- **Custom curves**: Draw any shape and the compiler builds it

### Execution

Multi-leg strategies execute atomically through the CompilerExecutor contract:
- All positions open in a single transaction
- The StrategyRegistry records the strategy on-chain
- Portfolio page groups positions by their parent strategy
- Close all legs at once or individually

## Risk Presets

The options chain offers three risk levels:
- **Safe**: Conservative strikes, lower leverage
- **Mid**: Moderate risk/reward
- **Degen**: Aggressive strikes, higher leverage

## Greeks

The SDK provides real-time Greeks computation:

| Greek | Description |
|-------|-------------|
| Delta | Sensitivity to spot price changes |
| Gamma | Rate of delta change (convexity) |
| Theta | Time decay (funding cost per day) |
| Vega | Sensitivity to volatility changes |

For power perps, gamma is always positive for longs -- that's the core value proposition. You're always buying convexity.
