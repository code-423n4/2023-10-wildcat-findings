## ```WildcatMarketWithdrawals.sol``` can be optimized
If the lender has been blocked, then no need to create escrow for the lender. We just need to get the escrow address by calling ```getEscrowAddress()``` it will save the gas.

i would suggest to change the code to this:
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L137
```

if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
      if (account.approval != AuthRole.Blocked) {
             address escrow = IWildcatSanctionsSentinel(sentinel).getEscrowAddress(
              accountAddress,
              borrower,
              address(asset)
      );
      } else {
             _blockAccount(state, accountAddress);
      }
      asset.safeTransfer(escrow, normalizedAmountWithdrawn);
      emit SanctionedAccountWithdrawalSentToEscrow(
        accountAddress,
        escrow,
        expiry,
        normalizedAmountWithdrawn
      );
}	
```
