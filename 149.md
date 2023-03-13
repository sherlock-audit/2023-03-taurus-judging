HonorLt

medium

# Mint limit is not reduced when the Vault is burning TAU

## Summary

Upon burning TAU, it incorrectly updates the `currentMinted` when Vault is acting on behalf of users.

## Vulnerability Detail

When the burn of `TAU` is performed, it calls `_decreaseCurrentMinted` to reduce the limit of tokens minted by the Vault:
```solidity
    function _decreaseCurrentMinted(address account, uint256 amount) internal virtual {
        // If the burner is a vault, subtract burnt TAU from its currentMinted.
        // This has a few highly unimportant edge cases which can generally be rectified by increasing the relevant vault's mintLimit.
        uint256 accountMinted = currentMinted[account];
        if (accountMinted >= amount) {
            currentMinted[msg.sender] = accountMinted - amount;
        }
    }
```

The issue is that it subtracts `accountMinted` (which is `currentMinted[account]`) from `currentMinted[msg.sender]`. When the vault is burning tokens on behalf of the user, the `account` != `msg.sender` meaning the `currentMinted[account]` is 0, and thus the `currentMinted` of Vault will be reduced by 0 making it pretty useless.

Another issue is that users can transfer their `TAU` between accounts, and then `amount > accountMinted` will not be triggered.

## Impact

`currentMinted` is incorrectly decreased upon burning so vaults do not get more space to mint new tokens.

## Code Snippet

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/TAU.sol#L76-L83

## Tool used

Manual Review

## Recommendation
A simple solution would be to:
```solidity
     uint256 accountMinted = currentMinted[msg.sender];
```
But I suggest revisiting and rethinking this function altogether.
