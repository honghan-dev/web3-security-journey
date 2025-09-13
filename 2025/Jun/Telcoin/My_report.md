# ConsensusRegistry::delegateStake incorrectly updates the delegator and validatorAddress in delegations mapping

# Summary

ConsensusRegistry::delegateStake incorrectly assigns the validatorAddress and delegator fields in the Delegation struct, resulting in reward attribution logic breaking downstream.

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
