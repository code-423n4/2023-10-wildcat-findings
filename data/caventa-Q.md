1.

Only allow close market can be called 1 time. Suggest to change WildcatMarket#closeMarket

```
bool public isMarketClosed = false;

function closeMarket() external onlyController nonReentrant {
  require(!isMarketClosed, "Market already closed");
  
  // ... (existing code)
  
  isMarketClosed = true;
  emit MarketClosed(block.timestamp);
}
```

2.

The 3rd parameter of createEscrow function should be asset. See interface IWildcatSanctionsSentinel#createEscrow

```solidity
  function createEscrow(
    address account,
    address borrower,
    address asset
  ) external returns (address escrowContract);
```

However, the 3rd parameter of createEscrow code in WildcatMarketBase#_blockAccount is address(this).

Suggestion: Change WildcatMarketBase#_blockAccount

```diff
function _blockAccount(MarketState memory state, address accountAddress) internal {
    // Fetch the account details for the given address
    Account memory account = _accounts[accountAddress];

    // Check if the account is already blocked
    if (account.approval != AuthRole.Blocked) {

      // Get the current scaled balance of the account
      uint104 scaledBalance = account.scaledBalance;

      // Set the account's authorization status to 'Blocked'
      account.approval = AuthRole.Blocked;

      // Emit an event to log the account's updated authorization status
      emit AuthorizationStatusUpdated(accountAddress, AuthRole.Blocked);

      // If the account has a non-zero balance, perform further operations
      if (scaledBalance > 0) {

        // Reset the account's scaled balance to 0
        account.scaledBalance = 0;

        // Create an escrow for the blocked account's assets
        address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
          accountAddress,
          borrower,
-          address(this)
+          asset
        );

        // Emit a transfer event showing the transfer of assets from the account to the escrow
        emit Transfer(accountAddress, escrow, state.normalizeAmount(scaledBalance));

        // Update the escrow's balance with the transferred assets
        _accounts[escrow].scaledBalance += scaledBalance;

        // Emit an event to log the sanction and the asset transfer to the escrow
        emit SanctionedAccountAssetsSentToEscrow(
          accountAddress,
          escrow,
          state.normalizeAmount(scaledBalance)
        );
      }

      // Update the account's details in the system
      _accounts[accountAddress] = account;
    }
  }
```