# [G-01] Optimize loops in `WildcatMarketController.sol`

[File: WildcatMarketController.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L133-L135)
```
  for (uint256 i = 0; i < count; i++) {
      arr[i] = _authorizedLenders.at(start + i);
    }
```

In function `getAuthorizedLenders()`, it's a waste of gas to calculate `start + i` on every loop iteration. Much better solution would be to start iterating from `i = start`. This will save gas, because `start + i` addition will be redundant.


The same issue occurs in function `getControlledMarkets()`

[File: WildcatMarketController.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L1212-L214)
```
    for (uint256 i = 0; i < count; i++) {
      arr[i] = _controlledMarkets.at(start + i);
    }
```

In both cases, the loop can be rewritten to form:

```
for (uint256 i = start; i < end; i++) {
```

# [G-02] Optimize loop in `WildcatMarketControllerFactory.sol`

[File: WildcatMarketControllerFactory.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L146-L148)
```
for (uint256 i = 0; i < count; i++) {
      arr[i] = _deployedControllers.at(start + i);
    }
```

In function `getDeployedControllers()`, it's a waste of gas to calculate `start + i` on every loop iteration. Much better solution would be to start iterating from `i = start`. This will save gas, because `start + i` addition will be redundant.

The loop can be rewritten to form:

```
for (uint256 i = start; i < end; i++) {
```

# [G-03] Unnecessary variable declaration in `WildcatMarketControllerFactory.sol`

[File: WildcatMarketControllerFactory.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L344-L346)
```
 bytes32 salt = bytes32(uint256(uint160(borrower)));
    return
      LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt, controllerInitCodeHash);
```

Variable `salt` is used only once, which means there's no need to declare it at all. Above code can be changed to:

```
    return
      LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, bytes32(uint256(uint160(borrower))), controllerInitCodeHash);
```


# [G-04] Optimize loops in `WildcatArchController.sol`

[File: WildcatArchController.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L93-L95)
```
 for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
```

In function `getRegisteredBorrowers()`, it's a waste of gas to calculate `start + i` on every loop iteration. Much better solution would be to start iterating from `i = start`. This will save gas, because `start + i` addition will be redundant.

The same issue occurs in functions `getRegisteredControllerFactories()`, `getRegisteredControllers()` and `getRegisteredMarkets()`

[File: WildcatArchController.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L136-L138)
```
for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllerFactories.at(start + i);
```

[File: WildcatArchController.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L179-L180)
```
 for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllers.at(start + i);
```

[File: WildcatArchController.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L222-L223)
```
for (uint256 i = 0; i < count; i++) {
      arr[i] = _markets.at(start + i);
```


# [G-05] Unnecessary variable declarations in `WildcatMarketWithdrawals.sol`

In function `getAvailableWithdrawalAmount()`

variables `previousTotalWithdrawn` and `newTotalWithdrawn ` are used only once, which means they don't need to be declared at all. Code can be changed to:

```
    return uint256(batch.normalizedAmountPaid).mulDiv(
      status.scaledAmount,
      batch.scaledTotalAmount
    ) - status.normalizedAmountWithdrawn;
```


# [G-06] `revert` check can be move up in `WildcatMarketWithdrawals.sol`

[File: WildcatMarketWithdrawals.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketWithdrawals.sol#L155-L163)
```
    uint128 normalizedAmountWithdrawn = newTotalWithdrawn - status.normalizedAmountWithdrawn;

    status.normalizedAmountWithdrawn = newTotalWithdrawn;
    state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;

    if (normalizedAmountWithdrawn == 0) {
      revert NullWithdrawalAmount();
    }
```

can be changed to:

```
    uint128 normalizedAmountWithdrawn = newTotalWithdrawn - status.normalizedAmountWithdrawn;

    if (normalizedAmountWithdrawn == 0) {
      revert NullWithdrawalAmount();
    }

    status.normalizedAmountWithdrawn = newTotalWithdrawn;
    state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;

    
```

# [G-07] Unnecessary variable declaration in `MathUtils.sol` and `FeeMath.sol`

[File: MathUtils.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MathUtils.sol#L34-L39)
```
    uint256 rate = rateBip.bipToRay();
    uint256 accumulatedInterestRay = rate * timeDelta;
    unchecked {
      return accumulatedInterestRay / SECONDS_IN_365_DAYS;
    }
```

Variable  `rate` is used only once, which means it doesn't need to be declared at all. Above code can be changed to:

```

    uint256 accumulatedInterestRay = rateBip.bipToRay() * timeDelta;
    unchecked {
      return accumulatedInterestRay / SECONDS_IN_365_DAYS;
    }
```

The same issue occurs in:

[File: FeeMath.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L23-L27)
```
 uint256 rate = rateBip.bipToRay();
    uint256 accumulatedInterestRay = rate * timeDelta;
    unchecked {
      return accumulatedInterestRay / SECONDS_IN_365_DAYS;
    }
```


# [G-08] Optimize loops in `FIFOQueue.sol.sol`

[File: FIFOQueue.sol.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L48-L49)
```
    for (uint256 i = 0; i < len; i++) {
      _values[i] = arr.data[startIndex + i];
```

In function `values()` it's a waste of gas to calculate `startIndex + i` on every loop iteration. Much better solution would be to start iterating from `i = startIndex`. This will save gas, because `startIndex + i` addition will be redundant.

The same issue occurs in function `shiftN()`:

[File: FIFOQueue.sol.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L475-L76)
```
    for (uint256 i = 0; i < n; i++) {
      delete arr.data[startIndex + i];
```

 # [G-09] Substraction in `FIFOQueue.sol.sol` can be unchecked

 [File: FIFOQueue.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L38-L40)
```
 function length(FIFOQueue storage arr) internal view returns (uint128) {
    return arr.nextIndex - arr.startIndex;
  }
```

By definition of FIFO Queue: `arr.nextIndex >= arr.startIndex`, thus above substraction can be unchecked.

The same issue occurs in:

[File: FIFOQueue.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L43-L45)
```
 uint256 startIndex = arr.startIndex;
    uint256 nextIndex = arr.nextIndex;
    uint256 len = nextIndex - startIndex;
```

By definition - in FIFO Queue `startIndex <= nextIndex `, thus `uint256 len = nextIndex - startIndex` can be unchecked.


# [G-10] Unnecessary variable declaration in `FIFOQueue.sol`

[File: FIFOQueue.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L44-L45)
```
    uint256 nextIndex = arr.nextIndex;
    uint256 len = nextIndex - startIndex;
```

Variable `nextIndex` is used only once, which means it doesn't need to be declared at all. Code can be changed to:
```
uint256 len = arr.nextIndex - startIndex;
```

[File: FIFOQueue.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L55-L59)
```
 function push(FIFOQueue storage arr, uint32 value) internal {
    uint128 nextIndex = arr.nextIndex;
    arr.data[nextIndex] = value;
    arr.nextIndex = nextIndex + 1;
  }
```

Function can be rewritten, so `nextIndex` will be redundant:

```
 function push(FIFOQueue storage arr, uint32 value) internal {
   
    arr.data[arr.nextIndex] = value;
    ++arr.nextIndex;
  }
```

# [G-11] In `FIFOQueue.sol` some additions can be changed to pre-increment

[File: FIFOQueue.sol](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L55-L59)
```
 function push(FIFOQueue storage arr, uint32 value) internal {
    uint128 nextIndex = arr.nextIndex;
    arr.data[nextIndex] = value;
    arr.nextIndex = nextIndex + 1;
  }
```

In function `push()`, we firstly assign `arr.nextIndex` to variable `nextIndex` and then we perform below addition: `arr.nextIndex = nextIndex + 1;`
Since `(nextIndex = arr.nextIndex)`, then `(arr.nextIndex = nextIndex + 1) == (arr.nextIndex = arr.nextIndex + 1)  == (++arr.nextIndex)`.
Thus we can change `arr.nextIndex = nextIndex + 1;` to `++arr.nextIndex ;`.

The same issue occurs in function `shift()`. `arr.startIndex = startIndex + 1;` can be changed to `++arr.startIndex;`
