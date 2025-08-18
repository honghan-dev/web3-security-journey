# Table of contents

### My Findings

- [L-01. Mismatch between TokenShares::TotalShares and actual token supply in LP token if user directly burn their token.](#my-finding-l01)
- [L-02. Incorrect desired_a < min_a validation in pool::get_deposit_amounts](#my-finding-l02)

## <a id="my-finding-l01"></a>L01. Mismatch between TokenShares::TotalShares and actual token supply in LP token if user directly burn their token

## Summary

TotalShares in the pool contract is not updated if users burn their LP tokens directly via the token contract's Token::burn() function, creating a discrepancy between the recorded TotalShares and the actual LP token supply.

## Finding Description

The liquidity pool contract maintains an internal `TokenShare::DataKey::TotalShares` variable to track the total amount of LP tokens in circulation. However, this accounting is only updated when users interact with the pool through designated functions (such as withdraw()).

In the `Token::burn()` function, it doesn't update the `TotalShares`. If users call the `burn()` function directly on the LP token contract, the tokens are removed from circulation, but the pool contract's `TotalShares` variable remains unchanged. This creates a permanent accounting mismatch.

```rust
fn burn(e: Env, from: Address, amount: i128) {
        // @audit-low burning token doesn't update the `TotalShares`
        from.require_auth();
        check_nonnegative_amount(&e, amount);
        bump_instance(&e);

        spend_balance(&e, from.clone(), amount);
        TokenUtils::new(&e).events().burn(from, amount);
}
```

## Impact Explanation

LOW - Impact is limited, however, the actual TotalShares would be accurate.

## Likelihood Explanation

LOW - Users typically interact with LP tokens through the pool's designated functions, and no clear financial benefit to burning tokens directly.

## Proof of Concept

User deposited fund into liquidity pool, getting 100 shares in return
TotalShares = 100
User burns 10 of their token, actual Total Supply is 90
TotalShares still shows 100

## Recommendation

Implement an access control such that only the pool can call burn function.

## <a id="my-finding-l02"></a>L02. Incorrect desired_a < min_a validation in pool::get_deposit_amounts

## Summary

In pool::get_deposit_amounts, the logic incorrectly compares desired_a against min_a, instead of comparing the computed amount_a against min_a.

## Finding Description

In the pool::get_deposit_amounts function, two branches handle different deposit scenarios. While the first branch correctly checks if the calculated amount meets the minimum requirement (amount_b < min_b), the second branch inconsistently checks an input parameter against a minimum (desired_a < min_a) instead of checking the calculated amount (amount_a < min_a).

```rust
pub fn get_deposit_amounts(
    // code omitted //
) -> (u128, u128) {
    // code omitted //

    if amount_b <= desired_b {
        if amount_b < min_b {
            panic_with_error!(e, LiquidityPoolValidationError::InvalidDepositAmount);
        }
        (desired_a, amount_b)
    } else {
        let amount_a = desired_b.fixed_mul_floor(&e, &reserve_a, &reserve_b);
        // @audit using desired_a to check against min_a
@>     if amount_a > desired_a || desired_a < min_a {
            panic_with_error!(e, LiquidityPoolValidationError::InvalidDepositAmount);
        }
        (amount_a, desired_b)
    }
}
```

## Impact Explanation

Informational. Since min_a is set to 0 in the liquidity_pool::deposit, impact is insignificant

## Likelihood Explanation

This is a code logic issue rather than an exploitable vulnerability.

## Proof of Concept

Currently, min_a is set to 0, and it is not exploitable. If the contract were modified to use non-zero minimums, the inconsistent validation pattern could result in a scenario where the calculated amount_a falls below the minimum (min_a), but the transaction still proceeds because the check incorrectly check desired_a against min_a instead.

## Recommendation

Replace desired_a with amount_a when check against min_a, similar to amount_b < min_b

```diff
pub fn get_deposit_amounts(
    // code omitted //
) -> (u128, u128) {
    // code omitted //
    if amount_b <= desired_b {
        if amount_b < min_b {
            panic_with_error!(e, LiquidityPoolValidationError::InvalidDepositAmount);
        }
        (desired_a, amount_b)
    } else {
        let amount_a = desired_b.fixed_mul_floor(&e, &reserve_a, &reserve_b);
        // @audit using desired_a to check against min_a
+    if amount_a > desired_a || amount_a < min_a {
-    if amount_a > desired_a || desired_a < min_a {
            panic_with_error!(e, LiquidityPoolValidationError::InvalidDepositAmount);
        }
        (amount_a, desired_b)
    }
}
}
```
