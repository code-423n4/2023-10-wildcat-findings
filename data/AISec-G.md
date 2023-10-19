
# 1) Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol?plain=1#L204-L208
```
    if (
      (protocolFeeBips > 0 && nullFeeRecipient) ||
      (hasOriginationFee && nullFeeRecipient) ||
      (hasOriginationFee && nullOriginationFeeAsset)
    )
```


# Recommended Mitigation Steps

```
if (protocolFeeBips > 0) {
  if (nullFeeRecipient) {
      ...
  }
}
```



# 2) Cache Array Length Outside of Loop

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol?plain=1#L154
```
    for (uint256 i = 0; i < lenders.length; i++) {
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol?plain=1#L170
```
    for (uint256 i = 0; i < lenders.length; i++) {
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol?plain=1#L183
```
    for (uint256 i; i < markets.length; i++) {
```

# Recommended Mitigation Steps
```
uint256 len = markets.length;
for (uint256 i ; i < len; i++) {
...
}
```
