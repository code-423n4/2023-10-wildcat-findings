## [L-01] Stop Using Solidity's transfer() Now
**Proof Of Concept**
https://consensys.io/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

```
  emit Transfer(accountAddress, escrow, state.normalizeAmount(scaledBalance));
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L177C7-L177C85

```
  emit Transfer(msg.sender, address(this), amount);

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L91C3-L92C1

```
   emit Transfer(address(0), msg.sender, amount);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L67C2-L67C51

```
 function transfer(address to, uint256 amount) external virtual nonReentrant returns (bool) {
    _transfer(msg.sender, to, amount);
    return true;
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L36C2-L38C17

```
 emit Transfer(from, to, amount);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L81C4-L81C37
```
   IERC20(asset).transfer(account, amount);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L38C2-L38C45


## [L-02] Keccak Constant values should used to immutable rather than constant
There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

```
bytes32 public constant override WildcatSanctionsEscrowInitcodeHash =
    keccak256(type(WildcatSanctionsEscrow).creationCode);

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L11C3-L13C1

## [L-03] Function Calls in Loop Could Lead to Denial of Service
Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to the gas limitations or failed transactions. Here are some of the instances entailed:

```
 for (uint256 i; i < markets.length; i++) {
      address market = markets[i];
      if (!_controlledMarkets.contains(market)) {
        revert NotControlledMarket();
      }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L183C4-L187C8

# Non- Critical 


## [N-01] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier.

```
uint256 constant BIP = 1e4;
uint256 constant HALF_BIP = 0.5e4;

uint256 constant RAY = 1e27;
uint256 constant HALF_RAY = 0.5e27;

uint256 constant BIP_RAY_RATIO = 1e23;
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L6C1-L12C39


## [N-02] For modern and more readable code; update import usages

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
*Recommendation*
`import {contract1 , contract2} from "filename.sol";`

```
import 'solady/utils/SafeTransferLib.sol';
import './market/WildcatMarket.sol';
import './interfaces/IWildcatArchController.sol';
import './interfaces/IWildcatMarketControllerEventsAndErrors.sol';
import './interfaces/IWildcatMarketControllerFactory.sol';
import './libraries/LibStoredInitCode.sol';
import './libraries/MathUtils.sol';

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L5C1-L12C1

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L4C1-L5C38

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L7C1-L9C54

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L11C1-L12C37

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L5C1-L11C40

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L5C1-L6C36

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L4C1-L10C1

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L4C1-L4C23

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/SafeCastLib.sol#L4C1-L4C23

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L4C1-L6C28

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L3C1-L7C34

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L4C1-L9C1

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L5C1-L8C1

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L4

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol#L4C1-L5C26

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Chainalysis.sol#L4



##  [N-03] Use a single file for all system-wide constants
There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

## [N-04] NatSpec comments should be increased in contracts
> all contest

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity Official documentation

in complext project such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html