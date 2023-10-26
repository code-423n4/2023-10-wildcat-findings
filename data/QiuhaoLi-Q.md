## [low] deployInitCode can't handle data bigger than 64k

In src/libraries/LibStoredInitCode.sol:`deployInitCode`, we store the creation code size as:

```solidity
    assembly {
      let size := mload(data)
      let createSize := add(size, 0x0b)
      // Shift (size + 1) to position it in front of the PUSH2 instruction.
      // Reuse `data.length` memory for the create prefix to avoid
      // unnecessary memory allocation.
      mstore(data, or(shl(64, add(size, 1)), 0x6100005f81600a5f39f300))
```

Here, if `data.length` >= 0xffff, the `mstore` operation will store the size at `0x61`, which will break the contract to be deployed. For example, in the PoC follow, we deploy a contract with `data.length` == 0xffff, so the overflowed size would be zero:

```solidity
  function test_deployInitCode_large() external {
    bytes memory data = new bytes(65535);
    address deployed = lib.deployInitCode(data);
    assertEq(deployed.codehash, keccak256(abi.encodePacked(uint8(0x00), data)));
  }
```

Output:

```solidity
Running 1 test for test/market/WildcatMarketConfig.t.sol:WildcatMarketConfigTest
[PASS] test_sanction_transferFrom() (gas: 883736)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 3.78ms
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```
As we can see, the deployed contract is empty.

To fix this issue, we should add a size check:
```solidity
diff --git a/src/libraries/LibStoredInitCode.sol b/src/libraries/LibStoredInitCode.sol
index 55de81e..ca8854b 100644
--- a/src/libraries/LibStoredInitCode.sol
+++ b/src/libraries/LibStoredInitCode.sol
@@ -5,6 +5,7 @@ library LibStoredInitCode {
   error InitCodeDeploymentFailed();
 
   function deployInitCode(bytes memory data) internal returns (address initCodeStorage) {
+    require(data.length < 0xffff);
     assembly {
       let size := mload(data)
       let createSize := add(size, 0x0b)
```


## [non-critical] We can't register a market back after it was removed

`src/WildcatArchController.sol:registerMarket` is only callable by the market controller:

```solidity
  function registerMarket(address market) external onlyController {
    if (!_markets.add(market)) {
      revert MarketAlreadyExists();
    }
    emit MarketAdded(msg.sender, market);
  }
```

And in `src/WildcatMarketController.sol`, the only place to call `registerMarket` is in `deployMarket`:

```solidity
...
    bytes32 salt = _deriveSalt(asset, namePrefix, symbolPrefix);
    market = LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt, marketInitCodeHash);
    if (market.codehash != bytes32(0)) {   <------
      revert MarketAlreadyDeployed();
    }
    LibStoredInitCode.create2WithStoredInitCode(marketInitCodeStorage, salt);

    archController.registerMarket(market);
...
```

As we can see, if the market was deployed once, we can't deploy it again.

This will lead to a problem if we want to register a market after it has been removed from the arch controller. To fix this, we should add a function in the controller to register a address as market directly.


## [non-critical] redundant code in src/libraries/FeeMath.sol:calculateLinearInterestFromBips

The method `src/libraries/FeeMath.sol:calculateLinearInterestFromBips` is the same as `src/libraries/MathUtils.sol:calculateLinearInterestFromBips`. Should use only one.

## A blocked/sanctioned account can still received interest

In `_blockAccount`, we only transfer the `scaledBlance` of the blocked account to the escrow contract, so the escrow will still get interests, which may lead to law issues.

```solidity
      if (scaledBalance > 0) {
        account.scaledBalance = 0;
        address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
          accountAddress,
          borrower,
          address(this)
        );
        emit Transfer(accountAddress, escrow, state.normalizeAmount(scaledBalance));
        _accounts[escrow].scaledBalance += scaledBalance; <-----
```

To fix this, we can also trigger a withdrawal for the escrow contract.

## [non-critical] state.totalSupply() can exceed state.maxTotalSupply

state.maxTotalSupply is a static value, but totalSupply() is a normalized amount that will increase with interest in the market. So it's possible that state.totalSupply() can exceed state.maxTotalSupply, which may break the invariants.

```solidity
  function totalSupply(MarketState memory state) internal pure returns (uint256) {
    return state.normalizeAmount(state.scaledTotalSupply);
  }
```