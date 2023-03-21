bytes032

medium

# approveTokens must approve 0 first

## Summary
Some tokens (like USDT) do not work when changing the allowance from an existing non-zero allowance value.They must first be approved by zero and then the actual allowance must be approved.

## Vulnerability Detail

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/SwapAdapters/UniswapSwapAdapter.sol#L53-L55

```solidity
    function approveTokens(address _tokenIn) external {
        IERC20(_tokenIn).approve(swapRouter, type(uint256).max);
    }
```

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/LiquidationBot/LiquidationBot.sol#L59-L61

```solidity
    function approveTokens(address _tokenIn, address _vault) external onlyMultisig {
        IERC20(_tokenIn).approve(address(_vault), type(uint256).max);
    }
```

## Impact
This vulnerability could result in an inability to transfer tokens due to the requirement of first setting the allowance to zero, then re-approving the allowance to the desired value.


## Code Snippet
https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/LiquidationBot/LiquidationBot.sol#L59-L61

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/SwapAdapters/UniswapSwapAdapter.sol#L53-L55

## Tool used
Manual review

## Recommendation
Add an approve(0) before approving;

```diff
    function approveTokens(address _tokenIn) external {
+       IERC20(_tokenIn).approve(0, type(uint256).max);
        IERC20(_tokenIn).approve(swapRouter, type(uint256).max);
    }
```