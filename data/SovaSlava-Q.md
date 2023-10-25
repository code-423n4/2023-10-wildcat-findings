# QA Report

### [Q-1] No need use `unchecked` construction
The construction is used for arithmetic operations to disable overflow checking. You don't need to use it for assignment. It doesn't make sense.
```diff
  function _setReentrancyGuard() internal {
    // Ensure that the reentrancy guard is not already set.
    _assertNonReentrant();

    // Set the reentrancy guard.
-    unchecked {
      _reentrancyGuard = _ENTERED;
-    }
  }
```
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/ReentrancyGuard.sol#L59-L66


### [Q-2] Functions  have no modifier reentrantview
If a contract uses reentrant protection, then all functions must have such protection.
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketConfig.sol#L30-L44

### [Q-3] A non-lender, having market tokens, cannot cash them out
If a lender has transferred market tokens to another person who is not an authorized lender in the same market, then he will not be able to receive an asset for these tokens. It is recommended that when non-lender(but who have tokens) calling the function queueWithdrawal(), check for balance of tokens from this market; if balance > 0, set role for this account as AuthRole.WithdrawOnly.
