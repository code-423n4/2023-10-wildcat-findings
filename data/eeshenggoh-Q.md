## Summary
Redundant checks should only check for zero address. This also waste deployment gas fees due to size of contract

## Proof of Concept

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L201C1-L208C8

## Recommended Mitigation Steps
Remove other checks
```
       bool hasOriginationFee = originationFeeAmount > 0;
       ...
      (hasOriginationFee && nullFeeRecipient) ||
      (hasOriginationFee && nullOriginationFeeAsset)

```