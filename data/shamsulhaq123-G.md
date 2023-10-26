## Gas Optimizations

## Sumary

|   No | ISSUE  | INSTANCE  |
|------|--------|-----------|
|[G-01]|Counting down in for statements is more gas efficient|12|
|[G-02]|Use do while loops instead of for loops|12|
|[G-03]|Public Functions To External|11|
|[G-04]| Use assembly to perform efficient back-to-back calls|1|
|[G-05]| Use nested if statements instead of &&|2|
|[G-06]| Using a positive conditional flow to save a NOT opcode|14|
|[G-07]| Unnecessary look up in if condition|2|
|[G-08]|Use selfbalance() instead of address(this).balance|6|
|[G-09]| Use constants instead of type(uintx).max|1|
|[G-10]|Use function instead of modifiers|


## [G-1] Counting down in for statements is more gas efficient
Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

```solidity

file: WildcatMarketController.sol

133  for (uint256 i = 0; i < count; i++)

154  for (uint256 i = 0; i < lenders.length; i++) 

170   for (uint256 i = 0; i < lenders.length; i++) 

183  for (uint256 i; i < markets.length; i++)

212  for (uint256 i = 0; i < count; i++)
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L154
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L170
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L183
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L212

```Solidity

file: WildcatMarketControllerFactory.sol

146  for (uint256 i = 0; i < count; i++) 

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L146

```Solidity

file: WildcatArchController.sol

93  for (uint256 i = 0; i < count; i++) 

136  for (uint256 i = 0; i < count; i++) 

179 for (uint256 i = 0; i < count; i++) 

222  for (uint256 i = 0; i < count; i++) 
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L136
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L179
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L222

```Solidity

file: libraries/FIFOQueue.sol

48  for (uint256 i = 0; i < len; i++) 

75  for (uint256 i = 0; i < n; i++)
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L75

## [G-2] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity

file: WildcatMarketController.sol

133  for (uint256 i = 0; i < count; i++)

154  for (uint256 i = 0; i < lenders.length; i++) 

170   for (uint256 i = 0; i < lenders.length; i++) 

183  for (uint256 i; i < markets.length; i++)

212  for (uint256 i = 0; i < count; i++)
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L154
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L170
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L183
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L212

```Solidity

file: WildcatMarketControllerFactory.sol

146  for (uint256 i = 0; i < count; i++) 

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L146

```solidit

file: WildcatArchController.sol

93  for (uint256 i = 0; i < count; i++) 

136  for (uint256 i = 0; i < count; i++) 

179 for (uint256 i = 0; i < count; i++) 

222  for (uint256 i = 0; i < count; i++) 
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L136
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L179
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L222

```Solidity

file: libraries/FIFOQueue.sol

48  for (uint256 i = 0; i < len; i++) 

75  for (uint256 i = 0; i < n; i++)
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L75

## [G-3] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity

file: WildcatMarketControllerFactory.sol

282  function deployController() public returns (address controller) 
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L282

```solidit

file: /market/WildcatMarketConfig.sol

149   function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant 

171   function setReserveRatioBips(uint16 _reserveRatioBips) public onlyController nonReentrant 
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L149
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L171

```solidit

file: market/WildcatMarket.sol

42   function depositUpTo(
    uint256 amount
  ) public virtual nonReentrant returns (uint256 /* actualAmount */) 

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L42-L44


```solidit

file: WildcatSanctionsSentinel.sol

39 function isSanctioned(address borrower, address account) public view override returns (bool) 

48   function overrideSanction(address account) public override 

56 function removeSanctionOverride(address account) public override 

69  public view override returns (address escrowAddress)

99 public override returns (address escrowContract)
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L39
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L48
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L56
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L69
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L99

```solidit

file: market/WildcatMarketToken.sol

16   function balanceOf(address account) public view virtual nonReentrantView returns (uint256) {
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L16

```solidity

file: WildcatSanctionsEscrow.sol

21    function balance() public view override returns (uint256)
  return IERC20(asset).balanceOf(address(this));
  }

  function canReleaseEscrow() public view override returns (bool) {
    return !WildcatSanctionsSentinel(sentinel).isSanctioned(borrower, account);
  }

  function escrowedAsset() public view override returns (address, uint256) {
    return (asset, balance());
  }

  function releaseEscrow() public override {
34    if (!canReleaseEscrow()) revert CanNotReleaseEscrow();
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L21-L34

## [G-4] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity

file: WildcatSanctionsEscrow.sol

21  function balance() public view override returns (uint256)
     return IERC20(asset).balanceOf(address(this));
  }

  function canReleaseEscrow() public view override returns (bool) {
    return !WildcatSanctionsSentinel(sentinel).isSanctioned(borrower, account);
  }

  function escrowedAsset() public view override returns (address, uint256) {
    return (asset, balance());
  }

  function releaseEscrow() public override {
34    if (!canReleaseEscrow()) revert CanNotReleaseEscrow(); 

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L21-L34

## [G-5] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity

file: WildcatMarketControllerFactory.sol

204 if (
      (protocolFeeBips > 0 && nullFeeRecipient) ||
      (hasOriginationFee && nullFeeRecipient) ||
      (hasOriginationFee && nullOriginationFeeAsset)
208    ) 

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L204-L208

```solidit

file: WildcatSanctionsSentinel.sol

41  !sanctionOverrides[borrower][account] &&

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L41

## [G-6] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidit

file: WildcatMarketController.sol

88  if (!_controlledMarkets.contains(market)) 

185   if (!_controlledMarkets.contains(market)) 

303 if (!archController.isRegisteredBorrower(msg.sender))
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L88
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L185
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L303

```solidit

file: WildcatMarketControllerFactory.sol

283  if (!archController.isRegisteredBorrower(msg.sender)) {

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L283

```solidit

file: WildcatArchController.sol

42 if (!_controllerFactories.contains(msg.sender)) 

49 if (!_controllers.contains(msg.sender))

64  if (!_borrowers.add(borrower)) 

71 if (!_borrowers.remove(borrower)) 

107     if (!_controllerFactories.add(factory))

114  if (!_controllerFactories.remove(factory)) 

150   if (!_controllers.add(controller))

157  if (!_controllers.remove(controller)) 

193  if (!_markets.add(market))

200  if (!_markets.remove(market)) 
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L42
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L49
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L64
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L71
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L107
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L114
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L150
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L157
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L193
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L200

## [G-7] Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidit

file: WildcatMarketControllerFactory.sol

79  if (
      constraints.minimumAnnualInterestBips > constraints.maximumAnnualInterestBips ||
      constraints.maximumAnnualInterestBips > 10000 ||
      constraints.minimumDelinquencyFeeBips > constraints.maximumDelinquencyFeeBips ||
      constraints.maximumDelinquencyFeeBips > 10000 ||
      constraints.minimumReserveRatioBips > constraints.maximumReserveRatioBips ||
      constraints.maximumReserveRatioBips > 10000 ||
      constraints.minimumDelinquencyGracePeriod > constraints.maximumDelinquencyGracePeriod ||
      constraints.minimumWithdrawalBatchDuration > constraints.maximumWithdrawalBatchDuration
88    ) 

204  if (
      (protocolFeeBips > 0 && nullFeeRecipient) ||
      (hasOriginationFee && nullFeeRecipient) ||
      (hasOriginationFee && nullOriginationFeeAsset)
208    )
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L79-L88
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L204-L208

## [G-8]  Use selfbalance() instead of address(this).balance

```solidity

file: /WildcatMarketControllerFactory.sol

44   uint256 internal immutable ownCreate2Prefix = LibStoredInitCode.getCreate2Prefix(address(this));
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L44

```solidit

file: market/WildcatMarketWithdrawals.

91    emit Transfer(msg.sender, address(this), amount);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L91

```solidit

file: market/WildcatMarket.sol

60  asset.safeTransferFrom(msg.sender, address(this), amount);

154    asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L60
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L154

```solidit

file: WildcatSanctionsSentinel.sol

77 address(this)
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L77

```Solidity

file: WildcatSanctionsEscrow.sol

22  return IERC20(asset).balanceOf(address(this));
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L22

## [G-9] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity

file: market/WildcatMarketToken.sol

49 if (allowed != type(uint256).max)

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L49

## [G-10] Use function instead of modifiers 
Deployment. Gas Saved: 115 926
Minimum Method Call. Gas Saved: 162
Average Method Call. Gas Saved: -264
Maximum Method Call. Gas Saved: -481
Overall gas change: 734 (2.459%)

```Solidity

file: WildcatMarketController.sol

80  modifier onlyBorrower()

87  modifier onlyControlledMarket(address market) 
``` 
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L80-L87

```solidity

file: 

  modifier onlyArchControllerOwner() 
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L65

```solidity

file: WildcatArchController.sol

41   modifier onlyControllerFactory() {
  

48  modifier onlyController()
```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L41-L48 