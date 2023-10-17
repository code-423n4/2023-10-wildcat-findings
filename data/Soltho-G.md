# Consider using `delete` for `TempMarketParameterStorage` of `WildcatMarketController`

The current approach sets the struct with some predefined custom values. Then this values are modified during execution, a quite gas-expensive approach since we are writing to storage multiple times, plus it uses dynamic-sized types(strings) for `namePrefix`and `symbolPrefix`, so is not a fixed gas cost. And after using it we reset it to the initial custom values, paying the gas of modifying a initialized storage value, again. Instead of that, we could use `delete` keyword after using the temporary storage struct to get some gas refunds for completely reseting the struct to default solidity values and reduce gas costs. The predefined values mentioned before could be saved into another immutable or constant values(the strings can only be constant) that are set in the constructor, so they are cheaper to read from too, and we can completely `delete` the temporary ones.

Simple demo : 
```solidity 
   // we write all the storage struct, very gas expensive and has unknown gas costs,
   // because it contains dynamic-sized values that are determined at runtime
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
   
    // _resetTmpMarketParameters();
    // use delete keyword instead to get a gas refund and reduce gas costs
    delete _tmpMarketParameters;

```
And for the `getMarketParameters` getter that requires those predefined values,we can return the other ones that are immutable and cheaper to read:

```
  // all this values are set in the constructor, they could even be constant instead of immutable if 
  // desired
  address immutable assetMarketParams;
  string namePrefixMarketParams;
  string symbolPrefixMarketParams;
  address immutable feeRecipientMarketParams;
  uint16 immutable protocolFeeBipsMarketParams;
  uint128 immutable maxTotalSupplyMarketParams;
  uint16 immutable annualInterestBipsMarketParams;
  uint16 immutable delinquencyFeeBipsMarketParams;
  uint32 immutable withdrawalBatchDurationMarketParams;
  uint16 immutable reserveRatioBipsMarketParams;
  uint32 immutable delinquencyGracePeriodMarketParams;


 function getMarketParameters() external view returns (MarketParameters memory parameters) {
    parameters.asset = assetMarketParams ;
    parameters.namePrefix = namePrefixMarketParams;
    parameters.symbolPrefix = symbolPrefixMarketParams;
    parameters.borrower = borrower;
    parameters.controller = address(this);
    parameters.feeRecipient = feeRecipientMarketParams;
    parameters.sentinel = sentinel;
    parameters.maxTotalSupply = maxTotalSupplyMarketParams;
    parameters.protocolFeeBips = protocolFeeBipsMarketParams;
    parameters.annualInterestBips = annualInterestBipsMarketParams;
    parameters.delinquencyFeeBips = delinquencyFeeBipsMarketParams;
    parameters.withdrawalBatchDuration = withdrawalBatchDurationMarketParams;
    parameters.reserveRatioBips = reserveRatioBipsMarketParams ;
    parameters.delinquencyGracePeriod = delinquencyGracePeriodMarketParams;
  }
```
Note that this will result in a more expensive deployment but can save a lot of gas in the long run.