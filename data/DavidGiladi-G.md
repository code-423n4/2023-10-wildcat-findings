
### Gas Optimization Issues
|Title|Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
|[G-1] Inefficient use of abi.encode() | [Inefficient use of abi.encode()](#inefficient-use-of-abiencode) | 2 | 200 |
|[G-3] Avoid unnecessary storage updates | [Avoid unnecessary storage updates](#avoid-unnecessary-storage-updates) | 2 | 1600 |
|[G-4] Multiplication and Division by 2 Should use in Bit Shifting | [Multiplication and Division by 2 Should use in Bit Shifting](#multiplication-and-division-by-2-should-use-in-bit-shifting) | 4 | 80 |
|[G-7] Caching msg.sender Variable is Inefficiency | [Caching msg.sender Variable is Inefficiency](#caching-msgsender-variable-is-inefficiency) | 1 | 12 |
|[G-8] Check Arguments Early | [Check Arguments Early](#check-arguments-early) | 2 | - |
|[G-21] Optimal State Variable Order | [Optimal State Variable Order](#optimal-state-variable-order) | 3 | 15000 |
|[G-23] Consider activating via-ir for deploying | [Consider activating via-ir for deploying](#consider-activating-via-ir-for-deploying) | 1 | - |
|[G-24] Short-circuit rules can be used to optimize some gas usage | [Short-circuit rules can be used to optimize some gas usage](#short-circuit-rules-can-be-used-to-optimize-some-gas-usage) | 1 | 2100 |

Total:  8 issues


<br><br>
# Gas Optimization Issues



## Inefficient use of abi.encode()
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 200

### Description
The `abi.encode()` function is less gas efficient than `abi.encodePacked()`, especially when encoding only static types. This detector identifies instances where `abi.encode()` is used and all arguments are static, suggesting that `abi.encodePacked()` could potentially be used instead for better gas efficiency. Note: `abi.encodePacked()` should not be used if any of the arguments are dynamic types, as it could lead to hash collisions.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: src/WildcatSanctionsSentinel.sol

70    return
71          address(
72            uint160(
73              uint256(
74                keccak256(
75                  abi.encodePacked(
76                    bytes1(0xff),
77                    address(this),
78                    keccak256(abi.encode(borrower, account, asset)),
79                    WildcatSanctionsEscrowInitcodeHash
80                  )
81                )
82              )
83            )
84          )
```
 use  abi.encodepacked() instead.

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L70-L84](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L70-L84)


#

```
File: src/WildcatSanctionsSentinel.sol

110    new WildcatSanctionsEscrow{ salt: keccak256(abi.encode(borrower, account, asset)) }()
```
 use  abi.encodepacked() instead.

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L110](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L110)


</details>

# 



## Avoid unnecessary storage updates
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 1600

### Description
Avoid updating storage when the value hasn't changed. If the old value is equal to the new value, not re-storing the value will avoid a `SSTORE` operation (costing 2900 gas), potentially at the expense of a `SLOAD` operation (2100 gas) or a `WARMACCESS` operation (100 gas).

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#
The function `setProtocolFeeConfiguration()` changes the state variable without first verifying if the values are different.


```
File: src/WildcatMarketControllerFactory.sol

211    _protocolFeeConfiguration = ProtocolFeeConfiguration({
212          feeRecipient: feeRecipient,
213          originationFeeAsset: originationFeeAsset,
214          originationFeeAmount: originationFeeAmount,
215          protocolFeeBips: protocolFeeBips
216        })
```

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L211-L216](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L211-L216)


#
The function `updateAccountAuthorization()` changes the state variable without first verifying if the values are different.


```
File: src/market/WildcatMarketConfig.sol

123    _accounts[_account] = account
```

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L123](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L123)


</details>

# 



## Multiplication and Division by 2 Should use in Bit Shifting
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 80

### Description
The expressions 'x * 2' and 'x / 2' can be optimized for gas efficiency by utilizing bitwise operations. In Solidity, you can achieve the same results by using bitwise left shift (x << 1) for multiplication and bitwise right shift (x >> 1) for division.

Using bitwise shift operations (SHL and SHR) instead of multiplication (MUL) and division (DIV) opcodes can lead to significant gas savings. The MUL and DIV opcodes cost 5 gas, while the SHL and SHR opcodes incur a lower cost of only 3 gas.

By leveraging these more efficient bitwise operations, you can reduce the gas consumption of your smart contracts and enhance their overall performance.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
#

```
File: src/libraries/MathUtils.sol

108    if or(iszero(b), gt(a, div(sub(not(0), div(b, 2)), BIP))) {
109            mstore(0, Panic_ErrorSelector)
110            mstore(Panic_ErrorCodePointer, Panic_Arithmetic)
111            revert(Error_SelectorPointer, Panic_ErrorLength)
112          }
```
 instead `2` use bit shifting `1` 

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L108-L112](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L108-L112)


#

```
File: src/libraries/MathUtils.sol

114    c := div(add(mul(a, BIP), div(b, 2)), b)
```
 instead `2` use bit shifting `1` 

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L114](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L114)


#

```
File: src/libraries/MathUtils.sol

158    if or(iszero(b), gt(a, div(sub(not(0), div(b, 2)), RAY))) {
159            mstore(0, Panic_ErrorSelector)
160            mstore(Panic_ErrorCodePointer, Panic_Arithmetic)
161            revert(Error_SelectorPointer, Panic_ErrorLength)
162          }
```
 instead `2` use bit shifting `1` 

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L158-L162](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L158-L162)


#

```
File: src/libraries/MathUtils.sol

164    c := div(add(mul(a, RAY), div(b, 2)), b)
```
 instead `2` use bit shifting `1` 

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L164](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L164)


</details>

# 


## Caching msg.sender Variable is Inefficiency
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 12

### Description
Detects instances where `msg.sender` is cached into local variables within a function. Using `msg.sender` directly is always cheaper than caching it into a local variable. The reason being the `CALLER` operation (which reads the `msg.sender` global variable) costs 2 gas. On the other hand, the `MLOAD/MSTORE` operations, which are used for memory access (where local variables are stored), cost 3 gas. Caching and then accessing a global variable like `msg.sender` from a local variable, therefore, introduces an unnecessary overhead in terms of gas. You can check the gas costs for each operation [here](https://docs.google.com/spreadsheets/d/1n6mRqkBz3iWcOlRem_mO09GtSKEKrAsfO7Frgx18pNU/edit#gid=0).

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/WildcatMarketControllerFactory.sol

328    controller = deployController()
```

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L328](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L328)


</details>

# 


## Check Arguments Early
- Severity: Gas Optimization
- Confidence: High

### Description
Checks that require() or revert() statements that check input arguments are at the top of the function. Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a gas load in a function that may ultimately revert in the unhappy case.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: src/WildcatMarketControllerFactory.sol

80    constraints.minimumAnnualInterestBips > constraints.maximumAnnualInterestBips ||
81          constraints.maximumAnnualInterestBips > 10000 ||
82          constraints.minimumDelinquencyFeeBips > constraints.maximumDelinquencyFeeBips ||
83          constraints.maximumDelinquencyFeeBips > 10000 ||
84          constraints.minimumReserveRatioBips > constraints.maximumReserveRatioBips ||
85          constraints.maximumReserveRatioBips > 10000 ||
86          constraints.minimumDelinquencyGracePeriod > constraints.maximumDelinquencyGracePeriod ||
87          constraints.minimumWithdrawalBatchDuration > constraints.maximumWithdrawalBatchDuration
```

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L80-L87](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L80-L87)


#

```
File: src/market/WildcatMarketConfig.sol

152    _annualInterestBips > BIP
```

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L152](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L152)


</details>

# 



## Optimal State Variable Order
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 15000

### Description
Detects optimal variable order in contract storage layouts to decrease the number of storage slots used. Each storage slot used can cost at least 5,000 gas for each write operation, and potentially up to 20,000 gas if you're turning a zero value into a non-zero value. Hence, optimizing storage usage can result in significant gas savings. The real-world savings could vary depending on your contract's specific logic and the state of the Ethereum network.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#
Optimization opportunity in storage variable layout of contract 
```
File: src/WildcatMarketController.sol

32    contract WildcatMarketController is IWildcatMarketControllerEventsAndErrors 
```

- original variable order (count: 12 slots)
    - IWildcatArchController archController
    - IWildcatMarketControllerFactory controllerFactory
    - address borrower
    - address sentinel
    - address marketInitCodeStorage
    - uint256 marketInitCodeHash
    - uint256 ownCreate2Prefix
    - uint32 MinimumDelinquencyGracePeriod
    - uint32 MaximumDelinquencyGracePeriod
    - uint16 MinimumReserveRatioBips
    - uint16 MaximumReserveRatioBips
    - uint16 MinimumDelinquencyFeeBips
    - uint16 MaximumDelinquencyFeeBips
    - uint32 MinimumWithdrawalBatchDuration
    - uint32 MaximumWithdrawalBatchDuration
    - uint16 MinimumAnnualInterestBips
    - uint16 MaximumAnnualInterestBips
    - EnumerableSet.AddressSet _authorizedLenders
    - EnumerableSet.AddressSet _controlledMarkets
    - TmpMarketParameterStorage _tmpMarketParameters
    - mapping(address => TemporaryReserveRatio) temporaryExcessReserveRatio


 - optimized variable order (count: 11 slots)
    - IWildcatArchController archController
    - uint32 MinimumDelinquencyGracePeriod
    - uint32 MaximumDelinquencyGracePeriod
    - uint32 MinimumWithdrawalBatchDuration
    - IWildcatMarketControllerFactory controllerFactory
    - uint32 MaximumWithdrawalBatchDuration
    - uint16 MinimumReserveRatioBips
    - uint16 MaximumReserveRatioBips
    - uint16 MinimumDelinquencyFeeBips
    - uint16 MaximumDelinquencyFeeBips
    - address borrower
    - uint16 MinimumAnnualInterestBips
    - uint16 MaximumAnnualInterestBips
    - address sentinel
    - address marketInitCodeStorage
    - uint256 marketInitCodeHash
    - uint256 ownCreate2Prefix
    - EnumerableSet.AddressSet _authorizedLenders
    - EnumerableSet.AddressSet _controlledMarkets
    - TmpMarketParameterStorage _tmpMarketParameters
    - mapping(address => TemporaryReserveRatio) temporaryExcessReserveRatio


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L32-L516](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L32-L516)


#
Optimization opportunity in storage variable layout of contract 
```
File: src/WildcatMarketControllerFactory.sol

13    contract WildcatMarketControllerFactory 
```

- original variable order (count: 11 slots)
    - IWildcatArchController archController
    - address sentinel
    - address marketInitCodeStorage
    - uint256 marketInitCodeHash
    - address controllerInitCodeStorage
    - uint256 controllerInitCodeHash
    - uint256 ownCreate2Prefix
    - uint32 MinimumDelinquencyGracePeriod
    - uint32 MaximumDelinquencyGracePeriod
    - uint16 MinimumReserveRatioBips
    - uint16 MaximumReserveRatioBips
    - uint16 MinimumDelinquencyFeeBips
    - uint16 MaximumDelinquencyFeeBips
    - uint32 MinimumWithdrawalBatchDuration
    - uint32 MaximumWithdrawalBatchDuration
    - uint16 MinimumAnnualInterestBips
    - uint16 MaximumAnnualInterestBips
    - ProtocolFeeConfiguration _protocolFeeConfiguration
    - EnumerableSet.AddressSet _deployedControllers
    - address _tmpMarketBorrowerParameter


 - optimized variable order (count: 10 slots)
    - IWildcatArchController archController
    - uint32 MinimumDelinquencyGracePeriod
    - uint32 MaximumDelinquencyGracePeriod
    - uint32 MinimumWithdrawalBatchDuration
    - address sentinel
    - uint32 MaximumWithdrawalBatchDuration
    - uint16 MinimumReserveRatioBips
    - uint16 MaximumReserveRatioBips
    - uint16 MinimumDelinquencyFeeBips
    - uint16 MaximumDelinquencyFeeBips
    - address marketInitCodeStorage
    - uint16 MinimumAnnualInterestBips
    - uint16 MaximumAnnualInterestBips
    - address controllerInitCodeStorage
    - address _tmpMarketBorrowerParameter
    - uint256 marketInitCodeHash
    - uint256 controllerInitCodeHash
    - uint256 ownCreate2Prefix
    - ProtocolFeeConfiguration _protocolFeeConfiguration
    - EnumerableSet.AddressSet _deployedControllers


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L13-L348](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L13-L348)


#
Optimization opportunity in storage variable layout of contract 
```
File: src/market/WildcatMarketBase.sol

14    contract WildcatMarketBase is ReentrancyGuard, IMarketEventsAndErrors 
```

- original variable order (count: 16 slots)
    - string version
    - address sentinel
    - address borrower
    - address feeRecipient
    - uint256 protocolFeeBips
    - uint256 delinquencyFeeBips
    - uint256 delinquencyGracePeriod
    - address controller
    - address asset
    - uint256 withdrawalBatchDuration
    - uint8 decimals
    - string name
    - string symbol
    - MarketState _state
    - mapping(address => Account) _accounts
    - WithdrawalData _withdrawalData


 - optimized variable order (count: 15 slots)
    - address sentinel
    - uint8 decimals
    - address borrower
    - address feeRecipient
    - address controller
    - address asset
    - string version
    - uint256 protocolFeeBips
    - uint256 delinquencyFeeBips
    - uint256 delinquencyGracePeriod
    - uint256 withdrawalBatchDuration
    - string name
    - string symbol
    - MarketState _state
    - mapping(address => Account) _accounts
    - WithdrawalData _withdrawalData



Recomended optimizations will use 3 less slots.

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L14-L552](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L14-L552)


</details>

# 


## Consider activating via-ir for deploying
- Severity: Gas Optimization
- Confidence: Medium

### Description
The IR-based code generator was developed to make code generation more transparent and auditable, while also enabling powerful optimization passes that can be applied across functions. 

It is possible to activate the IR-based code generator through the command line by using the flag --via-ir or by including the option {'viaIR': true}. 

Keep in mind that compiling with this option may take longer. However, you can simply test it before deploying your code. If you find that it provides better performance, you can add the --via-ir flag to your deploy command.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#
 
</details>

# 


## Short-circuit rules can be used to optimize some gas usage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 2100

### Description
Some conditions may be reordered to save an SLOAD (2100 gas), as we avoid reading state variables when the first part of the condition fails (with &&), or succeeds (with ||). For instance, consider a scenario where you have a `stateVariable` (a variable stored in contract storage) and a `localVariable` (a variable in memory). 

If you have a condition like `stateVariable > 0 && localVariable > 0`, if `localVariable > 0` is false, the Solidity runtime will still execute `stateVariable > 0`, which costs an SLOAD operation (2100 gas). However, if you reorder the condition to `localVariable > 0 && stateVariable > 0`, the `stateVariable > 0` check won't happen if `localVariable > 0` is false, saving you the SLOAD gas cost.

Similarly, for the `||` operator, if you have a condition like `stateVariable > 0 || localVariable > 0`, and `stateVariable > 0` is true, the Solidity runtime will still execute `localVariable > 0`. But if you reorder the condition to `localVariable > 0 || stateVariable > 0`, and `localVariable > 0` is true, the `stateVariable > 0` check won't happen, again saving you the SLOAD gas cost.

This detector checks for such conditions in the contract and reports if any condition could be optimized by taking advantage of the short-circuiting behavior of `&&` and `||`.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/WildcatSanctionsSentinel.sol

40    return
41          !sanctionOverrides[borrower][account] &&
42          IChainalysisSanctionsList(chainalysisSanctionsList).isSanctioned(account)
```


```
 // @audit: Switch IChainalysisSanctionsList(chainalysisSanctionsList).isSanctioned(account) && ! sanctionOverrides[borrower][account] 
```
[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L40-L42](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L40-L42)


</details>

# 
