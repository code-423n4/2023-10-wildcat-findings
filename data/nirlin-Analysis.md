##  <img src="https://user-images.githubusercontent.com/68193826/278458401-f97e2225-8cea-49c3-8423-8ad24e580144.jpg" loading="lazy" width="30" alt="" class="image-9"> Description and Overview
Wildcat introduces a unique model of lending and borrowing that we have not seen in DEFI ever before. Instead of giving all the control to the protocol, protocol gives all the control to borrowers to create a market with every possible parameter to be modifiable in accordance to the needs of borrowers and lenders. 

As the official whitepaper states:

```
Wildcat is primarily aimed at organisations such as market makers wishing to borrow funds in the medium-term, but can reasonably be extended
to parties such as DeFi protocols wishing to raise funds without the consequences of selling a native-token filled treasury into the ground to do so.
As a protocol that is - in its current form - reliant upon counterparty
selection by borrowers, Wildcat is fundamentally permissioned in nature.
Given that the positions that Wildcat vaults allow a lender to enter are
undercollateralised by design, it is not suitable for usage in many cases,
dependent on risk appetite.

```
##  <img src="https://user-images.githubusercontent.com/68193826/278458401-f97e2225-8cea-49c3-8423-8ad24e580144.jpg" loading="lazy" width="30" alt="" class="image-9"> Approach and Scope

For this audit I followed the top down approach of diving into code on the following scope: 
### **Scope**

- 1. **WildcatMarket.sol**:

  - This is the top level contract and entry point for the deposit into the markets and also to collect fees. It interact with all other below mentioned contract and libraries to give a relatively easy to understand entry point, abstracting away the complexities it using under the hood.

- 2. **WildcatMarketConfig.sol**:

  - this contract allows the borrower to set the important parameters such as authorization, totalsupply, annualInterestBips and reserveRatioBips. And also let unblock accounts(lender) and block accounts using the nuke function.

- 3. **WildcatToken.sol**:

  - This contract implements the erc20 token but the added functionality of scaling the balances.

- 4. **WildcatMarketWithdrawal.sol**:

  - this contract deals with the whole withdrawing process, queuing, executing and processing the withdrawal and view functions to get the current withdrawal data.

- 5. **WildcatMarketBase.sol**:
  - this functions serves as a whole base for the market with functions that update the state of the market, view functions to access each state, and internal functions to process the withdrawal.
- 6. **WildcatArchController.sol**:

  - This contract is a top level contract that stores all the data for all the controllers, borrowers and factories.

- 7. **WildcatControllerFactory.sol**:

  - This contract deploys all the controllers and have all the wildcat deployed set parameters with which all the controllers and markets are deployed, including max and min values for interest bips, reserve ratio bips, deliquincy fee bips, withdrawal batch duration, and grace periods. This contract also lets deploy market along with controller.



- 8. **WildcatMarketController.sol**:

  - this function is used to deploy the market, set interest rates, reserve ratios and authorize and deauthorize the lenders.

- 9. **WildcatSanctionsEscrow.sol**:
  - this contract is used as escrow when a lender is blocked and is used to refund the lender funds back to him if possible.

 -10.**WildcatSanctionsSentinal.sol**
  -  This contract deploys the escrow contract, and lets override the oracle blocking the lenders to prevent any oracle problem.
  
  
 ## Architecture of code base
 
 First have a look at the overall architecture of market how market is formed from these multiple contracts
 
 
![image](https://user-images.githubusercontent.com/68193826/278450785-24d850f4-3424-4d68-9865-c42d34278bbc.png)

MarketBase inherits from three main contracts, that each bring in their functionalities as discussed above, using different libraries and their own custom functions.

Than the `wildcatMarket.sol` inherits from all these contracts 

```solidity
contract WildcatMarket is
  WildcatMarketBase,
  WildcatMarketConfig,
  WildcatMarketToken,
  WildcatMarketWithdrawals
  ```
  
  
![image](https://user-images.githubusercontent.com/68193826/278451676-b81d76cc-e052-4bb6-9e76-c3f10a428a25.png)


The interaction between the controller factory, arch controller, sentinal, escrow deployment, controller and market deployment is as per following diagram:

![image](https://user-images.githubusercontent.com/68193826/278452144-db514576-3a38-4183-b2a8-123ed73da868.png)

##  <img src="https://user-images.githubusercontent.com/68193826/278458401-f97e2225-8cea-49c3-8423-8ad24e580144.jpg" loading="lazy" width="30" alt="" class="image-9"> Codebase Quality
I would say the codebase quality is good but can be improved, there are checks in place to handle different roles, each standard is ensured to be followed. And if something is not fully being followed that have been informed. But still it can be improved in following areas


| Codebase Quality Categories  | Comments |
| --- | --- |
| **Unit Testing**  | Code base is well-tested but there is still place to add fuzz testing and invariant testing and foundry have now solid support for those, I would say best in class.|
| **Code Comments**  | Overall comments in code base were not enough, more commenting can be done to more specifically describe specifically mathematical calculation. At many point if there were more detail commenting and natspec it would have been bit easy for the auditor and less question would have been asked from sponser, saving time for both. Specifically perpetual vault needs more comments as its flow is hard to understand |
| **Documentation** | Documentation was to the point but not detailed enough, such novelty requires more documentation thanthat, gitbook content can be further improved.|
| **Organization** | Code organisation was great, best in class, managing each part separately, specifically dividing the market functionalities into separate easy to understand contracts and than using inheritance to use them together.. |

##  <img src="https://user-images.githubusercontent.com/68193826/278458401-f97e2225-8cea-49c3-8423-8ad24e580144.jpg" loading="lazy" width="30" alt="" class="image-9"> Centralisation Risks 

Protocol is highly dependent on the well behaving of the borrower as borrower can run away any time. But wild cat reduces that risks by doing the KYC on its own end before adding the borrower into the market and letting him open the vaults. So it seems covered and safe in this case and there seems to be no concentration risk.

Another case is the deployer may go rouge and add the malicious borrowers into the wild cat system and set the wrong parameters, but it is not clear will either the deployer be multisig or not. This concern can be mitigated too by using the multisig.

##  <img src="https://user-images.githubusercontent.com/68193826/278458401-f97e2225-8cea-49c3-8423-8ad24e580144.jpg" loading="lazy" width="30" alt="" class="image-9"> Dependecies Used
If we look at the the package.json for the project, we can see that the not the latest version of openzeppelin is used:

'''
  "dependencies": {
    "@openzeppelin/contracts": "^4.9.3",
    "@types/node": "^20.4.2",
    "cli-barchart": "^0.2.3",
    "ethers": "^6.6.3"
  }
'''
  
  Project is using the version ^4.9.3 while the latest recent major upgrade is 0.5.0 which have many improvements and optimisation which makes it more efficient to use.
 
 Some of the main upgrades are:
1. Namespaced Storage for better upgrades security
2. Leveraging new Solidity compiler additions
3. Tooling updates
4. Gas-efficiency without compromising Security
5. Replacing revert strings with custom errors
6. Removing duplicated SLOADs and adding immutable variables
7. Packing variables in storage
8. Flexible and Transparent Access Management
9. Redefining Access Control through AccessManager
10. Best-in-class Audit & Testing Practices

So instead of using the old version, the project is using ^4.3.1. Replace the old version with newer one to get advantage of all these new updates.

##  <img src="https://user-images.githubusercontent.com/68193826/278458401-f97e2225-8cea-49c3-8423-8ad24e580144.jpg" loading="lazy" width="30" alt="" class="image-9"> Conclusion
Wildcat brings a novel solution to the existing problem that we have not seen before. Overall the code quality and mission statement is great. Had some hard time understanding the bip and ray mathematics, more documentations would have helped but the solution is gonna serve well to most of the people.




### Time spent:
27 hours