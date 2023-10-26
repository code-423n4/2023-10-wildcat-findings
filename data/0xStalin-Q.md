## [L-01] Internal accounting won't work for rebase & elastic tokens

The internal accounting will be messed up if the market's asset is a rebase or an elastic token.
- Internal accounting is set up to work for tokens that don't charge a fee on transfer, as well as for tokens whose supply isn't arbitrarily increased and decreased.

There are a couple of instances where the WildcatMarkets relies on the balance held by the markets and the escrow accounts, if these balances are arbitrarily manipulated by the underlying asset's contract, the wildcat market tokens could behave in unexpected ways.

> WildcatMarketBase contract
```solidity
function totalAssets() public view returns (uint256) {
  return IERC20(asset).balanceOf(address(this));
}
```

> WildcatSanctionsEscrow contract
```solidity
function balance() public view override returns (uint256) {
  return IERC20(asset).balanceOf(address(this));
}
```

**Fix:**
- I think the most viable option is to document the fact that these types of tokens are not supported by the Wildcat Markets and let know the users the downsides and risks of using these tokens as the market's assets.
- Another option could be to create a whitelist of valid tokens, but this might not be necessary if the users are let known in advance that the markets don't work for these erc20 variants.