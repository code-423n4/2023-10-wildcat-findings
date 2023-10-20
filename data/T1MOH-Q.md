## 1. Low. `setAnnualInterestBips()` will decrease reserveRatio for the next 2 weeks if it was above 90%
### Vulnerability details
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L468-L488

If borrower decreases APR, reserveRatio is set to 90% for the next 2 weeks. However it will decrease reserveRatio, if it was higher than 90% before.

### Recommended Mitigation Steps
Set reserveRatio to 90% only if it was lower before
```diff
  function setAnnualInterestBips(
    address market,
    uint16 annualInterestBips
  ) external virtual onlyBorrower onlyControlledMarket(market) {
+   uint128 currentReserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips());
    // If borrower is reducing the interest rate, increase the reserve
    // ratio for the next two weeks.
-   if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {
+   if (annualInterestBips < WildcatMarket(market).annualInterestBips() && currentReserveRatioBips < 9_000) {
      TemporaryReserveRatio storage tmp = temporaryExcessReserveRatio[market];

      if (tmp.expiry == 0) {
-       tmp.reserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips());
+       tmp.reserveRatioBips = currentReserveRatioBips;

        // Require 90% liquidity coverage for the next 2 weeks
        WildcatMarket(market).setReserveRatioBips(9000);
      }

      tmp.expiry = uint128(block.timestamp + 2 weeks);
    }

    WildcatMarket(market).setAnnualInterestBips(annualInterestBips);
  }
```

## 2. Low. Allow to execute withdrawal before expiry if market is closed
### Vulnerability details
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketWithdrawals.sol#L141

There is no sense to wait expiry on current batch if market is closed

### Recommended Mitigation Steps
Refactor condition:
```diff
  function executeWithdrawal(
    address accountAddress,
    uint32 expiry
  ) external nonReentrant returns (uint256) {
+   MarketState memory state = _getUpdatedState();
-   if (expiry > block.timestamp) {
+   if (expiry > block.timestamp && !state.isClosed) {
      revert WithdrawalBatchNotExpired();
    }
-   MarketState memory state = _getUpdatedState();
    ...
  }
```
