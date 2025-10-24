# Clementine Finding

## High-level

Clementine is Citrea’s BitVM-based, trust-minimized two-way peg bridge between Bitcoin and the Citrea network (Citrea is a Bitcoin rollup that brings EVM-compatible smart contracts secured by Bitcoin).

Clementine enables Bitcoin to move in and out of the Citrea chain without requiring trusted intermediaries, using BitVM to achieve on-chain verification.

⚙️ How Clementine Works (High-Level)

Clementine uses `BitVM(Bitcoin VM)` — a Bitcoin scripting and off-chain computation verification framework — to optimistically verify computations on Bitcoin.

Clementine’s two-way peg relies on optimistic verification using the BitVM paradigm:

1. Bridge Deposit (Peg-in)

When a user wants to move BTC to Citrea:

- The BTC is locked in a Bitcoin bridge contract (a P2TR or Taproot-based script).

- A proof of this lock event is sent to Citrea.

- On Citrea, the bridge smart contract mints wrapped BTC (say, cBTC) representing the deposit.

2. Bridge Withdrawal (Peg-out)

When a user wants to redeem BTC:

- They burn cBTC on Citrea, producing a withdrawal message.

- The message must then be executed on Bitcoin to unlock the BTC.

- However, the Bitcoin network itself doesn’t know about Citrea — so Clementine uses BitVM verification to check that the withdrawal proof is legitimate.

3. Off-chain Computation via BitVM

Here’s where BitVM’s design comes in:

- A Bridge Operator (Prover) claims that a particular withdrawal is valid.

- One or more Watchtowers / Verifiers monitor the bridge and can challenge if they detect fraud.

## Findings

## High finding

## [H - Incorrect usage of get_num_verifiers in send_operator_asserts_if_ready breaks state machine](https://cantina.xyz/code/ce181972-2b40-4047-8ee9-89ec43527686/findings/568)

### Summary

1. Breakdown on Clementine's role

```rust
Operator: Submits BitVM “asserts” claiming the bridge state is valid.

Verifiers: Check computation correctness off-chain.

Watchtowers: On-chain guardians holding challenge UTXOs, ensuring the operator’s honesty and liveness.

// Typical flow
Operator (off-chain)         Watchtower (Bitcoin)              FSM

   |-- Sends assert -------->|
   |                         |-- Validates & challenges if needed
   |                         |-- Spends UTXO (confirm or dispute)
   |<------------------------|
   |                         |
   |-- FSM checks: spent_utxos == total_watchtowers? ---> Yes
   |
   | (send_operator_asserts_if_ready())
   |-- Sends next assert ---->
```

2. `send_operator_asserts_if_ready(if no challenges)` mistakenly checks the `num_of_verifiers` instead of `num_of_watchtowers` before allowing the operator to send asserts(claims for state correctness).
3. `if self.spent_watchtower_utxos.len() == self.deposit_data.get_num_verifiers()` this comparison will failed, and operator can't proceed with the next assert

### How to spot this

1. Understand the assertion flow and the condition that enables operator to proceed with asserts

## [H - Premature sending of disprove transactions allows malicious operator to steal bridge funds](https://cantina.xyz/code/ce181972-2b40-4047-8ee9-89ec43527686/findings/439)

### Summary

1. `Kickoff(operator) → Challenge(watchtower) → Assertion → Disprove/Timeout(watchtower) → Next Round or Reimburse`
2. Watchtower will submit a disprove tx once it received the preimage from operator, however, the logic flaw causes disprove tx to send too early, before watchtower receive the preimage, causing the disprove tx to fail.

```rust
async fn disprove_if_ready(&mut self, context: &mut StateContext<T>) {
    // 1. Check if all assertions tx been submitted by the operator
    // 2. Check if operator already finalized computation trace
    // 3. 
    // MISSING: && self.all_operator_challenge_acks_received() // all watchtower has received acknowledgement from operator
    if self.operator_asserts.len() == ClementineBitVMPublicKeys::number_of_assert_txs()
        && self.latest_blockhash != Witness::default()
        && self.spent_watchtower_utxos.len() == self.deposit_data.get_num_watchtowers()
    {
        self.send_disprove(context).await;
    }
}
```

### How to spot this

1. Understand what are the condition to fulfill before sending disprove transaction. How can operator cause watchtower's disprove transaction to fail.

## [M - Incorrect logic in `calculate_bump_feerate` prevents transaction fees from being bumped properly](https://cantina.xyz/code/ce181972-2b40-4047-8ee9-89ec43527686/findings/684)

### Summary

1. Bitcoin => `satoshi/vB(virtual byte)`, Citrea(EVM compatible block) => `gas * gwei`
2. `calculate_bump_feerate` when operator wants to rebroadcast an unconfirmed transaction with a higher fee for miner to prioritize it. (Replace by fee).
3. Function incorrectly calculate the additional bump fee. Intended purpose, new fee = original fee + bump fee.

```rust
pub async fn calculate_bump_feerate(
    &self,
    txid: &Txid,
    new_feerate: FeeRate,
) -> Result<Option<Amount>> {
    // [...]
    // Conservative vsize calculation
    let original_tx_vsize = original_tx_weight.to_vbytes_floor();

    let original_feerate = original_tx_fee.to_sat() as f64 / original_tx_vsize as f64;

    // Use max of target fee rate and original + incremental rate
    // @audit wrong formula, this is inversely proportional to the size of virtual byte
    let min_bump_feerate = original_feerate + (222f64 / original_tx_vsize as f64); // [1]

    let effective_feerate_sat_per_vb = std::cmp::max(
        new_feerate.to_sat_per_vb_ceil(),
        min_bump_feerate.ceil() as u64,
    );

    // If original feerate is already higher than target, avoid bumping
    // @audit should compare to effective_feerate_sat_per_vb instead cause new_feerate could be ower than effective fee rate
    if original_feerate >= new_feerate.to_sat_per_vb_ceil() as f64 { // [2]
        return Ok(None);
    }

    Ok(Some(Amount::from_sat(effective_feerate_sat_per_vb)))
}
```

### How to spot this

1. First understand the flow of bumping up fee rate to prioritize transaction.
2. Understand how are the bump being implemented.

## [M - Incorrect error handling in state machine](https://cantina.xyz/code/ce181972-2b40-4047-8ee9-89ec43527686/findings/563)

### Summary

1. Too broad error handling in state machine, possible of halting entire process.
2. 1 error can cause the entire system to stop processing

```rust
// @audit one error crashes all the FSM
if !all_errors.is_empty() {
    // revert state machines to checkpointed state
    self.kickoff_machines = kickoff_machines_checkpoint;
    self.round_machines = round_machines_checkpoint;

    return Err(eyre::eyre!(
        "Multiple errors occurred during state processing: {:?}", all_errors
    ));
}
```

### How to spot this

1. Ensure proper error handling if one of the thread fails, so that it doesn't affect the entire system.

## []()

### Summary

1. In `Operator::get_reimbursement_txs`, within the branch that sends the kickoff when it is not yet onchain, the code constructs `move_txid` as `Txid::all_zeros()` and then calls `get_payout_info_from_move_txid` with this zero txid. It should use the actual deposit's `move_to_vault` txid, not a zero txid placeholder.

```rust
// core/src/operator.rs
// @audit uses placeholder instead of actual move_txid
let move_txid = Txid::all_zeros();

let (_, _, _, citrea_idx) = self
    .db
    .get_payout_info_from_move_txid(dbtx.as_deref_mut(), move_txid)
    .await?
    .ok_or_eyre("Couldn't find payout info from move txid")?;

// CODE OMITTED //
```

### How to spot this

1. Understand the bridge reimbursement flow, it should correctly retrieve the actual deposit txid.
