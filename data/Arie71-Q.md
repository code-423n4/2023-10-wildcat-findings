Reentrancy in WildcatMarketBase._blockAccount(MarketState,address) (src/market/WildcatMarketBase.sol#163-187):
External calls:
- escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(accountAddress,borrower,address(this)) (src/market/WildcatMarketBase.sol#172-176)
Event emitted after the call(s):
- SanctionedAccountAssetsSentToEscrow(accountAddress,escrow,state.normalizeAmount(scaledBalance)) (src/market/WildcatMarketBase.sol#179-183)
- Transfer(accountAddress,escrow,state.normalizeAmount(scaledBalance)) (src/market/WildcatMarketBase.sol#177)

Description:

Detects [reentrancies](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy) that allow manipulation of the order or value of events.

Recommendation:

Apply the [`check-effects-interactions` pattern](https://docs.soliditylang.org/en/latest/security-considerations.html#re-entrancy)