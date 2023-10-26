
## Gas Optimizations
| Number | Issue | Instances |
|-|:-|:-:|
| [[G-01](#g-01-dont-cache-value-if-it-is-only-used-once)] | Don’t `cache` value if it is only used once | 6 |
| [[G-02](#g-02-use--instead-of--for-uint-type)] | Use `!=` instead of `>` for `uint` type | 7 |
| [[G-03](#g-03-state-variables-can-be-cached-instead-of-re-reading-them-from-storage)] | `State` variables can be cached instead of re-reading them from storage | 3 |
| [[G-04](#g-04-x--y-costs-more-gas-than-x--x--y)] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` | 3 |
| [[G-05](#g-05-instead-of-counting-down-in-for-statements-count-up)] | Instead of counting down in for statements, count up | 3 |
| [[G-06](#g-06-remove-unused-errorevent-to-save-gas)] | Remove unused `error`/`event` to save gas | 2 |
| [[G-07](#g-07-avoid-zero-transfer-to-save-gas)] | Avoid `zero transfer` to save gas | 2 |
| [[G-08](#g-08-cache-external-call-function-to-save-gas)] | Cache `External` call function to save gas | 3 |
| [[G-09](#g-09-use-constant-for-typeuint256max-to-save-gas)] | Use constant for `type(uint256).max` to save gas | 2 |
| [[G-10](#g-10-use-if-instead-of)] | Use `if` instead of `&&` | 1 |
| [[G-11](#g-11-remove-unused-import-variables)] | Remove unused `import` variables | 3 |


## [G-01] Don’t `cache` value if it is only used once

If a value is only intended to be used once then it should not be cached. Caching the value will result in unnecessary stack manipulation.

_6 Interfaces in 4 Files_

```solidity
File : src/libraries/FeeMath.sol

19:    function calculateLinearInterestFromBips(
20:       uint256 rateBip,
21:       uint256 timeDelta
22:     ) internal pure returns (uint256 result) {
23:       uint256 rate = rateBip.bipToRay();
24:       uint256 accumulatedInterestRay = rate * timeDelta;//@audit don't cache rate * timeDelta
25:       unchecked {
26:       return accumulatedInterestRay / SECONDS_IN_365_DAYS;
27:       }
28:    }

```
[19-28](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L19C2-L28C4)

```solidity
File : src/libraries/MathUtils.sol

30:   function calculateLinearInterestFromBips(
31:     uint256 rateBip,
32:     uint256 timeDelta
33:   ) internal pure returns (uint256 result) {
34:     uint256 rate = rateBip.bipToRay();
35:     uint256 accumulatedInterestRay = rate * timeDelta; //@audit don't cache rate * timeDelta
36:     unchecked {
37:       return accumulatedInterestRay / SECONDS_IN_365_DAYS;
38:     }
39:   }

```
[30-39](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L30C1-L39C4)

```solidity
File : src/WildcatArchController.sol

  function getRegisteredBorrowers(
    uint256 start,
    uint256 end
  ) external view returns (address[] memory arr) {
    uint256 len = _borrowers.length();//@audit don't cache _borrowers.length()
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }
  }

```
[85-97](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L85C1-L97C1)

```solidity
File : src/WildcatArchController.sol

   function getRegisteredControllerFactories(
      uint256 start,
      uint256 end
    ) external view returns (address[] memory arr) {
      uint256 len = _controllerFactories.length(); //@audit don't cache _controllerFactories.length()
      end = MathUtils.min(end, len);
      uint256 count = end - start;
      arr = new address[](count);
      for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllerFactories.at(start + i);
      }
    }

```
[128-139](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L128C2-L139C4)

```solidity
File : src/WildcatArchController.sol

  function getRegisteredControllers(
    uint256 start,
    uint256 end
  ) external view returns (address[] memory arr) {
    uint256 len = _controllers.length();//@audit don't cache _controllers.length()
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllers.at(start + i);
    }
  }

```
[171-182](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L171C1-L182C4)

```solidity
File : src/WildcatArchController.sol

 function getRegisteredMarkets(
    uint256 start,
    uint256 end
  ) external view returns (address[] memory arr) {
    uint256 len = _markets.length();//@audit don't cache _markets.length()
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
      arr[i] = _markets.at(start + i);
    }
  }

```
[214-225](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L214C2-L225C4)


## [G-02] Use `!=` instead of `>` for `uint` type

_7 Interfaces in 2 Files_

```solidity
File : src/libraries/FeeMath.sol

67:    if (timeWithPenalty > 0) {

```
[67](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L67C5-L67C31)

```solidity
File : src/libraries/FeeMath.sol

155:   if (protocolFeeBips > 0) {

```
[155](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L155C5-L155C31)

```solidity
File : src/libraries/FeeMath.sol

159:   if (delinquencyFeeBips > 0) {

```
[159](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L159C5-L159C34)

```solidity
File : src/market/WildcatMarketBase.sol

79:  if ((parameters.protocolFeeBips > 0).and(parameters.feeRecipient == address(0))) {

```
[79](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L79C4-L79C87)

```solidity
File : src/market/WildcatMarketBase.sol

170:  if (scaledBalance > 0) {

```
[170](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L170C6-L170C31)

```solidity
File : src/market/WildcatMarketBase.sol

428:  if (availableLiquidity > 0) {

```
[428](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L428C7-L428C36)

```solidity
File : src/market/WildcatMarketBase.sol

472:  if (availableLiquidity > 0) {

```
[472](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L472C5-L472C34)


## [G-03] `State` variables can be cached instead of re-reading them from storage

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

**Note: These instance missed by bot-report**

_3 Interfaces in  File_

```solidity
File : src/libraries/FIFOQueue.sol

23:   function first(FIFOQueue storage arr) internal view returns (uint32) {
24:       if (arr.startIndex == arr.nextIndex) {
25:          revert FIFOQueueOutOfBounds();
26:       }
27:       return arr.data[arr.startIndex];
28:   }

```
[23-28](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L23C3-L28C4)

```diff
File : src/libraries/FIFOQueue.sol

    function first(FIFOQueue storage arr) internal view returns (uint32) {
+      uint128 arr_startIndex = arr.startIndex;        
-       if (arr.startIndex == arr.nextIndex) {
+       if (arr_startIndex == arr.nextIndex) {
          revert FIFOQueueOutOfBounds();
        }
-        return arr.data[arr.startIndex];
+        return arr.data[arr_startIndex];
    }

```

## [G-04] `<x> += <y>` costs more gas than `<x> = <x> + <y>` 

_3 Interfaces in 2 Files_

```solidity
File : src/libraries/FIFOQueue.sol

30:   function at(FIFOQueue storage arr, uint256 index) internal view returns (uint32) {
31:       index += arr.startIndex;
32:       if (index >= arr.nextIndex) {
33:          revert FIFOQueueOutOfBounds();
34:       } 
35:        return arr.data[index];
36:    }

```
[30-36](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L30C3-L36C4)

```diff
File : src/libraries/FIFOQueue.sol

    function at(FIFOQueue storage arr, uint256 index) internal view returns (uint32) {
-        index += arr.startIndex;
+        index = index + arr.startIndex;
        if (index >= arr.nextIndex) {
           revert FIFOQueueOutOfBounds();
        }
        return arr.data[index];
    }

```

```solidity
File : src/market/WildcatMarketBase.sol

322:   if (state.timeDelinquent > delinquencyGracePeriod) {
323:       apr += MathUtils.bipToRay(delinquencyFeeBips);
324:    }

```
[322-324](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L322C5-L324C6)

```diff
File : src/market/WildcatMarketBase.sol

    if (state.timeDelinquent > delinquencyGracePeriod) {
-        apr += MathUtils.bipToRay(delinquencyFeeBips);
+        apr = apr + MathUtils.bipToRay(delinquencyFeeBips);
    }

```

```solidity
File : src/market/WildcatMarketBase.sol

337:   if (state.timeDelinquent > delinquencyGracePeriod) {
338:      apr += delinquencyFeeBips;
339:    }

```
[337-339](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L337C2-L339C6)

```diff
File : src/market/WildcatMarketBase.sol

   if (state.timeDelinquent > delinquencyGracePeriod) {
-      apr += delinquencyFeeBips;
+      apr = apr + delinquencyFeeBips;
    }

```


## [G-05] Instead of counting down in for statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

_3 Interfaces in 1 File_

```solidity
File : src/libraries/FIFOQueue.sol

    for (uint256 i = 0; i < len; i++) {
         _values[i] = arr.data[startIndex + i];
    }

```
[48-50](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48C5-L50C6)

```diff
File : src/libraries/FIFOQueue.sol

-    for (uint256 i = 0; i < len; i++) {
-        _values[i] = arr.data[startIndex + i];
-     }
+    for (uint256 i = len - 1; i >= 0; --i) {
+        _values[i] = arr.data[startIndex + i];
+     }

```

```solidity
File : src/libraries/FIFOQueue.sol

  for (uint256 i = 0; i < n; i++) {
      delete arr.data[startIndex + i];
    }
    arr.startIndex = startIndex + n;
  }

```
[75-79](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L75C3-L79C4)

```diff
File : src/libraries/FIFOQueue.sol

-    for (uint256 i = 0; i < n; i++) {
+    for (uint256 i = n - 1; i >= 0; --i) {
       delete arr.data[startIndex + i];
    }
      arr.startIndex = startIndex + n;
  }

```

## [G-06] Remove unused `error`/`event` to save gas

_2 Interfaces in 2 Files_

```solidity
File : src/libraries/StringQuery.sol

17:   error InvalidCompactString();

```
[17](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/StringQuery.sol#L17C1-L17C30)

```solidity
File : src/WildcatMarketControllerFactory.sol

17:    event UpdateProtocolFeeConfiguration(
18:     address feeRecipient,
19:     uint16 protocolFeeBips,
20:     address originationFeeAsset,
21:     uint256 originationFeeAmount
22:    );

```
[17-22](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L17C1-L22C1)

## [G-07] Avoid `zero transfer` to save gas

**Note: These instances missed by bot-report**

_2 Interfaces in 1 File_

```solidity
File : src/market/WildcatMarket.sol

129:   asset.safeTransfer(msg.sender, amount);

```
[129](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L129C5-L129C44)

```solidity
File : src/market/WildcatMarket.sol

154:   asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);

```
[154](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L154C6-L154C83)

## [G-08] Cache `External` call function to save gas

_3 Interfaces in 1 File_

```solidity
File : src/market/WildcatMarketBase.sol

   );
        emit Transfer(accountAddress, escrow, state.normalizeAmount(scaledBalance));
        _accounts[escrow].scaledBalance += scaledBalance;
        emit SanctionedAccountAssetsSentToEscrow(
          accountAddress,
          escrow,
          state.normalizeAmount(scaledBalance)
        );
      }
      _accounts[accountAddress] = account;
    }

```
[176-186](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L176C6-L186C6)

```diff
File : src/market/WildcatMarketBase.sol

   );
+        uint104 state_normalizeAmount = state.normalizeAmount(scaledBalance);
-        emit Transfer(accountAddress, escrow, state.normalizeAmount(scaledBalance));
+        emit Transfer(accountAddress, escrow, state_normalizeAmount);
         _accounts[escrow].scaledBalance += scaledBalance;
         emit SanctionedAccountAssetsSentToEscrow(
          accountAddress,
          escrow,
-         state.normalizeAmount(scaledBalance)
+         state_normalizeAmount
        );
      }
      _accounts[accountAddress] = account;
    }

```

## [G-09] Use constant for `type(uint256).max` to save gas

_2 Interfaces in 1 File_

```solidity
File : src/market/WildcatMarketToken.sol

    if (allowed != type(uint256).max) {
        uint256 newAllowance = allowed - amount;
        _approve(from, msg.sender, newAllowance);
    }

```
[49-52](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L49C4-L52C6)

```diff
File : src/market/WildcatMarketToken.sol

+   uint256 constant MAX_TYPE = 2**256 - 1;
-    if (allowed != type(uint256).max) {
+    if (allowed != MAX_TYPE) {
        uint256 newAllowance = allowed - amount;
        _approve(from, msg.sender, newAllowance);
    }

```

## [G-10] Use `if` instead of `&&`

_1 Interface in 1 File_

```solidity
File : src/WildcatMarketControllerFactory.sol

 if (
      (protocolFeeBips > 0 && nullFeeRecipient) ||
      (hasOriginationFee && nullFeeRecipient) ||
      (hasOriginationFee && nullOriginationFeeAsset)
    ) {
      revert InvalidProtocolFeeConfiguration();
    }

```
[204-210](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L204C4-L210C6)

## [G-11] Remove unused `import` variables

_3 Interfaces in 2 Files_

```solidity
File : src/WildcatSanctionsEscrow.sol

import { IERC20 } from './interfaces/IERC20.sol';
import { IChainalysisSanctionsList } from './interfaces/IChainalysisSanctionsList.sol';//@audit remove IChainalysisSanctionsList
import { SanctionsList } from './libraries/Chainalysis.sol'; //@audit remove SanctionsList 
import { WildcatSanctionsSentinel } from './WildcatSanctionsSentinel.sol';
import { IWildcatSanctionsEscrow } from './interfaces/IWildcatSanctionsEscrow.sol';

```
[04-08](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L4C1-L8C84)

```solidity
File : src/WildcatSanctionsSentinel.sol

import { IChainalysisSanctionsList } from './interfaces/IChainalysisSanctionsList.sol';
import { IWildcatArchController } from './interfaces/IWildcatArchController.sol';
import { IWildcatSanctionsSentinel } from './interfaces/IWildcatSanctionsSentinel.sol';
import { SanctionsList } from './libraries/Chainalysis.sol';//@audit remove SanctionsList
import { WildcatSanctionsEscrow } from './WildcatSanctionsEscrow.sol';

```
[04-08](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L4C1-L8C71)
