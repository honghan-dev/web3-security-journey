# Table of contents

- ### [Contest Summary](#contest-summary)

- ### [Results Summary](#results-summary)

- ## Medium Risk Findings

  - [M-01. `AaveAssetHandler::executeUserFlashLoan` incorrect flashloan amount causes withdrawal reverts with multiple borrowings](#M-01)

- ## Low Risk Findings

  - *None reported*

- ## Informational

  - *None reported*

# Medium Findings

## <a id='M-01'></a>M-01 `AaveAssetHandler::executeUserFlashloan` incorrect `flashloan` amount causing user can't withdrawal if more than 1 borrowing

### Summary

User can't withdraw their desired amount because AaveAssetHandler::executeUserFlashLoan function only consider the first flashloanAmount[0] instead of totalFlashAmount for flash loan via Aave. Therefore user can't withdraw their fund when there is more than 1 borrowing.

### Finding Description

In the AaveAssetHandler::executeUserFlashLoan, it takes the total flashloan amount for repayment. However, in the function, it only takes the 1st flashloanAmount instead of the totalFlashAmount, which will cause the user's withdrawal to revert when the portfolio has more than 1 borrowing, because of insufficient flashloan amount to cover more than 1 borrowing.

```solidity
function executeUserFlashLoan(
    // code omitted //
  ) external override {
    // code omitted //

    for (uint256 i; i < borrowedLength; ) {
      address _underlyingToken = IAaveToken(borrowedTokens[i])
        .UNDERLYING_ASSET_ADDRESS();
      (, , uint currentVariableDebt, , , , , , ) = IPoolDataProvider(
        DATA_PROVIDER_ADDRESS
      ).getUserReserveData(_underlyingToken, _vault);
      underlying[i] = _underlyingToken; // Get the underlying asset for the borrowed token
      tokenBalance[i] =
        (currentVariableDebt * _portfolioTokenAmount) /
        _totalSupply; // Calculate the portion of the debt to repay
      // @audit the total flashloan amount to take
@>   totalFlashAmount += repayData._flashLoanAmount[i]; // Accumulate the total flash loan amount
      unchecked {
        ++i;
      }
    }

    // code omitted //

    // Initiate the flash loan from the Algebra pool
    address[] memory assets = new address[](1);
    assets[0] = repayData._flashLoanToken;
    uint256[] memory amounts = new uint256[](1);
   // @audit only account the 1st flashloan amount, instead of the totalFlashAmount
@>  amounts[0] = repayData._flashLoanAmount[0];

   // code omitted //

    IAavePool(repayData._token0).flashLoan(
      receiver,
      assets,
@>  amounts, // Flashloan amount
      interestRateModes,
      receiver,
      abi.encode(flashData),
      0
    );
  }
```

Subsequent encoded swap call will revert because of insufficient `flashloan` amount to proceed

```solidity
function swapAndTransferTransactions(
    // code omitted //
  )
  // code omitted //
  {
    uint256 tokenLength = flashData.debtToken.length; // Get the number of debt tokens
    transactions = new MultiTransaction[](tokenLength * 2); // Initialize the transactions array
    uint count;

    // Loop through the debt tokens to handle swaps and transfers
    // @audit insufficient flashloan amount to cover more than 1 debt token
    for (uint i; i < tokenLength; ) {
      // Check if the flash loan token is different from the debt token
      if (flashData.flashLoanToken != flashData.debtToken[i]) {
        // Transfer the flash loan token to the solver handler
@>        transactions[count].to = flashData.flashLoanToken;
        transactions[count].txData = abi.encodeWithSelector(
          bytes4(keccak256("transfer(address,uint256)")),
          flashData.solverHandler, // recipient
@>          flashData.flashLoanAmount[i]
            );
            count++;
            }
        }
    }
```

### Impact Explanation

`MEDIUM` - User can't withdraw as their withdrawal will revert due to insufficient flashloan.

### Likelihood Explanation

`MEDIUM` - User can't withdraw when the portfolio has more than 1 borrowing.

### Proof of Concept

Run this mock test:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.17;

import {Test, console} from "forge-std/Test.sol";
import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import {FunctionParameters} from "../../contracts/FunctionParameters.sol";
import {AaveAssetHandler} from "../../contracts/handler/Aave/AaveAssetHandler.sol";
import {IPoolDataProvider} from "../../contracts/handler/Aave/IPoolDataProvider.sol";

contract MockAaveToken {
    function UNDERLYING_ASSET_ADDRESS() external pure returns (address) {
        // return USDC
        return address(0xaf88d065e77c8cC2239327C5EDb3A432268e5831);
    }
}

// Mock Aave Pool
contract MockAavePool {
    // Mock flash loan function that records the requested amount
    function flashLoan(
        address receiver,
        address[] calldata assets,
        uint256[] calldata amounts,
        uint256[] calldata,
        address,
        bytes calldata params,
        uint16
    ) external {
        uint256 totalFlashloanAmount;
        for (uint256 i = 0; i < amounts.length; i++) {
            totalFlashloanAmount += amounts[i];
        }

        console.log("Flashloan amount taken:", totalFlashloanAmount);
    }
}

contract VelvetTest is Test {
    address aavePool;
    MockAaveToken mockAaveToken;
    AaveAssetHandler aaveAssetHandler;
    address vault = makeAddr("vault");

    function setUp() public {
        // 1) deploy mock AssetHandler
        aaveAssetHandler = new AaveAssetHandler();
        aavePool = address(new MockAavePool());
        mockAaveToken = new MockAaveToken();
        address USDC = 0xaf88d065e77c8cC2239327C5EDb3A432268e5831;

        // 3) Mock call from Aave data_provider
        vm.mockCall(
            aaveAssetHandler.DATA_PROVIDER_ADDRESS(),
            abi.encodeWithSelector(
                IPoolDataProvider.getUserReserveData.selector,
                USDC,
                vault
            ),
            // 9 slots: we care only about the 3rd (currentVariableDebt)
            abi.encode(
                uint256(0), // slot 1
                uint256(0), // slot 2
                uint256(1e18), // currentVariableDebt
                uint256(0),
                uint256(0),
                uint256(0),
                uint256(0),
                uint40(0),
                uint16(0)
            )
        );
    }

    function test_AaveAssetHandler() public {
        uint256[] memory flashloanAmount = new uint256[](2);
        flashloanAmount[0] = 1e18;
        flashloanAmount[1] = 1e18;

        address[] memory borrowedTokens = new address[](1);
        borrowedTokens[0] = address(mockAaveToken);

        FunctionParameters.withdrawRepayParams
            memory repayData = FunctionParameters.withdrawRepayParams({
                _factory: makeAddr("factory"),
                _token0: aavePool,
                _token1: makeAddr("token1"),
                _flashLoanToken: aavePool,
                _solverHandler: makeAddr("solver"),
                _swapHandler: makeAddr("swapper"),
                _bufferUnit: 300,
                _flashLoanAmount: flashloanAmount,
                _poolFees: new uint256[](1),
                firstSwapData: new bytes[](1),
                secondSwapData: new bytes[](1),
                isDexRepayment: false
            });

        aaveAssetHandler.executeUserFlashLoan(
            vault,
            address(this),
            1 ether,
            10 ether,
            borrowedTokens,
            repayData
        );

        uint256 totalFlashloanAmount;
        for (uint256 i = 0; i < flashloanAmount.length; i++) {
            totalFlashloanAmount += flashloanAmount[i];
        }

        console.log("Total flashloan amount required:", totalFlashloanAmount);
    }
}
```

```solidity
Test log shows that only 1st flashloanAmount is used for flashloan amount

Logs:
  `Flashloan` amount taken: 1000000000000000000
  Total `flashloan` amount required: 2000000000000000000
```

### Recommendation

Change the flashloanAmount[0] to totalFlashAmount

```solidity
function executeUserFlashLoan(
 // code omitted //
  ) external override {
 // code omitted
   uint256[] memory amounts = new uint256[](1);
-    amounts[0] = repayData._flashLoanAmount[0];
+  amounts[0] = totalFlashAmount;
  // code omitted //
}
```
