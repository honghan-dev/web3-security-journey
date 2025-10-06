# My Finding

## [M -]()

# Other finding that I missed

## High Severity

[H - The lock_pool operation can be dos](https://code4rena.com/audits/2025-01-pump-science/submissions/F-3)

### Summary

1. In the `pump_science::lock` method, it creates calls Meteora protocol to create a `lock_escrow` account.
2. However, the `lock_escrow` account creation flow in Meteora doesn't require signature from owner. Hence, any malicious owner can frontrun the transaction using the original owner key, pool key to create the exact same `lock_escrow` PDA
3. The subsequent `lock` call will fail because the PDA already been initialized.
4. Recommendation: If if the `lock_escrow` already exist before creating a new one

```rust
    /// CHECK: Lock account
    #[account(
        init, <@ // only can init once
        seeds = [
            "lock_escrow".as_ref(),
            pool.key().as_ref(),
            owner.key().as_ref(),
        ],
        space = 8 + std::mem::size_of::<LockEscrow>(),
        bump,
        payer = payer,
    )]
@>    pub lock_escrow: UncheckedAccount<'info>,

    /// CHECK: Owner account
@>    pub owner: UncheckedAccount<'info>,
```

### Why I missed and how to spot it

1. Didn't understand the entire lock process well, what can DOS the function.
2. Look into how each account are created, and whether the account can be created first, either via frontrunning.

- `init` keyword is crucial, cause it will panic if any accounts were created before.

3. Check how the authority process works in any function call.

# Medium Findings

## [H - Missing Update of migration_token_allocation on Global Struct](https://code4rena.com/audits/2025-01-pump-science/submissions/F-583)

### Summary

1. `migration_token_allocation` didn't get updated during initialization. The amount is set at none. This bricks the migration process.
2. `Global` config struct, has a default method to set a default `migration_token_allocation` value, however the method is not being called.
3. Furthermore, `update_setting` missed out updating the `migration_token_allocation` value.

### Why I miss and how to spot this

1. Didn't trace how all the critical value being updated during initialization.
2. Ensure all the fields are updated or being set to a default value.

## [M - Bonding Curve Invariant Check Incorrectly Validates SOL Balance Due to Rent Inclusion](https://code4rena.com/audits/2025-01-pump-science/submissions/F-589)

### Summary

1. After swap invariant check- actual native sol amount shouldn't be less than sol reserves in ledger.
2. `sol_escrow.lamports()` would include the rent exempt hence causing the `sol_escrow` larger, passing some checks that should result in error.

```rust
// Get raw lamports which includes rent
let sol_escrow_lamports = sol_escrow.lamports();
// Ensure real sol reserves are equal to bonding curve pool lamports 
if sol_escrow_lamports < bonding_curve.real_sol_reserves {
    return Err(ContractError::BondingCurveInvariant.into());
}
```

### Why I missed and how to spot it

1. Didn't understand Solana's rent exempt model. Every PDA created will contain `rent-exempt` amount (**approx 890,880 lamports**)
2. Might have to deduct the rent exempt amount before using it for comparison.

## [M - Abrupt fee transition from 8.76% to 1% at slot 250 due to incorrect linear decrease formula](https://code4rena.com/audits/2025-01-pump-science/submissions/F-616)

### Summary

1. Fee transition creates a significant 7.76% economic discontinuity at slot 250-251 boundary, causing incorrect fees implementation as protocol intended.
2. The incorrect calculation result in not a smooth fee curve.

### Why I missed and how to spot it

1. I didn't think that this is significant.
2. Should report any discrepancies between the functionality and the intention. Documentation highlights a smooth curve instead of a sudden drop.

## Low findings

### Precision loss related issue

1. Arithmetic operation steps can cause fee to truncate. Lesser fee charge on user.

```rust
let fee_bps = (-8_300_000_i64)
    .checked_mul(slots_passed as i64)
    .ok_or(ContractError::ArithmeticError)?
    .checked_add(2_162_600_000)
    .ok_or(ContractError::ArithmeticError)?
    .checked_div(100_000)
    .ok_or(ContractError::ArithmeticError)?;
    // @note dividing 100_000 here can cause truncation

    sol_fee = bps_mul(fee_bps as u64, amount, 10_000).unwrap();

    // (-8300000 * 200 + 2162600000) / 100000 = 4963
    // Actual calculation should be: 49.63%
    // But due to integer division, it becomes 49%
```

### Gas fee related issue

1. 0.04 sol hardcoded gas value, transaction might revert during network congestion

```rust
let token_a_amount = ctx
    .accounts
    .bonding_curve
    .real_sol_reserves
    .checked_sub(ctx.accounts.global.migrate_fee_amount)
    .ok_or(ContractError::ArithmeticError)?
    .checked_sub(40_000_000)  // Hardcoded 0.04 SOL for gas
    .ok_or(ContractError::ArithmeticError)?;
```
