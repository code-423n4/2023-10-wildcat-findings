# Low && NC


# Summary Low

|      |  issue  | instance  |
|------|---------|-----------|
|[L-01]|ControllerAdded emits wrong controllerFactory address|2|
|[L-02]|Missing Event Emission in Setter Function|2|
|[L-03]|unused return of external call|6|
|[L-04]|pess Dubious typecast|5|
|[L-05]|External Call Inside Loop|1|
|[L-06]|Dangerous Usage of block.timestamp in WildcatMarketBase.sol|7|
|[L-07]|Reentrancy no eth|3|
|[L-08]|Missing Zero Address Validation|2|
|[L-09]|Unchecked Return Value in WildcatSanctionsEscrow.sol|1|
|[L-10]|Avoiding Dangerous Strict Equality in Account Balance Checks|1|
|[L-11]|Use `safetransfer` Instead Of `transfer`|1|
|[L-12]|pess unprotected initialize|4|

## [L-01] ControllerAdded emits wrong controllerFactory address
In the WildcatArchController.registerController function the ControllerAdded is emitted ([Link](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L153)).

It is defined as:
[Link](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L38)

```solidity
38   event ControllerAdded(address indexed controllerFactory, address controller);
```
So the first parameter should be the controllerFactory address.

When the event is emitted the first parameter is msg.sender. The issue is that the controllerFactory address and msg.sender can be different. So the event in some cases contains wrong information.

same issue here:

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L196


## [L-02] Missing Event Emission in Setter Function

Description: The setter function setAnnualInterestBips in the WildcatMarketController.sol contract does not emit an event. According to best practices, setter functions should emit events to provide transparency and allow external systems to track and react to state changes.

```solidity
  function setAnnualInterestBips(
    address market,
    uint16 annualInterestBips
  ) external virtual onlyBorrower onlyControlledMarket(market) {
    // If borrower is reducing the interest rate, increase the reserve
    // ratio for the next two weeks.
    if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {
      TemporaryReserveRatio storage tmp = temporaryExcessReserveRatio[market];

      if (tmp.expiry == 0) {
        tmp.reserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips());

        // Require 90% liquidity coverage for the next 2 weeks
        WildcatMarket(market).setReserveRatioBips(9000);
      }

      tmp.expiry = uint128(block.timestamp + 2 weeks);
    }

    WildcatMarket(market).setAnnualInterestBips(annualInterestBips);
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L468-488

`Recommendation`:
It is recommended to follow best practices and emit events in setter functions. Events provide a way to notify external systems and interested parties about changes in the contract's state. Emitting events in setter functions helps enhance transparency, improve auditability, and enable efficient tracking of state changes.

same issue here:

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L195-L217


## [L-03] unused return of external call
The return value of an external call is not stored in a local or state variable.

```solidity
354  LibStoredInitCode.create2WithStoredInitCode(marketInitCodeStorage, salt);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L354

`Recommendation`:

Ensure that all the return values of the function calls are used.

same issue here:

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L357

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L415-L420

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L435-L440

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L297

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L300

## [L-04] pess Dubious typecast
Description:
Check docs

```solidity
539  uint104 scaledAmountBurned = uint104(MathUtils.min(scaledAvailableLiquidity, scaledAmountOwed));
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L539

Recommendation:
Use clear constants

same issue here:
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L510

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L478

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L484

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L95

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L151-L153


## [L-05] External Call Inside Loop
Description: The function updateLenderAuthorization in the WildcatMarketController.sol contract contains an external call inside a loop. Specifically, the line WildcatMarket(market).updateAccountAuthorization(lender,_authorizedLenders.contains(lender)) at line 188 performs an external call within the loop.

```solidity
188   WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L188

Recommendation:
It is recommended to follow the "pull over push" strategy for external calls. This strategy helps mitigate potential denial-of-service attacks that may arise from making external calls inside a loop.


## [L-06] Dangerous Usage of block.timestamp in WildcatMarketBase.sol
Description: The function _getUpdatedState in the WildcatMarketBase.sol contract contains dangerous comparisons using block.timestamp. Specifically, the comparisons `expiry != state.lastInterestAccruedTimestamp` at line 365 and `block.timestamp != state.lastInterestAccruedTimestamp` at line 378 rely on block.timestamp, which can be manipulated by miners.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L365-L378

Recommendation:
It is recommended to avoid relying on block.timestamp for critical comparisons. Miners have the ability to manipulate the value of block.timestamp, which can lead to unexpected behaviors and vulnerabilities in the contract.

same issue here:
```
// @audit in the _calculateCurrentState function 414,428,434
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L339-L442 
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L495

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L472

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L141

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L49

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L112

## [L-07] Reentrancy no eth
The resetReserveRatio function in the `WildcatMarketController` contract is susceptible to a reentrancy vulnerability. The vulnerability occurs due to improper ordering of state modifications and external calls. The function retrieves a `TemporaryReserveRatio` struct from the `temporaryExcessReserveRatio` mapping and performs checks on the struct's values. However, it then makes an external call to the `setReserveRatioBips` function of the `WildcatMarket` contract before deleting the entry in the `temporaryExcessReserveRatio` mapping. This creates a potential reentrancy vulnerability, as the `temporaryExcessReserveRatio` state variable can be accessed by other functions in the contract.

Description:
Detection of the [reentrancy bug](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy). Do not report reentrancies that involve Ether (see `reentrancy-eth`).

```solidity
490  function resetReserveRatio(address market) external virtual {
    TemporaryReserveRatio memory tmp = temporaryExcessReserveRatio[market];
    if (tmp.expiry == 0) {
      revertWithSelector(AprChangeNotPending.selector);
    }
    if (block.timestamp < tmp.expiry) {
      revertWithSelector(ExcessReserveRatioStillActive.selector);
    }

    WildcatMarket(market).setReserveRatioBips(uint256(tmp.reserveRatioBips).toUint16());
    delete temporaryExcessReserveRatio[market];
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L490-L501

`Recommendation`:
Apply the [`check-effects-interactions` pattern](http://solidity.readthedocs.io/en/v0.4.21/security-considerations.html#re-entrancy).

same issue here:
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L163-L187

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L468-L488


## [L-08] Missing Zero Address Validation
Description: The constructor function in the WildcatMarketControllerFactory.sol contract lacks a zero-check validation for the _sentinel parameter. Specifically, the line sentinel = _sentinel at line 78 assigns the _sentinel value to the sentinel variable without verifying if it is a zero address.

```solidity
78   sentinel = _sentinel;
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L78


`Recommendation`:
It is recommended to include a zero-check validation when assigning the _sentinel value to the sentinel variable. Zero address validation is important to ensure that an essential parameter is not set to an invalid or uninitialized value.

same issue here:
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L24-L26

## [L-09] Unchecked Return Value in WildcatSanctionsEscrow.sol

Description: The function releaseEscrow in the WildcatSanctionsEscrow.sol contract ignores the return value of the transfer function at line 38, which transfers an amount of tokens to an account using the IERC20 interface. Ignoring the return value of an external transfer operation can lead to potential issues, such as loss of funds or inconsistent state.

```solidity
38  IERC20(asset).transfer(account, amount);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L38

Recommendation:
It is recommended to either use the SafeERC20 library or ensure that the return value of the transfer function is checked in the releaseEscrow function. The SafeERC20 library provides additional checks and reverts the transaction in case of transfer failures, ensuring the safety of funds.


## [L-10] Avoiding Dangerous Strict Equality in Account Balance Checks
In the code snippet located at WildcatMarket.collectFees() in the Solidity contract WildcatMarket.sol, there is a usage of a dangerous strict equality check. Specifically, the line withdrawableFees == 0 is utilizing strict equality to determine if an account has enough Ether or tokens. This approach can be easily manipulated by attackers, leading to potential security vulnerabilities.

```solidity
102   if (withdrawableFees == 0) {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L102

Recommendation:
To ensure robustness and security in account balance checks, it is recommended to avoid using strict equality comparisons. Instead, utilize appropriate comparison operators and consider incorporating additional safeguards. Some recommended practices include:

## [L-11] Use `safetransfer` Instead Of `transfer`

```solidity
38  IERC20(asset).transfer(account, amount);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L38

## [L-12] pess unprotected initialize
Initializers must be protected

```solidity
41 modifier onlyControllerFactory() {
    if (!_controllerFactories.contains(msg.sender)) {
      revert NotControllerFactory();
    }
    _;
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L41-L46

`Recommendation`:
Protect initializers with modifiers/require statements

same issue here:
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L48-L53

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L80-L85

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L65-L70




# Summary NC

|       |  issue  | instance  |
|-------|---------|-----------|
|[NC-01]| naming convention|4|
|[NC-02]| Use of Assembly|3|
|[NC-03]| too many digits |1|


## [NC-01] naming convention
Description:

Solidity defines a [naming convention](https://solidity.readthedocs.io/en/v0.4.25/style-guide.html#naming-conventions) that should be followed. #### Rule exceptions - Allow constant variable name/symbol/decimals to be lowercase (`ERC20`). - Allow `_` at the beginning of the `mixed_case` match for private variables and unused parameters.

```solidity
11  EnumerableSet.AddressSet internal _markets;
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L11

`Recommendation`:

Follow the Solidity [naming convention](https://solidity.readthedocs.io/en/v0.4.25/style-guide.html#naming-conventions).

same issue here :
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L12
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L13
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L14


## [NC-02] Use of Assembly
Description: The function enforceParameterConstraints in the WildcatMarketController.sol contract utilizes assembly, specifically the code block marked as "INLINE ASM" from lines 403 to 409. Assembly is considered error-prone and should generally be avoided due to its complexity and potential for introducing vulnerabilities.

```solidity
    assembly {
      if or(iszero(mload(namePrefix)), iszero(mload(symbolPrefix))) {
        // revert EmptyString();
        mstore(0x00, 0xecd7b0d1)
        revert(0x1c, 0x04)
      }
    }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L403-L409

Recommendation:
It is recommended to avoid using assembly, particularly the evm assembly, in smart contracts. Assembly code is low-level and can be difficult to understand, maintain, and audit. It increases the risk of introducing errors and vulnerabilities into the contract code.

same issue here :

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L375-L385

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L509-L514

## [NC-03] too many digits 
Literals with many digits are difficult to read and review.

```solidity
  bytes32 public constant override WildcatSanctionsEscrowInitcodeHash =
    keccak256(type(WildcatSanctionsEscrow).creationCode);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L11-L12
Recommendation:

Use: - [Ether suffix](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html#ether-units), - [Time suffix](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html#time-units), or - [The scientific notation](https://solidity.readthedocs.io/en/latest/types.html#rational-and-integer-literals)
