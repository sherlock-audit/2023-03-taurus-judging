Chinmay

medium

# Oracle timeout will cause liquidations to fail

## Summary
Liquidations, repayments etc. depend on value returned by getCollPrice(). In certain cases, if oracle fails, this function will revert causing grief for users wanting to repay/should be liquidated. 

## Vulnerability Detail
The implementation of CustomOracleWrapper.sol shows that if the price returned by oracle nodes is older than the oracle timeout (currently 4 hours) , it reverts and thus doesn't allow liquidation/repayments. This will cause grief for the vault/user respectively. The debt will keep accruing on user's position and his position may go more and more underwater before the Oracle resumes.

This is amplified by the fact that the protocol uses custom data provided by its own node setup and we dont know about the number and coverage of these nodes. 

## Impact
In case of failed repayments, the user's position may keep going underwater increasing the final debt he has to pay. In case of failed liquidations, it adds bad debt for the whole vault and cause a liquidity crunch for withdrawls. 

## Code Snippet
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Oracle/Wrapper/CustomOracleWrapper.sol#L53

## Tool used

Manual Review

## Recommendation
Once it is discovered by getCollPrice function that Oracle has timed out, the vault should pause all user positions( at the current amounts) that try to liquidate/repay and prevent accrual of debts for no mistake of theirs. 