yixxas

medium

# Liquidation bot liquidate does not work as intended when catching errors

## Summary
Only String errors will be caught, but this is a very limited subset of errors and the custom errors returned by `liquidate()` function in vaults will not be caught.

## Vulnerability Detail
Liquidation bot uses the try catch clause to attempt to not revert when an error occurs. I believe this is so that offchain implementation can continuously call this function without reverting.

```solidity
try vault.liquidate(_liqParams.accountAddr, newCalcAmt, 0) {
	return true;
} catch Error(string memory) {
	// providing safe exit
	return false;
}
```

However, in our current implementation, we are only catching String errors. This is a very limited subset of errors that will be caught. The errors that will occur in `liquidate` such as custom errors will not be caught. 

## Impact
Due to the incorrect implementation of error catching, the liquidation bot will fail to catch errors like it is meant to and revert anyway. This prevents smooth offchain integration.

## Code Snippet
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/LiquidationBot/LiquidationBot.sol#L100-L105

## Tool used

Manual Review

## Recommendation
Consider using a catch all instead.

```diff
try vault.liquidate(_liqParams.accountAddr, newCalcAmt, 0) {
	return true;
- } catch Error(string memory) {
+ } catch {
	// providing safe exit
	return false;
}
```
