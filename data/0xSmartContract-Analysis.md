# 🛠️ Analysis - Wildcat Protocol
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|h) |Other recommendations | What is unique? How are the existing patterns used? |
|i) |New insights and learning from this audit | Things learned from the project |



## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-10-wildcat

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-10-wildcat#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [WildCat](https://wildcat-protocol.gitbook.io/wildcat/) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| The project does not currently have a slither result, a slither control was created from scratch |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-10-wildcat#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-10-wildcat#scoping-details)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-10-wildcat#attack-ideas-where-to-look-for-bugs)||

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

![image](https://github.com/code-423n4/2023-10-wildcat/assets/104318932/b50cd304-59cd-4c9b-aac2-7f8c3bf5af24)


</br>
</br>

## WildcatArchController.sol

Many Openzeppelin's EnumerableSet structures are used in the WildcatArchController contract. It is necessary to know what this structure basically means and its use in codes;

EnumerableMap: like Solidity's mapping type, but with key-value enumeration: this will let you know how many entries a mapping has, and iterate over them (which is not possible with mapping).

EnumerableSet: like EnumerableMap, but for sets. Can be used to store privileged accounts, issued IDs, etc.
use in code;
![image](https://github.com/code-423n4/2023-10-wildcat/assets/104318932/00001cf8-2ebe-44ed-a043-02e2d70cd4cf)




</br>
</br>

## WildcatMarketController.sol
The deployMarket() function is the most critical function of the contract
Deploys a create2 deployment of `WildcatMarket` unique to the combination of `asset, namePrefix, symbolPrefix` and registers it with the arch-controller;
![image](https://github.com/code-423n4/2023-10-wildcat/assets/104318932/1214ce6d-cfaf-47c5-861f-6be80f1b81bd)


## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content.


### What could they have done better?
- 1) Test suites do not test for re-entrancy
Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here. https://cointelegraph.com/news/curve-finance-pools-exploited-over-24-reentrancy-vulnerability

```solidity
nonReentrant  modifier functions : 24 results 

src\market\WildcatMarket.sol:
   26:   function updateState() external nonReentrant {
   27      MarketState memory state = _getUpdatedState();

   96:   function collectFees() external nonReentrant {
   97      MarketState memory state = _getUpdatedState();

  119:   function borrow(uint256 amount) external onlyBorrower nonReentrant {
  120      MarketState memory state = _getUpdatedState();

  142:   function closeMarket() external onlyController nonReentrant {
  143      MarketState memory state = _getUpdatedState();

src\market\WildcatMarketBase.sol:
  223:   function coverageLiquidity() external view nonReentrantView returns (uint256) {
  224      return currentState().liquidityRequired();

  231:   function scaleFactor() external view nonReentrantView returns (uint256) {
  232      return currentState().scaleFactor;

  252:   function borrowableAssets() external view nonReentrantView returns (uint256) {
  253      return currentState().borrowableAssets(totalAssets());

  260:   function accruedProtocolFees() external view nonReentrantView returns (uint256) {
  261      return currentState().accruedProtocolFees;

  276:   function currentState() public view nonReentrantView returns (MarketState memory state) {
  277      (state, , ) = _calculateCurrentState();

  285:   function scaledTotalSupply() external view nonReentrantView returns (uint256) {
  286      return currentState().scaledTotalSupply;

  292:   function scaledBalanceOf(address account) external view nonReentrantView returns (uint256) {
  293      return _accounts[account].scaledBalance;

  299:   function getAccountRole(address account) external view nonReentrantView returns (AuthRole) {
  300      return _accounts[account].approval;

src\market\WildcatMarketConfig.sol:
   74:   function nukeFromOrbit(address accountAddress) external nonReentrant {
   75      if (!IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {

   88:   function stunningReversal(address accountAddress) external nonReentrant {
   89      Account memory account = _accounts[accountAddress];

  134:   function setMaxTotalSupply(uint256 _maxTotalSupply) external onlyController nonReentrant {
  135      MarketState memory state = _getUpdatedState();

  149:   function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant {
  150      MarketState memory state = _getUpdatedState();

  171:   function setReserveRatioBips(uint16 _reserveRatioBips) public onlyController nonReentrant {
  172      if (_reserveRatioBips > BIP) {

src\market\WildcatMarketToken.sol:
  16:   function balanceOf(address account) public view virtual nonReentrantView returns (uint256) {
  17      (MarketState memory state, , ) = _calculateCurrentState();

  22:   function totalSupply() external view virtual nonReentrantView returns (uint256) {
  23      (MarketState memory state, , ) = _calculateCurrentState();

  31:   function approve(address spender, uint256 amount) external virtual nonReentrant returns (bool) {
  32      _approve(msg.sender, spender, amount);

  35  
  36:   function transfer(address to, uint256 amount) external virtual nonReentrant returns (bool) {
  37      _transfer(msg.sender, to, amount);

src\market\WildcatMarketWithdrawals.sol:

   24:   function getUnpaidBatchExpiries() external view nonReentrantView returns (uint32[] memory) {
   25      return _withdrawalData.unpaidBatches.values();

   77:   function queueWithdrawal(uint256 amount) external nonReentrant {
   78      MarketState memory state = _getUpdatedState();
 
  190:   function processUnpaidWithdrawalBatch() external nonReentrant {
  191      MarketState memory state = _getUpdatedState();


```
The accuracy of the functions has been tested, but it has not been tested whether the `nonReentrant` modifier in the function works correctly or not. For this, a test must be written that tries to reentrancy and was observed to fail.

Let's take the borrow function from the project as an example, we can write any test of this function, it has already been written by the project teams in the test files, but the risk of reentrancy with the lock modifier in this function has not been tested, a test file can be added as follows;

Project File:
```solidity
src\market\WildcatMarket.sol:

  119:   function borrow(uint256 amount) external onlyBorrower nonReentrant {
  120      MarketState memory state = _getUpdatedState();
```
</br>

Reentrancy Test  File:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {DSTest} from "forge-std//test.sol"
import {console2} from "forge-std/console2.sol";

contract WildcatMarketTest {
    using Assert for bool;

    WildcatMarket wildcatMarket;

    function beforeEach() public {
        wildcatMarket = new WildcatMarket();
    
    }

    function testNonReentrantModifier() public {
        bool r = wildcatMarket.borrow(100); 
        r.requireTrue("The first borrow call should be successful");

        bool reverted = false;
        try {
            wildcatMarket.borrow(100); 
        } catch (Error) {
            reverted = true;
        }

        reverted.requireTrue("The second borrow call should revert due to the nonReentrant modifier");
    }
}

```

</br>





## d) Security Approach of the Project

### Successful current security understanding of the project;

1 - First they did the main audit from [ Security review of market logic by alpeh_v](https://hackmd.io/@geistermeister/r15gj_y1p)  and resolved all the security concerns in the report

2- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.


### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. https://immunefi.com/

5- Pause Mechanism
This is a chaotic situation, which can be thought of as a choice between decentralization and security. Having a pause mechanism makes sense in order not to damage user funds in case of a possible problem in the project.

6- Upgradability
There are use cases of the Upgradable pattern in defi projects using mathematical models, but it is a design and security option.



## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-10-wildcat/blob/main/bot-report.md


**Other Audit Reports (Ackee - Trust):**
[ Security review of market logic by alpeh_v](https://hackmd.io/@geistermeister/r15gj_y1p)



##  f) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)     |               ,openzeppelin-contracts/utils/structs/EnumerableSet.sol |-  Version `4.9.3` is used by the project, it is recommended to use the newest version `5.0.0`                                                                                          |
| [`solady`](https://www.npmjs.com/package/solady)                                                                                        | [![npm](https://img.shields.io/npm/v/solady)](https://www.npmjs.com/package/solady)   | Ownable.sol, </br> LibBit.sol, </br> SafeTransferLib.sol   |  -  Version `0.0.124` is used by the project, it is recommended to use the newest version `0.0.129`  </br> - An updated audit of the solady library was done by SpearBit last week, I recommend checking this out [Audit Report](https://github.com/Vectorized/solady/blob/main/audits/cantina-solady-report.pdf)                                                                                  |


## g) Gas Optimizations
When the project is analyzed in terms of Gas Optimization, there is a very important gas optimization;
"Using Mapping instead of Openzeppelin's EnumerableSet library provides high gas optimization"


Other than that, there are some functions written in inline assembly or very minor gas optimizations but they are not mentioned because their effect is very very low, all possible minor gas optimizations are already listed in Auto Bot;
https://github.com/code-423n4/2023-10-wildcat/blob/main/bot-report.md

### Using Mapping instead of Openzeppelin's EnumerableSet library provides high gas optimization

The project used Openzeppelin's EnumerableSet library in the following parts;

```solidity
src\WildcatArchController.sol:
   4: import { EnumerableSet } from 'openzeppelin/contracts/utils/structs/EnumerableSet.sol';

   8  contract WildcatArchController is Ownable {
   9:   using EnumerableSet for EnumerableSet.AddressSet;
  10  
  11:   EnumerableSet.AddressSet internal _markets;
  12:   EnumerableSet.AddressSet internal _controllerFactories;
  13:   EnumerableSet.AddressSet internal _borrowers;
  14:   EnumerableSet.AddressSet internal _controllers;

src\WildcatMarketController.sol:
   4: import { EnumerableSet } from 'openzeppelin/contracts/utils/structs/EnumerableSet.sol';

  32  contract WildcatMarketController is IWildcatMarketControllerEventsAndErrors {
  33:   using EnumerableSet for EnumerableSet.AddressSet;

  70:   EnumerableSet.AddressSet internal _authorizedLenders;
  71:   EnumerableSet.AddressSet internal _controlledMarkets;


src\WildcatMarketControllerFactory.sol:
   4: import { EnumerableSet } from 'openzeppelin/contracts/utils/structs/EnumerableSet.sol';

  13  contract WildcatMarketControllerFactory {
  14:   using EnumerableSet for EnumerableSet.AddressSet;

  63:   EnumerableSet.AddressSet internal _deployedControllers;
```




The use of `EnumerableSet` takes place in the following steps;
1- Define Openzeppelin library with `import`
2- With `using EnumerableSet for EnumerableSet.AddressSet;` it can be used directly without specifying the library name with its definition.
3- In `EnumerableSet.AddressSet private s_priceUpdaters;` a private s_priceUpdaters type variable is declared using the EnumerableSet library

In the following 7 state variables for Gas Optimization, using mapping instead of EnumerableSet will save gas;

```solidity
src\WildcatArchController.sol:
  11:   EnumerableSet.AddressSet internal _markets;
  12:   EnumerableSet.AddressSet internal _controllerFactories;
  13:   EnumerableSet.AddressSet internal _borrowers;
  14:   EnumerableSet.AddressSet internal _controllers;

src\WildcatMarketController.sol:
  70:   EnumerableSet.AddressSet internal _authorizedLenders;
  71:   EnumerableSet.AddressSet internal _controlledMarkets;

src\WildcatMarketControllerFactory.sol:
  63:   EnumerableSet.AddressSet internal _deployedControllers;

```

EnumerableSet is offers efficient operations for adding, removing, and checking the existence of elements in the set.

Here are some key features of EnumerableSet:

Constant-Time Operations: The operations of adding, removing, and checking for existence in a set using EnumerableSet are performed in constant time, which means they have a fixed gas cost regardless of the size of the set. This makes it a suitable choice for scenarios where fast and efficient lookup is required.

Enumeration Capability: EnumerableSet allows you to iterate over the elements in the set. While sets do not guarantee any specific ordering of elements, you can efficiently enumerate through the set using functions like length(), at(index), etc. This enables you to perform operations on each element or retrieve specific elements from the set.

But using Mapping in this project seems more efficient

Because while EnumerableSet provides enumeration capabilities, iterating over a large set consumes a significant amount of gas, especially if you need to perform operations on each item, which is exactly what happens in this project. It's more efficient to use a mapping if accessing them by key has a fixed gas cost

Gas optimization could not be measured due to the complete change of the architecture, but it is obvious that it will provide a significant gas gain in the context of use cases.

## h) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ It is seen that the latest versions of imported important libraries such as Openzeppelin are not used in the project codes, it should be noted. https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/

✅A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

✅In the descriptions, it is stated that the project uses Timelock, but there is nothing about Timelock in the codes.




## i) New insights and learning from this audit 

🔎 1. Unique Credit Model

Wildcat’s approach inverts the standard on-chain credit model. Rather than borrowers approaching a pool of ready lenders, they are tasked with establishing a market and pinpointing potential lenders. This arrangement assumes that there's a prior relationship between the borrower and the lender, making the protocol more suited for private, bespoke credit agreements.

🔎 2. Collateralization Mechanism

Wildcat’s collateralization mechanism is unconventional. Lenders essentially collateralize their own loans. A specified percentage of the loan amount must be retained in the market as a reserve. If the borrower fails to uphold this reserve ratio, a penalty interest rate gets triggered to compensate the lenders.

🔎 3. Risk and Security

The protocol is meant for informed entities who understand and can navigate the inherent risks. Wildcat is hands-off; it doesn’t provide safety nets like insurance funds or underwriting. Users must be aware that the freedom it offers comes paired with significant risks, including the loss of assets due to mismanagement or default.

🔎 4. Interest Rates and Penalties

The interest rate structure is multi-faceted, incorporating base, protocol, and penalty APRs. Borrowers need to be vigilant in maintaining the reserve ratio to avoid triggering the penalty APR. The intricate interest model is designed to balance incentives and penalties ensuring both lenders and borrowers adhere to the agreed terms.

🔎 5. Sanctions and Restrictions

Wildcat Protocol incorporates an automated compliance mechanism through integration with Chainalysis. Addresses flagged for sanctions are automatically restricted from participating in the markets. This feature ensures that the protocol and its users remain compliant with legal requirements without manual oversight.


Wildcat Protocol offers a distinctive approach to on-chain fixed-rate private credit, addressing hurdles like collateralization, interest rate management, and legal compliance innovatively. While it presents enhanced freedom and flexibility, it also comes with inherent risks, making it a tool for informed, sophisticated entities. The delicate balance of freedom and responsibility makes Wildcat a niche yet potent tool in the blockchain credit landscape.




### Time spent:
16 hours