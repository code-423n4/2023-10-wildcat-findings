0. QA/LOW: WildcatSanctionsEscrow::releaseEscrow() - L38: unsafe use of transfer() function

The transfer() function for ERC-20 tokens is considered unsafe because it lacks error handling and does not provide guarantees against reentrancy attacks. The main reasons why transfer() is unsafe and not recommended include:

Lack of Error Handling: When you use the transfer() function to send tokens, it does not return a boolean value indicating the success or failure of the transfer. If the transfer fails for any reason (e.g., the recipient contract does not accept the tokens), there is no way to detect or handle this failure within the contract. This can lead to unexpected behavior and loss of tokens.

```solidity
  function releaseEscrow() public override {
    if (!canReleaseEscrow()) revert CanNotReleaseEscrow();

    uint256 amount = balance();

    IERC20(asset).transfer(account, amount); /// @audit-issue need to use safeTransfer library either from OZ or Solady

    emit EscrowReleased(account, asset, amount);
  }
```
Recommendation:

To mitigate these issues, it is recommended to use safe token transfer methods provided by established libraries like OpenZeppelin (OZ) or Solady. These safe transfer functions perform the following actions:

Return a boolean value to indicate the success or failure of the transfer.
Include error handling to revert the transaction if the transfer fails, ensuring that the contract's state is not modified in case of failure.
