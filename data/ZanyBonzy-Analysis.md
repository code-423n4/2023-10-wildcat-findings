# **Protocol Overview**
  - **About wildcat protocol**

The Wildcat Protocol is an upcoming Ethereum-based protocol for on-chain fixed-rate private credit. It targets sophisticated entities who want to bring credit agreements on-chain without surrendering control to third parties.

Wildcat markets are credit escrow mechanisms where nearly all parameters can be modified by the borrower at launch. Borrowers must also explicitly specify the desired lender addresses before credit can be obtained.

Collateralization is handled by requiring a percentage of the supply, the reserve ratio, to remain within the market.

The protocol is non-interventionist when it comes to any given market and has no ability to freeze borrower collateral or pause activity. It also uses no proxies and cannot help users who lose keys, burn market tokens, or experience other problems.

Wildcat markets are not aimed at retail or power users and come with significant counterparty and smart contract risks. The protocol monitors for addresses that are flagged by the Chainalysis oracle as being placed on a sanctions list and bars them from interacting with markets.

If counterparties are utilizing the protocol UI, the borrower must sign a master loan agreement. This agreement sets out various covenants, representations, and definitions of default, with jurisdiction for any conflict that arises being placed within the UK courts. Lenders can countersign this agreement or decline.

  - **Technicals**

The Wildcat protocol revolves around the archcontroller. This contract determines which controller factories can be used, which markets have been deployed, and which addresses are allowed to deploy contracts from those factories.

Borrowers deploy markets through controllers, which are contracts that define the logic, permissible lenders, and minimum and maximum values of market parameters. Multiple markets can be deployed through a single controller, allowing a single list of lenders to access multiple markets simultaneously.

Authorized lenders can deposit assets into any markets. In exchange for their deposits, they receive a market token that is parameterized by the borrower. These market tokens are rebasing. The interest rate on the borrower's debt compounds every time a non-static call is made to the market contract.

The interest rate paid by the borrower consists of three components: the base APR (accruing to the lender), the protocol APR (accruing to the Wildcat protocol itself), and the penalty APR (also accruing to the lender). The penalty APR is activated when a market has been delinquent for a certain period of time, defined by the borrower. The borrower has a grace period to bring the market back to a non-delinquent state before the penalty APR applies.

Borrowers can withdraw underlying assets from the market as long as the reserve ratio is maintained. Withdrawals are initiated by addresses with the WithdrawOnly or DepositAndWithdraw roles, and market tokens are burned to move assets into a claimable withdrawals pool. At the end of a withdrawal cycle, lenders can claim assets from the pool, subject to pro-rata dispersal if the amount requested exceeds the available assets.

Withdrawal requests that couldn't be honored due to insufficient reserves are marked as expired and enter a FIFO withdrawal queue. Non-zero withdrawal queues impact the reserve ratio, and any subsequent deposits by the borrower are routed into the claimable withdrawals pool until there are enough assets to honor all expired withdrawals.

Lenders flagged by Chainalysis as sanctioned are blocked from interacting with any markets they are a part of and have their market balances directed to an escrow contract. Lenders can retrieve their assets from the escrow contract if they are removed from the oracle or if the borrower manually overrides their sanctioned status.

  - **Roles**

ArchController Owner - Adds or removes controller factories from the ArchController, dictating who can deploy controllers and markets.

Borrower - Adds or removes lenders to or from controllers, permitting them to deposit into markets or restricting their ability to deposit further if they have already deposited.

Lenders authorized on a given controller can deposit assets to any markets launched through it, for which they receive a rebasing market token.

# **Architecture Review**
  - **Contracts Review(Scope)**

[WildcatMarketController](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol) - creates and oversees markets, setting their interest rates, reserve ratios, and approved lenders.

[WildcatMarketControllerFactory](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol) - is in charge of deploying controllers for the Wildcat protocol and managing information about protocol fees.

[WildcatArchController](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol) - is a central registry for borrowers, controller factories, controllers, and markets.

[WildcatSanctionsSentinel](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol) - works with Chainalysis to allow borrowers to override lenders' sanction statuses and deploy escrow accounts.

[WildcatSanctionsEscrow](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol) - holds assets for a sanctioned lender until he's removed from sanction's list or until the borrower overrides the his sanction status.	

[WildcatMarketBase](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol) - is the base contract for all Wildcat markets, and it provides the basic functionality for creating and managing markets. 

[WildcatMarketWithdrawals](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol) - provides the functionality for withdrawing tokens from Wildcat markets. 

[WildcatMarketConfig](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol) - provides methods for managing roles and configuring markets by the controller. 

[WildcatMarket](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol) - is the main contract for Wildcat markets, and it inherits all of the base contracts. 

[WildcatMarketToken](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol) - provides the ERC20 functionality for Wildcat markets.

  - **Other dependencies**

[MathUtils library](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol) - contains the math functions used in the codebase.

[SafeCastLib library](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/SafeCastLib.sol) - contains cast functions.

[FeeMath library](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol) - contains functions for calculating interest, protocol fees and delinquency fees.

[StringQuery library](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/StringQuery.sol) - contains helper functions for querying strings from methods that can return either string or bytes32.

[MarketState library](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol) - defines the market state struct and helper functions for reading from it and calculating basic values like required reserves and scaling/normalizing balances.

[LibStoredInitCode library](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol) - deploys contracts using code storage for init code to prevent oversized contracts.

[FIFOQueue library](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol) - implements a first-in-first-out queue used for unpaid withdrawal batches.

[Errors library](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Errors.sol) - contains helper functions and constants for errors and Solidity panic codes.

[Withdrawal library](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Withdrawal.sol) - defines the withdrawal state struct.

[BoolUtils library](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/BoolUtils.sol) - contains helpers for booleans.

[Chainalysis library](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Chainalysis.sol) - contains the constant for Chainalysis SanctionsList.

[ReentrancyGuard library](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/ReentrancyGuard.sol)	- is based on Seaport and prevents functions from being reentered.

[@solady/SafeTransferLib library](https://github.com/Vectorized/solady/blob/main/src/utils/SafeTransferLib.sol) - provides safe transfer functions for ERC20 deposits, withdrawals and other transfers.

[@solady/LibBit library](https://github.com/Vectorized/solady/blob/main/src/utils/LibBit.sol) - provides bit manipulation functions and querying strings from methods that can return either string or bytes32.

[@openzeppelin/EnumerableSet library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/structs/EnumerableSet.sol) - provides an efficient implementation of a set data structure. Controls the registries, market list, lenders and borrower's list, etc.

[@solady/Ownable library](https://github.com/Vectorized/solady/blob/main/src/auth/Ownable.sol) - provides a simple implementation of the Ownable contract pattern. It allows the archcontoller to have control over a smart contract.

# **Codebase Review**

  - **Comments and Documentation**

The codebase is well-documented and commented. There is not a lot of missing information or outdated documentation. The provided documents also include the whitepaper, manifesto, and GitBook.

  - **Error Handling**

The codebase mostly uses custom errors, but without details. However, the error names seem descriptive enough. In a few cases, errors are not used. There are also revert errors, but no require errors.

  - **Event Emission**

Important events are emitted where needed, but in some cases, they lack sender information or previous and new parameters. Additionally, to best follow the best practices of the Checks-Effect-Interactions (CEI), it is advisable to emit events before external calls and avoid emitting them inside loops.

  - **NATSPEC and Style Guide**

NATSPEC and the style guide are not consistently followed. Contract layout, order of functions, and function names do not follow the style guide in a number of cases. Additionally, in some cases, the library names do not match the filename. NATSPEC documentation appears to be lacking. Important tags, such as @params, @dev, @notice, and @return, are missing in a number of contracts.

  - **Tests** 

Test coverage is high, at about 90% This is commendable. 

# **Protocol Risks**

  - **Centralization Risks**: The protocol depends on the archcontroller to perform a host of important functions. The arch controller owner must be trusted not to act maliciously or get hacked. Additionally, the owner can renounce ownership and leave the protocol adminless.

  - **Imported Contracts**: There are about three or four directly imported libraries and some others created based on existing libraries. Any issues with these contracts can directly translate to risks in the protocols. The current OpenZeppelin contracts version is 4.9.3; as of the time of auditing, the latest version is 5.0.

  - **Smart Contract Errors**: Critical security flaws, access control errors, and logical inconsistencies can lead to loss of assets or system manipulations. Due to interdependencies between contracts, an issue with one contract might not be contained to the contract itself. It can lead to a domino effect, affecting the entire protocol.

  - **Chainalysis Oracle**: The Chainalysis oracle must be trusted not to go haywire or go on a banning spree. Borrowers, lenders, and escrows can get sanctioned, putting the protocol functionality at risk.

  - **Weird Tokens**: The protocol uses ERC20 tokens. Accommodations have to be made for non-standard tokens. The protocol does this by using Solmate's SafeTransfer library.

# **Approach to audit & Conclusions**

  To audit the codebase, we started by reviewing the provided documentation: the gitbook, whitepaper, and manifesto. We noted the important areas and asked the developers for explanations of unclear parts. Afterwards, we ran the contracts through the static analyzer and linter, Solhint. We compared the generated report to the bot report, noting similarities and differences, and generated a QA report from there. Once that was done, we began the process of manual inspection. We studied the contracts, compared the tests, and worked with various attack ideas to see how the functionality could be broken. We tested for common vulnerabilities, as well as other possible logical inconsistencies.
  
  The protocol introduces a new and innovative approach to the borrower-lender relationship, which is a departure from the traditional model. This alone makes the project interesting and worth watching. The codebase appears to be solid, but the identified risks need to be addressed. The issue of centralization should be taken seriously, as "trust" is not a secure enough foundation. Implementing a multi-signature wallet or timelocks would help to mitigate this risk. Additionally, the documentation and comments in the code should be improved to make it easier for developers to understand and collaborate. Finally, regular upgrades and security audits should be performed to keep the protocol safe from malicious attacks.




### Time spent:
40 hours