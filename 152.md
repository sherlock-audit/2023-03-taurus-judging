GimelSec

medium

# `LiquidationBot.fetchUnhealthyAccounts` may revert when offset if too big

## Summary

`LiquidationBot.fetchUnhealthyAccounts` can be used to fetch an unhealthy account array. But `offset` could be too big to cause a revert. 

## Vulnerability Detail

`LiquidationBot.fetchUnhealthyAccounts` would call `vault.getUsers` and fetch an account array. If `offset` is too big, it could suffer an out-of-gas revert. 
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/LiquidationBot/LiquidationBot.sol#L64
```solidity
    function fetchUnhealthyAccounts(
        uint256 _startIndex,
        address _vaultAddress
    ) external view returns (address[] memory unhealthyAccounts) {
        BaseVault vault = BaseVault(_vaultAddress);
        address[] memory accounts = vault.getUsers(_startIndex, _startIndex + offset);
        uint256 j;

        for (uint256 i; i < accounts.length; ++i) {
            if (!vault.getAccountHealth(accounts[i])) j++;
        }

        unhealthyAccounts = new address[](j);
        j = 0;

        for (uint256 i; i < accounts.length; i++) {
            if (!vault.getAccountHealth(accounts[i])) unhealthyAccounts[j++] = accounts[i];
        }

        return unhealthyAccounts;
    }
```

## Impact

Since `offset` can only be set by the multisig. If the offset is too big. It needs to wait for the  multisig to fix the problem.

## Code Snippet

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/LiquidationBot/LiquidationBot.sol#L64

## Tool used

Manual Review

## Recommendation

Make `offset` an argument of `fetchUnhealthyAccounts` instead of a state variable. 
