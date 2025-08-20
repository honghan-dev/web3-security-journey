
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

- `Validate inputs BEFORE function calls` - Always check for edge cases (zero, negative, extreme values) before calling mathematical functions, not after.

## [Stable pool rewards can get overinflated if the tokens have different decimals](https://cantina.xyz/code/990ce947-05da-443e-b397-be38a65f0bff/findings/271)

### Note

- Issue: Doesn't handle token decimal differences while pool contract does, causing inconsistent mathematical operations on incompatible number scales.
- Impact: Stable pools with mixed-decimal tokens get artificially inflated liquidity scores, receiving disproportionately higher rewards at expense of standard pools

Root Cause:
rust// Pool contract (CORRECT):
let x = xp.get(i).unwrap() + dx * precision_mul.get(i).unwrap();  // ✅ Scales input
let dy = (xp.get(j).unwrap() - y - 1) / precision_mul.get(j).unwrap();  // ✅ Unscales output

- **How to Spot This Bug Pattern:**

- `Check decimal handling consistency` - When multiple contracts perform similar math, ensure they handle token decimals identically
- `Look for precision multiplier usage` - If one contract uses precision_mul for decimal normalization, all related contracts should too

## [M. Permanent Pool Lockout via Arithmetic Overflow in Reward Distribution Logic](https://cantina.xyz/code/990ce947-05da-443e-b397-be38a65f0bff/findings/251)

### Note

- Issue: `340282366920938463463374607431768211455` = `3*10^38`, Reward system performs unchecked arithmetic operations that can overflow `u128` limits when calculating per-share rewards and user rewards.

- Impact: Once triggered, causes permanent pool lockout where all functions (claim, deposit, withdraw, even admin fixes) panic, making all user funds permanently inaccessible.

- **How to spot them**

- `Audit unchecked arithmetic operations` - Look for direct multiplication/division of large numbers without overflow protection

- `Identify scaling factors and precision multipliers` - Constants like `10^18` are red flags when multiplied with user-controlled or time-accumulated values

## [M. Incorrect fee application in Uniswap V2-Inspired AMM Protocol](https://cantina.xyz/code/990ce947-05da-443e-b397-be38a65f0bff/findings/181)

### Note

- Issue: Fee applied to output amount instead of input amount, breaking constant product formula `(x × y = k)` used in standard AMM implementations

- Impact: Creates systematic arbitrage opportunities and pricing discrepancies that disadvantage users and can be exploited to drain liquidity

```rust
// Correct Uniswap V2 formula:
dy = (y * dx * (1 - f)) / (x + dx * (1 - f))  // Fee applied to input in both numerator and denominator

// Broken implementation:
result = (y * dx) / (x + dx)  // Full input used in constant product
dy = result * (1 - f)         // Fee applied to output as simple multiplier
```

- **How to spot**

- `Audit AMM fee application order` - Fee should be applied to input before constant product calculation, not to output after

- `Verify constant product maintenance` - x × y should remain constant after trades (accounting for fees)

## [M. Reward distribution may be vulnerable to sandwich attacks](https://cantina.xyz/code/990ce947-05da-443e-b397-be38a65f0bff/findings/38)

### Note

- Issue: Reward distribution between pools uses instant liquidity snapshots that can be manipulated via flash loans to unfairly skew reward allocation

- Impact: Attackers can steal up to 90%+ of reward allocation from legitimate pools by temporarily inflating liquidity during snapshot timing

- **How to Spot This Bug**

- `Identify instant snapshot mechanisms` - Look for functions that capture state at exact moments for important decisions

- `Audit reward/incentive distribution logic` - Systems that allocate value based on momentary state are vulnerable

- `Look for flash loan attack vectors` - Temporary liquidity inflation can skew any instant-based calculations

## [L. Expired Allowance Keys Linger, Causing Rent Leak and Indexer Noise](https://cantina.xyz/code/990ce947-05da-443e-b397-be38a65f0bff/findings/686)

### Note

- Issue: Expired allowance entries remain in storage after expiration instead of being cleaned up, causing unnecessary storage bloat. (SPECIFIC TO SOROBAN ONLY)

### Recommendation

```rust
pub fn read_allowance(e: &Env, from: Address, spender: Address) -> AllowanceValue {
    let key = DataKey::Allowance(AllowanceDataKey { from, spender });
    match e.storage().temporary().get::<_, AllowanceValue>(&key) {
        Some(allowance) if allowance.expiration_ledger < e.ledger().sequence() => {
            e.storage().temporary().remove(&key);          // cleanup ✅
            AllowanceValue { amount: 0, expiration_ledger: allowance.expiration_ledger }
        },
        Some(allowance) => allowance,
        None => AllowanceValue::default(),
    }
}
```

## [L. Incorrect Admin Transfer Pattern](https://cantina.xyz/code/990ce947-05da-443e-b397-be38a65f0bff/findings/569)

### Note

- `Issue`: Admin transfer mechanism allows current admin to unilaterally transfer ownership without requiring new admin's acceptance or confirmation

- Use the claim-based pattern:

`commit_transfer_ownership(new_admin)` – Called by the current admin.
`claim_ownership()` – Called by new_admin, confirming the transfer.

- **How to spot**

- Understanding modern best practice of ownership
