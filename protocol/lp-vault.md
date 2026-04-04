# LP Vault Architecture

> This page covers how the LP Vault works under the hood. For how to deposit and earn yield, see [LP Vault (Earn Yield)](../products/lp-vault.md).

The LP Vault is an ERC-4626 vault that serves as the counterparty to **all power perpetual traders** across 10 markets. The vault earns trading fees, funding income, and net trader losses. Delta exposure is hedged via CoreWriter.

## Architecture

```
Traders → Router → PositionEngine → LPVault (collateral lock/unlock)
                                   → LPVault (fee income)
                                   → LPVault (P&L settlement)
```

The vault's `totalAssets()` reflects:
- USDC balance (free liquidity)
- Locked collateral (backing open positions)
- Unrealized aggregate P&L (O(1) computation)

## Key Functions

| Function | Access | Description |
|----------|--------|-------------|
| `deposit(assets, receiver)` | Public | Deposit USDC, receive LP shares |
| `withdraw(assets, receiver, owner)` | Public (after cooldown) | Withdraw USDC |
| `redeem(shares, receiver, owner)` | Public (after cooldown) | Redeem LP shares |
| `totalAssets()` | View | Total vault value in USDC (6-dec) |
| `convertToShares(assets)` | View | USDC → shares conversion |
| `convertToAssets(shares)` | View | Shares → USDC conversion |
| `lockCollateral(amount)` | Router only | Lock USDC for new position |
| `unlockCollateral(amount)` | Router/Liquidation | Release collateral on close |
| `settlePnL(pnl)` | Router/Liquidation | Adjust vault for trader P&L |
| `absorbBadDebt(amount)` | Liquidation only | Last resort: vault absorbs bad debt |

## Share Price

```
sharePrice = totalAssets / totalSupply
```

Share price increases when the vault earns (fees, funding, trader losses). Decreases when the vault loses (large moves exceeding hedge, bad debt absorption).

## Cooldown

1-hour withdrawal cooldown after deposit. Prevents flash loan attacks where an attacker deposits, manipulates the market, and withdraws in one transaction. Cooldown propagates on share transfers.

## Authorization

Both Router AND LiquidationEngine are authorized via `onlyAuthorized` modifier. Set via `setRouter()` and `setLiquidationEngine()` one-time wiring.

## Virtual Shares

Uses OpenZeppelin v5.x ERC-4626 with virtual shares to prevent inflation attacks (donation attacks where an attacker inflates share price by donating to the vault).
