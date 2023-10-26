## Introduction
This analysis focuses on the Wildcat  audit hosted on Code4rena, which pertains to the ENS (Ethereum Name Service) smart contract. The audit process aimed to identify potential vulnerabilities, propose mitigation strategies, and optimize gas usage within the codebase.
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
In the Recon phase, I conducted a preliminary review of the codebase to gain a broad understanding of its structure and functionality. This initial pass allowed me to familiarize myself with the code before diving deeper. Additionally, I examined the provided documentation (whitepaper and gitbook) and gathered supplementary information from online sources and the audit's Discord channel. Additionally I reviewed the preliminary audit report.
### Analysis
The Analysis phase constituted the primary focus of the audit, where I conducted an in-depth examination of the codebase. My approach involved meticulously documenting suspicious code sections, identifying minor bugs, and envisioning potential attack vectors. I explored how various attack scenarios could manifest within the code and delved into the relevant functions. Throughout this phase, I maintained communication with the protocol team to verify findings and clarify any ambiguities.
### Reporting
After compiling initial notes during the Analysis phase, I refined and expanded upon these findings. I also developed test cases to complement the identified issues and integrated them into the final report. Furthermore, I generated a Quality Assurance (QA) report alongside this comprehensive analysis.

## System Overview

The system can be explained based on the 3 actors that interact with it. These are the owner, borrowers and lenders. All 3 actors have their own unique paths in which they can interact with the market.
### Owner
The owner in the current implementation is the protocol team. The owners tasks / functionalities include allowing/disallowing borrowers from deploying new markets, as well as controllers. This is done by the owner interacting with the `WildcatMarketControllerFactory` and the `WildcatArchController`. The owner uses these to either register new deployments  at the `WildcatArchController` or to set the ranges for the parameters at the `WildcatMarketControllerFactory`

**Recommendations**
In the current implementation interest rates, protocol fees and delinquency fees are not able to go above 100%, as the max limit the the owner can set is a 100% and no deployed markets can go over the max limit. This limits the protocols capabilities to be used for real world lending, as interest rates can in reality go above 100% in multiple scenarios. Exemplary would be lending of currencies with high inflation rates, high risk lending or junk bonds.
### Borrower
The borrower is a entity that is authorized by the owner and can deploy new controllers / markets. For each controller the borrower can deploy multiple markets with different parameters. The borrower also needs to allow lenders to deposit to markets, by interacting with the controller as users are not automatically able to deposit. The authorization updating feature can be seen in the following image:
 ![[WildcatFinance-User Authorization.drawio (1).png]](https://user-images.githubusercontent.com/58374099/278272342-088d033f-a910-4826-8b4c-b95a2d7e19f8.png)


Additionally in case the borrower gets knowledge of a lender being sanctioned by OFAC, the borrower can call to the function `nukeFromOrbit()` to move the lenders assets to an escrow and prevent him from directly receiving assets from the market. This process can be seen in the following diagram:

![[WildcatFinance-nukeFromOrbit().drawio.png]](https://user-images.githubusercontent.com/58374099/278272311-bff9a9e1-c3e0-4a38-a0b7-6bc079fdc32a.png)

**Recommendations**
In the current implementation the users authorization to access a market is stored in 2 separate locations, which can have different values either between the controller and a market as well as between different markets deployed by the same controller. This opens the door to different vulnerabilities resulting out of inconsistencies. It is recommended to simplify the whole process to mitigate already existing and potential future vulnerabilities. As the authorization should anyways be the same for all markets deployed by a controller, it would make sense to offer a low gas view function on the controller which returns if a user is authorized and call that function for deposits / withdrawals. This would allow removing all the functionality/storage that needs to be saved in each market.

The current implementation does not directly check for the provided prefixes to be related to the borrower. This can lead to borrowers masking as other borrowers without any way to prevent this. It would be recommended to adapt this so that a borrower's market's prefix is somewhat related to the borrower.

### Lender
The lenders interactions with the protocol include the lender depositing into a market as well as withdrawing his funds. The withdrawal of funds is batched, as long as `withdrawalBatchDuration` is not 0. So the withdrawal process takes place in 2 iterations.

The deposit process is rather simple, as shown in the diagram below. A user can deposit up to a certain amount, then the market will check if the addition to the pool would go over the maximum total supply, and will either let the user deposit the amount of tokens needed to get to the total supply, or the provided parameter.

![[WildcatFinance-DepositMoney.drawio.png]](https://user-images.githubusercontent.com/58374099/278271975-54acee73-6369-466f-a950-e3d82fd1e0e4.png)

While the lender has money invested into the pool, interest is accrued. This interest is credited to the lender using a scale factor. The scale factor increases over time, according to the interest rates set and lets a user withdraw his funds including the gained interest.

The withdrawal process is a lot more complicated and split into the 2 functions `queueWithdrawal()` and `executeWithdrawal()`. Using `queueWithdrawal()` a user can queue his withdrawal request, which will then need to wait until the batch time has passed and the user can withdraw his funds. These batches are used, so that in case of there being too less funds for all lenders to be compensated, the funds are split accordingly to the amount of tokens the lenders burnt.

![[WildcatFinance-queueWithdrawal.drawio.png]](https://user-images.githubusercontent.com/58374099/278272333-66ed4537-4fb8-4a5c-8079-1316cabd4699.png)

When a lender has queued his withdrawal, and the batch period has passed, the lender can withdraw his funds. In case of thee being enough funds the lender just gets his funds + interest. in the case of there being too less liquidity for all lenders in the batch, the available liquidity gets split.

![[WildcatFinance-executeWithdrawal.drawio.png]](https://user-images.githubusercontent.com/58374099/278272286-bdc24636-5859-4792-905e-4143a6cacab3.png)

**Recommendations**
A lender currently has to pass the withdrawal an amount of underlying tokens he wants to withdraw. As these are scaled compared to the market tokens the user has to calculate by himself how many underlying tokens his received market tokens are worth now. Especially as the APR can be changed or a borrower could need to pay the delinquency fee, this would be pretty complicated. It would be simpler for the end user if the end user could just enter the amount of market tokens he wants to  withdraw, and the calculation to the underlying tokens would be done at the protocol side.

## Potential attack vectors

There are multiple assets which can be targets of attacks on the protocol:
- A market's underlying asset holdings
- A market's tokens being held by users
- A market's functionality
- A market controllers functionality
- The MarketControllerFactory's functionality

Based on this the attack vectors where already mapped pretty well in the contest description. Additionally to the attack vectors mentioned in the contest description the following attack vectors exist:

- Borrower addresses, controller factories or markets are removed from the archcontroller without the intention of the owner
- Borrower can not borrow from a deployed market
- Lenders can not deposit at a market they are authorized for and has not reached totalSupply
- Attacker forces borrower to pay the delinquency rate
- Attacker steals other users market tokens

## Testing

The testsuite is created in foundry and offers testcases for almost all functionalities. Overall the testsuite offers a coverage of around 90%. The tests are well structured and a lot of helper functions simplify the interaction with the contracts. Unfortunately these simplification also lead to some errors when writing new tests, as they for example sometimes mint new assets to users or change authorizations which is not directly stated in their name. This might be very helpful for unit tests, but can lead to unintended behavior in bigger testcases.

Recommendations:
- Testsuite is missing testcases which mimic real use of the project (many users, multiple deposits/withdrawals, changes of APR etc.)
- `WildcatMarketControllerFactory.t` does not test the deployment of new controllers at all
- Testuite is lacking lots of unit tests for getters etc.
- `BoolUtils.sol` is not tested at all
- Testsuite does not include any fuzz testcases, which should be added 

## Systemic Risk
The protocol team has chosen very well in keeping the protocol as hands off as possible. The current implementation limits the damage in case of the archcontroller / owner becoming corrupted. Still, the archcontroller is currently a single point of failure which if corrupted allows an attacker to remove borrowers and make them incapable of deploying new markets, effectively freezing the current protocol structure. Nevertheless this would not result in big issues for the end users, due to the hands off setup of the archcontroller.

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