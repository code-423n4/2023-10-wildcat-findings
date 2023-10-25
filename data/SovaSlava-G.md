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
```        

### [G-4] Immediately set the correct account status when calling stunningReversal()

This way we will avoid checking account status for AuthRole.Null role  every time we call the _getAccountWithRole() function and check the status only when we call the stunningReversal function. This will reduce gas consumption when calling the _getAccountWithRole function.

```diff

// WildcatMarketBase.sol
  function _getAccountWithRole(
    address accountAddress,
    AuthRole requiredRole
  ) internal returns (Account memory account) {
    account = _getAccount(accountAddress);
-    // If account role is null, see if it is authorized on controller.
-    if (account.approval == AuthRole.Null) {
-      if (IWildcatMarketController(controller).isAuthorizedLender(accountAddress)) {
-        account.approval = AuthRole.DepositAndWithdraw;
-        emit AuthorizationStatusUpdated(accountAddress, AuthRole.DepositAndWithdraw);
-      }
-    }
    // If account role is insufficient, revert.
    if (uint256(account.approval) < uint256(requiredRole)) {
      revert NotApprovedLender();
    }
  }
  
// WildcatMarketConfig.sol
 function stunningReversal(address accountAddress) external nonReentrant {
    Account memory account = _accounts[accountAddress];
    if (account.approval != AuthRole.Blocked) {
      revert AccountNotBlocked();
    }

    if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
      revert NotReversedOrStunning();
    }

-    account.approval = AuthRole.Null;
+     if (IWildcatMarketController(controller).isAuthorizedLender(accountAddress)) {
+        account.approval = AuthRole.DepositAndWithdraw;
+        emit AuthorizationStatusUpdated(accountAddress, AuthRole.DepositAndWithdraw);
+      }
-    emit AuthorizationStatusUpdated(accountAddress, account.approval);

    _accounts[accountAddress] = account;
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketConfig.sol#L88-L102
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L197-L213

        