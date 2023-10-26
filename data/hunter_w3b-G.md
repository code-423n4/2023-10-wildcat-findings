# Gas Optimization For The Wildcat Protocol

While striving for enhanced code clarity in the provided snippets, certain functions have been abbreviated to emphasize affected sections.

Developers should remain vigilant during the incorporation of these proposed modifications to avert potential vulnerabilities. Despite prior testing of the optimizations, developers bear the responsibility of conducting comprehensive reevaluation.

Conducting code reviews and additional testing is highly recommended to mitigate any plausible hazards that may arise from the refactoring endeavor.

These optimizations aim to reduce the gas costs associated with contract deployment and execution. Below is a summary of the key points and issues discussed:

# Summary

| Number | Issues                                                                                                           | Instances |
| :----: | :--------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Use Clones for Cheap Contract Deployments                                                                        |     1     |
| [G-02] | Unnecessary casting as variable is already of the same type                                                      |     3     |
| [G-03] | onlyController or onlyBorrower no uses of the nonreentrance modifier waste of gas                                |     6     |
| [G-04] | Shorten the array rather than copying to a new one                                                               |     8     |
| [G-05] | Use assembly to validate msg.sender                                                                              |     7     |
| [G-06] | Invert if-else statements that have a negation                                                                   |    17     |
| [G-07] | Timestamps in storage do not need to be uint256                                                                  |     3     |
| [G-08] | Understand the trade-offs when choosing between internal functions and modifiers                                 |     9     |
| [G-09] | Use fallback or receive instead of deposit() when transferring Ether                                             |     1     |
| [G-10] | Consider using alternatives to OpenZeppelin                                                                      |     -     |
| [G-11] | Use SUB or XOR instead of ISZERO(EQ()) to check for inequality (more efficient in certain scenarios)             |    11     |
| [G-12] | Avoid contract existence checks by using low level calls                                                         |    29     |
| [G-13] | `<X> += <Y>` Costs more gas than `<X> = <X> + <Y>` for state variables                                           |     4     |
| [G-14] | Store using Struct over multiple mappings                                                                        |     3     |
| [G-15] | Use bitmaps instead of bools when a significant amount of booleans are used                                      |     1     |
| [G-16] | Internal functions not called by the contract should be removed to save deployment Gas                           |     5     |
| [G-17] | Structs can be packed into fewer storage slots by editing time variables                                         |     4     |
| [G-18] | Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it) |     2     |
| [G-19] | A modifier used only once and not being inherited should be inlined to save gas                                  |     4     |
| [G-20] | Using > 0 costs more gas than != 0 when used on a uint in a require() and assert statement                       |    12     |
| [G-21] | Do not calculate constants                                                                                       |     2     |
| [G-22] | Do-While loops are cheaper than for loops                                                                        |    12     |
| [G-23] | Refactor event to avoid emitting data that is already present in transaction data                                |     1     |
| [G-24] | Pre-increment and pre-decrement are cheaper than +1 ,-1                                                          |     3     |

### These optimization techniques and recommendations can help developers reduce gas costs and improve the efficiency of the Wildcat Protocol contracts. It's essential for developers to exercise caution and conduct thorough testing and code reviews when implementing these optimizations to ensure the security and correctness of their contracts.

## [G-01] Use Clones for Cheap Contract Deployments

Use clones or metaproxies when deploying very similar smart contracts that are not called frequently

When deploying multiple similar smart contracts, the gas costs can be high. To reduce these costs, you can use minimal clones or metaproxies which store the address of the implementation contract in their bytecode and interact with it as a proxy.

However, there is a trade-off between the runtime cost and deployment cost of clones. Clones are more expensive to interact with than normal contracts due to the delegatecall they use, so they should only be used when you don’t need to interact with them frequently. For example, the Gnosis Safe contract uses clones to reduce deployment costs.

```solidity
File: src/WildcatSanctionsSentinel.sol

  function createEscrow(
    address borrower,
    address account,
    address asset
  ) public override returns (address escrowContract) {
    if (!IWildcatArchController(archController).isRegisteredMarket(msg.sender)) {
      revert NotRegisteredMarket();
    }

    escrowContract = getEscrowAddress(borrower, account, asset);

    if (escrowContract.codehash != bytes32(0)) return escrowContract;

    tmpEscrowParams = TmpEscrowParams(borrower, account, asset);

    new WildcatSanctionsEscrow{ salt: keccak256(abi.encode(borrower, account, asset)) }();

    emit NewSanctionsEscrow(borrower, account, asset);

    sanctionOverrides[borrower][escrowContract] = true;

    emit SanctionOverride(borrower, escrowContract);

    _resetTmpEscrowParams();
  }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L95-L120

## [G-02] Unnecessary casting as variable is already of the same type

### MintingHub.sol.controllerFactory(): pos should not be cast to address as it’s declared as an address

```solidity
File: src/WildcatMarketController.sol

306    } else if (msg.sender != address(controllerFactory)) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L306

```solidity
File:  src/WildcatMarketControllerFactory.sol

252    parameters.archController = address(archController);

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L252

```solidity
File:  src/market/WildcatMarketWithdrawals.sol

169        address(asset)

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L169

## [G-03] onlyController or onlyBorrower no uses of the nonreentrance modifier waste of gas

### Becuse the this function is only called by onlyBorrower modifier

```solidity
File: src/market/WildcatMarket.sol

119   function borrow(uint256 amount) external onlyBorrower nonReentrant {

// @audit   this function is only called by onlyController
142  function closeMarket() external onlyController nonReentrant {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L119

```solidity
File: src/market/WildcatMarketConfig.sol

115  ) external onlyController nonReentrant {


134  function setMaxTotalSupply(uint256 _maxTotalSupply) external onlyController nonReentrant {

149  function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant {

171  function setReserveRatioBips(uint16 _reserveRatioBips) public onlyController nonReentrant {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L115

## [G-04] Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
File: src/libraries/FIFOQueue.sol

46    _values = new uint32[](len);

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L46

```solidity
File: src/WildcatArchController.sol

92    arr = new address[](count);

135    arr = new address[](count);

178    arr = new address[](count);

221    arr = new address[](count);

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L92

```solidity
File: src/WildcatMarketController.sol

132    arr = new address[](count);

211    arr = new address[](count);

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L132

```solidity
File: src/WildcatMarketControllerFactory.sol

145    arr = new address[](count);

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L145

## [G-05] Use assembly to validate msg.sender

Using assembly to validate msg.sender can indeed save gas and provide more efficient validation. You can compare the msg.sender directly to the expected sender address in assembly and revert if they don't match. Here's an example of how to do this in Solidity assembly:

```solidity
function validateSender(address expectedSender) internal pure {
    address sender;
    assembly {
        // Load the sender address from the appropriate location in the call data.
        sender := sload(0x00) // 0x00 contains the sender's address
    }
    require(sender == expectedSender, "Sender validation failed");
}
```

We declare an address sender variable.
In the assembly block, we use sload to load the sender's address from the appropriate location in the call data. This location is implementation-specific and can vary across different Solidity versions.
We then compare sender to the expectedSender provided as an argument to the function.
If the comparison fails, we revert with an error message.

```solidity
File: src/market/WildcatMarketBase.sol

132    if (msg.sender != borrower) revert NotApprovedBorrower();

137    if (msg.sender != controller) revert NotController();

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L132

```solidity
File:  src/WildcatMarketController.sol

81    if (msg.sender != borrower) {

302    if (msg.sender == borrower) {

306    } else if (msg.sender != address(controllerFactory)) {


```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L81

```solidity
File: src/WildcatMarketControllerFactory.sol

66    if (msg.sender != archController.owner()) {

283    if (!archController.isRegisteredBorrower(msg.sender)) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L66

## [G-06] Invert if-else statements that have a negation

This is the same example we gave at the beginning of the article. In the code snippet below, the second function avoids an unnecessary negation. In theory, the extra ! increases the computational cost. But as we noted at the top of the article, you should benchmark both methods because the compiler is can sometimes optimize this.

```solidity
    function cond() public {
        if (!condition) {
            action1();
        } else {
            action2();
        }
    }

    function cond() public {
        if (condition) {
            action2();
        } else {
            action1();
        }
    }
```

```solidity
File: src/WildcatArchController.sol

42    if (!_controllerFactories.contains(msg.sender)) {

49    if (!_controllers.contains(msg.sender)) {

64    if (!_borrowers.add(borrower)) {

71    if (!_borrowers.remove(borrower)) {

107    if (!_controllerFactories.add(factory)) {

114    if (!_controllerFactories.remove(factory)) {

150    if (!_controllers.add(controller)) {

157    if (!_controllers.remove(controller)) {

193    if (!_markets.add(market)) {

200    if (!_markets.remove(market)) {
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol

```solidity
File: src/market/WildcatMarketConfig.sol

75    if (!IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L75

```solidity
File: src/WildcatMarketController.sol

88    if (!_controlledMarkets.contains(market)) {

185      if (!_controlledMarkets.contains(market)) {

303      if (!archController.isRegisteredBorrower(msg.sender)) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L88

```solidity
File: src/WildcatMarketControllerFactory.sol

283    if (!archController.isRegisteredBorrower(msg.sender)) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L283

```solidity
File: src/WildcatSanctionsEscrow.sol

34    if (!canReleaseEscrow()) revert CanNotReleaseEscrow();

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L34

## [G-07] Timestamps in storage do not need to be uint256

A timestamp of size uint48 will work for millions of years into the future. A block number increments once every 12 seconds. This should give you a sense of the size of numbers that are sensible.

```solidity
File: src/libraries/FeeMath.sol

32    uint256 timestamp

55    uint256 timestamp,

147    uint256 timestamp
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L32

## [G-08] Understand the trade-offs when choosing between internal functions and modifiers

Modifiers inject its implementation bytecode where it is used while internal functions jump to the location in the runtime code where the its implementation is. This brings certain trade-offs to both options.

Using modifiers more than once means repetitiveness and increase in size of the runtime code but reduces gas cost because of the absence of jumping to the internal function execution offset and jumping back to continue. This means that if runtime gas cost matter most to you, then modifiers should be your choice but if deployment gas cost and/or reducing the size of the creation code is most important to you then using internal functions will be best.

However, modifiers have the tradeoff that they can only be executed at the start or end of a functon. This means executing it at the middle of a function wouldn’t be directly possible, at least not without internal functions which kill the original purpose. This affects it’s flexibility. Internal functions however can be called at any point in a function.

Example showing difference in gas cost using modifiers and an internal function

Example showing difference in gas cost using modifiers and an internal function

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.19;

//  deployment gas cost: 195435
//  gas per call:
//  restrictedAction1: 28367
//  restrictedAction2: 28377
//  restrictedAction3: 28411

contract Modifier {
address owner;
uint256 val;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    function restrictedAction1() external onlyOwner {
        val = 1;
    }

    function restrictedAction2() external onlyOwner {
        val = 2;
    }

    function restrictedAction3() external onlyOwner {
        val = 3;
    }

}

//  deployment gas cost: 159309
//  gas per call:
//  restrictedAction1: 28391
//  restrictedAction2: 28401
//  restrictedAction3: 28435

contract InternalFunction {
address owner;
uint256 val;

    constructor() {
        owner = msg.sender;
    }

    function onlyOwner() internal view {
        require(msg.sender == owner);
    }

    function restrictedAction1() external {
        onlyOwner();
        val = 1;
    }

    function restrictedAction2() external {
        onlyOwner();
        val = 2;
    }

    function restrictedAction3() external {
        onlyOwner();
        val = 3;
    }

}
```

| Operation          | Deployment | restrictedAction1 | restrictedAction2 | restrictedAction3 |
| ------------------ | ---------- | ----------------- | ----------------- | ----------------- |
| Modifiers          | 195,435    | 28,367            | 28,377            | 28,411            |
| Internal Functions | 159,309    | 28,391            | 28,401            | 28,435            |

From the table above, we can see that the contract that uses modifiers cost more than 35k gas more than the contract using internal functions when deploying it due to repetition of the onlyOwner functionality in 3 functions.

During runtime, we can see that each function using modifiers cost a fixed 24 gas less than the functions using internal functions.

```solidity
File: src/ReentrancyGuard.sol

31  modifier nonReentrant() {

41  modifier nonReentrantView() {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/ReentrancyGuard.sol#L31

```solidity
File: src/WildcatArchController.sol

41  modifier onlyControllerFactory() {

48  modifier onlyController() {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L41

```solidity
File: src/market/WildcatMarketBase.sol

131  modifier onlyBorrower() {

136  modifier onlyController() {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L131

```solidity
File: src/WildcatMarketController.sol

80  modifier onlyBorrower() {

87  modifier onlyControlledMarket(address market) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L80

```solidity
File: src/WildcatMarketControllerFactory.sol

65  modifier onlyArchControllerOwner() {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L65

## [G-09] Use fallback or receive instead of deposit() when transferring Ether

Similar to above, you can “just transfer” ether to a contract and have it respond to the transfer instead of using a payable function. This of course, depends on the rest of the contract’s architecture.

Example Deposit in AAVE

```solidity
contract AddLiquidity{

    receive() external payable {
      IWETH(weth).deposit{msg.value}();
      AAVE.deposit(weth, msg.value, msg.sender, REFERRAL_CODE)
    }

}
```

The fallback function is capable of receiving bytes data which can be parsed with abi.decode. This servers as an alternative to supplying arguments to a deposit function.

```solidity
File: src/market/WildcatMarket.sol

86  function deposit(uint256 amount) external virtual {
87     uint256 actualAmount = depositUpTo(amount);
88     if (amount != actualAmount) {
89       revert MaxSupplyExceeded();
90     }
91  }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L86-L91

## [G-10] Consider using alternatives to OpenZeppelin

OpenZeppelin is a great and popular smart contract library, but there are other alternatives that are worth considering. These alternatives offer better gas efficiency and have been tested and recommended by developers.

Two examples of such alternatives are Solmate and Solady.

Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly.

## [G-11] Use SUB or XOR instead of ISZERO(EQ()) to check for inequality (more efficient in certain scenarios)

When using inline assembly to compare the equality of two values (e.g if owner is the same as caller()), It is sometimes more efficient to do

```solidity
if sub(caller, sload(owner.slot)) {
revert(0x00, 0x00)
}
```

over doing this

```solidity
if eq(caller, sload(owner.slot)) {
revert(0x00, 0x00)
}
```

xor can accomplish the same thing, but be aware that xor will consider a value with all the bits flipped to be equal also, so make sure that this cannot become an attack vector.

This trick will depend on the compiler version used and the context of the code.

```solidity
File: src/libraries/LibStoredInitCode.sol

35      if iszero(initCodeStorage) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol

```solidity
File: src/libraries/MathUtils.sol

88      if iszero(or(iszero(b), iszero(gt(a, div(sub(not(0), HALF_BIP), b))))) {

108      if or(iszero(b), gt(a, div(sub(not(0), div(b, 2)), BIP))) {

126      if iszero(eq(div(b, BIP_RAY_RATIO), a)) {

141      if iszero(or(iszero(b), iszero(gt(a, div(sub(not(0), HALF_RAY), b))))) {

176      if iszero(mul(d, iszero(mul(y, gt(x, div(not(0), y)))))) {

194      if iszero(mul(d, iszero(mul(y, gt(x, div(not(0), y)))))) {

200      z := add(iszero(iszero(mod(mul(x, y), d))), div(mul(x, y), d))

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol

```solidity
File: src/libraries/SafeCastLib.sol

9      if iszero(didNotOverflow) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/SafeCastLib.sol

```solidity
File: src/libraries/StringQuery.sol

47    if or(iszero(status), iszero(or(isBytes32, eq(returndatasize(), 0x60)))) {

49      if iszero(status) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/StringQuery.sol

```solidity
File: src/WildcatMarketController.sol

404      if or(iszero(mload(namePrefix)), iszero(mload(symbolPrefix))) {
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol

## [G-12] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File: src/WildcatArchController.sol

64    if (!_borrowers.add(borrower)) {

71    if (!_borrowers.remove(borrower)) {

78    return _borrowers.contains(borrower);

107    if (!_controllerFactories.add(factory)) {

114    if (!_controllerFactories.remove(factory)) {

121    return _controllerFactories.contains(factory);

150    if (!_controllers.add(controller)) {

157    if (!_controllers.remove(controller)) {

164    return _controllers.contains(controller);

193    if (!_markets.add(market)) {

200    if (!_markets.remove(market)) {

207    return _markets.contains(market);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L64

```solidity
File: src/market/WildcatMarketBase.sol

172        address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(

204      if (IWildcatMarketController(controller).isAuthorizedLender(accountAddress)) {

239    return IERC20(asset).balanceOf(address(this));

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L172

```solidity
File: src/WildcatMarketController.sol

185      if (!_controlledMarkets.contains(market)) {

188      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));

197    return _controlledMarkets.contains(market);

303      if (!archController.isRegisteredBorrower(msg.sender)) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L88

```solidity
File: src/WildcatMarketControllerFactory.sol

127    return _deployedControllers.contains(controller);

329    market = IWildcatMarketController(controller).deployMarket(
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L127

```solidity
File: src/market/WildcatMarketWithdrawals.sol

164    if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {

166      address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L164

```solidity
File: WildcatSanctionsEscrow.sol

22    return IERC20(asset).balanceOf(address(this));

26    return !WildcatSanctionsSentinel(sentinel).isSanctioned(borrower, account);

38    IERC20(asset).transfer(account, amount);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L22

```solidity
File: src/WildcatSanctionsSentinel.sol

42      IChainalysisSanctionsList(chainalysisSanctionsList).isSanctioned(account);

100    if (!IWildcatArchController(archController).isRegisteredMarket(msg.sender)) {
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L42

## [G-13] `<X> += <Y>` Costs more gas than `<X> = <X> + <Y>` for state variables

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

```solidity
File: src/market/WildcatMarketBase.sol

178        _accounts[escrow].scaledBalance += scaledBalance;

323      apr += MathUtils.bipToRay(delinquencyFeeBips);

338      apr += delinquencyFeeBips;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L178

```solidity
File: src/market/WildcatMarketWithdrawals.sol

104    _withdrawalData.accountStatuses[expiry][msg.sender].scaledAmount += scaledAmount;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L104

## [G-14] Store using Struct over multiple mappings

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

```solidity
File: src/market/WildcatMarketToken.sol

13  mapping(address => mapping(address => uint256)) public allowance;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L13

```solidity
File:  src/WildcatSanctionsSentinel.sol

20  mapping(address borrower => mapping(address account => bool sanctionOverride))
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L20

```solidity
File: src/libraries/Withdrawal.sol

34  mapping(uint256 => mapping(address => AccountWithdrawalStatus)) accountStatuses;
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L34

## [G-15] Use bitmaps instead of bools when a significant amount of booleans are used

A common pattern, especially in airdrops, is to mark an address as “already used” when claiming the airdrop or NFT mint.

However, since it only takes one bit to store this information, and each slot is 256 bits, that means one can store a 256 flags/booleans with one storage slot.

```solidity
File: src/WildcatSanctionsSentinel.sol

20   mapping(address borrower => mapping(address account => bool sanctionOverride))
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L20

## [G-16] Internal functions not called by the contract should be removed to save deployment Gas

```solidity
File: src/market/WildcatMarketBase.sol

163  function _blockAccount(MarketState memory state, address accountAddress) internal {


197  function _getAccountWithRole(

358  function _getUpdatedState() internal returns (MarketState memory state) {

448  function _writeState(MarketState memory state) internal {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L163

```solidity
File:  src/libraries/Withdrawal.sol

47  function availableLiquidityForPendingBatch(

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L40

## [G-17] Structs can be packed into fewer storage slots by editing time variables

```solidity
File: src/libraries/MarketState.sol

13    struct MarketState {
14        bool isClosed;
15        uint128 maxTotalSupply;
16        uint128 accruedProtocolFees;
17        // Underlying assets reserved for withdrawals which have been paid
18        // by the borrower but not yet executed.
19        uint128 normalizedUnclaimedWithdrawals;
20        // Scaled token supply (divided by scaleFactor)
21        uint104 scaledTotalSupply;
22        // Scaled token amount in withdrawal batches that have not been
23        // paid by borrower yet.
24        uint104 scaledPendingWithdrawals;
25        uint32 pendingWithdrawalExpiry;
26        // Whether market is currently delinquent (liquidity under requirement)
27        bool isDelinquent;
28        // Seconds borrower has been delinquent
29        uint32 timeDelinquent;
30        // Annual interest rate accrued to lenders, in basis points
31        uint16 annualInterestBips;
32        // Percentage of outstanding balance that must be held in liquid reserves
33        uint16 reserveRatioBips;
34        // Ratio between internal balances and underlying token amounts
35        uint112 scaleFactor;
36        uint32 lastInterestAccruedTimestamp;
}
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L13-L36

```solidity
File: src/WildcatMarketController.sol

18    struct TmpMarketParameterStorage {
19        address asset;
20        string namePrefix;
21        string symbolPrefix;
22        address feeRecipient;
23        uint16 protocolFeeBips;
24        uint128 maxTotalSupply;
25        uint16 annualInterestBips;
26        uint16 delinquencyFeeBips;
27        uint32 withdrawalBatchDuration;
28        uint16 reserveRatioBips;
29        uint32 delinquencyGracePeriod;
}
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L18-L29

```solidity
File: src/libraries/Withdrawal.sol

17    struct WithdrawalBatch {
18        // Total scaled amount of tokens to be withdrawn
19        uint104 scaledTotalAmount;
20        // Amount of scaled tokens that have been paid by borrower
21        uint104 scaledAmountBurned;
22        // Amount of normalized tokens that have been paid by borrower
23        uint128 normalizedAmountPaid;
24        }


26    struct AccountWithdrawalStatus {
27        uint104 scaledAmount;
28        uint128 normalizedAmountWithdrawn;
29        }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L17

## [G-18] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity
File: src/WildcatMarketControllerFactory.sol

286    _tmpMarketBorrowerParameter = msg.sender;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L286

```solidity
File: src/WildcatSanctionsEscrow.sol

17    sentinel = msg.sender;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L17

## [G-19] A modifier used only once and not being inherited should be inlined to save gas

```solidity
File: src/WildcatArchController.sol

//@audit  these modifier is only used once in functions  it should be inlined to save gas

41  modifier onlyControllerFactory() {
42    if (!_controllerFactories.contains(msg.sender)) {
43      revert NotControllerFactory();
44    }
45    _;
46  }


48 modifier onlyController() {
49    if (!_controllers.contains(msg.sender)) {
50      revert NotController();
51    }
52    _;
53  }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L41

```solidity
File: src/WildcatMarketController.sol

87  modifier onlyControlledMarket(address market) {
88    if (!_controlledMarkets.contains(market)) {
89      revert NotControlledMarket();
90    }
91    _;
92  }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L87-L92

```solidity
File: src/WildcatMarketControllerFactory.sol

65  modifier onlyArchControllerOwner() {
66    if (msg.sender != archController.owner()) {
67      revert CallerNotArchControllerOwner();
68    }
69    _;
70  }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L65-70

## [G-20] Using > 0 costs more gas than != 0 when used on a uint in a require() and assert statement

```solidity
File: src/libraries/FeeMath.sol

67    if (timeWithPenalty > 0) {

155    if (protocolFeeBips > 0) {

159    if (delinquencyFeeBips > 0) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L67

```solidity
File: src/market/WildcatMarket.sol

147    if (_withdrawalData.unpaidBatches.length() > 0) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L147

```solidity
File: src/market/WildcatMarketBase.sol

79    if ((parameters.protocolFeeBips > 0).and(parameters.feeRecipient == address(0))) {

170      if (scaledBalance > 0) {

428      if (availableLiquidity > 0) {

472    if (availableLiquidity > 0) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L79

```solidity
File: src/WildcatMarketControllerFactory.sol

201    bool hasOriginationFee = originationFeeAmount > 0;

205      (protocolFeeBips > 0 && nullFeeRecipient) ||

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L201

```solidity
File: src/market/WildcatMarketWithdrawals.sol

32    if ((expiry == expiredBatchExpiry).and(expiry > 0)) {

112    if (availableLiquidity > 0) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L32

## [G-21] Do not calculate constants

```solidity
File: src/libraries/Chainalysis.sol


6    IChainalysisSanctionsList constant SanctionsList = IChainalysisSanctionsList(
7      0x40C57923924B5c5c5455c48D93317139ADDaC8fb
8    );
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Chainalysis.sol#L6-L8

```solidity
File: src/WildcatSanctionsSentinel.sol

11  bytes32 public constant override WildcatSanctionsEscrowInitcodeHash =
12        keccak256(type(WildcatSanctionsEscrow).creationCode);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L11

## [G-22] Do-While loops are cheaper than for loops

If you want to push optimization at the expense of creating slightly unconventional code, Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

// times == 10 in both tests
contract Loop1 {
    function loop(uint256 times) public pure {
        for (uint256 i; i < times;) {
            unchecked {
                ++i;
            }
        }
    }
}

contract Loop2 {
    function loop(uint256 times) public pure {
        if (times == 0) {
            return;
        }

        uint256 i;

        do {
            unchecked {
                ++i;
            }
        } while (i < times);
    }
}
```

```solidity
File: src/libraries/FIFOQueue.sol

48    for (uint256 i = 0; i < len; i++) {

75    for (uint256 i = 0; i < n; i++) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48

```solidity
File: src/WildcatArchController.sol

93    for (uint256 i = 0; i < count; i++) {

136    for (uint256 i = 0; i < count; i++) {

179    for (uint256 i = 0; i < count; i++) {

222    for (uint256 i = 0; i < count; i++) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93

```solidity
File: src/WildcatMarketController.sol

133    for (uint256 i = 0; i < count; i++) {

154    for (uint256 i = 0; i < lenders.length; i++) {

170    for (uint256 i = 0; i < lenders.length; i++) {

183    for (uint256 i; i < markets.length; i++) {

212    for (uint256 i = 0; i < count; i++) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133

```solidity
File: src/WildcatMarketControllerFactory.sol

146    for (uint256 i = 0; i < count; i++) {

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L146

## [G-23] Refactor event to avoid emitting data that is already present in transaction data

```solidity
File: src/market/WildcatMarket.sol

160    emit MarketClosed(block.timestamp);

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L160

## [G-24] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
File: src/libraries/FIFOQueue.sol

58    arr.nextIndex = nextIndex + 1;

67    arr.startIndex = startIndex + 1;

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L58
