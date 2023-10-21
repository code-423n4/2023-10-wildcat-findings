LOW-1 Use of `balanceOf(address(this))` for accounting purposes

`totalAssets()` function uses IERC20(asset).balanceOf(address(this)) which is a really bad practice.

Even if right now it doesn't break the accounting of `withdrawableFees`, `borrowableAssets`, `availableLiquidity`, given that everyone could influence it by arbitrarily sending the underlying token ... I propose to limit that by computing the `totalAssets` value using other components like deposits and withdraws. 

This design choice originates from the lack of a specific repay method for the borrower.

Recommendation: Track the `totalAssets` using inflows and outflows and create a specific repay function for the borrower in order to negate any possible external influence. 