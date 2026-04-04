# Python SDK

Python SDK for the Paraflux Power Perpetual Protocol on HyperEVM. Built on [web3.py](https://web3py.readthedocs.io/) for EVM interaction.

## Install

```bash
pip install paraflux
```

Or for development:

```bash
pip install -e ".[dev]"
```

## Quick Start

```python
from paraflux import ParaFluxClient
from paraflux.types import ParaFluxConfig

client = ParaFluxClient(ParaFluxConfig(
    chain_id=999,
    rpc_url='https://rpc.hyperliquid.xyz/evm',
    private_key='0x...',    # omit for read-only
))

# Read market data
market = client.get_market_data('HYPE')
print(f'HYPE spot: {market.hype_spot}')
print(f'Funding rate: {market.funding_rate}')

# Open a 2x long with $500 collateral
from paraflux.types import OpenParams
tx = client.open_position(OpenParams(
    asset='HYPE',
    is_long=True,
    usd_amount=500,
    leverage=2,
))
client.wait_for_tx(tx)
```

## API Reference

### Read Methods (no signer needed)

| Method | Returns | Description |
|--------|---------|-------------|
| `get_market_data(asset?)` | `MarketData` | Index, spot, EMA, funding rate, OI, skew |
| `get_vault_state()` | `VaultState` | TVL, share price, utilization, available liquidity |
| `get_position(trader, asset, is_long)` | `PositionDetails \| None` | Full position with P&L, health, liq price |
| `estimate_fee(params)` | `FeeQuote` | Preview base + impact fees before trading |
| `is_liquidatable(trader, asset, is_long)` | `(bool, int)` | Health check |
| `get_liquidation_price(trader, asset, is_long)` | `float` | Spot price threshold |
| `get_insurance_state()` | `InsuranceState` | Fund balance, target, fill ratio |

### Protocol Analytics

| Method | Returns | Description |
|--------|---------|-------------|
| `get_protocol_stats()` | `ProtocolStats` | All-in-one: collateral, PnL, fees, bad debt, positions, funding |
| `get_cached_ema_index()` | `int` | Cached EMA index (WAD) |
| `get_last_oracle_update_block()` | `int` | Block number of last oracle update |
| `get_last_deposit_time(depositor)` | `int` | Unix timestamp of last deposit (cooldown tracking) |
| `get_cumulative_funding()` | `int` | Current funding accumulator (WAD) |
| `get_total_collateral_locked()` | `int` | Total collateral locked (USDC 6-dec) |

### Write Methods (require `private_key`)

| Method | Returns | Description |
|--------|---------|-------------|
| `open_position(params)` | `str` (tx hash) | Open long or short (handles approval, slippage) |
| `close_position(asset, is_long, min_payout_usd=0)` | `str` | Close a position |
| `add_collateral(asset, is_long, usd_amount)` | `str` | Add collateral |
| `remove_collateral(asset, is_long, usd_amount)` | `str` | Remove collateral (must stay above 20% margin) |
| `deposit(usd_amount)` | `str` | Deposit USDC into LP vault |
| `withdraw(usd_amount)` | `str` | Withdraw USDC from LP vault (1hr cooldown) |
| `redeem(shares)` | `str` | Redeem LP shares for USDC |
| `liquidate(trader, asset, is_long)` | `str` | Liquidate unhealthy position (permissionless, 5% bonus) |

### Admin Methods (require owner key)

| Method | Returns | Description |
|--------|---------|-------------|
| `pause()` | `str` | Pause the Router (no new trades) |
| `unpause()` | `str` | Unpause the Router |
| `set_trading_fee(fee_wad)` | `str` | Update base trading fee rate |
| `set_impact_coefficient(coeff_wad)` | `str` | Update impact fee coefficient |
| `set_protocol_fee_bps(bps)` | `str` | Update protocol fee share |
| `set_min_position_size(size_wad)` | `str` | Update minimum position size |
| `set_max_position_utilization_bps(bps)` | `str` | Update per-position utilization cap |

### Math Utilities

```python
from paraflux import math

math.wad_to_number(index)                # WAD int -> float
math.number_to_usdc(100.5)               # float -> 6-dec USDC int
math.compute_index(spot_wad)             # spot^2 / NORM
math.compute_pnl(size, entry, current, is_long)
math.estimate_fee(notional, tvl, trading_fee, impact_coeff)
math.compute_health(collateral, pnl, funding_owed, notional)
math.forecast_funding_rate(long_oi, short_oi, additional_size, is_long, base_vol, skew_sens)
```

## Example Bots

### Arb Bot

Trades index vs fair value (spot^2 / NORM). Opens when deviation exceeds threshold, closes when it narrows.

```bash
PRIVATE_KEY=0x... THRESHOLD_BPS=30 POSITION_USD=1000 python examples/arb_bot.py
```

### Funding Harvester

Positions on the underpopulated side to collect funding payments.

```bash
PRIVATE_KEY=0x... MIN_FUNDING_ANNUAL=0.10 POSITION_USD=5000 python examples/funding_harvester.py
```

Both bots support `DRY_RUN=true` to log what they *would* do without submitting transactions.

## Contract Addresses

Contract addresses are configured per-chain. Override via config:

```python
from paraflux.types import ParaFluxConfig, ContractAddresses

client = ParaFluxClient(ParaFluxConfig(
    chain_id=999,
    rpc_url='...',
    contracts=ContractAddresses(
        router='0x...',
        oracle='0x...',
        funding_engine='0x...',
        position_engine='0x...',
        lp_vault='0x...',
        liquidation_engine='0x...',
        insurance_fund='0x...',
        usdc='0xb88339CB7199b77E23DB6E890353E22632Ba630f',
    ),
))
```
