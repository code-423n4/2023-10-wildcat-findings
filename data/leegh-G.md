Delay the update of storage until just before using it, saving gas when transactions are reverted.
There are 2 instances:
1. *WildcatMarketController.sol* ( [#L343-L354](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L343-L354) ): move L343 before L354, just before calling the create function, saving gas if the intermediate operations revert.
```solidity
343:    _tmpMarketParameters = parameters;
344:
345:    if (originationFeeAsset != address(0)) {
346:      originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);
347:    }
348:
349:    bytes32 salt = _deriveSalt(asset, namePrefix, symbolPrefix);
350:    market = LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt, marketInitCodeHash);
351:    if (market.codehash != bytes32(0)) {
352:      revert MarketAlreadyDeployed();
353:    }
354:    LibStoredInitCode.create2WithStoredInitCode(marketInitCodeStorage, salt);
```


2. *WildcatMarketControllerFactory.sol* ( [#L286-L297](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L286-L297) ): move L286 before L297, just before calling the create function.
```solidity
286:    _tmpMarketBorrowerParameter = msg.sender;
287:    // Salt is borrower address
288:    bytes32 salt = bytes32(uint256(uint160(msg.sender)));
289:    controller = LibStoredInitCode.calculateCreate2Address(
290:      ownCreate2Prefix,
291:      salt,
292:      controllerInitCodeHash
293:    );
294:    if (controller.codehash != bytes32(0)) {
295:      revert ControllerAlreadyDeployed();
296:    }
297:    LibStoredInitCode.create2WithStoredInitCode(controllerInitCodeStorage, salt);
```