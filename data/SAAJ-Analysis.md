# Introduction

The analysis is related to Wildcat Protocol which is developed as a DeFi protocol with the  aim to provide undercollateralised lending.  The assessment is based on context of understanding how Wildcat Protocol as an innovative concept  have implemented on-chain credit provision that is undercollateralised.

The aim of this assessment is to look and elaborate on the protocol ability to leverage on transparency and programmability of blockchain technology to monitor the health and status of loan arrangements. The main context that is identified for assessing the codebase is to take a deepdive into Wildcat Protocol as a platform that connects risk-seeking capital (lenders) and borrowers, allowing for more efficient credit provision in the DeFi space.

# Mechanism Review

This section looks at the mechanism of financial markets developed by Wildcat Protocol that depends on correlated working of various contracts which are summarised as below:

```WildcatMarketController``` contract have the role as a deployer and overseeing Wildcat markets. It holds the responsibility of configuring crucial market parameters like APR (Annual Percentage Rate) and reserve ratios with maintaining a list of authorized lenders who can participate in these markets.

```WildcatMarketBase``` serves as a foundational contract for all Wildcat markets, it contains shared functionalities and properties that are inherited by other markets. It main function is to ensure consistency and efficiency across various markets.

```WildcatMarketControllerFactory``` as a factory contract focuses on the deployment of controllers and manages information regarding protocol fees. Controllers are essential for governing market behaviour with maintaining consistency.

```WildcatArchController``` contract acting as a registry, holds information about borrowers, controller factories, controllers, and markets, establishing the relationships between core components and provides a centralization of management to the whole project.

```WildcatMarketWithdrawals``` work is related to enabling withdrawals from Wildcat markets, as it allows users to access their assets

```WildcatMarketConfig``` contract is designed to handle role management and market configuration under the controller's purview. It provides a structured approach to setting and managing various market parameters.

```WildcatMarket``` serves the main contract for Wildcat markets, it inherits properties and functionalities from base contracts and is where the core market functionality is include like executing lending and borrowing operations.

```MarketState``` contract defines the market state struct and provides helper functions for reading from it. This contract is vital for calculating important values like required reserves and balances, which underpin the entire market operation.

```WildcatSanctionsSentinel``` act as a Bridge for the system with Chainalysis, this contract allows borrowers to override lenders' sanction statuses. It also deploys escrows for secure asset management, ensuring compliance with regulatory requirements.

```LibStoredInitCode``` is the main library used to optimize contract deployment by using code storage for init code, preventing oversized contracts. Efficient contract deployment is crucial for a seamless system.

```FIFOQueue``` contract is related to the context of unpaid withdrawal batches, this library defines a First-in-First-out queue. It provides a structured way of handling withdrawal requests, ensuring fairness and efficiency.

```WildcatMarketToken``` contract extends ERC20 functionality to Wildcat markets, allowing the creation and management of tokens within the system. These tokens are integral to users' participation in the markets.

```WildcatSanctionsEscrow``` contract serves as an escrow, holding assets until specific conditions are met. In this case, it manages the assets of accounts on the Chainalysis sanctions list ensuring regulatory compliance.

```Chainalysis``` is library which provide compliance with sanctions lists is essential. This contract defines constants related to the Chainalysis SanctionsList, which is vital for managing the compliance of users and their assets.
# Workflow
This section defines the workflow of the protocol in terms of how each contract interact with eachother and are discussed as below:

```WildcatArchController``` contract manages the overall architecture of the system, interacting with other contracts as needed.

When a new market needs to be created, ```WildcatArchController``` calls ```WildcatMarketControllerFactory``` to create a new ```WildcatMarketController```.

```WildcatMarketController``` manages the operations of the market, interacting with```WildcatMarket```, ```WildcatMarketBase```, ```WildcatMarketConfig```, ```WildcatMarketToken``` and ```WildcatMarketWithdrawals``` as needed.

```WildcatSanctionsSentinel``` monitors and enforces sanctions, interacting with ```WildcatSanctionsEscrow``` and ```WildcatMarket``` to prevent sanctioned funds from being traded.

# Quality Analysis
This section looks at the quality of the codebase in scope of the audit  which is necessary for identifying any existing and potential issues that may lead to greater damage in the future. Quality analysis provides an overview of issues which helps in developing a mitigating strategy to contain the damage.

In terms of code quality, codes are well-written and follow many best practices for Solidity development. The code is well-commented, which makes it easier to understand the purpose of each function and contract. The use of libraries and inheritance helps to keep the code DRY (Don't Repeat Yourself) and makes it easier to manage and update the code. The code is well-structured and use of libraries and inheritance shows a good grasp of code organization and reusability.

 Starting with the ```WildcatMarketConfig``` contract, which is responsible for managing the configuration of the market. The use of the SafeCastLib library for safe type casting is a good practice to prevent overflow and underflow errors which can lead to serious vulnerabilities.

The ```WildcatMarket.sol``` contract is the main contract for managing the market including functions for depositing and withdrawing assets, borrowing assets, and closing the market. The use of the nonReentrant modifier on these functions is a good practice to prevent reentrancy attacks.

```FIFOQueue ``` contract as a utility library that provides a first-in-first-out queue data structure. The functions in this contract are well-structured and include error handling to prevent out-of-bounds errors.

```Chainalysis``` contract  acts as a utility contract for interacting with a sanctions list provided by Chainalysis which is useful for ensuring compliance with regulations.

The overall quality of codebase appears to be well-structured however there are potential vulnerabilities related to access control, uninitialized variables, external calls, integer handling, event data validation, and dependencies on external contracts. The potential issues are highlighted on centralisation and systemic risk section.

# Centralization Risks

The centralization risk identified for the project are related to issues that are related to lack of transparency, increased vulnerability to privilege misuse and potential manipulation of any contract's functions.

The identified potential centralization risks are summarised as below:

**Owner or Admin Privileges:**

In the ```WildcatMarketConfig``` and ```WildcatMarket``` contracts, there are several functions that can only be called by the controller or borrower. The functions are ```closeMarket()```, ```setReserveRatioBips()```, ```setAnnualInterestBips()```, ```setMaxTotalSupply()``` and ```updateAccountAuthorization()```. If the controller or borrower is a single account or controlled by a small group, this could be a centralization risk.

**Sanctions and Blocking:**

The ```nukeFromOrbit()``` and ```stunningReversal()``` functions in ```WildcatMarketConfig``` allows for blocking and unblocking of accounts. These functions can be misused or controlled by a centralized entity and could lead to unfair blocking or unblocking of accounts.

**Fee Collection:**

```collectFees()``` function in ```WildcatMarket``` allows for the collection of protocol fees. If the fee recipient is a single account or a small group of accounts, this could be lead to funds misusing.

It's important to implement proper access control mechanisms to mitigate these centralised risks by using decentralized governance where possible, and ensure transparency in the operation of the contracts.

# Systemic Risks

Systemic risks identified for the project are linked to various factors such as bugs in the code, vulnerabilities to attacks, and design flaws.

**Lack of Input Validation**:

 The codebase lacks validatation inputs from external sources, such as function parameters or data from external contracts. Input validation is critical to prevent potential attacks, like integer overflow, underflow, or reentrancy attacks.

**Use of block.timestamp**:

 The use of block.timestamp for time-based logic may be susceptible to miner manipulation. Miners can manipulate timestamps to potentially benefit from certain conditions in the contract.

**Approval and Authorization Logic**:

The codebase uses authorization mechanism, but the logic for authorizing roles like ```DepositAndWithdraw``` should be carefully reviewed to ensure that it cannot be manipulated or bypassed.

**Handling of Large State Variables**:

The codebase have state variables like WithdrawalData and _accounts. If these data structures grow too large, it can lead to high gas costs and potential vulnerabilities due to gas limits.

**External Contract Calls**:

Any call to external contracts should be carefully reviewed for potential vulnerabilities, especially reentrancy and malicious contract interactions. External calls to other contracts, such as `archController` and `WildcatMarket` are made without verifying the return values or handling potential errors.

**Inadequate Error Handling:**

The project contains custom revert reasons like ```FeeSetWithoutRecipient```. Ensure that these errors provide sufficient information for debugging and security analysis.

**Uninitialized State Variables**:

 Several state variables are uninitialized upon contract deployment, and this can lead to unexpected behavior. ```_tmpMarketParameters``` is not reset after use, which might cause problems in the contract's operation.

 **Integer Overflow and Underflow**:

The project uses low-level assembly to perform integer comparisons, but it doesn't handle integer overflow and underflow explicitly. This can lead to common knwon vulnerabilities if not properly managed.

**Lack of Event Data Validation**:

Events are emitted, but there's no validation of the data passed to these events. This can lead to inconsistencies in the emitted event data, making it difficult for external observers to rely on event information.

# Architecture Recommendation
The analysis provides consideration for some improvement that are made recommended in context of ensuring smooth workflow and security for the overall ```WildCat``` project.

The recommendation is made for error handling, which would be beneficial to provide more specific error messages to help with debugging and understanding the reason for the failure. It's important to ensure that all external calls are marked as external to prevent exposing internal functions and use non-Reentrant modifier to prevent re-entrancy attacks. Also, consider using OpenZeppelin's contracts for standard functionality like ERC20 tokens, as they are widely tested and audited.

The recommendation is also made in area of including comprehensive and well-documented with comments explaining the purpose of each function as it would be beneficial to include comments explaining the logic within complex functions to make the code easier to understand for new developers.

It is also advised to consider implementing an upgradability pattern for critical contracts dealing with funds. This would allow to upgrade the contract logic while preserving the state, which is crucial for fixing bugs or adding new features after the contracts are deployed and any critical bugs are identified.
Some of the contracts use a simple access control mechanism with the ```onlyController``` and ```onlyBorrower``` modifiers.

Consideration must be given for using a more flexible access control mechanism like OpenZeppelin's ```AccessControl``` contract that allows for more granular permissions and roles.



# Conclusion
The time spent on assessing and analysing the code base is around 12 hours. Various resources are used like ```WildCat``` documentation and manifesto with other project audit reports that are linked to area of market makers.

The resources used for analysis helps in getting clear understanding of some key concepts which are utilised for identifying potential risks and making recommendation to improve project performance.




### Time spent:
12 hours