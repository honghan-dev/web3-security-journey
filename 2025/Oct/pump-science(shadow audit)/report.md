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

1. After swap invariant check, actual native sol amount shouldn't be less than sol reserves in ledger.
2. `sol_escrow.lamports()` would include the rent amount hence causing the `sol_escrow` larger, passing some checks that should result in error.

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
