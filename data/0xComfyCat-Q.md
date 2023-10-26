# Low
## It is not trivial to withdraw 100%
It is not trivial to withdraw 100% because the input amount of `queueWithdrawal` will be less than actual market token balance of lender when the transaction is executed due to interest accrual. Consider add function `queueWithdrawalAll`

## If borrower set reserve ratio higher than 90% they can reduce it to 90% consistently by calling `setAnnualInterestBips` with lower interest
`setAnnualInterestBips` always fixed reserveRatio to 90%. If somehow borrower deploy a market with 95% reserveRatio they can lower it to 90% consistently by calling `setAnnualInterestBips` every 2 weeks.

# Non-critical
## `annualInterestBips` is too restrictive
`annualInterestBips` is hard-capped at 100% in market config contract `setReserveRatioBips`. Borrower might want to deploy a market with higher interest rate to attract lenders.