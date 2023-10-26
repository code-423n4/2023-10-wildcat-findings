# Summary

| Id   | Title                                                                           |
|------|---------------------------------------------------------------------------------|
| L-01 | Interest rate constraints are not enforced when market interest rate is changed |
| L-02 | Market maximum total supply cannot be changed                                   |

## [L-01] Interest rate constraints are not enforced when market interest rate is changed 

When a market is created, the `MinimumAnnualInterestBips` and `MaximumAnnualInterestBips` constraints for the annual interest rate are enforced.
For existing markets, the annual interest rate can be changed by the borrower using the `WildcatMarketController.setAnnualInterestBips()` function. In this case the constraints are not enforced.

Given that the constraints are made publicly available by the `WildcatMarketController.getParameterConstraints()` function, a lender might assume that the interest rate will always stay within those constraints (in particular that the interest rate will not be lower than the announced minimum rate) during the lifetime of a market. This assumption is broken if the interest rate is lowered below the announced minimum.

```solidity
  /**
   * @dev Modify the interest rate for a market.
   * If the new interest rate is lower than the current interest rate,
   * the reserve ratio is set to 90% for the next two weeks.
   */
  function setAnnualInterestBips(
    address market,
    uint16 annualInterestBips
  ) external virtual onlyBorrower onlyControlledMarket(market) {
    // If borrower is reducing the interest rate, increase the reserve
    // ratio for the next two weeks.
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
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L468

## [L-02] Market maximum total supply cannot be changed

According to the protocol's documentation (https://wildcat-protocol.gitbook.io/wildcat/using-wildcat/day-to-day-usage/borrowers#altering-capacity), a market's capacity (maximum total supply) can be adjusted by the borrower.

The function `WildcatMarketConfig.setMaxTotalSupply()` supposedly implements this functionality. However, the function has the `onlyController` modifier, making it callable only by the controller. The controller, however, has no code to call this function. Thus the function is effectively uncallable.

```solidity
function setMaxTotalSupply(uint256 _maxTotalSupply) external onlyController nonReentrant {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L134