# [N-01] All contracts that `WildcatMarket` inherits can be made abstract

In OOP it is considered a best practice to daclare classes/contract that are not instantiated anywere and only are being inherited, abstract. Bacuse of this, the following contracts can be made abstract:

`WildcatMarketBase` - https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L14

`WildcatMarketConfig` - https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketConfig.sol#L9

`WildcatMarketToken` - https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L6

`WildcatMarketWithdrawals` - https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L11

# [N-02] Unused imports

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsEscrow.sol#L5