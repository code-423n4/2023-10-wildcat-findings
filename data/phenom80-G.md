
## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Avoid contract existence checks by using low level calls | 51 |
| [GAS-2](#GAS-2) | Cache state variables to save gas | 104 |
| [GAS-3](#GAS-3) | State variables should be cached in stack variables | 70 |
| [GAS-4](#GAS-4) | Function Call Caching | 155 |
| [GAS-5](#GAS-5) | State variables only set in the constructor should be declared immutable | 4 |
| [GAS-6](#GAS-6) | Optimize gas costs using >= and <= operators | 75 |
| [GAS-7](#GAS-7) | Using private for constants | 1 |
| [GAS-8](#GAS-8) | Simple checks for zero can be done using assembly to save gas | 1 |
| [GAS-9](#GAS-9) | State variables should be cached in stack variables | 70 |
| [GAS-10](#GAS-10) | Upgrade to Solidity 0.8.19 or Later for Gas Optimization | 1 |
| [GAS-11](#GAS-11) | Use assembly to check zero | 11 |
| [GAS-12](#GAS-12) | Use assembly to emit events, in order to save gas | 44 |
| [GAS-13](#GAS-13) | Use assembly to write address/contract type storage values | 292 |
| [GAS-14](#GAS-14) | Using storage instead of memory for structs/arrays saves gas | 35 |
### <a name="GAS-1"></a>[GAS-1] Avoid contract existence checks by using low level calls

#### Impact:
Using low level calls can help avoid unnecessary contract existence checks, resulting in lower gas costs.

*Instances (51)*:
```solidity
File: WildcatArchController.sol

89:     uint256 len = _borrowers.length();

90:     end = MathUtils.min(end, len);

132:     uint256 len = _controllerFactories.length();

133:     end = MathUtils.min(end, len);

175:     uint256 len = _controllers.length();

176:     end = MathUtils.min(end, len);

218:     uint256 len = _markets.length();

219:     end = MathUtils.min(end, len);

```

```solidity
File: WildcatMarketController.sol

53:   uint256 internal immutable ownCreate2Prefix = LibStoredInitCode.getCreate2Prefix(address(this));

96:     MarketControllerParameters memory parameters = controllerFactory.getMarketControllerParameters();

129:     uint256 len = _authorizedLenders.length();

130:     end = MathUtils.min(end, len);

208:     uint256 len = _controlledMarkets.length();

209:     end = MathUtils.min(end, len);

350:     market = LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt, marketInitCodeHash);

```

```solidity
File: WildcatMarketControllerFactory.sol

44:   uint256 internal immutable ownCreate2Prefix = LibStoredInitCode.getCreate2Prefix(address(this));

113:     initCodeStorage = LibStoredInitCode.deployInitCode(controllerInitCode);

123:     initCodeStorage = LibStoredInitCode.deployInitCode(marketInitCode);

142:     uint256 len = _deployedControllers.length();

143:     end = MathUtils.min(end, len);

289:     controller = LibStoredInitCode.calculateCreate2Address(

```

```solidity
File: libraries/FeeMath.sol

23:     uint256 rate = rateBip.bipToRay();

34:     baseInterestRay = MathUtils.calculateLinearInterestFromBips(

46:     uint256 protocolFeeRay = protocolFeeBips.bipMul(baseInterestRay);

104:       uint256 secondsRemainingWithoutPenalty = delinquencyGracePeriod.satSub(

115:     state.timeDelinquent = previousTimeDelinquent.satSub(timeDelta).toUint32();

119:     uint256 secondsRemainingWithPenalty = previousTimeDelinquent.satSub(delinquencyGracePeriod);

153:     baseInterestRay = state.calculateBaseInterest(timestamp);

156:       protocolFee = state.applyProtocolFee(baseInterestRay, protocolFeeBips);

160:       delinquencyFeeRay = state.updateDelinquency(

169:     uint256 scaleFactorDelta = prevScaleFactor.rayMul(baseInterestRay + delinquencyFeeRay);

```

```solidity
File: libraries/MathUtils.sol

34:     uint256 rate = rateBip.bipToRay();

```

```solidity
File: market/WildcatMarket.sol

53:     amount = MathUtils.min(amount, state.maximumDeposit());

56:     uint104 scaledAmount = state.scaleAmount(amount).toUint104();

101:     uint128 withdrawableFees = state.withdrawableProtocolFees(totalAssets());

124:     uint256 borrowable = state.borrowableAssets(totalAssets());

151:     uint256 totalDebts = state.totalDebts();

```

```solidity
File: market/WildcatMarketBase.sol

97:     name = string.concat(parameters.namePrefix, queryName(parameters.asset));

98:     symbol = string.concat(parameters.symbolPrefix, querySymbol(parameters.asset));

321:     uint256 apr = MathUtils.bipToRay(state.annualInterestBips).bipMul(BIP + protocolFeeBips);

424:       uint256 availableLiquidity = expiredBatch.availableLiquidityForPendingBatch(

449:     bool isDelinquent = state.liquidityRequired() > totalAssets();

471:     uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());

504:     uint104 scaledAvailableLiquidity = state.scaleAmount(availableLiquidity).toUint104();

511:     uint128 normalizedAmountPaid = state.normalizeAmount(scaledAmountBurned).toUint128();

533:     uint104 scaledAvailableLiquidity = state.scaleAmount(availableLiquidity).toUint104();

540:     uint128 normalizedAmountPaid = state.normalizeAmount(scaledAmountBurned).toUint128();

```

```solidity
File: market/WildcatMarketConfig.sol

141:     state.maxTotalSupply = _maxTotalSupply.toUint128();

```

```solidity
File: market/WildcatMarketToken.sol

66:     uint104 scaledAmount = state.scaleAmount(amount).toUint104();

```

```solidity
File: market/WildcatMarketWithdrawals.sol

83:     uint104 scaledAmount = state.scaleAmount(amount).toUint104();

111:     uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());

```

### <a name="GAS-2"></a>[GAS-2] Cache state variables to save gas

#### Impact:
Caching state variables in stack variables can save gas by reducing read operations.

*Instances (104)*:
```solidity
File: WildcatMarketController.sol

95:     controllerFactory = IWildcatMarketControllerFactory(msg.sender);

97:     archController = IWildcatArchController(parameters.archController);

226:     bytes32 salt = _deriveSalt(asset, namePrefix, symbolPrefix);

243:     parameters.controller = address(this);

256:     _tmpMarketParameters.asset = address(1);

259:     _tmpMarketParameters.feeRecipient = address(1);

320:     TmpMarketParameterStorage memory parameters = TmpMarketParameterStorage({

349:     bytes32 salt = _deriveSalt(asset, namePrefix, symbolPrefix);

478:         tmp.reserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips());

484:       tmp.expiry = uint128(block.timestamp + 2 weeks);

```

```solidity
File: WildcatMarketControllerFactory.sol

77:     archController = IWildcatArchController(_archController);

111:     bytes memory controllerInitCode = type(WildcatMarketController).creationCode;

112:     initCodeHash = uint256(keccak256(controllerInitCode));

121:     bytes memory marketInitCode = type(WildcatMarket).creationCode;

122:     initCodeHash = uint256(keccak256(marketInitCode));

211:     _protocolFeeConfiguration = ProtocolFeeConfiguration({

244:   address internal _tmpMarketBorrowerParameter = address(1);

252:     parameters.archController = address(archController);

288:     bytes32 salt = bytes32(uint256(uint160(msg.sender)));

298:     _tmpMarketBorrowerParameter = address(1);

328:     controller = deployController();

329:     market = IWildcatMarketController(controller).deployMarket(

344:     bytes32 salt = bytes32(uint256(uint160(borrower)));

```

```solidity
File: WildcatSanctionsEscrow.sol

36:     uint256 amount = balance();

```

```solidity
File: WildcatSanctionsSentinel.sol

11:   bytes32 public constant override WildcatSanctionsEscrowInitcodeHash =

31:     tmpEscrowParams = TmpEscrowParams(address(1), address(1), address(1));

104:     escrowContract = getEscrowAddress(borrower, account, asset);

108:     tmpEscrowParams = TmpEscrowParams(borrower, account, asset);

```

```solidity
File: libraries/Chainalysis.sol

6: IChainalysisSanctionsList constant SanctionsList = IChainalysisSanctionsList(

```

```solidity
File: libraries/FeeMath.sol

47:     protocolFee = uint256(state.scaledTotalSupply).rayMul(

61:     uint256 timeWithPenalty = updateTimeDelinquentAndGetPenaltyTime(

69:       delinquencyFeeRay = calculateLinearInterestFromBips(delinquencyFeeBips, timeWithPenalty);

172:     state.lastInterestAccruedTimestamp = uint32(timestamp);

```

```solidity
File: libraries/LibStoredInitCode.sol

84:     deployment = createWithStoredInitCode(initCodeStorage, 0);

103:     deployment = create2WithStoredInitCode(initCodeStorage, salt, 0);

```

```solidity
File: libraries/MathUtils.sol

45:     c = ternary(a < b, a, b);

52:     c = ternary(a < b, b, a);

```

```solidity
File: libraries/SafeCastLib.sol

18:     _assertNonOverflow(x == (y = uint8(x)));

22:     _assertNonOverflow(x == (y = uint16(x)));

26:     _assertNonOverflow(x == (y = uint24(x)));

30:     _assertNonOverflow(x == (y = uint32(x)));

34:     _assertNonOverflow(x == (y = uint40(x)));

38:     _assertNonOverflow(x == (y = uint48(x)));

42:     _assertNonOverflow(x == (y = uint56(x)));

46:     _assertNonOverflow(x == (y = uint64(x)));

50:     _assertNonOverflow(x == (y = uint72(x)));

54:     _assertNonOverflow(x == (y = uint80(x)));

58:     _assertNonOverflow(x == (y = uint88(x)));

62:     _assertNonOverflow(x == (y = uint96(x)));

66:     _assertNonOverflow(x == (y = uint104(x)));

70:     _assertNonOverflow(x == (y = uint112(x)));

74:     _assertNonOverflow(x == (y = uint120(x)));

78:     _assertNonOverflow(x == (y = uint128(x)));

82:     _assertNonOverflow(x == (y = uint136(x)));

86:     _assertNonOverflow(x == (y = uint144(x)));

90:     _assertNonOverflow(x == (y = uint152(x)));

94:     _assertNonOverflow(x == (y = uint160(x)));

98:     _assertNonOverflow(x == (y = uint168(x)));

102:     _assertNonOverflow(x == (y = uint176(x)));

106:     _assertNonOverflow(x == (y = uint184(x)));

110:     _assertNonOverflow(x == (y = uint192(x)));

114:     _assertNonOverflow(x == (y = uint200(x)));

118:     _assertNonOverflow(x == (y = uint208(x)));

122:     _assertNonOverflow(x == (y = uint216(x)));

126:     _assertNonOverflow(x == (y = uint224(x)));

130:     _assertNonOverflow(x == (y = uint232(x)));

134:     _assertNonOverflow(x == (y = uint240(x)));

138:     _assertNonOverflow(x == (y = uint248(x)));

```

```solidity
File: market/WildcatMarket.sol

27:     MarketState memory state = _getUpdatedState();

46:     MarketState memory state = _getUpdatedState();

63:     Account memory account = _getAccountWithRole(msg.sender, AuthRole.DepositAndWithdraw);

87:     uint256 actualAmount = depositUpTo(amount);

97:     MarketState memory state = _getUpdatedState();

120:     MarketState memory state = _getUpdatedState();

143:     MarketState memory state = _getUpdatedState();

150:     uint256 currentlyHeld = totalAssets();

```

```solidity
File: market/WildcatMarketBase.sol

77:     MarketParameters memory parameters = IWildcatMarketController(msg.sender).getMarketParameters();

99:     decimals = IERC20Metadata(parameters.asset).decimals();

101:     _state = MarketState({

172:         address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(

201:     account = _getAccount(accountAddress);

319:     MarketState memory state = currentState();

335:     MarketState memory state = currentState();

510:     uint104 scaledAmountBurned = uint104(MathUtils.min(scaledAvailableLiquidity, scaledAmountOwed));

539:     uint104 scaledAmountBurned = uint104(MathUtils.min(scaledAvailableLiquidity, scaledAmountOwed));

```

```solidity
File: market/WildcatMarketConfig.sol

22:     MarketState memory state = currentState();

78:     MarketState memory state = _getUpdatedState();

116:     MarketState memory state = _getUpdatedState();

117:     Account memory account = _getAccount(_account);

135:     MarketState memory state = _getUpdatedState();

150:     MarketState memory state = _getUpdatedState();

176:     MarketState memory state = _getUpdatedState();

```

```solidity
File: market/WildcatMarketToken.sol

65:     MarketState memory state = _getUpdatedState();

72:     Account memory fromAccount = _getAccount(from);

76:     Account memory toAccount = _getAccount(to);

```

```solidity
File: market/WildcatMarketWithdrawals.sol

63:     uint256 newTotalWithdrawn = uint256(batch.normalizedAmountPaid).mulDiv(

78:     MarketState memory state = _getUpdatedState();

81:     Account memory account = _getAccountWithRole(msg.sender, AuthRole.WithdrawOnly);

95:       state.pendingWithdrawalExpiry = uint32(block.timestamp + withdrawalBatchDuration);

144:     MarketState memory state = _getUpdatedState();

151:     uint128 newTotalWithdrawn = uint128(

166:       address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(

191:     MarketState memory state = _getUpdatedState();

200:     uint256 availableLiquidity = totalAssets() -

```

### <a name="GAS-3"></a>[GAS-3] State variables should be cached in stack variables

#### Impact:
Re-reading state variables from storage during a function call can result in higher gas costs. Caching state variables in stack variables can be a more gas-efficient approach.

*Instances (70)*:
```solidity
File: WildcatMarketController.sol

98:     borrower = parameters.borrower;

99:     sentinel = parameters.sentinel;

100:     marketInitCodeStorage = parameters.marketInitCodeStorage;

101:     marketInitCodeHash = parameters.marketInitCodeHash;

102:     MinimumDelinquencyGracePeriod = parameters.minimumDelinquencyGracePeriod;

103:     MaximumDelinquencyGracePeriod = parameters.maximumDelinquencyGracePeriod;

104:     MinimumReserveRatioBips = parameters.minimumReserveRatioBips;

105:     MaximumReserveRatioBips = parameters.maximumReserveRatioBips;

106:     MinimumDelinquencyFeeBips = parameters.minimumDelinquencyFeeBips;

107:     MaximumDelinquencyFeeBips = parameters.maximumDelinquencyFeeBips;

108:     MinimumWithdrawalBatchDuration = parameters.minimumWithdrawalBatchDuration;

109:     MaximumWithdrawalBatchDuration = parameters.maximumWithdrawalBatchDuration;

110:     MinimumAnnualInterestBips = parameters.minimumAnnualInterestBips;

111:     MaximumAnnualInterestBips = parameters.maximumAnnualInterestBips;

239:     parameters.asset = _tmpMarketParameters.asset;

240:     parameters.namePrefix = _tmpMarketParameters.namePrefix;

241:     parameters.symbolPrefix = _tmpMarketParameters.symbolPrefix;

244:     parameters.feeRecipient = _tmpMarketParameters.feeRecipient;

246:     parameters.maxTotalSupply = _tmpMarketParameters.maxTotalSupply;

247:     parameters.protocolFeeBips = _tmpMarketParameters.protocolFeeBips;

248:     parameters.annualInterestBips = _tmpMarketParameters.annualInterestBips;

249:     parameters.delinquencyFeeBips = _tmpMarketParameters.delinquencyFeeBips;

250:     parameters.withdrawalBatchDuration = _tmpMarketParameters.withdrawalBatchDuration;

251:     parameters.reserveRatioBips = _tmpMarketParameters.reserveRatioBips;

252:     parameters.delinquencyGracePeriod = _tmpMarketParameters.delinquencyGracePeriod;

```

```solidity
File: WildcatMarketControllerFactory.sol

91:     MinimumDelinquencyGracePeriod = constraints.minimumDelinquencyGracePeriod;

92:     MaximumDelinquencyGracePeriod = constraints.maximumDelinquencyGracePeriod;

93:     MinimumReserveRatioBips = constraints.minimumReserveRatioBips;

94:     MaximumReserveRatioBips = constraints.maximumReserveRatioBips;

95:     MinimumDelinquencyFeeBips = constraints.minimumDelinquencyFeeBips;

96:     MaximumDelinquencyFeeBips = constraints.maximumDelinquencyFeeBips;

97:     MinimumWithdrawalBatchDuration = constraints.minimumWithdrawalBatchDuration;

98:     MaximumWithdrawalBatchDuration = constraints.maximumWithdrawalBatchDuration;

99:     MinimumAnnualInterestBips = constraints.minimumAnnualInterestBips;

100:     MaximumAnnualInterestBips = constraints.maximumAnnualInterestBips;

286:     _tmpMarketBorrowerParameter = msg.sender;

```

```solidity
File: WildcatSanctionsEscrow.sol

17:     sentinel = msg.sender;

```

```solidity
File: libraries/FIFOQueue.sol

43:     uint256 startIndex = arr.startIndex;

44:     uint256 nextIndex = arr.nextIndex;

56:     uint128 nextIndex = arr.nextIndex;

62:     uint128 startIndex = arr.startIndex;

71:     uint128 startIndex = arr.startIndex;

```

```solidity
File: libraries/FeeMath.sol

95:     uint256 previousTimeDelinquent = state.timeDelinquent;

168:     uint256 prevScaleFactor = state.scaleFactor;

```

```solidity
File: libraries/MarketState.sol

90:     uint256 scaledWithdrawals = state.scaledPendingWithdrawals;

131:     uint256 expiry = state.pendingWithdrawalExpiry;

```

```solidity
File: libraries/MathUtils.sol

7: uint256 constant HALF_BIP = 0.5e4;

10: uint256 constant HALF_RAY = 0.5e27;

```

```solidity
File: market/WildcatMarketBase.sol

96:     asset = parameters.asset;

117:     sentinel = parameters.sentinel;

118:     borrower = parameters.borrower;

119:     controller = parameters.controller;

120:     feeRecipient = parameters.feeRecipient;

121:     protocolFeeBips = parameters.protocolFeeBips;

122:     delinquencyFeeBips = parameters.delinquencyFeeBips;

123:     delinquencyGracePeriod = parameters.delinquencyGracePeriod;

124:     withdrawalBatchDuration = parameters.withdrawalBatchDuration;

166:       uint104 scaledBalance = account.scaledBalance;

167:       account.approval = AuthRole.Blocked;

205:         account.approval = AuthRole.DepositAndWithdraw;

336:     uint256 apr = state.annualInterestBips;

362:       uint256 expiry = state.pendingWithdrawalExpiry;

411:       expiredBatchExpiry = state.pendingWithdrawalExpiry;

467:     uint32 expiry = state.pendingWithdrawalExpiry;

```

```solidity
File: market/WildcatMarketConfig.sol

98:     account.approval = AuthRole.Null;

119:       account.approval = AuthRole.DepositAndWithdraw;

121:       account.approval = AuthRole.WithdrawOnly;

178:     uint256 initialReserveRatioBips = state.reserveRatioBips;

```

```solidity
File: market/WildcatMarketWithdrawals.sol

62:     uint256 previousTotalWithdrawn = status.normalizedAmountWithdrawn;

99:     uint32 expiry = state.pendingWithdrawalExpiry;

```

### <a name="GAS-4"></a>[GAS-4] Function Call Caching

#### Impact:
Caching function results reduces gas consumption by avoiding repeated calls.

*Instances (155)*:
```solidity
File: WildcatArchController.sol

42:     if (!_controllerFactories.contains(msg.sender)) {

49:     if (!_controllers.contains(msg.sender)) {

64:     if (!_borrowers.add(borrower)) {

71:     if (!_borrowers.remove(borrower)) {

78:     return _borrowers.contains(borrower);

82:     return _borrowers.values();

89:     uint256 len = _borrowers.length();

90:     end = MathUtils.min(end, len);

94:       arr[i] = _borrowers.at(start + i);

99:     return _borrowers.length();

107:     if (!_controllerFactories.add(factory)) {

114:     if (!_controllerFactories.remove(factory)) {

121:     return _controllerFactories.contains(factory);

125:     return _controllerFactories.values();

132:     uint256 len = _controllerFactories.length();

133:     end = MathUtils.min(end, len);

137:       arr[i] = _controllerFactories.at(start + i);

142:     return _controllerFactories.length();

150:     if (!_controllers.add(controller)) {

157:     if (!_controllers.remove(controller)) {

164:     return _controllers.contains(controller);

168:     return _controllers.values();

175:     uint256 len = _controllers.length();

176:     end = MathUtils.min(end, len);

180:       arr[i] = _controllers.at(start + i);

185:     return _controllers.length();

193:     if (!_markets.add(market)) {

200:     if (!_markets.remove(market)) {

207:     return _markets.contains(market);

211:     return _markets.values();

218:     uint256 len = _markets.length();

219:     end = MathUtils.min(end, len);

223:       arr[i] = _markets.at(start + i);

228:     return _markets.length();

```

```solidity
File: WildcatMarketController.sol

53:   uint256 internal immutable ownCreate2Prefix = LibStoredInitCode.getCreate2Prefix(address(this));

88:     if (!_controlledMarkets.contains(market)) {

96:     MarketControllerParameters memory parameters = controllerFactory.getMarketControllerParameters();

122:     return _authorizedLenders.values();

129:     uint256 len = _authorizedLenders.length();

130:     end = MathUtils.min(end, len);

134:       arr[i] = _authorizedLenders.at(start + i);

139:     return _authorizedLenders.length();

143:     return _authorizedLenders.contains(lender);

156:       if (_authorizedLenders.add(lender)) {

172:       if (_authorizedLenders.remove(lender)) {

185:       if (!_controlledMarkets.contains(market)) {

188:       WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));

197:     return _controlledMarkets.contains(market);

201:     return _controlledMarkets.values();

208:     uint256 len = _controlledMarkets.length();

209:     end = MathUtils.min(end, len);

213:       arr[i] = _controlledMarkets.at(start + i);

218:     return _controlledMarkets.length();

227:     return LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt, marketInitCodeHash);

303:       if (!archController.isRegisteredBorrower(msg.sender)) {

341:     ) = controllerFactory.getProtocolFeeConfiguration();

346:       originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);

350:     market = LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt, marketInitCodeHash);

354:     LibStoredInitCode.create2WithStoredInitCode(marketInitCodeStorage, salt);

356:     archController.registerMarket(market);

357:     _controlledMarkets.add(market);

```

```solidity
File: WildcatMarketControllerFactory.sol

44:   uint256 internal immutable ownCreate2Prefix = LibStoredInitCode.getCreate2Prefix(address(this));

66:     if (msg.sender != archController.owner()) {

113:     initCodeStorage = LibStoredInitCode.deployInitCode(controllerInitCode);

123:     initCodeStorage = LibStoredInitCode.deployInitCode(marketInitCode);

127:     return _deployedControllers.contains(controller);

131:     return _deployedControllers.length();

135:     return _deployedControllers.values();

142:     uint256 len = _deployedControllers.length();

143:     end = MathUtils.min(end, len);

147:       arr[i] = _deployedControllers.at(start + i);

283:     if (!archController.isRegisteredBorrower(msg.sender)) {

289:     controller = LibStoredInitCode.calculateCreate2Address(

297:     LibStoredInitCode.create2WithStoredInitCode(controllerInitCodeStorage, salt);

299:     archController.registerController(controller);

300:     _deployedControllers.add(controller);

346:       LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt, controllerInitCodeHash);

```

```solidity
File: WildcatSanctionsSentinel.sol

75:               abi.encodePacked(

78:                 keccak256(abi.encode(borrower, account, asset)),

110:     new WildcatSanctionsEscrow{ salt: keccak256(abi.encode(borrower, account, asset)) }();

```

```solidity
File: libraries/FeeMath.sol

23:     uint256 rate = rateBip.bipToRay();

34:     baseInterestRay = MathUtils.calculateLinearInterestFromBips(

46:     uint256 protocolFeeRay = protocolFeeBips.bipMul(baseInterestRay);

104:       uint256 secondsRemainingWithoutPenalty = delinquencyGracePeriod.satSub(

110:       return timeDelta.satSub(secondsRemainingWithoutPenalty);

115:     state.timeDelinquent = previousTimeDelinquent.satSub(timeDelta).toUint32();

119:     uint256 secondsRemainingWithPenalty = previousTimeDelinquent.satSub(delinquencyGracePeriod);

122:     return MathUtils.min(secondsRemainingWithPenalty, timeDelta);

153:     baseInterestRay = state.calculateBaseInterest(timestamp);

156:       protocolFee = state.applyProtocolFee(baseInterestRay, protocolFeeBips);

160:       delinquencyFeeRay = state.updateDelinquency(

169:     uint256 scaleFactorDelta = prevScaleFactor.rayMul(baseInterestRay + delinquencyFeeRay);

```

```solidity
File: libraries/MarketState.sol

52:     return state.normalizeAmount(state.scaledTotalSupply);

60:     return uint256(state.maxTotalSupply).satSub(state.totalSupply());

70:     return amount.rayMul(state.scaleFactor);

77:     return amount.rayDiv(state.scaleFactor);

95:       state.normalizeAmount(scaledRequiredReserves) +

110:     return uint128(MathUtils.min(totalAvailableAssets, state.accruedProtocolFees));

127:     return totalAssets.satSub(state.liquidityRequired());

127:     return totalAssets.satSub(state.liquidityRequired());

140:       state.normalizeAmount(state.scaledTotalSupply) +

```

```solidity
File: libraries/MathUtils.sol

34:     uint256 rate = rateBip.bipToRay();

```

```solidity
File: libraries/StringQuery.sol

72:       uint256 sizeInBits = 255 - value.ffs();

```

```solidity
File: libraries/Withdrawal.sol

54:     uint256 priorScaledAmountPending = (state.scaledPendingWithdrawals - batch.scaledOwedAmount());

56:       state.normalizeAmount(priorScaledAmountPending) +

58:     return totalAssets.satSub(unavailableAssets);

```

```solidity
File: market/WildcatMarket.sol

53:     amount = MathUtils.min(amount, state.maximumDeposit());

53:     amount = MathUtils.min(amount, state.maximumDeposit());

56:     uint104 scaledAmount = state.scaleAmount(amount).toUint104();

60:     asset.safeTransferFrom(msg.sender, address(this), amount);

101:     uint128 withdrawableFees = state.withdrawableProtocolFees(totalAssets());

107:     asset.safeTransfer(feeRecipient, withdrawableFees);

124:     uint256 borrowable = state.borrowableAssets(totalAssets());

129:     asset.safeTransfer(msg.sender, amount);

147:     if (_withdrawalData.unpaidBatches.length() > 0) {

151:     uint256 totalDebts = state.totalDebts();

154:       asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);

157:       asset.safeTransfer(borrower, currentlyHeld - totalDebts);

```

```solidity
File: market/WildcatMarketBase.sol

97:     name = string.concat(parameters.namePrefix, queryName(parameters.asset));

98:     symbol = string.concat(parameters.symbolPrefix, querySymbol(parameters.asset));

177:         emit Transfer(accountAddress, escrow, state.normalizeAmount(scaledBalance));

182:           state.normalizeAmount(scaledBalance)

321:     uint256 apr = MathUtils.bipToRay(state.annualInterestBips).bipMul(BIP + protocolFeeBips);

323:       apr += MathUtils.bipToRay(delinquencyFeeBips);

340:     return MathUtils.bipToRay(apr);

361:     if (state.hasPendingExpiredBatch()) {

410:     if (state.hasPendingExpiredBatch()) {

415:         state.updateScaleFactorAndFees(

424:       uint256 availableLiquidity = expiredBatch.availableLiquidityForPendingBatch(

435:       state.updateScaleFactorAndFees(

449:     bool isDelinquent = state.liquidityRequired() > totalAssets();

471:     uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());

484:       _withdrawalData.unpaidBatches.push(expiry);

504:     uint104 scaledAvailableLiquidity = state.scaleAmount(availableLiquidity).toUint104();

510:     uint104 scaledAmountBurned = uint104(MathUtils.min(scaledAvailableLiquidity, scaledAmountOwed));

511:     uint128 normalizedAmountPaid = state.normalizeAmount(scaledAmountBurned).toUint128();

533:     uint104 scaledAvailableLiquidity = state.scaleAmount(availableLiquidity).toUint104();

539:     uint104 scaledAmountBurned = uint104(MathUtils.min(scaledAvailableLiquidity, scaledAmountOwed));

540:     uint128 normalizedAmountPaid = state.normalizeAmount(scaledAmountBurned).toUint128();

```

```solidity
File: market/WildcatMarketConfig.sol

23:     return state.maximumDeposit();

137:     if (_maxTotalSupply < state.totalSupply()) {

141:     state.maxTotalSupply = _maxTotalSupply.toUint128();

181:       if (state.liquidityRequired() > totalAssets()) {

187:       if (state.liquidityRequired() > totalAssets()) {

```

```solidity
File: market/WildcatMarketToken.sol

18:     return state.normalizeAmount(_accounts[account].scaledBalance);

24:     return state.totalSupply();

66:     uint104 scaledAmount = state.scaleAmount(amount).toUint104();

```

```solidity
File: market/WildcatMarketWithdrawals.sol

25:     return _withdrawalData.unpaidBatches.values();

83:     uint104 scaledAmount = state.scaleAmount(amount).toUint104();

111:     uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());

152:       MathUtils.mulDiv(batch.normalizedAmountPaid, status.scaledAmount, batch.scaledTotalAmount)

171:       asset.safeTransfer(escrow, normalizedAmountWithdrawn);

179:       asset.safeTransfer(accountAddress, normalizedAmountWithdrawn);

194:     uint32 expiry = _withdrawalData.unpaidBatches.first();

207:       _withdrawalData.unpaidBatches.shift();

```

### <a name="GAS-5"></a>[GAS-5] State variables only set in the constructor should be declared immutable

#### Impact:
Avoids a Gsset (20000 gas) in the constructor and reduces gas costs for state variable accesses.

*Instances (4)*:
```solidity
File: ReentrancyGuard.sol

49:   constructor() {

```

```solidity
File: WildcatMarketControllerFactory.sol

72:   constructor(

```

```solidity
File: WildcatSanctionsSentinel.sol

24:   constructor(address _archController, address _chainalysisSanctionsList) {

```

```solidity
File: market/WildcatMarketBase.sol

76:   constructor() {

```

### <a name="GAS-6"></a>[GAS-6] Optimize gas costs using >= and <= operators

#### Impact:
Using >= and <= operators can save gas compared to > and < operators.

*Instances (75)*:
```solidity
File: ReentrancyGuard.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: WildcatArchController.sol

2: pragma solidity >=0.8.20;

93:     for (uint256 i = 0; i < count; i++) {

136:     for (uint256 i = 0; i < count; i++) {

179:     for (uint256 i = 0; i < count; i++) {

222:     for (uint256 i = 0; i < count; i++) {

```

```solidity
File: WildcatMarketController.sol

2: pragma solidity >=0.8.20;

133:     for (uint256 i = 0; i < count; i++) {

154:     for (uint256 i = 0; i < lenders.length; i++) {

170:     for (uint256 i = 0; i < lenders.length; i++) {

183:     for (uint256 i; i < markets.length; i++) {

212:     for (uint256 i = 0; i < count; i++) {

474:     if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {

495:     if (block.timestamp < tmp.expiry) {

```

```solidity
File: WildcatMarketControllerFactory.sol

2: pragma solidity >=0.8.20;

80:       constraints.minimumAnnualInterestBips > constraints.maximumAnnualInterestBips ||

81:       constraints.maximumAnnualInterestBips > 10000 ||

82:       constraints.minimumDelinquencyFeeBips > constraints.maximumDelinquencyFeeBips ||

83:       constraints.maximumDelinquencyFeeBips > 10000 ||

84:       constraints.minimumReserveRatioBips > constraints.maximumReserveRatioBips ||

85:       constraints.maximumReserveRatioBips > 10000 ||

86:       constraints.minimumDelinquencyGracePeriod > constraints.maximumDelinquencyGracePeriod ||

87:       constraints.minimumWithdrawalBatchDuration > constraints.maximumWithdrawalBatchDuration

146:     for (uint256 i = 0; i < count; i++) {

201:     bool hasOriginationFee = originationFeeAmount > 0;

205:       (protocolFeeBips > 0 && nullFeeRecipient) ||

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

32:     if (index >= arr.nextIndex) {

48:     for (uint256 i = 0; i < len; i++) {

72:     if (startIndex + n > arr.nextIndex) {

75:     for (uint256 i = 0; i < n; i++) {

```

```solidity
File: libraries/FeeMath.sol

2: pragma solidity >=0.8.20;

67:     if (timeWithPenalty > 0) {

155:     if (protocolFeeBips > 0) {

159:     if (delinquencyFeeBips > 0) {

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

45:     c = ternary(a < b, a, b);

52:     c = ternary(a < b, b, a);

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

125:     if (amount > borrowable) {

152:     if (currentlyHeld < totalDebts) {

155:     } else if (currentlyHeld > totalDebts) {

```

```solidity
File: market/WildcatMarketBase.sol

2: pragma solidity >=0.8.20;

79:     if ((parameters.protocolFeeBips > 0).and(parameters.feeRecipient == address(0))) {

82:     if (parameters.annualInterestBips > BIP) {

85:     if (parameters.reserveRatioBips > BIP) {

88:     if (parameters.protocolFeeBips > BIP) {

91:     if (parameters.delinquencyFeeBips > BIP) {

170:       if (scaledBalance > 0) {

322:     if (state.timeDelinquent > delinquencyGracePeriod) {

337:     if (state.timeDelinquent > delinquencyGracePeriod) {

428:       if (availableLiquidity > 0) {

472:     if (availableLiquidity > 0) {

483:     if (batch.scaledAmountBurned < batch.scaledTotalAmount) {

```

```solidity
File: market/WildcatMarketConfig.sol

2: pragma solidity >=0.8.20;

137:     if (_maxTotalSupply < state.totalSupply()) {

152:     if (_annualInterestBips > BIP) {

172:     if (_reserveRatioBips > BIP) {

180:     if (_reserveRatioBips < initialReserveRatioBips) {

186:     if (_reserveRatioBips > initialReserveRatioBips) {

```

```solidity
File: market/WildcatMarketToken.sol

2: pragma solidity >=0.8.20;

```

```solidity
File: market/WildcatMarketWithdrawals.sol

2: pragma solidity >=0.8.20;

32:     if ((expiry == expiredBatchExpiry).and(expiry > 0)) {

49:     if (expiry > block.timestamp) {

112:     if (availableLiquidity > 0) {

141:     if (expiry > block.timestamp) {

```

### <a name="GAS-7"></a>[GAS-7] Using private for constants

#### Impact:
Using private for constants can save gas in deployment and is preferable for public constants.

*Instances (1)*:
```solidity
File: market/WildcatMarketBase.sol

24:   string public constant version = '1.0';

```

### <a name="GAS-8"></a>[GAS-8] Simple checks for zero can be done using assembly to save gas

#### Impact:
Using assembly for simple zero checks can save gas compared to the equivalent high-level Solidity code.

*Instances (1)*:
```solidity
File: market/WildcatMarket.sol

57:     if (scaledAmount == 0) revert NullMintAmount();

```

### <a name="GAS-9"></a>[GAS-9] State variables should be cached in stack variables

#### Impact:
Re-reading state variables from storage during a function call can result in higher gas costs. Caching state variables in stack variables can be a more gas-efficient approach.

*Instances (70)*:
```solidity
File: WildcatMarketController.sol

98:     borrower = parameters.borrower;

99:     sentinel = parameters.sentinel;

100:     marketInitCodeStorage = parameters.marketInitCodeStorage;

101:     marketInitCodeHash = parameters.marketInitCodeHash;

102:     MinimumDelinquencyGracePeriod = parameters.minimumDelinquencyGracePeriod;

103:     MaximumDelinquencyGracePeriod = parameters.maximumDelinquencyGracePeriod;

104:     MinimumReserveRatioBips = parameters.minimumReserveRatioBips;

105:     MaximumReserveRatioBips = parameters.maximumReserveRatioBips;

106:     MinimumDelinquencyFeeBips = parameters.minimumDelinquencyFeeBips;

107:     MaximumDelinquencyFeeBips = parameters.maximumDelinquencyFeeBips;

108:     MinimumWithdrawalBatchDuration = parameters.minimumWithdrawalBatchDuration;

109:     MaximumWithdrawalBatchDuration = parameters.maximumWithdrawalBatchDuration;

110:     MinimumAnnualInterestBips = parameters.minimumAnnualInterestBips;

111:     MaximumAnnualInterestBips = parameters.maximumAnnualInterestBips;

239:     parameters.asset = _tmpMarketParameters.asset;

240:     parameters.namePrefix = _tmpMarketParameters.namePrefix;

241:     parameters.symbolPrefix = _tmpMarketParameters.symbolPrefix;

244:     parameters.feeRecipient = _tmpMarketParameters.feeRecipient;

246:     parameters.maxTotalSupply = _tmpMarketParameters.maxTotalSupply;

247:     parameters.protocolFeeBips = _tmpMarketParameters.protocolFeeBips;

248:     parameters.annualInterestBips = _tmpMarketParameters.annualInterestBips;

249:     parameters.delinquencyFeeBips = _tmpMarketParameters.delinquencyFeeBips;

250:     parameters.withdrawalBatchDuration = _tmpMarketParameters.withdrawalBatchDuration;

251:     parameters.reserveRatioBips = _tmpMarketParameters.reserveRatioBips;

252:     parameters.delinquencyGracePeriod = _tmpMarketParameters.delinquencyGracePeriod;

```

```solidity
File: WildcatMarketControllerFactory.sol

91:     MinimumDelinquencyGracePeriod = constraints.minimumDelinquencyGracePeriod;

92:     MaximumDelinquencyGracePeriod = constraints.maximumDelinquencyGracePeriod;

93:     MinimumReserveRatioBips = constraints.minimumReserveRatioBips;

94:     MaximumReserveRatioBips = constraints.maximumReserveRatioBips;

95:     MinimumDelinquencyFeeBips = constraints.minimumDelinquencyFeeBips;

96:     MaximumDelinquencyFeeBips = constraints.maximumDelinquencyFeeBips;

97:     MinimumWithdrawalBatchDuration = constraints.minimumWithdrawalBatchDuration;

98:     MaximumWithdrawalBatchDuration = constraints.maximumWithdrawalBatchDuration;

99:     MinimumAnnualInterestBips = constraints.minimumAnnualInterestBips;

100:     MaximumAnnualInterestBips = constraints.maximumAnnualInterestBips;

286:     _tmpMarketBorrowerParameter = msg.sender;

```

```solidity
File: WildcatSanctionsEscrow.sol

17:     sentinel = msg.sender;

```

```solidity
File: libraries/FIFOQueue.sol

43:     uint256 startIndex = arr.startIndex;

44:     uint256 nextIndex = arr.nextIndex;

56:     uint128 nextIndex = arr.nextIndex;

62:     uint128 startIndex = arr.startIndex;

71:     uint128 startIndex = arr.startIndex;

```

```solidity
File: libraries/FeeMath.sol

95:     uint256 previousTimeDelinquent = state.timeDelinquent;

168:     uint256 prevScaleFactor = state.scaleFactor;

```

```solidity
File: libraries/MarketState.sol

90:     uint256 scaledWithdrawals = state.scaledPendingWithdrawals;

131:     uint256 expiry = state.pendingWithdrawalExpiry;

```

```solidity
File: libraries/MathUtils.sol

7: uint256 constant HALF_BIP = 0.5e4;

10: uint256 constant HALF_RAY = 0.5e27;

```

```solidity
File: market/WildcatMarketBase.sol

96:     asset = parameters.asset;

117:     sentinel = parameters.sentinel;

118:     borrower = parameters.borrower;

119:     controller = parameters.controller;

120:     feeRecipient = parameters.feeRecipient;

121:     protocolFeeBips = parameters.protocolFeeBips;

122:     delinquencyFeeBips = parameters.delinquencyFeeBips;

123:     delinquencyGracePeriod = parameters.delinquencyGracePeriod;

124:     withdrawalBatchDuration = parameters.withdrawalBatchDuration;

166:       uint104 scaledBalance = account.scaledBalance;

167:       account.approval = AuthRole.Blocked;

205:         account.approval = AuthRole.DepositAndWithdraw;

336:     uint256 apr = state.annualInterestBips;

362:       uint256 expiry = state.pendingWithdrawalExpiry;

411:       expiredBatchExpiry = state.pendingWithdrawalExpiry;

467:     uint32 expiry = state.pendingWithdrawalExpiry;

```

```solidity
File: market/WildcatMarketConfig.sol

98:     account.approval = AuthRole.Null;

119:       account.approval = AuthRole.DepositAndWithdraw;

121:       account.approval = AuthRole.WithdrawOnly;

178:     uint256 initialReserveRatioBips = state.reserveRatioBips;

```

```solidity
File: market/WildcatMarketWithdrawals.sol

62:     uint256 previousTotalWithdrawn = status.normalizedAmountWithdrawn;

99:     uint32 expiry = state.pendingWithdrawalExpiry;

```

### <a name="GAS-10"></a>[GAS-10] Upgrade to Solidity 0.8.19 or Later for Gas Optimization

#### Impact:
Upgrading to Solidity 0.8.19 or a later version may optimize gas usage.

*Instances (1)*:
```solidity
File: libraries/Chainalysis.sol

2: pragma solidity ^0.8.20;

```

### <a name="GAS-11"></a>[GAS-11] Use assembly to check zero

#### Impact:
Using assembly to check for zero can save gas by allowing more direct access to the EVM and reducing the overhead associated with high-level operations in Solidity.

*Instances (11)*:
```solidity
File: WildcatMarketController.sol

477:       if (tmp.expiry == 0) {

492:     if (tmp.expiry == 0) {

```

```solidity
File: market/WildcatMarket.sol

57:     if (scaledAmount == 0) revert NullMintAmount();

98:     if (state.accruedProtocolFees == 0) {

102:     if (withdrawableFees == 0) {

```

```solidity
File: market/WildcatMarketBase.sol

507:     if (scaledAmountOwed == 0) {

536:     if (scaledAmountOwed == 0) {

```

```solidity
File: market/WildcatMarketToken.sol

68:     if (scaledAmount == 0) {

```

```solidity
File: market/WildcatMarketWithdrawals.sol

84:     if (scaledAmount == 0) {

94:     if (state.pendingWithdrawalExpiry == 0) {

160:     if (normalizedAmountWithdrawn == 0) {

```

### <a name="GAS-12"></a>[GAS-12] Use assembly to emit events, in order to save gas

#### Impact:
Using assembly to emit events can save gas by making use of scratch space and free memory pointers, avoiding memory expansion costs.

*Instances (44)*:
```solidity
File: WildcatArchController.sol

67:     emit BorrowerAdded(borrower);

74:     emit BorrowerRemoved(borrower);

110:     emit ControllerFactoryAdded(factory);

117:     emit ControllerFactoryRemoved(factory);

153:     emit ControllerAdded(msg.sender, controller);

160:     emit ControllerRemoved(controller);

196:     emit MarketAdded(msg.sender, market);

203:     emit MarketRemoved(market);

```

```solidity
File: WildcatMarketController.sol

157:         emit LenderAuthorized(lender);

173:         emit LenderDeauthorized(lender);

```

```solidity
File: WildcatSanctionsEscrow.sol

40:     emit EscrowReleased(account, asset, amount);

```

```solidity
File: WildcatSanctionsSentinel.sol

50:     emit SanctionOverride(msg.sender, account);

58:     emit SanctionOverrideRemoved(msg.sender, account);

112:     emit NewSanctionsEscrow(borrower, account, asset);

116:     emit SanctionOverride(borrower, escrowContract);

```

```solidity
File: market/WildcatMarket.sol

67:     emit Transfer(address(0), msg.sender, amount);

68:     emit Deposit(msg.sender, amount, scaledAmount);

108:     emit FeesCollected(withdrawableFees);

130:     emit Borrow(amount);

160:     emit MarketClosed(block.timestamp);

```

```solidity
File: market/WildcatMarketBase.sol

168:       emit AuthorizationStatusUpdated(accountAddress, AuthRole.Blocked);

177:         emit Transfer(accountAddress, escrow, state.normalizeAmount(scaledBalance));

179:         emit SanctionedAccountAssetsSentToEscrow(

206:         emit AuthorizationStatusUpdated(accountAddress, AuthRole.DepositAndWithdraw);

373:         emit ScaleFactorUpdated(state.scaleFactor, baseInterestRay, delinquencyFeeRay, protocolFee);

386:       emit ScaleFactorUpdated(state.scaleFactor, baseInterestRay, delinquencyFeeRay, protocolFee);

452:     emit StateUpdated(state.scaleFactor, isDelinquent);

476:     emit WithdrawalBatchExpired(

486:       emit WithdrawalBatchClosed(expiry);

524:     emit Transfer(address(this), address(0), normalizedAmountPaid);

525:     emit WithdrawalBatchPayment(expiry, scaledAmountBurned, normalizedAmountPaid);

```

```solidity
File: market/WildcatMarketConfig.sol

99:     emit AuthorizationStatusUpdated(accountAddress, account.approval);

125:     emit AuthorizationStatusUpdated(_account, account.approval);

143:     emit MaxTotalSupplyUpdated(_maxTotalSupply);

158:     emit AnnualInterestBipsUpdated(_annualInterestBips);

192:     emit ReserveRatioBipsUpdated(_reserveRatioBips);

```

```solidity
File: market/WildcatMarketToken.sol

61:     emit Approval(approver, spender, amount);

81:     emit Transfer(from, to, amount);

```

```solidity
File: market/WildcatMarketWithdrawals.sol

91:     emit Transfer(msg.sender, address(this), amount);

96:       emit WithdrawalBatchCreated(state.pendingWithdrawalExpiry);

108:     emit WithdrawalQueued(expiry, msg.sender, scaledAmount);

172:       emit SanctionedAccountWithdrawalSentToEscrow(

182:     emit WithdrawalExecuted(expiry, accountAddress, normalizedAmountWithdrawn);

208:       emit WithdrawalBatchClosed(expiry);

```

### <a name="GAS-13"></a>[GAS-13] Use assembly to write address/contract type storage values

#### Impact:
Using assembly to write address or contract type storage values can save gas.

*Instances (292)*:
```solidity
File: ReentrancyGuard.sol

22:   uint256 private constant _NOT_ENTERED = 1;

23:   uint256 private constant _ENTERED = 2;

51:     _reentrancyGuard = _NOT_ENTERED;

65:       _reentrancyGuard = _ENTERED;

74:     _reentrancyGuard = _NOT_ENTERED;

```

```solidity
File: WildcatArchController.sol

89:     uint256 len = _borrowers.length();

93:     for (uint256 i = 0; i < count; i++) {

132:     uint256 len = _controllerFactories.length();

136:     for (uint256 i = 0; i < count; i++) {

175:     uint256 len = _controllers.length();

179:     for (uint256 i = 0; i < count; i++) {

218:     uint256 len = _markets.length();

222:     for (uint256 i = 0; i < count; i++) {

```

```solidity
File: WildcatMarketController.sol

53:   uint256 internal immutable ownCreate2Prefix = LibStoredInitCode.getCreate2Prefix(address(this));

95:     controllerFactory = IWildcatMarketControllerFactory(msg.sender);

96:     MarketControllerParameters memory parameters = controllerFactory.getMarketControllerParameters();

97:     archController = IWildcatArchController(parameters.archController);

98:     borrower = parameters.borrower;

99:     sentinel = parameters.sentinel;

100:     marketInitCodeStorage = parameters.marketInitCodeStorage;

101:     marketInitCodeHash = parameters.marketInitCodeHash;

102:     MinimumDelinquencyGracePeriod = parameters.minimumDelinquencyGracePeriod;

103:     MaximumDelinquencyGracePeriod = parameters.maximumDelinquencyGracePeriod;

104:     MinimumReserveRatioBips = parameters.minimumReserveRatioBips;

105:     MaximumReserveRatioBips = parameters.maximumReserveRatioBips;

106:     MinimumDelinquencyFeeBips = parameters.minimumDelinquencyFeeBips;

107:     MaximumDelinquencyFeeBips = parameters.maximumDelinquencyFeeBips;

108:     MinimumWithdrawalBatchDuration = parameters.minimumWithdrawalBatchDuration;

109:     MaximumWithdrawalBatchDuration = parameters.maximumWithdrawalBatchDuration;

110:     MinimumAnnualInterestBips = parameters.minimumAnnualInterestBips;

111:     MaximumAnnualInterestBips = parameters.maximumAnnualInterestBips;

129:     uint256 len = _authorizedLenders.length();

133:     for (uint256 i = 0; i < count; i++) {

154:     for (uint256 i = 0; i < lenders.length; i++) {

155:       address lender = lenders[i];

170:     for (uint256 i = 0; i < lenders.length; i++) {

171:       address lender = lenders[i];

184:       address market = markets[i];

208:     uint256 len = _controlledMarkets.length();

212:     for (uint256 i = 0; i < count; i++) {

239:     parameters.asset = _tmpMarketParameters.asset;

240:     parameters.namePrefix = _tmpMarketParameters.namePrefix;

241:     parameters.symbolPrefix = _tmpMarketParameters.symbolPrefix;

242:     parameters.borrower = borrower;

243:     parameters.controller = address(this);

244:     parameters.feeRecipient = _tmpMarketParameters.feeRecipient;

245:     parameters.sentinel = sentinel;

246:     parameters.maxTotalSupply = _tmpMarketParameters.maxTotalSupply;

247:     parameters.protocolFeeBips = _tmpMarketParameters.protocolFeeBips;

248:     parameters.annualInterestBips = _tmpMarketParameters.annualInterestBips;

249:     parameters.delinquencyFeeBips = _tmpMarketParameters.delinquencyFeeBips;

250:     parameters.withdrawalBatchDuration = _tmpMarketParameters.withdrawalBatchDuration;

251:     parameters.reserveRatioBips = _tmpMarketParameters.reserveRatioBips;

252:     parameters.delinquencyGracePeriod = _tmpMarketParameters.delinquencyGracePeriod;

256:     _tmpMarketParameters.asset = address(1);

257:     _tmpMarketParameters.namePrefix = '_';

258:     _tmpMarketParameters.symbolPrefix = '_';

259:     _tmpMarketParameters.feeRecipient = address(1);

260:     _tmpMarketParameters.protocolFeeBips = 1;

261:     _tmpMarketParameters.maxTotalSupply = 1;

262:     _tmpMarketParameters.annualInterestBips = 1;

263:     _tmpMarketParameters.delinquencyFeeBips = 1;

264:     _tmpMarketParameters.withdrawalBatchDuration = 1;

265:     _tmpMarketParameters.reserveRatioBips = 1;

266:     _tmpMarketParameters.delinquencyGracePeriod = 1;

343:     _tmpMarketParameters = parameters;

451:     constraints.minimumDelinquencyGracePeriod = MinimumDelinquencyGracePeriod;

452:     constraints.maximumDelinquencyGracePeriod = MaximumDelinquencyGracePeriod;

453:     constraints.minimumReserveRatioBips = MinimumReserveRatioBips;

454:     constraints.maximumReserveRatioBips = MaximumReserveRatioBips;

455:     constraints.minimumDelinquencyFeeBips = MinimumDelinquencyFeeBips;

456:     constraints.maximumDelinquencyFeeBips = MaximumDelinquencyFeeBips;

457:     constraints.minimumWithdrawalBatchDuration = MinimumWithdrawalBatchDuration;

458:     constraints.maximumWithdrawalBatchDuration = MaximumWithdrawalBatchDuration;

459:     constraints.minimumAnnualInterestBips = MinimumAnnualInterestBips;

460:     constraints.maximumAnnualInterestBips = MaximumAnnualInterestBips;

475:       TemporaryReserveRatio storage tmp = temporaryExcessReserveRatio[market];

478:         tmp.reserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips());

491:     TemporaryReserveRatio memory tmp = temporaryExcessReserveRatio[market];

```

```solidity
File: WildcatMarketControllerFactory.sol

44:   uint256 internal immutable ownCreate2Prefix = LibStoredInitCode.getCreate2Prefix(address(this));

77:     archController = IWildcatArchController(_archController);

78:     sentinel = _sentinel;

91:     MinimumDelinquencyGracePeriod = constraints.minimumDelinquencyGracePeriod;

92:     MaximumDelinquencyGracePeriod = constraints.maximumDelinquencyGracePeriod;

93:     MinimumReserveRatioBips = constraints.minimumReserveRatioBips;

94:     MaximumReserveRatioBips = constraints.maximumReserveRatioBips;

95:     MinimumDelinquencyFeeBips = constraints.minimumDelinquencyFeeBips;

96:     MaximumDelinquencyFeeBips = constraints.maximumDelinquencyFeeBips;

97:     MinimumWithdrawalBatchDuration = constraints.minimumWithdrawalBatchDuration;

98:     MaximumWithdrawalBatchDuration = constraints.maximumWithdrawalBatchDuration;

99:     MinimumAnnualInterestBips = constraints.minimumAnnualInterestBips;

100:     MaximumAnnualInterestBips = constraints.maximumAnnualInterestBips;

111:     bytes memory controllerInitCode = type(WildcatMarketController).creationCode;

112:     initCodeHash = uint256(keccak256(controllerInitCode));

113:     initCodeStorage = LibStoredInitCode.deployInitCode(controllerInitCode);

121:     bytes memory marketInitCode = type(WildcatMarket).creationCode;

122:     initCodeHash = uint256(keccak256(marketInitCode));

123:     initCodeStorage = LibStoredInitCode.deployInitCode(marketInitCode);

142:     uint256 len = _deployedControllers.length();

146:     for (uint256 i = 0; i < count; i++) {

228:     constraints.minimumDelinquencyGracePeriod = MinimumDelinquencyGracePeriod;

229:     constraints.maximumDelinquencyGracePeriod = MaximumDelinquencyGracePeriod;

230:     constraints.minimumReserveRatioBips = MinimumReserveRatioBips;

231:     constraints.maximumReserveRatioBips = MaximumReserveRatioBips;

232:     constraints.minimumDelinquencyFeeBips = MinimumDelinquencyFeeBips;

233:     constraints.maximumDelinquencyFeeBips = MaximumDelinquencyFeeBips;

234:     constraints.minimumWithdrawalBatchDuration = MinimumWithdrawalBatchDuration;

235:     constraints.maximumWithdrawalBatchDuration = MaximumWithdrawalBatchDuration;

236:     constraints.minimumAnnualInterestBips = MinimumAnnualInterestBips;

237:     constraints.maximumAnnualInterestBips = MaximumAnnualInterestBips;

244:   address internal _tmpMarketBorrowerParameter = address(1);

252:     parameters.archController = address(archController);

253:     parameters.borrower = _tmpMarketBorrowerParameter;

254:     parameters.sentinel = sentinel;

255:     parameters.marketInitCodeStorage = marketInitCodeStorage;

256:     parameters.marketInitCodeHash = marketInitCodeHash;

257:     parameters.minimumDelinquencyGracePeriod = MinimumDelinquencyGracePeriod;

258:     parameters.maximumDelinquencyGracePeriod = MaximumDelinquencyGracePeriod;

259:     parameters.minimumReserveRatioBips = MinimumReserveRatioBips;

260:     parameters.maximumReserveRatioBips = MaximumReserveRatioBips;

261:     parameters.minimumDelinquencyFeeBips = MinimumDelinquencyFeeBips;

262:     parameters.maximumDelinquencyFeeBips = MaximumDelinquencyFeeBips;

263:     parameters.minimumWithdrawalBatchDuration = MinimumWithdrawalBatchDuration;

264:     parameters.maximumWithdrawalBatchDuration = MaximumWithdrawalBatchDuration;

265:     parameters.minimumAnnualInterestBips = MinimumAnnualInterestBips;

266:     parameters.maximumAnnualInterestBips = MaximumAnnualInterestBips;

286:     _tmpMarketBorrowerParameter = msg.sender;

288:     bytes32 salt = bytes32(uint256(uint160(msg.sender)));

298:     _tmpMarketBorrowerParameter = address(1);

328:     controller = deployController();

344:     bytes32 salt = bytes32(uint256(uint160(borrower)));

```

```solidity
File: WildcatSanctionsEscrow.sol

17:     sentinel = msg.sender;

36:     uint256 amount = balance();

```

```solidity
File: WildcatSanctionsSentinel.sol

11:   bytes32 public constant override WildcatSanctionsEscrowInitcodeHash =

25:     archController = _archController;

26:     chainalysisSanctionsList = _chainalysisSanctionsList;

```

```solidity
File: libraries/Errors.sol

4: uint256 constant Panic_CompilerPanic = 0x00;

5: uint256 constant Panic_AssertFalse = 0x01;

6: uint256 constant Panic_Arithmetic = 0x11;

7: uint256 constant Panic_DivideByZero = 0x12;

8: uint256 constant Panic_InvalidEnumValue = 0x21;

9: uint256 constant Panic_InvalidStorageByteArray = 0x22;

10: uint256 constant Panic_EmptyArrayPop = 0x31;

11: uint256 constant Panic_ArrayOutOfBounds = 0x32;

12: uint256 constant Panic_MemoryTooLarge = 0x41;

13: uint256 constant Panic_UninitializedFunctionPointer = 0x51;

15: uint256 constant Panic_ErrorSelector = 0x4e487b71;

16: uint256 constant Panic_ErrorCodePointer = 0x20;

17: uint256 constant Panic_ErrorLength = 0x24;

18: uint256 constant Error_SelectorPointer = 0x1c;

```

```solidity
File: libraries/FIFOQueue.sol

43:     uint256 startIndex = arr.startIndex;

44:     uint256 nextIndex = arr.nextIndex;

48:     for (uint256 i = 0; i < len; i++) {

56:     uint128 nextIndex = arr.nextIndex;

62:     uint128 startIndex = arr.startIndex;

71:     uint128 startIndex = arr.startIndex;

75:     for (uint256 i = 0; i < n; i++) {

```

```solidity
File: libraries/FeeMath.sol

23:     uint256 rate = rateBip.bipToRay();

46:     uint256 protocolFeeRay = protocolFeeBips.bipMul(baseInterestRay);

95:     uint256 previousTimeDelinquent = state.timeDelinquent;

115:     state.timeDelinquent = previousTimeDelinquent.satSub(timeDelta).toUint32();

119:     uint256 secondsRemainingWithPenalty = previousTimeDelinquent.satSub(delinquencyGracePeriod);

153:     baseInterestRay = state.calculateBaseInterest(timestamp);

168:     uint256 prevScaleFactor = state.scaleFactor;

172:     state.lastInterestAccruedTimestamp = uint32(timestamp);

```

```solidity
File: libraries/MarketState.sol

90:     uint256 scaledWithdrawals = state.scaledPendingWithdrawals;

131:     uint256 expiry = state.pendingWithdrawalExpiry;

```

```solidity
File: libraries/MathUtils.sol

6: uint256 constant BIP = 1e4;

7: uint256 constant HALF_BIP = 0.5e4;

9: uint256 constant RAY = 1e27;

10: uint256 constant HALF_RAY = 0.5e27;

12: uint256 constant BIP_RAY_RATIO = 1e23;

34:     uint256 rate = rateBip.bipToRay();

```

```solidity
File: libraries/SafeCastLib.sol

18:     _assertNonOverflow(x == (y = uint8(x)));

22:     _assertNonOverflow(x == (y = uint16(x)));

26:     _assertNonOverflow(x == (y = uint24(x)));

30:     _assertNonOverflow(x == (y = uint32(x)));

34:     _assertNonOverflow(x == (y = uint40(x)));

38:     _assertNonOverflow(x == (y = uint48(x)));

42:     _assertNonOverflow(x == (y = uint56(x)));

46:     _assertNonOverflow(x == (y = uint64(x)));

50:     _assertNonOverflow(x == (y = uint72(x)));

54:     _assertNonOverflow(x == (y = uint80(x)));

58:     _assertNonOverflow(x == (y = uint88(x)));

62:     _assertNonOverflow(x == (y = uint96(x)));

66:     _assertNonOverflow(x == (y = uint104(x)));

70:     _assertNonOverflow(x == (y = uint112(x)));

74:     _assertNonOverflow(x == (y = uint120(x)));

78:     _assertNonOverflow(x == (y = uint128(x)));

82:     _assertNonOverflow(x == (y = uint136(x)));

86:     _assertNonOverflow(x == (y = uint144(x)));

90:     _assertNonOverflow(x == (y = uint152(x)));

94:     _assertNonOverflow(x == (y = uint160(x)));

98:     _assertNonOverflow(x == (y = uint168(x)));

102:     _assertNonOverflow(x == (y = uint176(x)));

106:     _assertNonOverflow(x == (y = uint184(x)));

110:     _assertNonOverflow(x == (y = uint192(x)));

114:     _assertNonOverflow(x == (y = uint200(x)));

118:     _assertNonOverflow(x == (y = uint208(x)));

122:     _assertNonOverflow(x == (y = uint216(x)));

126:     _assertNonOverflow(x == (y = uint224(x)));

130:     _assertNonOverflow(x == (y = uint232(x)));

134:     _assertNonOverflow(x == (y = uint240(x)));

138:     _assertNonOverflow(x == (y = uint248(x)));

```

```solidity
File: libraries/StringQuery.sol

12: uint256 constant SixtyThreeBytes = 0x3f;

13: uint256 constant ThirtyOneBytes = 0x1f;

14: uint256 constant OnlyFullWordMask = 0xffffffe0;

```

```solidity
File: market/WildcatMarket.sol

27:     MarketState memory state = _getUpdatedState();

46:     MarketState memory state = _getUpdatedState();

56:     uint104 scaledAmount = state.scaleAmount(amount).toUint104();

87:     uint256 actualAmount = depositUpTo(amount);

97:     MarketState memory state = _getUpdatedState();

101:     uint128 withdrawableFees = state.withdrawableProtocolFees(totalAssets());

120:     MarketState memory state = _getUpdatedState();

124:     uint256 borrowable = state.borrowableAssets(totalAssets());

143:     MarketState memory state = _getUpdatedState();

144:     state.annualInterestBips = 0;

145:     state.isClosed = true;

146:     state.reserveRatioBips = 0;

150:     uint256 currentlyHeld = totalAssets();

151:     uint256 totalDebts = state.totalDebts();

```

```solidity
File: market/WildcatMarketBase.sol

24:   string public constant version = '1.0';

77:     MarketParameters memory parameters = IWildcatMarketController(msg.sender).getMarketParameters();

96:     asset = parameters.asset;

99:     decimals = IERC20Metadata(parameters.asset).decimals();

117:     sentinel = parameters.sentinel;

118:     borrower = parameters.borrower;

119:     controller = parameters.controller;

120:     feeRecipient = parameters.feeRecipient;

121:     protocolFeeBips = parameters.protocolFeeBips;

122:     delinquencyFeeBips = parameters.delinquencyFeeBips;

123:     delinquencyGracePeriod = parameters.delinquencyGracePeriod;

124:     withdrawalBatchDuration = parameters.withdrawalBatchDuration;

151:     account = _accounts[accountAddress];

164:     Account memory account = _accounts[accountAddress];

166:       uint104 scaledBalance = account.scaledBalance;

167:       account.approval = AuthRole.Blocked;

171:         account.scaledBalance = 0;

201:     account = _getAccount(accountAddress);

205:         account.approval = AuthRole.DepositAndWithdraw;

319:     MarketState memory state = currentState();

335:     MarketState memory state = currentState();

336:     uint256 apr = state.annualInterestBips;

359:     state = _state;

362:       uint256 expiry = state.pendingWithdrawalExpiry;

408:     state = _state;

411:       expiredBatchExpiry = state.pendingWithdrawalExpiry;

423:       expiredBatch = _withdrawalData.batches[expiredBatchExpiry];

431:       state.pendingWithdrawalExpiry = 0;

450:     state.isDelinquent = isDelinquent;

451:     _state = state;

467:     uint32 expiry = state.pendingWithdrawalExpiry;

468:     WithdrawalBatch memory batch = _withdrawalData.batches[expiry];

489:     state.pendingWithdrawalExpiry = 0;

504:     uint104 scaledAvailableLiquidity = state.scaleAmount(availableLiquidity).toUint104();

511:     uint128 normalizedAmountPaid = state.normalizeAmount(scaledAmountBurned).toUint128();

533:     uint104 scaledAvailableLiquidity = state.scaleAmount(availableLiquidity).toUint104();

540:     uint128 normalizedAmountPaid = state.normalizeAmount(scaledAmountBurned).toUint128();

```

```solidity
File: market/WildcatMarketConfig.sol

22:     MarketState memory state = currentState();

78:     MarketState memory state = _getUpdatedState();

89:     Account memory account = _accounts[accountAddress];

98:     account.approval = AuthRole.Null;

116:     MarketState memory state = _getUpdatedState();

117:     Account memory account = _getAccount(_account);

119:       account.approval = AuthRole.DepositAndWithdraw;

121:       account.approval = AuthRole.WithdrawOnly;

135:     MarketState memory state = _getUpdatedState();

141:     state.maxTotalSupply = _maxTotalSupply.toUint128();

150:     MarketState memory state = _getUpdatedState();

156:     state.annualInterestBips = _annualInterestBips;

176:     MarketState memory state = _getUpdatedState();

178:     uint256 initialReserveRatioBips = state.reserveRatioBips;

185:     state.reserveRatioBips = _reserveRatioBips;

```

```solidity
File: market/WildcatMarketToken.sol

46:     uint256 allowed = allowance[from][msg.sender];

65:     MarketState memory state = _getUpdatedState();

66:     uint104 scaledAmount = state.scaleAmount(amount).toUint104();

72:     Account memory fromAccount = _getAccount(from);

76:     Account memory toAccount = _getAccount(to);

```

```solidity
File: market/WildcatMarketWithdrawals.sol

55:       batch = expiredBatch;

57:       batch = _withdrawalData.batches[expiry];

59:     AccountWithdrawalStatus memory status = _withdrawalData.accountStatuses[expiry][accountAddress];

62:     uint256 previousTotalWithdrawn = status.normalizedAmountWithdrawn;

78:     MarketState memory state = _getUpdatedState();

83:     uint104 scaledAmount = state.scaleAmount(amount).toUint104();

99:     uint32 expiry = state.pendingWithdrawalExpiry;

101:     WithdrawalBatch memory batch = _withdrawalData.batches[expiry];

144:     MarketState memory state = _getUpdatedState();

146:     WithdrawalBatch memory batch = _withdrawalData.batches[expiry];

157:     status.normalizedAmountWithdrawn = newTotalWithdrawn;

191:     MarketState memory state = _getUpdatedState();

194:     uint32 expiry = _withdrawalData.unpaidBatches.first();

197:     WithdrawalBatch memory batch = _withdrawalData.batches[expiry];

```

### <a name="GAS-14"></a>[GAS-14] Using storage instead of memory for structs/arrays saves gas

#### Impact:
Fetching data from a storage location and assigning it to a memory variable can result in higher gas costs. Using storage variables directly can be more gas-efficient.

*Instances (35)*:
```solidity
File: WildcatMarketController.sol

96:     MarketControllerParameters memory parameters = controllerFactory.getMarketControllerParameters();

320:     TmpMarketParameterStorage memory parameters = TmpMarketParameterStorage({

491:     TemporaryReserveRatio memory tmp = temporaryExcessReserveRatio[market];

```

```solidity
File: WildcatMarketControllerFactory.sol

111:     bytes memory controllerInitCode = type(WildcatMarketController).creationCode;

121:     bytes memory marketInitCode = type(WildcatMarket).creationCode;

```

```solidity
File: market/WildcatMarket.sol

27:     MarketState memory state = _getUpdatedState();

46:     MarketState memory state = _getUpdatedState();

63:     Account memory account = _getAccountWithRole(msg.sender, AuthRole.DepositAndWithdraw);

97:     MarketState memory state = _getUpdatedState();

120:     MarketState memory state = _getUpdatedState();

143:     MarketState memory state = _getUpdatedState();

```

```solidity
File: market/WildcatMarketBase.sol

77:     MarketParameters memory parameters = IWildcatMarketController(msg.sender).getMarketParameters();

164:     Account memory account = _accounts[accountAddress];

319:     MarketState memory state = currentState();

335:     MarketState memory state = currentState();

468:     WithdrawalBatch memory batch = _withdrawalData.batches[expiry];

```

```solidity
File: market/WildcatMarketConfig.sol

22:     MarketState memory state = currentState();

78:     MarketState memory state = _getUpdatedState();

89:     Account memory account = _accounts[accountAddress];

116:     MarketState memory state = _getUpdatedState();

117:     Account memory account = _getAccount(_account);

135:     MarketState memory state = _getUpdatedState();

150:     MarketState memory state = _getUpdatedState();

176:     MarketState memory state = _getUpdatedState();

```

```solidity
File: market/WildcatMarketToken.sol

65:     MarketState memory state = _getUpdatedState();

72:     Account memory fromAccount = _getAccount(from);

76:     Account memory toAccount = _getAccount(to);

```

```solidity
File: market/WildcatMarketWithdrawals.sol

59:     AccountWithdrawalStatus memory status = _withdrawalData.accountStatuses[expiry][accountAddress];

78:     MarketState memory state = _getUpdatedState();

81:     Account memory account = _getAccountWithRole(msg.sender, AuthRole.WithdrawOnly);

101:     WithdrawalBatch memory batch = _withdrawalData.batches[expiry];

144:     MarketState memory state = _getUpdatedState();

146:     WithdrawalBatch memory batch = _withdrawalData.batches[expiry];

191:     MarketState memory state = _getUpdatedState();

197:     WithdrawalBatch memory batch = _withdrawalData.batches[expiry];

```

