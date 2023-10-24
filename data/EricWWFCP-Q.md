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
  }
```
