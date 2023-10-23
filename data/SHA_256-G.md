# [G-01] Use `calldata` instead of `memory` for function parameters
lines: https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L85-L94,
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L128-L138,
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L171-L181,
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L214-L224

In this instances the dynamic array `arr` has the storage location `memory`. When the function gets called externally, the array values are kept in `calldata` and copied to `memory` during ABI decoding (using the opcode calldataload and mstore ). And during the for loop, `arr[i]` accesses the value in memory using a `mload`.Each iteration would cost at least 60 gas. Using `calldata` this can be completely avoided & it saves gas. This will also reduce the number of instructions and therefore reduce the deployment time cost of the contract.

# [G-02] OR in `if`-condition can be rewritten to single `if` conditions
lines: https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L79-L88,
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L204-L208
Refactoring the `if`-condition in a way it won’t be containing the `||` operator will save more gas.

# [G-03] Deleting an entry might incur significant gas costs.
Depending on the size of `temporaryExcessReserveRatio` mapping, deleting an entry might incur significant gas costs. Be aware of this, especially if the mapping contains many entries.

line: https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L500

# [G-04] `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for` - and `while`-loops
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.
lines : https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L93,
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L136,
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L179,
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L222

# [G-05] + LOW : `For` loops in public or external functions should be avoided due to high gas costs and possible DOS
In Solidity, for loops can potentially cause `Denial of Service (DoS)` attacks if not handled carefully. DoS attacks can occur when an attacker intentionally exploits the gas cost of a function, causing it to run out of gas or making it too expensive for other users to call. Below are some scenarios where for loops can lead to DoS attacks: Nested for loops can become exceptionally gas expensive and should be used sparingly

lines: https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L93,
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L136,
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L179,
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L222

# [G-06] Use constants instead of `type(uint256).max`
At the place where `type(uint256).max` is used it can be replaced with constant variable with the same value as this from the expression, it will save gas because constant variables are stored in the contract byte code.
lines: https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L49

# [G-07] Use hardcode address instead `address(this)`
Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s `script.sol` and solmate’s `LibRlp.sol` contracts can help achieve this.
Reference: https://book.getfoundry.sh/reference/forge-std/compute-create-address
lines: https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L172-L175,


