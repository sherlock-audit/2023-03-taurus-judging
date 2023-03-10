yixxas

medium

# Liquidation bot is vulnerable to sandwich attack

## Summary
Bot attempts to liquidate accounts that are no longer healthy, but hardcodes `_minExchangeRate` to be 0. `_minExchangeRate` is used as slippage check for collateral received when a position is liquidated hence it is an important protection from sandwich attacks.

## Vulnerability Detail
Liquidation bot is used to constantly scan for accounts that are no longer healthy via offchain. Because the liquidation bot will liquidate a position and accept any exchange rate, an adversary can perform a sandwich attack.

Adversary can inflate the price of collateral before this liquidate transaction, and force the liquidation bot to receive a much lower amount of collateral, and earning the difference.

## Impact
Adversary can perform a sandwich attack, arbitraging liquidation process at the cost of liquidation bot, resulting in it receiving less collateral than it should for liquidating a position.

## Code Snippet
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/LiquidationBot/LiquidationBot.sol#L89-L106

## Tool used

Manual Review

## Recommendation
Consider setting a reasonable `_minExchangeRate` for when calling liquidate via the bot, instead of setting it perpetually at 0.