## L1 - Market could be closed multiple times

`WildcatMarket#closeMarket` function does not check if market is already closed:
```solidity
File: WildcatMarket.sol
142:   function closeMarket() external onlyController nonReentrant { 
143:     MarketState memory state = _getUpdatedState();
144:     state.annualInterestBips = 0;
145:     state.isClosed = true; 
146:     state.reserveRatioBips = 0;
147:     if (_withdrawalData.unpaidBatches.length() > 0) {
148:       revert CloseMarketWithUnpaidWithdrawals();
149:     }
150:     uint256 currentlyHeld = totalAssets();
151:     uint256 totalDebts = state.totalDebts();
152:     if (currentlyHeld < totalDebts) {
153:       // Transfer remaining debts from borrower
154:       asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);
155:     } else if (currentlyHeld > totalDebts) { 
156:       // Transfer excess assets to borrower
157:       asset.safeTransfer(borrower, currentlyHeld - totalDebts);
158:     }
159:     _writeState(state);
160:     emit MarketClosed(block.timestamp);
161:   }
```
This could lead to a situation when the same market is closed more than 1 time, and an appropriate event is emitted, which would mislead external listeners.
Recommendation: Consider adding a check that the market is not closed yet in the `closeMarket` function.

## L2 - If admin removes the market from `WildcatArchController` - `nukeFromOrbit` and `executeWithdrawal` functions would be DOS for sanctioned lenders
`createEscrow` function that creates Escrow contracts for sanctioned lenders includes check that `msg.sender` is actually registered market:
```solidity
File: WildcatSanctionsSentinel.sol
095:   function createEscrow(
096:     address borrower,
097:     address account, 
098:     address asset
099:   ) public override returns (address escrowContract) {
100:     if (!IWildcatArchController(archController).isRegisteredMarket(msg.sender)) { 
101:       revert NotRegisteredMarket();
102:     }
...
```
At the same time `WildcatArchController` allows protocol admin to un-register any market using `removeMarket` function. This could lead to a situation when the market would be removed from `WildcatArchController` and sanctioned lenders would be unable to receive their funds due to DOS in `nukeFromOrbit` and `executeWithdrawal` functions that call the `createEscrow` function inside.
Recommendation: Consider adding a separate list of `removedMarkets` in `WildcatArchController` and check inside the `createEscrow` function if `msg.sender` is active or removed market.

## L3 - Protocol admin could accidentally front-run borrower with the fee increasing 
`WildcatMarketController#deployMarket` allows the borrower to deploy a new market and pay a fee to the protocol if it exists. This fee could be changed at any moment by admin.

Admin can accidentally front-run borrowers `deployMarket` call and set fee to bigger value, which borrower wasn't expected.

Recommendation: Consider adding a timelock to change fee parameters. This way, frontrunning will be impossible, and borrowers will know which fee they agree to.

## N1 - Redundant modifier `onlyControlledMarket` in `WildcatMarketController`

Modifier `onlyControlledMarket` checks that the called market is actually a market that is deployed and controlled by the current controller:
```solidity 
File: WildcatMarketController.sol
87:   modifier onlyControlledMarket(address market) { 
88:     if (!_controlledMarkets.contains(market)) {
89:       revert NotControlledMarket();
90:     }
91:     _;
92:   }
``` 
But this check is redundant since the market itself checks that the caller is a correct controller:
```solidity
File: WildcatMarketConfig.sol
171:   function setReserveRatioBips(uint16 _reserveRatioBips) public onlyController nonReentrant {
...
```
Recommendation: Consider removing not needed modifier.

## N2 - Redundant line in `WildcatSanctionsSentinel#createEscrow`

Line 114 in `createEscrow` change `sanctionOverrides` storage value for the newly created escrow address:
```solidity
File: WildcatSanctionsSentinel.sol
095:   function createEscrow(
...
110:     new WildcatSanctionsEscrow{ salt: keccak256(abi.encode(borrower, account, asset)) }(); 
111: 
112:     emit NewSanctionsEscrow(borrower, account, asset);
113: 
114:     sanctionOverrides[borrower][escrowContract] = true; 
119:   }
```

However, this value is not actually used anywhere in the codebase, since `isSanctioned` function actually checks `account` (lender address) to be sanctioned.

Recommendation: Consider removing not needed line 114.

