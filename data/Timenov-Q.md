## Summary
[I-1] Variable not in right place.
[I-2] Wrong naming of constant.
[I-3] View functions lack `nonReentrantView` modifier.
[I-4] Function parameters are in different order.

### [I-1] Variable not in right place.
Variables are defined at the beggining of the contract.

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L244

### [I-2] Wrong naming of constant.
Constant variables should be uppercase.

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L24

### [I-3] View functions lack `nonReentrantView` modifier.
Some functions do not have the `nonReentrantView` modifier.

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L238-L240

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L267-L269

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L307-L309

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L318-L326

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L334-L341

### [I-4] Function parameters are in different order.
If we take a look at the interface `IWildcatSanctionsSentinel` and in particular `function createEscrow(address account, address borrower, address asset)` and now look at the `WildcatSanctionsSentinel` which inherits the interface: `function createEscrow(address borrower, address account, address asset)`, we can see that the `account` and `borrower` differ in positions. Consider using the same order of parameters to avoid confusion.

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/interfaces/IWildcatSanctionsSentinel.sol#L71-L75

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsSentinel.sol#L95-L99