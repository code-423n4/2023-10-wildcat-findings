## Low Risk Issues

### [L-01] Doesn't support rebase tokens
The implementation doesn't handle rebase tokens which can change its value from the in-contract stored amount.

### [L-02] Blockwise compoundable fees can deviate from the initial interest rate by lot
The scaleFactor increase is compounded. If the lenders call invoke the `updateState` funcion frequently this can result in very high scaleFactor value when comapred to the fixed interest.

```solidity
File: src/libraries/FeeMath.sol

    uint256 prevScaleFactor = state.scaleFactor;
    uint256 scaleFactorDelta = prevScaleFactor.rayMul(baseInterestRay + delinquencyFeeRay);


    state.scaleFactor = (prevScaleFactor + scaleFactorDelta).toUint112();

```

*GitHub*: [168](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L168-L171)


## Non-critical Issues

### [NC-01] Manually revert on conditions to provide clearer error information to users
It is possible for the vm to revert in the following casees. Consider checking the condition and providing a custom error so that users can handle the error better.
*There are 2 instances of this issue*

```solidity
File: src/market/WildcatMarketWithdrawals.sol

89:            account.scaledBalance -= scaledAmount;

```

*GitHub*: [89](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L89-#L89)
```solidity
File: src/libraries/MarketState.sol

109:            uint256 totalAvailableAssets = totalAssets - state.normalizedUnclaimedWithdrawals;

```

*GitHub*: [109](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L109-#L109)