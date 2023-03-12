ck

medium

# `_lastUpdateTimestamp` in price oracle is ineffective

## Summary

`_lastUpdateTimestamp` in price oracle is ineffective

## Vulnerability Detail

In the function `GLPPriceOracle::getLatestPrice` the `_lastUpdateTimestamp` is recorded as `block.timestamp`. 

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

This value is then used to determine if the  oracle response is corrupt in the function `_isCorruptOracleResponse`

```solidity
    function _isCorruptOracleResponse(OracleResponse memory _response) internal view returns (bool) {
        if (_response.lastUpdateTimestamp == 0 || _response.lastUpdateTimestamp + ORACLE_TIMEOUT < block.timestamp)
            return true;

        if (_response.currentPrice == 0) return true;

        return false;
    }
```

The fact that `_lastUpdateTimestamp` is set to `block.timestamp` in the same transaction flow means that the condition `_response.lastUpdateTimestamp + ORACLE_TIMEOUT < block.timestamp` is unlikely to ever return true.

The way this can be made effective as a possible as a staleness check is if the `lastUpdateTimestamp` was stored and checked the next time the price checked again.

## Impact

Lack of storing `lastUpdateTimestamp` means there's no price staleness check.

## Code Snippet

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Oracle/CustomPriceOracle/GLPPriceOracle.sol#L34-L45

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Oracle/Wrapper/CustomOracleWrapper.sol#L52-L59

## Tool used

Manual Review

## Recommendation

Store the `lastUpdateTimestamp` for use in subsequent price checks.