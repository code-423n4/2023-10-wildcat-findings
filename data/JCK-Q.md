
## [L-01] The constructor should validate the decimals to make sure that the decimals are not greater than 18

```solidity
file:  src/market/WildcatMarketBase.sol

76     constructor() {
    MarketParameters memory parameters = IWildcatMarketController(msg.sender).getMarketParameters();

    if ((parameters.protocolFeeBips > 0).and(parameters.feeRecipient == address(0))) {
      revert FeeSetWithoutRecipient();
    }
    if (parameters.annualInterestBips > BIP) {
      revert InterestRateTooHigh();
    }
    if (parameters.reserveRatioBips > BIP) {
      revert ReserveRatioBipsTooHigh();
    }
    if (parameters.protocolFeeBips > BIP) {
      revert InterestFeeTooHigh();
    }
    if (parameters.delinquencyFeeBips > BIP) {
      revert PenaltyFeeTooHigh();
    }

    // Set asset metadata
    asset = parameters.asset;
    name = string.concat(parameters.namePrefix, queryName(parameters.asset));
    symbol = string.concat(parameters.symbolPrefix, querySymbol(parameters.asset));
    decimals = IERC20Metadata(parameters.asset).decimals();

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L76-L99


## [L-01] The archController and chainalysisSanctionsList root won’t revert if is set to 0x00

If archController and chainalysisSanctionsList root is set to 0x00 it could leave the system in an insecure state where the system will accept any message that it has never seen before and process it as if it were genuine.

```solidity
file: src/WildcatSanctionsSentinel.sol

24    constructor(address _archController, address _chainalysisSanctionsList) {
    archController = _archController;
    chainalysisSanctionsList = _chainalysisSanctionsList;
    _resetTmpEscrowParams();
  }

```
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L24-L28