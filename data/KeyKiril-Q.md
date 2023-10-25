From security point of view it is better, to place the ***cache the account data*** and ***revert if not authorized*** above the ***scale the mint amount*** and ***transfer deposit from caller***. As when the function ***"depositUpTo"*** in ***"WildcatMarket.sol"*** is executed it will first check if the account is authorized for a deposit.

```
  function depositUpTo(
    uint256 amount
  ) public virtual nonReentrant returns (uint256 /* actualAmount */) {
  
    // Scale the mint amount 
    uint104 scaledAmount = state.scaleAmount(amount).toUint104();
    if (scaledAmount == 0) revert NullMintAmount();

    // Transfer deposit from caller
    asset.safeTransferFrom(msg.sender, address(this), amount);


    // Cache account data and revert if not authorized to deposit.
    Account memory account = _getAccountWithRole(msg.sender, AuthRole.DepositAndWithdraw);
    account.scaledBalance += scaledAmount;
    _accounts[msg.sender] = account;

 
    return amount;
  }
```

The function ***"_getAccount"*** in ***"WildcatMarketBase.sol"*** is not protected with ***"onlyBorrower"***

```
function _getAccount(address accountAddress) internal view returns (Account memory account) {
    account = _accounts[accountAddress];
    if (account.approval == AuthRole.Blocked) {
      revert AccountBlacklisted();
    }
  }
```

The function ***"_blockAccount"*** in ***"WildcatMarketBase.sol"*** is not protected with ***"onlyBorrower"***

```
function _blockAccount(MarketState memory state, address accountAddress) internal {
    Account memory account = _accounts[accountAddress];
    if (account.approval != AuthRole.Blocked) {
      uint104 scaledBalance = account.scaledBalance;
      account.approval = AuthRole.Blocked;
      emit AuthorizationStatusUpdated(accountAddress, AuthRole.Blocked);

      if (scaledBalance > 0) {
        account.scaledBalance = 0;
        address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
          accountAddress,
          borrower,
          address(this)
        );
        emit Transfer(accountAddress, escrow, state.normalizeAmount(scaledBalance));
        _accounts[escrow].scaledBalance += scaledBalance;
        emit SanctionedAccountAssetsSentToEscrow(
          accountAddress,
          escrow,
          state.normalizeAmount(scaledBalance)
        );
      }
      _accounts[accountAddress] = account;
    }
  }
```

The function ***"nukeFromOrbit"*** in ***"WildcatMarketConfig.sol"*** is not protected with ***"onlySentinel"***

```
 function nukeFromOrbit(address accountAddress) external nonReentrant {
    if (!IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
      revert BadLaunchCode();
    }
    MarketState memory state = _getUpdatedState();
    _blockAccount(state, accountAddress);
    _writeState(state);
  }
```

The function ***"stunningReversal"*** in ***"WildcatMarketConfig.sol"*** is not protected with ***"onlyBorrower"***

```
 function stunningReversal(address accountAddress) external nonReentrant {
    Account memory account = _accounts[accountAddress];
    if (account.approval != AuthRole.Blocked) {
      revert AccountNotBlocked();
    }

    if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
      revert NotReversedOrStunning();
    }

    account.approval = AuthRole.Null;
    emit AuthorizationStatusUpdated(accountAddress, account.approval);

    _accounts[accountAddress] = account;
  }
```
