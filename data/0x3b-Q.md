| *Issue* | *Description*                                                                  |
|---------|--------------------------------------------------------------------------------|
| [L-01]  | Borrower can just stay in delinquency                                          |
| [L-02]  | Borrower gets counted for delinquency for his previous time updated            |
| [L-03]  | When `valueIfFalse` > `valueIfTrue` and `condition == true` an overflow occurs |
| [N-01]  | No account for leap years                                                      |
| [N-02]  | We can set the APR above 100%                                                  |

### [L-01] Borrower can just stay in delinquency
Borrower can be late with the payments and if the delinquency period is big (1 month) the lenders would be delayed with their withdraws at no cost for the borrower. He would be able to go 1 month in delinquency (making lenders wait for their withdraws) and then be 1 month clear (to clock back on the delinquency to 0). This can be executed at no extract cost and most likely will happen with some markets.

### [L-02] Borrower gets counted for delinquency for his previous time updated
Borrower gets counted for previous delinquent time and not up to now. This means that from the last update up to now is reserved and counted at the next update.

Example:
| *Prerequisites* | *Values*   |
|-----------------|------------|
| Borrower state  | delinquent |
| Grace period    | 7 days     |
| Update every    | 1 day      |
| Current day     | 7          |

Here if we [update](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L53-L71) the clock at day 8 he would **not pay** any delinquency fee since the update at the end of day 8 will count as update from the period of day 6 --> 7. And subsequently if he [updates](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L53-L71) at  day 9 he would pay the delinquency fee from day 7 --> 8, and again nothing for 8 --> 9.

This is because the calculation for getting [secondsRemainingWithoutPenalty](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L104-L106) uses `previousTimeDelinquent` instead of the current.

```solidity
    if (state.isDelinquent) {
      state.timeDelinquent = (previousTimeDelinquent + timeDelta).toUint32();
      uint256 secondsRemainingWithoutPenalty = delinquencyGracePeriod.satSub(
        previousTimeDelinquent
      );
      return timeDelta.satSub(secondsRemainingWithoutPenalty);
    }
```
### [L-03] When `valueIfFalse` > `valueIfTrue` and `condition == true` an overflow occurs
If `valueIfFalse` > `valueIfTrue` and `condition == true` will return wrong amounts as it will underflow.

```solidity
  function ternary(bool condition,uint256 valueIfTrue,uint256 valueIfFalse) internal pure returns (uint256 c) {
    assembly {
      c := add(valueIfFalse, mul(condition, sub(valueIfTrue, valueIfFalse)))
    }
  }
```
This is because we are subtracting false from true `sub(valueIfTrue, valueIfFalse)` and there is a lack of checks to make sure this subtraction will not underflow. This is not used in the current code in any wrong way so I point it out as low for future reference. 

### [N-01] No account for leap years
One year [for the protocol](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L14) is 365 days, while in reality is 365 days and 4 hours as every 4 years there is 1 day added.

### [N-02] We can set the APR above 100% 
The constructor of [**WildcatMarketBase**](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L82) does not allow us to set the `annualInterestBips` above 100%, however if we set `selinquencyFeeBips` high and APR of above 100% is achievable. Even for the borrower high APR and high `protocolFeeBips` can set his constant APR of above 100%.