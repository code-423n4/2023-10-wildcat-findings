Missing event for critical changes.
Events should be emitted when critical changes are made to the contracts.

There are 2 instances in `WildMarketControllerFactory`

```
288:  function deployController() public returns (address controller) {
    //@audit can we find a way to by-pass the check? -> this will result in attacker being able to pass malicious addresses
    if (!archController.isRegisteredBorrower(msg.sender)) {  
      revert NotRegisteredBorrower();
    }
    _tmpMarketBorrowerParameter = msg.sender;
    // Salt is borrower address
    bytes32 salt = bytes32(uint256(uint160(msg.sender)));
    controller = LibStoredInitCode.calculateCreate2Address(
      ownCreate2Prefix,
      salt,
      controllerInitCodeHash
    );
    if (controller.codehash != bytes32(0)) {
      revert ControllerAlreadyDeployed();
    }
    LibStoredInitCode.create2WithStoredInitCode(controllerInitCodeStorage, salt);
    _tmpMarketBorrowerParameter = address(1);
    archController.registerController(controller);
    _deployedControllers.add(controller);
308:  }
```

```
326:  function deployControllerAndMarket(
    string memory namePrefix,
    string memory symbolPrefix,
    address asset,
    uint128 maxTotalSupply,
    uint16 annualInterestBips,
    uint16 delinquencyFeeBips,
    uint32 withdrawalBatchDuration,
    uint16 reserveRatioBips,
    uint32 delinquencyGracePeriod
  ) external returns (address controller, address market) {
    controller = deployController();
    market = IWildcatMarketController(controller).deployMarket(
      asset,
      namePrefix,
      symbolPrefix,
      maxTotalSupply,
      annualInterestBips,
      delinquencyFeeBips,
      withdrawalBatchDuration,
      reserveRatioBips,
      delinquencyGracePeriod
    );                                                                                                                                                                                                                                                                           
349:  }
```

Fix: add NewController event to the two functions
```
event NewController(address borrower, address controller, string namePrefix, string symbolPrefix);
```