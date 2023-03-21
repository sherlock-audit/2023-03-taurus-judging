roguereddwarf

high

# Missing input validation for _rewardProportion parameter allows keeper to escalate his privileges and pay back all loans

## Summary
According to the Contest page and discussion with the sponsor, the role of a `keeper` is to perform liquidations and to swap yield token for `TAU` using the `SwapHandler.swapForTau` function:
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/SwapHandler.sol#L45-L52

They are also able to choose how much yield token to swap and what the proportion of the resulting TAU is that is distributed to users vs. not distributed in order to erase bad debt.

So a `keeper` is not trusted to perform any actions that go beyond swapping yield / performing liquidations.

However there is a missing input validation for the `_rewardProportion` parameter in the `SwapHandler.swapForTau` function.
This allows a keeper to "erase" all debt of users.
So users can withdraw their collateral without paying any of the debt.

## Vulnerability Detail
By looking at the code we can see that `_rewardProportion` is used to determine the amount of `TAU` that `_withholdTau` is called with:
[Link](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/SwapHandler.sol#L91)
```solidity
_withholdTau((tauReturned * _rewardProportion) / Constants.PERCENT_PRECISION);
```

Any value of `_rewardProportion` greater than `1e18` means that more `TAU` will be distributed to users than has been burnt (aka erasing debt).

It is easy to see how the `keeper` can chose the number so big that `_withholdTau` is called with a value close to `type(uint256).max` which will certainly be enough to erase all debt.

## Impact
A `keeper` can escalate his privileges and erase all debt.
This means that `TAU` will not be backed by any collateral anymore and will be worthless.

## Code Snippet
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/SwapHandler.sol#L45-L101

## Tool used
Manual Review

## Recommendation
I discussed this issue with the sponsor and it is intended that the `keeper` role can freely chose the value of the `_rewardProportion` parameter within the `[0,1e18]` range, i.e. 0%-100%.

Therefore the fix is to simply check that `_rewardProportion` is not bigger than `1e18`:
```diff
diff --git a/taurus-contracts/contracts/Vault/SwapHandler.sol b/taurus-contracts/contracts/Vault/SwapHandler.sol
index c04e3a4..ab5064b 100644
--- a/taurus-contracts/contracts/Vault/SwapHandler.sol
+++ b/taurus-contracts/contracts/Vault/SwapHandler.sol
@@ -59,6 +59,10 @@ abstract contract SwapHandler is FeeMapping, TauDripFeed {
             revert zeroAmount();
         }
 
+        if (_rewardProportion > Constants.PERCENT_PRECISION) [
+            revert invalidRewardProportion();
+        ]
+
         // Get and validate swap adapter address
         address swapAdapterAddress = SwapAdapterRegistry(controller).swapAdapters(_swapAdapterHash);
         if (swapAdapterAddress == address(0)) {
```