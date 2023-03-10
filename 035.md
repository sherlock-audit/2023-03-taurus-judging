cducrest-brainbot

high

# Protocol assumes 18 decimals collateral

## Summary

Multiple calculation are done with the amount of collateral token that assume the collateral token has 18 decimals. Currently the only handled collateral (staked GLP) uses 18 decimals. However, if other tokens are added in the future, the protocol may be broken.

## Vulnerability Detail

TauMath.sol calculates the collateral ratio (coll * price / debt) as such:

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Libs/TauMath.sol#L18

It accounts for the price decimals, and the debt decimals (TAU is 18 decimals) by multiplying by Constants.precision (`1e18`) and dividing by `10**priceDecimals`. The result is a number with decimal precision corresponding to the decimals of `_coll`.

This `collRatio` is later used and compared to values such as `MIN_COL_RATIO`, `MAX_LIQ_COLL_RATIO`, `LIQUIDATION_SURCHARGE`, or `MAX_LIQ_DISCOUNT` which are all expressed in `1e18`.

Secondly, in `TauDripFeed` the extra reward is calculated as:
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/TauDripFeed.sol#L91

This once again assumes 18 decimals for the collateral. This error is cancelled out when the vault calculates the user reward with:
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L90

However, if the number of decimals for the collateral is higher than 18 decimals, the rounding of `_extraRewardPerCollateral` may bring the rewards for users to 0.

For example: `_tokensToDisburse = 100 e18`, `_currentCollateral = 1_000_000 e33`, then `_extraRewardPerCollateral = 0` and no reward will be distributed.

## Impact

Collateral ratio cannot be properly computed (or used) if the collateral token does not use 18 decimals. If it uses more decimals, users will appear way more collateralised than they are. If it uses less decimals, users will appear way less collateralised. The result is that users will be able to withdraw way more TAU than normally able or they will be in a liquidatable position before they should.

It may be impossible to distribute rewards if collateral token uses more than 18 decimals.

## Code Snippet

Constants.precision is `1e18`:

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Libs/Constants.sol#L24

MIN_COL_RATIO is expressed in 1e18:

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L57

TauDripFeed uses `1e18` once again in calculation with collateral amount:

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/TauDripFeed.sol#L91

The calculation for maxRepay is also impacted:

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L251-L253

The calculation for expected liquidation collateral is also impacted:

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L367

## Tool used

Manual Review

## Recommendation

Account for the collateral decimals in the calculation instead of using `Constants.PRECISION`.