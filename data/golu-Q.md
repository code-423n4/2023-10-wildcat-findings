## Wrong Amount of Loan Interest is Calculated

## Relevant GitHub Links

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L19C1-L29C1

## Summary

The vulnerability involves miscalculating interest rates in a lending protocol's codebase. The code mistakenly treats the interestRate  meant for per-second calculations as if it's per-year. This results in underestimated interest due to incorrect division by 365 days. Precision loss also arises from omitting 6 hours in a year. Additionally, using 3 decimals for interest rates compounds precision issues. Corrective actions involve adjusting calculations based on intended time units.

## Vulnerability Details

In **2023-10-wildcat/src/libraries/MathUtils.sol and 2023-10-wildcat/src/libraries/FeeMath.sol** library contract 
 for interest calculation accumulatedInterestRay variable define and it return ** accumulatedInterestRay / SECONDS_IN_365_DAYS ** 

We can see that for calculating the interest, the function `return` We can see that for calculating the interest, the `365days`.

We know that the `interestRate` shows the interest rate of a loan per second. Also, we know that 365 days have 31,536,000 seconds. So, the interest rate per 365 days should be 31,536,000 times more than the interest rate per second. However, in the given code, both interest rate per second and interest rate per 365 days, are used interchangeably.

Also, note that 1 year is equal to 365 days and 6 hours. A lack of 6 hours in the computation can lead to precision loss.

The final note regarding calculating the interest is that 3 decimals is way small and can lead to precision loss easily.

## Impact

This can lead to a huge underestimation of the interest on a loan. Consider the interest rate per second is 10^-4. Consider three decimals considered for `intrestRate`, this value should be equal to one. Consider there is a loan with the value of $10,000, which has been given to the borrower for 365 days.

## Recommendations
 ** accumulatedInterestRay / SECONDS_IN_365_DAYS + `6 hours` **.






