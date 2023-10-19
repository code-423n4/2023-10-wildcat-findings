### LOW - 1 Year of 365.25 days more accurate than 365 days 

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MathUtils.sol#L14
```
uint256 constant SECONDS_IN_365_DAYS = 365 days;
```

Earth year is really closer to 365.256363004 days which is standardized as 365.25 days of 86400 SI seconds each in the Julian Year.

Using 365 days results in slightly higher results for linear interest rate from calculations
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MathUtils.sol#L37

Its best to use seconds instead to be more accurate e.g 31557600 seconds 

### LOW-2 Function duplication in libraries 

The function calculateLinearInterestFromBips(...) is found in both /src/libraries
/MathUtils.sol and src/libraries/FeeMath.sol

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MathUtils.sol#L30C12-L30C44
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

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L19
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

Such code duplication can lead to errors, harms auditability, increases code size for no reason, readability, maintainability of code. 

### NC-1 - Underscore function parameters

Majority of contracts for function arguments/parameters do not use underscore them e.g 
```
func(uint _param) 
// vs
func(uint param)
```
However we observe underscores for params/functions used in the following parts of code:
 
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L72
```
constructor(
    address _archController,
    address _sentinel,
    MarketParameterConstraints memory constraints
  )
```

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsSentinel.sol#L24
```
constructor(address _archController, address _chainalysisSanctionsList) {
    archController = _archController;
    chainalysisSanctionsList = _chainalysisSanctionsList;
    _resetTmpEscrowParams();
  }
```

**Impact**-> It results in inconsistency of general accepted practises, styles and standards, affects readability, maintainability of the code

**Recommendation** -> Recommended to make all function parameters/arguments have a leading underscore or make all them not have underscore as the majority do not currently. 



### NC-2 - Prefix Custom Errors with Contract Name

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L24
```
error NotRegisteredBorrower();
error InvalidProtocolFeeConfiguration();
error CallerNotArchControllerOwner();
error InvalidConstraints();
error ControllerAlreadyDeployed();
```

Below are more instances examples 

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L16

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MathUtils.sol#L19

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/StringQuery.sol#L16

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/LibStoredInitCode.sol#L5

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L17

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/ReentrancyGuard.sol#L17 

It is best to prefix the events with contract name so that it is explicitly clear where the errors are arising from.

**Impact**-> Without this it can be harder to follow, debug, trace, errors understand where the error is arising from, given that in Solidity events can be passed along a chain and resulting error may be from somewhere else.

**Recommendation** -> Recommended to prefix e.g 
```
error WildcatMarketControllerFactory__NotRegisteredBorrower();
error WildcatMarketControllerFactory__InvalidProtocolFeeConfiguration();
error MathUtils__MulDivFailed();
```

### NC-3 Arrays inputs for functions lack checks for address(0) values

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L153
```
function authorizeLenders(address[] memory lenders) external onlyBorrower {
```

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L169
```
function deauthorizeLenders(address[] memory lenders) external onlyBorrower {
```

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L182
```
function updateLenderAuthorization(address lender, address[] memory markets) external {
```

The above has examples of addresses used in arrays such that each individual address e.g lenders[i] or markets[i] is not checked for address(0) before using it is used in function logic. 

**Recommendation** -> Ensure checks that lenders[i] != address(0) 


### NC-4 - Lack sanity checks function inputs values that must != 0 

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L468
```
function setAnnualInterestBips(
    address market,
    uint16 annualInterestBips
  )
```
Can lead to setting of 0 annualInterestBips if we follow its logic to  WildcatMarket(market).setAnnualInterestBips(annualInterestBips);


https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L72
```
constructor(
    address _archController,
    address _sentinel,
    MarketParameterConstraints memory constraints
  ) {
    archController = IWildcatArchController(_archController);
    sentinel = _sentinel;
    if (
      constraints.minimumAnnualInterestBips > constraints.maximumAnnualInterestBips ||
      constraints.maximumAnnualInterestBips > 10000 ||
      constraints.minimumDelinquencyFeeBips > constraints.maximumDelinquencyFeeBips ||
      constraints.maximumDelinquencyFeeBips > 10000 ||
      constraints.minimumReserveRatioBips > constraints.maximumReserveRatioBips ||
      constraints.maximumReserveRatioBips > 10000 ||
      constraints.minimumDelinquencyGracePeriod > constraints.maximumDelinquencyGracePeriod ||
      constraints.minimumWithdrawalBatchDuration > constraints.maximumWithdrawalBatchDuration
    ) {
      revert InvalidConstraints();
    }
    MinimumDelinquencyGracePeriod = constraints.minimumDelinquencyGracePeriod;
    MaximumDelinquencyGracePeriod = constraints.maximumDelinquencyGracePeriod;
    MinimumReserveRatioBips = constraints.minimumReserveRatioBips;
    MaximumReserveRatioBips = constraints.maximumReserveRatioBips;
    MinimumDelinquencyFeeBips = constraints.minimumDelinquencyFeeBips;
    MaximumDelinquencyFeeBips = constraints.maximumDelinquencyFeeBips;
    MinimumWithdrawalBatchDuration = constraints.minimumWithdrawalBatchDuration;
    MaximumWithdrawalBatchDuration = constraints.maximumWithdrawalBatchDuration;
    MinimumAnnualInterestBips = constraints.minimumAnnualInterestBips;
    MaximumAnnualInterestBips = constraints.maximumAnnualInterestBips;

```
Constructor for Factory does not check if constraints for market are not zero values some like fees ca be but grace period, reserve ratio bips and minim batch duration ideally shouldn't 

**Impact**-> It can lead to errors, code not behaving as expected, unexpected reverts, potential bugs etc

**Recommendation** -> It is recommended to ensure all cases where inputs of values = 0 in functions are checked to ensure they do not cause any problems and revert in such cases so that zero values are not allowed e.g

### NC-5 - Format comments consistently 

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L269
```
/**
   * @dev Deploys a create2 deployment of `WildcatMarket` unique to the
   *      combination of `asset, namePrefix, symbolPrefix` and registers
   *      it with the arch-controller.
   *
   *      If a market has already been deployed with these parameters,
   *      reverts with `MarketAlreadyDeployed`.
   *
   *      If `msg.sender` is not `borrower` or `controllerFactory`,
   *      reverts with `CallerNotBorrowerOrControllerFactory`.
   *
   *->	    If `msg.sender == borrower && !archController.isRegisteredBorrower(msg.sender)`,
   *		  reverts with `NotRegisteredBorrower`.
   *
   *      If called by `controllerFactory`, skips borrower check.
   *
   *      If either string is empty, reverts with `EmptyString`.
   *
   *      If `originationFeeAmount` returned by controller factory is not zero,
   *      transfers `originationFeeAmount` of `originationFeeAsset` from
   *      `msg.sender` to `feeRecipient`.
   */
```
In above code 3 paragraph "If" is indented more than others 

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L74
```
/**
   * @notice  Calculate the number of seconds that the market has been in
   *          penalized delinquency since the last update, and update
   *          `timeDelinquent` in state.
   *
   * @dev When `isDelinquent`, equivalent to:
   *        max(0, timeDelta - max(0, delinquencyGracePeriod - previousTimeDelinquent))
   *      When `!isDelinquent`, equivalent to:
   *        min(timeDelta, max(0, previousTimeDelinquent - delinquencyGracePeriod))
   *
   * @param state Encoded state parameters
   * @param delinquencyGracePeriod Seconds in delinquency before penalties apply
   * @param timeDelta Seconds since the last update
   * @param `timeWithPenalty` Number of seconds since the last update where
   *        the market was in delinquency outside of the grace period.
   */
```
Above part of @dev When must be indented inner more to match with Calculate 


### NC-6 Arrays not checked for length;  if empty 

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L153
```
function authorizeLenders(address[] memory lenders) external onlyBorrower {
```

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L169
```
function deauthorizeLenders(address[] memory lenders) external onlyBorrower {
```

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L182
```
function updateLenderAuthorization(address lender, address[] memory markets) external {
```

The above lenders, and markets arrays are not checked if not empty by verifying that the length is not zero

**Recommendation** -> Ensure checks that lenders.length != 0 and markets.length != 0


### NC-7 Explicitly set library version in import statements 

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L4

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L4

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L4

For the above links set to @4.9.3 for OpenZeppelin contracts 
```
import { EnumerableSet } from 'openzeppelin/contracts@4.9.3/utils/structs/EnumerableSet.sol';
```

See RareSkills Solidity Style Guide Best Practises https://www.rareskills.io/post/solidity-style-guide

### NC-8 Apply Contract Level NatSpec 

Most QA Audits only focus on ensuring NatSpec is applied to mostly functions for @param, @notice @return and for events and modifiers. However contract itself should have rich NatSpec to provide initial inline guide and info about the contract 

See RareSkills Solidity Style Guide Best Practises https://www.rareskills.io/post/solidity-style-guide
Example is as below 
```
/// @title Controller for Wild Market 
/// @author XXXX
/// @notice Notes for non-technical readers/
/// @dev Notes for people who understand Solidity
contract WildcatMarketController is IWildcatMarketControllerEventsAndErrors {
  using EnumerableSet for EnumerableSet.AddressSet;
  using SafeCastLib for uint256;
  using SafeTransferLib for address;
```

### NC-9 Ensure no unnecessary virtual modifiers on functions that will not be overridden

Contracts have many virtual functions in them and it may be the case especially for external functions that these functions will not be overridden by child contracts, so the virtual modifier is not necessary.  

Carefully review all the following instances and correct where necessary 

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L142

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L402

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L108

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L118

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L249

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarket.sol#L86

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarket.sol#L44

See RareSkills Solidity Style Guide Best Practises https://www.rareskills.io/post/solidity-style-guide 

### NC-10 Consider time delay critical protocol/project parameter changes 

It is best when critical changes are made they are not actioned immediately so that users have time to decide to continue using protocol or prepare for the incoming changes 

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L195

for things like fees,etc consider all key overall control functionality carefully and review 

### NC-11 Unused modifier 

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L131C12-L131C24
```
 modifier onlyBorrower() {
    if (msg.sender != borrower) revert NotApprovedBorrower();
    _;
  }
```
Borrower modifier is not used anywhere in the WildcatMarketBase.sol code. Carefully review this and all other modifiers to see if there are any unused modifiers in code






