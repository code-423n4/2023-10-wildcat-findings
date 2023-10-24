###  SafeTransferLib Does Not Ensure That the Token Contract Exists

#### Impact:
The SafeTransferLib used in this code does not verify if the token contract exists, potentially leading to issues when transferring tokens. It's essential to ensure the token's contract existence before making transfers.

*Instances (1)*:
```solidity
File: WildcatMarketController.sol

346:       originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);

```