# [L-01] The ERC20 approve() function doesn't check for if spender address is zero.

The checking of zero address is a common security practice to prevent certain actions that could be malicious or unintentional. Typically, you want to avoid allowing transfers, approvals, or other token-related operations to this zero address to protect the integrity of the token.

Use a check for spender in approve function ```require(spender != address(0));```
line of code : https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L59-L61

# [L-02] approver should identify as `msg.sender`

This line of code 'https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L60' doesn't follow the ERC20 token standard. As `approver` is the owner so it is recommended to identify approver as msg.sender .

# [L-03] Inconsistencies in data types

Inconsistencies in data types can introduce vulnerabilities. In the contract `WildcatMarketWithdrawals.sol` a data type used `uint32` as expiry but in `WildcatMarketController.sol` the expiry data type used is `uint128`. It could lead to incorrect comparisons or calculations when these contracts interact. Using consistent data types makes the code more readable and understandable for both developers and auditors. It's easier to identify the purpose of a variable when the naming and data type conventions are consistent throughout the codebase.

lines : https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketWithdrawals.sol#L29, 
https://github.com/c4ode-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L15

# [L-04] Initialize missing of state variable in `WildcatSanctionsSentinel.sol`

It's essential to ensure that the `tmpEscrowParams` variable is properly initialized before use. In the constructor, `_resetTmpEscrowParams()` is called to initialize it, but it should also be initialized explicitly at the declaration point.
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsSentinel.sol#L18
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsSentinel.sol#L24-L28






