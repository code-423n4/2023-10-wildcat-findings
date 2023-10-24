## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Array is push()ed but not pop()ed | 1 |
| [L-2](#L-2) | Constructor / Initialization Function Lacks Parameter Validation | 1 |
| [L-3](#L-3) | Avoid Usage of decimals() in ERC-20 Contracts | 1 |
| [L-4](#L-4) | Use descriptive constant instead of 0 as a parameter | 5 |
| [L-5](#L-5) | Undetected Uninitialized Variables in Constructors | 8 |
| [L-6](#L-6) | SafeTransferLib Does Not Ensure That the Token Contract Exists | 1 |
| [L-7](#L-7) | Unbounded State Variables May Pose Security Risks | 1 |
| [L-8](#L-8) | Timestamp may be manipulation | 11 |
| [L-9](#L-9) | Consider implementing two-step procedure for updating protocol addresses | 49 |
| [L-10](#L-10) | Unspecific compiler version pragma. without specifying an upper bound is unsafe. Future versions of Solidity may introduce breaking changes, and your code may become incompatible. | 21 |
| [L-11](#L-11) | Use Ownable2Step rather than Ownable | 1 |
| [L-12](#L-12) | Avoid Reverting on Zero-Value Transfers | 4 |
### <a name="L-1"></a>[L-1] Array is push()ed but not pop()ed

#### Impact:
Array entries are added but never removed. Consider whether this is the intended behavior or if there should be a mechanism to remove old entries.

*Instances (1)*:
```solidity
File: market/WildcatMarketBase.sol

484:       _withdrawalData.unpaidBatches.push(expiry);

```

### <a name="L-2"></a>[L-2] Constructor / Initialization Function Lacks Parameter Validation

#### Impact:
Constructors and initialization functions are critical in contracts, as they set initial states during deployment. Parameters passed to these functions directly affect the behavior of the contract/protocol. Failing to validate parameters can lead to system failures, abnormal behavior, instability, or security vulnerabilities. It's essential to thoroughly validate all parameters in constructors and initialization functions, and transactions should be rolled back if exceptions are found.

*Instances (1)*:
```solidity
File: WildcatSanctionsSentinel.sol

24:   constructor(address _archController, address _chainalysisSanctionsList) {

```

### <a name="L-3"></a>[L-3] Avoid Usage of decimals() in ERC-20 Contracts

#### Impact:
The decimals() function is not part of the ERC-20 standard and is an optional extension. Relying on it may lead to compatibility issues with tokens that do not support this function.

*Instances (1)*:
```solidity
File: market/WildcatMarketBase.sol

99:     decimals = IERC20Metadata(parameters.asset).decimals();

```

### <a name="L-4"></a>[L-4] Use descriptive constant instead of 0 as a parameter

#### Impact:
Passing 0 or 0x0 as a function argument can sometimes result in a security issue. Consider using a constant variable with a descriptive name, so it's clear that the argument is intentionally being used, and for the right reasons.

*Instances (5)*:
```solidity
File: libraries/LibStoredInitCode.sol

84:     deployment = createWithStoredInitCode(initCodeStorage, 0);

103:     deployment = create2WithStoredInitCode(initCodeStorage, salt, 0);

```

```solidity
File: libraries/StringQuery.sol

41:     let status := staticcall(gas(), target, 0, 0x04, 0, 0)

41:     let status := staticcall(gas(), target, 0, 0x04, 0, 0)

52:           returndatacopy(0, 0, returndatasize())

```

### <a name="L-5"></a>[L-5] Undetected Uninitialized Variables in Constructors

#### Impact:
This issue identifies undetected uninitialized variables in constructors, which can lead to runtime errors or unexpected behavior. Ensure that all variables are properly initialized in the constructor.

*Instances (8)*:
```solidity
File: WildcatMarketControllerFactory.sol

78:     sentinel = _sentinel;

253:     parameters.borrower = _tmpMarketBorrowerParameter;

```

```solidity
File: WildcatSanctionsSentinel.sol

25:     archController = _archController;

26:     chainalysisSanctionsList = _chainalysisSanctionsList;

```

```solidity
File: market/WildcatMarketBase.sol

359:     state = _state;

408:     state = _state;

```

```solidity
File: market/WildcatMarketConfig.sol

156:     state.annualInterestBips = _annualInterestBips;

185:     state.reserveRatioBips = _reserveRatioBips;

```

### <a name="L-6"></a>[L-6] SafeTransferLib Does Not Ensure That the Token Contract Exists

#### Impact:
The SafeTransferLib used in this code does not verify if the token contract exists, potentially leading to issues when transferring tokens. It's essential to ensure the token's contract existence before making transfers.

*Instances (1)*:
```solidity
File: WildcatMarketController.sol

346:       originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);

```

### <a name="L-7"></a>[L-7] Unbounded State Variables May Pose Security Risks

#### Impact:
State variables should be bounded within reasonable value ranges to mitigate potential security risks, including griefing attacks.

*Instances (1)*:
```solidity
File: WildcatMarketControllerFactory.sol

211:     _protocolFeeConfiguration = ProtocolFeeConfiguration({

```

### <a name="L-8"></a>[L-8] Timestamp may be manipulation

#### Impact:
The block.timestamp can be manipulated by miners to perform MEV profiting or other time-based attacks.

*Instances (11)*:
```solidity
File: WildcatMarketController.sol

484:       tmp.expiry = uint128(block.timestamp + 2 weeks);

495:     if (block.timestamp < tmp.expiry) {

```

```solidity
File: market/WildcatMarket.sol

160:     emit MarketClosed(block.timestamp);

```

```solidity
File: market/WildcatMarketBase.sol

114:       lastInterestAccruedTimestamp: uint32(block.timestamp)

378:     if (block.timestamp != state.lastInterestAccruedTimestamp) {

384:           block.timestamp

434:     if (state.lastInterestAccruedTimestamp != block.timestamp) {

439:         block.timestamp

```

```solidity
File: market/WildcatMarketWithdrawals.sol

49:     if (expiry > block.timestamp) {

95:       state.pendingWithdrawalExpiry = uint32(block.timestamp + withdrawalBatchDuration);

141:     if (expiry > block.timestamp) {

```

### <a name="L-9"></a>[L-9] Consider implementing two-step procedure for updating protocol addresses

#### Impact:
A copy-paste error or a typo may end up bricking protocol functionality, or sending tokens to an address with no known private key. Consider implementing a two-step procedure for updating protocol addresses, where the recipient is set as pending and must 'accept' the assignment by making an affirmative call.

*Instances (49)*:
```solidity
File: ReentrancyGuard.sol

32:     _setReentrancyGuard();

59:   function _setReentrancyGuard() internal {

```

```solidity
File: WildcatMarketController.sol

255:   function _resetTmpMarketParameters() internal {

256:     _tmpMarketParameters.asset = address(1);

345:     if (originationFeeAsset != address(0)) {

346:       originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);

359:     _resetTmpMarketParameters();

481:         WildcatMarket(market).setReserveRatioBips(9000);

487:     WildcatMarket(market).setAnnualInterestBips(annualInterestBips);

490:   function resetReserveRatio(address market) external virtual {

499:     WildcatMarket(market).setReserveRatioBips(uint256(tmp.reserveRatioBips).toUint16());

```

```solidity
File: WildcatMarketControllerFactory.sol

203:     bool nullOriginationFeeAsset = originationFeeAsset == address(0);

```

```solidity
File: WildcatSanctionsEscrow.sol

18:     (borrower, account, asset) = WildcatSanctionsSentinel(sentinel).tmpEscrowParams();

22:     return IERC20(asset).balanceOf(address(this));

29:   function escrowedAsset() public view override returns (address, uint256) {

30:     return (asset, balance());

38:     IERC20(asset).transfer(account, amount);

```

```solidity
File: WildcatSanctionsSentinel.sol

27:     _resetTmpEscrowParams();

30:   function _resetTmpEscrowParams() internal {

110:     new WildcatSanctionsEscrow{ salt: keccak256(abi.encode(borrower, account, asset)) }();

118:     _resetTmpEscrowParams();

```

```solidity
File: libraries/MarketState.sol

127:     return totalAssets.satSub(state.liquidityRequired());

```

```solidity
File: libraries/Withdrawal.sol

58:     return totalAssets.satSub(unavailableAssets);

```

```solidity
File: market/WildcatMarket.sol

60:     asset.safeTransferFrom(msg.sender, address(this), amount);

101:     uint128 withdrawableFees = state.withdrawableProtocolFees(totalAssets());

107:     asset.safeTransfer(feeRecipient, withdrawableFees);

124:     uint256 borrowable = state.borrowableAssets(totalAssets());

129:     asset.safeTransfer(msg.sender, amount);

150:     uint256 currentlyHeld = totalAssets();

154:       asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);

157:       asset.safeTransfer(borrower, currentlyHeld - totalDebts);

```

```solidity
File: market/WildcatMarketBase.sol

99:     decimals = IERC20Metadata(parameters.asset).decimals();

238:   function totalAssets() public view returns (uint256) {

239:     return IERC20(asset).balanceOf(address(this));

252:   function borrowableAssets() external view nonReentrantView returns (uint256) {

253:     return currentState().borrowableAssets(totalAssets());

308:     return currentState().withdrawableProtocolFees(totalAssets());

426:         totalAssets()

449:     bool isDelinquent = state.liquidityRequired() > totalAssets();

471:     uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());

```

```solidity
File: market/WildcatMarketConfig.sol

134:   function setMaxTotalSupply(uint256 _maxTotalSupply) external onlyController nonReentrant {

149:   function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant {

171:   function setReserveRatioBips(uint16 _reserveRatioBips) public onlyController nonReentrant {

181:       if (state.liquidityRequired() > totalAssets()) {

187:       if (state.liquidityRequired() > totalAssets()) {

```

```solidity
File: market/WildcatMarketWithdrawals.sol

111:     uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());

171:       asset.safeTransfer(escrow, normalizedAmountWithdrawn);

179:       asset.safeTransfer(accountAddress, normalizedAmountWithdrawn);

200:     uint256 availableLiquidity = totalAssets() -

```

### <a name="L-10"></a>[L-10] Unspecific compiler version pragma. without specifying an upper bound is unsafe. Future versions of Solidity may introduce breaking changes, and your code may become incompatible.

*Instances (21)*:
```solidity
File: ReentrancyGuard.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: WildcatArchController.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: WildcatMarketController.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: WildcatMarketControllerFactory.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: WildcatSanctionsEscrow.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: WildcatSanctionsSentinel.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: libraries/BoolUtils.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: libraries/Errors.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: libraries/FIFOQueue.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: libraries/FeeMath.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: libraries/LibStoredInitCode.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: libraries/MarketState.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: libraries/MathUtils.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: libraries/SafeCastLib.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: libraries/StringQuery.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: libraries/Withdrawal.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: market/WildcatMarket.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: market/WildcatMarketBase.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: market/WildcatMarketConfig.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: market/WildcatMarketToken.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: market/WildcatMarketWithdrawals.sol

2: pragma solidity >=0.8.20;

```

### <a name="L-11"></a>[L-11] Use Ownable2Step rather than Ownable

#### Impact:
Ownable2Step and Ownable2StepUpgradeable prevent the contract ownership from mistakenly being transferred to an address that cannot handle it, by requiring the recipient of the owner permissions to actively accept via a contract call of its own.

*Instances (1)*:
```solidity
File: WildcatArchController.sol

8: contract WildcatArchController is Ownable {

```

### <a name="L-12"></a>[L-12] Avoid Reverting on Zero-Value Transfers

#### Impact:
While EIP-20 mandates accepting zero-value transfers, some tokens like LEND will revert if this is attempted. Reverting may lead to complete transaction failures, affecting other token operations, including batch operations. Consider checking and skipping transfers if the transfer amount is zero to prevent potential reverts and save gas.

*Instances (4)*:
```solidity
File: WildcatMarketController.sol

346:       originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);

```

```solidity
File: WildcatSanctionsEscrow.sol

38:     IERC20(asset).transfer(account, amount);

```

```solidity
File: market/WildcatMarket.sol

60:     asset.safeTransferFrom(msg.sender, address(this), amount);

157:       asset.safeTransfer(borrower, currentlyHeld - totalDebts);

```

