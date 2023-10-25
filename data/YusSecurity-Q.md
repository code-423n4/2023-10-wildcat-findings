## [QA-1] Risk of permanent fund loss in current repayment method

The existing repayment mechanism requires borrowers to transfer funds directly to the market's ERC20 account. This method is susceptible to errors and can result in the permanent loss of funds because it lacks safeguards against overpayments.

This design choice was made to enable third-party addresses to repay the loan during force majeure events.

A more effective approach would involve creating a function allowing anyone to repay the debt, capped at the `MarketState#totalDebts()`, with excess funds automatically returned to the sender.

## [QA-2] Constraints on market parameters applied only at creation

The protocol's constraints on market parameters are solely enforced during market creation, specifically within the `deployMarket()` function.

However, subsequent changes to market parameters made directly through functions like `WildcatMarketConfig#setMaxTotalSupply()`, `WildcatMarketConfig#setAnnualInterestBips()`, and `WildcatMarketConfig#setReserveRatioBips()` do not undergo any constraint checks.

## [QA-3] Borrowers cannot reduce market capacity below the current total supply

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

## [QA-4] `EscrowReleased` event may be emitted, even when an attempt to release escrow fails.

In the implementation of `WildcatSanctionsEscrow#releaseEscrow()`, the `IERC20(asset).transfer(account, amount)` function is used to release escrowed funds. The code does not check the return value, which means that the `EscrowReleased` event may be emitted even if the transfer of escrowed funds fails (e.g., when `transfer()` returns `false`).