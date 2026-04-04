# Next-Gen Products (Roadmap)

> **These products are on the roadmap and not yet available.** Each is backtested with real data. Deployment follows after core power perp launch is stable.

## Gamma Scalping Vault

**The opposite of Anchor.** Long p=2 + continuous delta-hedge. Profits when markets are volatile.

**Adaptive:** Automatically scales exposure based on the current vol regime. When vol is cheap, it deploys more. When vol is expensive, it scales down and earns lending yield on idle capital.

Multi-asset version runs across BTC, ETH, HYPE, and SOL simultaneously for smoother returns.

## Tail Hedge Vault

**Crash insurance.** Pay a little every day, get paid massively when everything breaks.

- Deploys ~8% of capital into leveraged long p=2 positions
- Remaining 92% earns lending yield
- Auto-rolls positions before they get liquidated from funding bleed
- Ramps up deployment after drawdowns (buys convexity when it is cheapest)

## Principal-Protected Notes (PPN)

**Deposit USDC. Get at least ~98% back. Keep the upside.**

- 90-93% of capital goes to lending (guaranteed yield)
- 7-10% goes to long p=2 (convex upside)
- 30-day epochs: at the end, lending yield covers the risk leg's worst case

The gateway product for conservative users, DAOs, and treasuries.

## Risk Summary

Every product can lose money:

| Product | Good Scenario | Bad Scenario | Worst Case |
|---------|--------------|-------------|-----------|
| **Gamma Scalp** | Volatile markets: big daily moves | Steady trending market | -7% annual bleed |
| **Tail Hedge** | Crash/spike events | Extended calm period | -12% annual bleed (offset by 5% lending) |
| **PPN** | Asset rallies during epoch | Asset crashes > lending yield | ~-2% per epoch (floored) |
