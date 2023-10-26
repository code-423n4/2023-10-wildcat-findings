## Summary

### Low Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[L-1] Casting block.timestamp can reduce the lifespan of a contract | [Casting block.timestamp can reduce the lifespan of a contract](#casting-blocktimestamp-can-reduce-the-lifespan-of-a-contract) | 1 |
|[L-2] Incorrect usage of using-for statement | [Incorrect usage of using-for statement](#incorrect-usage-of-using-for-statement) | 2 |
|[L-3] Reentrancy vulnerabilities | [Reentrancy vulnerabilities](#reentrancy-vulnerabilities) | 5 |
|[L-4] Code does not follow the best practice of check-effects-interaction | [Code does not follow the best practice of check-effects-interaction](#code-does-not-follow-the-best-practice-of-check-effects-interaction) | 20 |

Total: 4 issues


### Non-Critical Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[N-1] Function names should differ | [Function names should differ](#function-names-should-differ) | 9 |
|[N-2] Inconsistent method of specifying a floating pragma | [Inconsistent method of specifying a floating pragma](#inconsistent-method-of-specifying-a-floating-pragma) | 1 |
|[N-3] Library Should Be Defined in Separate Files From Their Usage | [Library Should Be Defined in Separate Files From Their Usage](#library-should-be-defined-in-separate-files-from-their-usage) | 4 |
|[N-4] Events are missing sender information | [Events are missing sender information](#events-are-missing-sender-information) | 14 |
|[N-5] Redundant Inheritance | [Redundant Inheritance](#redundant-inheritance) | 1 |
|[N-6] Event Emission Preceding External Calls: A Best Practice | [Event Emission Preceding External Calls: A Best Practice](#event-emission-preceding-external-calls-a-best-practice) | 2 |
|[N-7] Variable names too similar | [Variable names too similar](#variable-names-too-similar) | 10 |
|[N-8] Unnecessary Casting of Variables | [Unnecessary Casting of Variables](#unnecessary-casting-of-variables) | 1 |


Total:  8 issues
<br>
<br>
<br>


## Casting block.timestamp can reduce the lifespan of a contract
- Severity: Low
- Confidence: High

### Description
Casting `block.timestamp` to smaller integer types can reduce the effective lifespan of a contract. `block.timestamp` returns the current block timestamp as seconds since the unix epoch as `uint`. When it's cast to a smaller integer type, this effectively reduces the maximum timestamp value the contract can handle. As a result, the contract might stop functioning correctly after a certain point in time, even though `block.timestamp` itself will continue to increase with each block.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/market/WildcatMarketBase.sol

101    _state = MarketState({
102          isClosed: false,
103          maxTotalSupply: parameters.maxTotalSupply,
104          accruedProtocolFees: 0,
105          normalizedUnclaimedWithdrawals: 0,
106          scaledTotalSupply: 0,
107          scaledPendingWithdrawals: 0,
108          pendingWithdrawalExpiry: 0,
109          isDelinquent: false,
110          timeDelinquent: 0,
111          annualInterestBips: parameters.annualInterestBips,
112          reserveRatioBips: parameters.reserveRatioBips,
113          scaleFactor: uint112(RAY),
114          lastInterestAccruedTimestamp: uint32(block.timestamp)
115        })
```

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L101-L115](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L101-L115)


</details>

# 




## Incorrect usage of using-for statement
- Severity: Low
- Confidence: High

### Description
In Solidity, it is possible to use libraries for certain types, by the `using-for` statement (`using <library> for <type>`). However, the Solidity compiler doesn't check whether a given library has at least one function matching a given type. If it doesn't, such a statement has no effect and may be confusing. 

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: src/libraries/MarketState.sol

10    using MarketStateLib for Account global;
```
using-for statement is incorrect - no matching function for `Account` found in `MarketStateLib`

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L10](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L10)


#

```
File: src/libraries/Withdrawal.sol

10    using WithdrawalLib for WithdrawalData global;
```
using-for statement is incorrect - no matching function for `WithdrawalData` found in `WithdrawalLib`

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L10](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L10)


</details>

# 




## Reentrancy vulnerabilities
- Severity: Low
- Confidence: Medium

### Description

Detection of the [reentrancy bug](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy).
Only report reentrancy that acts as a double call (see `reentrancy-eth`, `reentrancy-no-eth`).

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
#
Reentrancy in 
```
File: src/market/WildcatMarketToken.sol

64    function _transfer(address from, address to, uint256 amount) internal virtual 
```
:
	External calls:
	- 
```
File: src/market/WildcatMarketToken.sol

65    MarketState memory state = _getUpdatedState()
```

		- 
```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```

	State variables written after the call(s):
	- 
```
File: src/market/WildcatMarketToken.sol

74    _accounts[from] = fromAccount
```

	- 
```
File: src/market/WildcatMarketToken.sol

78    _accounts[to] = toAccount
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L64-L82](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L64-L82)


#
Reentrancy in 
```
File: src/WildcatMarketController.sol

291    function deployMarket(
292        address asset,
293        string memory namePrefix,
294        string memory symbolPrefix,
295        uint128 maxTotalSupply,
296        uint16 annualInterestBips,
297        uint16 delinquencyFeeBips,
298        uint32 withdrawalBatchDuration,
299        uint16 reserveRatioBips,
300        uint32 delinquencyGracePeriod
301      ) external returns (address market) 
```
:
	External calls:
	- 
```
File: src/WildcatMarketController.sol

356    archController.registerMarket(market)
```

	State variables written after the call(s):
	- 
```
File: src/WildcatMarketController.sol

359    _resetTmpMarketParameters()
```

		- 
```
File: src/WildcatMarketController.sol

256    _tmpMarketParameters.asset = address(1)
```

		- 
```
File: src/WildcatMarketController.sol

257    _tmpMarketParameters.namePrefix = '_'
```

		- 
```
File: src/WildcatMarketController.sol

258    _tmpMarketParameters.symbolPrefix = '_'
```

		- 
```
File: src/WildcatMarketController.sol

259    _tmpMarketParameters.feeRecipient = address(1)
```

		- 
```
File: src/WildcatMarketController.sol

260    _tmpMarketParameters.protocolFeeBips = 1
```

		- 
```
File: src/WildcatMarketController.sol

261    _tmpMarketParameters.maxTotalSupply = 1
```

		- 
```
File: src/WildcatMarketController.sol

262    _tmpMarketParameters.annualInterestBips = 1
```

		- 
```
File: src/WildcatMarketController.sol

263    _tmpMarketParameters.delinquencyFeeBips = 1
```

		- 
```
File: src/WildcatMarketController.sol

264    _tmpMarketParameters.withdrawalBatchDuration = 1
```

		- 
```
File: src/WildcatMarketController.sol

265    _tmpMarketParameters.reserveRatioBips = 1
```

		- 
```
File: src/WildcatMarketController.sol

266    _tmpMarketParameters.delinquencyGracePeriod = 1
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L291-L360](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L291-L360)


#
Reentrancy in 
```
File: src/market/WildcatMarket.sol

42    function depositUpTo(
43        uint256 amount
44      ) public virtual nonReentrant returns (uint256 /* actualAmount */) 
```
:
	External calls:
	- 
```
File: src/market/WildcatMarket.sol

46    MarketState memory state = _getUpdatedState()
```

		- 
```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```

	State variables written after the call(s):
	- 
```
File: src/market/WildcatMarket.sol

65    _accounts[msg.sender] = account
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L42-L77](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L42-L77)


#
Reentrancy in 
```
File: src/market/WildcatMarketWithdrawals.sol

77    function queueWithdrawal(uint256 amount) external nonReentrant 
```
:
	External calls:
	- 
```
File: src/market/WildcatMarketWithdrawals.sol

78    MarketState memory state = _getUpdatedState()
```

		- 
```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```

	State variables written after the call(s):
	- 
```
File: src/market/WildcatMarketWithdrawals.sol

90    _accounts[msg.sender] = account
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L77-L121](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L77-L121)


#
Reentrancy in 
```
File: src/market/WildcatMarketConfig.sol

112    function updateAccountAuthorization(
113        address _account,
114        bool _isAuthorized
115      ) external onlyController nonReentrant 
```
:
	External calls:
	- 
```
File: src/market/WildcatMarketConfig.sol

116    MarketState memory state = _getUpdatedState()
```

		- 
```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```

	State variables written after the call(s):
	- 
```
File: src/market/WildcatMarketConfig.sol

123    _accounts[_account] = account
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L112-L126](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L112-L126)


</details>

# 


## Code does not follow the best practice of check-effects-interaction
- Severity: Low
- Confidence: Medium

### Description

Code should follow the best-practice of 
[check-effects-interaction](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/), 
where state variables are updated before any external calls are made. 
Doing so prevents a large class of reentrancy bugs.


<details>

<summary>
There are 20 instances of this issue:

</summary>

###
#

```
File: src/market/WildcatMarketBase.sol

163    function _blockAccount(MarketState memory state, address accountAddress) internal 
```
External calls:

```
File: src/market/WildcatMarketBase.sol

172    address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
173              accountAddress,
174              borrower,
175              address(this)
176            )
```

State variables written after the call(s):

```
File: src/market/WildcatMarketBase.sol

178    _accounts[escrow].scaledBalance += scaledBalance
```


```
File: src/market/WildcatMarketBase.sol

185    _accounts[accountAddress] = account
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L163-L187](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L163-L187)


#

```
File: src/market/WildcatMarketToken.sol

64    function _transfer(address from, address to, uint256 amount) internal virtual 
```
External calls:

```
File: src/market/WildcatMarketToken.sol

65    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarketToken.sol

80    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarketToken.sol

80    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L64-L82](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L64-L82)


#

```
File: src/market/WildcatMarket.sol

119    function borrow(uint256 amount) external onlyBorrower nonReentrant 
```
External calls:

```
File: src/market/WildcatMarket.sol

120    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarket.sol

128    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarket.sol

128    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L119-L131](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L119-L131)


#

```
File: src/market/WildcatMarket.sol

142    function closeMarket() external onlyController nonReentrant 
```
External calls:

```
File: src/market/WildcatMarket.sol

143    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarket.sol

159    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarket.sol

159    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L142-L161](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L142-L161)


#

```
File: src/market/WildcatMarket.sol

96    function collectFees() external nonReentrant 
```
External calls:

```
File: src/market/WildcatMarket.sol

97    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarket.sol

106    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarket.sol

106    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L96-L109](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L96-L109)


#

```
File: src/market/WildcatMarket.sol

42    function depositUpTo(
43        uint256 amount
44      ) public virtual nonReentrant returns (uint256 /* actualAmount */) 
```
External calls:

```
File: src/market/WildcatMarket.sol

46    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarket.sol

74    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarket.sol

74    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L42-L77](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L42-L77)


#

```
File: src/market/WildcatMarketWithdrawals.sol

137    function executeWithdrawal(
138        address accountAddress,
139        uint32 expiry
140      ) external nonReentrant returns (uint256) 
```
External calls:

```
File: src/market/WildcatMarketWithdrawals.sol

144    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```

State variables written after the call(s):

```
File: src/market/WildcatMarketWithdrawals.sol

157    status.normalizedAmountWithdrawn = newTotalWithdrawn
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L137-L188](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L137-L188)


#

```
File: src/market/WildcatMarketWithdrawals.sol

137    function executeWithdrawal(
138        address accountAddress,
139        uint32 expiry
140      ) external nonReentrant returns (uint256) 
```
External calls:

```
File: src/market/WildcatMarketWithdrawals.sol

144    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarketWithdrawals.sol

165    _blockAccount(state, accountAddress)
```


```
File: src/market/WildcatMarketBase.sol

172    address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
173              accountAddress,
174              borrower,
175              address(this)
176            )
```


```
File: src/market/WildcatMarketWithdrawals.sol

166    address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
167            accountAddress,
168            borrower,
169            address(asset)
170          )
```


```
File: src/market/WildcatMarketWithdrawals.sol

185    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarketWithdrawals.sol

185    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L137-L188](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L137-L188)


#

```
File: src/market/WildcatMarketConfig.sol

74    function nukeFromOrbit(address accountAddress) external nonReentrant 
```
External calls:

```
File: src/market/WildcatMarketConfig.sol

78    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarketConfig.sol

79    _blockAccount(state, accountAddress)
```


```
File: src/market/WildcatMarketBase.sol

172    address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
173              accountAddress,
174              borrower,
175              address(this)
176            )
```


```
File: src/market/WildcatMarketConfig.sol

80    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarketConfig.sol

80    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L74-L81](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L74-L81)


#

```
File: src/market/WildcatMarketWithdrawals.sol

190    function processUnpaidWithdrawalBatch() external nonReentrant 
```
External calls:

```
File: src/market/WildcatMarketWithdrawals.sol

191    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```

State variables written after the call(s):

```
File: src/market/WildcatMarketWithdrawals.sol

212    _withdrawalData.batches[expiry] = batch
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L190-L214](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L190-L214)


#

```
File: src/market/WildcatMarketWithdrawals.sol

190    function processUnpaidWithdrawalBatch() external nonReentrant 
```
External calls:

```
File: src/market/WildcatMarketWithdrawals.sol

191    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarketWithdrawals.sol

213    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarketWithdrawals.sol

213    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L190-L214](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L190-L214)


#

```
File: src/market/WildcatMarketWithdrawals.sol

77    function queueWithdrawal(uint256 amount) external nonReentrant 
```
External calls:

```
File: src/market/WildcatMarketWithdrawals.sol

78    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```

State variables written after the call(s):

```
File: src/market/WildcatMarketWithdrawals.sol

104    _withdrawalData.accountStatuses[expiry][msg.sender].scaledAmount += scaledAmount
```


```
File: src/market/WildcatMarketWithdrawals.sol

117    _withdrawalData.batches[expiry] = batch
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L77-L121](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L77-L121)


#

```
File: src/market/WildcatMarketWithdrawals.sol

77    function queueWithdrawal(uint256 amount) external nonReentrant 
```
External calls:

```
File: src/market/WildcatMarketWithdrawals.sol

78    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarketWithdrawals.sol

120    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarketWithdrawals.sol

120    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L77-L121](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L77-L121)


#

```
File: src/WildcatMarketController.sol

490    function resetReserveRatio(address market) external virtual 
```
External calls:

```
File: src/WildcatMarketController.sol

499    WildcatMarket(market).setReserveRatioBips(uint256(tmp.reserveRatioBips).toUint16())
```

State variables written after the call(s):

```
File: src/WildcatMarketController.sol

500    delete temporaryExcessReserveRatio[market]
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L490-L501](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L490-L501)


#

```
File: src/WildcatMarketController.sol

468    function setAnnualInterestBips(
469        address market,
470        uint16 annualInterestBips
471      ) external virtual onlyBorrower onlyControlledMarket(market) 
```
External calls:

```
File: src/WildcatMarketController.sol

481    WildcatMarket(market).setReserveRatioBips(9000)
```

State variables written after the call(s):

```
File: src/WildcatMarketController.sol

484    tmp.expiry = uint128(block.timestamp + 2 weeks)
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L468-L488](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L468-L488)


#

```
File: src/market/WildcatMarketConfig.sol

149    function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant 
```
External calls:

```
File: src/market/WildcatMarketConfig.sol

150    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarketConfig.sol

157    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarketConfig.sol

157    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L149-L159](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L149-L159)


#

```
File: src/market/WildcatMarketConfig.sol

134    function setMaxTotalSupply(uint256 _maxTotalSupply) external onlyController nonReentrant 
```
External calls:

```
File: src/market/WildcatMarketConfig.sol

135    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarketConfig.sol

142    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarketConfig.sol

142    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L134-L144](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L134-L144)


#

```
File: src/market/WildcatMarketConfig.sol

171    function setReserveRatioBips(uint16 _reserveRatioBips) public onlyController nonReentrant 
```
External calls:

```
File: src/market/WildcatMarketConfig.sol

176    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarketConfig.sol

191    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarketConfig.sol

191    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L171-L193](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L171-L193)


#

```
File: src/market/WildcatMarketConfig.sol

112    function updateAccountAuthorization(
113        address _account,
114        bool _isAuthorized
115      ) external onlyController nonReentrant 
```
External calls:

```
File: src/market/WildcatMarketConfig.sol

116    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarketConfig.sol

124    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarketConfig.sol

124    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L112-L126](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L112-L126)


#

```
File: src/market/WildcatMarket.sol

26    function updateState() external nonReentrant 
```
External calls:

```
File: src/market/WildcatMarket.sol

27    MarketState memory state = _getUpdatedState()
```


```
File: src/libraries/FeeMath.sol

50    state.accruedProtocolFees = (state.accruedProtocolFees + protocolFee).toUint128()
```


```
File: src/market/WildcatMarket.sol

28    _writeState(state)
```


```
File: src/libraries/MarketState.sol

91    uint256 scaledRequiredReserves = (state.scaledTotalSupply - scaledWithdrawals).bipMul(
92          state.reserveRatioBips
93        ) + scaledWithdrawals
```

State variables written after the call(s):

```
File: src/market/WildcatMarket.sol

28    _writeState(state)
```


```
File: src/market/WildcatMarketBase.sol

451    _state = state
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L26-L29](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L26-L29)


</details>

# 




## Redundant Inheritance
- Severity: Non-Critical
- Confidence: High

### Description
Detects when a contract inherits from another contract that it already indirectly inherits from.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/market/WildcatMarket.sol

10    contract WildcatMarket is
11      WildcatMarketBase,
12      WildcatMarketConfig,
13      WildcatMarketToken,
14      WildcatMarketWithdrawals
15    
```
 The contract `WildcatMarket` has redundant inheritance: `WildcatMarketBase` is already inherited indirectly.

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L10-L162](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L10-L162)


</details>

# 


## Function names should differ
- Severity: Non-Critical
- Confidence: High

### Description
In Solidity, while function overriding allows for functions with the same name to coexist, it is advisable to avoid this practice to enhance code readability and maintainability. Having multiple functions with the same name, even with different parameters or in inherited contracts, can cause confusion and increase the likelihood of errors during development, testing, and debugging.

<details>

<summary>
There are 9 instances of this issue:

</summary>

###
#

```
File: src/WildcatArchController.sol

85    function getRegisteredBorrowers(
86        uint256 start,
87        uint256 end
88      ) external view returns (address[] memory arr) 
```
```
/// @audit: function used on lines  81
```
[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L85-L96](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L85-L96)


#

```
File: src/WildcatArchController.sol

128    function getRegisteredControllerFactories(
129        uint256 start,
130        uint256 end
131      ) external view returns (address[] memory arr) 
```
```
/// @audit: function used on lines  124
```
[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L128-L139](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L128-L139)


#

```
File: src/WildcatArchController.sol

171    function getRegisteredControllers(
172        uint256 start,
173        uint256 end
174      ) external view returns (address[] memory arr) 
```
```
/// @audit: function used on lines  167
```
[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L171-L182](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L171-L182)


#

```
File: src/WildcatArchController.sol

214    function getRegisteredMarkets(
215        uint256 start,
216        uint256 end
217      ) external view returns (address[] memory arr) 
```
```
/// @audit: function used on lines  210
```
[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L214-L225](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L214-L225)


#

```
File: src/WildcatMarketController.sol

125    function getAuthorizedLenders(
126        uint256 start,
127        uint256 end
128      ) external view returns (address[] memory arr) 
```
```
/// @audit: function used on lines  121
```
[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L125-L136](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L125-L136)


#

```
File: src/WildcatMarketController.sol

204    function getControlledMarkets(
205        uint256 start,
206        uint256 end
207      ) external view returns (address[] memory arr) 
```
```
/// @audit: function used on lines  200
```
[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L204-L215](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L204-L215)


#

```
File: src/WildcatMarketControllerFactory.sol

138    function getDeployedControllers(
139        uint256 start,
140        uint256 end
141      ) external view returns (address[] memory arr) 
```
```
/// @audit: function used on lines  134
```
[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L138-L149](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L138-L149)


#

```
File: src/libraries/LibStoredInitCode.sol

87    function createWithStoredInitCode(
88        address initCodeStorage,
89        uint256 value
90      ) internal returns (address deployment) 
```
```
/// @audit: function used on lines  83
```
[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol#L87-L97](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol#L87-L97)


#

```
File: src/libraries/LibStoredInitCode.sol

106    function create2WithStoredInitCode(
107        address initCodeStorage,
108        bytes32 salt,
109        uint256 value
110      ) internal returns (address deployment) 
```
```
/// @audit: function used on lines  99
```
[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol#L106-L117](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol#L106-L117)


</details>

# 



## Inconsistent method of specifying a floating pragma
- Severity: Non-Critical
- Confidence: High

### Description
This detector checks for instances where some files use >=, and others use ^ to specify a floating pragma. The reported instances are examples of the method that has the fewest instances for a specific version, and only one of the other style is present in the codebase. 

Note that using >= without also specifying <= will lead to failures to compile, or external project incompatibility, when the major version changes and there are breaking changes. So ^ should be preferred regardless of the instance counts.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#
This is the most common style:

```
File: src/ReentrancyGuard.sol

2    pragma solidity >=0.8.20;
```

<br>

```
File: src/libraries/Chainalysis.sol

2    pragma solidity ^0.8.20;
```

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/ReentrancyGuard.sol#L2](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/ReentrancyGuard.sol#L2)


</details>

# 



## Library Should Be Defined in Separate Files From Their Usage
- Severity: Non-Critical
- Confidence: High

### Description
Library should be defined in separate files, so that it's easier for future projects to import them, and to avoid duplication later on if they need to be used elsewhere in the project.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
#

```
File: src/libraries/FIFOQueue.sol

16    library FIFOQueueLib 
```

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L16-L80](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L16-L80)


#

```
File: src/libraries/MarketState.sol

44    library MarketStateLib 
```

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L44-L144](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L44-L144)


#

```
File: src/libraries/MathUtils.sol

16    library MathUtils 
```

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L16-L203](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L16-L203)


#

```
File: src/libraries/Withdrawal.sol

37    library WithdrawalLib 
```

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L37-L60](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L37-L60)


</details>

# 



## Events are missing sender information
- Severity: Non-Critical
- Confidence: High

### Description
When an action is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including the msg.sender in the events of these types of action will make events much more useful to end users, especially when msg.sender is not tx.origin.

<details>

<summary>
There are 14 instances of this issue:

</summary>

###
#

```
File: src/market/WildcatMarketBase.sol

373    emit ScaleFactorUpdated(state.scaleFactor, baseInterestRay, delinquencyFeeRay, protocolFee)
```
  The event `ScaleFactorUpdated` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L373](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L373)


#

```
File: src/market/WildcatMarketBase.sol

386    emit ScaleFactorUpdated(state.scaleFactor, baseInterestRay, delinquencyFeeRay, protocolFee)
```
  The event `ScaleFactorUpdated` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L386](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L386)


#

```
File: src/market/WildcatMarketBase.sol

452    emit StateUpdated(state.scaleFactor, isDelinquent)
```
  The event `StateUpdated` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L452](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L452)


#

```
File: src/market/WildcatMarketBase.sol

476    emit WithdrawalBatchExpired(
477          expiry,
478          batch.scaledTotalAmount,
479          batch.scaledAmountBurned,
480          batch.normalizedAmountPaid
481        )
```
  The event `WithdrawalBatchExpired` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L476-L481](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L476-L481)


#

```
File: src/market/WildcatMarketBase.sol

486    emit WithdrawalBatchClosed(expiry)
```
  The event `WithdrawalBatchClosed` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L486](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L486)


#

```
File: src/market/WildcatMarketBase.sol

525    emit WithdrawalBatchPayment(expiry, scaledAmountBurned, normalizedAmountPaid)
```
  The event `WithdrawalBatchPayment` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L525](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L525)


#

```
File: src/market/WildcatMarketWithdrawals.sol

96    emit WithdrawalBatchCreated(state.pendingWithdrawalExpiry)
```
  The event `WithdrawalBatchCreated` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L96](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L96)


#

```
File: src/market/WildcatMarketWithdrawals.sol

208    emit WithdrawalBatchClosed(expiry)
```
  The event `WithdrawalBatchClosed` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L208](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L208)


#

```
File: src/market/WildcatMarketConfig.sol

143    emit MaxTotalSupplyUpdated(_maxTotalSupply)
```
  The event `MaxTotalSupplyUpdated` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L143](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L143)


#

```
File: src/market/WildcatMarketConfig.sol

158    emit AnnualInterestBipsUpdated(_annualInterestBips)
```
  The event `AnnualInterestBipsUpdated` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L158](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L158)


#

```
File: src/market/WildcatMarketConfig.sol

192    emit ReserveRatioBipsUpdated(_reserveRatioBips)
```
  The event `ReserveRatioBipsUpdated` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L192](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L192)


#

```
File: src/market/WildcatMarket.sol

108    emit FeesCollected(withdrawableFees)
```
  The event `FeesCollected` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L108](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L108)


#

```
File: src/market/WildcatMarket.sol

130    emit Borrow(amount)
```
  The event `Borrow` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L130](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L130)


#

```
File: src/market/WildcatMarket.sol

160    emit MarketClosed(block.timestamp)
```
  The event `MarketClosed` emits an event without including the sender's address:

[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L160](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L160)


</details>

# 


## Event Emission Preceding External Calls: A Best Practice
- Severity: Non-Critical
- Confidence: Medium

### Description

Ensure that events follow the best practice of check-effects-interaction, and are emitted before external calls


<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#
Reentrancy in 
```
File: src/market/WildcatMarketBase.sol

163    function _blockAccount(MarketState memory state, address accountAddress) internal 
```
:
	External calls:
	- 
```
File: src/market/WildcatMarketBase.sol

172    address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
173              accountAddress,
174              borrower,
175              address(this)
176            )
```

	Event emitted after the call(s):
	- 
```
File: src/market/WildcatMarketBase.sol

179    emit SanctionedAccountAssetsSentToEscrow(
180              accountAddress,
181              escrow,
182              state.normalizeAmount(scaledBalance)
183            )
```

	- 
```
File: src/market/WildcatMarketBase.sol

177    emit Transfer(accountAddress, escrow, state.normalizeAmount(scaledBalance))
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L163-L187](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L163-L187)


#
Reentrancy in 
```
File: src/WildcatSanctionsEscrow.sol

33    function releaseEscrow() public override 
```
:
	External calls:
	- 
```
File: src/WildcatSanctionsEscrow.sol

38    IERC20(asset).transfer(account, amount)
```

	Event emitted after the call(s):
	- 
```
File: src/WildcatSanctionsEscrow.sol

40    emit EscrowReleased(account, asset, amount)
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L33-L41](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L33-L41)


</details>

# 


## Variable names too similar
- Severity: Non-Critical
- Confidence: Medium

### Description
Detect variables with names that are too similar.

<details>

<summary>
There are 10 instances of this issue:

</summary>

###
#
Variable 
```
File: src/WildcatMarketController.sol

68    uint16 internal immutable MaximumAnnualInterestBips
```
 is too similar to 
```
File: src/WildcatMarketController.sol

67    uint16 internal immutable MinimumAnnualInterestBips
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L68](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L68)


#
Variable 
```
File: src/WildcatMarketController.sol

62    uint16 internal immutable MaximumDelinquencyFeeBips
```
 is too similar to 
```
File: src/WildcatMarketController.sol

61    uint16 internal immutable MinimumDelinquencyFeeBips
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L62](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L62)


#
Variable 
```
File: src/WildcatMarketController.sol

56    uint32 internal immutable MaximumDelinquencyGracePeriod
```
 is too similar to 
```
File: src/WildcatMarketController.sol

55    uint32 internal immutable MinimumDelinquencyGracePeriod
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L56](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L56)


#
Variable 
```
File: src/WildcatMarketController.sol

59    uint16 internal immutable MaximumReserveRatioBips
```
 is too similar to 
```
File: src/WildcatMarketController.sol

58    uint16 internal immutable MinimumReserveRatioBips
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L59](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L59)


#
Variable 
```
File: src/WildcatMarketController.sol

65    uint32 internal immutable MaximumWithdrawalBatchDuration
```
 is too similar to 
```
File: src/WildcatMarketController.sol

64    uint32 internal immutable MinimumWithdrawalBatchDuration
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L65](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L65)


#
Variable 
```
File: src/WildcatMarketControllerFactory.sol

59    uint16 internal immutable MaximumAnnualInterestBips
```
 is too similar to 
```
File: src/WildcatMarketControllerFactory.sol

58    uint16 internal immutable MinimumAnnualInterestBips
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L59](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L59)


#
Variable 
```
File: src/WildcatMarketControllerFactory.sol

53    uint16 internal immutable MaximumDelinquencyFeeBips
```
 is too similar to 
```
File: src/WildcatMarketControllerFactory.sol

52    uint16 internal immutable MinimumDelinquencyFeeBips
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L53](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L53)


#
Variable 
```
File: src/WildcatMarketControllerFactory.sol

47    uint32 internal immutable MaximumDelinquencyGracePeriod
```
 is too similar to 
```
File: src/WildcatMarketControllerFactory.sol

46    uint32 internal immutable MinimumDelinquencyGracePeriod
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L47](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L47)


#
Variable 
```
File: src/WildcatMarketControllerFactory.sol

50    uint16 internal immutable MaximumReserveRatioBips
```
 is too similar to 
```
File: src/WildcatMarketControllerFactory.sol

49    uint16 internal immutable MinimumReserveRatioBips
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L50](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L50)


#
Variable 
```
File: src/WildcatMarketControllerFactory.sol

56    uint32 internal immutable MaximumWithdrawalBatchDuration
```
 is too similar to 
```
File: src/WildcatMarketControllerFactory.sol

55    uint32 internal immutable MinimumWithdrawalBatchDuration
```


[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L56](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L56)


</details>

# 



## Unnecessary Casting of Variables
- Severity: Non-Critical
- Confidence: High

### Description
This detector scans for instances where a variable is casted to its own type. This is unnecessary and can be safely removed to improve code readability.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/market/WildcatMarketWithdrawals.sol

166    address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
167            accountAddress,
168            borrower,
169            address(asset)
170          )
```
Unnecessary cast: `address(asset)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L166-L170](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L166-L170)


</details>

# 
