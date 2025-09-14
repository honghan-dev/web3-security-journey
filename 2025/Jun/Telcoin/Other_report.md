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
