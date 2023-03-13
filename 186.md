GimelSec

medium

# `GLPPriceOracle.getLatestPrice` doesn't return correct `lastPrice`

## Summary

`GLPPriceOracle.getLatestPrice` should return ` (currentPrice, lastPrice, lastUpdateTimestamp, decimals)`. But it return the wrong `lastPrice`.

## Vulnerability Detail

`GLPPriceOracle.getLatestPrice` returns the wrong `lastPrice`. `currentPrice` and `lastPrice` share the same value in `GLPPriceOracle.getLatestPrice`.
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Oracle/CustomPriceOracle/GLPPriceOracle.sol#L44
```solidity
    function getLatestPrice(
        bytes calldata _maximise
    )
        external
        view
        override
        returns (uint256 _currentPrice, uint256 _lastPrice, uint256 _lastUpdateTimestamp, uint8 _decimals)
    {
        bool isMax = _maximise.parse32BytesToBool();
        uint256 price = glpManager.getPrice(isMax);
        return (price, price, block.timestamp, decimals);
    }
```

## Impact

Since `lastPrice` is not used in the protocol. It might be used in the future. It should return the correct value.

## Code Snippet

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Oracle/CustomPriceOracle/GLPPriceOracle.sol#L44
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Oracle/Wrapper/CustomOracleWrapper.sol#L45


## Tool used

Manual Review

## Recommendation

`GLPPriceOracle` should record the old price.
