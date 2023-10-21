### LOW-1 Use of `balanceOf(address(this))` for accounting purposes

`totalAssets()` function uses IERC20(asset).balanceOf(address(this)) which is a really bad practice.

Even if right now it doesn't break the accounting of `withdrawableFees`, `borrowableAssets`, `availableLiquidity`, given that everyone could influence it by arbitrarily sending the underlying token ... I propose to limit that by computing the `totalAssets` value using other components like deposits and withdraws. 

This design choice originates from the lack of a specific repay method for the borrower.

**Recommendation**: Track the `totalAssets` using inflows and outflows and create a specific repay function for the borrower in order to negate any possible external influence. 



### LOW-2 `MinimumDelinquencyGracePeriod` and `MaximumDelinquencyGracePeriod` should have hardcoded limits.

Those two variables have a significant influence on the flow of the borrower/lender relation. A `MaximumDelinquencyGracePeriod` of 356 days gives too much leniency to the borrower, rendering the delinquency penalty useless, while a `MinimumDelinquencyGracePeriod` of 0 will make lenders game the system in order to have the delinquency penalty active 100% of the time. 

**Recommendation**: Set a hard limit of let's say 1 day for the `MinimumDelinquencyGracePeriod` and 7-10 days for the `MaximumDelinquencyGracePeriod`