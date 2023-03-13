0x52

medium

# getAccountHealth will return false for some healthy accounts

## Summary

getAccountHealth will return false for account with a collRatio = MIN_COL_RATIO but this account isn't actually unhealthy and a call to liquidate it will revert.

## Vulnerability Detail

[BaseVault.sol#L123-L127](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L123-L127)

    function getAccountHealth(address _account) public view returns (bool) {
        uint256 ratio = _getCollRatio(_account);

        return (ratio > MIN_COL_RATIO);
    }

When calculating the health of an account the current collateral ratio is compared to MIN_COL_RATIO. This means that account in which the collateral ratio = MIN_COL_RATIO will return as unhealthy.

[BaseVault.sol#L432-L435](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L432-L435)

    function _calcLiquidationDiscount(uint256 _accountHealth) internal pure returns (uint256 liquidationDiscount) {
        if (_accountHealth >= MIN_COL_RATIO) {
            revert cannotLiquidateHealthyAccount();
        }

The issues is that if this account that was flagged as "unhealthy" is attempted to be liquidated, the liquidation call will revert when calculating the liquidation discount. The result is that an account that has been flagged as unhealthy can't actually be liquidated.

## Impact

Accounts flagged as "unhealthy" can't actually be liquidated.

## Code Snippet

[BaseVault.sol#L123-L127](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L123-L127)

## Tool used

Manual Review

## Recommendation

Modify getAccountHealth so that it doesn't flag accounts as unhealthy when collateral ratio = MIN_COL_RATIO

      function getAccountHealth(address _account) public view returns (bool) {
          uint256 ratio = _getCollRatio(_account);
  
    -     return (ratio > MIN_COL_RATIO);
    +     return (ratio >= MIN_COL_RATIO);
      }