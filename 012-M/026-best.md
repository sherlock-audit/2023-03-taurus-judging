roguereddwarf

medium

# BaseVault.sol: getUsersDetailsInRange and getUsers function do not include value for end index

## Summary
The `BaseVault.getUsers` and `BaseVault.getUsersDetailsInRange` function both take a start index and an end index and return user data.

However they will not write a value to the last position in the array that is returned.

This means that the last spot in the array has the default values which is 0.

This will cause problems for any protocols that integrate with Taurus as some of the user data might not be accounted for.
We can already see that this causes problems in the `LiquidationBot.fetchUnhealthyAccounts` function.

## Vulnerability Detail
I'll explain the issue based on the `BaseVault.getUsers` function since it is the same mistake in the `BaseVault.getUsersDetailsInRange` function.

Say we call `getUsers` with `_start=5` and `_end=10`.

The function creates a new array with length `10 - 5 + 1 = 6` since the data at the `_end` index is supposed to be included:
```solidity
users = new address[](_end - _start + 1);
```

Then the output array is populated with the appropriate values:
```solidity
for (uint256 i = _start; i < _end; ++i) {
            users[i - _start] = userAddresses[i];
        }
```

The issue is that the loop is only run as long as `ì < _end`. So `i < 10`.
This means that the last user address at index 10 is not written to the output array even though the output array is supposed to contain it and has a place reserved for it.

Any downstream components will read the 0 value at this position and think there is no user.

This already happens in the `LiquidationBot.fetchUnhealthyAccounts` function.

The last account that is fetched is the 0 address and it will never be marked as unhealthy because it cannot have any debt.

## Impact
The user data at the `end` index is not written to the output array. This leads to downstream problems because the user data at the `end` index will not be considered.

## Code Snippet
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L173-L181

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L160-L168

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/LiquidationBot/LiquidationBot.sol#L64-L84

## Tool used
Manual Review

## Recommendation
Simply write one more value to the output array in both functions:
```diff
diff --git a/taurus-contracts/contracts/Vault/BaseVault.sol b/taurus-contracts/contracts/Vault/BaseVault.sol
index 33f87f0..b95909b 100644
--- a/taurus-contracts/contracts/Vault/BaseVault.sol
+++ b/taurus-contracts/contracts/Vault/BaseVault.sol
@@ -162,7 +162,7 @@ abstract contract BaseVault is SwapHandler, UUPSUpgradeable {
 
         users = new UserDetails[](_end - _start + 1);
 
-        for (uint i = _start; i < _end; ++i) {
+        for (uint i = _start; i < _end + 1; ++i) {
             users[i - _start] = userDetails[userAddresses[i]];
         }
     }
@@ -175,7 +175,7 @@ abstract contract BaseVault is SwapHandler, UUPSUpgradeable {
 
         users = new address[](_end - _start + 1);
 
-        for (uint256 i = _start; i < _end; ++i) {
+        for (uint256 i = _start; i < _end + 1; ++i) {
             users[i - _start] = userAddresses[i];
         }
     }
```