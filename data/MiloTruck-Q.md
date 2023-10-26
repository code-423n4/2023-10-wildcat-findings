## Findings Summary

| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-interest-rates-might-be-inflated-slightly-above-the-markets-annual-interest-rate) | Interest rates might be inflated slightly above the market's annual interest rate | Low |
| [L-02](#l-02-deploymarket-might-be-dosed-depending-on-the-origination-fee-configuration) | `deployMarket()` might be DOSed depending on the origination fee configuration | Low |
| [L-03](#l-03-resetreserveratio-cannot-be-called-if-the-market-is-delinquent) | `resetReserveRatio()` cannot be called if the market is delinquent | Low |
| [L-04](#l-04-maximum-amount-of-assets-that-can-be-deposited-into-a-market-is-implicitly-limited-to-uint104) | Maximum amount of assets that can be deposited into a market is implicitly limited to `uint104` | Low |
| [L-05](#l-05-only-allow-lenders-to-call-executewithdrawal-for-themselves) | Only allow lenders to call `executeWithdrawal()` for themselves | Low |
| [N-01](#n-01-avoid-calling-_writestate-before-transferring-assets-in-borrow) | Avoid calling `_writeState()` before transferring assets in `borrow()` | Non-Critical |
| [N-02](#n-02-code-for-push-in-fifoqueuesol-can-be-shortened) | Code for `push()` in `FIFOQueue.sol` can be shortened | Non-Critical |
| [N-03](#n-03-redundant-checks-in-wildcatmarketbases-constructor) | Redundant checks in `WildcatMarketBase`'s constructor | Non-Critical |

## [L-01] Interest rates might be inflated slightly above the market's annual interest rate

### Bug Description

Whenever the state of a market is updated, `updateScaleFactorAndFees()` is called to increase the market's `scaleFactor` based on the annual interest rate:

[FeeMath.sol#L167-L171](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L167-L171)

```solidity
    // Calculate new scaleFactor
    uint256 prevScaleFactor = state.scaleFactor;
    uint256 scaleFactorDelta = prevScaleFactor.rayMul(baseInterestRay + delinquencyFeeRay);

    state.scaleFactor = (prevScaleFactor + scaleFactorDelta).toUint112();
```

As seen from above, the interest rate is not a fixed rate based on time, but rather, it compounds depending on how frequently `updateScaleFactorAndFees()` is called.

Due to how [compound interest works](https://www.investopedia.com/terms/c/compoundinterest.asp) works, if `updateState()` is called repeatedly, the final amount owed by the borrower in a market will be higher as compared to if `updateState()` was called once.

### Impact

Markets which have their state updated more frequently will have a slightly higher interest rate, which means the borrower will owe lenders slightly more assets. This could occur if:
- The market is popular, and state-changing functions are always called.
- A lender intentionally calls `updateState()` repeatedly.

This leads to a slight loss of funds for the borrower, and a slight gain for lenders.

### Proof of Concept

The following test demonstrates how the assets owed by a borrower after a year will be 0.1% higher if `updateState()` is called for a market every week:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.20;

import 'src/WildcatArchController.sol';
import 'src/WildcatMarketControllerFactory.sol';

import 'forge-std/Test.sol';
import 'test/shared/TestConstants.sol';
import 'test/helpers/MockERC20.sol';

contract MarketAnuualInterestRateTest is Test {
    // Wildcat contracts
    WildcatMarketController controller;
    
    // Test contracts
    MockERC20 asset = new MockERC20();

    // Users
    address BORROWER;
    address LENDER;

    function setUp() external {
        // Deploy Wildcat contracts
        WildcatArchController archController = new WildcatArchController();
        MarketParameterConstraints memory constraints = MarketParameterConstraints({
            minimumDelinquencyGracePeriod: MinimumDelinquencyGracePeriod,
            maximumDelinquencyGracePeriod: MaximumDelinquencyGracePeriod,
            minimumReserveRatioBips: MinimumReserveRatioBips,
            maximumReserveRatioBips: MaximumReserveRatioBips,
            minimumDelinquencyFeeBips: 0,
            maximumDelinquencyFeeBips: MaximumDelinquencyFeeBips,
            minimumWithdrawalBatchDuration: MinimumWithdrawalBatchDuration,
            maximumWithdrawalBatchDuration: MaximumWithdrawalBatchDuration,
            minimumAnnualInterestBips: MinimumAnnualInterestBips,
            maximumAnnualInterestBips: MaximumAnnualInterestBips
        });
        WildcatMarketControllerFactory controllerFactory = new WildcatMarketControllerFactory(
            address(archController),
            address(0),
            constraints
        );

        // Register controllerFactory in archController
        archController.registerControllerFactory(address(controllerFactory));

        // Setup borrower
        BORROWER = makeAddr("BORROWER");
        archController.registerBorrower(BORROWER);

        // Setup lender
        LENDER = makeAddr("LENDER");
        asset.mint(LENDER, 1000e18);

        // Deploy controller
        vm.prank(BORROWER);
        controller = WildcatMarketController(controllerFactory.deployController());

        // Register lender
        address[] memory arr = new address[](1);
        arr[0] = LENDER;
        vm.prank(BORROWER);
        controller.authorizeLenders(arr);
    }

    function test_lenderCanInflateInterestRate() public {
        // Deploy market with 10% annual interest rate
        vm.prank(BORROWER);
        address _market = controller.deployMarket(
            address(asset),
            "Market ",
            "MKT-",
            type(uint128).max,
            500, // annual interest rate = 50%
            0, // delinquency fee = 0%
            MaximumWithdrawalBatchDuration,
            MaximumReserveRatioBips,
            MaximumDelinquencyGracePeriod
        );
        WildcatMarket market = WildcatMarket(_market);

        // Lender deposits into market
        vm.startPrank(LENDER);
        asset.approve(address(market), 1000e18);
        market.depositUpTo(1000e18);
        vm.stopPrank();

        // Save a snapshot before interest compounding
        uint256 snapshot = vm.snapshot();

        // Calculate assets owed by borrower after 1 year
        skip(365 days);
        market.updateState();
        console2.log("Assets owed:", market.balanceOf(LENDER));

        // Revert to before interest compounding
        vm.revertTo(snapshot);

        // Calculate assets owed if updateState() were to be called every week
        for (uint256 i = 0; i < 52; ++i) {
            skip(1 weeks);
            market.updateState();
        }
        console2.log("Assets owed:", market.balanceOf(LENDER));
    }
}
```

### Recommendation

In `_getUpdatedState()`, consider calling `updateScaleFactorAndFees()` after a certain time period has passed, such as a week. This ensures that a lender cannot intentionally call `updateState()` repeatedly to inflate the interest rate. 

## [L-02] `deployMarket()` might be DOSed depending on the origination fee configuration 

### Bug Description

In `deployMarket()`, the origination fee is transferred to the `feeRecipient` as follows:

[WildcatMarketController.sol#L345-L347](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L345-L347)

```solidity
    if (originationFeeAsset != address(0)) {
      originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);
    }
```

The function only checks if `originationFeeAsset` is a non-zero address before calling `safeTransferFrom()`. 

This could cause `deployMarket()` to revert if `originationFeeAsset` is set to a token that reverts when transferring a zero value amount (eg. [LEND](https://etherscan.io/token/0x80fB784B7eD66730e8b1DBd9820aFD29931aab03)), and `originationFeeAmount` is set to 0.

### Impact

If the protocol's origination fee is configured as 0, but `originationFeeAsset` is a ERC20 token that reverts for zero-value transfers, `deployMarket()` will be DOSed.

### Recommendation

Consider checking that `originationFeeAmount` is non-zero as well:

[WildcatMarketController.sol#L345-L347](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L345-L347)

```diff
-   if (originationFeeAsset != address(0)) {
+   if (originationFeeAsset != address(0) && originationFeeAmount != 0) {
      originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);
    }
```

## [L-03] `resetReserveRatio()` cannot be called if the market is delinquent

### Bug Description

When a borrower first changes a market's annual interest rate using `setAnnualInterestBips()`, the market's reserve ratio is set to 90%. After 2 weeks, the borrower can call `resetReserveRatio()` to reset the reserve ratio back to its original value.

`resetReserveRatio()` calls `setReserveRatioBips()` to reset the market's reserve ratio:

[WildcatMarketController.sol#L499](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L499)

```solidity
    WildcatMarket(market).setReserveRatioBips(uint256(tmp.reserveRatioBips).toUint16());
```

However, `setReserveRatioBips()` checks that the market is not delinquent when decreasing its reserve ratio:

[WildcatMarketConfig.sol#L180-L184](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L180-L184)

```solidity
    if (_reserveRatioBips < initialReserveRatioBips) {
      if (state.liquidityRequired() > totalAssets()) {
        revert InsufficientReservesForOldLiquidityRatio();
      }
    }
```

This means that if a market becomes delinquent after `setAnnualInterestBips()` and the borrower has no funds to bring the market's reserve ratio back above delinquency, he will be unable to reset the market's reserve ratio.

### Impact

A borrower could be unable to call `resetReserveRatio()` even after 2 weeks if his market is still delinquent. This could be unfair to a borrower since calling `resetReserveRatio()` will most probably make the market non-delinquent, since it reduces the reserve ratio.

Note that a borrower can side-step this issue by using a flashloan, transferring assets into the market, calling `resetReserveRatio()` and then borrowing the assets to repay the flashloan. This, however, might incur a flashloan fee.

### Recommendation

In `resetReserveRatio()`, consider setting the market's reserve ratio directly instead of calling `setReserveRatioBips()` to avoid the check shown above.

## [L-04] Maximum amount of assets that can be deposited into a market is implicitly limited to `uint104`

### Bug Description

According to the [README](https://github.com/code-423n4/2023-10-wildcat/tree/main#additional-context), the amount of assets that can be borrowed in a market should be up to `type(uint128).max`:

> The `totalSupply` is nowhere near 2^128.

Whenever a lender calls `depositUpTo()` to deposit assets, the asset amount is scaled up and added to `scaledTotalSupply`:

[WildcatMarket.sol#L55-L56](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L55-L56)

```solidity
    // Scale the mint amount
    uint104 scaledAmount = state.scaleAmount(amount).toUint104();
```

[WildcatMarket.sol#L70-L71](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L70-L71)

```solidity
    // Increase supply
    state.scaledTotalSupply += scaledAmount;
```

However, `scaledTotalSupply` is a `uint104` instead of `uint128`:

[MarketState.sol#L20-L21](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L20-L21)

```solidity
  // Scaled token supply (divided by scaleFactor)
  uint104 scaledTotalSupply;
```

This means that the maximum amount of assets that can be borrowed through a market is implicitly limited by `type(uint104).max * scaleFactor / 1e27`.

When a market is first deployed, its `scaleFactor` is `1e27`, which limits the maximum amount borrowable to `type(uint104).max`.

### Impact

Borrowers will not be able to borrow more than `type(uint104).max` assets, which limits the functionality of the protocol should an underlying asset have high decimals (eg. 24) and a total supply more than `type(uint104).max`.

### Recommendation

Consider increasing the precision of `scaleFactor`, such as changing it to a `uint128` instead.

## [L-05] Only allow lenders to call `executeWithdrawal()` for themselves

### Bug Description

In `WildcatMarketWithdrawals.sol`, the `executeWithdrawal()` function has no access controls:

[WildcatMarketWithdrawals.sol#L137-L140](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L137-L140)

```solidity
  function executeWithdrawal(
    address accountAddress,
    uint32 expiry
  ) external nonReentrant returns (uint256) {
```

This means that anyone can call the function and specify any lender as the `accountAddress` to withdraw assets to the lender's address.

### Impact

Lenders might not want their assets to be transferred to their address without their permission. For example, a lender's address could be a smart contract wallet that is compromised or going through an upgrade, and is unable to receive funds temporarily.

### Recommendation

Do not allow `executeWithdrawal()` to be called for any lender on their behalf. This can be achieved by removing the `accountAddress` parameter, and using `msg.sender` as the lender's address instead.

## [N-01] Avoid calling `_writeState()` before transferring assets in `borrow()`

`borrow()` calls `_writeState()` before transferring assets to the the borrower:

[WildcatMarket.sol#L128-L129](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L128-L129)

```solidity
    _writeState(state);
    asset.safeTransfer(msg.sender, amount);
```

However, `_writeState()` calls `totalAssets()` when checking if the market is delinquent:

[WildcatMarketBase.sol#L448-L453](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L448-L453)

```solidity
  function _writeState(MarketState memory state) internal {
    bool isDelinquent = state.liquidityRequired() > totalAssets();
    state.isDelinquent = isDelinquent;
    _state = state;
    emit StateUpdated(state.scaleFactor, isDelinquent);
  }
```

`totalAssets()` returns the current asset balance of the market:

[WildcatMarketBase.sol#L238-L240](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L238-L240)

```solidity
  function totalAssets() public view returns (uint256) {
    return IERC20(asset).balanceOf(address(this));
  }
```

Since the transfer of assets is only performed _after_ `_writeState()` is called, `totalAssets()` will be higher than it should be in `_writeState()`. 

Note that there currently isn't any risk of this causing `_writeState()` to update the market's delinquency status incorrectly, since:
- If a market was already delinquent, `borrowableAssets()` would return 0. Hence, `borrow()` cannot possibly update the market from delinquent to non-delinquent.
- `borrowableAssets()` prevents `borrow()` from making the market delinquent, hence the market cannot go from non-delinquent to delinquent.

### Recommendation

Consider calling `_writeState()` after assets have been transferred to the borrower:

[WildcatMarket.sol#L128-L129](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L128-L129)

```diff
-   _writeState(state);
    asset.safeTransfer(feeRecipient, withdrawableFees);
+   _writeState(state);
```

## [N-02] Code for `push()` in `FIFOQueue.sol` can be shortened

The following code:

[FIFOQueue.sol#L55-L59](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L55-L59)

```solidity
  function push(FIFOQueue storage arr, uint32 value) internal {
    uint128 nextIndex = arr.nextIndex;
    arr.data[nextIndex] = value;
    arr.nextIndex = nextIndex + 1;
  }
```

can be shortened to:

```solidity
  function push(FIFOQueue storage arr, uint32 value) internal {
    arr.data[arr.nextIndex++] = value;
  }
```

## [N-03] Redundant checks in `WildcatMarketBase`'s constructor

The constructor of the `WildcatMarketBase` contract contains the following checks:

[WildcatMarketBase.sol#L79-L93](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L79-L93)

```solidity
    if ((parameters.protocolFeeBips > 0).and(parameters.feeRecipient == address(0))) { // redundant
      revert FeeSetWithoutRecipient();
    }
    if (parameters.annualInterestBips > BIP) { // redundant
      revert InterestRateTooHigh();
    }
    if (parameters.reserveRatioBips > BIP) { // redundant
      revert ReserveRatioBipsTooHigh();
    }
    if (parameters.protocolFeeBips > BIP) {
      revert InterestFeeTooHigh();
    }
    if (parameters.delinquencyFeeBips > BIP) { // redundant
      revert PenaltyFeeTooHigh();
    }
```

However, these checks are redundant as they are already accounted for in `WildcatMarketControllerFactory.sol`:

[WildcatMarketControllerFactory.sol#L79-L90](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L79-L90)

```solidity
    if (
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
```

[WildcatMarketControllerFactory.sol#L201-L210](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L201-L210)

```solidity
    bool hasOriginationFee = originationFeeAmount > 0;
    bool nullFeeRecipient = feeRecipient == address(0);
    bool nullOriginationFeeAsset = originationFeeAsset == address(0);
    if (
      (protocolFeeBips > 0 && nullFeeRecipient) ||
      (hasOriginationFee && nullFeeRecipient) ||
      (hasOriginationFee && nullOriginationFeeAsset)
    ) {
      revert InvalidProtocolFeeConfiguration();
    }
```

### Recommendation

Consider removing the checks that are marked as "redundant" above.