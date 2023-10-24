1.the value of start should be checked to avoid `Arithmetic over/underflow`
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L85#L96
```solidity
  function getRegisteredBorrowers(
    uint256 start,
    uint256 end
  ) external view returns (address[] memory arr) {
    uint256 len = _borrowers.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start;//@audit oob
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }
  }
```

2.allowed should be checked bigger than amount to avoid `Arithmetic over/underflow`
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L41#L57

3.`_transfer` to zero address should be checked 
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L64#L83