## Low Issues


| |Issue|Instances|Gas|
|-|:-|:-:|:-|
| [GAS-1](#GAS-1) | Gas savings can be achieved by changing the model for assigning value to the structure | 2 | 246 |
| [GAS-2](#GAS-2) | No need to load the whole structure | 1 | 60 |
| [GAS-3](#GAS-3) | No need to cache operation result | 1 | 24 |

Total: 4 instances over 3 issues

### <a name="GAS-1"></a>[GAS-1] Gas savings can be achieved by changing the model for assigning value to the structure ***123 gas***
You can reduce gas by changing the following code :

Instance 1: 
From :
```solidity
File: src/WildcatMarketControllerFactory.sol

    _protocolFeeConfiguration = ProtocolFeeConfiguration({
      feeRecipient: feeRecipient,
      originationFeeAsset: originationFeeAsset,
      originationFeeAmount: originationFeeAmount,
      protocolFeeBips: protocolFeeBips
    });

```

To :
```solidity
File: src/WildcatMarketControllerFactory.sol

    _protocolFeeConfiguration.feeRecipient= feeRecipient;
    _protocolFeeConfiguration.originationFeeAsset= originationFeeAsset;
    _protocolFeeConfiguration.originationFeeAmount= originationFeeAmount;
    _protocolFeeConfiguration.protocolFeeBips= protocolFeeBips;

```
[Link to code](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L211)

Instance 2:

From :
```solidity
File: src/market/WildcatMarketBase.sol

    _state = MarketState({
      isClosed: false,
      maxTotalSupply: parameters.maxTotalSupply,
      accruedProtocolFees: 0,
      normalizedUnclaimedWithdrawals: 0,
      scaledTotalSupply: 0,
      scaledPendingWithdrawals: 0,
      pendingWithdrawalExpiry: 0,
      isDelinquent: false,
      timeDelinquent: 0,
      annualInterestBips: parameters.annualInterestBips,
      reserveRatioBips: parameters.reserveRatioBips,
      scaleFactor: uint112(RAY),
      lastInterestAccruedTimestamp: uint32(block.timestamp)
    });

```

To :
```solidity
File: src/market/WildcatMarketBase.sol

     _state.isClosed= false;
      _state.maxTotalSupply= parameters.maxTotalSupply;
      _state.accruedProtocolFees= 0;
      _state.normalizedUnclaimedWithdrawals= 0;
      _state.scaledTotalSupply= 0;
      _state.scaledPendingWithdrawals= 0;
      _state.pendingWithdrawalExpiry= 0;
      _state.isDelinquent= false;
      _state.timeDelinquent= 0;
      _state.annualInterestBips= parameters.annualInterestBips;
      _state.reserveRatioBips= parameters.reserveRatioBips;
      _state.scaleFactor= uint112(RAY);
      _state.lastInterestAccruedTimestamp= uint32(block.timestamp);

```
[Link to code](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L101)

[PASS] test_deployMarket_OriginationFee() (gas: 5510039)
[PASS] test_deployMarket_OriginationFee() (gas: 5509916)

### <a name="GAS-2"></a>[GAS-2] No need to load the whole structure
When only one or two items from a structure are used, it is more gas efficient to only load those items instead of the whole structure. save ***60 gas***

Instance 1:
function `resetReserveRatio()` only use `expiry` and `reserveRatioBips` from the market structure. Therefore, you can change the following code :

```solidity

  function resetReserveRatio(address market) external virtual {
    TemporaryReserveRatio memory tmp = temporaryExcessReserveRatio[market];
    if (tmp.expiry == 0) {
      revertWithSelector(AprChangeNotPending.selector);
    }
    if (block.timestamp < tmp.expiry) {
      revertWithSelector(ExcessReserveRatioStillActive.selector);
    }

    WildcatMarket(market).setReserveRatioBips(uint256(tmp.reserveRatioBips).toUint16());
    delete temporaryExcessReserveRatio[market]; //@audit-issue Gas/NC : you can reduce gas fees by removing this line as this line does not remove the variable anyway
  }

```

To the following code:

```solidity

  function resetReserveRatio(address market) external virtual {
    uint256 tmpExpiry = temporaryExcessReserveRatio[market].expiry;
    uint256 tmpRatio = temporaryExcessReserveRatio[market].reserveRatioBips;
    if (tmpExpiry == 0) {
      revertWithSelector(AprChangeNotPending.selector);
    }
    if (block.timestamp < tmpExpiry) {
      revertWithSelector(ExcessReserveRatioStillActive.selector);
    }

    WildcatMarket(market).setReserveRatioBips(tmpRatio.toUint16());
    delete temporaryExcessReserveRatio[market]; //@audit-issue Gas/NC : you can reduce gas fees by removing this line as this line does not remove the variable anyway
  }

```

[PASS] test_resetReserveRatio_NotPending() (gas: 12942)
[PASS] test_resetReserveRatio_NotPending() (gas: 12882)

[Link to code](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L491)

### <a name="GAS-3"></a>[GAS-3] No need to cache operation result
When the result of an arithmetique operation is only used once, then it is more gas efficient to inject that operation directly, rather than caching its result. you can save ***24 gas***

Change the following code:
```solidity
  
  function withdrawableProtocolFees(
    MarketState memory state,
    uint256 totalAssets
  ) internal pure returns (uint128) {
    uint256 totalAvailableAssets = totalAssets - state.normalizedUnclaimedWithdrawals;
    return uint128(MathUtils.min(totalAvailableAssets, state.accruedProtocolFees));
  }

```
To the following code:
```solidity

  function withdrawableProtocolFees(
    MarketState memory state,
    uint256 totalAssets
  ) internal pure returns (uint128) {
    return uint128(MathUtils.min(totalAssets - state.normalizedUnclaimedWithdrawals, state.accruedProtocolFees));
  }

```

[PASS] test_withdrawableProtocolFees() (gas: 283999)
[PASS] test_withdrawableProtocolFees() (gas: 283973)

[Link to code](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L105)