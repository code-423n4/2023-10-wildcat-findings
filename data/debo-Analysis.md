## Any comments for the judge to contextualize your findings
I found reentrancy.
I have found valid reentrancy in another audit.
I'm a certified warden.

## Approach taken in evaluating the codebase
Search for keywords that are linked to potential vulnerabilities.
Then write exploit and impact analysis for it.

## Architecture recommendations
Use emit events more.

## Codebase quality analysis
Use the latest version of solidity.

## Centralisation risks
The WildcatMarketBase Contract has many areas of centralisation that can be threats:

Borrower: The contract has one account with the power to borrow assets from the contract. 
If this account is hacked, it could progress to a loss of funds.

Controller: The contract has one controller account that can make market parameters. 
If this account is hacked, it could change the market prices in malicious ways.

Sentinel: The contract has one sentinel account that is able to stop other accounts. 
If this account is hacked, it could progress to malicious stopping access of accounts.

Fee Recipient: The contract has one account that accepts protocol charges. 
If this account is hacked, it could progress to theft of charges.

Withdrawal Batch Period: The contract has one fixed period prior to withdrawal batches being transacted. 
If this is fixed incredibly high, it might slow down withdrawals for stakeholders of the contract.

Delinquency Grace Period: The contract has one fixed grace duration prior to delinquency building a penalty charge. If this is fixed incredibly low, it could unfairly penalise account holders.

## Mechanism review
This seem to be a well structured mechanism.

## Systemic risks
Governance risk: If the private key is compromised then there could be a loss of funds and data protection breaches.

### Time spent:
35 hours