### Gas saving in for-loops

The for-loops are not gas optimized. Refer to this [guide](https://mirror.xyz/haruxe.eth/DW5verFv8KsYOBC0SxqWORYry17kPdeS94JqOVkgxAA) by haruxe 
```
    for (uint256 i = 0; i < count; i++) 
    {
        ...
    }
```

These can be safely rewritten in the following manner for same functionality with gas savings.
```solidity
    for (uint256 i = 0; i != count; ) 
    {
        ...
        unchecked {
            ++i
       }
   }
```

Reasons for gas saving
- ++i is cheaper than i++
- unchecked is cheaper in gas
- i != count is cheaper than i < count

Affected functions are listed below:
- src/WildcatArchController.sol:WildcatArchController.[getRegisteredBorrowers](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L93-L95)
- src/WildcatArchController.sol:WildcatArchController.[getRegisteredControllerFactories](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L136-L138)
- src/WildcatArchController.sol:WildcatArchController.[getRegisteredControllers](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L179-L181)
- src/WildcatArchController.sol:WildcatArchController.[getRegisteredMarkets](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L222-L224C6)
- src/WildcatMarketController.sol:WildcatMarketController.[getAuthorizedLenders](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L133-L135)
- src/WildcatMarketController.sol:WildcatMarketController.[authorizeLenders](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L154C9-L159)
- src/WildcatMarketController.sol:WildcatMarketController.[deauthorizeLenders](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L170-L174C8)
- src/WildcatMarketController.sol:WildcatMarketController.[updateLenderAuthorization](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L183C21-L189)
- src/WildcatMarketController.sol:WildcatMarketController.[getControlledMarkets](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L212-L214)
- src/WildcatMarketControllerFactory.sol:WildcatMarketController.[getDeployedControllers](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L146-L148)
- src/libraries/FIFOQueue.sol:FIFOQueue.[values](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L48-L50)
- src/libraries/FIFOQueue.sol:FIFOQueue.[shiftN](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L75-L77)



An example for optimizing for-loops in WildcatArchController.getRegisteredBorrowers
Before:
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
Gas used:
```
| src/WildcatArchController.sol:WildcatArchController contract |                 |       |        |       |         
| getRegisteredBorrowers(uint256,uint256)(address[])           | 1863            | 2099  | 2099   | 2336  | 2       
```
After:
```
    function getRegisteredBorrowers(
        uint256 start,
        uint256 end
    ) external view returns (address[] memory arr) {
        uint256 len = _borrowers.length();
        end = MathUtils.min(end, len);
        uint256 count = end - start;
        arr = new address[](count);
        for (uint256 i = 0; i != count; ) {
            arr[i] = _borrowers.at(start + i);
            unchecked {
                ++i;
            }
        }
    }
```
Gas used:
```
| src/WildcatArchController.sol:WildcatArchController contract |                 |       |        |       |         
| getRegisteredBorrowers(uint256,uint256)(address[])           | 1848            | 2075  | 2075   | 2303  | 2       
```