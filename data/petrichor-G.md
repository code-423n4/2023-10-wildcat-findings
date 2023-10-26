# GAS OPTIMIZATION

|      |  issue  |  instance  |
|------|---------|------------|
|[G-01]|Make 3 event parameters indexed when possible|10|
|[G-02]|Avoid contract existence checks by using low level calls|9|
|[G-03]|Use constants instead of type(uintx).max|1|
|[G-04]|Using Fixed Bytes Is Cheaper Than Using String|1|
|[G-05]| Use assembly to validate msg.sender|1|
|[G-06]| Can Make The Variable Outside The Loop To Save Gas |3|
|[G-07]|Expressions for constant values such as a call to keccak256(), should use immutable rather than constant|1|
|[G-08]|Amounts should be checked for 0 before calling a transfer|2|
|[G-09]| Use hardcode address instead address(this)|3|
|[G-10]|Structs can be packed to use fewer storage slots|2|




##  [G-01] Make 3 event parameters indexed when possible

 If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
File:  src/WildcatArchController.sol
 29 event MarketAdded(address indexed controller, address market);
  event MarketRemoved(address market);

  event ControllerFactoryAdded(address controllerFactory);
  event ControllerFactoryRemoved(address controllerFactory);

  event BorrowerAdded(address borrower);
  event BorrowerRemoved(address borrower);

  event ControllerAdded(address indexed controllerFactory, address controller);
  event ControllerRemoved(address controller);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L29-L39


```solidity
File:  src/WildcatMarketControllerFactory.sol
 16  event NewController(address borrower, address controller, string namePrefix, string symbolPrefix);
  event UpdateProtocolFeeConfiguration(
    address feeRecipient,
    uint16 protocolFeeBips,
    address originationFeeAsset,
   22  uint256 originationFeeAmount
  );
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L16-L22


## [G‑02] Avoid contract existence checks by using low level calls

 In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.



```solidity
File:  src/WildcatSanctionsSentinel.sol
100  if (!IWildcatArchController(archController).isRegisteredMarket(msg.sender)) {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L100


```solidity
File:  src/market/WildcatMarketBase.sol
99   decimals = IERC20Metadata(parameters.asset).decimals();

172  address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(

204  if (IWildcatMarketController(controller).isAuthorizedLender(accountAddress)) {

239  return IERC20(asset).balanceOf(address(this));        
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L99
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L172
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L204
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L239



```solidity
File:  src/market/WildcatMarketConfig.sol
75   if (!IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {

94   if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {    
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L75
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L94



```solidity
File:  src/market/WildcatMarketWithdrawals.sol
164    if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {

166    address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(    
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L164
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L166


```solidity
File:  src/WildcatSanctionsSentinel.sol
42  IChainalysisSanctionsList(chainalysisSanctionsList).isSanctioned(account);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L42





## [G-03] Use constants instead of type(uintx).max


The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.



```solidity
File:  src/market/WildcatMarketToken.sol
49   if (allowed != type(uint256).max) {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L49

## [G-04] Using Fixed Bytes Is Cheaper Than Using String

If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

```solidity
File:  src/market/WildcatMarketBase.sol
57  string public name;

60  string public symbol;
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L57-L60




## [G-05] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the deployMarket function with the least amount of opcodes necessary. to save gas.

```solidity
File:  src/WildcatMarketController.sol
    if (msg.sender == borrower) {
      if (!archController.isRegisteredBorrower(msg.sender)) {
        revert NotRegisteredBorrower();
      }
    } else if (msg.sender != address(controllerFactory)) {
      revert CallerNotBorrowerOrControllerFactory();
    }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L302-L308


## [G-06] Can Make The Variable Outside The Loop To Save Gas 

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.


```solidity
File:  src/WildcatMarketController.sol
155   address lender = lenders[i];

171  address lender = lenders[i];

184  address market = markets[i];
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L155
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L171
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L184







## [G-07] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

In Solidity, when you have expressions involving constant values, such as a call to keccak256(), it is recommended to use the immutable keyword instead of constant to save gas.

```solidity
File:  src/WildcatSanctionsSentinel.sol
11    bytes32 public constant override WildcatSanctionsEscrowInitcodeHash =
    keccak256(type(WildcatSanctionsEscrow).creationCode);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L11-L12


## [G-08] Amounts should be checked for 0 before calling a transfer

In Solidity, when performing token transfers or transactions involving amounts, it is often recommended to check if the amount is zero before initiating the transfer. This check helps to save gas by avoiding unnecessary transaction executions when the amount is already zero.

```solidity
File:  src/market/WildcatMarket.sol
129   asset.safeTransfer(msg.sender, amount);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L129


```solidity
File:  src/market/WildcatMarketToken.sol
54  _transfer(from, to, amount);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L54




## [G-09] Use hardcode address instead address(this)


The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.





```solidity
File:  src/market/WildcatMarketBase.sol
175   address(this)

239   return IERC20(asset).balanceOf(address(this));

524    emit Transfer(address(this), address(0), normalizedAmountPaid);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L175
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L239
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L524



## [G-10] Structs can be packed to use fewer storage slots

In Solidity, structs can be packed to use fewer storage slots, which can lead to gas savings in certain cases. When a struct is declared, by default, each member of the struct is allocated its own storage slot, even if the members could potentially fit within a single slot.

```solidity
File:  src/WildcatMarketController.sol
struct TmpMarketParameterStorage {
  address asset;
  string namePrefix;
  string symbolPrefix;
  address feeRecipient;
  uint16 protocolFeeBips;
  uint128 maxTotalSupply;
  uint16 annualInterestBips;
  uint16 delinquencyFeeBips;
  uint32 withdrawalBatchDuration;
  uint16 reserveRatioBips;
  uint32 delinquencyGracePeriod;
}
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L18-L30


```solidity
File:   src/libraries/MarketState.sol
struct MarketState {
  bool isClosed;
  uint128 maxTotalSupply;
  uint128 accruedProtocolFees;
  // Underlying assets reserved for withdrawals which have been paid
  // by the borrower but not yet executed.
  uint128 normalizedUnclaimedWithdrawals;
  // Scaled token supply (divided by scaleFactor)
  uint104 scaledTotalSupply;
  // Scaled token amount in withdrawal batches that have not been
  // paid by borrower yet.
  uint104 scaledPendingWithdrawals;
  uint32 pendingWithdrawalExpiry;
  // Whether market is currently delinquent (liquidity under requirement)
  bool isDelinquent;
  // Seconds borrower has been delinquent
  uint32 timeDelinquent;
  // Annual interest rate accrued to lenders, in basis points
  uint16 annualInterestBips;
  // Percentage of outstanding balance that must be held in liquid reserves
  uint16 reserveRatioBips;
  // Ratio between internal balances and underlying token amounts
  uint112 scaleFactor;
  uint32 lastInterestAccruedTimestamp;
}
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L13-L37
