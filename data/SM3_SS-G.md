


## Gas Optimizations

| Number | Issue | Instances |
|--------|-------|-----------|
|[G-01]| Can make the variable outside the loop to save gas  |  3  |
|[G-02]| Use the existing memory value  |  4  |
|[G-03]| The creation of an intermediary array can be avoided  |  4  |
|[G-04]| Use != 0 instead of > 0 for unsigned integer comparison  | 12   |
|[G-05]| Loop best practice to save gas  |  9  |
|[G-06]| Internal functions that are not called by the contract should be removed to save deployment gas  |  60  |
|[G-07]| Counting down in for statements is more gas efficient  |  9  |
|[G-08]| Use do while loops instead of for loops  |  6  |
|[G-09]| storage variable can be replaced with calldata variable  |  5  |
|[G-10]| Conditional check can be performed during variable initialization  |  1  |
|[G-11]| Cache external calls outside of loop to avoid re-calling function on each iteration  |  1  |
|[G-12]| Duplicated require()/revert() Checks Should Be Refactored To A Modifier Or Function  | 7  |
|[G-13]| Combine events to save 2 Glogtopic (375 gas)  |  2  |
|[G-14]| Failure to check the zero address in the constructor causes the contract to be deployed again  |  2  |
|[G-15]| bytes constants are more eficient than string constans  |  3  |

## [G-1] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file: src/WildcatMarketController.sol

155   address lender = lenders[i];

171   address lender = lenders[i];

184   address market = markets[i];

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L155


## [G-2] Use the existing memory value


### use the existine memory value of  arr.nextIndex instead of using arr.nextIndex in all three function to do addition

```solidity
file:  src/libraries/FIFOQueue.sol

55    function push(FIFOQueue storage arr, uint32 value) internal {
    uint128 nextIndex = arr.nextIndex;
    arr.data[nextIndex] = value;
    arr.nextIndex = nextIndex + 1;
  }

61 function shift(FIFOQueue storage arr) internal {
    uint128 startIndex = arr.startIndex;
    if (startIndex == arr.nextIndex) {
      revert FIFOQueueOutOfBounds();
    }
    delete arr.data[startIndex];
    arr.startIndex = startIndex + 1;
  }

70 function shiftN(FIFOQueue storage arr, uint128 n) internal {
    uint128 startIndex = arr.startIndex;
    if (startIndex + n > arr.nextIndex) {
      revert FIFOQueueOutOfBounds();
    }
    for (uint256 i = 0; i < n; i++) {
      delete arr.data[startIndex + i];
    }
    arr.startIndex = startIndex + n;
  }


```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L55-L59

### use the existine memory value of  state.reserveRatioBips instead of using state.reserveRatioBips 

```solidity
file: src/market/WildcatMarketConfig.sol

171    function setReserveRatioBips(uint16 _reserveRatioBips) public onlyController nonReentrant {
    if (_reserveRatioBips > BIP) {
      revert ReserveRatioBipsTooHigh();
    }

    MarketState memory state = _getUpdatedState();

    uint256 initialReserveRatioBips = state.reserveRatioBips;

    if (_reserveRatioBips < initialReserveRatioBips) {
      if (state.liquidityRequired() > totalAssets()) {
        revert InsufficientReservesForOldLiquidityRatio();
      }
    }
    state.reserveRatioBips = _reserveRatioBips;
    if (_reserveRatioBips > initialReserveRatioBips) {
      if (state.liquidityRequired() > totalAssets()) {
        revert InsufficientReservesForNewLiquidityRatio();
      }
    }
    _writeState(state);
    emit ReserveRatioBipsUpdated(_reserveRatioBips);
  }
}

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L171-L194


## [G-3] The creation of an intermediary array can be avoided

Sometimes, it’s not necessary to create an intermediate array to store values.

```solidity
file: src/WildcatArchController.sol

81    function getRegisteredBorrowers() external view returns (address[] memory) {

88    ) external view returns (address[] memory arr) {

124   function getRegisteredControllerFactories() external view returns (address[] memory) {

167   function getRegisteredControllers() external view returns (address[] memory) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L81


## [G-4] Use != 0 instead of > 0 for unsigned integer comparison

It’s generally more gas-efficient to use != 0 instead of > 0 when comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation, while the > 0 comparison requires an additional subtraction operation. As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.


```solidity
file:  src/WildcatMarketControllerFactory.sol

201    bool hasOriginationFee = originationFeeAmount > 0;

205    (protocolFeeBips > 0 && nullFeeRecipient) ||

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L201

```solidity
file: src/libraries/FeeMath.sol

67    if (timeWithPenalty > 0) {

155   if (protocolFeeBips > 0) {

159   if (delinquencyFeeBips > 0) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L67

```solidity
file:  src/market/WildcatMarket.sol
 
147   if (_withdrawalData.unpaidBatches.length() > 0) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L147

```solidity
file: src/market/WildcatMarketBase.sol

79     if ((parameters.protocolFeeBips > 0).and(parameters.feeRecipient == address(0))) {

170    if (scaledBalance > 0) {

428    if (availableLiquidity > 0) {

472    if (availableLiquidity > 0) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L79

```solidity
file:  src/market/WildcatMarketWithdrawals.sol

32     if ((expiry == expiredBatchExpiry).and(expiry > 0)) {

112    if (availableLiquidity > 0) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L32


## [G-5] Loop best practice to save gas

```solidity
function Plusi() public view {
		for(uint i=0; i<10;) {
			console.log("Number ==",i);
			unchecked{
				++i;
			}
		}
	}
best practice
-----------------------------------------------------
for (uint i = 0; i < length; i = unchecked_inc(i)) {
    // do something that doesn't change the value of i
}
function unchecked_inc(uint i) returns (uint) {
    unchecked {
        return i + 1;
    }
}

```


```solidity
file: src/WildcatArchController.sol

93   for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }

136  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllerFactories.at(start + i);
    }

179  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllers.at(start + i);
    }

222 for (uint256 i = 0; i < count; i++) {
      arr[i] = _markets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93-L95


```solidity
file:  src/WildcatMarketController.sol

133     for (uint256 i = 0; i < count; i++) {
      arr[i] = _authorizedLenders.at(start + i);
    }

154  for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.add(lender)) {
        emit LenderAuthorized(lender);
      }
    
170    for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.remove(lender)) {
        emit LenderDeauthorized(lender);
      }
    }

183    for (uint256 i; i < markets.length; i++) {
      address market = markets[i];
      if (!_controlledMarkets.contains(market)) {
        revert NotControlledMarket();
      }
      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
    }


212  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controlledMarkets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133-L135



## [G-6] Internal functions that are not called by the contract should be removed to save deployment gas

Internal functions in Solidity are only intended to be invoked within the contract or by other internal functions. If an internal function is not called anywhere within the contract, it serves no purpose and contributes unnecessary overhead during deployment. Removing such functions can lead to substantial gas savings.


```solidity
file: src/libraries/FIFOQueue.sol

19:   function empty(FIFOQueue storage arr) internal view returns (bool) {

23:   function first(FIFOQueue storage arr) internal view returns (uint32) {

30:   function at(FIFOQueue storage arr, uint256 index) internal view returns (uint32) {

38:   function length(FIFOQueue storage arr) internal view returns (uint128) {

42:   function values(FIFOQueue storage arr) internal view returns (uint32[] memory _values) {

55:   function push(FIFOQueue storage arr, uint32 value) internal {

61:   function shift(FIFOQueue storage arr) internal {

70:   function shiftN(FIFOQueue storage arr, uint128 n) internal {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L19

```solidity
file:  src/libraries/FeeMath.sol

30:   function calculateBaseInterest(
        MarketState memory state,
        uint256 timestamp
     ) internal pure returns (uint256 baseInterestRay) {

40:   function applyProtocolFee(
41:     MarketState memory state,
42:     uint256 baseInterestRay,
43:     uint256 protocolFeeBips
44:   ) internal pure returns (uint256 protocolFee) {

53:   function updateDelinquency(
54:     MarketState memory state,
55:     uint256 timestamp,
56:     uint256 delinquencyFeeBips,
57:     uint256 delinquencyGracePeriod
58:   ) internal pure returns (uint256 delinquencyFeeRay) {

142:   function updateScaleFactorAndFees(
143:     MarketState memory state,
144:     uint256 protocolFeeBips,
145:     uint256 delinquencyFeeBips,
146:     uint256 delinquencyGracePeriod,
147:     uint256 timestamp
148:   )
149:     internal
150:     pure
151:     returns (uint256 baseInterestRay, uint256 delinquencyFeeRay, uint256 protocolFee)
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L30

```solidity
file: src/libraries/LibStoredInitCode.sol

7:   function deployInitCode(bytes memory data) internal returns (address initCodeStorage) {

48:   function getCreate2Prefix(address deployer) internal pure returns (uint256 create2Prefix) {

54:   function calculateCreate2Address(
55:     uint256 create2Prefix,
56:     bytes32 salt,
57:     uint256 initCodeHash
58:   ) internal pure returns (address create2Address) {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol#L7

```solidity
file: src/libraries/MarketState.sol

51:   function totalSupply(MarketState memory state) internal pure returns (uint256) {

59:   function maximumDeposit(MarketState memory state) internal pure returns (uint256) {

66:   function normalizeAmount(
67:     MarketState memory state,
68:     uint256 amount
69:   ) internal pure returns (uint256) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L51

```solidity
file: src/libraries/MathUtils.sol

30:   function calculateLinearInterestFromBips(
31:     uint256 rateBip,
32:     uint256 timeDelta
33:   ) internal pure returns (uint256 result) {

44:   function min(uint256 a, uint256 b) internal pure returns (uint256 c) {

51:   function max(uint256 a, uint256 b) internal pure returns (uint256 c) {

59:   function satSub(uint256 a, uint256 b) internal pure returns (uint256 c) {

85:   function bipMul(uint256 a, uint256 b) internal pure returns (uint256 c) {

105:   function bipDiv(uint256 a, uint256 b) internal pure returns (uint256 c) {

121:   function bipToRay(uint256 a) internal pure returns (uint256 b) {

138:   function rayMul(uint256 a, uint256 b) internal pure returns (uint256 c) {

155:   function rayDiv(uint256 a, uint256 b) internal pure returns (uint256 c) {

173:   function mulDiv(uint256 x, uint256 y, uint256 d) internal pure returns (uint256 z) {

191:   function mulDivUp(uint256 x, uint256 y, uint256 d) internal pure returns (uint256 z) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L30

```solidity
file:  src/libraries/SafeCastLib.sol

17:   function toUint8(uint256 x) internal pure returns (uint8 y) {

21:   function toUint16(uint256 x) internal pure returns (uint16 y) {

25:   function toUint24(uint256 x) internal pure returns (uint24 y) {

29:   function toUint32(uint256 x) internal pure returns (uint32 y) {

33:   function toUint40(uint256 x) internal pure returns (uint40 y) {

37:   function toUint48(uint256 x) internal pure returns (uint48 y) {

41:   function toUint56(uint256 x) internal pure returns (uint56 y) {

45:   function toUint64(uint256 x) internal pure returns (uint64 y) {

49:   function toUint72(uint256 x) internal pure returns (uint72 y) {

53:   function toUint80(uint256 x) internal pure returns (uint80 y) {

57:   function toUint88(uint256 x) internal pure returns (uint88 y) {

61:   function toUint96(uint256 x) internal pure returns (uint96 y) {

65:   function toUint104(uint256 x) internal pure returns (uint104 y) {

69:   function toUint112(uint256 x) internal pure returns (uint112 y) {

73:   function toUint120(uint256 x) internal pure returns (uint120 y) {

77:   function toUint128(uint256 x) internal pure returns (uint128 y) {

81:   function toUint136(uint256 x) internal pure returns (uint136 y) {

85:   function toUint144(uint256 x) internal pure returns (uint144 y) {

89:   function toUint152(uint256 x) internal pure returns (uint152 y) {

93:   function toUint160(uint256 x) internal pure returns (uint160 y) {

97:   function toUint168(uint256 x) internal pure returns (uint168 y) {

101:   function toUint176(uint256 x) internal pure returns (uint176 y) {

105:   function toUint184(uint256 x) internal pure returns (uint184 y) {

109:   function toUint192(uint256 x) internal pure returns (uint192 y) {

113:   function toUint200(uint256 x) internal pure returns (uint200 y) {

117:   function toUint208(uint256 x) internal pure returns (uint208 y) {

121:   function toUint216(uint256 x) internal pure returns (uint216 y) {

125:   function toUint224(uint256 x) internal pure returns (uint224 y) {

129:   function toUint232(uint256 x) internal pure returns (uint232 y) {

133:   function toUint240(uint256 x) internal pure returns (uint240 y) {

137:   function toUint248(uint256 x) internal pure returns (uint248 y) {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/SafeCastLib.sol#L17



## [G‑7] Counting down in for statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

### Test Code

```solidity
contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;
    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }
    function testGas() public {
        c0.AddNum();
        c1.AddNum();
    }
}
contract Contract0 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=0;i<=9;i++){
            _num = _num +1;
        }
        num = _num;
    }
}
contract Contract1 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=9;i>=0;i--){
            _num = _num +1;
        }
        num = _num;
    }
}
```


```solidity
file: src/WildcatArchController.sol

93   for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }

136  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllerFactories.at(start + i);
    }

179  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllers.at(start + i);
    }

222 for (uint256 i = 0; i < count; i++) {
      arr[i] = _markets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93-L95


```solidity
file:  src/WildcatMarketController.sol

133     for (uint256 i = 0; i < count; i++) {
      arr[i] = _authorizedLenders.at(start + i);
    }

154  for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.add(lender)) {
        emit LenderAuthorized(lender);
      }
    
170    for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.remove(lender)) {
        emit LenderDeauthorized(lender);
      }
    }

183    for (uint256 i; i < markets.length; i++) {
      address market = markets[i];
      if (!_controlledMarkets.contains(market)) {
        revert NotControlledMarket();
      }
      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
    }


212  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controlledMarkets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133-L135


## [G‑8] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity
file: src/WildcatArchController.sol

222 for (uint256 i = 0; i < count; i++) {
      arr[i] = _markets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L222-L224


```solidity
file:  src/WildcatMarketController.sol

133     for (uint256 i = 0; i < count; i++) {
      arr[i] = _authorizedLenders.at(start + i);
    }

212  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controlledMarkets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133-L135


```solidity
file: src/WildcatMarketControllerFactory.sol

146    for (uint256 i = 0; i < count; i++) {
      arr[i] = _deployedControllers.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L146-L148


```solidity
file: src/libraries/FIFOQueue.sol

48    for (uint256 i = 0; i < len; i++) {
      _values[i] = arr.data[startIndex + i];
    }

75  for (uint256 i = 0; i < n; i++) {
      delete arr.data[startIndex + i];
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48-L50

## [G-9] storage variable can be replaced with calldata variable

```solidity
file: src/libraries/FIFOQueue.sol

19   function empty(FIFOQueue storage arr) internal view returns (bool) {

23   function first(FIFOQueue storage arr) internal view returns (uint32) {

30   function at(FIFOQueue storage arr, uint256 index) internal view returns (uint32) {

38   function length(FIFOQueue storage arr) internal view returns (uint128) {
    
42   function values(FIFOQueue storage arr) internal view returns (uint32[] memory _values) {


```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L19

## [G-10] Conditional check can be performed during variable initialization

```solidity
file: src/WildcatMarketControllerFactory.sol

201   bool hasOriginationFee = originationFeeAmount > 0;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L201


## [G-11] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

### cache  WildcatMarket(market) exteranl call outside the loop

```solidity
file: src/WildcatMarketController.sol

183    for (uint256 i; i < markets.length; i++) {
      address market = markets[i];
      if (!_controlledMarkets.contains(market)) {
        revert NotControlledMarket();
      }
      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L183-L189

## [G-12] Duplicated require()/revert() Checks Should Be Refactored To A Modifier Or Function

Saves deployment costs.


```solidity
file: src/WildcatMarketController.sol

 @audit the same condation is also used in line  185

88   if (!_controlledMarkets.contains(market)) 

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L88


```solidity
file: src/market/WildcatMarket.sol

 @audit the same condation is also used in line  121

48   if (state.isClosed) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L48


```solidity
file:  src/market/WildcatMarketBase.sol

 @audit the same condation is also used in line  337

322   if (state.timeDelinquent > delinquencyGracePeriod)

 @audit the same condation is also used in line  410

361   if (state.hasPendingExpiredBatch())

 @audit the same condation is also used in line  472

428   if (availableLiquidity > 0)

 @audit the same condation is also used in line  536

507   if (scaledAmountOwed == 0) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L322


```solidity
file: src/market/WildcatMarketWithdrawals.sol

 @audit the same condation is also used in line  141

49   if (expiry > block.timestamp)

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L49

## [G-13] Combine events to save 2 Glogtopic (375 gas)

The events below are only emitted once in the handleRewards function. We can combine the events into one singular event to save two Glogtopic (375 gas) that would otherwise be paid for the additional two events.

```solidity
file:  src/WildcatArchController.sol
 
67       emit BorrowerAdded(borrower);
  }

  function removeBorrower(address borrower) external onlyOwner {
    if (!_borrowers.remove(borrower)) {
      revert BorrowerDoesNotExist();
    }
74   emit BorrowerRemoved(borrower);

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L67-L74


```solidity
file: src/WildcatArchController.sol

109       }
    emit ControllerFactoryAdded(factory);
  }

  function removeControllerFactory(address factory) external onlyOwner {
    if (!_controllerFactories.remove(factory)) {
      revert ControllerFactoryDoesNotExist();
    }
117    emit ControllerFactoryRemoved(factory);
  }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L109-L118

## [G-14] Failure to check the zero address in the constructor causes the contract to be deployed again

Zero address control is not performed in the constructor in all 2 contracts within the scope of the audit. Bypassing this check could cause the contract to be deployed by mistakenly entering a zero address. In this case, the contract will need to be redeployed. This means extra gas consumption as contract deployment costs are high

```solidity
file:  src/WildcatMarketControllerFactory.sol

72    constructor(
      address _archController,
      address _sentinel,
      MarketParameterConstraints memory constraints
    ) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L72-L76

```solidity
file: src/WildcatSanctionsSentinel.sol

24   constructor(address _archController, address _chainalysisSanctionsList) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L24

## [G-15] bytes constants are more eficient than string constans

If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

```solidity
file: src/market/WildcatMarketBase.sol

24   string public constant version = '1.0';

57   string public name;

60   string public symbol;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L24