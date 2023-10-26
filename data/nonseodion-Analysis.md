## Suggestions
1. While auditing the codebase I remember it was mentioned somewhere that the limitation of only lenders being able to withdraw from markets would make it difficult for the market token to be used by non-lenders. I've thought about this and the problem can be solved if a borrower registers a contract as one of its lenders. This contract will serve as an intermediary for multiple users to deposit to markets and withdraw. Hence, a liquidity pool can be created on any DEX using the market tokens and anyone holding it can redeem it for the underlying tokens.

## Takeaways
Here's a list of interesting mechanisms I learned while reviewing the codebase.

1. Inversion of Control during contract Deployment
When you deploy a contract with Create2 you can predict its address using the initcodehash, deployer's address and a salt. If the contract has parameters, its initcodehash would be dynamic. Which means you will need to expend gas in adding the parameters to the creation code, generating the initcodehash and use that to generate the address. With inversion of control you can use a fixed initcodehash, the constructor of the contract being deployed will just need to call into `msg.sender` to retrieve its parameters. I first saw it being used in Uniswap V3 to deploy liquidity pools, that's where I got the term "inversion of control".

2. Use a stop opcode if you don't want your code to be run. The codebase puts a stop instruction in front of a contract bytecode which if the contract is just meant to be used to store the bytecode. This prevents anyone from running the code and changing its state.

3. We can leverage the EVM inherent underflow properties to check if a value is zero by subtracting one from it and checking if it's equal to type(uint256).max On its own it may not be useful but may be used to reduce two separate comparisons into one saving gas. This was used in the `hasPendingExpiredBatch` function in MarketState.sol contract.

```
  // Equivalent to expiry > 0 && expiry <= block.timestamp
  result := gt(timestamp(), sub(expiry, 1))
```

## Mechanisim Suggestions
1. Lenders shouldn’t wait for withdrawal delay if the market is closed. This is completely unnecessary as a closed market should already be able to satisfy the withdrawal request of every user.



### Time spent:
30 hours