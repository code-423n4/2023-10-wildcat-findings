## Impact

Potential ways the access roles for market interactions could be maliciously altered. In the `WildcatMarketController` contract, the `updateAccountAuthorization()` function allows updating authorization status for any account on any controlled market. 

This function is not restricted only to the market admin or controller. Any external caller can update any account's authorization.

This could be abused to:

- Revoke authorization for legitimate lenders, blocking them from deposit/withdrawal.

- Grant authorization to malicious accounts, allowing them to deposit/withdraw/borrow without approval. 

For example:

```solidity

// Revoke authorization for a legitimate lender
marketController.updateAccountAuthorization(legitLender, market, false) 

// Authorize a malicious account
marketController.updateAccountAuthorization(maliciousAccount, market, true)

```

Now the legitimate lender would be blocked while the malicious account would have permissions to deposit/withdraw/borrow.

This violates the expected authorization control flow where only admins/controllers can manage access.

## Proof of Concept

**[Snippet](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L182-L190)**

```solidity
  function updateLenderAuthorization(address lender, address[] memory markets) external {

    // Update lender authorization on each market
    for (uint256 i; i < markets.length; i++) {
      address market = markets[i];
      if (!_controlledMarkets.contains(market)) {
        revert NotControlledMarket();
      }
      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
    }
  }
```

Things to note:

- This function is declared `external`, so anyone can call it. 

- It loops through the provided `markets` array and calls the `WildcatMarket.updateAccountAuthorization()` function on each one.

- The `lender` and `isAuthorizedLender(lender)` params control what authorization is set on each market.

- There is no restriction on who can call this function or what `lender`/authorization they can set.

So, the `external` declaration at line 1 combined with the lack of access controls allows any caller to update any account's authorization on any controlled market.

## Recommended Mitigation Steps

`updateAccountAuthorization` should be restricted to only controller or market admin roles.

Additional context:

- The impact could be amplified by first deploying a malicious market controller, then using it to update permissions.

- Affected markets may rely on the controller for proper access control. Malicious updates would violate that trust.