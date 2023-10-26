# WILDCAT PROTOCOL - GAS REPORT.

| ID    |   Title    |
|:---:  |:---          |
| 1.  |  In cycles you can use ‘unchecked{++i;}’ instead of ‘i++’ to improve gas efficiency.  |
| 2   | Use calldata instead of memory on immutable variables to improve gas efficiency.   |
| 3.  |  Cache the length of the array in a variable to avoid reading it on every cycle to improve gas efficiency.  |
| 4.  |  Ccache memory variable outside the cycle and just overwrite it in the cycle and avoid creating a new variable on every cycle to improve gas efficiency.  |
| 5.  |  Use the declared returns variables to improve gas efficiency.  |
| 6.  | Reorder the Struct declaration to improve gas efficiency.   |

### 1.- In cycles you can use ‘unchecked{++i;}’ instead of ‘i++’ to improve gas efficiency.

In order to improve gas efficiency you can change the way you increment the variable “i” in the for loops, this helps you to reduce gas costs.

for example, you can change the for loops in your codebase from this:
```solidity
for (uint256 i = 0; i < count; i++) {
   arr[i] = _borrowers.at(start + i);
}
```
To this:
```solidity
for (uint256 i = 0; i < count; ) {
   arr[i] = _borrowers.at(start + i);
unchecked{++i;}
}
```

Github links where code can be improved:

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L93-L95
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L124-L139
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L179-L181
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L222-L224

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L133-L135
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L154-L159
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L170-L175
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L183-L189
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L212-L214

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L146-L148


### 2.- Use calldata instead of memory on immutable variables to improve gas efficiency.

the use of calldata reserved word instead of memory in the function's immutable arguments improve gas efficiency cause the calldata arguments are not copied to memory this helps to reduce the gas cost that the function uses.

For example, you can change the memory argument to calldata in the arguments:
```solidity
function authorizeLenders(address[] memory lenders) external onlyBorrower {}
```

Change it for this:
```solidity
function authorizeLenders(address[] calldata lenders) external onlyBorrower {}
```

You can improve this in the following sections:

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L153
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L169
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L182
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L221-L225
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L293-L294
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L370-L374
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L394-L402

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L317-L327

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L163



### 3.- Cache the length of the array in a variable to avoid reading it on every cycle to improve gas efficiency.

Reading the length of an array in every iteration of a for loop can increase the gas cost of the functions, for that reason it is recommended to cache the length value of the array in a memory variable before the cycle starts and use this variable instead of reading the length of the array on every iteration.

For example, in the authorizedLenders function in the WildCat Market Controller you can cache the array of the length before the cycle.

You can change this:
```solidity
function authorizeLenders(address[] memory lenders) external onlyBorrower {
   for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.add(lender)) {
         emit LenderAuthorized(lender);
      }
   }
}
```

For this:
```solidity
function authorizeLenders(address[] memory lenders) external onlyBorrower {
   uint256 length = lenders.length;
   for (uint256 i = 0; i < length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.add(lender)) {
         emit LenderAuthorized(lender);
      }
   }
}
```
You can improve this in the following lines:
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L153-L160
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L169-L176
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L182-L190

 

### 4.- Create the cache memory variable outside the cycle and just overwrite it in the cycle and avoid creating a new variable on every cycle to improve gas efficiency.

Creating a new variable on every for iteration can increase the gas costs of the functions, it’s recommended to create a unique variable outside the for loop and only overwrite this variable on every iteration of the for loop, this helps to improve the gas efficiency of the function.

For example, in the authorizedLenders function in the WildCat Market Controller you can create the address variable outside the loop and just overwrite this variable inside the loop.

You can change this:
```solidity
function authorizeLenders(address[] memory lenders) external onlyBorrower {
   for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.add(lender)) {
         emit LenderAuthorized(lender);
      }
   }
}
```

For This:
```solidity
function authorizeLenders(address[] memory lenders) external onlyBorrower {
   address lender;
   for (uint256 i = 0; i < lenders.length; i++) {
      lender = lenders[i];
      if (_authorizedLenders.add(lender)) {
         emit LenderAuthorized(lender);
      }
   }
}
```
You can make this improvement on the following lines:
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L153-L160
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L169-L176
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L182-L190


### 5.- Use the declared returns variables to improve gas efficiency.

When you declare the name of the variable that will be returned, it’s not necessary to use the return keyword to return a value, you just need to assign the variable declared in the returns statement at the beginning of the function and solidity will implicity return that value when the function ends.

if you declared a variable to be returned but not use this value in the function you are increasing the gas cost of the function, for that reason it is recommended to use/assign the variable declared in the returns statement at the beginning of the function or not to define that variable at all, this will help to improve gas efficiency of the function.

for example, in the function getProtocolFeeConfiguration in the WildcatMarketControllerFactory, you can assign the returns variables declared to be returned by solidity:

You can change this:
```solidity
function getProtocolFeeConfiguration()
external
view
returns (address feeRecipient, address originationFeeAsset, uint80 originationFeeAmount, uint16 protocolFeeBips)
{
   return (
      _protocolFeeConfiguration.feeRecipient,
      _protocolFeeConfiguration.originationFeeAsset,
      _protocolFeeConfiguration.originationFeeAmount,
      _protocolFeeConfiguration.protocolFeeBips
   );
}
```
For this:
```solidity
function getProtocolFeeConfiguration()external
view
returns (address feeRecipient, address originationFeeAsset, uint80 originationFeeAmount, uint16 protocolFeeBips)
{
   feeRecipient = _protocolFeeConfiguration.feeRecipient;
   originationFeeAsset = _protocolFeeConfiguration.originationFeeAsset;
   originationFeeAmount = _protocolFeeConfiguration.originationFeeAmount;
   protocolFeeBips = _protocolFeeConfiguration.protocolFeeBips;
}
```

Github links:

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L165-L181

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsSentinel.sol#L65-L85

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L19-L28

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MarketState.sol#L87-L98

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MathUtils.sol#L30-L39

***Look for code consistency throughout the whole code base***


### 6.- Reorder the Struct declaration to improve gas efficiency.

The current MarkeState struct used to store all market info is ordered in a way that makes the contract use 5 storage slots, as you know more slots used is equal to more gas expended for the contract and the users, if you order the struct to be more storage efficient you can reduce the gas cost for the contract and the users

So, in order to improve gas and storage efficiency you can reorder the struct in a next way,  it’s important to let you know that this change don’t need any other change in the code.

The current struct uses 5 storage slots, so you can change the Struct to only use 4 storage slots, for that, you need to change from this:

```solidity
​​struct MarketState {
   bool isClosed; // 1
   uint128 maxTotalSupply; //2
   uint128 accruedProtocolFees; //2
   // Underlying assets reserved for withdrawals which have been paid
   // by the borrower but not yet executed.
   uint128 normalizedUnclaimedWithdrawals; //3
   // Scaled token supply (divided by scaleFactor)
   uint104 scaledTotalSupply; //3
   // Scaled token amount in withdrawal batches that have not been
   // paid by borrower yet.
   uint104 scaledPendingWithdrawals; //4
   uint32 pendingWithdrawalExpiry; //4
   // Whether market is currently delinquent (liquidity under requirement)
   bool isDelinquent; //4
   // Seconds borrower has been delinquent
   uint32 timeDelinquent; //5
   // Annual interest rate accrued to lenders, in basis points
   uint16 annualInterestBips; //5
   // Percentage of outstanding balance that must be held in liquid reserves
   uint16 reserveRatioBips; //5
   // Ratio between internal balances and underlying token amounts
   uint112 scaleFactor; //5
   uint32 lastInterestAccruedTimestamp; //5
}


```

To this:

```solidity
struct MarketState {
   uint128 maxTotalSupply;//1
   uint128 accruedProtocolFees; //1
   // Underlying assets reserved for withdrawals which have been paid
   // by the borrower but not yet executed.
   uint128 normalizedUnclaimedWithdrawals; //2
   // Scaled token supply (divided by scaleFactor)
   uint104 scaledTotalSupply;//2
   bool isClosed;//2
   // Scaled token amount in withdrawal batches that have not been
   // paid by borrower yet.
   uint104 scaledPendingWithdrawals;//3
   uint32 pendingWithdrawalExpiry;//3
   // Whether market is currently delinquent (liquidity under requirement)
   bool isDelinquent;//3
   // Seconds borrower has been delinquent
   uint32 timeDelinquent;//3
   // Annual interest rate accrued to lenders, in basis points
   uint16 annualInterestBips;//3
   // Percentage of outstanding balance that must be held in liquid reserves
   uint16 reserveRatioBips;//3
   uint32 lastInterestAccruedTimestamp; //3
   // Ratio between internal balances and underlying token amounts
   uint112 scaleFactor; //4
}
```
this help to save 1 storage slot, which improves storage and gas efficiency in the contract.

