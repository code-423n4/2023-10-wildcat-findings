# [QA-01] `WildcatMarketToken.sol` function `_transfer` has underflow risk

## Lines of code

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L73

## Impact

In the file `WildcatMarketToken.sol`, function `_transfer` doesn’t check `scaledAmount` is less than or equal to `fromAccount.scaledBalance`. If someone tries to transfer amount greater than scaledBalance, the function will revert with `Reason: Arithmetic over/underflow`, such vague error message will confuse users and make it hard to debug.

## Proof of Concept

In the function `_transfer`, `scaledAmount` is subtracted from `fromAccount`, but the code does not check if `scaledAmount` is less than or equal to `fromAccount.scaledBalance`. 

For example, `scaledAmount` is 200, while `fromAccount.scaledBalance` is 150, function will abort with error `Reason: Arithmetic over/underflow`, which may confuse users.

```
function _transfer(address from, address to, uint256 amount) internal virtual {
    MarketState memory state = _getUpdatedState();
    uint104 scaledAmount = state.scaleAmount(amount).toUint104();

    if (scaledAmount == 0) {
      revert NullTransferAmount();
    }

    Account memory fromAccount = _getAccount(from);
    fromAccount.scaledBalance -= scaledAmount;
    _accounts[from] = fromAccount;

    Account memory toAccount = _getAccount(to);
    toAccount.scaledBalance += scaledAmount;
    _accounts[to] = toAccount;

    _writeState(state);
    emit Transfer(from, to, amount);
}
```

## Tools Used

Manual analysis.

## Recommended Mitigation Steps

Compare `scaledAmount` and `fromAccount.scaledBalance` before updating, throw appropriate error if `scaledAmount` is greater than `fromAccount.scaledBalance`. 

```
function _transfer(address from, address to, uint256 amount) internal virtual {
    MarketState memory state = _getUpdatedState();
    uint104 scaledAmount = state.scaleAmount(amount).toUint104();

    if (scaledAmount == 0) {
      revert NullTransferAmount();
    }

    Account memory fromAccount = _getAccount(from);
    if (fromAccount.scaledBalance < scaledAmount) {
      revert InsufficientBalance(from, fromAccount.scaledBalance, scaledAmount);
    }
    fromAccount.scaledBalance -= scaledAmount;
    _accounts[from] = fromAccount;

    Account memory toAccount = _getAccount(to);
    toAccount.scaledBalance += scaledAmount;
    _accounts[to] = toAccount;

    _writeState(state);
    emit Transfer(from, to, amount);
}

// error definition
error InsufficientBalance(address indexed from, uint256 fromBalance, uint256 value);
```


# [QA-02] `WildcatMarketToken.sol` function `transferFrom` has underflow risk

## Lines of code

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L73

## Impact

In the file `WildcatMarketToken.sol`, function `_transfer` doesn’t check `scaledAmount` is less than or equal to `fromAccount.scaledBalance`. If someone tries to transfer amount greater than scaledBalance, the function will revert with `Reason: Arithmetic over/underflow`, such vague error message will confuse users and make it hard to debug.

## Proof of Concept

In the function `_transfer`, `scaledAmount` is subtracted from `fromAccount`, but the code does not check if `scaledAmount` is less than or equal to `fromAccount.scaledBalance`. 

For example, `scaledAmount` is 200, while `fromAccount.scaledBalance` is 150, function will abort with error `Reason: Arithmetic over/underflow`, which may confuse users.

```
function transferFrom(
    address from,
    address to,
    uint256 amount
  ) external virtual nonReentrant returns (bool) {
    uint256 allowed = allowance[from][msg.sender];

    // Saves gas for unlimited approvals.
    if (allowed != type(uint256).max) {
      uint256 newAllowance = allowed - amount;
      _approve(from, msg.sender, newAllowance);
    }

    _transfer(from, to, amount);

    return true;
}
```

## Tools Used

Manual analysis.

## Recommended Mitigation Steps

Compare `amount` with `allowed` before updating, throw appropriate error if `scaledAmount` is greater than `fromAccount.scaledBalance`. 

```
function transferFrom(
    address from,
    address to,
    uint256 amount
  ) external virtual nonReentrant returns (bool) {
    uint256 allowed = allowance[from][msg.sender];

    // Saves gas for unlimited approvals.
    if (allowed != type(uint256).max) {
      if (allowed < amount) {
        revert InsufficientAllowance(msg.sender, allowed, amount);
      }
      uint256 newAllowance = allowed - amount;
      _approve(from, msg.sender, newAllowance);
    }

    _transfer(from, to, amount);

    return true;
}

// error definition
error InsufficientAllowance(address indexed spender, uint256 allowed, uint256 value);
```