# Other findings

## [M - Non-persistent header tracking leads to transaction loss on node restart](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/1177)

### Note

- proposer maintains proposed headers in memory (`proposed_headers` map) but only persists the latest header to database (`last_proposed` table).
- When a node restarts, all previously proposed but uncommitted headers are lost from memory and cannot be recovered, causing their transaction batches to never be rescheduled for inclusion.
- This creates a responsibility gap where the proposer forgets its commitment to retry failed proposals, potentially leading to transaction delays until other proposers pick up the slack.

### How to spot this bug

Architecture understanding

1. Understand the proposer system
2. What Should Be Persistent -> Ask: "What breaks if this is lost on restart?"
3. Critical consensus data must survive restarts, especially in multi-proposer systems where individual node failures affect the whole network.

## [M - Logic error in Proposer's calc_min_delay() function leads to reduced network throughput](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/1104)

### Note

- Issue: Documentation says `whole committee` gets zero delay in odd rounds, but code only gives zero delay to the single leader
- Impact: Reduced network throughput in every odd round (50% of all rounds)
- Root Cause: Function logic contradicts its own NatSpec comments - classic spec vs implementation mismatch

## [M - primary: requested_parents DoS issue](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/1095)

### Note

- Attack: Send headers for far-future rounds (999999) with fake parent certificates
- Impact: Node tracks fake certificate requests forever → memory exhaustion → crash
- Root Cause: No future round validation + garbage collection only removes old entries, not future ones

### Prevention Checklist

🔍 Key Red Flags

- Unbounded collections (HashMap, Vec) that grow from external input
- Missing validation on input size/range (especially temporal bounds)
- Incomplete garbage collection (only removes old, not invalid future data)

### ⚡ Quick Audit Questions

- "What grows without limits?" - Find collections that accumulate data
- "Can malicious input cause unbounded growth?" - Test edge cases
- "Does cleanup handle all invalid cases?" - Check GC completeness
- Golden Rule: External input → Internal storage growth = potential DoS vulnerability

## [M - Transactions Are Removing From The Meme Pool Prematurely Which Results In Transaction Being Lost After Epoch Transitions](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/967)

### Summary

- Bug Type: Premature state cleanup with incomplete recovery mechanism
- Issue: Transactions removed from mempool after batch validation but before blockchain inclusion
- Root Cause: Missing recovery logic to restore pending batches after epoch transitions
- Impact: Validated transactions permanently lost during epoch boundaries
- Timing: Batch validation complete → Epoch transition → New proposer ignores stored batches

### ⚡ Quick Audit Questions

"When is data removed vs when is it actually committed?" - Look for timing gaps
"What happens on restart between these phases?" - Check recovery mechanisms
"Are there intermediate 'success' states that aren't final?" - Map all process phases
"Does persistent storage match what gets recovered?" - Verify storage/recovery consistency

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

# Summary

- Issue: Multiple validation gaps in consensus header sync allow malicious headers to be accepted and processed
- Root Cause: Two-task sync architecture with inconsistent validation - tracking task has weak validation, streaming task has stronger validation but fatal error handling(process stopped when encountered incorrect digest)
- Attack Vector: Send headers with valid signatures but wrong round certificates, malicious timestamps, or incorrect metadata
- Impact: RPC API poisoning, execution engine manipulation, eventual sync termination when honest headers fail validation

### Key Audit Questions

"Are validation rules consistent across all processing paths?" - Compare validation depth between parallel tasks/components
"Does any single bad input permanently disable functionality?" - Check for fatal error handling vs graceful retry mechanisms
"Can validation be bypassed through alternative entry points?" - Map all ways external data enters the system and verify equivalent security checks

## [M - Workers store gossiped batches without validation](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/761)

## Summary

- Issue: Batches fetched via gossip are stored without validation, while directly reported batches are properly validated
- Root Cause: Two different code paths for batch storage with inconsistent validation requirements
- Attack Vector: Gossip invalid batch digests to trigger automatic fetching and storage of malformed batches
- Impact: Storage DoS (disk filled with invalid data) and network DoS (bandwidth wasted on invalid batch requests)

### Key Audit Questions

- "Do all data entry points have equivalent validation?" - Map every way data can enter storage and verify validation consistency
- "Can automated/triggered actions bypass critical checks?" - Check gossip, sync, and automatic mechanisms for validation gaps
- "Are there alternative paths to the same functionality?" - Identify if bypass routes exist around main validation logic
