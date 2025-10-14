# Other finding

## High Severity

## [H - Unsafe UTXO Selection Allows DoS via RBF Attack on DA Service](https://cantina.xyz/code/49b9e08d-4f8f-4103-b6e5-f5f43cf9faa1/findings/322)

### Summary

1. Bitcoin UTXO has multiple fields, `utxo.safe == true` when it is a confirmed UTXO(onchain).
2. Attacker can submit a `Replace-By-Fee(RBF)` transactions with low fees, and then subsequently submit another transaction when the DA service include it.
3. This attack no longer possible if when the utxo is onchain, `utxo.safe == true`

```rust
  let utxos: Vec<UTXO> = utxos
            .into_iter()
            .filter(|utxo| {
                utxo.spendable // @note wallet can spend this if confirmed
                    && utxo.solvable
                    && utxo.amount > Amount::from_sat(REVEAL_OUTPUT_AMOUNT)
                    // @audit - didn't check utxo.safe, not necessary onchain
            })
            .map(Into::into)
            .collect();
```

### Why I missed this and how to spot this

1. Didn't understand how Bitcoin utxo works, what are the fields it contains and what are the safety feature.
2. Understand Bitcoin model, what it contains and what kind of attack vector is possible if certain safety check is not done.

## [H -]()
