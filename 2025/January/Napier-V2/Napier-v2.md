# Napier V2 - Finding report

## High Severity

1. [Wrong reward distribution formula, YT holders can get more rewards for themselves by just collecting accrued YBT yield](https://cantina.xyz/code/58cd719b-9004-4eca-a113-41d1691c0711/findings/141)
  -  The reward calculation error lies in distributing rewards based solely on YT balance / Total YT supply while ignoring that claimed YBT generates additional rewards for the claimer. This creates a mathematical imbalance where early claimers double-dip: receiving their share of protocol rewards while also collecting rewards from their personally-held YBT.

## Medium Severity

### 1. [Wrong initial TwoCurve liquidity computation](https://cantina.xyz/code/58cd719b-9004-4eca-a113-41d1691c0711/findings/54)
  - Issue was due to asset token and vault token decimals mismatch.
  - When adding initial liquidity to `TwoCurveCryptoNG` pools, the ZapMathLib
  computeSharesToTwoCrypto function fails to account for different decimal precisions between vaults and their underlying assets.
  - **Root Cause**
  The function passes a value in asset decimals to previewSupply(), which expects vault decimals. This mismatch causes the calculation to produce effectively zero shares when these decimals differ (e.g., USDC with 6 decimals in an 18-decimal Morpho vault)

### 2. [Collect front run permit](https://cantina.xyz/code/58cd719b-9004-4eca-a113-41d1691c0711/findings/70)

- Issue of front running with `Permit` [EIP-2612](https://eips.ethereum.org/EIPS/eip-2612)

1. `PrincipalToken` - A token contract with a `permitCollector` function that uses EIP-712 signatures
2. `TwoCryptoZap` - A router contract that calls `permitCollector` as part of its `collectWithPermit` function

Here's how the vulnerability works:

1. A user creates a signature to approve the `TwoCryptoZap` contract as a collector for their `PrincipalToken`
2. The user calls `collectWithPermit` on `TwoCryptoZap`, which internally calls:
   - `_permitCollector` to set up the approval 
   - `PrincipalToken.collect` to actually collect the tokens

3. An attacker can monitor the mempool, extract the permit signature, and frontrun by:
   - Directly calling `permitCollector` on the `PrincipalToken` contract
   - This increments the user's nonce
   
4. When the user's original transaction executes:
   - The `_permitCollector` call fails because the nonce has already been used
   - The entire `collectWithPermit` function reverts
   - The user's collection action never happens

### The Recommended Fix

Implementing a try/catch pattern:

```solidity
function _permitCollector(CollectInput calldata input, address owner) internal {
    if (input.permit.deadline > 0) {
        try PrincipalToken(input.principalToken).permitCollector(
            owner, address(this), input.permit.deadline, input.permit.v, input.permit.r, input.permit.s
        ) {
            // Permit succeeded
        } catch {
            // Check if we already have collector approval before failing
            if (!PrincipalToken(input.principalToken).isApprovedCollector(owner, address(this))) {
                revert CollectorNotApproved();
            }
        }
    }
}
```