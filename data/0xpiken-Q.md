## [01] [`processUnpaidWithdrawalBatch()`]([https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L57-L63](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L190-L214)) can be be reverted due to lack of available liquidity
Use `satSub` instead:
```diff
  function processUnpaidWithdrawalBatch() external nonReentrant {
    MarketState memory state = _getUpdatedState();

    // Get the next unpaid batch timestamp from storage (reverts if none)
    uint32 expiry = _withdrawalData.unpaidBatches.first();

    // Cache batch data in memory
    WithdrawalBatch memory batch = _withdrawalData.batches[expiry];

    // Calculate assets available to process the batch
-   uint256 availableLiquidity = totalAssets() -
-     (state.normalizedUnclaimedWithdrawals + state.accruedProtocolFees);
+   uint256 availableLiquidity = totalAssets().satSub(state.normalizedUnclaimedWithdrawals + state.accruedProtocolFees);

    _applyWithdrawalBatchPayment(batch, state, expiry, availableLiquidity);

    // Remove batch from unpaid set if fully paid
    if (batch.scaledTotalAmount == batch.scaledAmountBurned) {
      _withdrawalData.unpaidBatches.shift();
      emit WithdrawalBatchClosed(expiry);
    }

    // Update stored batch
    _withdrawalData.batches[expiry] = batch;
    _writeState(state);
  }
```

## [02] The value of `annualInterestBips` should be strictly limited to a range.
Check if the value is between `MinimumAnnualInterestBips` and `MaximumAnnualInterestBips` when calling [`WildcatMarketController#setAnnualInterestBips()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L468-L488):
```diff
  function setAnnualInterestBips(
    address market,
    uint16 annualInterestBips
  ) external virtual onlyBorrower onlyControlledMarket(market) {
    // If borrower is reducing the interest rate, increase the reserve
    // ratio for the next two weeks.
+   assertValueInRange(
+     annualInterestBips,
+     MinimumAnnualInterestBips,
+     MaximumAnnualInterestBips,
+     AnnualInterestBipsOutOfBounds.selector
+   );
    if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {
      TemporaryReserveRatio storage tmp = temporaryExcessReserveRatio[market];

      if (tmp.expiry == 0) {
        tmp.reserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips());

        // Require 90% liquidity coverage for the next 2 weeks
        WildcatMarket(market).setReserveRatioBips(9000);
      }

      tmp.expiry = uint128(block.timestamp + 2 weeks);
    }

    WildcatMarket(market).setAnnualInterestBips(annualInterestBips);
  }
```
## [03] `WildcatMarketToken` doesn't comply EIP20.
Below is described in [EIP-20](https://eips.ethereum.org/EIPS/eip-20):
> Note Transfers of 0 values MUST be treated as normal transfers and fire the Transfer event.

Change codes as below:
```diff
  function _transfer(address from, address to, uint256 amount) internal virtual {
    MarketState memory state = _getUpdatedState();
    uint104 scaledAmount = state.scaleAmount(amount).toUint104();

-   if (scaledAmount == 0) {
-     revert NullTransferAmount();
-   }
+   if (scaledAmount != 0) {
      Account memory fromAccount = _getAccount(from);
      fromAccount.scaledBalance -= scaledAmount;
      _accounts[from] = fromAccount;

      Account memory toAccount = _getAccount(to);
      toAccount.scaledBalance += scaledAmount;
      _accounts[to] = toAccount;
+   }

    _writeState(state);
    emit Transfer(from, to, amount);
  }
```