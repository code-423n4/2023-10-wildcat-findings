| *Issue* | *Description*                                 |
|---------|-----------------------------------------------|
| [G-01]  | Reduce the function count                     |
| [G-02]  | Use `scaledOwedAmount` instead of remaking it |
| [G-03]  | Revert earlier                                |


### [G-01] Reduce the function count
 [**WildcatArchController**](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol) is mostly setters and getters, where the getters are used to cycle an array of Enumerable set and return some of it's variables. However there are 4 different functions that do the same, where the only different is their Enumerable sets --> [1](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L128-L139) and [2](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L171-L182).
Instead of using different function for every set, you can make one and add to the inputs the set you wanna check.

Example:
```solidity
  function getRegisteredControllerFactories(uint256 start,uint256 end) external view returns (address[] memory arr) {
    return getSet(_controllerFactories,uint256 start,uint256 end)
  }
  function getSet(EnumerableSet set, uint256 start,uint256 end) external view returns (address[] memory arr) {
    uint256 len = set.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
      arr[i] = set.at(start + i);
    }
  }
```
Same could be done with *[*WildcatMarketController**](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol)

### [G-02] Use `scaledOwedAmount` instead of remaking it
Instead of remaking functions that are already made, you can just re-use them. My suggestion is instead of doing the math bellow, [scaledOwedAmount](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L38-L40) can be called to do the same, thus saving some deployment gas.


[**WildcatMarketBase**](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L505)
```solidity
    uint104 scaledAmountOwed = batch.scaledTotalAmount - batch.scaledAmountBurned;
```
[**Withdrawal**](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L38-L40)
```solidity
  function scaledOwedAmount(WithdrawalBatch memory batch) internal pure returns (uint104) {
    return batch.scaledTotalAmount - batch.scaledAmountBurned;
  }
```

### [G-03] Revert earlier
Under **WildcatMarket**'s `depositUpTo` [this revert statement](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L63) ( [_getAccountWithRole](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L197-L213)  revert with the call to [_getAccount](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L150-L155) if the account doesn't have the lender permission ) could be placed upper in the function in order to revert earlier and not waste gas.
