# WILDCAT PROTOCOL GAS OPTIMIZATIONS


## INTRODUCTION
The majority of optimizations underwent benchmarking using the protocol's tests, specifically employing the following configuration: `solc version 0.8.19`, optimizer enabled, and `200` runs. For optimizations that were not subjected to benchmarking, their rationale is elucidated through EVM gas expenses and opcodes.

Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. Additionally, certain code excerpts may be abbreviated for brevity, and comments within the code snippets may include @audit tags to facilitate the explanation of issues.


## TABLE OF FINDINGS
| Number |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| G-01| Pre-calculate equations or calculations which contain only constant values in constructor | 1 | 71 |
| G-02| Assigning state variables directly with named struct constructors wastes gas | 2 | - |
| G-03| Refactor event to avoid emitting data that is already present in transaction data | 1 | 375  |
| G-04| Update to state var should only happen once in a function | 2 | 2900 |
| G-05| Declaring Unnecessary variables | 7 | 21 |
| G-06| Avoid updating storage when the value hasn't changed  | 1 | 2000 |
| G-07| Same cast is done multiple times | 4 | - |




## [G-01]  Pre-calculate equations or calculations which contain only constant values in constructor
For equations or calcuations that only involve constants or immutable values they could be calculated in the contract's constructor and saved to an immutable variable. Using the immutable variable would be cheaper than performing the calculation every time the function is called.

### 1 Instance
1. #### Perform the `MathUtils.bipToRay(delinquencyFeeBips)` in the constructor.
- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L323
```solidity
file: src/market/WildcatMarketBase.sol

318:  function effectiveBorrowerAPR() external view returns (uint256) {
319:    MarketState memory state = currentState();
320:    // apr + (apr * protocolFee)
321:    uint256 apr = MathUtils.bipToRay(state.annualInterestBips).bipMul(BIP + protocolFeeBips);
322:    if (state.timeDelinquent > delinquencyGracePeriod) {
323:      apr += MathUtils.bipToRay(delinquencyFeeBips);    //@audit calculation involving only constant
324:    }
325:    return apr;
326:  }
```
Since delinquencyFeeBips is a constant value, the value of MathUtils.bipToRay(delinquencyFeeBips) can be calculated before compiling the contract. Calling `MathUtils.bipToRay()` on a known, constant value is a waste of gas rather the calcualtion should be done in the constructor then saved to an immutable variable so that in the `effectiveBorrowerAPR()` the calculations would be replaced with cheaper stack read. The code could be refactored as shown in the diff below:
```diff
diff --git a/src/market/WildcatMarketBase.sol b/src/market/WildcatMarketBase.sol
index b35d65f..c302a25 100644
--- a/src/market/WildcatMarketBase.sol
+++ b/src/market/WildcatMarketBase.sol
@@ -53,6 +53,8 @@ contract WildcatMarketBase is ReentrancyGuard, IMarketEventsAndErrors {
   /// @dev Token decimals (same as underlying asset).
   uint8 public immutable decimals;

+  uint public immutable delinquencyFeeBipsToRay;
+
   /// @dev Token name (prefixed name of underlying asset).
   string public name;

@@ -122,6 +124,7 @@ contract WildcatMarketBase is ReentrancyGuard, IMarketEventsAndErrors {
     delinquencyFeeBips = parameters.delinquencyFeeBips;
     delinquencyGracePeriod = parameters.delinquencyGracePeriod;
     withdrawalBatchDuration = parameters.withdrawalBatchDuration;
+    delinquencyFeeBipsToRay = MathUtils.bipToRay(delinquencyFeeBips);
   }

   // ===================================================================== //
@@ -320,7 +323,7 @@ contract WildcatMarketBase is ReentrancyGuard, IMarketEventsAndErrors {
     // apr + (apr * protocolFee)
     uint256 apr = MathUtils.bipToRay(state.annualInterestBips).bipMul(BIP + protocolFeeBips);
     if (state.timeDelinquent > delinquencyGracePeriod) {
-      apr += MathUtils.bipToRay(delinquencyFeeBips);
+      apr += delinquencyFeeBipsToRay;
     }
     return apr;
   }
```
#### Gas saving for `effectiveBorrowerAPR()` function obtained via protocol test: Avg 71 gas
```
BEFORE
test/market/WildcatMarketBase.t.sol:WildcatMarketBaseTest
[PASS] test_effectiveBorrowerAPR() (gas: 630964)
```
```
AFTER
test/market/WildcatMarketBase.t.sol:WildcatMarketBaseTest
[PASS] test_effectiveBorrowerAPR() (gas: 630893)
```




## [G-02] Assigning state variables directly with named struct constructors wastes gas
Using named arguments for struct means that the compiler needs to organize the fields in memory before doing the assignment, which wastes gas. Setting each field directly in storage (use dot-notation) is more gas efficient.

### Proof of concept
```solidity
struct Addresses {
    address address1;
    address address2;
    address address3;
}


contract UsingNamedStruct {

    Addresses public myAddresses;

    function setMyAddresses() public {
        myAddresses = Addresses({
            address1: address(1),
            address2: address(1),
            address3: address(1)
        });
    }
}
```
```
test/UsingNamedStruct.t.sol:UsingNamedStructTest
[PASS] test_setMyAddresses() (gas: 71812)
```

```solidity
struct Addresses {
    address address1;
    address address2;
    address address3;
}

contract SettingStorageFields {

    Addresses public myAddresses;


    function setMyAddresses() public {
        myAddresses.address1 = address(1);
        myAddresses.address2 = address(1);
        myAddresses.address3 = address(1);
    }
    
}
```
```
 test/SettingStorageField.t.sol:SettingStorageFieldsTest
[PASS] test_setMyAddresses() (gas: 71722)
```

### 2 Instances
1. #### Set the fields of `_protocolFeeConfiguration` directly rather than using named struct.
- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L211
```solidity
file: src/WildcatMarketControllerFactory.sol

195:  function setProtocolFeeConfiguration(
196:    address feeRecipient,
197:    address originationFeeAsset,
198:    uint80 originationFeeAmount,
199:    uint16 protocolFeeBips
200:  ) external onlyArchControllerOwner {
201:    bool hasOriginationFee = originationFeeAmount > 0;
.
.
.    
211:    _protocolFeeConfiguration = ProtocolFeeConfiguration({  //@audit set each field in storage directly
212:      feeRecipient: feeRecipient,
213:      originationFeeAsset: originationFeeAsset,
214:      originationFeeAmount: originationFeeAmount,
215:      protocolFeeBips: protocolFeeBips
216:    });
217:  }
```
We can make the `setProtocolFeeConfiguration()` function more gas efficient by setting each field (attribute) of the `_protocolFeeConfiguration` directly rather than using named arguments because in using named arguments the compiler would need to organize the fields in memory before doing the assignment. The code can be refactored as shown in the diff below:
```diff
diff --git a/src/WildcatMarketControllerFactory.sol b/src/WildcatMarketControllerFactory.sol
index ff4903e..1c14c55 100644
--- a/src/WildcatMarketControllerFactory.sol
+++ b/src/WildcatMarketControllerFactory.sol
@@ -208,12 +208,11 @@ contract WildcatMarketControllerFactory {
     ) {
       revert InvalidProtocolFeeConfiguration();
     }
-    _protocolFeeConfiguration = ProtocolFeeConfiguration({
-      feeRecipient: feeRecipient,
-      originationFeeAsset: originationFeeAsset,
-      originationFeeAmount: originationFeeAmount,
-      protocolFeeBips: protocolFeeBips
-    });
+
+    _protocolFeeConfiguration.feeRecipient = feeRecipient;
+    _protocolFeeConfiguration.originationFeeAsset = originationFeeAsset;
+    _protocolFeeConfiguration.originationFeeAmount = originationFeeAmount;
+    _protocolFeeConfiguration.protocolFeeBips = protocolFeeBips;
   }
```

2. #### Set the fields of `_state` directly rather than using named struct.
- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L101
```solidity
76:   constructor() {
.
.
.
101:    _state = MarketState({  //@audit set each field in storage directly
102:      isClosed: false,
103:      maxTotalSupply: parameters.maxTotalSupply,
104:      accruedProtocolFees: 0,
105:      normalizedUnclaimedWithdrawals: 0,
106:      scaledTotalSupply: 0,
107:      scaledPendingWithdrawals: 0,
108:      pendingWithdrawalExpiry: 0,
109:      isDelinquent: false,
110:      timeDelinquent: 0,
111:      annualInterestBips: parameters.annualInterestBips,
112:      reserveRatioBips: parameters.reserveRatioBips,
113:      scaleFactor: uint112(RAY),
114:      lastInterestAccruedTimestamp: uint32(block.timestamp)
115:    });
.
.
.
125:  }
```

```diff
diff --git a/src/market/WildcatMarketBase.sol b/src/market/WildcatMarketBase.sol
index b35d65f..df7daf6 100644
--- a/src/market/WildcatMarketBase.sol
+++ b/src/market/WildcatMarketBase.sol
@@ -98,21 +98,21 @@ contract WildcatMarketBase is ReentrancyGuard, IMarketEventsAndErrors {
     symbol = string.concat(parameters.symbolPrefix, querySymbol(parameters.asset));
     decimals = IERC20Metadata(parameters.asset).decimals();

-    _state = MarketState({
-      isClosed: false,
-      maxTotalSupply: parameters.maxTotalSupply,
-      accruedProtocolFees: 0,
-      normalizedUnclaimedWithdrawals: 0,
-      scaledTotalSupply: 0,
-      scaledPendingWithdrawals: 0,
-      pendingWithdrawalExpiry: 0,
-      isDelinquent: false,
-      timeDelinquent: 0,
-      annualInterestBips: parameters.annualInterestBips,
-      reserveRatioBips: parameters.reserveRatioBips,
-      scaleFactor: uint112(RAY),
-      lastInterestAccruedTimestamp: uint32(block.timestamp)
-    });
+
+    _state.isClosed = false;
+    _state.maxTotalSupply = parameters.maxTotalSupply;
+    _state.accruedProtocolFees = 0;
+    _state.normalizedUnclaimedWithdrawals = 0;
+    _state.scaledTotalSupply = 0;
+    _state.scaledPendingWithdrawals = 0;
+    _state.pendingWithdrawalExpiry = 0;
+    _state.isDelinquent = false
+    _state.timeDelinquent = 0;
+    _state.annualInterestBips = parameters.annualInterestBips;
+    _state.reserveRatioBips = parameters.reserveRatioBips;
+    _state.scaleFactor = uint112(RAY);
+    _state.lastInterestAccruedTimestamp = uint32(block.timestamp);
+
```




## [G-03] Refactor event to avoid emitting data that is already present in transaction data

### 1 Instance
1. #### Refactor `MarketClosed()` to avoid emitting `block.timestamp` which already available in the transaction data
- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L160
```solidity
file: src/market/WildcatMarket.sol

142:  function closeMarket() external onlyController nonReentrant {
143:    MarketState memory state = _getUpdatedState();
144:    state.annualInterestBips = 0;
145:    state.isClosed = true;
146:    state.reserveRatioBips = 0;
147:    if (_withdrawalData.unpaidBatches.length() > 0) {
148:      revert CloseMarketWithUnpaidWithdrawals();
149:    }
150:    uint256 currentlyHeld = totalAssets();
151:    uint256 totalDebts = state.totalDebts();
152:    if (currentlyHeld < totalDebts) {
153:      // Transfer remaining debts from borrower
154:      asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);
155:    } else if (currentlyHeld > totalDebts) {
156:      // Transfer excess assets to borrower
157:      asset.safeTransfer(borrower, currentlyHeld - totalDebts);
158:    }
159:    _writeState(state);
160:    emit MarketClosed(block.timestamp); //@audit superfluous emit
161:  }
```
In the `closeMarket()` function ablove `block.timestamp` does not have to be emitted since the timestamp is already present in the transaction data.
```diff
diff --git a/src/market/WildcatMarket.sol b/src/market/WildcatMarket.sol
index 492b413..e3e1129 100644
--- a/src/market/WildcatMarket.sol
+++ b/src/market/WildcatMarket.sol
@@ -157,6 +157,6 @@ contract WildcatMarket is
       asset.safeTransfer(borrower, currentlyHeld - totalDebts);
     }
     _writeState(state);
-    emit MarketClosed(block.timestamp);
+    emit MarketClosed();
   }
```
```
Estimated gas saved: 375 gas units
```



## [G-04]  Update to state var should only happen once in a function

### 1 Instance
1. #### ` _tmpMarketBorrowerParameter` state variable was written to twice in the `deployController` function
- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L286&&#L298
```solidity
file: src/WildcatMarketControllerFactory.sol

282:  function deployController() public returns (address controller) {
283:    if (!archController.isRegisteredBorrower(msg.sender)) {
284:      revert NotRegisteredBorrower();
285:    }
286:    _tmpMarketBorrowerParameter = msg.sender;   //@audit first write to state var
287:    // Salt is borrower address
288:    bytes32 salt = bytes32(uint256(uint160(msg.sender)));
289:    controller = LibStoredInitCode.calculateCreate2Address(
290:      ownCreate2Prefix,
291:      salt,
292:      controllerInitCodeHash
293:    );
294:    if (controller.codehash != bytes32(0)) {
295:      revert ControllerAlreadyDeployed();
296:    }
297:    LibStoredInitCode.create2WithStoredInitCode(controllerInitCodeStorage, salt);
298:    _tmpMarketBorrowerParameter = address(1);   //@audit second write to state var
299:    archController.registerController(controller);
300:    _deployedControllers.add(controller);
301:  }
```
In the `deployController()` function above the state variable `_tmpMarketBorrowerParameter` as assigned different values twice in the function. The assignment of `msg.sender` to the variable is redundant as subsequently in the function `msg.sender` was used directly and not the `_tmpMarketBorrowerParameter`. Removing this redundant statement helps us avoid a Gsreset `2900 gas`. The code could be refactored as shown in the diff below: 
```diff
diff --git a/src/WildcatMarketControllerFactory.sol b/src/WildcatMarketControllerFactory.sol
index ff4903e..2e4fa77 100644
--- a/src/WildcatMarketControllerFactory.sol
+++ b/src/WildcatMarketControllerFactory.sol
@@ -283,7 +283,7 @@ contract WildcatMarketControllerFactory {
     if (!archController.isRegisteredBorrower(msg.sender)) {
       revert NotRegisteredBorrower();
     }
-    _tmpMarketBorrowerParameter = msg.sender;
+
     // Salt is borrower address
     bytes32 salt = bytes32(uint256(uint160(msg.sender)));
     controller = LibStoredInitCode.calculateCreate2Address(
```
```
Estimated gas saved: 2900
```



## [G-05] Declaring Unnecessary variables
Some varibles were defined even though they are used once. Not defining variables can reduce gas cost and contract size.
### 7 Instances
- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L208
```solidity
file: src/WildcatMarketController.sol

204:  function getControlledMarkets(
205:    uint256 start,
206:    uint256 end
207:  ) external view returns (address[] memory arr) {
208:    uint256 len = _controlledMarkets.length();  //@audit len declaration redundant
209:    end = MathUtils.min(end, len);
210:    uint256 count = end - start;
211:    arr = new address[](count);
212:    for (uint256 i = 0; i < count; i++) {
213:      arr[i] = _controlledMarkets.at(start + i);
214:    }
215:  }
```
We can save `3` gas units in the `getControlledMarkets()` function if rather than declaring variable `len` we used `_controlledMarkets.length()` directly since `len` is used only once in the function.
```diff
diff --git a/src/WildcatMarketController.sol b/src/WildcatMarketController.sol
index 55f62f6..efef50b 100644
--- a/src/WildcatMarketController.sol
+++ b/src/WildcatMarketController.sol
@@ -205,8 +205,7 @@ contract WildcatMarketController is IWildcatMarketControllerEventsAndErrors {
     uint256 start,
     uint256 end
   ) external view returns (address[] memory arr) {
-    uint256 len = _controlledMarkets.length();
-    end = MathUtils.min(end, len);
+    end = MathUtils.min(end, _controlledMarkets.length());
     uint256 count = end - start;
     arr = new address[](count);
     for (uint256 i = 0; i < count; i++) 
```

- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L89
```solidity
file: src/WildcatArchController.sol

85:  function getRegisteredBorrowers(
86:    uint256 start,
87:    uint256 end
88:  ) external view returns (address[] memory arr) {
89:    uint256 len = _borrowers.length();   //@audit len declaration redundant
90:    end = MathUtils.min(end, len);
91:    uint256 count = end - start;
92:    arr = new address[](count);
93:    for (uint256 i = 0; i < count; i++) {
94:      arr[i] = _borrowers.at(start + i);
95:    }
96:  }
```
We can save `3` gas units in the `getRegisteredBorrowers()` function if rather than declaring variable `len` we used `_borrowers.length()` directly since `len` is used only once in the function.
```diff
diff --git a/src/WildcatArchController.sol b/src/WildcatArchController.sol
index 7b0a173..bbdafd9 100644
--- a/src/WildcatArchController.sol
+++ b/src/WildcatArchController.sol
@@ -86,8 +86,7 @@ contract WildcatArchController is Ownable {
     uint256 start,
     uint256 end
   ) external view returns (address[] memory arr) {
-    uint256 len = _borrowers.length();
-    end = MathUtils.min(end, len);
+    end = MathUtils.min(end, _borrowers.length());
     uint256 count = end - start;
     arr = new address[](count);
     for (uint256 i = 0; i < count; i++) 
```

- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L132
```solidity
file: src/WildcatArchController.sol

128:  function getRegisteredControllerFactories(
129:    uint256 start,
130:    uint256 end
131:  ) external view returns (address[] memory arr) {
132:    uint256 len = _controllerFactories.length();   //@audit len declaration redundant
133:    end = MathUtils.min(end, len);
134:    uint256 count = end - start;
135:    arr = new address[](count);
136:    for (uint256 i = 0; i < count; i++) {
137:      arr[i] = _controllerFactories.at(start + i);
138:    }
139:  }
```
We can save `3` gas units in the `getRegisteredControllerFactories()` function if rather than declaring variable `len` we used `_controllerFactories.length()` directly since `len` is used only once in the function.
```diff
diff --git a/src/WildcatArchController.sol b/src/WildcatArchController.sol
index 7b0a173..f810286 100644
--- a/src/WildcatArchController.sol
+++ b/src/WildcatArchController.sol
@@ -129,8 +129,8 @@ contract WildcatArchController is Ownable {
     uint256 start,
     uint256 end
   ) external view returns (address[] memory arr) {
-    uint256 len = _controllerFactories.length();
-    end = MathUtils.min(end, len);
+
+    end = MathUtils.min(end, _controllerFactories.length());
     uint256 count = end - start;
     arr = new address[](count);
     for (uint256 i = 0; i < count; i++) {
```

- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L175
```solidity
file: src/WildcatArchController.sol

171:  function getRegisteredControllers(
172:    uint256 start,
173:    uint256 end
174:  ) external view returns (address[] memory arr) {
175:    uint256 len = _controllers.length();    //@audit len declaration redundant
176:    end = MathUtils.min(end, len);
177:    uint256 count = end - start;
178:    arr = new address[](count);
179:    for (uint256 i = 0; i < count; i++) {
180:      arr[i] = _controllers.at(start + i);
181:    }
182:  }
```
We can save `3` gas units in the `getRegisteredControllers()` function if rather than declaring variable `len` we used `_controllers.length()` directly since `len` is used only once in the function.
```diff
diff --git a/src/WildcatArchController.sol b/src/WildcatArchController.sol
index 7b0a173..79947bd 100644
--- a/src/WildcatArchController.sol
+++ b/src/WildcatArchController.sol
@@ -172,8 +172,8 @@ contract WildcatArchController is Ownable {
     uint256 start,
     uint256 end
   ) external view returns (address[] memory arr) {
-    uint256 len = _controllers.length();
-    end = MathUtils.min(end, len);
+
+    end = MathUtils.min(end, _controllers.length());
     uint256 count = end - start;
     arr = new address[](count);
     for (uint256 i = 0; i < count; i++) {
```

- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L218
```solidity
file: src/WildcatArchController.sol

214:  function getRegisteredMarkets(
215:    uint256 start,
216:    uint256 end
217:  ) external view returns (address[] memory arr) {
218:    uint256 len = _markets.length();    //@audit len declaration redundant
219:    end = MathUtils.min(end, len);
220:    uint256 count = end - start;
221:    arr = new address[](count);
222:    for (uint256 i = 0; i < count; i++) {
223:      arr[i] = _markets.at(start + i);
224:    }
225:  }
```
We can save `3` gas units in the `getRegisteredMarkets()` function if rather than declaring variable `len` we used `_markets.length()` directly since `len` is used only once in the function.
```diff
diff --git a/src/WildcatArchController.sol b/src/WildcatArchController.sol
index 7b0a173..0a5529b 100644
--- a/src/WildcatArchController.sol
+++ b/src/WildcatArchController.sol
@@ -215,8 +215,8 @@ contract WildcatArchController is Ownable {
     uint256 start,
     uint256 end
   ) external view returns (address[] memory arr) {
-    uint256 len = _markets.length();
-    end = MathUtils.min(end, len);
+
+    end = MathUtils.min(end, _markets.length());
     uint256 count = end - start;
     arr = new address[](count);
     for (uint256 i = 0; i < count; i++) {
```

- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L22
```solidity
file: src/market/WildcatMarketConfig.sol

21:  function maximumDeposit() external view returns (uint256) {
22:    MarketState memory state = currentState();    //@audit state declaration redundant
23:    return state.maximumDeposit();
24:  }
```
We can save `3` gas units in the `maximumDeposit()` function if rather than declaring variable `state` we used `currentState()` directly since `state` is used only once in the function.
```diff
diff --git a/src/market/WildcatMarketConfig.sol b/src/market/WildcatMarketConfig.sol
index 539a435..25e9075 100644
--- a/src/market/WildcatMarketConfig.sol
+++ b/src/market/WildcatMarketConfig.sol
@@ -19,8 +19,7 @@ contract WildcatMarketConfig is WildcatMarketBase {
    *      currently be deposited to the market.
    */
   function maximumDeposit() external view returns (uint256) {
-    MarketState memory state = currentState();
-    return state.maximumDeposit();
+    return  currentState().maximumDeposit();
   }
```

- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L35
```solidity
file: src/libraries/MathUtils.sol

30:  function calculateLinearInterestFromBips(
31:    uint256 rateBip,
32:    uint256 timeDelta
33:  ) internal pure returns (uint256 result) {
34:    uint256 rate = rateBip.bipToRay();
35:    uint256 accumulatedInterestRay = rate * timeDelta;    //@audit accumulatedInterestRay declaration redundant
36:    unchecked {
37:      return accumulatedInterestRay / SECONDS_IN_365_DAYS;
38:    }
39:  }
```

```diff
diff --git a/src/libraries/MathUtils.sol b/src/libraries/MathUtils.sol
index 5e6c9ba..f9eb6c4 100644
--- a/src/libraries/MathUtils.sol
+++ b/src/libraries/MathUtils.sol
@@ -32,9 +32,9 @@ library MathUtils {
     uint256 timeDelta
   ) internal pure returns (uint256 result) {
     uint256 rate = rateBip.bipToRay();
-    uint256 accumulatedInterestRay = rate * timeDelta;
+    uint256 accumulatedInterestRay = ;
     unchecked {
-      return accumulatedInterestRay / SECONDS_IN_365_DAYS;
+      result = (rate * timeDelta) / SECONDS_IN_365_DAYS;
     }
   }
```



## [G-06] Avoid updating storage when the value hasn't changed 
If the old value is equal to the new value, not re-storing the value will avoid a Gsreset (2900 gas), potentially at the expense of a Gcoldsload (2100 gas) or a Gwarmaccess (100 gas)

### 2 Instances
- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L211
- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L112




## [G-07] Same cast is done multiple times
It's cheaper to do it once, and store the result to a variable. The examples below are the second+ instance of a given cast of the variable

### 4 Instances
- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L256&&#L259
- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L324&&#L345
- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L202&&#L203
- https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L31






## CONCLUSION
As you embark on the journey of incorporating the recommended changes, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.