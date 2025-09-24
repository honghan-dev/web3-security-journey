# My report

# ConsensusRegistry::delegateStake incorrectly updates the delegator and validatorAddress in delegations mapping

<details>
<Summary>ConsensusRegistry::delegateStake incorrectly assigns the validatorAddress and delegator fields in the Delegation struct, resulting in reward attribution logic breaking downstream.</summary>

# Finding Description

The documentation states that the delegator will receive all the stake rewards and staked balance, here

The Delegation struct is declared as:

```solidity
struct Delegation {
        bytes32 blsPubkeyHash;
        address validatorAddress; <@--
        address delegator; <@--
        uint8 validatorVersion;
        uint64 nonce;
    }
```

However, In the `ConsensusRegistry::delegateStake` the struct is updated incorrectly

```rust
function delegateStake(
        bytes calldata blsPubkey,
        address validatorAddress,
        bytes calldata validatorSig
    )
        external
        payable
        override
        whenNotPaused
    {
    // code omitted //
        // @audit - incorrect delegator and validatorAddress position
        delegations[validatorAddress] =
            Delegation(
            blsPubkeyHash,
            msg.sender, // <@ - should be validatorAddress
            validatorAddress, // <@ - should be delegator address
            validatorVersion,
            nonce + 1
           );

        _recordStaked(blsPubkey, validatorAddress, true, validatorVersion, stakeAmt);
    }
```

This swaps validator and delegator addresses, which leads to incorrect behaviour downstream. Because delegation.delegator is set to the validator’s address (mistakenly), _getRecipient will return the validator’s address even when a delegator exists.

`In StakeManager::_getRecipient:`

```rust
/// @dev Identifies the validator's rewards recipient, ie the stake originator
    /// @return _Returns the validator's delegator if one exists, else the validator itself
    function_getRecipient(address validatorAddress) internal view returns (address) {

        Delegation storage delegation = delegations[validatorAddress];
        
        // @audit - this will return the `validatorAddress` instead of the delegator
        address recipient = delegation.delegator; <@- validator address instead 
        
        if (recipient == address(0x0)) recipient = validatorAddress;

        return recipient;
    }
```

As a result, validator can call ConsensusRegistry::claimStakeRewards or ConsensusRegistry::unstake and steal delegator's fund

`ConsensusRegistry::claimStakeRewards`:

```solidity
function claimStakeRewards(address validatorAddress) external override whenNotPaused nonReentrant {
        // code omitted //

        // require caller is either the validator or its delegator
@>        address recipient = _getRecipient(validatorAddress);
            // @audit - this check pass
@>        if (msg.sender != validatorAddress && msg.sender != recipient) revert NotRecipient(recipient);
        // @audit fund goes to recipient(validator)
        uint256 rewards =_claimStakeRewards(validatorAddress, recipient, validatorVersion);

        emit RewardsClaimed(recipient, rewards);
    }
```

`ConsensusRegistry::unstake`

```solidity
function unstake(address validatorAddress) external override whenNotPaused nonReentrant {
        // code omitted //

        // require caller is either the validator or its delegator
@>        address recipient = _getRecipient(validatorAddress);
            // @audit - this check pass
@>        if (msg.sender != validatorAddress && msg.sender != recipient) revert NotRecipient(recipient);

        // code omitted //

        // return stake and send any outstanding rewards
        // @audit fund goes to recipient(validator)
        uint256 stakeAndRewards = _unstake(validatorAddress, recipient, validator.stakeVersion);


        emit RewardsClaimed(recipient, stakeAndRewards);
    }
```

# Impact Explanation

'HIGH' - This allows the validator to call claimStakeRewards and steal funds from the delegator.

# Likelihood Explanation

HIGH - This occurs every time a delegator uses delegateStake to delegate stake to a validator.

# Proof of Concept

To simplify the test case, I've changed the visibility of the StakeManager::delegations from internal to public

- mapping(address => Delegation) internal delegations;

- mapping(address => Delegation) public delegations;

Insert the following test into the ConsensusRegistryTest.t.sol

```solidity
function test_delegateStakeIncorrectAddress() public {
        vm.prank(crOwner);
        uint256 validator5PrivateKey = 5;
        validator5 = vm.addr(validator5PrivateKey);
        address delegator = _addressFromSeed(42);
        vm.deal(delegator, stakeAmount_);

        consensusRegistry.mint(validator5);

        // validator signs delegation
        bytes32 structHash = consensusRegistry.delegationDigest(validator5BlsPubkey, validator5, delegator);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(validator5PrivateKey, structHash);
        bytes memory validatorSig = abi.encodePacked(r, s, v);

        // Check event emission
        bool isDelegate = true;
        vm.expectEmit(true, true, true, true);
        emit ValidatorStaked(
            ValidatorInfo(
                validator5BlsPubkey,
                validator5,
                PENDING_EPOCH,
                uint32(0),
                ValidatorStatus.Staked,
                false,
                isDelegate,
                uint8(0)
            )
        );
        vm.prank(delegator);
        consensusRegistry.delegateStake{ value: stakeAmount_ }(validator5BlsPubkey, validator5, validatorSig);

        // Require to change `delegations` visibility to access
        // Retrieve the `delegations` field
        (
            bytes32 genBlsPubkeyHash,
            address genValidatorAddress,
            address genDelegator,
            uint8 genStakeVersion,
            uint64 genNonce
        ) = consensusRegistry.delegations(validator5);

        // Demonstrate that both addresses were swapped
        assertEq(genValidatorAddress, delegator);
        assertEq(genDelegator, validator5);
    }
```

Test log:

```sh
[PASS] test_delegateStakeIncorrectAddress() (gas: 432199)
Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 9.64ms (2.54ms CPU time)
```

Test passed, indicating that both addresses were swapped

# Recommendation

Update the correct address in delegateStake

```diff
delegations[validatorAddress] = Delegation(
            blsPubkeyHash,
-         validatorAddress
-         msg.sender,
            validatorVersion,
            nonce + 1
        );
```

</details>

# Other findings that I've missed

# High Findings

## [H - Nodes can be slashed for requesting large P2P data](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/1023)

### Summary

1. Caused by having a `max_rpc_message_size` (1mb)
2. Node A requesting missing batch from Node B, if the batch size request is `>` 1mb, Node B's respond can't go through due to:

```rust
if self.decode_buffer.len() > self.max_chunk_size {
    return Err(std::io::Error::other("encode data > max_chunk_size"));
}
```

3. Then the system will penalize Node B for failing to respond

```rust
ReqResEvent::OutboundFailure { peer, .. } => {
    self.swarm.behaviour_mut().peer_manager.process_penalty(peer, Penalty::Medium);
}
```

### Why I miss this and how to spot this

1. Didn't understand the `request/respond` flow when a node needs additional batch data.
2. Didn't know what happens when a node requested a large batch of data, and didn't trace the batch data flow in detail
3. In `request/respond` crate, check if there's any cap(important for protection), but also what if the limit is triggered?

## [Complete consensus takeover due to missing BLS proof of possession verification](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/845)

### Summary

1. Protocol didn't verify validator's proof of procession when validator join using their BLS key
2. Malicious can reverse engineer to figure out the rouge key. `PK_rogue = PK_target - (PK₁ + PK₂ + ... + PKₙ)`
3. Then submit this rogue key `Aggregate = PK₁ + PK₂ + ... + PKₙ + PK_rogue = PK_target` as their own BLS key.
4. This allow them to take over the consensus

### Why I miss this and how to spot this

1. I didn't understand how BLS works. [read here](https://github.com/honghan-dev/web3-security-notes/blob/master/Cryptography/bls.md) for some simple write up on how BLS works.
2. Didn't notice protocol didn't have any POP(Proof of Possession) validation before allowing Validator add their key.
3. Look for input validation when validator registers. Ensure validator actually owns the key before adding them.

## [Lack of authentication mechanism for transaction batches](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/745)

### Summary

1. Batches published by worker P2P doesn't have a mechanism to ensure it is sent by an approved Validators.
2. `validate_batch()` check only if the batch is well-formed without enforcing authentication, any connected peer can submit batches.
3. This could lead to unbounded storage consumption on the node.

### Why I miss this and how to spot this

1. Didn't check whether node validates the sender before accepting.
2. Ensure that node validates the batch, and also validates whether the sender is approved. (This can filter out many other unapproved nodes from sending)

# Medium Findings

## [M - Non-persistent header tracking leads to transaction loss on node restart](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/1177)

### Summary

- proposer maintains proposed headers in memory (`proposed_headers` map) but only persists the latest header to database (`last_proposed` table).
- This becomes an issue because every Telcoin node maintains their own mempool, and not gossip to other node.
- When a node restarts, all previously proposed but uncommitted headers are lost from memory and cannot be recovered, causing their transaction batches to never be rescheduled for inclusion.

### What went wrong and how to spot this

1. Didn't understand the proposer flow in detail.
2. Understand what data should persist in database, so node can retrieve if it restarts
3. Critical consensus data must survive restarts, where individual node failures affect the whole network.

## [M - Logic error in Proposer's calc_min_delay() function leads to reduced network throughput](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/1104)

### Summary

1. Telcoin implements time-based rounds for block proposal:

- During even rounds, only the designated leader should propose a header (with zero delay)
- During odd rounds, all committee members should propose headers with minimal delay to maximize the proposal rate

2. Issue: Documentation says `whole committee` gets zero delay in odd rounds, but code only gives zero delay to the single leader because of this check

```rust
/// Calculate the min delay to use when resetting the min_delay_interval.
///
/// The min delay is reduced when this authority expects to become the leader of the next round.
/// Reducing the min delay increases the chances of successfully committing a leader.
///
/// NOTE: If the next round is even, the leader schedule is used to identify the next leader. If
/// the next round is odd, the whole committee is used in order to keep the proposal rate as
/// high as possible (which leads to a higher round rates). Using the entire committee here also
/// helps boost scores for weaker nodes that may be trying to resync.
fn calc_min_delay(&self) -> Duration {
    // check next round
    let next_round = self.round + 1;

    // compare:
    // - leader schedule for even rounds
    // - entire committee for odd rounds
    //
    // NOTE: committee size is asserted >1 during Committee::load()
    if (next_round % 2 == 0
        && self.leader_schedule.leader(next_round).id() == self.authority_id)
@>      || (next_round % 2 != 0
@>          && self.committee.leader(next_round as u64).id() == self.authority_id)
    {
        Duration::ZERO
    } else {
        self.min_header_delay
    }
}
```

### What went wrong and how to spot this

1. Didn't understand round advancement flow enough
2. Compare the function against what it documents says it will do.

## [M - primary: requested_parents DoS issue](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/1095)

### Summary

1. Malicious committee members can cause unbounded memory growth of requested_parents map by sending vote requests for arbitrarily future rounds.
2. These entries stays forever in the memory, because garbage collection mechanism only removes entries from past rounds, didn't remove any unnecessary future rounds

### What went wrong and how to spot this

1. First understand what are the things that a node request from peers.
2. What data structure the field has, (HashMap or Vec), and whether there are limit impose on it
3. Understand how are those data being cleared. In this case, garbage collector only clears past rounds, but not future rounds.

### Key Takeaway

1. Always ask how malicious node can DOS other nodes, when other node request data from them.
2. Map out what are the data being request and respond between different nodes.

## [M - Transactions Are Removing From The Meme Pool Prematurely Which Results In Transaction Being Lost After Epoch Transitions](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/967)

### Summary

1. Transactions removed from mempool after batch validation but before blockchain inclusion
2. Node can't recover the transaction batch if the batch wasn't committed during epoch transition, then it's permanently lost
3. Batch validation complete → Epoch transition → New proposer ignores stored batches

### What went wrong and how to spot them

1. Incorrect transaction update flow
2. Understand the transaction flow, transaction -> batch -> commit -> remove, prevent transaction from being removed prematurely.
3. Track transaction flow carefully, ensure the flow is correct and answer what happens if things didn't go as planned. Is there any way to recover?
4. Identify what are the important that node must have during restart or any transition.

## [M - Permanent leader selection bias creates distorted consensus reward distribution](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/865)

## Issue Overview

• **Bug Type**: Systematic bias in weighted random selection due to constrained seed space
• **Root Cause**: Using only even numbers (0,2,4,6...) as RNG seeds + fixed validator ordering by BLS key
• **Impact**: Certain committee positions get 13.44% vs 11.58% selection rates (should be 12.5% each)
• **Attack Vector**: Validators can mine favorable BLS keys to land in advantaged committee positions

## Two-Layer Architecture

• **Committee Selection**: Random shuffling determines WHO gets into committee (fair)
• **Leader Selection**: Within committee, validators ordered by BLS key create predictable bias (unfair)
• **Key Point**: Random membership doesn't eliminate within-committee positional advantage

## Bug Spotting Checklist

### **Random Number Generation Red Flags**

```rust
// Look for:
StdRng::from_seed(simple_sequential_value)    // Predictable seeds
rng_seed = counter * 2                        // Constrained entropy space  
seed_source = even_numbers_only               // Missing odd values
```

### **Deterministic Ordering Issues**

```rust  
// Watch for:
map.values().collect()                        // Fixed iteration order
sort_by_key(|item| item.public_key)          // Predictable sorting
authorities.iter()                           // Consistent ordering
```

### **Selection Algorithm Bias**

```rust
// Red flags:
choose_weighted() with deterministic RNG      // Systematic bias possible
sequential_seeds + fixed_ordering             // Bias amplification  
equal_weights + unequal_results              // Selection bias indicator
```

### **Key Audit Questions**

1. **"Is the randomness source truly random?"** - Check for constrained seed spaces
2. **"Does ordering affect selection probability?"** - Test if position matters
3. **"Can participants influence their position?"** - Look for gaming vectors
4. **"Are equal inputs producing equal outputs?"** - Verify fairness metrics

**Golden Rule**: When deterministic ordering meets constrained randomness, systematic bias often emerges. Always test actual distribution vs expected distribution with statistical analysis.

## [M - Force burning a validator in the scheduled committees will reduce the size of future committees](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/846)

### Summary

- Issue: Ejecting malicious validators reduces committee size below BFT safety thresholds
- Root Cause: System uses actual committee count instead of design target for next epoch sizing
- Impact: BFT fault tolerance degrades (7→6 validators reduces tolerance from 2 to 1 faults)
- Attack Vector: Forces system into vulnerable state where fewer Byzantine nodes can break consensus

### Audit Questions

- "Can security parameters degrade during operation?" - Check for minimum bounds
- "What happens when validators are removed?" - Trace impact on BFT tolerance
- "Are design assumptions enforced continuously?" - Verify parameter stability
- "Can the system handle originally-planned fault scenarios after changes?" - Test degraded states

## [M - Gossip message authorization bypass](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/835)

### Summary

- Issue: Topic-based access control can be bypassed by sending restricted content through permissionless topics(bypassing topic restricted message, `tn-txs` and `tn-worker`)
- Root Cause: Authorization validates topic permissions but message processing ignores which topic content arrived on
- Attack Vector: Send batch messages (committee-only) via "tn-txn" topic (permissionless) to bypass committee restrictions
- Impact: Non-committee members can influence worker processes and spam sensitive gossip channels

```rust
Topic "tn-txn", auth=None, any sender → ACCEPT
Topic "tn-worker", auth=Some({A,B,C}), sender=A → ACCEPT  
Topic "tn-worker", auth=Some({A,B,C}), sender=D → REJECT
Topic "unknown", any sender → REJECT
Message without source, any topic → REJECT
```

### Key Audit Questions

- "Does content processing validate its source?" - Check if handlers verify origin context
- "Can restricted content arrive through unrestricted channels?" - Test cross-topic attacks
- "Are authorization and processing tightly coupled?" - Look for gaps between layers

## [M - Malicious Consensus Header Response Permanently Disables Node State Synchronization](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/772)

### Summary

- Issue: Multiple validation gaps in consensus header sync allow malicious headers to be accepted and processed
- Root Cause: Two-task sync architecture with inconsistent validation - tracking task has weak validation, streaming task has stronger validation but fatal error handling(process stopped when encountered incorrect digest)
- Attack Vector: Send headers with valid signatures but wrong round certificates, malicious timestamps, or incorrect metadata
- Impact: RPC API poisoning, execution engine manipulation, eventual sync termination when honest headers fail validation

### Key Audit Questions

"Are validation rules consistent across all processing paths?" - Compare validation depth between parallel tasks/components
"Does any single bad input permanently disable functionality?" - Check for fatal error handling vs graceful retry mechanisms
"Can validation be bypassed through alternative entry points?" - Map all ways external data enters the system and verify equivalent security checks

## [M - Workers store gossiped batches without validation](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/761)

### Summary

1. Storing fetched batches via gossip without validation, while directly reported batches are properly validated.
2. Root Cause: Two different code paths for batch storage with inconsistent validation requirements
3. ossip invalid batch digests to trigger automatic fetching and storage of malformed batches
4. Storage DoS (disk filled with invalid data) and network DoS (bandwidth wasted on invalid batch requests)

### How to spot this

1. Critical data such as block header fetched from other node should go through validation before inserting in storage.
2. Trace for any alternate path that node receive data, eg. restart, network partition.

## [M - Permanently stuck funds when validator is burned with a balance greater than the stakeAmount](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/888)

### Summary

1. When slashing a validators, the burn function zeros out balance before unstaking, causing rewards to become permanently trapped in the contract

```solidity
// State update in question
state_variable = 0;
```

### What went wrong and how to spot the bug

1. Wasn't aware that slashing should only affect validator's initial stake, but the entire amount including reward.
2. Take note of documentation, whether it only slashing initial amount or the entire amount including reward.

## [Incorrect calculation of max rejected stake leads to premature batch rejection](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/722)

### Summary

1. Quorum waiter incorrectly calculates the max amount of stake, missing the proposer stake in the total stake, causing the actual `max_rejected_stake`
to be low.

### What went wrong and how to spot them

1. `available_stake` missed out proposer's amount, it only include stake amount from other validators
2. Didn't keep track of what fields each struct holds. `Authority & committee` Didn't understand that the function only loops through other validator for the available stake, but didn't get the proposer's stake. Hence the amount is missed out.
3. Understand the flow of proposer during each round.

## [Database disconnection when catching up gas accumulator mid-epoch causes node failure](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/699)

### @TODO

## [Observer crash during peer rotation when peer list is empty](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/694)

### Summary

1. The network enforces a restrictive peer limit defined by `config.target_num_peers`, `prune_connected_peers()` is called to aggressively disconnect peers.
2. Inadvertently creates a race condition where the `prune_connected_peers()` could remove all peers leaving the `connected_peers` empty.
3. And when `rotate_left` is called on an empty `VecDeque` list, it causes panic and the node to crash

### How to spot them

1. Know the data structure
    - VecDeque is a growable ring buffer.
    - Insert/remove at head or tail is O(1), but calling methods like `rotate_left` assumes non-empty.

2. Trace lifecycle of the collection
    - Start (bootstrap phase): can it be empty before the first peer joins?
    - Runtime (steady state): can pruning, disconnection, or errors reduce it to zero?
    - Can pruning + disconnects happen together → empty VecDeque?

3. Check assumptions before operations
    - Any operation that requires len > 0 (like rotate_left, indexing, popping, front/back) should be guarded.
    - Add defensive checks (if !is_empty() { ... }).

4. Think in race conditions
    - Peer list can shrink between scheduling and execution of rotation.
    - Even if you checked len > 0, a concurrent prune/disconnect could empty it before rotate_left.
