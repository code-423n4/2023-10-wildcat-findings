# QA Report

| Id                                                                   | Issue                                                   |
|----------------------------------------------------------------------|---------------------------------------------------------|
| [L-1](#l-1-wildcatmarketcontroller-does-not-implement-marketfactory) | WildcatMarketController does not implement marketFactory |
| [L-2](#l-2-not-every-contract-is-a-wildcatsanctionsescrow)       | Not Every contract is a WildcatSanctionsEscrow       |
| [NC-1](#nc-1-contract-should-inherit-interface)                      | Contract should inherit interface                       |


## Low Risk 

### L-1 WildcatMarketController does not implement marketFactory
`WildcatMarketController` is cast as `IWildcatMarketController` in code, however it does not have a `public` field of `marketFactory` 
or an equivalent function. 

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/interfaces/IWildcatMarketControllerFactory.sol#L24-L25
``` Solidity
  // Returns market factory used by controller
  function marketFactory() external view returns (address);
  ```

Although `IWildcatMarketController.marketFactory` is never called within the code under audit, it would be fair to assume
that `WildcatMarketController` does implement `IWildcatMarketController`.

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L269-L30
It is not immediately clear how a market factory can be referenced without the including borrower that is part of the salt,
and without an function implementation it cannot currently be being used (i.e. another protocol or front end).

Two options:
1) If it is not going to be needed: Delete the function.
2) It is desired for the future: Add the borrower as input and use the salt calculation to return the `CREATE2` address that the market controller would be deployed to.


### L-2 Not every contract is a WildcatSanctionsEscrow

When a `WildcatSanctionsEscrow` is deployed by the `WildcatMarketSentinel` in the create escrow use case (lender is on 
the ChainAnalysis sanction list and the appropriate update state is made), the address is generated and if any code is 
present it is assumed to be the corresponding `WildcatSanctionsEscrow`.

While mathematically highly improbable it is possible for address collisions, and when the inputs are known (as they are
in this case), offline brute force can be attempted to deploy a contract at that address.

`WildcatMarketSentinel.createEscrow` uses the `getEscrow address`
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsSentinel.sol#L61-L85

If any code is found then returns the contract assuming it is the corresponding `WildcatSanctionsEscrow` 
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsSentinel.sol#L106-L106

Although there is sizable state space to search through, combining divide and conquer techniques with distributed computing offline
(i.e. without any chain dependency), it is possible to search through the entire salt state space in years.
A collision is not guaranteed with the brute force approach, however neither is it impossible.

#### Recommended Mitigation
Keep a mapping of `WildcatSanctionsEscrow` addresses created by the `WildcatSanctionsSentinel` and only return addresses
found in there.
When/if the address is not in the mapping, but a contract is already deployed (collision case), then the subsequent 
deployment attempt would revert, avoiding transferring market token ownership to an unknown contract.


## Non-Critical

### NC-1 Contract should inherit interface
There are number of contracts that implement interfaces, but they do not inherit the interface, which is a missed 
opportunity.

Benefits include:
- Shift-left testing; During contract compilation if a contract fails to adhere to it's inherited interfaces, 
then compilation will fail, moving what would be a runtime error to a compile time error.
- Single source of truth; NatSpec con reside solely in the interface, providing that single point of function documentation.
- Code reuse; Errors and Events can be defined solely in the interface, rather than both the contract and interface. 
- Explicit relationships; while any interface can be applied to a contract, by stating the inheritance you're being 
clear with intended usage.

#### Instances
- `WildcatMarketController` and `IWildcatMarketController`
- `WildcatArchController`and `IWildcatArchController`
- `WildcatMarketControllerFactory` and `IWildcatMarketControllerFactory`

Minor addition is needed in `WildcatArchController` to work around `owner()` conflict.
``` Solidity
  function owner() override(Ownable, IWildcatArchController) public view returns (address){
    return Ownable.owner();
  }
```

WildcatMarketControllerFactory needs explicit getter functions rather than relying on the implicitly generates ones
(as the scope of the immutables differ to the interface), also adjusting variable names.

Following the [Solidity Naming Conventions](https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables),
change the scope of the variables from `pubilc` to `private`, also adding the underscore during refactoring (updating references within the file), then add the getters.
``` Solidity
  function archController() external view returns (address){
    return address(_archController);
  }

  function sentinel() external view returns (address){
    return _sentinel;
  }
```

