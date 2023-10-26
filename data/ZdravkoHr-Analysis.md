# Submitted Issues

### 2 Highs
| Description | Impact | Likelihood |
| ----------- | ----------- | ----------- |
| Wrong argument ordering in createEscrow | HIGH | HIGH |
| Markets can never be closed | HIGH | HIGH |

### 5 Mediums
| Description | Impact | Likelihood |
| ----------- | ----------- | ----------- |
| CREATE2 success is not checked | MEDIUM | LOW |
| Everyone can get the WithdrawOnly role | MEDIUM | HIGH |
| Borrower can set market interest outside of the minimimum and maximum range | MEDIUM | HIGH
| Borrower may have to pay more fee for delinquency than needed | HIGH | LOW
| Max total supply cannot be changed | MEDIUM | HIGH

### QA Report
| Description | Severity |
| ----------- | -------- |
| Lender's deposit may be stuck in the mempool | LOW |
| Market token name missing space | NC |
| Escrow address is not stored in WildcatMarket | NC |
| Mistake in a comment | NC |

### Gas Optimizations
| Add an early return |
| ------------------- |

# Auditing approach

During this 10-days audit the codebase was tested in various different ways. The first step after cloning the repository was ensuring that all tests provided pass successfully when run. After that began the phase when I got some high-level overview of the functionality by reading the code. There were some parts of the project that caught my attention. These parts were tested with Foundry POCs. Some of them were false positives, others turned out to be true positives that were reported. The contest's main page does a wonderful job providing main invariants that should never be broken and ideas for exploits. The [*gitbook*](https://wildcat-protocol.gitbook.io/wildcat/) is also very good resource, even though some inconsistencies and wrong numbers were found there. These were fixed by the sponsors. I also had a look at the external contracts that the protocol integrates with, for example, OpenZeppelin's EnumerableSet.

# Mechanism review

Wildcat is a protocol that lets borrowers take uncollateralized loans from approved lenders. It inverts the typical model of lending and borrowing in DEFI - borrowers create vaults and lenders deposit their funds in these vaults. Via reserve ratio percents, lenders' liquidity plays the role of their own collateral. In order for a borrower to be eligible to deploy a market he has to first sign a **Service Agreement** offchain. 

The "grandparent" of the protocol is the ArchController contract. It handles registration of borrowers, controllers and factories. A borrower that is registered in the ArchController can then use a registered factory to deploy a controller. The factory holds the settings for the controller that when deployed, makes a call to it to fetch them. 

### Deployments overview
All deployments in the protocol happen through the [*LibStoredInitCode*](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol) library. Let's have a look at an example of deploying a controller. The idea is that we deploy a contract which holds the controller's creation code prefixed by a STOP opcode (0x00) to avoid executing the code as a smart contract:  
  - Call [*LibStoredInitCode.deployInitCode(data)*](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol) with the creation code of the controller.
  - The function makes use of assembly to deploy a new contract. It uses the memory location [data, data + 32] (the length of the controller's creation code) to not waste additional memory space.
  - In this location we store a prefix code. 
  - The prefix code looks like this: **0x6100005f81600a5f39f300** (total of 11 bytes)
  - Get the size of the controller's creation code and add 1 to it (because we will prepend a STOP opcode)
  - According to this [*EIP*](https://eips.ethereum.org/EIPS/eip-2677) the creation code cannot exceed 2 bytes. For example's sake let's assume that the controller's size is 0xac. Then the size + 1 will be 0xad.
  - We shift the new size to the left by 64 bits (8 bytes). 
  - This leaves us with ad000000000000000 (total of 9 bytes)
  - Apply bitwise **OR** to the prefix code and the new size to position the size between **61** and **5f**
  - The new result is 0x61**00ad**5f81600a5f39f300 (total of 11 bytes)
  - This result gets saved in the memory slot [data, data + 32]
  - Since the EVM stores whole words in the memory (32 bytes), it will be prepended by 21 bytes of zeroes.
  - The next step is to create the contract **create(0, add(data, 21), createSize)**. Here we are ignoring the 21 bytes added by the EVM and start loading from **0x6100ad5f81600a5f39f300**. We load **createSize** bytes. createSize is the original size + 11 (because the prepended code is 11 bytes).
  - A new contract is created. Let's further break down the creation code of the newly created contract (**0x6100ad5f81600a5f39f300** + the rest of the controller's creation code).

  | Byte | Opcode | Stack |   
  | ---- | ------ | ----- |
  | 0x61 | PUSH1 | 0x00ad |
  | 0x5f   | PUSH0 |  0x00  |

  The next opcode is 0x39 (codecopy). It will copy a part of the init code and will store it in memory. It will store it in 0x00, will start copying from 0x0a (10) and will copy 0x00ad bytes. Lets take a look at the beginning of the init code again: **0x6100ad5f81600a5f39f300**. Starting from 0x0a, this will copy the code from the end of the prefix code (which is 0x00, or the STOP) to the end of the controller's creation code. The code is now copied in memory and we have the following stack: 

  | Byte | Opcode | Stack |   
  | ---- | ------ | ----- |
  | 0x61 | PUSH1 | 0x00ad |
  | 0x5f   | PUSH0 |  0x00  |

The next and final opcode is 0xf3 (return). We return 0x0ad bytes loading from 0x00. Which is the controller's initialization code prepended with a stop opcode. This is now the runtime code of the newly created contract.

 - This newly created contract can now be used to deploy the actual controller.

### Deposits and withdrawals overview
When a borrower creates a controller, he then authorizes the lenders that he wants to interact with. He also deploys a market using this controller. Each market works with only one ERC20 token. A fee for creating a new market may be enabled.

The authorized lenders can deposit their ERC20 funds in a market they are approved for by calling either **deposit** or **depositUpTo** and specified amount.
  - The state of the market is updated. This happens every time a non-view function is called.
  - If the market is closed or the depositor tries to deposit an amount of funds that will go over the market capacity, the transaction will be reverted.
  - The specified amount of tokens are transferred from the lender to the market. This means that the lender has to approve the market to spend his tokens before the deposit.
  - The ledner's account's balance is incremented by the **scaled amount** of the assets for internal accounting. In other words, lender gets minted market tokens.
  - The new tokens are added to the market's balance.

Lenders can also withdraw funds that they have deposited. First, they have to queue a new withdrawal: 
 - Withdrawals happen in batches depending on the specified withdrawal cycle length on deployment. 
 - If they do not have at least the **WithdrawOnly** role, the transaction will be reverted.
 - Update the state and handle expired batch, if existent.
 - An amount of market token is transferred from the lender to the market.
 - If a batch already exists, add the withdrawal to it, otherwise create a new batch.
 - Check if there is some liquidity in the market to be transferred to the unclaimed rewards pool.
 - When tokens are transferred to the unclaimed rewards pool, they are burned from the total market supply.
 - The minimum of the available liquidity and the requested amount is transferred to the unclaimed rewards pool.
 - When the withdrwal's cycle period ends, the request has to be executed by calling **executeWithdrawal**. 
 - If there are enough funds in the unclaimed rewards pool, then all the lenders that requested in this batch can be paid. Otherwise, they will receive the same % of assets of the unclaimed pool as their % in the batch. For example, if Lender A requests 2000 tokens and Lender B requests 8000 tokens, but in the unclaimed pool there are only 600 tokens, Lender A will receive 120 tokens and Lender B will receive the rest.
 - In case that the lender is sanctioned, the assets will be send to an escrow contract.
 - If a batch cannot be fully paid, it is added to a FIFO queue. 
 - Later on, when the market's funds increase, these batches will be paid in the order they were added to the queue.

### Borrowing overview
The main idea of the protocol is to let borrowers borrow. This happens when a eligible account calls the **borrow** function:
 - If the market is closed, the transaction will be reverted.
 - If the borrower tries to take a loan greater than the allowed amount, the transaction will be reverted. The allowed amount is determined by using a reserve ratio percent.
 - The specified amount is transferred from the market to the borrower

### Interest and taxes overview
There are several interest and taxes mechanisms in the protocol:
  - base interest APR - this is the % which the lender has to pay annually on top of the lender's funds.
  - protocol fee - a % of the base interest that has to be paid annually to the protocol's fee recipient.
  - delinqency fee - a % of interest that must be paid to the lender while the market is in a delinquent state. Delinquent state means that the current supply of assets is less than the required liquidity. The market has a grace period in which a delinquent market's owner does not have to pay. After the grace period ends, if still delinquent, delinquency interest accrual starts.

The whole interest system is cleverly managed by using a **scaleFactor** in RAYs.

When a lender deposits funds, the amount deposited (called normal amount) is divided by that scaleFactor and the result is recorded as a scaledAmount.  As time passes and interest accrues, the scaleFactor is increased. Then, when converting back to a normal amount, the scaledAmount is multiplied by the scaleFactor. This means that as more time passes the same scaledAmount will be equal to a larger and larger normal amount.

### Other borrower permissions
   - Borrower can close the market whenever he decides to, given that it is not currently delinquent, or that it wont become delinquent. Lenders can still withdraw from a closed market, but they cannot deposit anymore. Borrowing from a closed market is prohibited.
   - Borrower can change the interest APR. This sounds problematic, but preventive actions are taken in this scenario. If a borrower wants to increase the APR, they are free to do so. On the other hand, if they decrease it, the reserve ratio of market is set to 90% for 2 weeks.
   - The borrower can adjust the max capacity of their markets.





# Codebase quality

This codebase's quality is good even though some vulnerabilities were found. It makes use of clever techniques for deploying contracts and internal balance tracking. The code's tests have a good coverage too. However, they are somehow convoluted and hard to be understand at first.

# Architecture recommendations

During my analysis of the codebase, I thought some changes may be done to improve the protocol:

 1. **Stop interest accrual for sanctioned lenders** - the protocol uses [*Chainalysis*](https://www.chainalysis.com/free-cryptocurrency-sanctions-screening-tools/) oracle to check if users are sanctioned. If a user is sanctioned, he gets blocked from Wildcat. This means that all his market tokens are sent to an escrow created specially for him. When the sanction is lifted off, the user is free to take back all of his assets. Because the escrow holds market tokens, this means that sanctioned users benefit from interest on their assets just like any other lender. Initially, I though that was a finding but @d1llon explained that its intended to be like that. I continued discussing thing issue with @functionZer0. One of the main reasons for this decision is that possible solution for stopping interest accrual would be to force the lender to fully exit the market. However, there may be some cases where the market is not liquid enough for such operation. I think it's still possible to achieve this goal without making the lender exit his position. This will require modifying the escrow a bit. It can remember the normalized amount that was sent to it and at the time of withdrawal - adjust the scaled amount accordingly.
 2. **Store the escrows' addresses in the market** - currently, when an escrow is created, assets are sent to it but its address is not saved anywhere. This will probably be handled by a frontend, but still it makes querying its address a lot more harder. I would suggest storing the addresses in a **mapping(address user => address escrow)**.
 3. **Make use of sanctioned user's funds** - realistically, if a user gets in the sanctions list its very unlikely that he will get out of it. Of course, the possibility of the oracle becoming malicious exists and that is why the sanctions overrides are implemented. I suggest adding a long period of time (10 years or something like that) where if the lender has not been unsanctioned during it, its funds get distributed to the current lenders depending on the portion of the whole market's balance that they hold. This utilization of idle funds will be a good incentives for lenders to both follow the rules and provide liquidity.
 4. **Create functions for some actions that call updateState()** - the market keeps important information in a struct called **state**. This state holds the interest, capacity, delinquency and other settings. In the current implementation, a function for returning borrowed assets does not exist. This means that the borrower has to do a direct ERC20 transfer in order to pay his debts. After transferring, updateState has to be called. If it is not, some problems may arise. This 2-step payment makes it harder for the borrower. There may be a situation when the second transaction gets stuck in the mempool and the state is not updated. Consider adding functions that call updateState() after the given operation.

# Centralization risks

There are some centralization risks. For example, if the account of the archcontroller's owner gets compromised, a lot of things may go wrong. Protocol fee may be set to a too high one, and the fee recipient to a foreign entity. This will cause a huge lose of money. 

There also exists the risk of a borrower losing his private key. The developers have thought of that scenario letting other accounts pay borrower's debt on his behalf by just transferring tokens. This may be used by the same borrower that lost his private keys.

# Systematic risks
 1. The Chainalysis oracle becoming malicious may cause a discord. In the case of wrongly blocked lender, the borrower can just override the sanction himself. However, the interval between the sanction and the overriding is still bad experience for the lender that may have wanted to interact with a market.
 2. If the borrowers decides to misbehave, he may choose to not pay his debts. This situation would be solved offchain with the proper legislation, but such an action can undermine the reputation of the protocol. Another faulty action of the borrower would be removing sanctions of correctly blocked lenders.
 3. Of course, the possibility of exploiting the system through a vulnerability in external integrations, such as dependency contracts, always exists.




### Time spent:
50 hours