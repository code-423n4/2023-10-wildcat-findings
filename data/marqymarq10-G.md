## Issue #1
`ReentrancyGuard.sol` `_NOT_ENTERED`, `_ENTERED` & `_reentranceyGuard` all use **uint256**. This requires touching 3 cold storage slots (2,100 gas * 3). You can use uint80 (or lower) to pack the storage variables into one slot. Using a warm storage slot costs only 100 gas. This will save a significant amount of gas for modifier `nonReentrant()` & modifier  `nonReentrantView()`.

[`RentranceyGuard::L:20-23`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/ReentrancyGuard.sol#L20-L23)

## Issue #2
There are numerous unoptimized `for-loops` through out the protocol. It saves on gas to use a `while-loop` instead. Here is an example implementation:
```javascript
uint256 i;
uint256 count = lenders.length;
while (i < count) {
   // do stuff
   unchecked {
      ++i;
   }
}
```
When using a `for-loop`, you are generating each stack variable each time you loop through. Additionally, using an unchecked block consumes less gas as it avoids overflow checks, which in this case it is worth while.

**Occurrences:**
- [`WildcatArchController::L93-95`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L93-L95)
- [`WildcatArchController::L136-138`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L136-L138)
- [`WildcatArchController::L179-181`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L179-L181)
- [`WildcatArchController::L222-225`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L222-L225)
- [`WildcatMarketController::L133-135`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L133-L135)
- [`WildcatMarketController::L154-159`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L154-L159)
- [`WildcatMarketController::L170-175`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L170-L175)
- [`WildcatMarketController::L183-189`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L183-L189)
- [`WildcatMarketController::L212-214`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L212-L214)
- [`WildcatMarketControllerFactory::L146-148`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L146-L148)


## Issue #3
It is cheaper to store a `uint32` parameter as `uint256`. In `WildcatMarketWithdraws.sol`, `expiry` is `uint32` parameter that is used in multiple functions. Since it is not a packed storage variable, it is stored in 32 bytes on the stack. Casting it as a `uint32` requires the EVM to use unnecessary shift opcodes when performing operations with it.