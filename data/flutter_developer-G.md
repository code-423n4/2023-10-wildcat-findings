## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | | | Deploys markets and manages their configurable parameters (APR, reserve ratio) andmaintains set of approved lenders. |


| [G-02] | | | Base contract for Wildcat markets.|

| [G-03] | | | Deploys controllers and manages protocol fee information.|

| [G-04] | | | Registry for borrowers, controller factories, controllers and markets.|

| [G-05] | | |Withdrawal functionality for Wildcat markets.|

| [G-06] | | | 	Generic math functions used in the codebase.|


| [G-07] | | | uint cast functions.|

| [G-08] | | | Contains functions for calculating interest, protocol fees and delinquency fees.|

| [G-09] | | | 	Methods for role management and configuration by controller.|


| [G-10] | | | Helper functions for querying strings from methods that can return either string or bytes32.|


| [G-11] | | | Main contract for Wildcat markets that inherits all base contracts.|

| [G-12] | | | 	Defines the market state struct and helper functions for reading from it and calculating basic values like required reserves and scaling/normalizing balances.|


| [G-13] | | | 	Contract that interfaces with Chainalysis, allows borrowers to override lenders' sanction statuses and deploys escrows.|




| [G-14]  | | | Library for deploying contracts using code storage for init code to prevent oversized contracts|

| [G-15] | | | First-in-first-out queue used for unpaid withdrawal batches.|

| [G-16] | | | ERC20 functionality for Wildcat markets.|

| [G-17] | | | Helper functions and constants for errors and Solidity panic codes.|


| [G-18] | | | Defines withdrawal state struct.|

| [G-19] | | | Reentrancy guard from Seaport. | 

| [G-20] | | | Escrow contract that holds assets for a particular account until it is removed from the Chainalysis sanctions list or until the borrower overrides the account's sanction status.|


| [G-21] | | |  helper for  booleans .|


| [G-22] | | | Constant for Chainalysis SanctionsList.|





### Description of Code 


## [G-01] Deploys markets and manages their configurable parameters (APR, reserve ratio) and maintains set of approved lenders.

```solidity
file: src/WildCatMarketController.sol
```
    WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L188
```





## [G-02] Base contract for Wildcat markets.

```solidity
file: src/market/WidcatMarketBase.sol

```
 solidity line of code 
 
contract WildcatMarketBase is ReentrancyGuard, IMarketEventsAndErrors {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L14
```


## [G-03] Deploys controllers and manages protocol fee information.

```solidity
  file:src/WildcatMarketControllerFactory.sol
```
   function setProtocolFeeConfiguration(
    address feeRecipient,
    address originationFeeAsset,
    uint80 originationFeeAmount,
    uint16 protocolFeeBips
  ) external onlyArchControllerOwner {
    bool hasOriginationFee = originationFeeAmount > 0;
    bool nullFeeRecipient = feeRecipient == address(0);
    bool nullOriginationFeeAsset = originationFeeAsset == address(0);
    if (
      (protocolFeeBips > 0 && nullFeeRecipient) ||
      (hasOriginationFee && nullFeeRecipient) ||
      (hasOriginationFee && nullOriginationFeeAsset)
    ) {
      revert InvalidProtocolFeeConfiguration();
    }
    _protocolFeeConfiguration = ProtocolFeeConfiguration({
      feeRecipient: feeRecipient,
      originationFeeAsset: originationFeeAsset,
      originationFeeAmount: originationFeeAmount,
      protocolFeeBips: protocolFeeBips
    });
  }

```
 https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L195-L217
```



## [G-04] Registry for borrowers, controller factories, controllers and markets.

```solidity
file: src/WildcatArchController.sol
```
     if (!_borrowers.add(borrower)) {
      revert BorrowerAlreadyExists();
    }
    emit BorrowerAdded(borrower);
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L64
```


## [G-05] Withdrawal functionality for Wildcat markets.

```solidity
file: src/market/WildcatMarketWidthrawals.sol
```
    AccountWithdrawalStatus storage status = _withdrawalData.accountStatuses[expiry][
      accountAddress];
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L147
```



## [G-06] 	Generic math functions used in the codebase.


```solidity
file: src/libraries/MathUtils.sol
```
      if or(iszero(b), gt(a, div(sub(not(0), div(b, 2)), RAY))) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L159
```



## [G-07] 
```solidity
file: 
```
  _assertNonOverflow(x == (y = uint216(x)));
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/SafeCastLib.sol#L122
```




## [G-08]  Contains functions for calculating interest, protocol fees and delinquency fees.


```solidity
file: src/libraries/FeeMath.sol

```
 function calculateLinearInterestFromBips(
    uint256 rateBip,
    uint256 timeDelta
  ) internal pure returns (uint256 result) {
    uint256 rate = rateBip.bipToRay();
    uint256 accumulatedInterestRay = rate * timeDelta;
    unchecked {
      return accumulatedInterestRay / SECONDS_IN_365_DAYS;
    }
  }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L19
```



## [G-09]  	Methods for role management and configuration by controller.

```solidity
file: src/market/WildcatMarketConfig.sol

```
 if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
      revert NotReversedOrStunning();
    }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L94-L96
```



## [G-10]  Helper functions for querying strings from methods that can return either string or bytes32.

```solidity
file: src/libraries/StringQuery.sol
```
returndatacopy(0x00, 0x00, 0x20)
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/StringQuery.sol#L67
```




## [G-11]  Main contract for Wildcat markets that inherits all base contracts.

```solidity
file: src/market/WildcatMaket.sol 

```
 WildcatMarketBase
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L11
```




## [G-12]  	Defines the market state struct and helper functions for reading from it and calculating basic values like required reserves and scaling/normalizing balances.

```solidity
file: src/libraries/MarketState.sol  
```
  function withdrawableProtocolFees(
    MarketState memory state,
    uint256 totalAssets
  ) internal pure returns (uint128) {
    uint256 totalAvailableAssets = totalAssets - state.normalizedUnclaimedWithdrawals;
    return uint128(MathUtils.min(totalAvailableAssets, state.accruedProtocolFees));
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L105-L111
```



## [G-13] 	Contract that interfaces with Chainalysis, allows borrowers to override lenders' sanction statuses and deploys escrows.

```solidity
file: src/WildcatSanctionsSentin.sol
```
  !sanctionOverrides[borrower][account] &&
      IChainalysisSanctionsList(chainalysisSanctionsList).isSanctioned(account);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L41-L42
```



## [G-14]  Library for deploying contracts using code storage for init code to prevent oversized contracts.


```solidity
file: src/libraries/LibStoredInitCode.sol
```

    deployment = create2WithStoredInitCode(initCodeStorage, salt, 0);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol#L103
```




##   [G-15]  First-in-first-out queue used for unpaid withdrawal batches.


```solidity
file: src/libraries/FIFOQueue.sol
```

    delete arr.data[startIndex];
    arr.startIndex = startIndex + 1;
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L66-L67
```




## [G-16]  ERC20 functionality for Wildcat markets.


```solidity
file: src/WildcatMarketToken.sol
```
  if (allowed != type(uint256).max) {
      uint256 newAllowance = allowed - amount;
      _approve(from, msg.sender, newAllowance);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L48-L52
```



## [G-17]  Helper functions and constants for errors and Solidity panic codes.

```solidity
file:  src/libraries/Errors.sol
```


uint256 constant Panic_CompilerPanic = 0x00;
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Errors.sol#L4
```


## [G-18]  Defines withdrawal state struct.



```solidity
file: src/libraries/Widthrawals.sol
```
    uint256 unavailableAssets = state.normalizedUnclaimedWithdrawals +
      state.normalizeAmount(priorScaledAmountPending) +
```
 https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L55-L56
```



## [G-19] | | | Reentrancy guard from Seaport. 

```solidity
file: 
```
_reentrancyGuard = _ENTERED;
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/ReentrancyGuard.sol#L65
```



### [G-20]  Escrow contract that holds assets for a particular account until it is removed from the Chainalysis sanctions list or until the borrower overrides the account's sanction status.


```solidity
file: src/WildcatSanctionsEscrow.sol
```
 return IERC20(asset).balanceOf(address(this));
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L22
```



## [G-21] helpers for boolean .|


```solidity
file: src/libraries/BoolUtils.sol
```
     c := xor(a, b)
```
 https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/BoolUtils.sol#L19
```


## [G-22] Constant for Chainalysis SanctionsList.

```solidity
file: src/libraries/Chainalysis.sol
```
IChainalysisSanctionsList constant SanctionsList = IChainalysisSanctionsList(
  0x40C57923924B5c5c5455c48D93317139ADDaC8fb
);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Chainalysis.sol#L6
```