The function ***"setAnnualInterestBips"*** in ***"WildcatMarketController.sol"*** is casting the ***"annualInterestBips"*** as ***"uint16"***. Where the interest bips can be maximum 10,000.

```
function setAnnualInterestBips(
    address market,
    uint16 annualInterestBips
  ) external virtual onlyBorrower onlyControlledMarket(market) {
```

But in ***"WildcatMarketConfig"***, ***"annualInterestBips"*** is stated as ***"uint256"***

```
 function annualInterestBips() external view returns (uint256) {
    return _state.annualInterestBips;
  }
```

Therefore, using anything more than ***"uint16"*** for ***"annualInterestBips"*** is losing gas.