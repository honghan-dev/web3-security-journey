# Other finding

## High Severity

## [H - Unsafe UTXO Selection Allows DoS via RBF Attack on DA Service](https://cantina.xyz/code/49b9e08d-4f8f-4103-b6e5-f5f43cf9faa1/findings/322)

### Summary

1. Bitcoin UTXO has multiple fields, `utxo.safe == true` when it is a confirmed UTXO(onchain).
2. Attacker can submit a `Replace-By-Fee(RBF)` transactions with low fees, and then subsequently submit another transaction when the DA service include it.
3. The operation will be DOS when the DA service tries to submit an utxo that is not onchain.
4. This attack no longer possible if when the utxo is onchain, `utxo.safe == true`

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
3. What are the criteria transaction has to meet for Bitcoin layer to include it onchain

## [H - DoS Vulnerability in send_raw_deposit_transaction Allowing Unlimited Deposit Mempool Growth](https://cantina.xyz/code/49b9e08d-4f8f-4103-b6e5-f5f43cf9faa1/findings/55)

### Summary

1. `send_raw_deposit_transaction` RPC endpoint has not upper bound on how many transactions and it doesn't check for any duplicate transaction.
2. The function performs a dry-run before adding the transaction into the mempool. Dry-run could be expensive. This causes the memory to grow exponentially and eventually leading to OOM(Out-of-memory)
3. Recommendation, implementing both a `duplicate check` and a `rate limit`.

### Why I missed this and how to spot this

1. Didn't understand the flow of adding new deposit transaction, what it checks and how many transactions it accepts.
2. Ensure that code implement a rate limit and duplicate check when it comes to accept transactions before adding into the mempool.
