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