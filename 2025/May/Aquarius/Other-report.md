
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

## [M02. Reward Emissions Accumulate Even When Pool Has No Liquidity, Causing Irrecoverable Token Lock](https://cantina.xyz/code/990ce947-05da-443e-b397-be38a65f0bff/findings/516)

### Note

- Admin transfer token to protocol to incentivize users.

- Issue 1: Protocol continues emitting and accounting for `rewards (accumulated)` even when no users are eligible to receive them `(working_supply == 0)`. (Protocol distributes the token despite no user to incentivize)
- Issue 2: Ghost tokens (emitted during zero-liquidity periods) are counted as "distributed" in accounting, preventing admin recovery of genuinely unused rewards

- `Root Cause:`: Protocol distributes the token despite no user to incentivize. Admin then cannot reclaim the undistributed token, because of this continuous distribution without user working supply check.

- **How to spot bug**

- `Trace emission vs distribution flows` - Check if reward emission logic runs independently of actual distribution capability.

- `Test zero-state scenarios` - Ask "What happens during periods with no eligible users/liquidity?"

## [M03. Stableswap calculator fails due to 0 value swaps](https://cantina.xyz/code/990ce947-05da-443e-b397-be38a65f0bff/findings/381)

### Note

- Issue: Calculator's iterative liquidity estimation reduces input amounts to zero, then calls Newton-Raphson method with `dx = 0`, causing precision errors and underflow panic.

- Impact: One malicious pool can break reward allocation for ALL pools sharing the same token pair, causing system-wide DoS of reward distribution

- **How to spot**

- `Validate inputs BEFORE function calls` - Always check for edge cases (zero, negative, extreme values) before calling mathematical functions, not after
