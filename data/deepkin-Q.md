## Global
### Code duplication for chunk getters 
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L85
```
  function getRegisteredBorrowers(
    uint256 start,
    uint256 end
  ) external view returns (address[] memory arr) {
    uint256 len = _borrowers.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }
  }
```
This function is literally reused in multiple places like 
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L128
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L171
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L214
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L125
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L204
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L138

##### Recommendation 
Incapsulate logic into lib function with AddressSet or list parameter. Code will be more readable and you will minimise refactoring error.

## ReentrancyGuard.sol
### Unavailable link in the ReentrancyGuard doc
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/ReentrancyGuard.sol#L7
##### Recommendation 
Keep dev doc clean and valid. Create your own blob if needed.

### Not consistent approach in ReentrancyGuard _setReentrancyGuard() and _clearReentrancyGuard()
```
  function _setReentrancyGuard() internal {
    // Ensure that the reentrancy guard is not already set.
    _assertNonReentrant();

    // Set the reentrancy guard.
    unchecked {
      _reentrancyGuard = _ENTERED;
    }
  }
```
_setReentrancyGuard() use unchecked block to set current value but _clearReentrancyGuard()

```
function _clearReentrancyGuard() internal {
    // Clear the reentrancy guard.
    _reentrancyGuard = _NOT_ENTERED;
  }
```
While both _NOT_ENTERED and _ENTERED is integers.
##### Recommendation 
Use consistent approach, unchecked here doesn't looks useful

## WildcatMarketControllerFactory.sol
### Magic numbers
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L81
```
    if (
      constraints.minimumAnnualInterestBips > constraints.maximumAnnualInterestBips ||
      constraints.maximumAnnualInterestBips > 10000 ||
      constraints.minimumDelinquencyFeeBips > constraints.maximumDelinquencyFeeBips ||
      constraints.maximumDelinquencyFeeBips > 10000 ||
      constraints.minimumReserveRatioBips > constraints.maximumReserveRatioBips ||
      constraints.maximumReserveRatioBips > 10000 ||
      constraints.minimumDelinquencyGracePeriod > constraints.maximumDelinquencyGracePeriod ||
      constraints.minimumWithdrawalBatchDuration > constraints.maximumWithdrawalBatchDuration
    ) {
      revert InvalidConstraints();
    }
```
##### Recommendation 
Use constants like MathUtils.BIP.

### Order of functions
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L106C1-L106C1
```
  function _storeControllerInitCode()
    internal
    virtual
    returns (address initCodeStorage, uint256 initCodeHash)
  {
    bytes memory controllerInitCode = type(WildcatMarketController).creationCode;
    initCodeHash = uint256(keccak256(controllerInitCode));
    initCodeStorage = LibStoredInitCode.deployInitCode(controllerInitCode);
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L116
```
  function _storeMarketInitCode()
    internal
    virtual
    returns (address initCodeStorage, uint256 initCodeHash)
  {
    bytes memory marketInitCode = type(WildcatMarket).creationCode;
    initCodeHash = uint256(keccak256(marketInitCode));
    initCodeStorage = LibStoredInitCode.deployInitCode(marketInitCode);
  }
```

Both used in constructor exactly once.

##### Recommendation 
Put in the end to optimise contract gas usage.

## WildcatMarketBase.sol
### Code duplication for _applyWithdrawalBatchPayment() and _applyWithdrawalBatchPaymentView()
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L498-L551
Both functions pretty big and similar, duplication of such parts can bring inconsistency.
##### Recommendation 
Use DRY principle and share main logic part.
