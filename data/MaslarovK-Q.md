# QA report

## **Table of Contents**

| |Issue|
|-|:-|
| QA&#x2011;01 | Function lacks check if the starting index is lower than the end index|

## QA-01 Function lacks check if the starting index is lower than the end index

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L125-L136
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L138-L149

### Impact

Low, a flaw in contract's actions

If the start index is bigger than the end index, will result in underflow and the function will revert.

```solidity
function getAuthorizedLenders(
    uint256 start,
    uint256 end
  ) external view returns (address[] memory arr) {
    uint256 len = _authorizedLenders.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
      arr[i] = _authorizedLenders.at(start + i);
    }
  }
```

### Vulnerability Details
The function is going to revert every time this happens.

### Recommendation

Add `require(start < end, "Start can't be bigger than end.")`.