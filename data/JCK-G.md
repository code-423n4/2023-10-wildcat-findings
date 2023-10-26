

## Gas Optimizations

| Number | Issue | Instances | Total gas saved |
|--------|-------|-----------|-----------------|
|[G-01]| Return values from external calls can be cached to avoid unnecessary call |  4  |  |
|[G-02]| Pack structs by putting data types that can fit together next to each other | 2  | 40000 |
|[G-03]| Optimize check order: avoid making any state reads/writes if we have to validate some function parameters |  4  |  |
|[G-04]| Don’t cache calls that are only used once |  12  |  |
|[G-05]| Access mappings directly rather than using accessor functions |  4  |  |
|[G-06]| public functions not called by the contract should be declared external instead |  5  |  |
|[G-07]| Using private for immutable to saves gas |  27  | 506898 |
|[G-08]| Amounts should be checked for 0 before calling a transfer |  4  |  |
|[G-09]| Move storage pointer to top of function to avoid offset calculation |  2  |  |
|[G-10]| Cache storage values in memory to minimize SLOADs |  2  | 1100 |
|[G-11]| Using bools for storage incurs overhead |  2  | 4000 |
|[G-12]| Do not initialize variables with default values | 19   | 57 |
|[G-13]| Use hardcoded address instead of address(this) |   11  |  |
|[G-14]| Use uint256(1)/uint256(2) instead for true and false boolean states |  15  | 256500 |
|[G-15]| Use assembly to validate msg.sender |  6  | 18 |
|[G-16]| A modifier used only once and not being inherited should be inlined to save gas |  8  |  |
|[G-17]| Use assembly for loops |  9  |  |
|[G-18]| Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it) |  2  |  |
|[G-19]| Can make the variable outside the loop to save gas | 3  | 2193 |
|[G-20]| Use the existing memory value |  4  |  |
|[G-21]| The creation of an intermediary array can be avoided |  4  |  |
|[G-22]| Use != 0 instead of > 0 for unsigned integer comparison | 12   | 36 |
|[G-23]| Loop best practice to save gas |  12  |  |
|[G-24]| Internal functions that are not called by the contract should be removed to save deployment gas |  71  |  |
|[G-25]| Counting down in for statements is more gas efficient |  12  | 146052 |
|[G-26]| Use do while loops instead of for loops |  9  |  |
|[G-27]| storage variable can be replaced with calldata variable |  8  |  |
|[G-28]| Conditional check can be performed during variable initialization |  1  |  |
|[G-29]| Cache external calls outside of loop to avoid re-calling function on each iteration |  1  |  |
|[G-30]| Duplicated require()/revert() Checks Should Be Refactored To A Modifier Or Function |  7  | 196 |
|[G-31]| Combine events to save 2 Glogtopic (375 gas) |  2  | 750 |
|[G-32]| Failure to check the zero address in the constructor causes the contract to be deployed again |  2  |  |
|[G-33]| bytes constants are more eficient than string constans |  3  | 75993 |
|[G-34]| Use previousTimeDelinquent cached value in if stament instead of using state.timeDelinquent  |  1  |  |
|[G-35]| Possible Optimization in WildcatMarketBase.sol |  1  |  |
|[G-36]| Possible Optimization in WildcatMarket.sol |  1  |  |
|[G-37]| Possible Optimization in FeeMath.sol |  1  |  |
|[G-38]| Possible Optimization in WildcatMarketConfig.sol |  2  |  |

## [G-01] Return values from external calls can be cached to avoid unnecessary call

External calls are expensive as they use the STATICCALL/CALL opcode (~100 gas). If you are calling the same external function more than once you should cache the return value to avoid an unnecessary STATICCALL/CALL.

### cache WildcatMarket(market) external call 

```solidity
file:  src/WildcatMarketController.sol

468     function setAnnualInterestBips(
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
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L468-L488

### cache state.normalizeAmount(scaledBalance) external call 

```solidity
file:

163     function _blockAccount(MarketState memory state, address accountAddress) internal {
    Account memory account = _accounts[accountAddress];
    if (account.approval != AuthRole.Blocked) {
      uint104 scaledBalance = account.scaledBalance;
      account.approval = AuthRole.Blocked;
      emit AuthorizationStatusUpdated(accountAddress, AuthRole.Blocked);

      if (scaledBalance > 0) {
        account.scaledBalance = 0;
        address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
          accountAddress,
          borrower,
          address(this)
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
  }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L163-L187

### cache state.liquidityRequired() external call 

```solidity
file: src/market/WildcatMarketConfig.sol

171    function setReserveRatioBips(uint16 _reserveRatioBips) public onlyController nonReentrant {
    if (_reserveRatioBips > BIP) {
      revert ReserveRatioBipsTooHigh();
    }

    MarketState memory state = _getUpdatedState();

    uint256 initialReserveRatioBips = state.reserveRatioBips;

    if (_reserveRatioBips < initialReserveRatioBips) {
      if (state.liquidityRequired() > totalAssets()) {
        revert InsufficientReservesForOldLiquidityRatio();
      }
    }
    state.reserveRatioBips = _reserveRatioBips;
    if (_reserveRatioBips > initialReserveRatioBips) {
      if (state.liquidityRequired() > totalAssets()) {
        revert InsufficientReservesForNewLiquidityRatio();
      }
    }
    _writeState(state);
    emit ReserveRatioBipsUpdated(_reserveRatioBips);
  }
}

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L171-L194

### cache state.liquidityRequired() external call outside the if statment 

```solidity
file:  src/market/WildcatMarketWithdrawals.sol

164     if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
      _blockAccount(state, accountAddress);
      address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
        accountAddress,
        borrower,
        address(asset)
      );

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L164-L170

## [G-02] Pack structs by putting data types that can fit together next to each other

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot. This saves gas when writing to storage ~20000 gas

```solidity
file: src/WildcatMarketController.sol

18     struct TmpMarketParameterStorage {
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
file:  src/libraries/MarketState.sol

13   struct MarketState {
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

## [G-03] Optimize check order: avoid making any state reads/writes if we have to validate some function parameters


```solidity
file:  src/market/WildcatMarketConfig.sol

152    if (_annualInterestBips > BIP) {
      revert InterestRateTooHigh();
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L152-L154

```solidity
file:  src/libraries/FeeMath.sol

159   if (delinquencyFeeBips > 0) {
      delinquencyFeeRay = state.updateDelinquency(
        timestamp,
        delinquencyFeeBips,
        delinquencyGracePeriod
      );
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L159-L165


```solidity
file: src/market/WildcatMarket.sol

125   if (amount > borrowable) {
      revert BorrowAmountTooHigh();
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L125-L127


```solidity
file:  src/market/WildcatMarketConfig.sol

94     if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
      revert NotReversedOrStunning();
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L94-L96

## [G-04] Don’t cache calls that are only used once

```solidity
file: src/libraries/FeeMath.sol

23    uint256 rate = rateBip.bipToRay();

46    uint256 protocolFeeRay = protocolFeeBips.bipMul(baseInterestRay);

104   uint256 secondsRemainingWithoutPenalty = delinquencyGracePeriod.satSub(
        previousTimeDelinquent
      );

119   uint256 secondsRemainingWithPenalty = previousTimeDelinquent.satSub(delinquencyGracePeriod);

169   uint256 scaleFactorDelta = prevScaleFactor.rayMul(baseInterestRay + delinquencyFeeRay);

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L23


```solidity
file: src/libraries/MathUtils.sol

34    uint256 rate = rateBip.bipToRay();

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L34


```solidity
file: src/libraries/Withdrawal.sol

54    uint256 priorScaledAmountPending = (state.scaledPendingWithdrawals - batch.scaledOwedAmount());

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L54

```solidity
file: src/market/WildcatMarket.sol

87    uint256 actualAmount = depositUpTo(amount);
 
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L87


```solidity
file: src/market/WildcatMarketBase.sol

533  uint104 scaledAvailableLiquidity = state.scaleAmount(availableLiquidity).toUint104();

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L533

```solidity
file: src/WildcatArchController.sol

89   uint256 len = _borrowers.length();

132  uint256 len = _controllerFactories.length();

175  uint256 len = _controllers.length();

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L89

## [G-05] Access mappings directly rather than using accessor functions

When you have a mapping, accessing its values through accessor functions involves an additional layer of indirection, which can incur some gas cost. This is because accessing a value from a mapping typically involves two steps: first, locating the key in the mapping, and second, retrieving the corresponding value.

```solidity
file: src/market/WildcatMarketBase.sol
 
293  return _accounts[account].scaledBalance;

300  return _accounts[account].approval;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L293


```solidity
file: src/market/WildcatMarketWithdrawals.sol

35   return _withdrawalData.batches[expiry];

42   return _withdrawalData.accountStatuses[expiry][accountAddress];
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L35

## [G-06] public functions not called by the contract should be declared external instead

when a function is declared as public, it is generated with an internal and an external interface. This means the function can be called both internally (within the contract) and externally (by other contracts or accounts). However, if a public function is never called internally and is only expected to be invoked externally, it is more gas-efficient to explicitly declare it as external.

```solidity
file: src/WildcatSanctionsEscrow.sol

29   function escrowedAsset() public view override returns (address, uint256) {

33   function releaseEscrow() public override {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L29


```solidity
file: src/WildcatSanctionsSentinel.sol

95    function createEscrow(
    address borrower,
    address account,
    address asset
  ) public override returns (address escrowContract) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L95-L99


```solidity
file: src/market/WildcatMarketConfig.sol

149   function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant {

171   function setReserveRatioBips(uint16 _reserveRatioBips) public onlyController nonReentrant {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L149


## [G-07] Using private for immutable to saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

### Using public for immutable variable gas value: 704034
### Using private for immutable variable gas value: 685260

### Tools Used  Remix

```solidity
file: src/WildcatMarketController.sol

41   IWildcatArchController public immutable archController;

43   IWildcatMarketControllerFactory public immutable controllerFactory;

45   address public immutable borrower;

47   address public immutable sentinel;

49   address public immutable marketInitCodeStorage;

51   uint256 public immutable marketInitCodeHash;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L41

```solidity
file: src/WildcatMarketControllerFactory.sol

31   IWildcatArchController public immutable archController;

34   address public immutable sentinel;

36   address public immutable marketInitCodeStorage;

38   uint256 public immutable marketInitCodeHash;

40   address public immutable controllerInitCodeStorage;

42   uint256 public immutable controllerInitCodeHash;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L31


```solidity
file: src/WildcatSanctionsEscrow.sol

11   address public immutable override sentinel;

12   address public immutable override borrower;

13   address public immutable override account;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L11


```solidity
file:  src/WildcatSanctionsSentinel.sol

14   address public immutable override chainalysisSanctionsList;

16   address public immutable override archController;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L14

```solidity
file: src/market/WildcatMarketBase.sol

27    address public immutable sentinel;

30    address public immutable borrower;

33    address public immutable feeRecipient; 

36    uint256 public immutable protocolFeeBips;

39    uint256 public immutable delinquencyFeeBips; 

42    uint256 public immutable delinquencyGracePeriod;'

45    address public immutable controller;

48    address public immutable asset;

51    uint256 public immutable withdrawalBatchDuration;

54    uint8 public immutable decimals;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L27


## [G-08] Amounts should be checked for 0 before calling a transfer

It can be beneficial to check if an amount is zero before invoking a transfer function, such as transfer or safeTransfer, to avoid unnecessary gas costs associated with executing the transfer operation. If the amount is zero, the transfer operation would have no effect, and performing the check can prevent unnecessary gas consumption.

```solidity
file: src/WildcatSanctionsEscrow.sol

38    IERC20(asset).transfer(account, amount);

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L38


```solidity
file: src/market/WildcatMarket.sol

60   asset.safeTransferFrom(msg.sender, address(this), amount);

129  asset.safeTransfer(msg.sender, amount);

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L60


```solidity
file: src/market/WildcatMarketToken.sol

54   _transfer(from, to, amount);

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L54

## [G-09] Move storage pointer to top of function to avoid offset calculation

We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.


```solidity
file:  src/WildcatMarketController.sol

475    TemporaryReserveRatio storage tmp = temporaryExcessReserveRatio[market];


```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L475

```solidity
file: src/market/WildcatMarketWithdrawals.sol

147   AccountWithdrawalStatus storage status = _withdrawalData.accountStatuses[expiry][
      accountAddress
    ];


```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L147-L149

## [G-10] Cache storage values in memory to minimize SLOADs

The code can be optimized by minimizing the number of SLOADs.

### setAnnualInterestBips(): tmp should be cached (Save 3 SLOAD: 600 gas)

```solidity
file: src/WildcatMarketController.sol

474       if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {
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
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L474-L488

### first(): arr should be cached (Save 3 SLOAD: 500 gas)

```solidity
file: src/libraries/FIFOQueue.sol

23    function first(FIFOQueue storage arr) internal view returns (uint32) {
    if (arr.startIndex == arr.nextIndex) {
      revert FIFOQueueOutOfBounds();
    }
    return arr.data[arr.startIndex];
  }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L23-L28

## [G-11] Using bools for storage incurs overhead

// Booleans are more expensive than uint256 or any type that takes up a full
// word because each write operation emits an extra SLOAD to first read the
// slot's contents, replace the bits taken up by the boolean, and then write
// back. This is the compiler's defense against contract upgrades and
// pointer aliasing, and it cannot be disabled.


```solidity
file: src/libraries/MarketState.sol

14   bool isClosed;

27   bool isDelinquent;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L14

## [G-12] Do not initialize variables with default values

```solidity
file: src/WildcatSanctionsSentinel.sol

49   sanctionOverrides[msg.sender][account] = true;

57    sanctionOverrides[msg.sender][account] = false;

114  sanctionOverrides[borrower][escrowContract] = true;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L49

```solidity
file: src/WildcatArchController.sol

93    for (uint256 i = 0; i < count; i++) {'

136   for (uint256 i = 0; i < count; i++) {

179   for (uint256 i = 0; i < count; i++) {

222   for (uint256 i = 0; i < count; i++) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93


```solidity
file: src/WildcatMarketController.sol

133   for (uint256 i = 0; i < count; i++) {
    
154   for (uint256 i = 0; i < lenders.length; i++) {

170   for (uint256 i = 0; i < lenders.length; i++) {

212   for (uint256 i = 0; i < count; i++) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133


```solidity
file: src/libraries/FIFOQueue.sol

48    for (uint256 i = 0; i < len; i++) {

75    for (uint256 i = 0; i < n; i++) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48


```solidity
file: src/market/WildcatMarket.sol

144   state.annualInterestBips = 0;'

145   state.isClosed = true;'

146   state.reserveRatioBips = 0;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L144

```solidity
file: src/market/WildcatMarketBase.sol

171   account.scaledBalance = 0;

431   state.pendingWithdrawalExpiry = 0;

489   state.pendingWithdrawalExpiry = 0;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L171

## [G-13] Use hardcoded address instead of address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. Refrences

```solidity
file: src/market/WildcatMarket.sol

60   asset.safeTransferFrom(msg.sender, address(this), amount);

154  asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L60

```solidity
file:  src/WildcatMarketController.sol
 
53     uint256 internal immutable ownCreate2Prefix = LibStoredInitCode.getCreate2Prefix(address(this));

243    parameters.controller = address(this);

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L53


```solidity
file: src/WildcatMarketControllerFactory.sol

44   uint256 internal immutable ownCreate2Prefix = LibStoredInitCode.getCreate2Prefix(address(this));

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L44


```solidity
file: src/WildcatSanctionsEscrow.sol

22   return IERC20(asset).balanceOf(address(this));

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L22


```solidity
file:  src/WildcatSanctionsSentinel.sol

77      address(this),

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L77


```solidity
file: src/market/WildcatMarketBase.sol

175    address(this)

239   return IERC20(asset).balanceOf(address(this));

524   emit Transfer(address(this), address(0), normalizedAmountPaid);

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L175


```solidity
file: src/market/WildcatMarketWithdrawals.sol

91    emit Transfer(msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L91


## [G-14] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. see source:


```solidity
file: src/WildcatSanctionsSentinel.sol

49   sanctionOverrides[msg.sender][account] = true;

57   sanctionOverrides[msg.sender][account] = false;

114  sanctionOverrides[borrower][escrowContract] = true;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L49


```solidity
file: src/market/WildcatMarket.sol

145   state.isClosed = true;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L145

```solidity
file: src/WildcatMarketControllerFactory.sol

201    bool hasOriginationFee = originationFeeAmount > 0;

202    bool nullFeeRecipient = feeRecipient == address(0);

203    bool nullOriginationFeeAsset = originationFeeAsset == address(0);

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#l201

```solidity
file:   src/market/WildcatMarketBase.sol

449     bool isDelinquent = state.liquidityRequired() > totalAssets();

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L449

```solidity
file: src/WildcatSanctionsSentinel.sol

20   mapping(address borrower => mapping(address account => bool sanctionOverride))

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L20

```solidity
file: src/libraries/MarketState.sol

14    bool isClosed;

27    bool isDelinquent;'

130  function hasPendingExpiredBatch(MarketState memory state) internal view returns (bool result) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L14


```solidity
file: src/market/WildcatMarketConfig.sol

114    bool _isAuthorized

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L114

```solidity
file: src/libraries/MathUtils.sol

72    bool condition,

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L72


```solidity
file: src/libraries/StringQuery.sol

38  bool isBytes32;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/StringQuery.sol#L38

## [G-15] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file: src/WildcatMarketController.sol

302   if (msg.sender == borrower) {

306   } else if (msg.sender != address(controllerFactory)) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L302

```solidity
file: src/WildcatMarketControllerFactory.sol

66   if (msg.sender != archController.owner()) {

283  if (!archController.isRegisteredBorrower(msg.sender)) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L66

```solidity
file:  src/market/WildcatMarketBase.sol

132   if (msg.sender != borrower) revert NotApprovedBorrower();

137   if (msg.sender != controller) revert NotController();

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L132

## [G-16] A modifier used only once and not being inherited should be inlined to save gas

```solidity
file: src/WildcatArchController.sol

41    modifier onlyControllerFactory() {

48    modifier onlyController() {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L41


```solidity
file: src/WildcatMarketController.sol

87    modifier onlyControlledMarket(address market) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L87

```solidity
file: src/WildcatMarketControllerFactory.sol

65    modifier onlyArchControllerOwner() {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L65


```solidity
file: src/market/WildcatMarketBase.sol

131  modifier onlyBorrower() {

136   modifier onlyController() {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L131

```solidity
file: src/ReentrancyGuard.sol

31    modifier nonReentrant() {

41    modifier nonReentrantView() { 

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/ReentrancyGuard.sol#L31

## [G-17] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

Note that in order to do this optimization safely we will need to cache and restore the free memory pointer after the loop. We will also set the zero slot (0x60) back to 0.

```solidity
file: src/WildcatArchController.sol

93   for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }

136  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllerFactories.at(start + i);
    }

179  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllers.at(start + i);
    }

222 for (uint256 i = 0; i < count; i++) {
      arr[i] = _markets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93-L95


```solidity
file:  src/WildcatMarketController.sol

133     for (uint256 i = 0; i < count; i++) {
      arr[i] = _authorizedLenders.at(start + i);
    }

212  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controlledMarkets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133-L135


```solidity
file: src/WildcatMarketControllerFactory.sol

146    for (uint256 i = 0; i < count; i++) {
      arr[i] = _deployedControllers.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L146-L148


```solidity
file: src/libraries/FIFOQueue.sol

48    for (uint256 i = 0; i < len; i++) {
      _values[i] = arr.data[startIndex + i];
    }

75  for (uint256 i = 0; i < n; i++) {
      delete arr.data[startIndex + i];
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48-L50

## [G-18] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity
file:  src/WildcatMarketControllerFactory.sol

286   _tmpMarketBorrowerParameter = msg.sender;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L286


```solidity
file: src/WildcatSanctionsEscrow.sol

17   sentinel = msg.sender;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L17

## [G-19] Can make the variable outside the loop to save gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here’s an example:

```solidity
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}

```

### Making variable inside the loop gas value:   662604
### Making variable outside the loop gas value:  661873

### Tools used Remix 
```solidity
file: src/WildcatMarketController.sol

155   address lender = lenders[i];

171   address lender = lenders[i];

184   address market = markets[i];

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L155


## [G-20] Use the existing memory value


### use the existine memory value of  arr.nextIndex instead of using arr.nextIndex in all three function to do addition

```solidity
file:  src/libraries/FIFOQueue.sol

55    function push(FIFOQueue storage arr, uint32 value) internal {
    uint128 nextIndex = arr.nextIndex;
    arr.data[nextIndex] = value;
    arr.nextIndex = nextIndex + 1;
  }

61 function shift(FIFOQueue storage arr) internal {
    uint128 startIndex = arr.startIndex;
    if (startIndex == arr.nextIndex) {
      revert FIFOQueueOutOfBounds();
    }
    delete arr.data[startIndex];
    arr.startIndex = startIndex + 1;
  }

70 function shiftN(FIFOQueue storage arr, uint128 n) internal {
    uint128 startIndex = arr.startIndex;
    if (startIndex + n > arr.nextIndex) {
      revert FIFOQueueOutOfBounds();
    }
    for (uint256 i = 0; i < n; i++) {
      delete arr.data[startIndex + i];
    }
    arr.startIndex = startIndex + n;
  }


```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L55-L59

### use the existine memory value of  state.reserveRatioBips instead of using state.reserveRatioBips 

```solidity
file: src/market/WildcatMarketConfig.sol

171    function setReserveRatioBips(uint16 _reserveRatioBips) public onlyController nonReentrant {
    if (_reserveRatioBips > BIP) {
      revert ReserveRatioBipsTooHigh();
    }

    MarketState memory state = _getUpdatedState();

    uint256 initialReserveRatioBips = state.reserveRatioBips;

    if (_reserveRatioBips < initialReserveRatioBips) {
      if (state.liquidityRequired() > totalAssets()) {
        revert InsufficientReservesForOldLiquidityRatio();
      }
    }
    state.reserveRatioBips = _reserveRatioBips;
    if (_reserveRatioBips > initialReserveRatioBips) {
      if (state.liquidityRequired() > totalAssets()) {
        revert InsufficientReservesForNewLiquidityRatio();
      }
    }
    _writeState(state);
    emit ReserveRatioBipsUpdated(_reserveRatioBips);
  }
}

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L171-L194


## [G-21] The creation of an intermediary array can be avoided

Sometimes, it’s not necessary to create an intermediate array to store values.

```solidity
file: src/WildcatArchController.sol

81    function getRegisteredBorrowers() external view returns (address[] memory) {

88    ) external view returns (address[] memory arr) {

124   function getRegisteredControllerFactories() external view returns (address[] memory) {

167   function getRegisteredControllers() external view returns (address[] memory) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L81

## [G-22] Use != 0 instead of > 0 for unsigned integer comparison

It’s generally more gas-efficient to use != 0 instead of > 0 when comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation, while the > 0 comparison requires an additional subtraction operation. As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.


```solidity
file:  src/WildcatMarketControllerFactory.sol

201    bool hasOriginationFee = originationFeeAmount > 0;

205    (protocolFeeBips > 0 && nullFeeRecipient) ||

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L201

```solidity
file: src/libraries/FeeMath.sol

67    if (timeWithPenalty > 0) {

155   if (protocolFeeBips > 0) {

159   if (delinquencyFeeBips > 0) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L67

```solidity
file:  src/market/WildcatMarket.sol
 
147   if (_withdrawalData.unpaidBatches.length() > 0) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L147

```solidity
file: src/market/WildcatMarketBase.sol

79     if ((parameters.protocolFeeBips > 0).and(parameters.feeRecipient == address(0))) {

170    if (scaledBalance > 0) {

428    if (availableLiquidity > 0) {

472    if (availableLiquidity > 0) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L79

```solidity
file:  src/market/WildcatMarketWithdrawals.sol

32     if ((expiry == expiredBatchExpiry).and(expiry > 0)) {

112    if (availableLiquidity > 0) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L32


## [G-23] Loop best practice to save gas

```solidity
function Plusi() public view {
		for(uint i=0; i<10;) {
			console.log("Number ==",i);
			unchecked{
				++i;
			}
		}
	}
best practice
-----------------------------------------------------
for (uint i = 0; i < length; i = unchecked_inc(i)) {
    // do something that doesn't change the value of i
}
function unchecked_inc(uint i) returns (uint) {
    unchecked {
        return i + 1;
    }
}

```


```solidity
file: src/WildcatArchController.sol

93   for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }

136  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllerFactories.at(start + i);
    }

179  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllers.at(start + i);
    }

222 for (uint256 i = 0; i < count; i++) {
      arr[i] = _markets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93-L95


```solidity
file:  src/WildcatMarketController.sol

133     for (uint256 i = 0; i < count; i++) {
      arr[i] = _authorizedLenders.at(start + i);
    }

154  for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.add(lender)) {
        emit LenderAuthorized(lender);
      }
    
170    for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.remove(lender)) {
        emit LenderDeauthorized(lender);
      }
    }

183    for (uint256 i; i < markets.length; i++) {
      address market = markets[i];
      if (!_controlledMarkets.contains(market)) {
        revert NotControlledMarket();
      }
      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
    }


212  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controlledMarkets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133-L135


```solidity
file: src/WildcatMarketControllerFactory.sol

146    for (uint256 i = 0; i < count; i++) {
      arr[i] = _deployedControllers.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L146-L148


```solidity
file: src/libraries/FIFOQueue.sol

48    for (uint256 i = 0; i < len; i++) {
      _values[i] = arr.data[startIndex + i];
    }

75  for (uint256 i = 0; i < n; i++) {
      delete arr.data[startIndex + i];
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48-L50

## [G-24] Internal functions that are not called by the contract should be removed to save deployment gas

Internal functions in Solidity are only intended to be invoked within the contract or by other internal functions. If an internal function is not called anywhere within the contract, it serves no purpose and contributes unnecessary overhead during deployment. Removing such functions can lead to substantial gas savings.

```solidity
file: src/libraries/BoolUtils.sol

5:   function and(bool a, bool b) internal pure returns (bool c) {

11:   function or(bool a, bool b) internal pure returns (bool c) {

17:   function xor(bool a, bool b) internal pure returns (bool c) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/BoolUtils.sol#L5

```solidity
file: src/libraries/FIFOQueue.sol

19:   function empty(FIFOQueue storage arr) internal view returns (bool) {

23:   function first(FIFOQueue storage arr) internal view returns (uint32) {

30:   function at(FIFOQueue storage arr, uint256 index) internal view returns (uint32) {

38:   function length(FIFOQueue storage arr) internal view returns (uint128) {

42:   function values(FIFOQueue storage arr) internal view returns (uint32[] memory _values) {

55:   function push(FIFOQueue storage arr, uint32 value) internal {

61:   function shift(FIFOQueue storage arr) internal {

70:   function shiftN(FIFOQueue storage arr, uint128 n) internal {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L19

```solidity
file:  src/libraries/FeeMath.sol

30:   function calculateBaseInterest(
        MarketState memory state,
        uint256 timestamp
     ) internal pure returns (uint256 baseInterestRay) {

40:   function applyProtocolFee(
41:     MarketState memory state,
42:     uint256 baseInterestRay,
43:     uint256 protocolFeeBips
44:   ) internal pure returns (uint256 protocolFee) {

53:   function updateDelinquency(
54:     MarketState memory state,
55:     uint256 timestamp,
56:     uint256 delinquencyFeeBips,
57:     uint256 delinquencyGracePeriod
58:   ) internal pure returns (uint256 delinquencyFeeRay) {

142:   function updateScaleFactorAndFees(
143:     MarketState memory state,
144:     uint256 protocolFeeBips,
145:     uint256 delinquencyFeeBips,
146:     uint256 delinquencyGracePeriod,
147:     uint256 timestamp
148:   )
149:     internal
150:     pure
151:     returns (uint256 baseInterestRay, uint256 delinquencyFeeRay, uint256 protocolFee)
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L30

```solidity
file: src/libraries/LibStoredInitCode.sol

7:   function deployInitCode(bytes memory data) internal returns (address initCodeStorage) {

48:   function getCreate2Prefix(address deployer) internal pure returns (uint256 create2Prefix) {

54:   function calculateCreate2Address(
55:     uint256 create2Prefix,
56:     bytes32 salt,
57:     uint256 initCodeHash
58:   ) internal pure returns (address create2Address) {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol#L7

```solidity
file: src/libraries/MarketState.sol

51:   function totalSupply(MarketState memory state) internal pure returns (uint256) {

59:   function maximumDeposit(MarketState memory state) internal pure returns (uint256) {

66:   function normalizeAmount(
67:     MarketState memory state,
68:     uint256 amount
69:   ) internal pure returns (uint256) {

76:   function scaleAmount(MarketState memory state, uint256 amount) internal pure returns (uint256) {

87:   function liquidityRequired(
88:     MarketState memory state
89:   ) internal pure returns (uint256 _liquidityRequired) {

105:   function withdrawableProtocolFees(
106:     MarketState memory state,
107:     uint256 totalAssets
108:   ) internal pure returns (uint128) {

123:   function borrowableAssets(
124:     MarketState memory state,
125:     uint256 totalAssets
126:   ) internal pure returns (uint256) {

130:   function hasPendingExpiredBatch(MarketState memory state) internal view returns (bool result) {

138:   function totalDebts(MarketState memory state) internal pure returns (uint256) {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L51

```solidity
file: src/libraries/MathUtils.sol

30:   function calculateLinearInterestFromBips(
31:     uint256 rateBip,
32:     uint256 timeDelta
33:   ) internal pure returns (uint256 result) {

44:   function min(uint256 a, uint256 b) internal pure returns (uint256 c) {

51:   function max(uint256 a, uint256 b) internal pure returns (uint256 c) {

59:   function satSub(uint256 a, uint256 b) internal pure returns (uint256 c) {

85:   function bipMul(uint256 a, uint256 b) internal pure returns (uint256 c) {

105:   function bipDiv(uint256 a, uint256 b) internal pure returns (uint256 c) {

121:   function bipToRay(uint256 a) internal pure returns (uint256 b) {

138:   function rayMul(uint256 a, uint256 b) internal pure returns (uint256 c) {

155:   function rayDiv(uint256 a, uint256 b) internal pure returns (uint256 c) {

173:   function mulDiv(uint256 x, uint256 y, uint256 d) internal pure returns (uint256 z) {

191:   function mulDivUp(uint256 x, uint256 y, uint256 d) internal pure returns (uint256 z) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L30

```solidity
file:  src/libraries/SafeCastLib.sol

17:   function toUint8(uint256 x) internal pure returns (uint8 y) {

21:   function toUint16(uint256 x) internal pure returns (uint16 y) {

25:   function toUint24(uint256 x) internal pure returns (uint24 y) {

29:   function toUint32(uint256 x) internal pure returns (uint32 y) {

33:   function toUint40(uint256 x) internal pure returns (uint40 y) {

37:   function toUint48(uint256 x) internal pure returns (uint48 y) {

41:   function toUint56(uint256 x) internal pure returns (uint56 y) {

45:   function toUint64(uint256 x) internal pure returns (uint64 y) {

49:   function toUint72(uint256 x) internal pure returns (uint72 y) {

53:   function toUint80(uint256 x) internal pure returns (uint80 y) {

57:   function toUint88(uint256 x) internal pure returns (uint88 y) {

61:   function toUint96(uint256 x) internal pure returns (uint96 y) {

65:   function toUint104(uint256 x) internal pure returns (uint104 y) {

69:   function toUint112(uint256 x) internal pure returns (uint112 y) {

73:   function toUint120(uint256 x) internal pure returns (uint120 y) {

77:   function toUint128(uint256 x) internal pure returns (uint128 y) {

81:   function toUint136(uint256 x) internal pure returns (uint136 y) {

85:   function toUint144(uint256 x) internal pure returns (uint144 y) {

89:   function toUint152(uint256 x) internal pure returns (uint152 y) {

93:   function toUint160(uint256 x) internal pure returns (uint160 y) {

97:   function toUint168(uint256 x) internal pure returns (uint168 y) {

101:   function toUint176(uint256 x) internal pure returns (uint176 y) {

105:   function toUint184(uint256 x) internal pure returns (uint184 y) {

109:   function toUint192(uint256 x) internal pure returns (uint192 y) {

113:   function toUint200(uint256 x) internal pure returns (uint200 y) {

117:   function toUint208(uint256 x) internal pure returns (uint208 y) {

121:   function toUint216(uint256 x) internal pure returns (uint216 y) {

125:   function toUint224(uint256 x) internal pure returns (uint224 y) {

129:   function toUint232(uint256 x) internal pure returns (uint232 y) {

133:   function toUint240(uint256 x) internal pure returns (uint240 y) {

137:   function toUint248(uint256 x) internal pure returns (uint248 y) {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/SafeCastLib.sol#L17

```solidity
file:  src/libraries/Withdrawal.sol

38:   function scaledOwedAmount(WithdrawalBatch memory batch) internal pure returns (uint104) {

47:   function availableLiquidityForPendingBatch(
48:     WithdrawalBatch memory batch,
49:     MarketState memory state,
50:     uint256 totalAssets
51:   ) internal pure returns (uint256) {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L38


## [G‑25] Counting down in for statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

by changing this logic you can save 12171 gas per one for loop 

Tools used Remix


```solidity
file: src/WildcatArchController.sol

93   for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }

136  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllerFactories.at(start + i);
    }

179  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllers.at(start + i);
    }

222 for (uint256 i = 0; i < count; i++) {
      arr[i] = _markets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93-L95


```solidity
file:  src/WildcatMarketController.sol

133     for (uint256 i = 0; i < count; i++) {
      arr[i] = _authorizedLenders.at(start + i);
    }

154  for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.add(lender)) {
        emit LenderAuthorized(lender);
      }
    
170    for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.remove(lender)) {
        emit LenderDeauthorized(lender);
      }
    }

183    for (uint256 i; i < markets.length; i++) {
      address market = markets[i];
      if (!_controlledMarkets.contains(market)) {
        revert NotControlledMarket();
      }
      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
    }


212  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controlledMarkets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133-L135


```solidity
file: src/WildcatMarketControllerFactory.sol

146    for (uint256 i = 0; i < count; i++) {
      arr[i] = _deployedControllers.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L146-L148


```solidity
file: src/libraries/FIFOQueue.sol

48    for (uint256 i = 0; i < len; i++) {
      _values[i] = arr.data[startIndex + i];
    }

75  for (uint256 i = 0; i < n; i++) {
      delete arr.data[startIndex + i];
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48-L50

### Test Code

```solidity
contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;
    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }
    function testGas() public {
        c0.AddNum();
        c1.AddNum();
    }
}
contract Contract0 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=0;i<=9;i++){
            _num = _num +1;
        }
        num = _num;
    }
}
contract Contract1 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=9;i>=0;i--){
            _num = _num +1;
        }
        num = _num;
    }
}
```

## [G‑26] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity
file: src/WildcatArchController.sol

93   for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }

136  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllerFactories.at(start + i);
    }

179  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllers.at(start + i);
    }

222 for (uint256 i = 0; i < count; i++) {
      arr[i] = _markets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93-L95


```solidity
file:  src/WildcatMarketController.sol

133     for (uint256 i = 0; i < count; i++) {
      arr[i] = _authorizedLenders.at(start + i);
    }

212  for (uint256 i = 0; i < count; i++) {
      arr[i] = _controlledMarkets.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133-L135


```solidity
file: src/WildcatMarketControllerFactory.sol

146    for (uint256 i = 0; i < count; i++) {
      arr[i] = _deployedControllers.at(start + i);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L146-L148


```solidity
file: src/libraries/FIFOQueue.sol

48    for (uint256 i = 0; i < len; i++) {
      _values[i] = arr.data[startIndex + i];
    }

75  for (uint256 i = 0; i < n; i++) {
      delete arr.data[startIndex + i];
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48-L50

## [G-27] storage variable can be replaced with calldata variable

```solidity
file: src/libraries/FIFOQueue.sol

19   function empty(FIFOQueue storage arr) internal view returns (bool) {

23   function first(FIFOQueue storage arr) internal view returns (uint32) {

30   function at(FIFOQueue storage arr, uint256 index) internal view returns (uint32) {

38   function length(FIFOQueue storage arr) internal view returns (uint128) {
    
42   function values(FIFOQueue storage arr) internal view returns (uint32[] memory _values) {

55   function push(FIFOQueue storage arr, uint32 value) internal {'

61   function shift(FIFOQueue storage arr) internal {

70   function shiftN(FIFOQueue storage arr, uint128 n) internal {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L19

## [G-28] Conditional check can be performed during variable initialization

```solidity
file: src/WildcatMarketControllerFactory.sol

201   bool hasOriginationFee = originationFeeAmount > 0;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L201


## [G-29] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

### cache  WildcatMarket(market) exteranl call outside the loop

```solidity
file: src/WildcatMarketController.sol

183    for (uint256 i; i < markets.length; i++) {
      address market = markets[i];
      if (!_controlledMarkets.contains(market)) {
        revert NotControlledMarket();
      }
      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L183-L189

## [G-30] Duplicated require()/revert() Checks Should Be Refactored To A Modifier Or Function

Saves deployment costs.


```solidity
file: src/WildcatMarketController.sol

 @audit the same condation is also used in line  185

88   if (!_controlledMarkets.contains(market)) 

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L88


```solidity
file: src/market/WildcatMarket.sol

 @audit the same condation is also used in line  121

48   if (state.isClosed) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L48


```solidity
file:  src/market/WildcatMarketBase.sol

 @audit the same condation is also used in line  337

322   if (state.timeDelinquent > delinquencyGracePeriod)

 @audit the same condation is also used in line  410

361   if (state.hasPendingExpiredBatch())

 @audit the same condation is also used in line  472

428   if (availableLiquidity > 0)

 @audit the same condation is also used in line  536

507   if (scaledAmountOwed == 0) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L322


```solidity
file: src/market/WildcatMarketWithdrawals.sol

 @audit the same condation is also used in line  141

49   if (expiry > block.timestamp)

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L49

## [G-31] Combine events to save 2 Glogtopic (375 gas)

The events below are only emitted once in the handleRewards function. We can combine the events into one singular event to save two Glogtopic (375 gas) that would otherwise be paid for the additional two events.

```solidity
file:  src/WildcatArchController.sol
 
67       emit BorrowerAdded(borrower);
  }

  function removeBorrower(address borrower) external onlyOwner {
    if (!_borrowers.remove(borrower)) {
      revert BorrowerDoesNotExist();
    }
74   emit BorrowerRemoved(borrower);

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L67-L74


```solidity
file: src/WildcatArchController.sol

109       }
    emit ControllerFactoryAdded(factory);
  }

  function removeControllerFactory(address factory) external onlyOwner {
    if (!_controllerFactories.remove(factory)) {
      revert ControllerFactoryDoesNotExist();
    }
117    emit ControllerFactoryRemoved(factory);
  }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L109-L118

## [G-32] Failure to check the zero address in the constructor causes the contract to be deployed again

Zero address control is not performed in the constructor in all 2 contracts within the scope of the audit. Bypassing this check could cause the contract to be deployed by mistakenly entering a zero address. In this case, the contract will need to be redeployed. This means extra gas consumption as contract deployment costs are high

```solidity
file:  src/WildcatMarketControllerFactory.sol

72    constructor(
      address _archController,
      address _sentinel,
      MarketParameterConstraints memory constraints
    ) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L72-L76

```solidity
file: src/WildcatSanctionsSentinel.sol

24   constructor(address _archController, address _chainalysisSanctionsList) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L24

## [G-33] bytes constants are more eficient than string constans

If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

### Using of string constant gas value: 711113
### Using of bytes constant gas value: 685782

### Tools Used Remix 
```solidity
file: src/market/WildcatMarketBase.sol

24   string public constant version = '1.0';

57   string public name;

60   string public symbol;

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L24

## [G-34] Use previousTimeDelinquent cached value in if stament instead of using state.timeDelinquent 

```solidity
file: src/libraries/FeeMath.sol

94       // Seconds in delinquency at last update
    uint256 previousTimeDelinquent = state.timeDelinquent;

    if (state.isDelinquent) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L94-L97

## [G-35] Possible Optimization in WildcatMarketBase.sol

In the _applyWithdrawalBatchPayment function, the scaledAvailableLiquidity variable is updated before the _scaledAmountOwed  cap check. This means that even if the _scaledAmountOwed  cap check fails and reverts the transaction, the scaledAvailableLiquidity variable will still have been updated, wasting gas. Moving the scaledAvailableLiquidity update after the _scaledAmountOwed  cap check could save gas in the case where the _scaledAmountOwed  cap check fails.

```solidity
file:  src/market/WildcatMarketBase.sol

498    function _applyWithdrawalBatchPayment(
       WithdrawalBatch memory batch,
       MarketState memory state,
       uint32 expiry,
       uint256 availableLiquidity
     )    internal {
       uint104 scaledAvailableLiquidity = state.scaleAmount(availableLiquidity).toUint104();
       uint104 scaledAmountOwed = batch.scaledTotalAmount - batch.scaledAmountBurned;
       // Do nothing if batch is already paid
       if (scaledAmountOwed == 0) {
         return;
       }
    
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L498-L509


## [G-36] Possible Optimization in WildcatMarket.sol

In the closeMarket function, the state variable is updated before the _withdrawalData.unpaidBatches.length()  cap check. This means that even if the _withdrawalData.unpaidBatches.length()  cap check fails and reverts, the state variable will still have been updated, wasting gas. Moving the state update after the _withdrawalData.unpaidBatches.length()  cap check could save gas in the case where the _withdrawalData.unpaidBatches.length()  cap check fails.

```solidity
file:  src/market/WildcatMarket.sol

144    function closeMarket() external onlyController nonReentrant {
       MarketState memory state = _getUpdatedState();
       state.annualInterestBips = 0;
       state.isClosed = true;
       state.reserveRatioBips = 0;
       if (_withdrawalData.unpaidBatches.length() > 0) {
         revert CloseMarketWithUnpaidWithdrawals();
       }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L144-L149


## [G-37] Possible Optimization in FeeMath.sol

In the updateScaleFactorAndFees function, the baseInterestRay variable is updated before the protocolFeeBips cap check. This means that even if the protocolFeeBips cap check fails and reverts, the baseInterestRay variable will still have been updated, wasting gas. Moving the baseInterestRay update after the protocolFeeBips cap check could save gas in the case where the protocolFeeBips cap check fails.

```solidity
file: src/libraries/FeeMath.sol

153      baseInterestRay = state.calculateBaseInterest(timestamp);

    if (protocolFeeBips > 0) {
      protocolFee = state.applyProtocolFee(baseInterestRay, protocolFeeBips);
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L153-L157

## [G-38] Possible Optimization in WildcatMarketConfig.sol

In the setAnnualInterestBips function, the state variable is updated before the _annualInterestBips  cap check. This means that even if the _annualInterestBips  cap check fails and reverts, the state variable will still have been updated, wasting gas. Moving the state update after the _annualInterestBips  cap check could save gas in the case where the _annualInterestBips  cap check fails.

```solidity
file:  src/market/WildcatMarketConfig.sol

149    function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant {
    MarketState memory state = _getUpdatedState();

    if (_annualInterestBips > BIP) {
      revert InterestRateTooHigh();
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L149-L154


### the same issue is in contract WildcatMarketToken.sol function _transfer


```solidity
file: src/market/WildcatMarketToken.sol

64    function _transfer(address from, address to, uint256 amount) internal virtual {
    MarketState memory state = _getUpdatedState();
    uint104 scaledAmount = state.scaleAmount(amount).toUint104();

    if (scaledAmount == 0) {
      revert NullTransferAmount();
    }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L64-L70