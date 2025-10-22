# My finding

## Medium Finding

<details>
<Summary>Last pending commitment will be missing when sequencer restarts</Summary>

## Summary

The LedgerDB::get_pending_commitments doesn't include the last commitment index when querying commitment data in LedgerDB::get_data_range. This results in incomplete commitments recovery whenever the sequencer restarts.

## Finding Description

In LedgerDB::get_pending_commitments:

```rust
fn get_pending_commitments(&self) -> anyhow::Result<Vec<SequencerCommitment>> {
        let commitment_indexes = self.db.get::<PendingSequencerCommitment>(&())?;
        match commitment_indexes {
            Some(mut commitment_indexes) => {
                if commitment_indexes.is_empty() {
                    return Ok(vec![]);
                }
                commitment_indexes.sort_unstable();
                let start = commitment_indexes[0]; // @note first pending index
                let end = commitment_indexes[commitment_indexes.len() - 1]; // @note last pending index

                // @audit [HIGH] doesn't include the last commitment
@>          self.get_data_range::<SequencerCommitmentByIndex, _,_>(&(start..end)) // @note doesn't include the last pending index
            }
            None => Ok(vec![]),
        }
    }
```

However, Range type (start..end) is exclusive, meaning the last pending commitment (end) is not included in the query.

This range is then passed in LedgerDB::get_data_range to retrieve the values.

```rust
fn get_data_range<T, K, V>(&self, range: &std::ops::Range<K>) -> Result<Vec<V>, anyhow::Error>
    where
        T: Schema<Key = K, Value = V>,
        K: Into<u64> + Copy + SeekKeyEncoder<T>,
    {
        let mut raw_iter = self.db.iter()?;
        let max_items = (range.end.into() - range.start.into()) as usize; // @note smaller range than expected
        raw_iter.seek(&range.start)?;
        let iter = raw_iter.take(max_items);
        let mut out = Vec::with_capacity(max_items);
        for res in iter {
            let batch = res?.value;
            out.push(batch)
        }
        Ok(out)
    }
```

This will miss the final pending commitment

## Impact Explanation

[HIGH] - Missing the last pending commitment can cause incomplete state recovery,

## Likelihood Explanation

[MEDIUM] - This happens whenever a sequencer restarts, with an existing pending commitment

## Proof of Concept

Assume the following pending commitment indexes exist in the database: `[8, 9, 10, 11, 12]`

get_pending_commitments sets:

```rust
start = 8
end = 12
It then passes start..end (i.e., 8..12) to get_data_range. This range includes indexes 8 through 11, excluding index 12 — the last pending commitment.

As a result, the commitment at index 12 is never recovered, causing incomplete state restoration.
```

## Recommendation

Include the last index in the LedgerDB::get_data_range
</details>

---

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

1. Reorg detection is by comparing the blocks(up to finality_depth) from Bitcoin network and check if the tip has changed.
2. If then compare the position of the Bitcoin blocks and the `recent_blocks(blocks being recorded in the Citrea network)`.
3. `pos != i` reorg detection always trigger because incorrect index comparison.
4. pos checks the recent block starting with index 0, whereas finality_depth index check starting with 1. This result in reorg always get triggered because it is not comparing the same thing.

### Why I missed this and how to spot

1. Didn't understand reorg detection and the flow of the detection.
2. Ensure reorg detection is properly triggered. Understand what being compared in the function, eg.`blockhash`, `blockheight` or `tip`  This is a classic off by one index, understanding.

## [M - Insufficient Funds Due to Incorrect Amount Comparison](https://cantina.xyz/code/49b9e08d-4f8f-4103-b6e5-f5f43cf9faa1/findings/544)

### Summary

1. UTXO(unspent transaction output) are Bitcoin's way of tracking ownership. Each wallet potentially consist of multiple UTXOs.
2. The `choose_utxos` function incorrectly compares the total accumulated UTXO value against the remaining amount needed (after subtracting required UTXOs) instead of comparing against the original target amount.
3. This causes early termination when the function thinks it has enough funds, but actually returns insufficient UTXOs, leading to invalid transactions where `inputs < outputs`.

### Why I missed this and how to spot

1. Didn't understand the UTXO selection flow, didn't notice that the selection comparison wasn't done correctly.
2. Ensure the logics are correct, using the correct value as comparison.

## [M - Infinite HTTP client timeout in FeeService can freeze the sequencer](https://cantina.xyz/code/49b9e08d-4f8f-4103-b6e5-f5f43cf9faa1/findings/417)

### Summary

1. `FeeService` Bitcoin fee rates from mempool.service API using `reqwest` HTTP without a configured timeout.
2. This could freeze sequencer operation if the network call never completes as this module is used in `run_da_queue()` and `da_block_monitor`

### Why I missed this and how to spot

1. Didn't consider the impact of not having timeout, that can freeze certain sequencer operation.
2. Ensure any request has a timeout, or alternate method if the response wouldn't complete.

## [M - Bitcoin DA Unbounded pre-decompression accumulation causes DoS/OOM](https://cantina.xyz/code/49b9e08d-4f8f-4103-b6e5-f5f43cf9faa1/findings/273)

### Summary

1. The protocol will split ZK proof into multiple chunks if the proof is too large.
2. However, the `Bitcoin_da::extract_relevant_zk_proofs()` has no limit for pre-decompression accumulation.
3. ZK proof can be large causing OOM(out-of-memory) which leads to DOS.

### Why I missed this and how to spot this

1. Didn't check that anything untrusted needs to have validation and bound limit.
2. Ensure any input(reply from other node/prover) needs to be validated and have upper bound limit.
