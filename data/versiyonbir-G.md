# BYTES CONSTANT MORE EFFICIENT THAN STRING LITERAL
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L24-L24

The contract was found to be using version string constant. This can be optimized by using bytes32 constant to save gas.

Unless explicitly required, if the string is lesser than 32 bytes, it is recommended to use bytes32 constant instead of a string constant as it’ll save some gas.

# ABI ENCODE IS LESS EFFICIENT THAN ABI ENCODEPACKED
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L78-L78
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L110-L110

The contract is using abi.encode() in the function createEscrow. In abi.encode(), all elementary types are padded to 32 bytes and dynamic arrays include their length, whereas abi.encodePacked() will only use the minimal required memory to encode the data.

Unless explicitly needed , it is recommended to use abi.encodePacked() instead of abi.encode().

# CONSTANT STATE VARIABLES
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L11-L11
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L12-L12
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L13-L13
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L14-L14
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L63-L63
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L70-L70
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L70-L70
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L71-L71

The contract has defined state variables whose values are never modified throughout the contract.
The variables whose values never change should be marked as constant to save gas.

Make sure that the values stored in the variables flagged above do not change throughout the contract. If this is the case, then consider setting these variables as constant.

# DEFINE CONSTRUCTOR AS PAYABLE
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L55-L57
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/ReentrancyGuard.sol#L49-L52
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L72-L104
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L76-L125
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L94-L112
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L16-L19
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L24-L28

Developers can save around 10 opcodes and some gas if the constructors are defined as payable.
However, it should be noted that it comes with risks because payable constructors can accept ETH during deployment.

It is suggested to mark the constructors as payable to save some gas. Make sure it does not lead to any adverse effects in case an upgrade pattern is involved.

# GAS OPTIMIZATION IN INCREMENTS
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93-L93
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L136-L136
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L179-L179
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L222-L222
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48-L48
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L75-L75
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L146-L146
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133-L133
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L154-L154
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L183-L183
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L212-L212

++i costs less gas compared to i++ or i += 1 for unsigned integers. In i++, the compiler has to create a temporary variable to store the initial value. This is not the case with ++i in which the value is directly incremented and returned, thus, making it a cheaper alternative.

Consider changing the post-increments (i++) to pre-increments (++i) as long as the value is not used in any calculations or inside returns. Make sure that the logic of the code is not changed.

# ARRAY LENGTH CACHING
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L154-L159
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L170-L175
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L183-L189

During each iteration of the loop, reading the length of the array uses more gas than is necessary. In the most favorable scenario, in which the length is read from a memory variable, storing the array length in the stack can save about 3 gas per iteration. In the least favorable scenario, in which external calls are made during each iteration, the amount of gas wasted can be significant.

Consider storing the array length of the variable before the loop and use the stored length instead of fetching it in each iteration.

# PUBLIC CONSTANTS CAN BE PRIVATE
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L24-L24
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L11-L12

Public constant variables cost more gas because the EVM automatically creates getter functions for them and adds entries to the method ID table. The values can be read from the source code instead.

If reading the values for the constants are not necessary, consider changing the public visibility to private.

# CHEAPER INEQUALITIES IN IF()
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L67-L67
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L155-L155
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L159-L159
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L72-L72
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L125-L125
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L147-L147
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L152-L152
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L80-L80
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L81-L81
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L82-L82
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L83-L83
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L84-L84
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L85-L85
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L86-L86
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L87-L87
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L205-L205
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L79-L79
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L82-L82
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L85-L85
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L88-L88
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L91-L91
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L210-L210
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L322-L322
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L337-L337
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L472-L472
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L483-L483
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L474-L474
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L495-L495
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L137-L137
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L152-L152
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L172-L172
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L180-L180
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L186-L186
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L32-L32
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L49-L49
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L112-L112
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L141-L141

The contract was found to be doing comparisons using inequalities inside the if statement.
When inside the if statements, non-strict inequalities (>=, <=) are usually cheaper than the strict equalities (>, <).

It is recommended to go through the code logic, and, if possible, modify the strict inequalities with the non-strict ones to save ~3 gas as long as the logic of the code is not affected.

# CHEAPER CONDITIONAL OPERATORS
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L67-L67
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L155-L155
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L159-L159
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L147-L147
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L205-L205
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L79-L79
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L170-L170
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L428-L428
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L472-L472
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L32-L32
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L112-L112

During compilation, x != 0 is cheaper than x > 0 for unsigned integers in solidity inside conditional statements.

Consider using x != 0 in place of x > 0 in uint wherever possible.
# UNNECESSARY CHECKED ARITHMETIC IN LOOP

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93-L93
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L136-L136
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L179-L179
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L222-L222
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48-L48
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L75-L75
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L146-L146
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133-L133
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L154-L154
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L183-L183
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L212-L212

Increments inside a loop could never overflow due to the fact that the transaction will run out of gas before the variable reaches its limits. Therefore, it makes no sense to have checked arithmetic in such a place.

It is recommended to have the increment value inside the unchecked block to save some gas.

# SUPERFLUOUS EVENT FIELDS
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L160-L160

block.timestamp and block.number are by default added to event information. Adding them manually costs extra gas.

block.timestamp and block.number do not need to be added manually. Consider removing them from the emitted events.

# STORAGE VARIABLE CACHING IN MEMORY
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L13-L13
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L12-L12
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L14-L14
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L11-L11
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L63-L63
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L61-L61
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L31-L31
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L244-L244
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L68-L68
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L36-L36
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L39-L39
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L42-L42
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L70-L70
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L43-L43
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L70-L70
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L71-L71
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L74-L74
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L41-L41
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L45-L45
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L76-L76
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L11-L11
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L13-L13
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L14-L14

SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each)

Storage variables read multiple times inside a function should instead be cached in the memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.
