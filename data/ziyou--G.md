|no |issue|number of issue|
|------|-----|---------------|
|[G-01]|Loop best practice to save gas|3|


## [G1] Loop best practice to save gas
```solidity

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L154
154    for (uint256 i = 0; i < lenders.length; i++) {
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L170
170    for (uint256 i = 0; i < lenders.length; i++) {
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L183
183    for (uint256 i; i < markets.length; i++) {
```
```solidity
best practice
-----------------------------------------------------
for (uint i = 0; i < lenders.length; i = unchecked_inc(i)) {
// do something that doesn't change the value of i
}
function unchecked_inc(uint i) returns (uint) {
unchecked {
return i + 1;
}
}
```