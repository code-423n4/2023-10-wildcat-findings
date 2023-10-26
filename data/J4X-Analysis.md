## Introduction
This analysis report focuses on the audit of the Wildcat Finance protocol hosted on Code4rena. The primary objectives of this audit were to identify potential vulnerabilities, propose mitigation strategies, and optimize gas usage within the codebase.

```
| Spec        | Value           |
| ----------- | --------------- |
| Name        | Wildcat Finance |
| Duration    | 10 days         |
| nSLOC       | 1879            |
```

## Methodology
The audit process was divided into three distinct phases:

### Recon
In the Recon phase, I conducted an initial review of the codebase to gain a broad understanding of its structure and functionality. This preliminary pass allowed me to familiarize myself with the code before diving into a deeper analysis. I also reviewed provided documentation (whitepaper and gitbook), gathered additional information from online sources, and participated in the audit's Discord channel. Additionally, I reviewed the preliminary audit report.

### Analysis
The Analysis phase formed the core of the audit, involving a detailed examination of the codebase. My approach included meticulously documenting suspicious code sections, identifying minor bugs, and envisioning potential attack vectors. I explored various attack scenarios that could manifest within the code and delved into the relevant functions. Throughout this phase, I maintained ongoing communication with the protocol team to verify findings and clarify any ambiguities.

### Reporting
After compiling initial notes during the Analysis phase, I refined and expanded upon these findings. I also developed test cases to complement the identified issues and integrated them into the final report. Furthermore, I generated a Quality Assurance (QA) report alongside this comprehensive analysis.

## System Overview

The system can be explained based on three key actors that interact with it: the owner, borrowers, and lenders. Each of these actors has their unique pathways for interacting with the market.

### Owner
In the current implementation, the owner is the protocol team. The owner's tasks and functionalities include allowing or disallowing borrowers from deploying new markets and controllers. This is accomplished by the owner interacting with the WildcatMarketControllerFactory and the WildcatArchController. The owner uses these to either register new deployments at the WildcatArchController or to set the ranges for the parameters at the WildcatMarketControllerFactory.

**Recommendations**
In the current implementation, interest rates, protocol fees, and delinquency fees cannot exceed 100%, as the maximum limit the owner can set is 100%. This limitation restricts the protocol's ability to be used for real-world lending, as interest rates can often exceed 100% in various scenarios, such as lending in high inflation environments or high-risk lending. It is advisable to allow for more flexibility in interest rates to cater to real-world use cases.

### Borrower
The borrower is an entity authorized by the owner and can deploy new markets. The borrower is able to deploy one controller, from which he can then deploy multiple markets with different parameters. The borrower also needs to allow lenders to deposit funds into markets, as users are not automatically able to deposit. The authorization updating feature can be seen in the following image:
 ![[WildcatFinance-User Authorization.drawio (1).png]](https://user-images.githubusercontent.com/58374099/278272342-088d033f-a910-4826-8b4c-b95a2d7e19f8.png)

Additionally in case the borrower gets knowledge of a lender being sanctioned by OFAC, the borrower can call to the function `nukeFromOrbit()` to move the lenders assets to an escrow and prevent him from directly receiving assets from the market. This process can be seen in the following diagram:

![[WildcatFinance-nukeFromOrbit().drawio.png]](https://user-images.githubusercontent.com/58374099/278272311-bff9a9e1-c3e0-4a38-a0b7-6bc079fdc32a.png)

**Recommendations**
In the current implementation the users authorization to access a market is stored in 2+ separate locations, which can have different values either between the controller and a market as well as between different markets deployed by the same controller. This opens the door to different vulnerabilities resulting out of inconsistencies. It is recommended to simplify the whole process to mitigate already existing and potential future vulnerabilities. As the authorization should anyways be the same for all markets deployed by a controller, it would make sense to offer a low gas view function on the controller which returns if a user is authorized and call that function for deposits / withdrawals at markets deployed through the controller. This would allow removing all the functionality/storage that needs to be saved in each market.

The implementation does not directly check for the provided prefixes to be related to the borrower. This can lead to borrowers masking as other borrowers without any way to prevent this. It would be recommended to adapt this so that a borrower's market's prefix is somewhat related to the borrower.

### Lender
The lenders interactions with the protocol include the lender depositing into a market as well as withdrawing his funds. The withdrawal of funds is batched, as long as `withdrawalBatchDuration` is not 0. So the withdrawal process takes place in 2 iterations.

The deposit process is rather simple, as shown in the diagram below. A user can deposit underlying tokens up to a certain amount. Then the market will check if the addition to the pool would go over the maximum total supply, and will either let the user deposit the amount of tokens needed to get to the total supply, or the provided parameter.

![[WildcatFinance-DepositMoney.drawio.png]](https://user-images.githubusercontent.com/58374099/278271975-54acee73-6369-466f-a950-e3d82fd1e0e4.png)

While the lender has money invested into the pool, interest is accrued. This interest is credited to the lender using a scale factor. The scale factor increases over time, according to the interest rates set and lets a user withdraw his funds including the gained interest.

The withdrawal process is a lot more complicated and split into the 2 functions `queueWithdrawal()` and `executeWithdrawal()`. Using `queueWithdrawal()` a user can queue his withdrawal request, which will then need to wait until the batch time has passed and the user can withdraw his funds. These batches are used, so that in case of there being too less funds for all lenders to be compensated, the funds are split accordingly to the amount of tokens the lenders burnt.

![[WildcatFinance-queueWithdrawal.drawio.png]](https://user-images.githubusercontent.com/58374099/278272333-66ed4537-4fb8-4a5c-8079-1316cabd4699.png)

When a lender has queued his withdrawal, and the batch period has passed, the lender can withdraw his funds. In case of thee being enough funds the lender just gets his funds + interest. In the case of there being too less liquidity for all lenders in the batch, the available liquidity gets split.

![[WildcatFinance-executeWithdrawal.drawio.png]](https://user-images.githubusercontent.com/58374099/278272286-bdc24636-5859-4792-905e-4143a6cacab3.png)

**Recommendations**
A lender currently has to pass the withdrawal an amount of underlying tokens he wants to withdraw. As these are scaled compared to the market tokens the user has to calculate by himself how many underlying tokens his received market tokens are worth now. Especially as the APR can be changed or a borrower could need to pay the delinquency fee, this would be pretty complicated. It would be simpler for the end user if the end user could just enter the amount of market tokens he wants to  withdraw, and the calculation to the underlying tokens would be done at the protocol side.

## Potential attack vectors

There are multiple assets which can be targets of attacks on the protocol:
- A market's underlying asset holdings
- A market's tokens being held by users
- A market's functionality
- A controllers functionality
- A factory's functionality

Based on this the attack vectors where already mapped pretty well in the contest description. Additionally to the attack vectors mentioned in the contest description the following attack vectors exist:

- Borrower addresses, controller factories or markets are removed from the archcontroller without the intention of the owner
- Borrower can not borrow from a deployed market
- Lenders can not deposit at a market they are authorized for and that has not reached the total supply
- Attacker forces borrower to pay the delinquency rate
- Attacker steals other users market tokens

## Testing

The testsuite is created in foundry and offers testcases for almost all functionalities. Overall the testsuite offers a coverage of around 90%. The tests are well structured and a lot of helper functions simplify the interaction with the contracts. Unfortunately these simplification also lead to some errors when writing new tests, as they for example sometimes mint new assets to users or change authorizations which is not directly stated in their name. This might be very helpful for unit tests, but can lead to unintended behavior in bigger testcases.

Recommendations:
- The testsuite is missing testcases which mimic real usecases of the project (many users, multiple deposits/withdrawals, changes of APR etc.)
- `WildcatMarketControllerFactory.t` does not test the deployment of new controllers at all
- The testuite is lacking unit tests for getter functions
- The `BoolUtils.sol` library is not tested at all
- The testsuite should include more fuzztests to test edge cases

## Systemic Risk
The protocol team has chosen very well in keeping the protocol as hands off as possible. The current implementation limits the damage in case of the archcontroller / owner becoming corrupted. Still, the archcontroller is currently a single point of failure which, if corrupted, allows an attacker to remove borrowers and make them incapable of deploying new markets, effectively freezing the current protocol structure. Nevertheless this would not result in big issues for the end users, due to the hands off setup of the archcontroller.

## Code Quality
The code adheres to high quality standards and is easily readable. Additionally NatSpec comments are added above the functions which allow for easier understanding. Combining the market out of different contracts, which each are separated by their field of functionality simplifies the code. Separating objects like the state into libraries, also makes the code more structured.

## Conclusions
The audit spanned a full ten days and comprised three phases:

- Day 1-2: Documentation review, initial code inspection, and gaining an understanding of the system's functionalities.
- Days 3-9: In-depth code analysis, vulnerability identification, and bug hunting.
- Day 10: Reporting findings, including bug reports and this comprehensive analysis.

Total time spent: 35 hours.



### Time spent:
35 hours