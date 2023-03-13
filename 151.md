tsvetanovv

medium

# Unsafe ERC20.transfer()

## Summary

The ERC20.transfer() and ERC20.transferFrom() functions return a boolean value indicating success. This parameter needs to be checked for success. Some tokens do not revert if the transfer failed but return false instead.

## Vulnerability Detail

Using unsafe ERC20 methods can revert the transaction for certain tokens.

## Impact

Tokens that don't actually perform the transfer and return false are still counted as a correct transfer and tokens that don't correctly implement the latest EIP20 spec will be unusable in the protocol as they revert the transaction because of the missing return value.

## Code Snippet
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/LiquidationBot/LiquidationBot.sol#L108-L114

```solidity
    function withdrawLiqRewards(address _token, uint256 _amount) external onlyMultisig {
        IERC20 collToken = IERC20(_token);
        if (_amount > collToken.balanceOf(address(this))) revert insufficientFunds();
        collToken.transfer(msg.sender, _amount); 

        emit CollateralWithdrawn(msg.sender, _amount);
    }
```

## Tool used

Manual Review

## Recommendation

Recommend using OpenZeppelin's SafeERC20 versions with the safeTransfer and safeTransferFrom functions that handle the return value check as well as non-standard-compliant tokens.