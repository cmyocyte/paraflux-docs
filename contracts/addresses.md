# Deployed Addresses

## HyperEVM Mainnet (Chain 999)

### Core Protocol

| Contract | Address |
|----------|---------|
| PowerPerpRouter | TBD — updated after mainnet deploy |
| PositionEngine | TBD — updated after mainnet deploy |
| FundingEngine | TBD — updated after mainnet deploy |
| LP Vault | TBD — updated after mainnet deploy |
| LiquidationEngine | TBD — updated after mainnet deploy |
| InsuranceFund | TBD — updated after mainnet deploy |

### Realized Vol Oracles

| Asset | ImpliedVolOracle |
|-------|-----------------|
| **BTC** | `0x38d64bfaF0140B9678675a6c6922CEDADAFA059E` |
| **ETH** | `0x9b4cC61A104F7DF3aA0B8F8BF1c21062A7C211F3` |
| **SOL** | `0x7f25526576c40758Ef70D42e82166d37b4e2E2E1` |
| **HYPE** | `0xf4d699b21d482732222412297cFc652f70e62BC0` |
| **XYZ100** | `0x592d1Cd3d0c20E8EE41F0Cfe2F081346d0DE0B30` |
| **TSLA** | `0x97416F15e47A2FA91D7cA858a6Ba707F38657748` |
| **Gold** | `0x21e8FE4A97c2DA1DfAc7543B44eCC59EE5433Cff` |
| **Silver** | `0xd89aF3011eFE37dAc812d9De10b17a6E4725d387` |
| **EUR/USD** | `0x206d1366c108C90536d330Befc73B50870698466` |

### Tokens

| Token | Address |
|-------|---------|
| USDC (Mainnet) | `0xb88339CB7199b77E23DB6E890353E22632Ba630f` |

### HyperEVM Precompiles

System addresses built into HyperEVM — not deployed contracts.

| Precompile | Address | Purpose |
|-----------|---------|---------|
| Oracle (core perps) | `0x0807` | Spot price for BTC, ETH, SOL, HYPE (core perps) |
| BBO (all assets) | `0x080e` | Best bid/offer for all assets including HIP-3 (Gold, Silver, XYZ100, TSLA, EUR/USD) |
| CoreWriter | `0x3333...3` | Send orders from EVM to Hyperliquid L1 order book |

Note: Core protocol contract addresses will be published after mainnet deployment is complete. Check [app.paraflux.xyz](https://app.paraflux.xyz) for latest.
