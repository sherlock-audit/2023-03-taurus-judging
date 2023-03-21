KingNFT

medium

# ````emergencyClosePosition()```` is missing the ````whenPaused```` modifier

## Summary
The ````emergencyClosePosition()```` is designed to be called when the contract is paused, as the call may cause users' financial loss. But it is missing to append the ````whenPaused```` modifier. By calling it, users may suffer unwarranted losses when the contract is not paused.

## Vulnerability Detail
```solidity
File: contracts\Vault\BaseVault.sol
221:     /**
222:      * @dev Function allowing a user to automatically close their position.
223:      * Note that this function is available even when the contract is paused.
224:      * Note that since this function does not call updateReward, it should only be used when the contract is paused.
225:      *
226:      */
227:     function emergencyClosePosition() external { // @audit not paused
228:         _modifyPosition(msg.sender, userDetails[msg.sender].collateral, userDetails[msg.sender].debt, false, false);
229:     }

```

## Impact
Users may suffer unexpected losses when the contract is not paused

## Code Snippet
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L227

## Tool used

Manual Review

## Recommendation
append the ````whenPaused```` modifier.