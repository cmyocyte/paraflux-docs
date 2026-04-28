# Vol Index Trading (Roadmap)

> **This product is on the roadmap and not yet available.** The design is complete and contracts are built, but vol index trading will launch after the core power perp protocol is live and stable.

## What It Will Be

A perpetual contract whose price tracks a weighted volatility index across multiple assets. Long vol index = buy fear. Short vol index = earn carry when markets are calm.

## How the Index Will Work

The index will read volatility data from external sources -- specifically **Volmex BVIV** (Bitcoin Implied Volatility Index) and **EVIV** (Ethereum Implied Volatility Index). These are market-derived implied volatility measures, not our own realized vol oracle output.

We are honest about this distinction: our RealizedVolOracle computes what happened in the past. Implied vol (what the market expects) requires bilateral price discovery, which our pool-to-peer model does not produce. Volmex provides this data from options markets.

## Planned Features

- **Perpetual contract** on the composite vol index
- **4x max leverage** (vol is less volatile than squared payoffs)
- **Contango funding** (~7% annualized + skew adjustment)
- **Partial liquidation** to preserve positions during spikes
- **Circuit breaker** (30% max move per block)
- **TWAP smoothing** for stable index readings

## When to Expect It

After the core power perp launch is stable across all 9 markets and the Volmex BVIV integration is complete. Follow our socials for announcements.

## Further Reading

- [Realized Vol Oracle](../protocol/vol-oracle.md) -- what we have today
- [LP Vault](../products/lp-vault.md) -- earn yield now while waiting for vol index trading
