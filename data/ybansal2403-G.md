## Gas Optimizations

| Number        | Issue                                                                                          | Instances |
| ------------- | :--------------------------------------------------------------------------------------------- | :-------: |
| [G-01](#g-01) | State variables that are used multiple times in a function should be cached in stack variables |     1     |
| [G-02](#g-02) | x += y and x -= y are more expensive than x = x + y and x = x-y for state variables            |     1     |
| [G-03](#g-03) | Use hardcode address instead of address(this)                                                  |    10     |
| [G-04](#g-04) | Use assembly for math (add, sub, mul, div)                                                     |    18     |
| [G-05](#g-05) | Avoid contract existence checks by using low level calls                                       |    16     |
| [G-06](#g-06) | Use Assembly To Check For address(0)                                                           |     4     |
| [G-07](#g-07) | Do not calculate constants                                                                     |     1     |
| [G-08](#g-08) | Use constants instead of type(uintx).max                                                       |     1     |

<a name='g-01'></a>

## [G-01] State variables that are used multiple times in a function should be cached in stack variables

_The following instances are missed in the automated report_

When performing multiple operations on a state variable in a function, it is recommended to cache it first. Either multiple reads or multiple writes to a state variable can save gas by caching it on the stack. Caching of a state variable replaces each Gwarmaccess `100 gas` with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. Saves 100 gas per instance.

Total Instances: `1`

Gas benchmarks based on function `test_first()` at dir `test/libraries/FIFOQueue.t.sol:FIFOQueueTest` which calls our function of interest
| | Avg |
| ------ | ------ |
| Before | 49377 |
| After | 49280 |

Gas Saving: `97`

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L27

```solidity
File: src/libraries/FIFOQueue.sol

//@audit  arr.startIndex at line 24
27: return arr.data[arr.startIndex];
```

```diff
diff --git a/src/libraries/FIFOQueue.sol b/src/libraries/FIFOQueue.sol
index e8d0dde..0752b78 100644
--- a/src/libraries/FIFOQueue.sol
+++ b/src/libraries/FIFOQueue.sol
@@ -21,10 +21,11 @@ library FIFOQueueLib {
   }

   function first(FIFOQueue storage arr) internal view returns (uint32) {
-    if (arr.startIndex == arr.nextIndex) {
+    uint128 arr_start_index = arr.startIndex;
+    if (arr_start_index == arr.nextIndex) {
       revert FIFOQueueOutOfBounds();
     }
-    return arr.data[arr.startIndex];
+    return arr.data[arr_start_index];
   }

   function at(FIFOQueue storage arr, uint256 index) internal view returns (uint32) {

```

<a name='g-02'></a>

## [G-02] x += y and x -= y are more expensive than x = x + y and x = x-y for state variables

Using the addition/subtraction operator instead of plus-equals or minus-equals saves `113 gas`.

Total Instances: `1`

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L178

```solidity
File:src/market/WildcatMarketBase.sol
178:  _accounts[escrow].scaledBalance += scaledBalance;

```

```diff
diff --git a/src/market/WildcatMarketBase.sol b/src/market/WildcatMarketBase.sol
index b35d65f..8ef9464 100644
--- a/src/market/WildcatMarketBase.sol
+++ b/src/market/WildcatMarketBase.sol
@@ -175,7 +175,7 @@ contract WildcatMarketBase is ReentrancyGuard, IMarketEventsAndErrors {
           address(this)
         );
         emit Transfer(accountAddress, escrow, state.normalizeAmount(scaledBalance));
-        _accounts[escrow].scaledBalance += scaledBalance;
+        _accounts[escrow].scaledBalance = _accounts[escrow].scaledBalance+scaledBalance;
         emit SanctionedAccountAssetsSentToEscrow(
           accountAddress,
           escrow,

```

<a name='g-03'></a>

## [G-03] Use hardcode address instead of address(this)

It can be more gas-efficient to use a hardcoded address instead of the address(this) expression.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;

    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");

        // Do something
    }
}
```

Total Instances: `10`

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L53C3-L53C99
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L243C5-L243C43

```solidity
File: src/WildcatMarketController.sol

53: uint256 internal immutable ownCreate2Prefix = LibStoredInitCode.getCreate2Prefix(address(this));

243: parameters.controller = address(this);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L175
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L239
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L524C1-L524C68

```solidity
File: src/market/WildcatMarketBase.sol

175:           address(this)

239:     return IERC20(asset).balanceOf(address(this));

524:    emit Transfer(address(this), address(0), normalizedAmountPaid);

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L44

```solidity
File: src/WildcatMarketControllerFactory.sol

44:    uint256 internal immutable ownCreate2Prefix = LibStoredInitCode.getCreate2Prefix(address(this));

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L91

```solidity
File: src/market/WildcatMarketWithdrawals.sol

91:  emit Transfer(msg.sender, address(this), amount);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L154

```solidity
File: src/market/WildcatMarket.sol

154: asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L77

```solidity
File: src/WildcatSanctionsSentinel.sol

77: address(this),
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L22

```solidity
File: src/WildcatSanctionsEscrow.sol

22: return IERC20(asset).balanceOf(address(this));

```

<a name='g-04'></a>

## [G-04] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

Addition:

```solidity
//addition in Solidity
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```

Gas: 303

```solidity
//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}

```

Gas: 263

Subtraction

```solidity

//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
  uint256 c = a - b;
}
```

Gas: 300

```solidity
//subtraction in assembly
function subAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := sub(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```

Gas: 263

Multiplication

```solidity
//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```

Gas: 325

```solidity
//multiplication in assembly
function mulAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := mul(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```

Gas: 265

Division

```solidity
//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```

Gas: 325

```solidity
//division in assembly
function divAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := div(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```

Gas: 265

Total Instances: `18`

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L131
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L210C4-L210C33

```solidity
File: src/WildcatMarketController.sol
131: uint256 count = end - start;

210: uint256 count = end - start;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L505
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L534

```solidity
File: src/market/WildcatMarketBase.sol
505:  uint104 scaledAmountOwed = batch.scaledTotalAmount - batch.scaledAmountBurned;

534:     uint104 scaledAmountOwed = batch.scaledTotalAmount - batch.scaledAmountBurned;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L144

```solidity
File: src/WildcatMarketControllerFactory.sol
144:   uint256 count = end - start;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L91
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L134
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L177
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L220

```solidity
File: src/WildcatArchController.sol
91: uint256 count = end - start;

134: uint256 count = end - start;

177:  uint256 count = end - start;

220  uint256 count = end - start;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L67C4-L67C55
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L155

```solidity
File: src/market/WildcatMarketWithdrawals.sol

67:  return newTotalWithdrawn - previousTotalWithdrawn;

155:  uint128 normalizedAmountWithdrawn = newTotalWithdrawn - status.normalizedAmountWithdrawn;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L35

```solidity
File: src/libraries/MathUtils.sol

35: uint256 accumulatedInterestRay = rate * timeDelta;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L24

```solidity
File: src/libraries/FeeMath.sol

24: uint256 accumulatedInterestRay = rate * timeDelta;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L45C5-L45C42
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L58
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L67
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L78

```solidity
File: src/libraries/FIFOQueue.sol

45: uint256 len = nextIndex - startIndex;

58: arr.nextIndex = nextIndex + 1;

67: arr.startIndex = startIndex + 1;

78: arr.startIndex = startIndex + n;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L50

```solidity
File: src/market/WildcatMarketToken.sol

50: uint256 newAllowance = allowed - amount;

```

<a name='g-05'></a>

## [G-05] Avoid contract existence checks by using low level calls

_The following instances are missed in the automated report_

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

Total Instances: `16`

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L95
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L97

```solidity
File: src/WildcatMarketController.sol

95: controllerFactory = IWildcatMarketControllerFactory(msg.sender);

97: archController = IWildcatArchController(parameters.archController);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L77
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L99
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L172
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L204
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L239

```solidity
File: src/market/WildcatMarketBase.sol

77:     MarketParameters memory parameters = IWildcatMarketController(msg.sender).getMarketParameters();

99:     decimals = IERC20Metadata(parameters.asset).decimals();

172:    address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(

204:    if (IWildcatMarketController(controller).isAuthorizedLender(accountAddress)) {

239:    return IERC20(asset).balanceOf(address(this));

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L77
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L329

```solidity
File: src/WildcatMarketControllerFactory.sol

77:  archController = IWildcatArchController(_archController);

329:  market = IWildcatMarketController(controller).deployMarket(
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L164
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L166C7-L166C73

```solidity
File: src/market/WildcatMarketWithdrawals.sol

164: if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {

166: address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L75C5-L75C87
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L94

```solidity
File: src/market/WildcatMarketConfig.sol

75: if (!IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {

94:  if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L42C7-L42C81

```solidity
File: src/WildcatSanctionsSentinel.sol

42:IChainalysisSanctionsList(chainalysisSanctionsList).isSanctioned(account);

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L22
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L38

```solidity
File: src/WildcatSanctionsEscrow.sol

22: return IERC20(asset).balanceOf(address(this));

38:   IERC20(asset).transfer(account, amount);


```

<a name='g-06'></a>

## [G-06] Use Assembly To Check For address(0)

It's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```solidity
contract MyContract {
    function isZeroAddress(address addr) public pure returns (bool) {
        uint256 addrInt = uint256(addr);

        assembly {
            // Load the zero address into memory
            let zero := mload(0x00)

            // Compare the address to the zero address
            let isZero := eq(addrInt, zero)

            // Return the result
            mstore(0x00, isZero)
            return(0, 0x20)
        }
    }
}
```

In the above example, we have a function isZeroAddress that takes an address as input and returns a boolean value indicating whether the address is equal to the zero address. Inside the function, we convert the address to an integer using uint256(addr), and then use assembly to compare the integer to the zero address.

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution.

Total Instances: `4`

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L79
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L524

```solidity
File:src/market/WildcatMarketBase.sol
79:     if ((parameters.protocolFeeBips > 0).and(parameters.feeRecipient == address(0))) {

524:    emit Transfer(address(this), address(0), normalizedAmountPaid);

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L202C4-L202C56
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L203

```solidity
File:src/WildcatMarketControllerFactory.sol
202: bool nullFeeRecipient = feeRecipient == address(0);

203: bool nullOriginationFeeAsset = originationFeeAsset == address(0);
```

<a name='g-07'></a>

## [G-07] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

Total Instances: `1`

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L11C3-L13C1

```solidity
File:src/WildcatSanctionsSentinel.sol

11:  bytes32 public constant override WildcatSanctionsEscrowInitcodeHash =
12:    keccak256(type(WildcatSanctionsEscrow).creationCode);

```

<a name='g-08'></a>

## [G-08] Use constants instead of type(uintx).max

It's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:

```solidity
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;

    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");

        // Do something
    }
}
```

In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

Total Instances: `1`

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L49

```solidity
File: src/market/WildcatMarketToken.sol

49:  if (allowed != type(uint256).max) {

```
