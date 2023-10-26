# Table Of Contents

- [Table Of Contents](#table-of-contents)
  - [Validate local variables immediately(Save 1 SSTORE and more in case of a revert- 2200+ Gas)](#validate-local-variables-immediatelysave-1-sstore-and-more-in-case-of-a-revert--2200-gas)
  - [We can optimize the function `setAnnualInterestBips()`: Save 2100 Gas some of the time](#we-can-optimize-the-function-setannualinterestbips-save-2100-gas-some-of-the-time)
  - [Avoid making external calls if we can revert with a cheaper check](#avoid-making-external-calls-if-we-can-revert-with-a-cheaper-check)
  - [Leverage mapping and dot notation for struct assignment](#leverage-mapping-and-dot-notation-for-struct-assignment)
    - [WildcatMarketControllerFactory.sol.setProtocolFeeConfiguration(): Use dot notation](#wildcatmarketcontrollerfactorysolsetprotocolfeeconfiguration-use-dot-notation)
    - [WildcatMarketBase.sol: Use dot notation for the assignment](#wildcatmarketbasesol-use-dot-notation-for-the-assignment)
  - [Cache repeatedly read fields of a struct](#cache-repeatedly-read-fields-of-a-struct)
  - [No need to cache calls if being used once](#no-need-to-cache-calls-if-being-used-once)


## Validate local variables immediately(Save 1 SSTORE and more in case of a revert- 2200+ Gas)

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketWithdrawals.sol#L155-L162
```solidity
File: /src/market/WildcatMarketWithdrawals.sol
155:    uint128 normalizedAmountWithdrawn = newTotalWithdrawn - status.normalizedAmountWithdrawn;

157:    status.normalizedAmountWithdrawn = newTotalWithdrawn;
158:    state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;

160:    if (normalizedAmountWithdrawn == 0) {
161:      revert NullWithdrawalAmount();
162:    }
```
We assign the local variable `normalizedAmountWithdrawn` on Line 155,then on Line 157 we make an SSTORE (`status.normalizedAmountWithdrawn = newTotalWithdrawn;`) 
We however have a check to validate the value of the local variable we assigned initially and we are checking if it's equal to 0 `normalizedAmountWithdrawn == 0`. If it's 0, we would revert the entire operation.
We can therefore avoid making the SSTORE by prioritizing the validity checks as shown below 
```diff
diff --git a/src/market/WildcatMarketWithdrawals.sol b/src/market/WildcatMarketWithdrawals.sol
index 7e6587b..572e9b4 100644
--- a/src/market/WildcatMarketWithdrawals.sol
+++ b/src/market/WildcatMarketWithdrawals.sol
@@ -154,12 +154,13 @@ contract WildcatMarketWithdrawals is WildcatMarketBase {

     uint128 normalizedAmountWithdrawn = newTotalWithdrawn - status.normalizedAmountWithdrawn;

-    status.normalizedAmountWithdrawn = newTotalWithdrawn;
-    state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;
-
     if (normalizedAmountWithdrawn == 0) {
       revert NullWithdrawalAmount();
     }
+    status.normalizedAmountWithdrawn = newTotalWithdrawn;
+    state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;
```

## We can optimize the function `setAnnualInterestBips()`: Save 2100 Gas some of the time

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketConfig.sol#L149-L159
```solidity
File: /src/market/WildcatMarketConfig.sol
149:  function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant {
150:    MarketState memory state = _getUpdatedState();

152:    if (_annualInterestBips > BIP) {
153:      revert InterestRateTooHigh();
154:    }

156:    state.annualInterestBips = _annualInterestBips;
157:    _writeState(state);
158:    emit AnnualInterestBipsUpdated(_annualInterestBips);
159:  }
```
On [Line 150](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketConfig.sol#L150) we are making a function call to `_getUpdatedState()` which has the following [implementation](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L358-L388) largely truncated to just show the first few lines since they are enough to demonstrate our point

```solidity
  function _getUpdatedState() internal returns (MarketState memory state) {
    state = _state;
    // Handle expired withdrawal batch
    if (state.hasPendingExpiredBatch()) {

  }
```
Note that `_state` is a state variable, which means the first line does an SLOAD(2100 Gas) since it's cold
The function then returns a variable in memory which in our main function `setAnnualInterestBips` is being stored in `state` here `MarketState memory state = _getUpdatedState();`

The main function then has a check `_annualInterestBips > BIP` which would revert if it's not met.
In case of this revert, the gas spent on the first line making the function call would be wasted. 
We can reorder this operations to have the check that might lead to a revert come early, which would save us 2100 Gas if we revert

```diff
   function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant {
-    MarketState memory state = _getUpdatedState();
-
+
     if (_annualInterestBips > BIP) {
       revert InterestRateTooHigh();
     }

+    MarketState memory state = _getUpdatedState();
+
     state.annualInterestBips = _annualInterestBips;
     _writeState(state);
     emit AnnualInterestBipsUpdated(_annualInterestBips);
```

## Avoid making external calls if we can revert with a cheaper check

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L65-L70
```solidity
File: /src/WildcatMarketControllerFactory.sol
65:  modifier onlyArchControllerOwner() {
66:    if (msg.sender != archController.owner()) {
67:      revert CallerNotArchControllerOwner();
68:    }
69:    _;
70:  }
```
The above modifer is being used in the following function https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L195-L217
```solidity
195:  function setProtocolFeeConfiguration(
196:    address feeRecipient,
197:    address originationFeeAsset,
198:    uint80 originationFeeAmount,
199:    uint16 protocolFeeBips
200:  ) external onlyArchControllerOwner {
201:    bool hasOriginationFee = originationFeeAmount > 0;
202:    bool nullFeeRecipient = feeRecipient == address(0);
203:    bool nullOriginationFeeAsset = originationFeeAsset == address(0);
204:    if (
205:      (protocolFeeBips > 0 && nullFeeRecipient) ||
206:      (hasOriginationFee && nullFeeRecipient) ||
207:      (hasOriginationFee && nullOriginationFeeAsset)
208:    ) {
209:      revert InvalidProtocolFeeConfiguration();
210:    }
```

Before performing any operation inside this function, we would first execute the code inside the modifier 
The code in the modifier is an external function call(very expensive) to `archController.owner()` and comparing it against the `msg.sender`.  if not equal then revert and don't execute the function code.
In our main function, we also have some checks for the function parameters and we only execute the rest of the function if this checks pass, otherwise we would revert. Since this checks inside the function are way cheaper compared to the check from the modifier, we could save some gas by performing the cheap checks first and only doing the more expensive(external call) after being sure we pass the cheap ones.
To achieve this, we should inline the modifier and then move the modifier check to below the checks in the function as shown below

```diff
diff --git a/src/WildcatMarketControllerFactory.sol b/src/WildcatMarketControllerFactory.sol
index ff4903e..283e645 100644
--- a/src/WildcatMarketControllerFactory.sol
+++ b/src/WildcatMarketControllerFactory.sol
@@ -62,13 +62,6 @@ contract WildcatMarketControllerFactory {

   EnumerableSet.AddressSet internal _deployedControllers;

-  modifier onlyArchControllerOwner() {
-    if (msg.sender != archController.owner()) {
-      revert CallerNotArchControllerOwner();
-    }
-    _;
-  }

@@ -197,7 +190,7 @@ contract WildcatMarketControllerFactory {
     address originationFeeAsset,
     uint80 originationFeeAmount,
     uint16 protocolFeeBips
-  ) external onlyArchControllerOwner {
+  ) external {
     bool hasOriginationFee = originationFeeAmount > 0;
     bool nullFeeRecipient = feeRecipient == address(0);
     bool nullOriginationFeeAsset = originationFeeAsset == address(0);
@@ -208,6 +201,9 @@ contract WildcatMarketControllerFactory {
     ) {
       revert InvalidProtocolFeeConfiguration();
     }
+    if (msg.sender != archController.owner()) {
+      revert CallerNotArchControllerOwner();
+    }
     _protocolFeeConfiguration = ProtocolFeeConfiguration({
       feeRecipient: feeRecipient,
       originationFeeAsset: originationFeeAsset,
```

## Leverage mapping and dot notation for struct assignment

In dot notation, values are directly written to storage variable, When we use the current method in the code the compiler will allocate some memory to store the struct instance first before writing it to storage

### WildcatMarketControllerFactory.sol.setProtocolFeeConfiguration(): Use dot notation

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L195-L217
```solidity
195:  function setProtocolFeeConfiguration(

211:    _protocolFeeConfiguration = ProtocolFeeConfiguration({
212:      feeRecipient: feeRecipient,
213:      originationFeeAsset: originationFeeAsset,
214:      originationFeeAmount: originationFeeAmount,
215:      protocolFeeBips: protocolFeeBips
216:    });
217:  }
```

```diff
diff --git a/src/WildcatMarketControllerFactory.sol b/src/WildcatMarketControllerFactory.sol
index ff4903e..ab6f422 100644
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
+    _protocolFeeConfiguration.feeRecipient = feeRecipient;
+    _protocolFeeConfiguration.originationFeeAsset = originationFeeAsset;
+    _protocolFeeConfiguration.originationFeeAmount = originationFeeAmount;
+    _protocolFeeConfiguration.protocolFeeBips = protocolFeeBips;
+
   }
```

### WildcatMarketBase.sol: Use dot notation for the assignment

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L101-L115
```solidity
File: /src/market/WildcatMarketBase.sol#L101-L115
101:    _state = MarketState({
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
```

## Cache repeatedly read fields of a struct

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L23-L28
```solidity
File: /src/libraries/FIFOQueue.sol
23:  function first(FIFOQueue storage arr) internal view returns (uint32) {
24:    if (arr.startIndex == arr.nextIndex) {
25:      revert FIFOQueueOutOfBounds();
26:    }
27:    return arr.data[arr.startIndex];
28:  }
```
With the refactors suggested, If we do revert on the check `arr.startIndex == arr.nextIndex` we would waste ~6 Gas for the mstore and mload but if we do not revert, then we save ~94 Gas
```diff
diff --git a/src/libraries/FIFOQueue.sol b/src/libraries/FIFOQueue.sol
index e8d0dde..ff4343e 100644
--- a/src/libraries/FIFOQueue.sol
+++ b/src/libraries/FIFOQueue.sol
@@ -21,10 +21,11 @@ library FIFOQueueLib {
   }

   function first(FIFOQueue storage arr) internal view returns (uint32) {
-    if (arr.startIndex == arr.nextIndex) {
+    uint128 _startIndex = arr.startIndex;
+    if (_startIndex == arr.nextIndex) {
       revert FIFOQueueOutOfBounds();
     }
-    return arr.data[arr.startIndex];
+    return arr.data[_startIndex];
   }
```

## No need to cache calls if being used once

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarket.sol#L86-L91
```solidity
File: /src/market/WildcatMarket.sol
86:  function deposit(uint256 amount) external virtual {
87:    uint256 actualAmount = depositUpTo(amount);
88:    if (amount != actualAmount) {
89:      revert MaxSupplyExceeded();
90:    }
91:  }
```

```diff
   function deposit(uint256 amount) external virtual {
-    uint256 actualAmount = depositUpTo(amount);
-    if (amount != actualAmount) {
+    if (amount != depositUpTo(amount)) {
       revert MaxSupplyExceeded();
     }
   }
```
