# ****The Wildcat Protocol**** Analysis

**1. Analysis of the Codebase:**

The Wildcat project showcases a strong dedication to best practices and code quality, featuring both distinctive elements and established patterns:

- **Unique Aspects**:
    - This code employs **Chainalysis** to access a list of sanctioned users, enabling the dApp to determine whether to permit or restrict their usage.
    - The contracts in this codebase are designed as **immutable**. Once deployed, the core logic cannot be updated.
- **Existing Patterns**:
    - The codebase leverages well-established Math Libraries from widely recognized projects.
    - Yul Assembly is used as needed for tasks such as value comparisons, address computation, and more.
    - EnumerableSet is employed instead of traditional arrays for more efficient data storage.
    - The project adheres to the decentralization ethos by granting users considerable power while maintaining limited control for protocol owners.

**2. Architecture Feedback:**

[Architecture of WildCat Protocol](https://github.com/zzzuhaibmohd/Code4rena/blob/main/Architecture%20of%20WildCat%20Protocol.png)

The following diagram shown above illustrates the execution flow for Wildcat Protocol.

### **Contracts Overview:**

***WildcatMarketControllerFactory.sol***

- This contract is utilized to create a new instance of the WildcatMarketController template, which can subsequently be employed by a borrower to deploy a market.
- The contract currently uses magic numbers when comparing input as part of the constructor. Enhance code readability by replacing these magic numbers with constants.

***WildcatMarketController.sol***

- The contract enables borrowers to deploy a market and manage the addition or removal of lenders from the market. An important enhancement here would be to prevent borrowers from also becoming lenders within the same market.
- Furthermore, borrowers have the ability to update the annual interest rates for the lenders.

***WildcatMarket.sol***

- This contract inherits several other contracts, including WildcatMarketBase, WildcatMarketConfig, WildcatMarketToken, and WildcatMarketWithdrawals.
- It provides the functionality for lenders to deposit funds into a market.
- Borrowers can utilize the contract to borrow funds or close a market when they no longer require the funds.

***WildcatMarketBase.sol***

- The contract primarily comprises helper functions that retrieve various parameters and information related to a market.

***WildcatMarketConfig.sol***

- This contract allows setting the Total Supply, Annual Interest Rates, and Reserve Ratio rates.
- It includes a critical function to blacklist addresses listed on the Chainalysis Sanction List.

***WildcatMarketToken.sol***

- An ERC20 token with a custom implementation for standard ERC20 functions such as transfer, transferFrom, and approve, among others.

***WildcatMarketWithdrawals.sol***

- The contract primarily manages all the logic related to lenders' fund withdrawals. Lenders can queue a withdrawal, and the transfer of funds is processed based on the current liquidity in the market.

***WildcatSanctionsSentinel.sol***

- The contract helps create an Escrow contract if a lender is in the Chainanalysis Sanction List.
- It also contains function that can override the sanction list status.

***WildcatSanctionsEscrow.sol***

- Once an escrow contract is deployed, the functions that are part of the contract check for the current status of address and decide to release or hold the funds.

***WildcatArchController.sol***

- The contract has associated function that are relevant to protocol owners.
- The contracts allows to register a borrower, market etc.,

**3. Centralization Risks:**

The Wildcat protocol's design aims to maintain decentralization.

- The contracts are **immutable**, meaning cannot be updated.
- The protocol owner decide who can become a borrower and can also remove them if need be.
- The borrower gets to decide who the lenders are going to be as part of the deployed market.

**4. Systemic Risks:**

Systemic risks relate to potential issues affecting the entire system. In WildCat, these include:

- **Borrower can steal Funds** : The Wildcat Protocol makes it a point to do proper KYC and other background check before starting a new market for a borrower but still there is a clear risk of borrower running away with all the funds of the lenders.
- **Denial of Service via Withdrawals:** A market has set Reserve ratio below which the withdrawals of the lenders is not processed. Only when new lenders deposit funds or if the borrower pays back the funds is the transaction processed.
- **Borrower becomes a Lender:**  Allowing borrowers to add themselves as lenders within the same market can create artificial market activity through wash trading, potentially tricking other lenders into depositing more funds into the market.

**5. Other Recommendations:**

To enhance the ****Wildcat**** Protocol project, the following recommendations are made:

- The protocol can interact with any underlying asset which is non OZ compliant, so return values of `transfer()`/`transferFrom()` are to be checked.
- Do not hardcode addresses like the Chainalysis Sanctions List, as they may change in the future.
- The code appears to use many `immutables`. It's essential to reconsider this choice, particularly for address data types like `feeRecipient` , since they cannot be updated alter on.
- Implement a two-step ownership transfer via `Ownable2Step.sol` for enhanced security.
- Missing checks for `address(0)` when setting address state variables.
- Use `SafeCast` to downcast safely.
- Follow the best practice of check-effects-interaction.
- Follow the @natspec standard for all functions to improve code readability and documentation.

**6. Time Spent:**

The analysis was conducted within approximately 75 hours, ensuring a thorough examination of the codebase, architecture, risks, and recommendations.

### Time spent:
72 hours