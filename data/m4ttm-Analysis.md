## 1\. Audit Approach

|     |     |     |
| --- | --- | --- |
| Step | Task | Details |
| 1   | Run Tests | Tests run successfully |
| 2   | Coverage | 90% test coverage for contracts in audit scope |
| 3   | Slither | Reviewed Slither results, no vulnerabilities discovered |
| 4   | Surya | Generate graphs to understand the overall project structure. Provided an initial insight to the contract inheritance and function call flow |
| 5   | Solidity Metrics | Generate metrics reports to obtain initial insight on the codebase, noting areas of potential concern |
| 6   | Code Review | Line by line code review |
| 7   | Test Review | Review of each test and it's purpose |

## 2\. Mechanism Summary

Wildcat offers a protocol for uncollateralised loans, backed by a legally binding contract. Someone who wishes to create a loan creates a `WildcatMarket` using a `WildcatMarketController`, which are deployed using a `WildcatMarketControllerFactory`. These are deployed through a single `WildcatArchController`. Sanctioned addresses are prevented from using the protocol using the Chainanalysis oracle and `WildcatSanctionsEscrow` is used to hold funds should an address become sanctioned.

## 3\. Centralisation Risks

### 3.1 Owner controls who can borrow and lend

Using the `WildcatArchController`, the owner of the protocol has the ability to control who borrows and lends using the protocol and can restrict an existing borrower from creating new loans.

### 3.2 Borrowers control who can lend to their markets

Borrowers have the power to control who lends to their markets and can prevent lenders of their choosing from lending them money.

## 4\. Systemic Risks

### 4.1 Uncollateralised Loans

An element of trust in necessary due to the nature of uncollateralised loans. Although the loans are supposed to be backed up by a legally binding contract, the borrower could still default and there is no guarantee the lender will be repaid, unlike collateralised loans which can simply be liquidated.

## 5\. Quality Analysis

### 5.1 Codebase

The code is well structured and organised into separate contracts and libraries each with a clear purpose. There are some clear gas optimisations that could be made, such as removing unnecessary reentrancy guards or avoiding copying structs to memory.

### 5.2 Documentation

Detailed and clear [documentation](https://wildcat-protocol.gitbook.io/wildcat/) is provided for the project at both a user level and a technical level. Natspec comments are well used throughout the code.

### 5.3 Tests

Tests are clearly structured and cover most of the core functionality but coverage could be better than 90%. Testing could be further improved with the use of fuzz testing, or formal verification to give users the extra assurance that their funds are safe.

## 6\. Architecture Improvements

### 6.1 Use Reentrancy Guards More Sparingly

Reentrancy guards can offer an extra layer of protection but come with additional gas overhead and can offer a false sense of security when not well understood due to not protecting against cross contract reentrancy. Consider whether reentrancy guards are necessary for a particular function by following the checks, effects, interactions pattern and checking for external calls that can exploit an outdated piece of state.

### 6.2 Combine Controllers into a single Factory Contract

`WildcatArchController`, `WildcatMarketControllerFactory` and `WildcatMarketController` are used to create and deploy markets and act as registries to control particular parameters as well as managing access control. This functionality could be achieved through the use of a single deployed contract, making the system easier to read on block explorers and minimising deployment cost.

### Time spent:
25 hours