GimelSec

high

# `swap()` will be reverted if `path` has more tokens.

## Summary

`swap()` will be reverted if `path` has more tokens, the keepers will not be able to successfully call `swapForTau()`.

## Vulnerability Detail

In `test/SwapAdapters/00_UniswapSwapAdapter.ts`:

```javascript
    // Get generic swap parameters
    const basicSwapParams = buildUniswapSwapAdapterData(
      ["0xyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy", "0xzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz"],
      [3000],
      testDepositAmount,
      expectedReturnAmount,
      0,
    ).swapData;
```

We will get:

```text
000000000000000000000000000000000000000000000000000000024f49cbca
0000000000000000000000000000000000000000000000056bc75e2d63100000
0000000000000000000000000000000000000000000000055de6a779bbac0000
0000000000000000000000000000000000000000000000000000000000000080
000000000000000000000000000000000000000000000000000000000000002b
yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy000bb8zzzzzzzzzzzzzzzzzz
zzzzzzzzzzzzzzzzzzzzzz000000000000000000000000000000000000000000
```

Then the `swapOutputToken` is `_swapData[length - 41:length - 21]`.

But if we have more tokens in path:
```javascript
    const basicSwapParams = buildUniswapSwapAdapterData(
      ["0xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx", "0xyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy", "0xzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz"],
      [3000, 3000],
      testDepositAmount,
      expectedReturnAmount,
      0,
    ).swapData;
```

```text
000000000000000000000000000000000000000000000000000000024f49cbca
0000000000000000000000000000000000000000000000056bc75e2d63100000
0000000000000000000000000000000000000000000000055de6a779bbac0000
0000000000000000000000000000000000000000000000000000000000000080
0000000000000000000000000000000000000000000000000000000000000042
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx000bb8yyyyyyyyyyyyyyyyyy
yyyyyyyyyyyyyyyyyyyyyy000bb8zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz
zzzz000000000000000000000000000000000000000000000000000000000000
```

`swapOutputToken` is `_swapData[length - 50:length - 30]`, the `swap()` function will be reverted.


## Impact

The keepers will not be able to successfully call `SwapHandler.swapForTau()`. Someone will get a reverted transaction if they misuse `UniswapSwapAdapter`.

## Code Snippet

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/SwapAdapters/UniswapSwapAdapter.sol#L30

## Tool used

Manual Review

## Recommendation

Limit the swap pools, or check if the balance of `_outputToken` should exceed `_amountOutMinimum`.
