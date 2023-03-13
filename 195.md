0x52

medium

# Adversary can call distributeTauRewards with amount = 0 to purposefully decrease reward rate

## Summary

When distributeTauRewards is called with amount = 0 it doesn't withhold any extra tokens but resets the distribution time which result in reward rate decreasing. Imagine 100 TAU is being vested. After 12 hours an adversary can call distributeTauRewards with amount = 0. This will distributed 50 TAU and cause the other 50 TAU to be vested over the next 24 hours, effectively halving the reward rate.

## Vulnerability Detail

See summary

## Impact

Reward rate can be maliciously lowered

## Code Snippet

[TauDripFeed.sol#L51-L60](https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/TauDripFeed.sol#L51-L60)

## Tool used

Manual Review

## Recommendation

Only allow rewards to be distributed if it increases reward rate