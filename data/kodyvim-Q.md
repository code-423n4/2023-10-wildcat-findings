## Allow borrowers to specify loan recipient
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L129
```solidity
function borrow(uint256 amount) external onlyBorrower nonReentrant {
    MarketState memory state = _getUpdatedState();
    if (state.isClosed) {
      revert BorrowFromClosedMarket();
    }
    uint256 borrowable = state.borrowableAssets(totalAssets());
    if (amount > borrowable) {
      revert BorrowAmountTooHigh();
    }
    _writeState(state);
@>    asset.safeTransfer(msg.sender, amount);
    emit Borrow(amount);
  }
```
Allow borrowers to specify the address to receive loans to avoid tokens that have blacklist restrictions from impairing the protocol functionality.