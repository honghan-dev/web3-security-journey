
## Other notable findings

## Table of content

## [H01. Boost modifications apply retroactively to reward shares, allowing boosting of past timespans](https://cantina.xyz/code/990ce947-05da-443e-b397-be38a65f0bff/findings/1)

### Note

- Issue 1: Protocol calculates rewards using current balance to calculate rewards for past time periods.
- Issue 2: veICE is a decaying asset, so using current (decayed) balance retroactively reduces rewards that should have been calculated with the higher historical balance.

- **How to spot this bug pattern:**

- `Trace reward calculation flows` - Follow the data from user action → reward calculation → balance updates
- `Test time-sensitive scenarios` - Ask "What if I boost mid-period?" or "What if my position decays before claiming?"

## [M01. Stableswap pools can have price changes even when swaps are killed](https://cantina.xyz/code/990ce947-05da-443e-b397-be38a65f0bff/findings/645)

### Note

- Issue 1: Kill switch only blocks explicit swap() function but doesn't block functions that perform internal swaps
- Issue 2: ``withdraw_one_coin()` and `remove_liquidity_imbalance()` can change token ratios even when swaps are "killed", defeating emergency protections

- **How to spot this bug pattern**

- `Test bypass scenarios` - Ask "If function X is blocked, how else can I achieve the same result?"
- `Look for inconsistent protection patterns` - Check if security controls are applied uniformly across all relevant functions
