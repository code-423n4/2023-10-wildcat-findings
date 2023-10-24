Three issues with the following function in src/WildcatArchController.sol

1) Function can run out of gas if for loop is too long. If the '_borrowers' set has a max value or is managed off chain this can be considered not an issue.

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
  }    for (uint256 i = 0; i < count; i++) {
```

2) src/WildcatArchController.sol
line 93: https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L93C1-L93C42
```
 for (uint256 i = 0; i < count; i++) {
```
change 'i++' to '++i' to save some gas. Also no need to instantiate i to 0 as it will already have a value of 0 by default.
