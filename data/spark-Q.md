# `revertWithSelectorAndArgument` not return the full error message

Link: https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/Errors.sol#L60
```solidity
/**
 * @dev Reverts with the given error selector and argument.
 * @param errorSelector The left-padded error selector.
 * @param argument The argument to the error.
 */
function revertWithSelectorAndArgument(uint256 errorSelector, uint256 argument) pure {
  assembly {
    mstore(0, errorSelector)
    mstore(0x20, argument)
    revert(Error_SelectorPointer, 0x24) //@audit
  }
}
```
Since both `errorSelector` and `argument` are unit256, the size for revert should be `0x40` instead of `0x20`


