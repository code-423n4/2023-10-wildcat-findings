# Low
## [L-01] Lenders can also deposit when not authorized on the controller
[WildcatMarketController Line 169](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L169)

**Issue Description:**
The contest description incorrectly states that "Lenders that are authorized on a given controller (i.e. granted a role) can deposit assets to any markets that have been launched through it.". However, this is not the case as borrowers need to call `updateLenderAuthorization()` when deauthorizing a lender. If a borrower forgets to call this function, the lender can be deauthorized on the controller but still deposit new funds into the market until the lender or someone else calls `updateLenderAuthorization`.

**Recommended Mitigation Steps:**
It is recommended to update the documentation to state that this is only correct if `updateLenderAuthorization()` was called afterward. If the intent is to have the functionality work as described in the contest description, an atomic removal of lenders would need to be implemented.

---
## [L-02] No controllers can be deployed if certain tokens are chosen as `feeAsset`
[WildcatMarketController Line 345](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L345)

**Issue Description:**
The `MarketControllerFactory` allows for setting an `originationFeeAsset` as well as `originationFeeAmount`, which are used to send a fee to the recipient each time a new market is deployed. When a market is deployed, the `originationFeeAmount` of the `originationFeeAsset` is transferred from the borrower to the `feeRecipient`. There is one additional check in place that verifies if the `originationFeeAsset` address is 0 and only transfers a fee if it is not zero.

```solidity
if (originationFeeAsset != address(0)) {
	originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);
}
```

The issue with this implementation is that some tokens, like [LEND](https://www.coingecko.com/de/munze/aave-old), may revert in case of a zero transfer. This means that if a token like [LEND](https://www.coingecko.com/de/munze/aave-old) is set as the `originationFeeAsset`, and later on the fee is reduced to zero, this function will always fail to execute, preventing any new markets from being deployed.

Additionally, not checking for zero transfers could lead to gas waste in the case of a token that does not revert but simply transfers nothing during a zero transfer.

**Recommended Mitigation Steps:**
To fix this issue, an additional check needs to be added to the if clause, ensuring that `originationFeeAmount` is greater than zero:

```solidity
if (originationFeeAsset != address(0) && originationFeeAmount > 0) {
	originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);
}
```

---
## [L-03]  `getDeployedControllers()` will not return the last index
[WildcatMarketControllerFactory Line 138](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L138)

**Issue Description:**
The `getDeployedControllers()` function takes a start and end index of the controllers to be retrieved. However, due to the current implementation of the function, it only returns controllers from the start index to `end-1`. In other words, the controller at the `end` index is not included in the returned list. 

**POC:**
This simple POC can be added to the WIldcatMarketController test to check for the issue.

```solidity
function test_getControllers() external {
//Deploy 10 contracts
	for(uint256 i = 0; i < 10; i++)
	{
		controllerFactory.deployController();
	}
	  
	//Want to access 2-7 (6 controllers)
	address[] memory controllers = controllerFactory.getDeployedControllers(2, 7);
	
	//Check that the length is correct
	require(controllers.length == 5, "We have not received an incorrect number");
	
	//We only received controllers 2-6 due to the implementation
}
```

The same issue also exists in the functions `WildcatMarketController.getAuthorizedLenders()`, `WildcatMarketController.getControlledMarkets()` , `WildcatArchController.getRegisteredBorrowers()`, 
`WildcatArchController.getRegisteredControllerFactories()`, 
`WildcatArchController.getRegisteredControllers()` and
`WildcatArchController.getRegisteredMarkets()`.

**Recommended Mitigation Steps:**
To resolve this issue, the `getDeployedControllers()` function should be rewritten as follows:

```solidity
function getDeployedControllers(
uint256 start,
uint256 end
) external view returns (address[] memory arr) {
	uint256 len = _deployedControllers.length();
	end = MathUtils.min(end, len-1);
	uint256 count = end - start + 1;
	arr = new address[](count);
	for (uint256 i = 0; i < count; i++) {
		arr[i] = _deployedControllers.at(start + i);
	}
}
```

The same change should be applied to other affected functions as well.

---
## [L-04]  Rebasing tokens will lead to borrowers needing to pay a lower APR

**Issue Description:**
Rebasing tokens, which are not excluded from the contest, can be used as underlying assets for markets deployed using the protocol. Rebasing tokens can be implemented in various ways, but the critical point is when the balance of addresses holding the tokens gradually increases. As borrowers/market contracts hold these tokens while they are lent, the newly accrued tokens may either be credited to the borrower, or inside the market itself, which would count as the borrower adding liquidity. This can result in the borrower needing to pay a lower Annual Percentage Rate (APR) than initially set.

**Recommended Mitigation Steps:**
This issue can be mitigated in several ways:

- **Option 1:** Disallow rebasing tokens from the protocol to prevent this situation.
- **Option 2:** Add a warning to the documentation, informing users that when lending rebasing tokens, the rebasing interest their tokens gain while inside the market will be counted as the borrower paying down their debt.
- **Option 3 (Complicated):** Implement functionality for rebasing tokens by checking the market's balance at each interaction and adding the change to a separate variable that tracks rebasing awards.

---
## [L-05]  Reserve ratio can be set to 100%
[MarketControllerFactory Line 85](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L85)

**Issue Description:**
The protocol allows borrowers to set a reserve ratio that they must maintain to avoid being charged a delinquency fee. In the current implementation, this parameter can be set to 100%, rendering the entire functionality redundant, as borrowers would not be able to withdraw any funds from the market.

**Recommended Mitigation Steps:**
To mitigate this issue, modify the check on `maximumReserveRatioBips` to revert if `constraints.maximumReserveRatioBips >= 10000`.

---
## [L-06]  `scaleFactor` can theoretically overflow
[FeeMath Line 169](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L169)

**Issue Description:**
The `scaleFactor` of a market is multiplied by the fee rate to increase the scale. In a very rare edge case, where a market has a 100% interest (e.g., a junk bond) and is renewed each year with the borrower paying lenders the full interest, the scale factor would overflow after 256 years (as the scale factor doubles every year) when it attempts to increase during the withdrawal amount calculation.

**Recommended Mitigation Steps:**
While this issue is unlikely to occur in practice, a check should be added in the withdrawal process to prevent an overflow. If an overflow is detected, lenders should be forced to withdraw with a scale factor of `uint256.max`, and the borrower should close the market to enable borrowing again.

---
## [L-07]  Misleading ERC20 queries `balanceOf()` and `totalSupply()`
[WildcatMarketToken Line 16](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L16)
[WildcatMarketToken Line 22](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L22)

**Issue Description:**
The `WildcatMarketToken` contract includes standard `ERC20` functions, `balanceOf()` and `totalSupply()`. However, these functions return the balance of the underlying tokens instead of the market tokens. This discrepancy between the function names and their actual behavior could lead to confusion or issues when interacting with other protocols.

**Recommended Mitigation Steps:**
To address this issue, it is recommended to rename the existing functions to `balanceOfScaled()` and `totalScaledSupply()`, and additionally implement `balanceOf()` and `totalSupply()` functions that return the balance of the market token.

---
## [L-08] Closed markets can't be reopened
[WildcatMarket Line 142](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L142)

**Issue Description:**
Markets include a functionality where users can close markets directly, effectively transferring all funds back into the market and setting the `isClosed` parameter of the state to true. While this prevents new lenders from depositing into the market, it only allows lenders to withdraw their funds and interest. The issue is that, once a borrower uses this function, the market cannot be reopened. If the borrower wants to have another market for the same asset, they must deploy a new market with a new prefix to avoid salt collisions. If a lender does this often it might end up in the Market names looking like "CodearenaV1234.56DAI" due to new prefixes being needed each time. Additionally the markets list would get more and more bloated. 

**Recommended Mitigation Steps:**
To mitigate this issue, borrowers should be allowed to reset a market. This would require all lenders to withdraw their funds before the reset, but it would reset all parameters, including the scale factor, allowing the market to be restarted.

---
## [L-09] Choosable prefix allows borrowers to mimic other borrowers.
[WildcatMarketBase Line 97](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L97)

**Issue Description:**
When a new market is deployed, its name and prefix are generated by appending the underlying asset's name and symbol to the name and symbol prefixes provided by the borrower. In the contest description this is explained as Code4rena deploying a market and passing Code4rena as name prefix and C4 as symbol prefix.

A malicious borrower could exploit this functionality to deploy markets for other well-known assets with the same Code4rena prefixes, potentially tricking lenders into lending them money and mimicking other borrowers. This could lead to confusion and potentially fraudulent activities.

**Recommended Mitigation Steps:**
The recommended solution for this is to let each borrower choose a borrower identifier (that gets issued to them by wildcat). This in the Code4rena example would be the "Code4rena" and "C4" strings. Now the hashes (hashes instead of strings to save gas on storage cost) of those identifiers get stored onchain whenever a new borrower gets added. Whenever Code4rena deploys a new market they pass their identifier as well as a chosen Postfix which would allow them to deploy multiple markets for the same asset (for example with different interest rates). The protocol then verifies if the identifiers match the hash and revert if that is not the case. The market mame would then for example look like this "Code4renaShortTermDAI".

# Non-Critical
## [NC-01]  Badly named constant `BIP`
[MathUtils Line 6](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L6)

**Issue Description:**
The variable `BIP` is used to represent the maximum BIP in the protocol, where at most different rates can be set to 10,000, equivalent to 100%. The variable name could be misleading, as it may incorrectly suggest that it represents one BIP, which should be equal to 1.

**Recommended Mitigation Steps:**
To enhance clarity, rename the variable to `MAX_BIP`.

---
## [NC-02]  Incorrect documentation on capacity resizing

**Issue Description:**
The [whitepaper]((https://github.com/wildcat-finance/wildcat-whitepaper/blob/main/whitepaper_v0.2.pdf)) (on page 5) states that the "maximum capacity of a vault can be adjusted at will up or down by the borrower depending on the current need." However, the `maxCapacity` can only be reduced down to the current liquidity inside the market, as per the code. This discrepancy between the documentation and the code could lead to misunderstandings.

```solidity
if (_maxTotalSupply < state.totalSupply()) {
	revert NewMaxSupplyTooLow();
}
```

**Recommended Mitigation Steps:**
Revise the whitepaper to accurately reflect that the "maximum capacity of a vault can be adjusted at will, but only down to the current supply of the market."

---
## [NC-03]  Incorrect documentation on authentication process
[Whitepaper, page 7](https://github.com/wildcat-finance/wildcat-whitepaper/blob/main/whitepaper_v0.2.pdf)

**Issue Description:**
The whitepaper states that "Vaults query a specified controller to determine who can interact." However, the interaction rights are set by the controller on the market. There is no callback from the market to the controller, except for the initial call of the lender when authorization is null. The whitepaper should be updated to align with the actual code implementation.

**Recommended Mitigation Steps:**
Update the documentation to describe the functionality as it exists in the code.

---
## [NC-04]  Incorrect documentation of `registerControllerFactory()`
[WildcatArchController Line 106](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L106)

**Issue Description:**
The documentation in the [GitBook](https://wildcat-protocol.gitbook.io/wildcat/technical-deep-dive/component-overview/wildcatarchcontroller.sol) states that the function `registerControllerFactory()` only reverts if "The controller factory has already been registered." This is inaccurate because the function also uses the `onlyOwner()` modifier, which causes it to revert if called by someone other than the owner.

**Recommended Mitigation Steps:**
Revise the documentation to include the statement "Called by someone other than the owner."

---
## [NC-05]  Incorrect documentation of `removeControllerFactory()`
[WildcatArchController Line 113](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L113)

**Issue Description:**
The documentation in the GitBook](https://wildcat-protocol.gitbook.io/wildcat/technical-deep-dive/component-overview/wildcatarchcontroller.sol) states that the function `removeControllerFactory()` only reverts if "The controller factory address does not exist or has been removed." This is not entirely accurate, as the function also employs the `onlyOwner()` modifier, causing it to revert if called by someone other than the owner.

**Recommended Mitigation Steps:**
Update the documentation to include the statement "Called by someone other than the owner."

---
## [NC-06]  Documentation of functions is missing

**Issue Description:**
The GitBook](https://wildcat-protocol.gitbook.io/wildcat/technical-deep-dive/component-overview/wildcatarchcontroller.sol) documentation lists numerous functions in the component overview section that lack descriptions. This omission can make it challenging for users and developers to understand the purpose and usage of these functions.

**Recommended Mitigation Steps:**
Provide descriptions for the missing functions in the documentation to enhance clarity and understanding.

---
## [NC-07]  Incorrect comment in `_depositBorrowWithdraw()`
[BaseMarketTest Line 71](https://github.com/code-423n4/2023-10-wildcat/blob/main/test/BaseMarketTest.sol#L71)

**Issue Description:**
Inside the function the comments state that 80% of the market assets get borrowed and 100% of withdrawal get withdrawn. This is not the case as instead the provided parameters `depositAmount`, `borrowAmount` and `withdrawalAmount` are used. There are no require statements in the function that check for these requirements 80%/100% being fulfilled by the parameters.

**Recommended Mitigation Steps:**
To address this issue, consider either adding require statements to verify that the parameters adhere to the stated percentages, or remove the comments if they no longer apply to the function's functionality.