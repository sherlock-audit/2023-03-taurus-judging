ahmedovv

medium

# Wrong modifier on liquidate function.

## Summary

```onlyKeeper``` modifier used.

## Vulnerability Detail

## Impact

on [L86](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/LiquidationBot/LiquidationBot.sol#L86) Clearly says that the function ```liquidate``` can be invoked by anyone, but ```onlyKeeper``` [modifier](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/LiquidationBot/LiquidationBot.sol#L89) is attached to the modifier which restricts users.

For example on [L43](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/SwapHandler.sol#L43) of [SwapHandler.sol](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/SwapHandler.sol) Says that the function [swapForTau](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/SwapHandler.sol#L45) is callable only by registered keepers and the modifier is attached correctly

## Code Snippet

```solidity
    /// @dev This function can be invoked by any one to liquidate the account and returns true if succeeds, false otherwise
    /// note This function takes liquidation params as input in which the amount can be
    ///      provided either inclusive/exclusive of offset
    function liquidate(LiqParams memory _liqParams) external onlyKeeper returns (bool) {
        BaseVault vault = BaseVault(_liqParams.vaultAddress);
        uint256 newCalcAmt = _liqParams.amount;

        if (_liqParams.offset) {
            // Calculate the new amount by deducting the offset
            newCalcAmt -= ((newCalcAmt * percOffset) / OFFSET_PRECISION);
        }

        if (newCalcAmt > tau.balanceOf(address(this))) revert insufficientFunds();

        try vault.liquidate(_liqParams.accountAddr, newCalcAmt, 0) {
            return true;
        } catch Error(string memory) {
            // providing safe exit
            return false;
        }
    }
```

## Tool used

Manual Review

## Recommendation

Remove ```onlyKeeper``` modifier or fix function comment.
