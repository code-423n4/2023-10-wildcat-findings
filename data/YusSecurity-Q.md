## [QA-1] Borrowers cannot reduce market capacity below the current total supply

The whitepaper suggests that borrowers should have the ability to reduce capacity irrespective of the current market supply:

> If a borrower decides they want less, this is also fine - dropping the capacity does
not have any impact on assets that are currently in the vault, as the capacity
dictates openness to new deposits. If a lender deposits 100 WETH into a vault
and the borrower decides that actually, that’ll do, they can drop the maximum
capacity to zero if they wish, but they’re still on the hook to ultimately repay
the lender that 100 WETH plus any interest accrued.

However, in the current implementation, this isn't possible due to a check in `WildcatMarketConfig#setMaxTotalSupply(uint256 _maxTotalSupply)`:

```sol
if (_maxTotalSupply < state.totalSupply()) {
  revert NewMaxSupplyTooLow();
}
```

This check can inconvenience borrowers who wish to gradually reduce their market supply by preventing them from going below the current market supply, resulting in increased manual maintenance efforts.