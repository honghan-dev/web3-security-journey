# Napier V2 - Finding report

## High Severity

1. [Wrong reward distribution formula, YT holders can get more rewards for themselves by just collecting accrued YBT yield](https://cantina.xyz/code/58cd719b-9004-4eca-a113-41d1691c0711/findings/141)
  -  The reward calculation error lies in distributing rewards based solely on YT balance / Total YT supply while ignoring that claimed YBT generates additional rewards for the claimer. This creates a mathematical imbalance where early claimers double-dip: receiving their share of protocol rewards while also collecting rewards from their personally-held YBT.

## Medium Severity

1. [Wrong initial TwoCurve liquidity computation](https://cantina.xyz/code/58cd719b-9004-4eca-a113-41d1691c0711/findings/54)
  - Issue was due to asset token and vault token decimals mismatch.
  - When adding initial liquidity to `TwoCurveCryptoNG` pools, the ZapMathLib
  computeSharesToTwoCrypto function fails to account for different decimal precisions between vaults and their underlying assets.
  - **Root Cause**
  The function passes a value in asset decimals to previewSupply(), which expects vault decimals. This mismatch causes the calculation to produce effectively zero shares when these decimals differ (e.g., USDC with 6 decimals in an 18-decimal Morpho vault)

2. [Collect front run permit](https://cantina.xyz/code/58cd719b-9004-4eca-a113-41d1691c0711/findings/70)