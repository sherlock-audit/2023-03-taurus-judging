GimelSec

high

# If a user has less debt, the user may suffer loss of funds.

## Summary

In `BaseVault.updateReward`, it calculates the rewards that an account should receive. And if the account has debts, it uses the reward to pay off the debts. However, if the reward is more than the debts, the surplus is added back to drip feed. Thus, the accounts with better health could be punished.

## Vulnerability Detail

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L95
```solidity
    modifier updateReward(address _account) {
        …

            if (_tauEarned > 0) {
                uint256 _userDebt = userDetails[_account].debt;
                if (_tauEarned > _userDebt) {
                    // If user has earned more than enough TAU to pay off their debt, pay off debt and add surplus to drip feed
                    userDetails[_account].debt = 0;

                    _withholdTau(_tauEarned - _userDebt);
                    _tauEarned = _userDebt;
                } else {
                    // Pay off as much debt as possible
                    userDetails[_account].debt = _userDebt - _tauEarned;
                }

                emit TauEarned(_account, _tauEarned);
            }
        …
    }
```

Suggest that Alice has 100 debt and 10 collateral. And Bob has 100 debt and 5 collateral. Their `lastUpdatedRewardPerCollateral` are both 0. And `cumulativeTauRewardPerCollateral` is 20.

Alice should receive 200, but she only receives 100 to pay off the debt. And Bob can receive full reward to pay off his debt.

 
## Impact

The reward mechanism pushes the user to borrow more debt. because if you don't have enough debt. You cannot take the full reward. In other words, this mechanism punishes the accounts with better health.

## Code Snippet

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L95

## Tool used

Manual Review

## Recommendation

The surplus reward should be sent to the account.
