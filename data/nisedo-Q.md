## QA1: Remove unused imports
```diff
File: WildcatMarketBase.sol
- import "../libraries/FeeMath.sol";

File: WildcatMarketControllerFactory.sol
- import {EnumerableSet} from "openzeppelin/contracts/utils/structs/EnumerableSet.sol";
- import "./interfaces/WildcatStructsAndEnums.sol";
- import "./interfaces/IWildcatMarketController.sol";
- import "./interfaces/IWildcatArchController.sol";
- import "./libraries/LibStoredInitCode.sol";
- import "./libraries/MathUtils.sol";
- import "./market/WildcatMarket.sol";

File: WildcatMarketWithdrawals.sol
- import "../libraries/MarketState.sol";
- import "../libraries/FeeMath.sol";
- import "../libraries/FIFOQueue.sol";
- import "../interfaces/IWildcatSanctionsSentinel.sol";

File: WildcatMarketConfig.sol
- import "../libraries/FeeMath.sol";

File: WildcatMarket.sol
- import "../libraries/FeeMath.sol";

File: WildcatSanctionsSentinel.sol
- import {SanctionsList} from "./libraries/Chainalysis.sol";

File: WildcatSanctionsEscrow.sol
- import {IChainalysisSanctionsList} from "./interfaces/IChainalysisSanctionsList.sol";
- import {SanctionsList} from "./libraries/Chainalysis.sol";
```

## QA2: Remove todo comments
[FIFOQueue.sol#L10-L12](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L10-L12)
```solidity
File: FIFOQueue.sol
10: // @todo - make array tightly packed for gas efficiency with multiple reads/writes
11: //         also make a memory version of the array with (nextIndex, startIndex, storageSlot)
12: //         so that multiple storage reads aren't required for tx's using multiple functions
```

## QA3: Remove useless unchecked block
[ReentrancyGuard.sol#L64-L66](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/ReentrancyGuard.sol#L64-L66)
```solidity
File: ReentrancyGuard.sol
64    unchecked {
65      _reentrancyGuard = _ENTERED;
66    }
```

## QA4: Incorrect Documentation on `WildcatMarketConfig.nukeFromOrbit()` Access Control

### Description
The documentation for the function `WildcatMarketConfig.nukeFromOrbit()` claims that the function will revert if the caller is not the sentinel.

> [Reverts if: the caller is not the sentinel.](https://wildcat-protocol.gitbook.io/wildcat/technical-deep-dive/component-overview/wildcat-market-overview/wildcatmarketconfig.sol#nukefromorbit)

However, there is no check to ensure that the caller is the sentinel in the code. This can lead to potential misuse or misunderstanding of the function's access control, as the function relies on the result of `IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)` to determine behavior, rather than directly validating the caller's identity.

```solidity
File: WildcatMarketConfig.sol
  function nukeFromOrbit(address accountAddress) external nonReentrant {
    if (!IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
      revert BadLaunchCode();
    }
    MarketState memory state = _getUpdatedState();
    _blockAccount(state, accountAddress);
    _writeState(state);
  }
```

### Recommendations
1. Update the function `WildcatMarketConfig.nukeFromOrbit()` to include an explicit check to verify that the caller is indeed the sentinel. This can be done using a simple `if` or `require` statement.
2. Update the documentation to accurately reflect the function's behavior.