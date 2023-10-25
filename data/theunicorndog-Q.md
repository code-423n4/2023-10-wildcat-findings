# Lines of code

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L169


# Vulnerability details

## Impact
The interest is calculated using calculateLinearInterestFromBips(). i.e. \$100 becomes \$110 in one year. But the codebase calls WildcatMarketBase._getUpdatedState() on an adhoc basis, the scalefactor becomes messed up. Because the code treats it as a geometric compounding series. For an example, 50% interest rate called once is not the same as 10% called 5 times.

A malicious lender may do small deposits/withdrawal 10 times a day (to force a market updateState() for 10 years and get the borrower to be liable for an extra ~36% more interest for a 10% APR interest loan.

## Proof of Concept
```
    function test_updateStateProofOfConcept() external {
        _deposit(alice, 1e18);
        MarketState memory state2 = pendingState();
        updateState(state2);
        market.updateState();

        console.log("BEFORE:");
        console.log("scaleFactor: ", state2.scaleFactor);
        console.log(
            "lastInterestAccruedTimestamp: ",
            state2.lastInterestAccruedTimestamp
        );

        MarketState memory state = pendingState();

        for (uint256 i = 0; i < 36500; ++i) {
            fastForward(0.1 days);
            state = pendingState();
            updateState(state);
            market.updateState();
        }

        console.log("AFTER:");
        console.log("scaleFactor: ", state.scaleFactor);
        console.log(
            "lastInterestAccruedTimestamp: ",
            state.lastInterestAccruedTimestamp
        );
    }
```

```
Logs:
  BEFORE:
  scaleFactor:  1000000000000000000000000000
  lastInterestAccruedTimestamp:  1
  AFTER:
  scaleFactor:  2718244592656813831737160257
  lastInterestAccruedTimestamp:  315360001
```

## Tools Used
VSCode

## Recommended Mitigation Steps

Just modify the code to treat the entire interest rate exercise as a linear and non-compounding, i.e. an arithmetic series, scaleFactor should be ``1 + 0.1 + 0.1 + 0.1 + 0.1 + 0.1``. 

BEFORE:
```
    uint256 scaleFactorDelta = prevScaleFactor.rayMul(baseInterestRay + delinquencyFeeRay);
```

AFTER:
```
    uint256 scaleFactorDelta = baseInterestRay + delinquencyFeeRay;
```

With the fix above, the output log shows a 1999999999999999999999999000, which gives a much more accurate of 2x scaleFactor after ten years of 10% linear APR interest ``(1+0.1*10)``. This is independent of how many times WildcatMarketBase._getUpdatedState() is called.
```
Logs:
  BEFORE:
  scaleFactor:  1000000000000000000000000000
  lastInterestAccruedTimestamp:  1
  AFTER:
  scaleFactor:  1999999999999999999999999000
  lastInterestAccruedTimestamp:  315360001
```

## Afternote
I read from discord that the interest rate is meant to be compounding. In that case, a geometric compounding series should be adopted. But this needs to be done very carefully since the interest rate is handled in a linear manner in the current code. Some kind of mathematical exponential interest function similar to the AAVE's calculateCompoundInterest should be adopted instead. 

I have chatted with functi0nZer0, and this is the intended design. The _getUpdatedState() will be called probably once a month as the base case, and any more frequent that that, does not really change the interest owed. Although it is quite odd to have interest owed to be different due to the number of times _getUpdatedState() is called, it is still acceptable. I calculated 1.68% difference in interest owed for continuous compounding versus a monthly compounding for a extreme 50% APR loan. 



## Assessed type

Math