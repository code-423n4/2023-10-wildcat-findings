# L-01 Controller can't call `setMaxTotalSupply` due to lack of function to do it.

## Lines of code
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketConfig.sol#L134

## Impact
In `setMaxTotalSupply()` function in `WildcatMarketConfig.sol` contract there is a modifier `onlyController`. It means that only controller can call it.

```solidity
function setMaxTotalSupply(uint256 _maxTotalSupply) external onlyController nonReentrant {
    MarketState memory state = _getUpdatedState();

    if (_maxTotalSupply < state.totalSupply()) {
      revert NewMaxSupplyTooLow();
    }

    state.maxTotalSupply = _maxTotalSupply.toUint128();
    _writeState(state);
    emit MaxTotalSupplyUpdated(_maxTotalSupply);
  }
```

However, controller can't set `maxTotalSupply` parameter since there is no any logic to do it in the `WildcatMarketController.sol` contract.

## Recommendation
Сonsider to implement a function in the `WildcatMarketController.sol` contract so that it is possible to change the `maxTotalSupply` parameter.


# L-02 Lack of important checks in transfer/transferFrom functions.

## Impact 
It's possible to have a revert in situation when caller hasn't enough balance to transfer.

```solidity
Account memory fromAccount = _getAccount(from);
fromAccount.scaledBalance -= scaledAmount; // @audit there is no underflow check
_accounts[from] = fromAccount;
```

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L64-L82

Moreover, it's also possible to have an error when there is not enough allowance in `transferFrom()` function:

```solidity
if (allowed != type(uint256).max) {
    uint256 newAllowance = allowed - amount; // @audit no underflow check
    _approve(from, msg.sender, newAllowance);
}
```

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L41-L57

## Recommendation
Consider to implement a statement to check that sender has enought value.

```solidity
function _transfer(address from, address to, uint256 amount) internal virtual {
    MarketState memory state = _getUpdatedState();
    uint104 scaledAmount = state.scaleAmount(amount).toUint104();

    if (scaledAmount == 0) {
      revert NullTransferAmount();
    }

    Account memory fromAccount = _getAccount(from);
++  if (fromAccount.scaledBalance < scaledAmount) revert InvalidScaledAmount();
    fromAccount.scaledBalance -= scaledAmount;
    _accounts[from] = fromAccount;

    Account memory toAccount = _getAccount(to);
    toAccount.scaledBalance += scaledAmount;
    _accounts[to] = toAccount;

    _writeState(state);
    emit Transfer(from, to, amount);
    }
```

And also think about to implement check that sender has enough allowance in `transferFrom()` function:

```solidity
    function transferFrom(
        address from,
        address to,
        uint256 amount
    ) external virtual nonReentrant returns (bool) {
        uint256 allowed = allowance[from][msg.sender];

++      if (allowed < amount) revert InvalidAllowance();
        // Saves gas for unlimited approvals.
        if (allowed != type(uint256).max) {
        uint256 newAllowance = allowed - amount;
        _approve(from, msg.sender, newAllowance);
        }

        _transfer(from, to, amount);

        return true;
    }
```


# L-03 Lack of check that `end` value more than `start` value when getting arrays information.

## Impact
Unexpected error can happened due to the fact that there is no check that `end` value is more than `start`.

```solidity
function getRegisteredBorrowers(
    uint256 start,
    uint256 end
) external view returns (address[] memory arr) {
    uint256 len = _borrowers.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start; // @audit it can underflow
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
        arr[i] = _borrowers.at(start + i);
    }
}
```

This problem exists in the next several places:

In `WildcatArchController.sol`:

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L85-L96

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L128-L139

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L171-L182

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L214-L225


In `WildcatMarketController.sol`

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L125-L136

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L204-L215


In `WildcatMarketControllerFactory.sol`

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L138-L149


## Recommendation
Consider to add `if` condition with custom error in case of revert, e.g: 

```solidity
function getDeployedControllers(
    uint256 start,
    uint256 end
) external view returns (address[] memory arr) {
++  if (start >= end) revert InvalidInputData();
    uint256 len = _deployedControllers.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
        arr[i] = _deployedControllers.at(start + i);
    }
}
```


