| List | Head | Details |
| --- | --- | --- |
| 1   | introduction to The Wildcat Protocol | introduction to  The Wildcat Protocol, its features and components |
| 2   | Audit approach | Process and steps I followed |
| 6   | Architecture recommendations | some recommendations related to the architecture |
| 7   | Codebase Analysis | Analysis of the codebase |
| 8   | Codebase Quality | how was the quality of the codebase |
| 9   | How could they have done it better? | Some best code practice suggestions |
| 10  | Possible Systemic Risks | The possible systemic risks based on my analysis |
| 11  | Centralization risks | Concerns associated with centralized systems |
| 12  | Security Approach | Security Approach of the Project |
| 13  | Gas Optimizations | Details about my gas optimizations findings and gas savings |
| 14  | Documents | how was the Documents that they provided for us |
| 15  | Recommendations | some recommendations to the  Wildcat Protocol teams |
| 16  | Conclusion | Conclusion |
| 17  | Time spent on analysis | The Over all time spend for this reports |

## Introduction

Wildcat is a protocol that allows borrowers to deploy undercollateralized credit lines.

### some features of Wildcat

- Vaults can be created for any ERC20 that the borrower wishes to acquire,
- The maximum capacity of a vault can be adjusted at will up or down by the borrower depending on their current need,
- Borrowers can determine the reserve ratio of a vault on creation, the minimum percentage of deposits that cannot be withdrawn but still accrue interest. Note the implication here that vaults are uncollateralized by a borrower: collateral is rather sourced from the lenders themselves,
- Borrowers determine the length of the withdrawal cycle: the rolling period during which multiple lenders will have their redemption amounts prorated if their sum exceeds the available vault collateral,
- Borrowers can choose a grace period after which vaults that have an insufficient reserve ratio incur a penalty APR that indicate the strength of a borrower’s intention to maintain promised levels of collateral,
- Borrowers maintain their own list of addresses that are eligible to lend to the vaults deployed from a given controller contract, subject to their own KYC processes, and
- Borrowers can terminate a vault at any time, dropping the APR to 0% while simultaneously returning all outstanding collateral and interest: this blocks their ability to withdraw, and allows lenders to exit at their leisure.

### From the perspective of lenders

- Lenders can deposit to - and withdraw from - any active vault deployed by a borrower that has approved them on the relevant controller contract,
- Lenders that have had their approval to interact with a borrower revoked cannot deposit further, but retain the ability to withdraw,
- Lenders benefit from a penalty APR that triggers after a vault remains delinquent (under its reserve ratio) for a rolling length of time beyond the aforementioned grace period. This penalty is distributed to all lenders, and no part of it is receivable by the Wildcat protocol itself.

There are three main components to the Wildcat protocol: the controllers, the factory, and the vault tokens (or rather, the vaults): • Controllers handle permissions for both borrowers and lenders alike, • The factory calls into a controller when deploying a vault, • Vaults query a specified controller to determine who can interact.

### Main components of the Wildcat protocol

There are three main components to the Wildcat protocol: the controllers, the factory, and the vault tokens (or rather, the vaults)

• Controllers handle permissions for both borrowers and lenders alike,

• The factory calls into a controller when deploying a vault,

• Vaults query a specified controller to determine who can interact.

## Audit Approach

1.  **Initial Scope and Documentation Review**: Thoroughly went through the Contest Readme File and project documentation, White paper to understand the protocol's objectives and functionalities.
    
2.  **High-level Architecture Understanding**: Performed an initial architecture review of the codebase by going through all files without going into function details.
    
3.  **Test Environment Setup and Analysis**: Set up a test environment and execute all tests. Additionally, use Static Analyzer tools like Slither to identify potential vulnerabilities.
    
4.  **Comprehensive Code Review**: Conducted a line-by-line code review focusing on understanding code functionalities.
    
    - **Understand Codebase Functionalities**: Began by going through the functionalities of the codebase to gain a clear understanding of its operations.
    - **Identify Access Control Issues**: Thoroughly examine the codebase for any access control problems that might allow unauthorized users to execute critical functions.
    - **Evaluate Function Execution Order**: Checked random sequence of function executions to ensure the protocol's logic cannot be disrupted by changing the order of calls.
    - **Assess State Variable Handling**: Identify state variables and assess the possibility of breaking assumptions to manipulate them into exploitable states, leading to unintended exploitation of the protocol's functionality.
5.  **Report Writing**: Write a Report by compiling all the insights I gained throughout the line-by-line code review.
    

## Architecture recommendations

I suggest having the contract under audit equipped with a complete set of NatSpec detailing all input and output parameters pertaining to each functions although much has been done in specifying each @dev. Additionally, a thorough set of flowcharts, and if possible a video walkthrough, could mean a lot to the developers/auditors/users in all areas of interests.

### Codebase Analysis

- The codebase is well-structured and follows Solidity best practices. The contracts are well-documented, which aids in understanding the functionality and purpose of each contract. The use of libraries for common functions helps to reduce code duplication and increase readability. However, the complexity of the system could potentially lead to unexpected behavior or vulnerabilities.

## Codebase Quality

- **Modular Design:** The code is organized into different contracts, each responsible for specific functionality. This makes the codebase easier to understand and maintain.
    
- **Clear Comments:** There are comments throughout the code that explain the purpose of functions, variables, and sections. This enhances readability and comprehension.
    
- **Use of Interfaces:** The code leverages interfaces to interact with other contracts. This is a good practice for decoupling and reusability.
    
- **Access Control:** The contracts implement access control using modifiers like onlyOwner, and onlyBorrower. This ensures that certain functions can only be called by authorized parties.
    
- **Events:** Events are used to provide transparency and enable easier tracking of important contract interactions.
    

## How could they have done it better?

While the provided code seems to be functional, there are always areas for improvement to ensure better security, efficiency, and readability in the contract and the protocol.

- Here are some potential areas for improvement:
    
- **Modularization**: Break down the code into smaller, well-defined functions and contracts. This makes the code easier to understand, test, and maintain.
    
- **Comments and Documentation**: Provide thorough comments and documentation throughout the code to explain the purpose, functionality, and any potential gotchas of each component.
    
- **Input Validation**: Validate and sanitize all user inputs to prevent unexpected behavior or attacks. For example, checking that input amounts are positive, non-zero, and within reasonable bounds.
    
- **Consistent Naming Conventions**: Use consistent and descriptive variable and function names to make the code more understandable.
    
- **Gas Efficiency**: Optimize the code for gas efficiency. This involves minimizing unnecessary computations, storage, and external calls to save on transaction costs.
    
- **Avoid Reentrancy Vulnerabilities**: Implement safeguards to prevent reentrancy attacks, such as using the "Checks-Effects-Interactions" pattern and using the reentrancyGuard modifier to prevent multiple calls to the same function.
    
- **Use Latest Solidity Version**: Keep up-to-date with the latest Solidity version and use its latest features and improvements.
    

## Possible Systemic Risks:

- **Algorithmic Complexity**: The contract involves complex mathematical calculations and interactions. Errors in these calculations could lead to incorrect outcomes, affecting the functioning of the contract and potentially leading to unexpected losses for users.
    
- **Rounding Errors**: Rounding errors can accumulate over time due to frequent calculations involving floating-point arithmetic. These errors could lead to discrepancies between expected and actual token balances, causing loss of funds.
    
- **Front-Running and Manipulation**: Complex financial systems can be susceptible to front-running attacks where malicious actors place transactions ahead of others to gain an advantage. The evolving nature of the bonding curve could provide opportunities for manipulation.
    
- **Unintended Consequences**: The interactions between different parts of the contract could lead to unintended consequences, negatively impacting users. For instance, changes in the curve parameters over time could lead to unforeseen behaviors.
    

## Centralization Risk

All the governance controls seem like centralization risks throughout the code, as has been pointed out here:

https://github.com/code-423n4/2023-10-wildcat/blob/main/bot-report.md#m01-centralization-risk-for-privileged-functions

## Security Approach of the Project

- **Minimize Complexity**: Keep the smart contract logic as simple as possible to reduce the attack surface. Complex systems are more prone to bugs and vulnerabilities.
    
- **Separation of Concerns**: Divide the contract's functionalities into smaller, modular components. This can help isolate vulnerabilities and make the contract easier to audit.
    
- **Secure Coding Guidelines**: Enforce secure coding practices among developers. Follow best practices for Solidity development to minimize the risk of introducing vulnerabilities.
    

Remember that security is an ongoing process, and regular reviews, updates, and improvements are essential to stay ahead of emerging threats. Collaborating with experienced blockchain security professionals and conducting frequent security assessments can significantly enhance the security posture of the project.

## Gas Optimization

Wildcat Protocol is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and codebase size.

## Documents

The documentation provided for the Wildcat Protocol is quite comprehensive and detailed in terms of explaining its functionality, parameter usage, purpose, and overall architecture. However, here are some suggestions that could further enhance the clarity and understanding of the documentation:

- Use [Grammarly](http://www.grammarly.com/): documentation should have correct grammar, spelling, punctuation.
    
    ```
    Wildcat is a protocol that allows borrowers to deploy undercollateralised credit lines. 
    
    should be written: 
    
    Wildcat is a protocol that allows borrowers to deploy undercollateralized credit lines. 
    
    // undercollateralised  <------ incorrect
    
    // undercollateralized  <------ correct
    ```
    
- **Visual Examples**: Include diagrams or visual aids to help readers understand concepts better.
    
- **Practical Examples**: Provide specific numerical examples to help readers apply calculations and parameters in real-world scenarios.
    
- **Detailed Use Cases**: Expand use cases to demonstrate how the contract applies in real life.
    
- **Highlighted Key Terms**: Boldly highlight technical terms when first mentioned to help readers identify important concepts.
    

## Recommendations:

To enhance the security posture of the Wildcat Protocol, several measures can be considered. Implementing extra security mechanisms like timelock delay consensus or requiring multisig approvals for critical changes can provide an extra layer of protection. Introducing a decentralized validation process for significant upgrades could help ensure that changes are authorized by a broader consensus. Additionally, documentation should be improved to provide comprehensive insights into roles, responsibilities, and interactions within the system. Thorough testing, particularly focused on upgrade and timelock processes, is essential for identifying and mitigating potential vulnerabilities. For further assurance, a formal audit by a reputable smart contract security firm is highly recommended.

## Conclusion

In general, the `Wildcat project` exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

## Time Spent on the Audit

- Approximately 22 hours

### Time spent:
22 hours