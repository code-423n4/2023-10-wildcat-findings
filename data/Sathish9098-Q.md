# LOW FINDINGS

##

## [L-] A year is not always 365 days

On leap years, the number of days is 366, so calculations during those years will return the wrong value

```solidity
FILE: Breadcrumbs2023-10-wildcat/src/libraries/MathUtils.sol

14: uint256 constant SECONDS_IN_365_DAYS = 365 days;

```
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MathUtils.sol#L14


