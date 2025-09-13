# Other findings

## [M - Non-persistent header tracking leads to transaction loss on node restart](https://cantina.xyz/code/26d5255b-6f68-46cf-be55-81dd565d9d16/findings/1177)

### How to spot this bug

Architecture understanding

1. Understand the proposer system
2. What Should Be Persistent -> Ask: "What breaks if this is lost on restart?"
3. Critical consensus data must survive restarts, especially in multi-proposer systems where individual node failures affect the whole network.
