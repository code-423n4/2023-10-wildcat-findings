 
 ![WildCat](https://user-images.githubusercontent.com/58845085/278267516-05c5b04a-734a-4595-b625-17e32f2272f0.png) 

# The Wildcat Protocol Analysis

## Description overview of Wildcat Protocol

The protocol operates through a central contract (archcontroller) that controls which factories and markets are allowed. Borrowers deploy markets through controllers, and authorized lenders can deposit assets in exchange for market-specific tokens.

Here's a concise explanation of its key things:

``Credit Escrow Markets`` : Wildcat allows borrowers to create customized lending markets for specific assets, interest rates, and chosen lenders, inverting the traditional model where borrowers appeal to existing pools of lenders.

``Collateralization``: Borrowers do not need to deposit collateral upfront but are required to maintain a reserve ratio within the market. Lenders effectively collateralize their own loans, which serves as a buffer for withdrawal requests.

``No Underwriters or Insurance``: The protocol does not rely on underwriters or insurance funds. It's a hands-off system with no ability to freeze borrower collateral or pause activity.

``No Proxies``: Wildcat avoids using proxies for added security. If issues occur, the protocol cannot assist users.

``Sanctions Monitoring``: The protocol monitors addresses flagged by Chainalysis and prevents them from interacting with markets to prevent disruptions.

``Interest Rates``: Interest rates paid by borrowers consist of base APR, protocol APR, and penalty APR. Penalty APR applies when a market falls below the reserve ratio for an extended period.

``Withdrawals``: Borrowers can withdraw assets as long as the reserve ratio is maintained. Withdrawals are initiated by authorized addresses, and market tokens are burned to create a claimable withdrawals pool.

``Sanctioned Addresses`` : Addresses flagged as sanctioned are blocked, and a mechanism exists to handle their assets.

## Summery 

## Approach followed when reviewing the code

To begin, assessed the code's scope, which guided our approach to reviewing and analyzing the project.

### Scope

- ``WildcatMarketController`` : Deploys markets and manages their configurable parameters (APR, reserve ratio) and maintains a set of approved lenders

- ``WildcatMarketBase`` : This contract serves as the foundation for a lending market, handling important aspects like fees, interest rates, withdrawals, and participant roles.

- ``WildcatMarketControllerFactory`` : The contract is used to manage and deploy market controllers, customize various parameters, and control fees within a DeFi ecosystem, with permissions and constraints managed through the associated arch-controller.

- ``WildcatArchController`` - The ``WildcatArchController`` is a contract designed to provide governance and control over various components of a DeFi ecosystem, allowing for the management of borrowers, controller factories, controllers, and markets.

- ``WildcatMarketWithdrawals`` : This contract is part of a larger financial system and plays a crucial role in managing and processing withdrawals for a specific financial market. It ensures that lenders can queue withdrawals and that these withdrawals are executed correctly, taking into account various conditions, including sanctions and available liquidity


## Analysis of the code base

#### WildcatMarketController.sol

- ``Constructor``: Initializes the contract with the arch-controller and sets various constraints for market parameters.

- ``onlyArchControllerOwner``: A custom modifier to restrict access to the owner of the arch-controller.

- ``isDeployedController``: Checks if a controller is deployed for a specific address.
- ``getDeployedControllersCount``: Returns the count of deployed controllers.
- ``getDeployedControllers``: Returns an array of deployed controller addresses.
- ``getDeployedControllers(start, end)``: Returns a slice of deployed controller addresses.
- ``getProtocolFeeConfiguration``: Retrieves the current protocol fee configuration.
- ``setProtocolFeeConfiguration``: Updates the protocol fee configuration (requires ownership of the arch-controller).
- ``getParameterConstraints``: Returns the immutable constraints on market parameters.
- ``getMarketControllerParameters``: Retrieves parameters used for market controller deployment.
- ``deployController``: Deploys a new market controller for a registered borrower.
- ``deployControllerAndMarket``: Deploys a controller and a new market through that controller.
- ``computeControllerAddress``: Computes the address for a controller to be deployed for a specific borrower.

### Deploying New Market

This flow diagram shows the steps involved in the ``deployMarket()`` function, including checking if the sender is a borrower, enforcing parameter constraints, getting fee configuration, transferring origination fees, deriving a unique salt, and checking if the market is already deployed. If the market is not deployed, it proceeds with deploying the market, registering it, adding it to controlled markets, and finally resetting temporary parameters

![Deploynew](https://user-images.githubusercontent.com/58845085/278300315-749f730d-7fa1-4974-9813-ebd1756304d9.png)

#### WildcatMarketControllerFactory.sol

- Modifiers
``onlyArchControllerOwner``: A custom modifier that restricts access to certain functions to the owner of the arch-controller.
Constructor: The constructor initializes the contract and sets various constraints for market parameters.

- ``Internal Functions``

``_storeControllerInitCode`` and ``_storeMarketInitCode``:
Internal functions used to store initialization code for controllers and markets.
``_tmpMarketBorrowerParamete``r: A temporary variable used during the deployment of controllers.

- ``View Functions``
``isDeployedController``: Checks if a controller has been deployed.
``getDeployedControllersCount``: Returns the number of deployed controllers.
``getDeployedControllers``: Returns an array of deployed controller addresses.
``getDeployedControllers(uint256 start, uint256 end)``: Returns a range of deployed controller addresses.
``getProtocolFeeConfiguration``: Retrieves the protocol fee configuration.

- ``External Functions``

``setProtocolFeeConfiguration``: Allows the owner of the arch-controller to update the protocol fee configuration.
``getParameterConstraints``: Returns the immutable constraints on market parameters.
``getMarketControllerParameters``: Returns parameters for the controller.
``deployController``: Deploys a new controller for a borrower.
``deployControllerAndMarket``: Deploys a new controller and a market through that controller for a borrower.
``computeControllerAddress``: Calculates the address for a controller based on the borrower's address.


### Deploying New Controller

The ``deployController`` function is responsible for deploying a new controller contract for a registered borrower

![`TPUZ(H((Z~HU%9YRJ0AYG](https://user-images.githubusercontent.com/58845085/278313030-0f82061d-a886-4caa-84e5-610b33f37b51.png)


## Recommentations 

### Develop a liquidation mechanism that is more fair to borrowers

The current liquidation mechanism is very punitive to borrowers, and could lead to cascading failures across the Wildcat ecosystem if multiple markets are liquidated at the same time.

#### Wildcat Protocol liquidation mechanism

                                     Borrower
                                    /       \
                                   /         \
                                  /           \
                             Deposit    Withdraw
                              |              |
                              |              |
                         Market Contract     Claimable Withdrawals Pool
                              |              |
                              |              |
                             Withdraw      Liquidated
                              |              |
                              |              |
                                     Lender
                                          
If the value of the underlying asset in the market has fallen below the reserve ratio, the borrower will not be able to withdraw any assets from the claimable withdrawals pool. This means that the borrower will lose all of their collateral.

This is a very punitive liquidation mechanism, and it is one of the main systemic risks of the Wildcat Protocol. 

#### More efficient liquidation mechanism:

                                     Borrower
                                    /       \
                                   /         \
                                  /           \
                             Deposit    Withdraw
                              |              |
                              |              |
                         Market Contract     Claimable Withdrawals Pool
                              |              |
                              |              |
                              Liquidate     Repay
                              |              |
                              |              |
                                     Lender
                                     
                                     
The market contract would first attempt to liquidate the market gradually. If the borrower is able to repay their loan within a certain period of time, the market would be re-opened and the borrower would be able to continue borrowing. If the borrower is unable to repay their loan within the specified period of time, the market would be liquidated completely and the borrower would lose their collateral.

The Wildcat team could also implement a liquidation insurance mechanism to further protect borrowers. This would involve pooling funds from lenders to cover the losses of borrowers who are liquidated. This would help to reduce the risk of cascading failures across the Wildcat ecosystem.

By implementing a more efficient and fair liquidation mechanism, the Wildcat team can make the protocol more attractive to borrowers and reduce the risk of systemic failures.

### Provide more clarity around the master loan agreement and the nukeFromOrbit function
The current documentation for these features is somewhat vague. More clarity would help users to understand the risks and benefits of using them.

In addition to providing more clarity about the specific terms of these features, the documentation should also explain the risks and benefits of using them in a more general sense.

### Develop a more robust oracle solution
The Chainalysis oracle is a single point of failure. A more robust solution would involve using multiple oracles and aggregating their results.

### Implement a mechanism to protect lenders from market manipulation
Malicious borrowers could potentially manipulate the market in order to liquidate lenders or force them to accept a lower interest rate. A mechanism to protect lenders from this type of manipulation would be beneficial.


## Systemic Risks

Here's an analysis of potential systemic Risks

- ``Counterparty risk``: Wildcat markets are peer-to-peer agreements between borrowers and lenders, and there is no insurance or underwriting involved. This means that lenders are taking on a significant amount of risk, and there is no guarantee that they will be able to recover their funds if the borrower defaults.

- ``Smart contract risk`` : The Wildcat Protocol is a complex smart contract system, and there is always the risk of bugs or vulnerabilities that could be exploited by malicious actors. This could lead to the loss of funds for borrowers and lenders alike.

- ``Liquidation risk`` : If the value of the underlying asset in a Wildcat market falls below the reserve ratio, the market will be liquidated. This means that lenders will be able to withdraw their funds, but the borrower will lose all of their collateral. This could lead to cascading failures across the Wildcat ecosystem if multiple markets are liquidated at the same time.

- ``Oracle risk`` : The Wildcat Protocol relies on the Chainalysis oracle to identify sanctioned addresses. If this oracle is compromised or fails, it could allow malicious actors to interact with Wildcat markets without being detected.

- ``Centralization risk`` : The Wildcat Protocol is currently controlled by a single entity. This means that this entity has a lot of power over the protocol, and could potentially make changes that are harmful to users.

- ``Abuse of the master loan agreement``: The master loan agreement that Wildcat provides is optional, and lenders can countersign it or decline. If a lender chooses not to countersign the agreement, they may have difficulty enforcing their rights in the event of a default.

- ``Abuse of the nukeFromOrbit function`` : The nukeFromOrbit function is a powerful tool that can be used to seize the market balances of sanctioned lenders. If this function is abused, it could lead to the loss of funds for innocent lenders.

- ``Systemic contagion`` : If a major borrower on the Wildcat Protocol defaults, it could lead to a cascade of defaults across other markets. This could have a negative impact on the entire DeFi ecosystem.


## Centralization Risks

Centralization risks depends on how the protocol is operated and governed. The protocol's success and resilience will rely on the careful management of these potential centralization points and the ability to adapt to changing circumstances.

Users and participants should be aware of these risks and consider whether they align with their goals and risk tolerance.

#### WildcatArchController.sol

```solidity

function registerBorrower(address borrower) external onlyOwner {

function removeBorrower(address borrower) external onlyOwner {

function registerControllerFactory(address factory) external onlyOwner {

function removeControllerFactory(address factory) external onlyOwner {

function removeController(address controller) external onlyOwner {

function removeMarket(address market) external onlyOwner {

```

#### Implement a decentralized governance model
A decentralized governance model would give the community more control over the Wildcat Protocol. This would reduce the risk of the protocol being controlled by a single entity.


## Other Recommendations 

### Test Coverage

The current test coverage is ``90% ``

The audit scope of the contracts should be the aim of reaching 100% test coverage. However, the tests provided by the protocol are at a high level of comprehension, giving us the opportunity to easily conduct Proof of Concepts (PoCs) to test certain bugs present in the project.


### Use latest version of OpenZeppelin Contracts 

Currently contracts using vulnerable version like V4.9.0 Use atleast V4.9.2 to avoid known bugs 

```
// OpenZeppelin Contracts (last updated v4.9.0) (token/ERC20/ERC20.sol)

```
### Consider Adding OpenZeppelin access control

OpenZeppelin provides a library for role-based access control, which allows you to manage who can execute specific functions within your smart contract

```solidity
import "@openzeppelin/contracts/access/AccessControl.sol";

```

## New insights and learning from this audit

Wildcat Protocol challenges traditional DeFi lending models by shifting responsibility and control to the borrowers and lenders themselves. It offers a unique approach to collateralization, risk management, and interest rates. However, these innovative features come with increased complexity and risk, making it essential for users to thoroughly understand the protocol's mechanics and the counterparties they engage with.

## Time spent
10 Hours














### Time spent:
10 hours