Ruhum

high

# User can prevent liquidations by frontrunning the tx and slightly increasing their collateral

## Summary
User can prevent liquidations by frontrunning the tx and decreasing their debt so that the liquidation transaction reverts.

## Vulnerability Detail
In the liquidation transaction, the caller has to specify the amount of debt they want to liquidate, `_debtAmount`. The maximum value for that parameter is the total amount of debt the user holds:
```sol
    function liquidate(
        address _account,
        uint256 _debtAmount,
        uint256 _minExchangeRate
    ) external onlyLiquidator whenNotPaused updateReward(_account) returns (bool) {
        if (_debtAmount == 0) revert wrongLiquidationAmount();

        UserDetails memory accDetails = userDetails[_account];

        // Since Taurus accounts' debt continuously decreases, liquidators may pass in an arbitrarily large number in order to
        // request to liquidate the entire account.
        if (_debtAmount > accDetails.debt) {
            _debtAmount = accDetails.debt;
        }

        // Get total fee charged to the user for this liquidation. Collateral equal to (liquidated taurus debt value * feeMultiplier) will be deducted from the user's account.
        // This call reverts if the account is healthy or if the liquidation amount is too large.
        (uint256 collateralToLiquidate, uint256 liquidationSurcharge) = _calcLiquidation(
            accDetails.collateral,
            accDetails.debt,
            _debtAmount
        );
```

In `_calcLiquidation()`, the contract determines how much collateral to liquidate when `_debtAmount` is paid by the caller. In that function, there's a check that reverts if the caller tries to liquidate more than they are allowed to depending on the position's health.
```sol
    function _calcLiquidation(
        uint256 _accountCollateral,
        uint256 _accountDebt,
        uint256 _debtToLiquidate
    ) internal view returns (uint256 collateralToLiquidate, uint256 liquidationSurcharge) {
        // ... 
        
        // Revert if requested liquidation amount is greater than allowed
        if (
            _debtToLiquidate >
            _getMaxLiquidation(_accountCollateral, _accountDebt, price, decimals, totalLiquidationDiscount)
        ) revert wrongLiquidationAmount();
```

The goal is to get that if-clause to evaluate to `true` so that the transaction reverts. To modify your position's health you have two possibilities: either you increase your collateral or decrease your debt. So instead of preventing the liquidation by pushing your position to a healthy state, you only modify it slightly so that the caller's liquidation transaction reverts.

Given that Alice has:
- 100 TAU debt
- 100 Collateral (price = $1 so that collateralization rate is 1)
Her position can be liquidated. The max value is:
```sol
    function _getMaxLiquidation(
        uint256 _collateral,
        uint256 _debt,
        uint256 _price,
        uint8 _decimals,
        uint256 _liquidationDiscount
    ) internal pure returns (uint256 maxRepay) {
        // Formula to find the liquidation amount is as follows
        // [(collateral * price) - (liqDiscount * liqAmount)] / (debt - liqAmount) = max liq ratio
        // Therefore
        // liqAmount = [(max liq ratio * debt) - (collateral * price)] / (max liq ratio - liqDiscount)
        maxRepay =
            ((MAX_LIQ_COLL_RATIO * _debt) - ((_collateral * _price * Constants.PRECISION) / (10 ** _decimals))) /
            (MAX_LIQ_COLL_RATIO - _liquidationDiscount);

        // Liquidators cannot repay more than the account's debt
        if (maxRepay > _debt) {
            maxRepay = _debt;
        }

        return maxRepay;
    }
```
$(1.3e18 * 100e18 - (100e18 * 1e18 * 1e18) / 1e18) / 1.3e18 = 23.07e18$ (leave out liquidation discount for easier math)

The liquidator will probably use the maximum amount they can liquidate and call `liquidate()` with `23.07e18`. Alice frontruns the liquidator's transaction and increases the collateral by `1`. That will change the max liquidation amount to:
$(1.3e18 * 100e18 - 101e18 * 1e18) / 1.3e18 = 22.3e18$.

That will cause `_calcLiquidation()` to revert because `23.07e18 > 22.3e18`.

The actual amount of collateral to add or debt to decrease depends on the liquidation transaction. But, generally, you would expect the liquidator to liquidate as much as possible. Thus, you only have to slightly move the position to cause their transaction to revert

## Impact
User can prevent liquidations by slightly modifying their position without putting it at a healthy state.

## Code Snippet
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L342-L363

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L396-L416

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L240

## Tool used

Manual Review

## Recommendation
In `_calcLiquidation()` the function shouldn't revert if `_debtToLiqudiate > _getMaxLiquidation()`. Instead, just continue with the value `_getMaxLiquidation()` returns. 