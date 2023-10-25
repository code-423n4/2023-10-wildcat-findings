G-1. Using the `indexed` keyword for value types such as `uint`, `bool`, and `address` saves gas costs.

If you add indexed keyword, you can save gas at least 80 gas in emitting event.
Up to three `indexed` can be used per event and should be added.
Please read this document : https://gist.github.com/Tomosuke0930/9bf61e01a8c3e214d95a9b84dcb41d97

Instances : 

File : src/WildcatArchController.sol

30 :   event MarketRemoved(address market);

32 :   event ControllerFactoryAdded(address controllerFactory);

33 :   event ControllerFactoryRemoved(address controllerFactory);

35 :   event BorrowerAdded(address borrower);

36 :   event BorrowerRemoved(address borrower);

38 :   event ControllerAdded(address indexed controllerFactory, address controller);

39 :   event ControllerRemoved(address controller);

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L30

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L32

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L33

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L35
 
 https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L36
 
 https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L38
 
 https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L39
 
 File : src/WildcatMarketControllerFactory.sol
 
 16 :   event NewController(address borrower, address controller, string namePrefix, string symbolPrefix);
 
 17-21 :   event UpdateProtocolFeeConfiguration(
    address feeRecipient,
    uint16 protocolFeeBips,
    address originationFeeAsset,
    uint256 originationFeeAmount
  );
 
 https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L16
 
 https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L17C3-L22C5



G-2. No need to initialize variables with default values 

If a variable is not set/initialized, it is assumed to have the default value(0, false, 0x0 etc depending on the data type). If you explicitly initialize it with its default value, you are just wasting gas.

Declaring uint256 i = 0; means doing an MSTORE of the value 0 instead you could just declare uint256 i to declare the variable without assigning it's default value, saving 3 gas per declaration.

Instances : 

File : src/WildcatMarketControllerFactory.sol

146 :     for (uint256 i = 0; i < count; i++) {

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L146


File : src/WildcatMarketController.sol

133 :     for (uint256 i = 0; i < count; i++) {

154 :     for (uint256 i = 0; i < lenders.length; i++) {

170 :     for (uint256 i = 0; i < lenders.length; i++) {

183 :     for (uint256 i; i < markets.length; i++) {

212 :     for (uint256 i = 0; i < count; i++) {


https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L133

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L154

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L170

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L183

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L212


File : src/WildcatArchController.sol

93 :     for (uint256 i = 0; i < count; i++) {

136 :     for (uint256 i = 0; i < count; i++) {

179 :     for (uint256 i = 0; i < count; i++) {

222 :     for (uint256 i = 0; i < count; i++) {


https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L93

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L136

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L179

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L222

File : src/libraries/FIFOQueue.sol

48 :     for (uint256 i = 0; i < len; i++) {

75 :     for (uint256 i = 0; i < n; i++) {

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L48

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L75


G-3. Use hardcoded address instead of `address(this)`

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. Refrences

Instances :

File : src/WildcatSanctionsSentinel.sol

77 :               abi.encodePacked(
                bytes1(0xff),
                address(this),
                keccak256(abi.encode(borrower, account, asset)),
                WildcatSanctionsEscrowInitcodeHash
              )

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsSentinel.sol#L77


File : src/WildcatMarketController.sol	

53 :   uint256 internal immutable ownCreate2Prefix = LibStoredInitCode.getCreate2Prefix(address(this));

243 :     parameters.controller = address(this);

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L53

File : src/market/WildcatMaerketBase.sol

175 : if (scaledBalance > 0) {
        account.scaledBalance = 0;
        address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
          accountAddress,
          borrower,
          address(this)
        );
		
239 :     return IERC20(asset).balanceOf(address(this));

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L175C24-L175C24

File : src/market/WildcatMarket.sol

60 :     asset.safeTransferFrom(msg.sender, address(this), amount);

154 :       asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarket.sol#L60

File : src/WildcatMarketControllerFactory.sol

44 :     return IERC20(asset).balanceOf(address(this));

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L44


G-4. Save gas by not calculating the constants

File : src/libraries/MathUtils.sol

6 : uint256 constant BIP = 1e4;

7 : uint256 constant HALF_BIP = 0.5e4;

9 : uint256 constant RAY = 1e27;

10 : uint256 constant HALF_RAY = 0.5e27;

12 : uint256 constant BIP_RAY_RATIO = 1e23;

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MathUtils.sol#L6


G-5. Amounts should be checked for 0 before calling a `transfer`

It can be beneficial to check if an amount is zero before invoking a transfer function, such as transfer or safeTransfer, to avoid unnecessary gas costs associated with executing the transfer operation. If the amount is zero, the transfer operation would have no effect, and performing the check can prevent unnecessary gas consumption.

File : src/WildcatMarketController.sol

346 :       originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L346

File : src/Market/WildcatMarketToken.sol

37 :     _transfer(msg.sender, to, amount);

54 :     _transfer(from, to, amount);

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L37

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L54



G-6. Instead of `address(0)` write it out, it saves gas

use : 0x00000000000000000000000
instead of : address(0)

File : src/WildcatMarketControllerFactory.sol

202 :     bool nullFeeRecipient = feeRecipient == address(0);

203 :     bool nullOriginationFeeAsset = originationFeeAsset == address(0);

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L202

File : src/WildcatMarketController.sol

324 :       feeRecipient: address(0),

345 :     if (originationFeeAsset != address(0)) {

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L324



G-7.  Comparisons: `> 0` is less efficient than `!= 0` for unsigned integers Comparison

Using `> 0` uses slightly more gas than using `!= 0`. 

Use `!= 0` when comparing uint variables to zero, which cannot hold values below zero

File : src/market/WildcatMarketBase.sol

79 :     if ((parameters.protocolFeeBips > 0).and(parameters.feeRecipient == address(0))) {

170 :       if (scaledBalance > 0) {

428 :       if (availableLiquidity > 0) {

472 :     if (availableLiquidity > 0) {

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L79

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L170

File : src/market/WildcatMarketWithdrawals.sol

112 :     if (availableLiquidity > 0) {

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketWithdrawals.sol#L112


File : src/market/WildcatMarket.sol

147 :     if (_withdrawalData.unpaidBatches.length() > 0) {

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarket.sol#L147


File : src/libraries/FeeMath.sol

67 :     if (timeWithPenalty > 0) {

155 :     if (protocolFeeBips > 0) {

159 :     if (delinquencyFeeBips > 0) {

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L67


G-8. Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas.

File : src/WildcatMarketController.sol

155 :       address lender = lenders[i];

171 :       address lender = lenders[i];

184 :       address market = markets[i];

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L155

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L171

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L184



G-9. Gas saving is achieved by removing the `delete` keyword 

The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary.

File : src/WildcatMarketController.sol

500 :     delete temporaryExcessReserveRatio[market];

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L500

File : src/libraries/FIFOQueue.sol

66 :     delete arr.data[startIndex];

76 :       delete arr.data[startIndex + i];

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L66

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L76


G-10. `OR` in `if`-condition can be rewritten to two single if conditions

Refactoring the if-condition in a way it won’t be containing the || operator will save more gas.

File : src/WildcatMarketControllerFactory.sol

80-89 :     if (
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
	  
204-209 :     if (
      (protocolFeeBips > 0 && nullFeeRecipient) ||
      (hasOriginationFee && nullFeeRecipient) ||
      (hasOriginationFee && nullOriginationFeeAsset)
    ) {
      revert InvalidProtocolFeeConfiguration();
	  
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L79C5-L89C35





			  