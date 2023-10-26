**Introduction**

The Wildcat Protocol is an as-yet-unreleased protocol launching on the Ethereum mainnet that addresses what we see as blockers in the sphere of on-chain fixed-rate private credit.

# Approach taken in evaluating the codebase

> My analysis of The Wildcat Protocol Included understanding the architecture, mechanism, overall codebase, and possible risks associated with the protocol.

- Day 1: I spent time reading the different available documentation in order to have a deep understanding of the protocol. 

- Day 2: I analyzed the codebase for better understanding, Performed a Mechanism review, and investigated possible systemic risks, and centralization risks. 

- Day 3: I dedicated this day to coming up with possible Architecture recommendations after identifying possible risks Lastly, I prepared the final analysis report.

# Codebase quality analysis

The Wildcat protocol’s codebase is well-structured and well-documented. However, there are areas where improvements could be made, particularly in terms of gas efficiency, permission management, and adherence to best practices.

> Analysis of the codebase (What’s unique? What’s using existing patterns?): 

- Unique: Codebase carries out specific governance mechanisms that are uniquely designed for its specific use case. 

- Existing Patterns: The Wildcat Protocol adheres to common contract management patterns, such as the use of onlyowner, onlyArchControllerOwner, and function for administrative transition (setProtocolFeeConfiguration). 


**Strengths**

- The Wildcat Protocol uses the latest compiler versions (0.8.20) 

**Weaknesses**

- Contract files use floating instead of fixed compiler versions. This is not recommended unless for libraries and packages. 

- Code does not follow the best practice of check-effects-interaction 

- The codebase is vulnerable to centralization risks and a single point of failure.

- Some files are missing NatSpec and some functions are missing @dev and @notice tags.

- Import declarations should import specific identifiers, rather than the whole file

# Centralization risks

> Contracts with privileged functions need owner/admin to be trusted not to perform malicious updates or drain funds. This may also cause a single point failure.

**WildcatArchController.sol**

```
63: function registerBorrower(address borrower) external onlyOwner { 

70: function removeBorrower(address borrower) external onlyOwner { 

106: function registerControllerFactory(address factory) external onlyOwner { 

113: function removeControllerFactory(address factory) external onlyOwner { 

156: function removeController(address controller) external onlyOwner { 
199: function removeMarket(address market) external onlyOwner {
```

**WildcatMarketControllerFactory.sol**

```
function setProtocolFeeConfiguration( 
 address feeRecipient,
 address originationFeeAsset, 
 uint80 originationFeeAmount,  
uint16 protocolFeeBips 
) external onlyArchControllerOwner {
```

# Mechanism review

- The mechanisms implemented in the Wildcat Protocol, including the ERC20 standard and the various modules for ownership management and execution, are well-designed. They provide a comprehensive range of features and capabilities.

Nonetheless, there are some potential issues and risks associated with these mechanisms. For example, the protocol wrongly assumes there are no fees on transfer and also the single point of failure issue highlighted in this analysis. These issues should be carefully considered and mitigated to ensure the security and reliability of the ecosystem.

# Systemic risks

- Like any smart contract-based system, The Wildcat Protocol is exposed to potential coding bugs or vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the protocol. 

- Test Coverage: The test coverage provided by The Wildcat Protocol is >90%, however, I recommend 100% test coverage.

- External Contract Dependencies: The Wildcat Protocol relies on external contracts e.g. OpenZepplin and Solady. If any of these contracts have vulnerabilities, it would affect the protocol.


# Architecture recommendations

- I recommend creating a live testnet app. Here is an [example](https://app.opendollar.com/). 

- I greatly recommend The Wildcat Protocol be designed with decentralization as the main priority to avoid the huge centralization issue the protocol is currently vulnerable to.
 
- Consider implementing Governance functions to be controlled by time locks

- I recommend creating a website for the protocol to keep users informed with an official website.

### Time spent:
23 hours