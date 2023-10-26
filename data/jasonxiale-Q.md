## [L-01] `WildcatMarketConfig.setMaxTotalSupply` is never be called
File:
   https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketConfig.sol#L134-L144

`WildcatMarketConfig.setMaxTotalSupply` has a modifier which only can be called by controller, but after searching through controller's code, the function is never be called.

## [L-02] `WildcatMarketControllerFactory.setProtocolFeeConfiguration` missing `UpdateProtocolFeeConfiguration` event
File:
    https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L195C12-L217
Accoring to [UpdateProtocolFeeConfiguration](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L17C9-L22), an event `UpdateProtocolFeeConfiguration` is defined for function `setProtocolFeeConfiguration`, but the event is never used

## [L-03] `WildcatMarketControllerFactory.deployController` missing `NewController` event
File:
    https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L282-L301
According to [comments](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L279-L280) and [NewController](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L16), an event `NewController` should be emitted in function `deployController`, but the event is never emitted

## [L-04] In `WildcatMarketController.updateLenderAuthorization`, _authorizedLenders.contains(lender) can cached
File:
    https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L188
In [WildcatMarketController.updateLenderAuthorization](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L182C12-L190), `_authorizedLenders.contains(lender)` can be cached to save some gas within for-loop
```diff
diff --git a/src/WildcatMarketController.sol b/src/WildcatMarketController.sol
index 55f62f6..7ce3c6f 100644
--- a/src/WildcatMarketController.sol
+++ b/src/WildcatMarketController.sol
@@ -180,12 +180,13 @@ contract WildcatMarketController is IWildcatMarketControllerEventsAndErrors {
    *      status.
    */
   function updateLenderAuthorization(address lender, address[] memory markets) external {
+    bool _isAuthorized = _authorizedLenders.contains(lender);
     for (uint256 i; i < markets.length; i++) {
       address market = markets[i];
       if (!_controlledMarkets.contains(market)) {
         revert NotControlledMarket();
       }
-      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
+      WildcatMarket(market).updateAccountAuthorization(lender, _isAuthorized);
     }
   }

```

## [L-05] `MathUtils.calculateLinearInterestFromBips` can be removed
File: 
    https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MathUtils.sol#L30-L39

  [MathUtils.calculateLinearInterestFromBips](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MathUtils.sol#L30-L39) is only called by [FeeMath.sol#L34-L37](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L34-L37), but the same function `calculateLinearInterestFromBips` is defined in [FeeMath.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L19C12-L28), which means we can remove [MathUtils.calculateLinearInterestFromBips](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MathUtils.sol#L30-L39) 


## [L-06] `LibStoredInitCode.create2WithStoredInitCode` doesn't check `create2` return value
File:
    https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/LibStoredInitCode.sol#L115
According to [yul document](https://docs.soliditylang.org/en/latest/yul.html), `create2` might return **0**, so the function should check `create2` return value
