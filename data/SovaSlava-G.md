# Gass report


### [G-1] Unnecessary permission check
Сhecking that the market belongs to the controller is not needed because the market contract itself has an `onlycontroller` modifier in the `updateAccountAuthorization` function.

```solidity
// WildcatMarketController.sol
  function updateLenderAuthorization(address lender, address[] memory markets) external {
    for (uint256 i; i < markets.length; i++) {
      address market = markets[i];
        // @audit-issue Unnecessary permission check (Remove)    <------
      if (!_controlledMarkets.contains(market)) {
        revert NotControlledMarket();
      }
      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
    }
  }

// WildcatMarketConfig.sol
// This contract has imported from WildcatMarket.sol
  function updateAccountAuthorization(
    address _account,
    bool _isAuthorized
  ) external onlyController nonReentrant {  <---------
  ...
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L182-L188


### [G-2] Unnecessary variable reset
It is not necessary to reset the variable's value, because the new value will only be needed the next time the function is called. The next time the function is called, the required value will be written to the variable.
```diff
function deployController() public returns (address controller) {
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
-  _tmpMarketBorrowerParameter = address(1);
    archController.registerController(controller);
    _deployedControllers.add(controller);
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L282-L300

### [G-3] Unnecessary cast hex value to bytes
```diff
function getEscrowAddress(
    address borrower,
    address account,
    address asset
  ) public view override returns (address escrowAddress) {
    return
      address(
        uint160(
          uint256(
            keccak256(
              abi.encodePacked(
-               bytes1(0xff),
+               hex"ff" 
                address(this),
                keccak256(abi.encode(borrower, account, asset)),
                WildcatSanctionsEscrowInitcodeHash
              )
            )
          )
        )
      );
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsSentinel.sol#L65-L85

### [G-4] Move the array length check to the beginning of the function.
if we move the check for the length of _withdrawalData.unpaidBatches to the beginning of the function, then if the length is 0, then there will be an immediate revert, and no gas will be wasted on reading state from storage. 
```diff
 function closeMarket() external onlyController nonReentrant {
+    if (_withdrawalData.unpaidBatches.length() > 0) {
+      revert CloseMarketWithUnpaidWithdrawals();
+   }
    MarketState memory state = _getUpdatedState();
    state.annualInterestBips = 0;
    state.isClosed = true;
    state.reserveRatioBips = 0;
-    if (_withdrawalData.unpaidBatches.length() > 0) {
-      revert CloseMarketWithUnpaidWithdrawals();
-   }
    uint256 currentlyHeld = totalAssets();
    uint256 totalDebts = state.totalDebts();
    if (currentlyHeld < totalDebts) {
      // Transfer remaining debts from borrower
      asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);
    } else if (currentlyHeld > totalDebts) {
      // Transfer excess assets to borrower
      asset.safeTransfer(borrower, currentlyHeld - totalDebts);
    }
    _writeState(state);
    emit MarketClosed(block.timestamp);
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarket.sol#L142-L160


### [G-5] Unnecessary IF in setReserveRatioBips()
There is no need to check that _reserveRatioBips > initialReserveRatioBips because we know this since we got to this point in the code. Gass diffs show that optimized code takes gas less. 
 test_setReserveRatioBips_InsufficientReservesForNewLiquidityRatio()  359349  ->  359327 
 test_setReserveRatioBips_InsufficientReservesForOldLiquidityRatio()  605989  ->  605980 
 test_setReserveRatioBips_NotController(uint16)  the same
 test_setReserveRatioBips_ReserveRatioBipsTooHigh the same
```diff
 function setReserveRatioBips(uint16 _reserveRatioBips) public onlyController nonReentrant {
    if (_reserveRatioBips > BIP) {
      revert ReserveRatioBipsTooHigh();
    }

    MarketState memory state = _getUpdatedState();

    uint256 initialReserveRatioBips = state.reserveRatioBips;

    if (_reserveRatioBips < initialReserveRatioBips) {
      if (state.liquidityRequired() > totalAssets()) {
        revert InsufficientReservesForOldLiquidityRatio();
      }
    }
    state.reserveRatioBips = _reserveRatioBips;
-    if (_reserveRatioBips > initialReserveRatioBips) {
      if (state.liquidityRequired() > totalAssets()) {
        revert InsufficientReservesForNewLiquidityRatio();
      }
-    }
    _writeState(state);
    emit ReserveRatioBipsUpdated(_reserveRatioBips);
  }

```
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketConfig.sol#L171C4-L192