## [L-01] ArchController allows adding a MarketControllerFactory that points to another ArchController

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L106

`MarketControllerFactory` uses `ArchController` for authorization purposes. ArchController allows for authorizing `MarketControllerFactory` instances that point to another ArchController, and if this happens, the impact would be to allow otherwise unauthorized actors to perform operations i.e. change the protocol fee configuration or deploy controllers.

Consider adding a check in `ArchController.registerControllerFactory` to make sure that the to-be-added `MarketControllerFactory` is correctly paired with it.

## [L-02] WildcatMarketController's authorizeLenders and deauthorizeLenders miss checks if the operation was successful

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L153
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L169

These functions don't return or log any error if the operation fails (addition of a lender already in the set, removal of a lender not present). This may for example trick borrowers who mistype a lender address into thinking they successfully removed a lender when in fact they didn't. 

Consider returning errors consistently with how checks are done in the ArchController.

## [L-03] Floating pragma

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarket.sol#L2

All contracts in scope are floating the pragma version.

Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or a package.

Consider locking the pragma version to a specific Solidity version i.e. `0.8.21`

## [L-04] WildcatSanctionsSentinel.removeSanctionOverride should not be allowed for escrows

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsSentinel.sol#L56

When `WildcatSanctionsSentinel` creates an escrow, it sets up a sanction override for its address. The borrower is incorrectly allowed to remove this override through the `removeSanctionOverride` function, which is meant to be used with lender addresses instead.

Consider storing the addresses of escrows in the Sentinel, and do not allow borrowers to remove the sanction override for them

## [L-05] WildcatMarketController.authorizeLenders allows the borrower to authorize themselves as lender

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L153

In WildcatMarket instances, the borrower has complete control over the addresses authorized to perform lender operations. While this does not represent an issue per se, it would be reasonable to prevent borrowers from adding themselves as lenders to avoid unforeseen side-effects.

Consider preventing the borrower from adding themselves as lender in `WildcatMarketController` or `WildcatMarket`.

## [N-01] Underflow protection produces obscure error messages

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketWithdrawals.sol#L89
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L50
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L73

In several places, the protocol relies on Solidity 0.8.x underflow protection for preventing users from performing actions they shouldn't.

Some examples of these are: 
- the check on available balance on `WildcatMarketWithdrawals.queueWithdrawal`
- the allowance check on `WildcatMarketToken.transferFrom`
- the balance check on `WildcatMarketToken._transfer`

Consider adding proper `require` statements to instead deliver more user-friendly error messages.