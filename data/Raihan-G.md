# gas 

# summary

|      |  issue  | instance  |
|------|---------|-----------|
|[G-01]|Structs can be packed to use fewer storage slots|2|
|[G-02]|Avoid contract existence checks by using low level calls|16|
|[G-03]|Can Make The Variable Outside The Loop To Save Gas|3|
|[G-04]|Use assembly to validate msg.sender|2|
|[G-05]|Use constants instead of type(uintx).max|1|
|[G-06]|Using assembly to revert with an error message|21|
|[G-07]|Cache external calls outside of loop to avoid re-calling function on each iteration|1|
|[G-08]|Using Fixed Bytes Is Cheaper Than Using String|2|
|[G-09]|Use hardcode address instead address(this)|3|
|[G-10]|Amounts should be checked for 0 before calling a transfer|2|
|[G-11]|Split revert statements|5|
|[G-12]|Make 3 event parameters indexed when possible|10|
|[G-13]|Expressions for constant values such as a call to keccak256(), should use immutable rather than constant|1|
|[G-14]|Refactor a modifier to call a local function instead of directly having the code in the modifier, saving bytecode size and thereby deployment cost|7|
|[G-15]|Write gas-optimal for-loops|~|
|[G-16]|Use assembly for math (add, sub, mul, div)|6|
|[G-17]|Refector function to save gas|4|
|[G-18]|Don’t make variables public unless it is necessary to do so|~|


## [G-01] Structs can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs) we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

There are 2 instances of this issue:
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

## [G‑02] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

There are 16 instances of this issue:

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


```solidity
File:  src/market/WildcatMarketConfig.sol
75   if (!IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {

94   if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {    
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L75


```solidity
File:  src/market/WildcatMarketWithdrawals.sol
164    if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {

166    address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(    
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L164

```solidity
File:  src/WildcatSanctionsSentinel.sol
42  IChainalysisSanctionsList(chainalysisSanctionsList).isSanctioned(account);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L42

```solidity
File:  src/WildcatMarketController.sol
188   WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));

447   if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {

478   tmp.reserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips());   

481   WildcatMarket(market).setReserveRatioBips(9000);

487   WildcatMarket(market).setAnnualInterestBips(annualInterestBips);

499   WildcatMarket(market).setReserveRatioBips(uint256(tmp.reserveRatioBips).toUint16());
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L188

## [G-03] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

There are 3 instances:

```solidity
File:  src/WildcatMarketController.sol
155   address lender = lenders[i];

171  address lender = lenders[i];

184  address market = markets[i];
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L155

## [G-04] Use assembly to validate msg.sender
We can use assembly to efficiently validate msg.sender for the deployMarket function with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

There are 1 instances of this issue:
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

`Fix code`
```solidity
assembly {
    // Load msg.sender into memory
    let sender := caller()

    // Compare msg.sender with borrower
    if eq(sender, borrower) {
      // Check if borrower is a registered borrower
      let isRegistered := iszero(call(gas(), borrower, 0, 0, 0, 0, 0))
      if isRegistered {
        // Throw an exception if borrower is not registered
        revert(0, 0)
      }
    } else {
      // Compare msg.sender with controllerFactory
      if iszero(eq(sender, controllerFactory)) {
        // Throw an exception if caller is neither borrower nor controllerFactory
        revert(0, 0)
      }
    }
  }

```

## [G-05] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
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


There are 1 instances of this issue:
```solidity
File:  src/market/WildcatMarketToken.sol
49   if (allowed != type(uint256).max) {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L49


## [G-06] Using assembly to revert with an error message
When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.


Here’s an example:
```
/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```
From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.


There are 21 instances of this issue:

```solidity
File:  src/market/WildcatMarketToken.sol
68  if (scaledAmount == 0) {
      revert NullTransferAmount();
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L68


```solidity
File:  src/market/WildcatMarketWithdrawals.sol
84  if (scaledAmount == 0) {
      revert NullBurnAmount();
    }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L84

```solidity
File:  src/ReentrancyGuard.sol
83  if (_reentrancyGuard != _NOT_ENTERED) {
      revert NoReentrantCalls();
    }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/ReentrancyGuard.sol#L83


```solidity
File:  src/WildcatMarketControllerFactory.sol
283 if (!archController.isRegisteredBorrower(msg.sender)) {
      revert NotRegisteredBorrower();
    }

294 if (controller.codehash != bytes32(0)) {
      revert ControllerAlreadyDeployed();
    }    
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L283


```solidity
File:  src/WildcatSanctionsEscrow.sol
34    if (!canReleaseEscrow()) revert CanNotReleaseEscrow();
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L34


```solidity
File: src/WildcatSanctionsSentinel.sol
100 if (!IWildcatArchController(archController).isRegisteredMarket(msg.sender)) {
      revert NotRegisteredMarket();
    }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L100


```solidity
File:  src/libraries/FIFOQueue.sol
24  if (arr.startIndex == arr.nextIndex) {
      revert FIFOQueueOutOfBounds();
    }

32  if (index >= arr.nextIndex) {
      revert FIFOQueueOutOfBounds();
    }  

63  if (startIndex == arr.nextIndex) {
      revert FIFOQueueOutOfBounds();
    }    

72  if (startIndex + n > arr.nextIndex) {
      revert FIFOQueueOutOfBounds();
    }      
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L24


```solidity
File:  src/market/WildcatMarket.sol
48  if (state.isClosed) {
      revert DepositToClosedMarket();
    }

57  if (scaledAmount == 0) revert NullMintAmount();

88  if (amount != actualAmount) {
      revert MaxSupplyExceeded();
    }

98  if (state.accruedProtocolFees == 0) {
      revert NullFeeAmount();
    }  

102 if (withdrawableFees == 0) {
      revert InsufficientReservesForFeeWithdrawal();
    }    

125 if (amount > borrowable) {
      revert BorrowAmountTooHigh();
    }      
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L48


```solidity
File:  src/market/WildcatMarketBase.sol
    if (parameters.annualInterestBips > BIP) {
      revert InterestRateTooHigh();
    }
    if (parameters.reserveRatioBips > BIP) {
      revert ReserveRatioBipsTooHigh();
    }
    if (parameters.protocolFeeBips > BIP) {
      revert InterestFeeTooHigh();
    }
    if (parameters.delinquencyFeeBips > BIP) {
      revert PenaltyFeeTooHigh();
    }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L82-L93


## [G-07] Cache external calls outside of loop to avoid re-calling function on each iteration
Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

There are 1 instances of this issue:
```solidity
File:  src/WildcatMarketController.sol
// @audit WildcatMarket is external
188  WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L188


## [G-08] Using Fixed Bytes Is Cheaper Than Using String
As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

There are 2 instances of this issue:
```solidity
File:  src/market/WildcatMarketBase.sol
57  string public name;

60  string public symbol;
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L57-L60


## [G-09] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    #L
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)


There are 3 instances of this issue:

```solidity
File:  src/market/WildcatMarketBase.sol
175   address(this)

239   return IERC20(asset).balanceOf(address(this));

524    emit Transfer(address(this), address(0), normalizedAmountPaid);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L175


## [G-10] Amounts should be checked for 0 before calling a transfer
Checking non-zero transfer values can avoid an expensive external call and save gas.

There are 2 instances of this issue:
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


## [G-11] Split revert statements
Similar to splitting require statements, you will usually save some gas by not having a boolean operator in the if statement.

There are 2 instances of this issue:
```
contract CustomErrorBoolLessEfficient {
    error BadValue();

    function requireGood(uint256 x) external pure {
        if (x < 10 || x > 20) {
            revert BadValue();
        }
    }
}

contract CustomErrorBoolEfficient {
    error TooLow();
    error TooHigh();

    function requireGood(uint256 x) external pure {
        if (x < 10) {
            revert TooLow();
        }
        if (x > 20) {
            revert TooHigh();
        }
    }
}
```

```solidity
File:  src/WildcatMarketControllerFactory.sol
79  if (
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




204 if (
      (protocolFeeBips > 0 && nullFeeRecipient) ||
      (hasOriginationFee && nullFeeRecipient) ||
      (hasOriginationFee && nullOriginationFeeAsset)
    ) {
      revert InvalidProtocolFeeConfiguration();
    }    
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L79-L90


##  [G-12] Make 3 event parameters indexed when possible
It is the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

There are 10 instances of this issue:
```solidity
File:  src/WildcatArchController.sol
  event MarketAdded(address indexed controller, address market);
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
  event NewController(address borrower, address controller, string namePrefix, string symbolPrefix);
  event UpdateProtocolFeeConfiguration(
    address feeRecipient,
    uint16 protocolFeeBips,
    address originationFeeAsset,
    uint256 originationFeeAmount
  );
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L16-L22

## [G-13] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

There are 1 instances of this issue:
```solidity
File:  src/WildcatSanctionsSentinel.sol
11    bytes32 public constant override WildcatSanctionsEscrowInitcodeHash =
    keccak256(type(WildcatSanctionsEscrow).creationCode);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L11-L12


## [G-14] Refactor a modifier to call a local function instead of directly having the code in the modifier, saving bytecode size and thereby deployment cost [Ref](https://0xmacro.com/blog/solidity-gas-optimizations-cheat-sheet/#:~:text=Refactor%20a%20modifier%20to%20call%20a%20local%20function%20instead%20of%20directly%20having%20the%20code%20in%20the%20modifier%2C%20saving%20bytecode%20size%20and%20thereby%20deployment%20cost)
Modifiers code is copied in all instances where it's used, increasing bytecode size. By doing a refractor to the internal function, one can reduce bytecode size significantly at the cost of one JUMP. Consider doing this only if you are constrained by bytecode size.

`Before:`
```
modifier onlyOwner() {
                require(owner() == msg.sender, "Ownable: caller is not the owner");
                _;
}
```

`After:`

```
modifier onlyOwner() {
                _checkOwner();
                _;
}

function _checkOwner() internal view virtual {
   require(owner() == msg.sender, "Ownable: caller is not the owner");
}
```


```solidity
File:  src/WildcatArchController.sol
41   modifier onlyControllerFactory() {

48   modifier onlyController() {  
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L41


```solidity
File:  src/WildcatMarketController.sol
80     modifier onlyBorrower() {

87     modifier onlyControlledMarket(address market) {  
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L80


```solidity
File:  src/WildcatMarketControllerFactory.sol
65     modifier onlyArchControllerOwner() {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L65


```solidity
File:  src/market/WildcatMarketBase.sol
131  modifier onlyBorrower() {

136  modifier onlyController() {  
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L131


## [G-15] Write gas-optimal for-loops
This is what a gas-optimal for loop looks like, if you combine the two tricks above:

for (uint256 i; i < limit; ) {
    
    // inside the loop
    
    unchecked {
        ++i;
    }
}

The two differences here from a conventional for loop is that i++ becomes ++i (as noted above), and it is unchecked because the limit variable ensures it won’t overflow.

## [G-16] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

Addition:
```
//addition in SolidityCache multiple accesses of a mapping/array
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```
Gas: 303
```
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
```
//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
  uint256 c = a - b;
}
```
Gas: 300
```
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
```
//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325
```
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
```
//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325

```
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


```solidity
File:  src/WildcatArchController.sol
// @audit 134 177 20 also these line
91  uint256 count = end - start;
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L91


```solidity
File:  src/libraries/FeeMath.sol
24   uint256 accumulatedInterestRay = rate * timeDelta;
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L24


```solidity
File:  src/libraries/MathUtils.sol
35    uint256 accumulatedInterestRay = rate * timeDelta;
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L35


## [G-17] Refector function to save gas

```solidity
File: src/WildcatArchController.sol
85  function getRegisteredBorrowers(
    uint256 start,
    uint256 end
  ) external view returns (address[] memory arr) {
    uint256 len = _borrowers.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L85-L96

`Fix code`
```diff
  function getRegisteredBorrowers(
    uint256 start,
    uint256 end
  ) external view returns (address[] memory arr) {
+   EnumerableSet.AddressSet memory borrowers = _borrowers;
-   uint256 len = _borrowers.length();
+   uint256 len = borrowers.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
-     arr[i] = _borrowers.at(start + i);      
+     arr[i] = borrowers.at(start + i);
    }
  }

```

note : also in this function same issue

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L128-L140

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L171-L182

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L214-L225


## [G-18] Don’t make variables public unless it is necessary to do so
A public storage variable has an implicit public function of the same name. A public function increases the size of the jump table and adds bytecode to read the variable in question. That makes the contract larger.

Remember, private variables aren’t private, it’s not difficult to extract the variable value using web3.js.

This is especially true for constants which are meant to be read by humans rather than smart contracts.

