# [G-1] Function setAnnualInterestBips() can be more optimized.

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L468-L488

`temporaryExcessReserveRatio[market]` in `setAnnualInterestBips()` function doesn't cache to memory and accesses the state multiple times.

```solidity
function setAnnualInterestBips(
    address market,
    uint16 annualInterestBips
) external virtual onlyBorrower onlyControlledMarket(market) {
    // If borrower is reducing the interest rate, increase the reserve
    // ratio for the next two weeks.
    if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {
        TemporaryReserveRatio storage tmp = temporaryExcessReserveRatio[market];

        if (tmp.expiry == 0) { // @audit First time reading from state
        tmp.reserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips()); // @audit 2nd time reading from state 

        // Require 90% liquidity coverage for the next 2 weeks
        WildcatMarket(market).setReserveRatioBips(9000);
        }

        tmp.expiry = uint128(block.timestamp + 2 weeks); // @audit 3rd time reading from state
    }

    WildcatMarket(market).setAnnualInterestBips(annualInterestBips);
}
```

We can cache `temporaryExcessReserveRatio[market]` to memory since it will be cheaper to use.

```solidity
function setAnnualInterestBips(
    address market,
    uint16 annualInterestBips
) external virtual onlyBorrower onlyControlledMarket(market) {
    // If borrower is reducing the interest rate, increase the reserve
    // ratio for the next two weeks.
    if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {
--      TemporaryReserveRatio storage tmp = temporaryExcessReserveRatio[market];
++      TemporaryReserveRatio memory tmp = temporaryExcessReserveRatio[market];

        if (tmp.expiry == 0) {
            tmp.reserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips());

            // Require 90% liquidity coverage for the next 2 weeks
            WildcatMarket(market).setReserveRatioBips(9000);
        }

        tmp.expiry = uint128(block.timestamp + 2 weeks);
++      temporaryExcessReserveRatio[market] = tmp;
    }

    WildcatMarket(market).setAnnualInterestBips(annualInterestBips);
}
```

Gas benchmarks based on function `setAnnualInterestBips()` which calls our function of interest.

|          | avg      | median   | max      |
|----------|----------|----------|----------|
| Before   | 39422    | 34829    | 71794    |
| After    | 37468    | 25871    | 71843    |

## Recommendation:
Consider to cache `temporaryExcessReserveRatio[market]` to memory.
