## GAS1: State variables that are used multiple times in a function should be cached in stack variables 
```diff
File: WildcatSanctionsEscrow.sol
  function releaseEscrow() public override {
    if (!canReleaseEscrow()) revert CanNotReleaseEscrow();


    uint256 amount = balance();
    
+   address _account = account;
+   address _asset = asset;

-   IERC20(asset).transfer(account, amount);
+   IERC20(_asset).transfer(_account, amount);

-   emit EscrowReleased(account, asset, amount);
+   emit EscrowReleased(_account, _asset, amount);
  }
```
`Overall gas change: -194421 (-0.230%)`


## GAS2: Pack struct
```solidity
File: MarketState.sol
	struct MarketState {
		uint128 maxTotalSupply;                 // 16 bytes
		uint128 accruedProtocolFees;            // 16 bytes

		uint128 normalizedUnclaimedWithdrawals; // 16 bytes
		uint104 scaledTotalSupply;              // 13 bytes
		uint16 annualInterestBips;              // 2 bytes
		bool isClosed;                          // 1 byte
		
		uint104 scaledPendingWithdrawals;       // 13 bytes
		uint112 scaleFactor;                    // 14 bytes
		uint16 reserveRatioBips;                // 2 bytes
		bool isDelinquent;                      // 1 byte

		uint32 pendingWithdrawalExpiry;         // 4 bytes
		uint32 timeDelinquent;                  // 4 bytes
		uint32 lastInterestAccruedTimestamp;    // 4 bytes
	}
```

## GAS3: closeMarket() function can be optimized
```diff
File: WildcatMarket.sol
  function closeMarket() external onlyController nonReentrant {
    MarketState memory state = _getUpdatedState();
+    if (_withdrawalData.unpaidBatches.length() > 0) {
+      revert CloseMarketWithUnpaidWithdrawals();
+    }
    state.annualInterestBips = 0;
    state.isClosed = true;
    state.reserveRatioBips = 0;
-    if (_withdrawalData.unpaidBatches.length() > 0) {
-      revert CloseMarketWithUnpaidWithdrawals();
-    }
    uint256 currentlyHeld = totalAssets();
    uint256 totalDebts = state.totalDebts();
    if (currentlyHeld < totalDebts) {
      // Transfer remaining debts from borrower
      asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);
    } else if (currentlyHeld > totalDebts) {
      // Transfer excess assets to borrower
      asset.safeTransfer(borrower, currentlyHeld - totalDebts);
    }
    _writeState(state);
    emit MarketClosed(block.timestamp);
  }
```
`Overall gas change: -1524 (-0.002%)`

## GAS4: setAnnualInterestBips() function can be optimized
```solidity
File: WildcatMarketConfig.sol
  function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant {
+   if (_annualInterestBips > BIP) {
+     revert InterestRateTooHigh();
+   }
    MarketState memory state = _getUpdatedState();

-   if (_annualInterestBips > BIP) {
-     revert InterestRateTooHigh();
-   }

    state.annualInterestBips = _annualInterestBips;
    _writeState(state);
    emit AnnualInterestBipsUpdated(_annualInterestBips);
  }
```
`Overall gas change: -7567 (-0.009%)`

## GAS6: stunningReversal() function can be optimized
```diff
File: WildcatMarketConfig.sol
  function stunningReversal(address accountAddress) external nonReentrant {
+   if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
+     revert NotReversedOrStunning();
+   }
    Account memory account = _accounts[accountAddress];
    if (account.approval != AuthRole.Blocked) {
      revert AccountNotBlocked();
    }


-   if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
-     revert NotReversedOrStunning();
-   }


    account.approval = AuthRole.Null;
    emit AuthorizationStatusUpdated(accountAddress, account.approval);


    _accounts[accountAddress] = account;
  }
```

## GAS7: queueWithdrawal() function can be optimized
```diff
File: WildcatMarketWithdrawals.sol
  function queueWithdrawal(uint256 amount) external nonReentrant {
    MarketState memory state = _getUpdatedState();

+   uint104 scaledAmount = state.scaleAmount(amount).toUint104();
+   if (scaledAmount == 0) {
+     revert NullBurnAmount();
+   }

    // Cache account data and revert if not authorized to withdraw.
    Account memory account = _getAccountWithRole(msg.sender, AuthRole.WithdrawOnly);


-   uint104 scaledAmount = state.scaleAmount(amount).toUint104();
-   if (scaledAmount == 0) {
-     revert NullBurnAmount();
-   }


    // Reduce caller's balance and emit transfer event.
    account.scaledBalance -= scaledAmount;
    _accounts[msg.sender] = account;
    emit Transfer(msg.sender, address(this), amount);


    // If there is no pending withdrawal batch, create a new one.
    if (state.pendingWithdrawalExpiry == 0) {
      state.pendingWithdrawalExpiry = uint32(block.timestamp + withdrawalBatchDuration);
      emit WithdrawalBatchCreated(state.pendingWithdrawalExpiry);
    }
    // Cache batch expiry on the stack for gas savings.
    uint32 expiry = state.pendingWithdrawalExpiry;


    WithdrawalBatch memory batch = _withdrawalData.batches[expiry];


    // Add scaled withdrawal amount to account withdrawal status, withdrawal batch and market state.
    _withdrawalData.accountStatuses[expiry][msg.sender].scaledAmount += scaledAmount;
    batch.scaledTotalAmount += scaledAmount;
    state.scaledPendingWithdrawals += scaledAmount;


    emit WithdrawalQueued(expiry, msg.sender, scaledAmount);


    // Burn as much of the withdrawal batch as possible with available liquidity.
    uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());
    if (availableLiquidity > 0) {
      _applyWithdrawalBatchPayment(batch, state, expiry, availableLiquidity);
    }


    // Update stored batch data
    _withdrawalData.batches[expiry] = batch;


    // Update stored state
    _writeState(state);
  }
```
`Overall gas change: -9258 (-0.011%)`

## GAS8: executeWithdrawal() function can be optimized
```diff
File: WildcatMarketWithdrawals.sol
  function executeWithdrawal(
    address accountAddress,
    uint32 expiry
  ) external nonReentrant returns (uint256) {
    if (expiry > block.timestamp) {
      revert WithdrawalBatchNotExpired();
    }
    MarketState memory state = _getUpdatedState();


    WithdrawalBatch memory batch = _withdrawalData.batches[expiry];
    AccountWithdrawalStatus storage status = _withdrawalData.accountStatuses[expiry][
      accountAddress
    ];


    uint128 newTotalWithdrawn = uint128(
      MathUtils.mulDiv(batch.normalizedAmountPaid, status.scaledAmount, batch.scaledTotalAmount)
    );


    uint128 normalizedAmountWithdrawn = newTotalWithdrawn - status.normalizedAmountWithdrawn;

+   if (normalizedAmountWithdrawn == 0) {
+     revert NullWithdrawalAmount();
+   }

    status.normalizedAmountWithdrawn = newTotalWithdrawn;
    state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;


-   if (normalizedAmountWithdrawn == 0) {
-     revert NullWithdrawalAmount();
-   }


    if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
      _blockAccount(state, accountAddress);
      address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
        accountAddress,
        borrower,
        address(asset)
      );
      asset.safeTransfer(escrow, normalizedAmountWithdrawn);
      emit SanctionedAccountWithdrawalSentToEscrow(
        accountAddress,
        escrow,
        expiry,
        normalizedAmountWithdrawn
      );
    } else {
      asset.safeTransfer(accountAddress, normalizedAmountWithdrawn);
    }


    emit WithdrawalExecuted(expiry, accountAddress, normalizedAmountWithdrawn);


    // Update stored state
    _writeState(state);


    return normalizedAmountWithdrawn;
  }
```
`gas: -413 (-0.070%)`

## GAS9: The function `MathUtils.calculateLinearInterestFromBips()` is duplicated and can be deleted
The function `calculateLinearInterestFromBips()` is duplicated in the following files in `FeeMath.sol` and `MathUtils.sol`:
```solidity
File: FeeMath.sol
20:     function calculateLinearInterestFromBips(uint256 rateBip, uint256 timeDelta)
21:         internal
22:         pure
23:         returns (uint256 result)
24:     {
25:         uint256 rate = rateBip.bipToRay();
26:         uint256 accumulatedInterestRay = rate * timeDelta;
27:         unchecked {
28:             return accumulatedInterestRay / SECONDS_IN_365_DAYS;
29:         }
30:     }
```
```solidity
File: MathUtils.sol
30:     function calculateLinearInterestFromBips(
31:         uint256 rateBip,
32:         uint256 timeDelta
33:     ) internal pure returns (uint256 result) {
34:         uint256 rate = rateBip.bipToRay();
35:         uint256 accumulatedInterestRay = rate * timeDelta;
36:         unchecked {
37:             return accumulatedInterestRay / SECONDS_IN_365_DAYS;
38:         }
39:     }
```
The `MathUtils.calculateLinearInterestFromBips()` function can be deleted to save gas and optimize the code. Don't forget to update the references to the function in the code by replacing them with the `FeeMath.calculateLinearInterestFromBips()` function instead of `MathUtils.calculateLinearInterestFromBips() function`.

## GAS10: The function `MathUtils._applyWithdrawalBatchPayment()` can be optimized
```diff
File: WildcatMarketBase.sol
  function _applyWithdrawalBatchPayment(
    WithdrawalBatch memory batch,
    MarketState memory state,
    uint32 expiry,
    uint256 availableLiquidity
  ) internal {
+   uint104 scaledAmountOwed = batch.scaledTotalAmount - batch.scaledAmountBurned;
+   // Do nothing if batch is already paid
+   if (scaledAmountOwed == 0) {
+     return;
+   }
    uint104 scaledAvailableLiquidity = state.scaleAmount(availableLiquidity).toUint104();
-   uint104 scaledAmountOwed = batch.scaledTotalAmount - batch.scaledAmountBurned;
-   // Do nothing if batch is already paid
-   if (scaledAmountOwed == 0) {
-     return;
-   }
    uint104 scaledAmountBurned = uint104(MathUtils.min(scaledAvailableLiquidity, scaledAmountOwed));
    uint128 normalizedAmountPaid = state.normalizeAmount(scaledAmountBurned).toUint128();


    batch.scaledAmountBurned += scaledAmountBurned;
    batch.normalizedAmountPaid += normalizedAmountPaid;
    state.scaledPendingWithdrawals -= scaledAmountBurned;


    // Update normalizedUnclaimedWithdrawals so the tokens are only accessible for withdrawals.
    state.normalizedUnclaimedWithdrawals += normalizedAmountPaid;


    // Burn market tokens to stop interest accrual upon withdrawal payment.
    state.scaledTotalSupply -= scaledAmountBurned;


    // Emit transfer for external trackers to indicate burn.
    emit Transfer(address(this), address(0), normalizedAmountPaid);
    emit WithdrawalBatchPayment(expiry, scaledAmountBurned, normalizedAmountPaid);
  }
```
`Overall gas change: -5380 (-0.006%)`