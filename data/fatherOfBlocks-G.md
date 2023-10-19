**src/WildcatMarketControllerFactory.sol**
- L5 - WildcatStructsAndEnums is imported, but it is never used, therefore it should be removed.

- L17 - The UpdateProtocolFeeConfiguration event is created, but it is never used, therefore it should be removed.


**src/WildcatSanctionsEscrow.sol**
- L5 - IChainalysisSanctionsList is imported, but it is never used, therefore it should be removed.


**src/libraries/Errors.sol**
- L4/5/7/8/9/10/12/13 - Multiple events Panic_CompilerPanic, Panic_AssertFalse, Panic_DivideByZero, Panic_InvalidEnumValue,Panic_InvalidStorageByteArray, Panic_EmptyArrayPop, Panic_MemoryTooLarge and Panic_UninitializedFunctionPointer are created, but they are never used, so They should therefore be eliminated.


**src/libraries/StringQuery.sol**
- L12/13/17 - The constants SixtyThreeBytes and ThirtyOneBytes are created but are never used. Same as the InvalidCompactString error, therefore they should be eliminated.


**src/market/WildcatMarket.sol**
- L4 - The FeeMath library is imported, but it is never used, therefore this generates unnecessary extra gas expenditure.


**src/market/WildcatMarketConfig.sol**
- L5 - The FeeMath library is imported, but it is never used, therefore this generates unnecessary extra gas expenditure.

