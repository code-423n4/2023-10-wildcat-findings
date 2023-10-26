# QA Report - Wildcat Protocol Audit Contest | 16 Oct 2023 - 26 Oct 2023

---

# Executive summary

### Overview

|               |                                                      |
| :------------ | :--------------------------------------------------- |
| Project Name  | Wildcat Protocol                                     |
| Repository    | https://github.com/code-423n4/2023-10-wildcat        |
| Twitter (X)   | [Link](https://twitter.com/wildcatfi?lang=bg)        |
| Documentation | [Link](https://wildcat-protocol.gitbook.io/wildcat/) |
| Methods       | Manual Review                                        |
| Total SLOC    | 2,332 over 22 contracts                              |

---

---

# Findings Summary

| ID              | Title                                                                                                                                                                            | Severity                  |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| [L-01](#L-01)   | Risk of Silent Overflow in `FeeMath#updateScaleFactorAndFees()`                                                                                                                  | _Low_                     |
| [L-02](#L-02)   | During the deployment of the Market, the Protocol doesn't ensure that the `Withdrawal Cycle Length` and `Grace Period Length` are in hours, as mentioned in the documentation    | _Low_                     |
| [L-03](#L-03)   | The direct transfer of the `underlying asset` from anyone other than the borrower to the Market should be disabled                                                               | _Low_                     |
| [L-05](#L-05)   | If the borrower or some of the lenders in the Wildcat Protocol become blacklisted in the underlying token, it can have significant implications for the protocol's functionality | _Low_                     |
| [NC-01](#NC-01) | Anyone can call `updateAccountAuthorization()` function                                                                                                                          | _Non Critical_            |
| [NC-02](#NC-02) | Wrongly emitted events                                                                                                                                                           | _Non Critical_            |
| [NC-03](#NC-03) | Incorrect Year Duration Calculation                                                                                                                                              | _Non Critical_            |
| [NC-04](#NC-04) | During deploying of market in `WildcatMarketController` contract the amount of transferred `originationFeeAsset` maybe should be approved by borrower                            | _Non Critical_            |
| [NC-05](#NC-05) | The direct transfer of the `underlying asset` from anyone other than the borrower to the Market should be disabled                                                               | _Non Critical_            |
| [S-01](#S-01)   | Create `repay()` function in `WildMarket` contract to handle repaying a market action by borrower                                                                                | _Suggestion/Optimization_ |
| [S-02](#S-02)   | Add on a Blocked role shift if the lender attempts to deposit while sanctioned                                                                                                   | _Suggestion/Optimization_ |

## <a name="L-01"></a>[L-01] Risk of Silent Overflow in `FeeMath#updateScaleFactorAndFees()`

### Explanation
In [`FeeMath#updateScaleFactorAndFees()`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L142-L174) function, there is a potential risk of silent overflow in the following line of code:

```solidity
state.scaleFactor = (prevScaleFactor + scaleFactorDelta).toUint112();
```

This line of code attempts to update the `state.scaleFactor` by adding `prevScaleFactor` to `scaleFactorDelta` and then converting the result to a `uint112`. The risk here is that if the sum of `prevScaleFactor` and `scaleFactorDelta` exceeds the maximum value that can be represented by a `uint112`, it will silently overflow, leading to incorrect results and potentially unexpected behavior.

### Mitigation  Steps
To mitigate this risk, it's essential to ensure that the sum of `prevScaleFactor` and `scaleFactorDelta` does not exceed the maximum value representable by a `uint112`. You can add a check to verify this condition before performing the addition and conversion.

```solidity
// Ensure that the sum of prevScaleFactor and scaleFactorDelta doesn't exceed uint112 max
require(
    prevScaleFactor <= type(uint112).max - scaleFactorDelta,
    "Potential overflow detected"
);

// Update state.scaleFactor
state.scaleFactor = (prevScaleFactor + scaleFactorDelta).toUint112();
```

---



## <a name="L-02"></a>[L-02] During the deployment of the Market, the Protocol doesn't ensure that the `Withdrawal Cycle Length` and `Grace Period Length`` are in hours, as mentioned in the documentation
### Impact
Wildcat Protocol does not ensure that the `Withdrawal Cycle Length` and `Grace Period Length` are in hours, as mentioned in the documentation. This discrepancy between the code and documentation can lead to confusion and incorrect configuration of these parameters, potentially affecting the protocol's behavior.

### Explanation
The Wildcat Protocol allows for the deployment of markets with various parameters, including 'Withdrawal Cycle Length' and 'Grace Period Length,' which are expected to be specified in hours, as indicated in the documentation.

However, in the code responsible for deploying a market, there is no explicit enforcement of this requirement. This means that users can input values for `Withdrawal Cycle Length` and `Grace Period Length` that are not in hours, potentially leading to incorrect configuration.

This issue can result in markets with unexpected behavior, as the protocol may interpret these values differently than intended if they are not in the expected time unit (hours).

### Proof of Concept
- [deployMarket()](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L291-L360)
```jsx
  function deployMarket(
    address asset,
    string memory namePrefix,
    string memory symbolPrefix,
    uint128 maxTotalSupply,
    uint16 annualInterestBips,
    uint16 delinquencyFeeBips,
    uint32 withdrawalBatchDuration,
    uint16 reserveRatioBips,
    uint32 delinquencyGracePeriod
  ) external returns (address market) {
    if (msg.sender == borrower) {
      if (!archController.isRegisteredBorrower(msg.sender)) {
        revert NotRegisteredBorrower();
      }
    } else if (msg.sender != address(controllerFactory)) {
      revert CallerNotBorrowerOrControllerFactory();
    }

    enforceParameterConstraints(
      namePrefix,
      symbolPrefix,
      annualInterestBips,
      delinquencyFeeBips,
      withdrawalBatchDuration,
      reserveRatioBips,
      delinquencyGracePeriod
    );

    TmpMarketParameterStorage memory parameters = TmpMarketParameterStorage({
      asset: asset,
      namePrefix: namePrefix,
      symbolPrefix: symbolPrefix,
      feeRecipient: address(0),
      maxTotalSupply: maxTotalSupply,
      protocolFeeBips: 0,
      annualInterestBips: annualInterestBips,
      delinquencyFeeBips: delinquencyFeeBips,
      withdrawalBatchDuration: withdrawalBatchDuration,
      reserveRatioBips: reserveRatioBips,
      delinquencyGracePeriod: delinquencyGracePeriod
    });

    address originationFeeAsset;
    uint80 originationFeeAmount;
    (
      parameters.feeRecipient,
      originationFeeAsset,
      originationFeeAmount,
      parameters.protocolFeeBips
    ) = controllerFactory.getProtocolFeeConfiguration();

    _tmpMarketParameters = parameters;

    if (originationFeeAsset != address(0)) {
      originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);
    }

    bytes32 salt = _deriveSalt(asset, namePrefix, symbolPrefix);
    market = LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt, marketInitCodeHash);
    if (market.codehash != bytes32(0)) {
      revert MarketAlreadyDeployed();
    }
    LibStoredInitCode.create2WithStoredInitCode(marketInitCodeStorage, salt);

    archController.registerMarket(market);
    _controlledMarkets.add(market);

    _resetTmpMarketParameters();
  }
```

### Tools Used
- Manual Inspection

### Recommended Mitigation Steps
To address this issue and ensure consistency between the code and documentation, the following mitigation steps are recommended:

1. **Code Validation:** Implement validation checks in the code responsible for deploying markets to ensure that `Withdrawal Cycle Length` and `Grace Period Length` are specified in hours. If a user attempts to deploy a market with values in a different time unit, the code should revert.

---

## <a name="L-03"></a>[L-03] If the borrower or some of the lenders in the Wildcat Protocol become blacklisted in the underlying token, it can have significant implications for the protocol's functionality 

### Impact
If the borrower or some of the lenders in the Wildcat Protocol become blacklisted in the underlying token, it can have significant implications for the protocol's functionality.
**The likelihood of this issue actually can be calssified as medium, because of the protocol logic and flow, one market can exists very long time.**

### Explanation
The Wildcat Protocol Markets includes two assets. `Underlying Tokens (Normalized Amount) and Market Tokens (Scaled Amount).`

When Market is deploying the borrower specify the `Underlying Asset`, which basically is the asset that the borrower wish to borrow, such as DAI or WETH. 
After the market is deployed, borrower can borrow specific amount of underlying asset and lenders can deplosit amount of underlying asset and after some amount of time to withdraw it.

If the borrower or some of the lenders in the Wildcat Protocol become blacklisted in the underlying token, it can have significant implications for the protocol's functionality.

### Proof of Concept
> From the Contest Audit [Page](https://code4rena.com/contests/2023-10-the-wildcat-protocol#top): We anticipate that any ERC-20 can be used as an underlying asset for a market.

- [borrow()](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarket.sol#L119-L131)
```jsx
  function borrow(uint256 amount) external onlyBorrower nonReentrant {
    MarketState memory state = _getUpdatedState();
    if (state.isClosed) {
      revert BorrowFromClosedMarket();
    }
    uint256 borrowable = state.borrowableAssets(totalAssets());
    if (amount > borrowable) {
      revert BorrowAmountTooHigh();
    }
    _writeState(state);
    asset.safeTransfer(msg.sender, amount);
    emit Borrow(amount);
  }
```

- [deposit() and depositUpTo()](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarket.sol#L42-L91)
```jsx
  function depositUpTo(
    uint256 amount
  ) public virtual nonReentrant returns (uint256 /* actualAmount */) {
    // Get current state
    MarketState memory state = _getUpdatedState();

    if (state.isClosed) {
      revert DepositToClosedMarket();
    }

    // Reduce amount if it would exceed totalSupply
    amount = MathUtils.min(amount, state.maximumDeposit());

    // Scale the mint amount
    uint104 scaledAmount = state.scaleAmount(amount).toUint104();
    if (scaledAmount == 0) revert NullMintAmount();

    // Transfer deposit from caller
    asset.safeTransferFrom(msg.sender, address(this), amount);

    // Cache account data and revert if not authorized to deposit.
    Account memory account = _getAccountWithRole(msg.sender, AuthRole.DepositAndWithdraw);
    account.scaledBalance += scaledAmount;
    _accounts[msg.sender] = account;

    emit Transfer(address(0), msg.sender, amount);
    emit Deposit(msg.sender, amount, scaledAmount);

    // Increase supply
    state.scaledTotalSupply += scaledAmount;

    // Update stored state
    _writeState(state);

    return amount;
  }

  /**
   * @dev Deposit exactly `amount` underlying assets and mint market tokens
   *      for `msg.sender`.
   *
   *     Reverts if the deposit amount would cause the market to exceed the
   *     configured `maxTotalSupply`.
   */
  function deposit(uint256 amount) external virtual {
    uint256 actualAmount = depositUpTo(amount);
    if (amount != actualAmount) {
      revert MaxSupplyExceeded();
    }
  }
```

- [queueWithdrawal()](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketWithdrawals.sol#L77-L121)
```jsx
  function queueWithdrawal(uint256 amount) external nonReentrant {
    MarketState memory state = _getUpdatedState();

    // Cache account data and revert if not authorized to withdraw.
    Account memory account = _getAccountWithRole(msg.sender, AuthRole.WithdrawOnly);

    uint104 scaledAmount = state.scaleAmount(amount).toUint104();
    if (scaledAmount == 0) {
      revert NullBurnAmount();
    }

    // Reduce caller's balance and emit transfer event.
    account.scaledBalance -= scaledAmount;
    _accounts[msg.sender] = account;
    emit Transfer(msg.sender, address(this), amount);

    // If there is no pending withdrawal batch, create a new one.
    if (state.pendingWithdrawalExpiry == 0) {
      state.pendingWithdrawalExpiry = uint32(block.timestamp + withdrawalBatchDuration);
      emit WithdrawalBatchCreated(state.pendingWithdrawalExpiry);
    }
    // Cache batch expiry on the stack for gas savings.
    uint32 expiry = state.pendingWithdrawalExpiry;

    WithdrawalBatch memory batch = _withdrawalData.batches[expiry];

    // Add scaled withdrawal amount to account withdrawal status, withdrawal batch and market state.
    _withdrawalData.accountStatuses[expiry][msg.sender].scaledAmount += scaledAmount;
    batch.scaledTotalAmount += scaledAmount;
    state.scaledPendingWithdrawals += scaledAmount;

    emit WithdrawalQueued(expiry, msg.sender, scaledAmount);

    // Burn as much of the withdrawal batch as possible with available liquidity.
    uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());
    if (availableLiquidity > 0) {
      _applyWithdrawalBatchPayment(batch, state, expiry, availableLiquidity);
    }

    // Update stored batch data
    _withdrawalData.batches[expiry] = batch;

    // Update stored state
    _writeState(state);
  }
```

### Tools Used
Manual Inspection

### Recommended Mitigation Steps
There isn't a really elegant way to fix this.

---

## <a name="NC-01"></a>[NC-01] Anyone can call `updateAccountAuthorization()` function

### Explanation
Restrict the function to be called only from market borrowers as it done in `deauthorizeLenders()` and `authorizeLenders()` functions.

### GitHub Links: https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L153-L190

---

## <a name="NC-02"></a>[NC-02] Wrongly emitted events
### Example Instance:
[`depositUpTo()`](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarket.sol#L42-L77) function:
```jsx
    emit Transfer(address(0), msg.sender, amount);
```


However the event params are actually this: 
```jsx
// Interface: IMarketEventsAndErrors

    event Transfer(address from, address to, uint256 value);  
```

### Mitigation
Emit the events correctly. For our Example instance is should be:
```jsx
    emit Transfer(msg.sender, address(this), amount);
```

---

## <a name="NC-03"></a>[NC-03] Incorrect Year Duration Calculation

#### Description
In the codebase of `MathUtils`, there's a constant definition that assumes a year always has 365 days. However, this assumption is not accurate because leap years have 366 days. This discrepancy in year duration can lead to incorrect calculations, especially when precise time intervals are essential.

#### GitHub Links: https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/MathUtils.sol#L14

```solidity
// MathUtils.sol

14: uint256 constant SECONDS_IN_365_DAYS = 365 days;
```

#### Impact
During leap years, calculations that rely on this constant may produce incorrect results, affecting the accuracy of time-based operations.

#### Recommendation
To ensure accurate time-related calculations, consider using a more precise definition for the number of seconds in a year that accounts for leap years. One way to do this is by relying on built-in Solidity time units, such as `1 years`, which automatically adjust for leap years.

---

## <a name="NC-04"></a>[NC-04] During deploying of market in `WildcatMarketController` contract the amount of transferred `originationFeeAsset` maybe should be approved by borrower
### Impact
During the deployment of a market in the `WildcatMarketController` contract, the code transfers the `originationFeeAsset` from the `borrower` to the `feeRecipient` without verifying whether the `borrower` has approved this transfer. This may result in unexpected behavior if the `originationFeeAsset` requires approval for token transfers. (In General will revert)

### Explanation
The `deployMarket` function in the `WildcatMarketController` contract deploys a new market with various parameters, including the transfer of an `originationFeeAsset` from the `borrower` to the `feeRecipient`. However, the code does not include a check to ensure that the `borrower` has approved this transfer, which is required for certain token standards, such as ERC-20 and ERC-777.

If the `originationFeeAsset` being used is an ERC-20 or ERC-777 token that requires approval for transfers, the deployment may fail or result in undesired behavior if the `borrower` has not granted the necessary approval beforehand. This can potentially disrupt the deployment process and require manual intervention to rectify.

### Proof of Concept
- [`deployMarket()`]()
```jsx
  function deployMarket(
    address asset,
    string memory namePrefix,
    string memory symbolPrefix,
    uint128 maxTotalSupply,
    uint16 annualInterestBips,
    uint16 delinquencyFeeBips,
    uint32 withdrawalBatchDuration,
    uint16 reserveRatioBips,
    uint32 delinquencyGracePeriod
  ) external returns (address market) {
    if (msg.sender == borrower) {
      if (!archController.isRegisteredBorrower(msg.sender)) {
        revert NotRegisteredBorrower();
      }
    } else if (msg.sender != address(controllerFactory)) {
      revert CallerNotBorrowerOrControllerFactory();
    }

    // ...code...

    address originationFeeAsset;
    uint80 originationFeeAmount;
    (
      parameters.feeRecipient,
      originationFeeAsset,
      originationFeeAmount,
      parameters.protocolFeeBips
    ) = controllerFactory.getProtocolFeeConfiguration();

    _tmpMarketParameters = parameters;

    if (originationFeeAsset != address(0)) {
      originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount); //@note
    }

    // ...code...

    archController.registerMarket(market);
    _controlledMarkets.add(market);

    _resetTmpMarketParameters();
  }
```

### Tools Used
- Manual Inspection
---

## <a name="NC-05"></a>[NC-05] The direct transfer of the `underlying asset` from anyone other than the borrower to the Market should be disabled

### Explanation

The Wildcat Protocol allows for the direct transfer of the underlying asset to the Market by anyone other than the borrower. This will lead to breaking of the **intended flow and logic of the protocol** and to unexpected behaviour.

---

## <a name="S-01"></a>[S-01] Create `repay()` function in `WildMarket` contract to handle repaying a market action by borrower

### Summary:
Currently there is no specific path that the lender need to take to repay to the Market. Any transfer from borrower to Market is considered a repaying/payment by the borrower. After the borrower transferred amount to Market, he should call `updateState()` function, therefore some unexpected behavior can occur (for example, lender to be still in grace period even repaying).

So it will be better to implement function like `repay()` which do both stuff - `transferring` and calling `updateState()` function 

---

## <a name="S-02"></a>[S-02] Add on a Blocked role shift if the lender attempts to deposit while sanctioned

---
