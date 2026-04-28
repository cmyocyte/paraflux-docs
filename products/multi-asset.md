# Markets

Paraflux offers squared perpetual markets across crypto, commodities, and equities through a unified LP vault.

## Launch Roadmap

### Phase 1 -- Launch
| Market | Asset | Power | Oracle |
|--------|-------|-------|--------|
| BTC^2 | Bitcoin | S^2 | Core perp (0x0807) |
| ETH^2 | Ethereum | S^2 | Core perp (0x0807) |

### Phase 2 -- Multi-Asset
| Market | Asset | Power | Oracle |
|--------|-------|-------|--------|
| HYPE^2 | Hyperliquid | S^2 | Core perp (0x0807) |
| SOL^2 | Solana | S^2 | Core perp (0x0807) |
| GOLD^2 | Gold | S^2 | HIP-3 BBO (0x080e) |
| SILVER^2 | Silver | S^2 | HIP-3 BBO (0x080e) |

### Phase 3 -- Expansion
| Market | Asset | Power | Oracle |
|--------|-------|-------|--------|
| XYZ100^2 | Nasdaq | S^2 | HIP-3 BBO (0x080e) |
| TSLA^2 | Tesla | S^2 | HIP-3 BBO (0x080e) |
| EUR/USD^2 | Euro/Dollar | S^2 | HIP-3 BBO (0x080e) |

Vol oracles are already deployed on mainnet for all 9 assets. Power perp markets roll out in phases.

## Oracle Infrastructure

### Core Perps (0x0807)
Crypto markets use the Hyperliquid core perp oracle precompile. These are the most liquid markets on Hyperliquid with deep order books and tight spreads.

### HIP-3 BBO (0x080e)
Real-world assets use the HIP-3 Best Bid/Offer precompile. This reads mid-prices from builder-deployed perpetual exchanges (trade.xyz, Felix, etc.). Spread quality is excellent -- GOLD at 0.002%, SILVER at 0.001%.

### Price Processing
All oracle prices pass through the OracleModule which applies:
- **EMA smoothing** to reduce manipulation risk
- **Multi-step catch-up** for stale price recovery
- **Circuit breaker** rejecting updates that deviate beyond a configurable threshold

## Shared Vault Architecture

All markets share a single LP vault. This means:
- LPs get diversified exposure across all markets
- Adding a new market doesn't fragment liquidity
- Each market has its own OracleModule, FundingEngine, and PositionEngine
- The Router resolves the correct engines per market via `vault.markets(marketId)`

## Adding New Markets

New markets deploy via MarketFactory in a single transaction. Cost: ~$0.65 per new market. Each deployment creates fresh Oracle, Funding, Position, and Liquidation engines wired to the shared vault.

## Cross-Asset Strategies

With multiple markets live, traders can build cross-asset strategies:
- Long HYPE^2 + Short BTC^2 for relative vol exposure
- Short GOLD^2 as a macro hedge alongside crypto longs
- Long ETH^2 + Short SOL^2 for L1 rotation
