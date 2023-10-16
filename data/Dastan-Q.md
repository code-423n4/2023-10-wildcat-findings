## LOW RISK ISSUE
## Unspecific Compiler Version Pragma

### Description

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included
with multiple different versions of applications, it may be a security risk for
application implementations.

A known vulnerable compiler version may accidentally be selected or security
tools might fall-back to an older compiler version ending up checking
a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

### Example

🤦 Bad:
```solidity
pragma solidity ^0.8.0;
```

🚀 Good:
```solidity
pragma solidity 0.8.4;
```

### Background Information

- [Consensys Audit of 1inch](https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#unspecific-compiler-version-pragma)
- [Solidity docs](https://docs.soliditylang.org/en/latest/layout-of-source-files.html?highlight=pragma#version-pragma)


## Files analyzed
- ./src/libraries/Chainalysis.sol: - [Link for reference](https://github.com/code-423n4/2023-10-wildcat/blob/18bda5daf6a668cfe5dd6c0749a908e193e7b90c/src/libraries/Chainalysis.sol#L2)
- ./src/libraries/BoolUtils.sol : - [Link for reference](https://github.com/code-423n4/2023-10-wildcat/blob/18bda5daf6a668cfe5dd6c0749a908e193e7b90c/src/libraries/BoolUtils.sol#L2)
/src/libraries/MathUtils.sol::2 => pragma solidity >=0.8.20: - [Link for reference](https://github.com/code-423n4/2023-10-wildcat/blob/18bda5daf6a668cfe5dd6c0749a908e193e7b90c/src/libraries/MathUtils.sol#L2)


## GAS REPORT

Use `!= 0` instead of `> 0` for Unsigned Integer Comparison

### Description

When dealing with unsigned integer types, comparisons with `!= 0` are cheaper
than with `> 0`.

### Example

🤦 Bad:
```solidity
// `a` being of type unsigned integer
require(a > 0, "!a > 0");
```

🚀 Good:
```solidity
// `a` being of type unsigned integer
require(a != 0, "!a > 0");
```

./src/libraries/FeeMath.sol::67 => if (timeWithPenalty > 0) - [Link for reference](https://github.com/code-423n4/2023-10-wildcat/blob/18bda5daf6a668cfe5dd6c0749a908e193e7b90c/src/libraries/FeeMath.sol#L67)
./src/libraries/FeeMath.sol::155 => if (protocolFeeBips > 0) - [Link for reference](https://github.com/code-423n4/2023-10-wildcat/blob/18bda5daf6a668cfe5dd6c0749a908e193e7b90c/src/libraries/FeeMath.sol#L155)
./src/libraries/FeeMath.sol::159 => if (delinquencyFeeBips > 0) - [Link for reference](https://github.com/code-423n4/2023-10-wildcat/blob/18bda5daf6a668cfe5dd6c0749a908e193e7b90c/src/libraries/FeeMath.sol#L159)
src/libraries/MarketState.sol::133 => // Equivalent to expiry > 0 && expiry <= block.timestamp :- [Link for reference](https://github.com/code-423n4/2023-10-wildcat/blob/18bda5daf6a668cfe5dd6c0749a908e193e7b90c/src/libraries/MarketState.sol#L134)
./2023-10-wildcat/src/libraries/FIFOQueue.sol::48 => for (uint256 i = 0; i < len; i++) - [Link for reference](https://github.com/code-423n4/2023-10-wildcat/blob/18bda5daf6a668cfe5dd6c0749a908e193e7b90c/src/libraries/FIFOQueue.sol#L48)
./2023-10-wildcat/src/libraries/FIFOQueue.sol::75 => for (uint256 i = 0; i < n; i++)- [Link for reference](https://github.com/code-423n4/2023-10-wildcat/blob/18bda5daf6a668cfe5dd6c0749a908e193e7b90c/src/libraries/FIFOQueue.sol#L48)
## 
