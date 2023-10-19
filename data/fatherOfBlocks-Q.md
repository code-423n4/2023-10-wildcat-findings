**src/WildcatSanctionsSentinel.sol**
- L74 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**src/WildcatSanctionsEscrow.sol**
- L18 - Before setting the value of borrower, account and asset, it should be validated that the addresses are different from 0x, since otherwise they would generate DoS in multiple functions.

- L29/33 - The escrowedAsset and releaseEscrow functions are public but are not used in the contract, therefore their visibility should be external.


**src/market/WildcatMarketBase.sol**
- L320 - There is commented code that reduces the understanding of the code, therefore it should be eliminated, since it does not contribute anything.


**src/WildcatMarketController.sol**
- L255/370/394 - According to the Solidity style guide, functions should be laid out in the following order :constructor(), receive(), fallback(), external, public, internal, private, but the cases below do not follow this pattern

- L378/405 - There is commented code that reduces the understanding of the code, therefore it should be eliminated, since it does not contribute anything.
