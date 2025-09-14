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
