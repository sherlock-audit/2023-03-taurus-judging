0x52

medium

# baseVault#emergencyClosePosition permanently breaks award accounting

## Summary

When using emergency close it should store the unclaimed rewards to be distributed later since the user can't claim them later

## Vulnerability Detail

[BaseVault.sol#L227-L229](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L227-L229)

    function emergencyClosePosition() external {
        _modifyPosition(msg.sender, userDetails[msg.sender].collateral, userDetails[msg.sender].debt, false, false);
    }

`emergencyClosePosition` allows a user to completely close their position without updating their debt. This allows users to force close their position even when the contract is paused. Because it bypasses `updateReward`, all unclaimed TAU will be silently lost and becomes unrecoverable.

[BaseVault.sol#L78-L118](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L78-L118)

    modifier updateReward(address _account) {
        // Disburse available yield from the drip feed
        _disburseTau();

        // If user has collateral, pay down their debt and recycle surplus rewards back into the tauDripFeed.
        uint256 _userCollateral = userDetails[_account].collateral;
        if (_userCollateral > 0) {
            ...
        } else {
            // If this is a new user, add them to the userAddresses array to keep track of them.
            if (userDetails[_account].startTimestamp == 0) {
                userAddresses.push(_account);
                userDetails[_account].startTimestamp = block.timestamp;
            }
        }

        // Update user lastUpdatedRewardPerCollateral
        userDetails[_account].lastUpdatedRewardPerCollateral = cumulativeTauRewardPerCollateral;
        _;
    }

Since `emergencyClosePosition` completely closes out the entire position the next time a user interacts with the contract it will bypass all the reward logic and update their `lastUpdatedRewardPerCollateral`. This means that all accrued rewards before calling `emergencyClosePosition` can never be claimed.

Example:

A user's `rewardPerCollateral` is 1e18 and the `cumulativeRewardPerCollateral` is 1.1e18 with a collateral of 100e18. This would mean that the user is owed 10e18 in rewards. They call `emergencyClosePosition` and now that 10e18 is lost. Since this value is never used to offset any debt it is essentially lost.

## Impact

TAU will be silently lost and becomes unrecoverable.

## Code Snippet

[BaseVault.sol#L78-L118](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L78-L118)

## Tool used

Manual Review

## Recommendation

Lost rewards should be tracked distributed to the rest of the vault.