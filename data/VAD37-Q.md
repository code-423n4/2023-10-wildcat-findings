# Low issues

1. [Low issues](#low-issues)
   1. [L1 - Wildcat Admin `ArchController.removeMarket()` have unintended side-effect that prevent lender from being nuked or sanctioned, rendering `Sentinel` meaningless](#l1---wildcat-admin-archcontrollerremovemarket-have-unintended-side-effect-that-prevent-lender-from-being-nuked-or-sanctioned-rendering-sentinel-meaningless)
   2. [L2 - `ArchController` admin lack direct ability to restore market after removed](#l2---archcontroller-admin-lack-direct-ability-to-restore-market-after-removed)
   3. [L3 - `updateLenderAuthorization()` was not called immediately after `authorizeLenders()` in `deauthorizeLenders()`. Delaying mistake might lead to shenannigan between `borrower` and `lender`](#l3---updatelenderauthorization-was-not-called-immediately-after-authorizelenders-in-deauthorizelenders-delaying-mistake-might-lead-to-shenannigan-between-borrower-and-lender)
   4. [L4 - Multiple duplication of parameter contraints check in multiple place cause confusion. Lead to issue like protocol can prevent creating new market when protocol fee set above 10%](#l4---multiple-duplication-of-parameter-contraints-check-in-multiple-place-cause-confusion-lead-to-issue-like-protocol-can-prevent-creating-new-market-when-protocol-fee-set-above-10)

## L1 - Wildcat Admin `ArchController.removeMarket()` have unintended side-effect that prevent lender from being nuked or sanctioned, rendering `Sentinel` meaningless

When `borrower` want nuke lender or lender is `blacklisted` by ChainAnalysis, anyone can access `Sentinel` contract and immdiately freeze lenders funds. [Link](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketConfig.sol#L50-L81)
This process can be hindered by controller checking if called is registed market. [Link](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsSentinel.sol#L100-L102)
Market is automatic registered on deploy through controller by whitelist borrower. But market can also be removed by Wildcat admin. [Link](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L199-L204)
This seem like this is just for maintainance/legacy purpose, but it can also be used to prevent lender funds from being freezed, if and only if admin mistake or compromised.

## L2 - `ArchController` admin lack direct ability to restore market after removed

`WildcatArchController.sol` register market permission is reserved only for controller (`WildcatMarketController.sol`). [Link](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L192)
And `market` can only be registered once, because controller check if address already be deployed. [Link](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L351-L353).

Market can be removed by admin for reason L1 above. Removing market only affect function to freeze lender funds and nothing else.

For market to be registered again, admin would need to go through several hoops by register `owner` as `controllerFactory` then registered `controller` and then finally register any address as `market`.
This can be seen as abusing admin privilage. Having unintended consequence.

## L3 - `updateLenderAuthorization()` was not called immediately after `authorizeLenders()` in `deauthorizeLenders()`. Delaying mistake might lead to shenannigan between `borrower` and `lender`

It is understandable within protocol whitepaper context that `borrower` and `lender` are legal entity and should be worthy of trust and competence.

But that does not excuse them from making simple mistake as changing `lender` status to *withdrawal only* and then ignoring `@dev` [comment](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L165-L167).
Forgeting to update that permission or simply 2nd transaction not executed for whatever reason like gas cost or down L2 network.

Because market list is Enumerable so the fixed is quite as easy as looping through all market and update lender status directly.

```git
File: WildcatMarketController.sol
169:   function deauthorizeLenders(address[] memory lenders, bool forceUpdate) external onlyBorrower {
170:     for (uint256 i = 0; i < lenders.length; i++) {
171:       address lender = lenders[i];
172:       if (_authorizedLenders.remove(lender)) {
173:         emit LenderDeauthorized(lender);
174:       }//@audit why not update lender authorization during add/remove?
+           if(forceUpdate) //@ force update to prevent out-of-gas too many market issue.
+              updateLenderAuthorization(lender, _controlledMarkets.values());
177:     }
178:   }
```

The impact is `lender` might have some free delay time to deposit some more funds into `borrower` at their own demise. Not a huge lost for `borrower`.

## L4 - Multiple duplication of parameter contraints check in multiple place cause confusion. Lead to issue like protocol can prevent creating new market when protocol fee set above 10%

`WildcatMarketControllerFactory` have its own paramater constrains set during constructor.
`WildcatMarketController` deployed from factory just duplicate these parameters when create new market `WildcatMarket`.
`WildcatMarket` have its own hard-cap check too through `MathUtils` library constant `BIP` which is 1e4 or 10%. In multiple places. [Link1](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L82-L93). [Link2](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketConfig.sol#L152-L154).

While test file contain [constant file](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/test/shared/TestConstants.sol#L8-L30) reveal that maximum parameters for factory also set to 10% same as `BIP`.

If factory set its own maximum parameter limit during constructor above 10%. This include yield rate can be above 10% a year.
Factory contract can still be deployed will be deployed along with Controller contract.
But when it come to `WildcatMarket.sol` version `1.0`, it can not be created. It will simply revert during constructor when deployed through `WildcatMarketController`.

Other unintended affect is there is limit check on protocol fee. [Link](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L198-L219). When protocol set fee above 10%. Market deployment also reverted due to [hardcode limit check.](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L88-L90)
