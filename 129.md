bytes032

high

# Front running updatePrice can be exploited to generate rewards even when users are liquidatable and avoid getting liquidated when the time comes.

## Summary

Front running updatePrice can be exploited to generate rewards even when users are liquidatable and avoid getting liquidated when the time comes.

## Vulnerability Detail

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Oracle/CustomPriceOracle/CustomPriceOracle.sol

The `CustomPriceOracle` allows trusted nodes to update the price of an asset by calling the `updatePrice` function. The updated price is stored in the `currentPrice` variable and the previous price is stored in the `lastPrice` variable. The `lastUpdateTimestamp` variable records the timestamp of the last update. The contract exposes the `getLatestPrice` function that returns the current and previous prices, the last update timestamp, and the number of decimals for the asset. Only trusted nodes registered with the contract owner can update the price using the `updatePrice` function.

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Oracle/Wrapper/CustomOracleWrapper.sol

Then, the price is fetched by the OraclePriceManager, going through the CustomOracleWrapper implementation. Its  `getExternalPrice` function is used to retrieve the current price of an underlying asset from the registered custom oracle implementation. 

The function first calls the `_getResponses` internal function, which retrieves the current price, last price, last update timestamp, and number of decimals for the asset from the custom oracle implementation. The `_isCorruptOracleResponse` internal function checks if the response is valid and returns `true` if the response is corrupt. The `getExternalPrice` function returns the current price, number of decimals, and success status of the oracle response.

`_isCorruptOracleResponse` first checks if the `lastUpdateTimestamp` field of the `OracleResponse` struct is zero or if it is older than the `ORACLE_TIMEOUT` value (4 hours) from the current block timestamp. If either of these conditions is true, it means that the custom oracle implementation has not beenupdated at all, and the response is considered corrupt.

The vulnerability arises because the `_isCorruptOracleResponse` function checks if the `lastUpdateTimestamp` field of the `OracleResponse` struct is older than the `ORACLE_TIMEOUT` value of 4 hours. This time delay is too long in the crypto market and can cause significant problems for the protocol. For instance, a token in the top 50 can drop more than 10-20% in an hour.


The long timeout opens the possibility of front-running the trusted node when updating the price, to avoid getting liquidated and generate rewards at the same time. This can occur when a user deposits collateral and borrows debt, and the price of collateral decreases, causing the account to be liquidatable. If the price is stale, the trusted node will execute a transaction to update the price. However, the user can front run the update price transaction using `modifyPosition` to repay their debt and generate rewards at the same (because `modifyPosition` calls `updateRewards`) time, avoiding being liquidated. This can be repeated as often as desired, causing users to generate rewards even if they are liquidatable and avoid getting liquidated when the time comes.

Consider this case.
1. Alice deposits 1.21e18 collateral and borrows 1e18 debt, her account is currently health.
2. Price of collateral decreases by 10% and her account should be liquidatable and she shouldn't be generating rewards at this point, but cannot be liquidated, because the price is stale.
3. The trusted node executes a transaction to update the price 
4. Alice knows that after the update she is going to be liquidatable, so she front runs the update price transaction using `modifyPosition` and repays her debt so that she cannot be liquidated and also generates additional rewards along the way.


## Impact

This vulnerability can be exploited to generate rewards even when users are liquidatable, and avoid getting liquidated when the time comes.

## Code Snippet

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L278-L284
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Oracle/Wrapper/CustomOracleWrapper.sol
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Oracle/CustomPriceOracle/CustomPriceOracle.sol

## Tool used
Manual review

## Recommendation
As far as I know, the the reason we didn't go with Chainlink oracle is price feed being not available on Chainlink for GLP. However, the `GLPPriceOracle` doesn't even make use of `_response.lastUpdateTimestamp + ORACLE_TIMEOUT < block.timestamp`, because you are returning `return (price, price, block.timestamp, decimals);` when its `getLatestPrice` is called.

As a result, its pretty much only the CustomPriceOracle contract who is going to be affected by this check. My first recommendation is to ditch the contract and make the wrapper composable with Chailink price feeds, but if you insist on using CustomPriceOracle, you absolutely must reduce the timeout to no longer than 5 minutes.