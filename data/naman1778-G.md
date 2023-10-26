## [G-01] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

Consider using abi.encodePacked() here:

Use abi.encodePacked() where possible to save gas

There are 2 instances of this issue in 1 file:

```
File: src/WildcatSanctionsSentinel.sol	

78: keccak256(abi.encode(borrower, account, asset)),

110: new WildcatSanctionsEscrow{ salt: keccak256(abi.encode(borrower, account, asset)) }();
```

    diff --git a/src/WildcatSanctionsSentinel.sol b/src/WildcatSanctionsSentinel.sol
    index c33be80..9c1cdd5 100644
    --- a/src/WildcatSanctionsSentinel.sol
    +++ b/src/WildcatSanctionsSentinel.sol
    @@ -75,7 +75,7 @@ contract WildcatSanctionsSentinel is IWildcatSanctionsSentinel {
                   abi.encodePacked(
                     bytes1(0xff),
                     address(this),
    -                keccak256(abi.encode(borrower, account, asset)),
    +                keccak256(abi.encodePacked(borrower, account, asset)),
                     WildcatSanctionsEscrowInitcodeHash
                   )
                 )
    @@ -107,7 +107,7 @@ contract WildcatSanctionsSentinel is IWildcatSanctionsSentinel {

         tmpEscrowParams = TmpEscrowParams(borrower, account, asset);

    -    new WildcatSanctionsEscrow{ salt: keccak256(abi.encode(borrower, account, asset)) }();
    +    new WildcatSanctionsEscrow{ salt: keccak256(abi.encodePacked(borrower, account, asset)) }();

         emit NewSanctionsEscrow(borrower, account, asset);

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }

    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }

    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-02] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

Theer are 7 instances of this isssue in 6 files:

```
File: src/WildcatMarketControllerFactory.sol	

203: bool nullOriginationFeeAsset = originationFeeAsset == address(0);
```

    diff --git a/src/WildcatMarketControllerFactory.sol b/src/WildcatMarketControllerFactory.sol
    index ff4903e..e268aa2 100644
    --- a/src/WildcatMarketControllerFactory.sol
    +++ b/src/WildcatMarketControllerFactory.sol
    @@ -200,11 +200,11 @@ contract WildcatMarketControllerFactory {
       ) external onlyArchControllerOwner {
         bool hasOriginationFee = originationFeeAmount > 0;
         bool nullFeeRecipient = feeRecipient == address(0);
    -    bool nullOriginationFeeAsset = originationFeeAsset == address(0);
    +    bool nullOriginationFeeAsset = ;
         if (
           (protocolFeeBips > 0 && nullFeeRecipient) ||
           (hasOriginationFee && nullFeeRecipient) ||
    -      (hasOriginationFee && nullOriginationFeeAsset)
    +      (hasOriginationFee && (originationFeeAsset == address(0)))
         ) {
           revert InvalidProtocolFeeConfiguration();
         }

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol

```
File: src/libraries/MathUtils.sol	

35: uint256 accumulatedInterestRay = rate * timeDelta;
```

    diff --git a/src/libraries/MathUtils.sol b/src/libraries/MathUtils.sol
    index 5e6c9ba..beacc3d 100644
    --- a/src/libraries/MathUtils.sol
    +++ b/src/libraries/MathUtils.sol
    @@ -32,9 +32,8 @@ library MathUtils {
         uint256 timeDelta
       ) internal pure returns (uint256 result) {
         uint256 rate = rateBip.bipToRay();
    -    uint256 accumulatedInterestRay = rate * timeDelta;
         unchecked {
    -      return accumulatedInterestRay / SECONDS_IN_365_DAYS;
    +      return (rate * timeDelta) / SECONDS_IN_365_DAYS;
         }
       }

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol

```
File: src/libraries/FeeMath.sol	

24: uint256 accumulatedInterestRay = rate * timeDelta;
```
    diff --git a/src/libraries/FeeMath.sol b/src/libraries/FeeMath.sol
    index 8005ce5..85208f6 100644
    --- a/src/libraries/FeeMath.sol
    +++ b/src/libraries/FeeMath.sol
    @@ -21,9 +21,8 @@ library FeeMath {
         uint256 timeDelta
       ) internal pure returns (uint256 result) {
         uint256 rate = rateBip.bipToRay();
    -    uint256 accumulatedInterestRay = rate * timeDelta;
         unchecked {
    -      return accumulatedInterestRay / SECONDS_IN_365_DAYS;
    +      return (rate * timeDelta) / SECONDS_IN_365_DAYS;
         }
       }

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol

```
File: src/libraries/MarketState.sol	

109: uint256 totalAvailableAssets = totalAssets - state.normalizedUnclaimedWithdrawals;

131: uint256 expiry = state.pendingWithdrawalExpiry;
```

    diff --git a/src/libraries/MarketState.sol b/src/libraries/MarketState.sol
    index 891fb2c..ae274fe 100644
    --- a/src/libraries/MarketState.sol
    +++ b/src/libraries/MarketState.sol
    @@ -106,8 +106,7 @@ library MarketStateLib {
         MarketState memory state,
         uint256 totalAssets
       ) internal pure returns (uint128) {
    -    uint256 totalAvailableAssets = totalAssets - state.normalizedUnclaimedWithdrawals;
    -    return uint128(MathUtils.min(totalAvailableAssets, state.accruedProtocolFees));
    +    return uint128(MathUtils.min(totalAssets - state.normalizedUnclaimedWithdrawals, state.accruedProtocolFees));
       }

       /**

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol

```
File: src/libraries/FIFOQueue.sol	

44: uint256 nextIndex = arr.nextIndex;
```

    diff --git a/src/libraries/FIFOQueue.sol b/src/libraries/FIFOQueue.sol
    index e8d0dde..2c195ea 100644
    --- a/src/libraries/FIFOQueue.sol
    +++ b/src/libraries/FIFOQueue.sol
    @@ -41,8 +41,7 @@ library FIFOQueueLib {

       function values(FIFOQueue storage arr) internal view returns (uint32[] memory _values) {
         uint256 startIndex = arr.startIndex;
    -    uint256 nextIndex = arr.nextIndex;
    -    uint256 len = nextIndex - startIndex;
    +    uint256 len = arr.nextIndex - startIndex;
         _values = new uint32[](len);

         for (uint256 i = 0; i < len; i++) {

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol

```
File: src/market/WildcatMarketToken.sol	

50: uint256 newAllowance = allowed - amount;
```

    diff --git a/src/market/WildcatMarketToken.sol b/src/market/WildcatMarketToken.sol
    index f6a7f94..0ccd946 100644
    --- a/src/market/WildcatMarketToken.sol
    +++ b/src/market/WildcatMarketToken.sol
    @@ -47,8 +47,7 @@ contract WildcatMarketToken is WildcatMarketBase {

         // Saves gas for unlimited approvals.
         if (allowed != type(uint256).max) {
    -      uint256 newAllowance = allowed - amount;
    -      _approve(from, msg.sender, newAllowance);
    +      _approve(from, msg.sender, allowed - amount);
         }

         _transfer(from, to, amount);

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }

    contract Contract0 {
        uint256 num = 5;
        uint256 sum;
        function not_optimized() public returns(bool){
            uint256 num1 = num;
            sum = num1 + 2;
        }
    }

    contract Contract1 {
        uint256 num = 5;
        uint256 sum;
        function optimized() public returns(bool){
            sum = num + 2;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63799                                     | 244             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24462           | 24462 | 24462  | 24462 | 1       |


| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63599                                     | 243             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24460           | 24460 | 24460  | 24460 | 1       |

## [G-03] Instead of counting down in for statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 12 instances of this issue in 4 files:

```
File: src/WildcatMarketController.sol	

133: for (uint256 i = 0; i < count; i++) {

154: for (uint256 i = 0; i < lenders.length; i++) {

170: for (uint256 i = 0; i < lenders.length; i++) {

183: for (uint256 i; i < markets.length; i++) {

212: for (uint256 i = 0; i < count; i++) {
```

    diff --git a/src/WildcatMarketController.sol b/src/WildcatMarketController.sol
    index 55f62f6..3edd80b 100644
    --- a/src/WildcatMarketController.sol
    +++ b/src/WildcatMarketController.sol
    @@ -130,7 +130,7 @@ contract WildcatMarketController is IWildcatMarketControllerEventsAndErrors {
         end = MathUtils.min(end, len);
         uint256 count = end - start;
         arr = new address[](count);
    -    for (uint256 i = 0; i < count; i++) {
    +    for (uint256 i = count - 1; i >= 0; i--) {
           arr[i] = _authorizedLenders.at(start + i);
         }
       }
    @@ -151,7 +151,7 @@ contract WildcatMarketController is IWildcatMarketControllerEventsAndErrors {
        *      to existing market accounts
        */
       function authorizeLenders(address[] memory lenders) external onlyBorrower {
    -    for (uint256 i = 0; i < lenders.length; i++) {
    +    for (uint256 i = lenders.length - 1; i >= 0; i--) {
           address lender = lenders[i];
           if (_authorizedLenders.add(lender)) {
             emit LenderAuthorized(lender);
    @@ -167,7 +167,7 @@ contract WildcatMarketController is IWildcatMarketControllerEventsAndErrors {
        *      to existing market accounts
        */
       function deauthorizeLenders(address[] memory lenders) external onlyBorrower {
    -    for (uint256 i = 0; i < lenders.length; i++) {
    +    for (uint256 i = lenders.length - 1; i >= 0; i--) {
           address lender = lenders[i];
           if (_authorizedLenders.remove(lender)) {
             emit LenderDeauthorized(lender);
    @@ -180,7 +180,7 @@ contract WildcatMarketController is IWildcatMarketControllerEventsAndErrors {
        *      status.
        */
       function updateLenderAuthorization(address lender, address[] memory markets) external {
    -    for (uint256 i; i < markets.length; i++) {
    +    for (uint256 i = markets.length - 1; i >= 0; i--) {
           address market = markets[i];
           if (!_controlledMarkets.contains(market)) {
             revert NotControlledMarket();
    @@ -209,7 +209,7 @@ contract WildcatMarketController is IWildcatMarketControllerEventsAndErrors {
         end = MathUtils.min(end, len);
         uint256 count = end - start;
         arr = new address[](count);
    -    for (uint256 i = 0; i < count; i++) {
    +    for (uint256 i = count - 1; i >= 0 ; i--) {
           arr[i] = _controlledMarkets.at(start + i);
         }
       }

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol

```
File: src/WildcatMarketControllerFactory.sol	

146: for (uint256 i = 0; i < count; i++) {
```

    diff --git a/src/WildcatMarketControllerFactory.sol b/src/WildcatMarketControllerFactory.sol
    index ff4903e..17f931c 100644
    --- a/src/WildcatMarketControllerFactory.sol
    +++ b/src/WildcatMarketControllerFactory.sol
    @@ -143,7 +143,7 @@ contract WildcatMarketControllerFactory {
         end = MathUtils.min(end, len);
         uint256 count = end - start;
         arr = new address[](count);
    -    for (uint256 i = 0; i < count; i++) {
    +    for (uint256 i = count - 1; i >= 0; i--) {
           arr[i] = _deployedControllers.at(start + i);
         }
       }

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol

```
File: src/WildcatArchController.sol	

93: for (uint256 i = 0; i < count; i++) {

136: for (uint256 i = 0; i < count; i++) {

179: for (uint256 i = 0; i < count; i++) {

222: for (uint256 i = 0; i < count; i++) {
```

    diff --git a/src/WildcatArchController.sol b/src/WildcatArchController.sol
    index 7b0a173..bc63a86 100644
    --- a/src/WildcatArchController.sol
    +++ b/src/WildcatArchController.sol
    @@ -90,7 +90,7 @@ contract WildcatArchController is Ownable {
         end = MathUtils.min(end, len);
         uint256 count = end - start;
         arr = new address[](count);
    -    for (uint256 i = 0; i < count; i++) {
    +    for (uint256 i = count - 1; i >= 0; i--) {
           arr[i] = _borrowers.at(start + i);
         }
       }
    @@ -133,7 +133,7 @@ contract WildcatArchController is Ownable {
         end = MathUtils.min(end, len);
         uint256 count = end - start;
         arr = new address[](count);
    -    for (uint256 i = 0; i < count; i++) {
    +    for (uint256 i = count - 1; i >= 0; i--) {
           arr[i] = _controllerFactories.at(start + i);
         }
       }
    @@ -176,7 +176,7 @@ contract WildcatArchController is Ownable {
         end = MathUtils.min(end, len);
         uint256 count = end - start;
         arr = new address[](count);
    -    for (uint256 i = 0; i < count; i++) {
    +    for (uint256 i = count - 1; i >= 0; i--) {
           arr[i] = _controllers.at(start + i);
         }
       }
    @@ -219,7 +219,7 @@ contract WildcatArchController is Ownable {
         end = MathUtils.min(end, len);
         uint256 count = end - start;
         arr = new address[](count);
    -    for (uint256 i = 0; i < count; i++) {
    +    for (uint256 i = count - 1; i >= 0; i--) {
           arr[i] = _markets.at(start + i);
         }
       }

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol

```
File: src/libraries/FIFOQueue.sol	

48: for (uint256 i = 0; i < len; i++) {

75: for (uint256 i = 0; i < n; i++) {
```

    diff --git a/src/libraries/FIFOQueue.sol b/src/libraries/FIFOQueue.sol
    index e8d0dde..a80304c 100644
    --- a/src/libraries/FIFOQueue.sol
    +++ b/src/libraries/FIFOQueue.sol
    @@ -45,7 +45,7 @@ library FIFOQueueLib {
         uint256 len = nextIndex - startIndex;
         _values = new uint32[](len);

    -    for (uint256 i = 0; i < len; i++) {
    +    for (uint256 i = len - 1; i >= 0; i--) {
           _values[i] = arr.data[startIndex + i];
         }

    @@ -72,7 +72,7 @@ library FIFOQueueLib {
         if (startIndex + n > arr.nextIndex) {
           revert FIFOQueueOutOfBounds();
         }
    -    for (uint256 i = 0; i < n; i++) {
    +    for (uint256 i = n - 1; i >= 0; i--) {
           delete arr.data[startIndex + i];
         }
         arr.startIndex = startIndex + n;

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol

#### Test Code

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

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-04] Revert Transaction as soon as possible

Always try reverting transactions as early as possible when using require statements . In case a transaction revert occurs, the user will pay the gas up until the revert was executed

There are 2 instances of this issue in 2 files:

```
File: src/market/WildcatMarketWithdrawals.sol	

137: function executeWithdrawal(
138:   address accountAddress,
139:   uint32 expiry
140: ) external nonReentrant returns (uint256) {
141:   if (expiry > block.timestamp) {
142:     revert WithdrawalBatchNotExpired();
143:   }
144:   MarketState memory state = _getUpdatedState();
145: 
146:   WithdrawalBatch memory batch = _withdrawalData.batches[expiry];
147:   AccountWithdrawalStatus storage status = _withdrawalData.accountStatuses[expiry][
148:     accountAddress
149:   ];
150: 
151:   uint128 newTotalWithdrawn = uint128(
152:     MathUtils.mulDiv(batch.normalizedAmountPaid, status.scaledAmount, batch.scaledTotalAmount)
153:   );
154: 
155:   uint128 normalizedAmountWithdrawn = newTotalWithdrawn - status.normalizedAmountWithdrawn;
156: 
157:   status.normalizedAmountWithdrawn = newTotalWithdrawn;
158:   state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;
159: 
161:   if (normalizedAmountWithdrawn == 0) {
162:     revert NullWithdrawalAmount();
163:   }
```

    diff --git a/src/market/WildcatMarketWithdrawals.sol b/src/market/WildcatMarketWithdrawals.sol
    index 7e6587b..b48028d 100644
    --- a/src/market/WildcatMarketWithdrawals.sol
    +++ b/src/market/WildcatMarketWithdrawals.sol
    @@ -154,13 +154,14 @@ contract WildcatMarketWithdrawals is WildcatMarketBase {

         uint128 normalizedAmountWithdrawn = newTotalWithdrawn - status.normalizedAmountWithdrawn;

    -    status.normalizedAmountWithdrawn = newTotalWithdrawn;
    -    state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;
    -
         if (normalizedAmountWithdrawn == 0) {
           revert NullWithdrawalAmount();
         }

    +    status.normalizedAmountWithdrawn = newTotalWithdrawn;
    +    state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;
    +
    +
         if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
           _blockAccount(state, accountAddress);
           address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol

```
File: src/market/WildcatMarketConfig.sol	

149: function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant {
150:   MarketState memory state = _getUpdatedState();
151: 
152:   if (_annualInterestBips > BIP) {
153:     revert InterestRateTooHigh();
154:   }
```

    diff --git a/src/market/WildcatMarketConfig.sol b/src/market/WildcatMarketConfig.sol
    index 539a435..0a7840f 100644
    --- a/src/market/WildcatMarketConfig.sol
    +++ b/src/market/WildcatMarketConfig.sol
    @@ -147,12 +147,12 @@ contract WildcatMarketConfig is WildcatMarketBase {
        * @dev Sets the annual interest rate earned by lenders in bips.
        */
       function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant {
    -    MarketState memory state = _getUpdatedState();
    -
         if (_annualInterestBips > BIP) {
           revert InterestRateTooHigh();
         }

    +    MarketState memory state = _getUpdatedState();
    +
         state.annualInterestBips = _annualInterestBips;
         _writeState(state);
         emit AnnualInterestBipsUpdated(_annualInterestBips);

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol