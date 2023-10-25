# [LOW] Lender's deposit may be stuck in the mempool

When a user makes a deposit to a market, his request may be stuck for a long time in the mempool leading to unfavourable conditions for him. Consider adding a slippage parameter to the deposit functions.

# [NC] Escrow address is not stored in WildcatMarket

When the market [creates](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketWithdrawals.sol#L166-L170) an escrow for a blocked user, it does not save the address of the new escrow to the storage. This makes escrow addresses harder for finding. Consider storing the address in storage.

# [NC] Market token name missing space

*WildcatMarketBase* sets name to the concatenated value of prefix + name. Because of this the prefix and the actual name are not separated. According to the [*gitbook*](https://wildcat-protocol.gitbook.io/wildcat/using-wildcat/day-to-day-usage/borrowers) in section **Market Token Name Prefix**, they should be separated with a space. Consider adding one.

```jsx
    name = string.concat(parameters.namePrefix, queryName(parameters.asset));
```
