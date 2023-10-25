
## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Interest calculation per year is wrong in non leap years | 2 |
| [L-2](#L-2) | The market `max_capacity` could be bypassed | 1 |
| [L-3](#L-3) | Missing contract-existence checks before yul `call()` | 1 |

Total: 3 instances over 2 issues

### <a name="L-1"></a>[L-1] Interest calculation per year is wrong in non leap years
To calculate linear interest from bips, the function `calculateLinearInterestFromBips` divid by the number of seconds in 365 days per year which is not alwayse the case (leap years).

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/libraries/FeeMath.sol

	unchecked {
26:       return accumulatedInterestRay / SECONDS_IN_365_DAYS;
	}


```
[Link to code](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L26)

```solidity
File: src/libraries/FeeMath.sol

	unchecked {
37:       return accumulatedInterestRay / SECONDS_IN_365_DAYS;
	}


```
[Link to code](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L26)

</details>

### <a name="L-2"></a>[L-2] The market `max_capacity` could be bypassed
The market `max_capacity` could be bypassed by performing a direct transfer of tokens.

#### Impact
The attacker could send as much tokens as he want to the market without paying any interest or fees

#### Proof of Concept
To reproduce this action all you have to do is :
1) Create a market with the max_capacity you want
2) Transfer more tokens than the max_capacity directly to the market address

### <a name="L-3"></a>[L-3] Missing contract-existence checks before yul `call()`
Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that `extcodesize()` is non-zero.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/libraries/StringQuery.sol

39:   assembly {
        mstore(0, rightPaddedFunctionSelector)
        let status := staticcall(gas(), target, 0, 0x04, 0, 0)

```
[Link to code](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/StringQuery.sol#L39-L41)

</details>