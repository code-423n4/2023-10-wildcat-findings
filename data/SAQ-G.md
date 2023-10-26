Hashmat Jan:
# Gas-Optmization

| Number | Gas                                                                                                                | context |
| :----: | :----------------------------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE |    3    |
| [G-02] | Use do while  loops instead of for loops                                                                           |   12    |
| [G-03] | Using > 0 costs more gas than != 0 when used on a uint in a require() and assert statement                         |   12    |
| [G-04] | USE BITMAPS TO SAVE GAS                                                                                            |    1    |
| [G-05] | Structs can be packed into fewer storage slots by editing time variables                                           |    4    |
| [G-06] | Avoid contract existence checks by using low level calls                                                           |   28    |
| [G-07] | Using fixed bytes is cheaper than using string                                                                     |    3    |
| [G-08] | Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)   |    2    |
| [G-09] | A modifier used only once and not being inherited should be inlined to save gas                                    |    4    |
| [G-10] | Refactor event to avoid emitting data that is already present in transaction data                                  |    1    |
| [G-11] | Do not calculate constants                                                                                         |    2    |
| [G-12] | SHOULD USE ARGUMENTS INSTEAD OF STATE VARIABLE                                                                     |    1    |
| [G-13] | Use constants instead of type(uintx).max                                                                           |    1    |
| [G-14] | Use nested if and, avoid multiple check combinations &&                                                            |    3    |
| [G-15] | Missing zero address checks in the constructor                                                                     |    4    |
| [G-16] | INTERNAL FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE REMOVED TO SAVE DEPLOYMENT GAS                             |    5    |
| [G-17] | Pre-increment and pre-decrement are cheaper than +1 ,-1                                                            |    2    |
| [G-18] | Using delete statement can save gas                                                                                |    5    |
| [G-19] | abi.encode() is less efficient than abi.encodePacked()                                                             |    2    |
| [G-20] | <X> += <Y> Costs more gas than <X> = <X> + <Y> for state variables                                                 |    4    |                                                                                                                 

## [G-01] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

solidity
File: src/market/WildcatMarketToken.sol

13  mapping(address => mapping(address => uint256)) public allowance;


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L13

solidity
File:  src/WildcatSanctionsSentinel.sol

20  mapping(address borrower => mapping(address account => bool sanctionOverride))

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L20

solidity
File: src/libraries/Withdrawal.sol

34  mapping(uint256 => mapping(address => AccountWithdrawalStatus)) accountStatuses;

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L34

## [G-02] Use do while  loops instead of for loops

solidity
File: src/libraries/FIFOQueue.sol

48    for (uint256 i = 0; i < len; i++) {

75    for (uint256 i = 0; i < n; i++) {


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48

solidity
File: src/WildcatArchController.sol

93    for (uint256 i = 0; i < count; i++) {

136    for (uint256 i = 0; i < count; i++) {

179    for (uint256 i = 0; i < count; i++) {

222    for (uint256 i = 0; i < count; i++) {


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93

solidity
File: src/WildcatMarketController.sol

133    for (uint256 i = 0; i < count; i++) {

154    for (uint256 i = 0; i < lenders.length; i++) {

170    for (uint256 i = 0; i < lenders.length; i++) {

183    for (uint256 i; i < markets.length; i++) {

212    for (uint256 i = 0; i < count; i++) {


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133

solidity
File: src/WildcatMarketControllerFactory.sol

146    for (uint256 i = 0; i < count; i++) {


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L146

## [G-03] Using > 0 costs more gas than != 0 when used on a uint in a require() and assert statement

solidity
File: src/libraries/FeeMath.sol

67    if (timeWithPenalty > 0) {

155    if (protocolFeeBips > 0) {

159    if (delinquencyFeeBips > 0) {


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L67

solidity
File: src/market/WildcatMarket.sol

147    if (_withdrawalData.unpaidBatches.length() > 0) {


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L147

solidity
File: src/market/WildcatMarketBase.sol

79    if ((parameters.protocolFeeBips > 0).and(parameters.feeRecipient == address(0))) {

170      if (scaledBalance > 0) {

428      if (availableLiquidity > 0) {

472    if (availableLiquidity > 0) {


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L79

solidity
File: src/WildcatMarketControllerFactory.sol

201    bool hasOriginationFee = originationFeeAmount > 0;

205      (protocolFeeBips > 0 && nullFeeRecipient) ||


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L201

solidity
File: src/market/WildcatMarketWithdrawals.sol

32    if ((expiry == expiredBatchExpiry).and(expiry > 0)) {

112    if (availableLiquidity > 0) {


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L32

## [G-04] USE BITMAPS TO SAVE GAS

solidity
File: src/WildcatSanctionsSentinel.sol

20   mapping(address borrower => mapping(address account => bool sanctionOverride))

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L20

## [G-05] Structs can be packed into fewer storage slots by editing time variables

solidity
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

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L13-L36

solidity
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

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L18-L29

solidity
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

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L17

## [G-06] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

solidity
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

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L64

solidity
File: src/market/WildcatMarketBase.sol

172        address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(

204      if (IWildcatMarketController(controller).isAuthorizedLender(accountAddress)) {

239    return IERC20(asset).balanceOf(address(this));


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L172

solidity
File: src/WildcatMarketController.sol

185      if (!_controlledMarkets.contains(market)) {

188      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));

197    return _controlledMarkets.contains(market);

303      if (!archController.isRegisteredBorrower(msg.sender)) {


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L88

solidity
File: src/WildcatMarketControllerFactory.sol

127    return _deployedControllers.contains(controller);

329    market = IWildcatMarketController(controller).deployMarket(

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L127

solidity
File: src/market/WildcatMarketWithdrawals.sol

164    if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {

166      address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L164

solidity
File: WildcatSanctionsEscrow.sol

22    return IERC20(asset).balanceOf(address(this));

26    return !WildcatSanctionsSentinel(sentinel).isSanctioned(borrower, account);

38    IERC20(asset).transfer(account, amount);

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L22

solidity
File: src/WildcatSanctionsSentinel.sol

42      IChainalysisSanctionsList(chainalysisSanctionsList).isSanctioned(account);

100    if (!IWildcatArchController(archController).isRegisteredMarket(msg.sender)) {

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L42

## [G-07] Using fixed bytes is cheaper than using string

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

solidity
File: src/market/WildcatMarketBase.sol

24  string public constant version = '1.0';

57  string public name;

60  string public symbol;


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L24

## [G-08] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

solidity
File: src/WildcatMarketControllerFactory.sol

286    _tmpMarketBorrowerParameter = msg.sender;


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L286

solidity
File: src/WildcatSanctionsEscrow.sol

17    sentinel = msg.sender;


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L17

## [G-09] A modifier used only once and not being inherited should be inlined to save gas

solidity
File: src/WildcatArchController.sol

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

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L41

solidity
File: src/WildcatMarketController.sol

87  modifier onlyControlledMarket(address market) {
88    if (!_controlledMarkets.contains(market)) {
89      revert NotControlledMarket();
90    }
91    _;
92  }

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L87-L92

solidity
File: src/WildcatMarketControllerFactory.sol

65  modifier onlyArchControllerOwner() {
66    if (msg.sender != archController.owner()) {
67      revert CallerNotArchControllerOwner();
68    }
69    _;
70  }

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L65-70

## [G-10] Refactor event to avoid emitting data that is already present in transaction data

solidity
File: src/market/WildcatMarket.sol

160    emit MarketClosed(block.timestamp);


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L160

## [G-11] Do not calculate constants

solidity
File: src/libraries/Chainalysis.sol


6    IChainalysisSanctionsList constant SanctionsList = IChainalysisSanctionsList(
7      0x40C57923924B5c5c5455c48D93317139ADDaC8fb
8    );

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Chainalysis.sol#L6-L8

solidity
File: src/WildcatSanctionsSentinel.sol

11  bytes32 public constant override WildcatSanctionsEscrowInitcodeHash =
12        keccak256(type(WildcatSanctionsEscrow).creationCode);

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L11

## [G-12] SHOULD USE ARGUMENTS INSTEAD OF STATE VARIABLE

Emitting storage values instead of the memory one.

solidity
File: src/WildcatSanctionsEscrow.sol

40    emit EscrowReleased(account, asset, amount);

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L40

## [G-13] Use constants instead of type(uintx).max

solidity
File: src/market/WildcatMarketToken.sol

49    if (allowed != type(uint256).max) {


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L49

## [G-14] Use nested if and, avoid multiple check combinations &&

solidity
File: src/WildcatMarketControllerFactory.sol

204    if (
205      (protocolFeeBips > 0 && nullFeeRecipient) ||
206      (hasOriginationFee && nullFeeRecipient) ||
207      (hasOriginationFee && nullOriginationFeeAsset)
208    ) {

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L204-L208

## [G-15] Missing zero address checks in the constructor

solidity
File: src/WildcatMarketControllerFactory.sol

72  constructor(
73    address _archController,
74    address _sentinel,
75    MarketParameterConstraints memory constraints
76  ) {
77    archController = IWildcatArchController(_archController);
78    sentinel = _sentinel;

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L72-78

solidity
File: src/WildcatSanctionsSentinel.sol

24  constructor(address _archController, address _chainalysisSanctionsList) {
25    archController = _archController;
26    chainalysisSanctionsList = _chainalysisSanctionsList;

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L24-L26

## [G-16] INTERNAL FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE REMOVED TO SAVE DEPLOYMENT GAS

solidity
File: src/market/WildcatMarketBase.sol

163  function _blockAccount(MarketState memory state, address accountAddress) internal {


197  function _getAccountWithRole(

358  function _getUpdatedState() internal returns (MarketState memory state) {

448  function _writeState(MarketState memory state) internal {


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L163

solidity
File:  src/libraries/Withdrawal.sol

47  function availableLiquidityForPendingBatch(


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L40

## [G-17] Pre-increment and pre-decrement are cheaper than +1 ,-1

solidity
File: src/libraries/FIFOQueue.sol

58    arr.nextIndex = nextIndex + 1;

67    arr.startIndex = startIndex + 1;


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L58

## [G-18] Using delete statement can save gas

solidity
File:  src/market/WildcatMarket.sol

144    state.annualInterestBips = 0;

146    state.reserveRatioBips = 0;


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L144

solidity
File: src/market/WildcatMarketBase.sol

171        account.scaledBalance = 0;

431      state.pendingWithdrawalExpiry = 0;

489    state.pendingWithdrawalExpiry = 0;


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L171

## [G-19] abi.encode() is less efficient than abi.encodePacked()

solidity
File: src/WildcatSanctionsSentinel.sol

78                keccak256(abi.encode(borrower, account, asset)),

110    new WildcatSanctionsEscrow{ salt: keccak256(abi.encode(borrower, account, asset)) }();

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L78

## [G-20] <X> += <Y> Costs more gas than <X> = <X> + <Y> for state variables

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

solidity
File: src/market/WildcatMarketBase.sol

178        _accounts[escrow].scaledBalance += scaledBalance;

323      apr += MathUtils.bipToRay(delinquencyFeeBips);

338      apr += delinquencyFeeBips;


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L178

solidity
File: src/market/WildcatMarketWithdrawals.sol

104    _withdrawalData.accountStatuses[expiry][msg.sender].scaledAmount += scaledAmount;


https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L104