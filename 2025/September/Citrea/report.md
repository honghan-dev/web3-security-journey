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

## Medium Finding

## [SPV Merkle Proof Duplication Allows Forged Transaction Inclusion](https://cantina.xyz/code/49b9e08d-4f8f-4103-b6e5-f5f43cf9faa1/findings/660)

### Summary

1. SPV(Simplified payment verification) for bitcoin light clients to verify that a transaction was included in a block with downloading the entire blockchain.
2. `BitcoinLightClient::_verifyInclusion` function uses `bitcoin-spv::prove::verify256Merkle` which only valid the lower bits of the index.
3. Attacker can then provide an incorrect index, that results with the same lower bits as other merkle leaf

```rust
// example
Index 1 = 001 // correct
Index 5 = 101 (which is 1 + 4, where 4 = 2^2) // incorrect!!!
Index 9 = 1001 (which is 1 + 8, where 8 = 2^3)

// using this formula
valid_index + (k × 2^depth)
```

```solidity
  function _verifyInclusion(bytes32 _blockHash, bytes32 _wtxId, bytes calldata _proof, uint256 _index) internal view returns (bool) {
        require(_proof.length == coinbaseDepths[_blockHash] * 32, "Invalid proof length");
        bytes32 _witnessRoot = witnessRoots[_blockHash];
        return ValidateSPV.prove(_wtxId, _witnessRoot, _proof, _index); // <@ uses bitcoin-spv library, only checks lower bit
    }
```

### Why I missed this and how to spot this

1. Didn't understand fully how merkle verification works in this protocol and didn't trace in detail the lbrary that the protocol used to verify merkle tree.
2. Ensure any `merkle verification validate index bounds`. `Bitcoin-spv` library only uses the lower bits, so an index bound check is critical

## [M - Infinite timeout in the Bitcoin JSON-RPC client can freeze all critical Citrea compoments](https://cantina.xyz/code/49b9e08d-4f8f-4103-b6e5-f5f43cf9faa1/findings/569)

### Summary

1. Citrea `ClientBuilder` did not set timeout, could cause it to hang forever, when no reply from bitcoin node.
2. `BitcoinService` does use exponential backoff for RPC calls, but it won't get activated if it didn't receive any respond from bitcoin node.

### Why I missed this and how to spot this

1. Didn't understand the network connection of the protocol, go through how protocol request data from other node.
2. Ensure there's proper `timeout`, limit bound on receiving blocks.

## [M - Logic Error in Reorg Detection in chain-state update triggers unnecessary reorg handling](https://cantina.xyz/code/49b9e08d-4f8f-4103-b6e5-f5f43cf9faa1/findings/563)

### Summary

1. `pos != i` reorg detection always trigger because incorrect index comparison.
2. pos checks the recent block starting with index 0, whereas finality_depth index check starting with 1.
3. This result in reorg always get triggered because it is not comparing the same thing.

### Why I missed this and how to spot

1. Didn't understand the concept of reorg and reorg detection
2. Ensure reorg detection is properly triggered. This is a classic off by one index, understanding.
