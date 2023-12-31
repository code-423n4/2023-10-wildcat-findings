# Wildcat Audit Contest Analysis Report | 16 Oct 2023 - 26 Oct 2023

---

| #   | Topic                                                                         |
| --- | ----------------------------------------------------------------------------- |
| 1   | Notes during Wildcat Auditing (Mechanism review)                              |
| 2   | Architecture overview (Codebase Explanation, Examples Executions & Scenarios) |
| 3   | Code Audit Approach                                                           |
| 4   | Codebase Complexity and Codebase Quality                                      |
| 5   | Centralization Risks                                                          |
| 6   | Systemic Risks                                                                |

---

# 1. Notes during Wildcat Auditing (Mechanism review)

**The Wildcat Finance DeFi Protocol is a unique Ethereum-based protocol designed for on-chain fixed-rate private credit. It introduces novel features and approaches that may not be suitable for retail users but cater to sophisticated entities looking to bring credit agreements on-chain while maintaining control over various aspects.**

**Overview:**

- **Primary Offering:** Wildcat's primary offering is credit escrow mechanisms known as "markets", where almost every parameter can be customized at launch, and base APRs and capacities can be adjusted later by borrowers.

- **Borrower Requirements:** Borrowers are required to create a market for a specific asset, offer a particular interest rate, and specify lender addresses explicitly before obtaining credit. This means there should be an existing relationship between counterparties.

- **Collateralization:** Instead of requiring borrowers to provide collateral, a percentage of the supply (reserve ratio) must remain within the market, accruing interest. This serves as a buffer for lenders against withdrawal requests and penalizes borrowers if this ratio is breached.
    To clarify Typically, in lending or borrowing scenarios, borrowers are required to provide collateral as security for the loan they receive. This collateral serves as a guarantee that the lender can claim if the borrower fails to repay the loan. In the case of the Wildcat Finance DeFi Protocol, they have a different approach. Instead of borrowers providing specific assets as collateral, there is a concept called the **"reserve ratio"**. The reserve ratio is a percentage of the total supply of assets associated with a particular market within the protocol. This reserve, which is a portion of the total assets available in the market, is not utilized by the borrower. However, it still generates interest over time. In other words, the assets within this reserve ratio earn interest while they are held within the market.
    So, Wildcat Protocol doesn't require borrowers to provide traditional collateral like specific assets. Instead, they require a certain percentage of the total supply of assets associated with a market (known as the "reserve ratio") to be kept within that market. This reserve ratio generates interest over time, and it serves as a buffer or security for the lenders in case the borrower doesn't meet their obligations.

- **No Underwriters or Insurance:** The protocol does not have underwriters or insurance funds. It is entirely hands-off, and the protocol cannot freeze borrower collateral or pause activity.

- **No Proxies:** The protocol does not utilize proxies, so if users lose keys or face other issues, the protocol cannot assist.

- **Sanctions List:** The protocol monitors for addresses flagged by the Chainalysis oracle and bars them from interacting with markets to prevent poisoning other users.

- **Not for Retail Users:** Wildcat is designed for sophisticated entities, and users must sign a master loan agreement with jurisdiction in the UK courts if utilizing the protocol's UI.

**Technical Briefing:**

- **Archcontroller:** The Wildcat protocol revolves around a single contract known as the "archcontroller." This contract controls which factories can be used, which markets are deployed, and which addresses can deploy contracts.

- **Market Deployment:** Borrowers deploy markets through "controllers" that define logic, permissible lenders, and market parameter limits. Multiple markets can be deployed through a single controller, enabling the use of the same list of lenders for multiple markets.

- **Market Tokens:** Lenders authorized on a controller can deposit assets into markets and receive "market tokens." These tokens are rebasing and can be redeemed at parity for the underlying asset as interest accumulates.
    Parity: In this context, "parity" means that the value or price of the rebasing token remains roughly equivalent to the value of the underlying asset it represents. For example, if the rebasing token is designed to represent 1 gram of gold, then "parity" would mean that the token's price should stay close to the market price of 1 gram of gold.
    For example, if a rebasing token is initially designed to represent the value of 1 ounce of gold, and interest accumulates over time, the token's supply might be adjusted periodically through rebasing to ensure that one token still represents a value close to 1 ounce of gold, even as market conditions change.

- **Interest Rates:** Borrowers pay interest rates consisting of a base APR (accruing to lenders), a protocol APR (accruing to the Wildcat protocol), and a penalty APR (accruing to lenders). The penalty APR activates when a market breaches its reserve ratio for a duration exceeding the grace period defined by the borrower.

- **Withdrawals:** Borrowers can withdraw underlying assets from markets as long as the reserve ratio is maintained. Withdrawals are initiated by addresses with specific roles, and assets are moved into a 'claimable withdrawals pool.' At the end of a withdrawal cycle, lenders can claim assets from the pool, subject to pro-rata dispersal.

- **Withdrawal Queues:** Withdrawal requests that couldn't be honored due to insufficient reserves enter a FIFO withdrawal queue. Non-zero withdrawal queues impact the reserve ratio. Lenders need to burn market tokens to claim assets from the pool.

- **Sanctioned Addresses:** Addresses flagged as sanctioned by Chainalysis are blocked from interacting with markets. A "nukeFromOrbit" function allows for the transfer of their balances into an escrow contract, which can be retrieved under specific conditions.

- **A borrower deploying a market with a `base APR of 10%`, a `protocol APR of 30%`, a `penalty APR of 20%` will pay a true APR of 13% (10% + (10% * 30%)) under normal circumstances and 33% when the market has been delinquent for long enough for the penalty APR to activate. The 30% protocol APR is calculated as 30% of the base APR, so 30% of 10% = 3%  (10% base, 3% protocol)**

---

# 2. Architecture overview (Codebase Explanation, Examples Executions & Scenarios)

![Alt text](image-1.png)

**Overview Architecture -> https://prnt.sc/GISTKqpwap2K**
*Credits: "0xkazim"*

## All Possible Actions of Borrower:
  1. Launching A New Market
  2. Sourcing Deposits
  3. Borrowing From A Market
     - borrower call `borrow(uint256 amount)`, after that within `borrow()` function -> `_getUpdatedState()` -> `check if the requested amount is lower than borrowable amount in the market (if not, then revert)` -> `_writeState()` -> `transfer the requested amount of underlying assets to the borrower`
     - Remember that the capacity you set for your market only dictates the maximum amount that you are able to source from lenders, and that your reserve ratio will dictate the amount of the supply that you cannot remove from a market. If you have created a market with a maximum capacity of 1,000,000 USDC and a reserve ratio of 20%, this means you can borrow up to 800,000 USDC provided that the market is 'full' (i.e. supply is equal to capacity). In the event where the supply to this market is 600,000 USDC, you can only borrow up to 480,000 USDC.
     - **Attack Ideas:**
       - Can borrower borrow more that `borrowable amount` without enter the grace period
  4. Repaying A Market
     - borrower just transfer amount of underlying asset to market ans then should call `updateState()`
  5. Reducing APR
  6. Altering Capacity
  7. Closing A Market
  8. Removal From The ArchController

## All Possible Actions of Lender:
  1. Deposits
  2. Withdrawals
     - The Unclaimed Withdrawals Pool
     - Claiming
     - Expired Claims and The Withdrawal Queue

## The Sentinel Actions:
1. Lender Gets Sanctioned
2. Borrower Gets Sanctioned



### Structs Parameters Explanation:

#### MarketState Struct
- sponsor comment: scaledPendingWithdrawals is tracked because borrowers are obligated to honour 100% of pending withdrawals and the reserve ratio for the rest of the total supply to avoid delinquency.
```jsx
struct MarketState {
    bool isClosed;
    uint128 maxTotalSupply;
    uint128 accruedProtocolFees;
    // Underlying assets reserved for withdrawals which have been paid
    // by the borrower but not yet executed. 
    // sponsor explanation: amount of underlying assets paid to batches but not yet claimed in executeWithdrawal
    uint128 normalizedUnclaimedWithdrawals;
    // Scaled token supply (divided by scaleFactor)
    uint104 scaledTotalSupply;
    // Scaled token amount in withdrawal batches that have not been
    // paid by borrower yet.
    // sponsor explanation: sum of unpaid amounts for all withdrawal batches
    uint104 scaledPendingWithdrawals;
    /// expiry timestamp of the current unexpired withdrawal batch
    uint32 pendingWithdrawalExpiry;
    // Whether market is currently delinquent (liquidity under requirement)
    bool isDelinquent;
    // Seconds borrower has been delinquent
    uint32 timeDelinquent;
    // Annual interest rate accrued to lenders, in basis points
    uint16 annualInterestBips;
    // Percentage of outstanding balance that must be held in liquid reserves
    uint16 reserveRatioBips;
    // Ratio between internal balances and underlying token amounts
    uint112 scaleFactor;
    uint32 lastInterestAccruedTimestamp;
}
```

###

### Functions Flow Explanation:

#### WildcatMarketBase.sol (Base contract for Wildcat markets)

##### `_getUpdatedState()` function flow:

1. **_getUpdatedState()** function performs the following actions:

   - Returns cached MarketState after accruing interest and delinquency / protocol fees and processing expired withdrawal batch, if any.
   - Used by functions that make additional changes to `state`.
   
   - Apply pending interest, delinquency fees and protocol fees to the state and process the pending withdrawal batch if one exists and has expired (accruing interest and delinquency / protocol fees and processing expired withdrawal batch, if any).
   - Used by functions that make additional changes to `state`.
   - Returned `state` does not match `_state` if interest is accrued.
   - Calling function must update `_state` or revert.

2. The function follows a specific logic sequence:

    - It first checks if there is a pending expired withdrawal batch by calling `state.hasPendingExpiredBatch()` verify whether `pendingWithdrawalExpiry` is greater than 0 and less than the current `block.timestamp`.

    - If this condition holds true, the function proceeds to check if `expiry != state.lastInterestAccruedTimestamp`. This check ensures that interest is only accrued if sufficient time has passed since the last update. It also considers cases where `withdrawalBatchDuration` is set to 0.

    - If the above check returns true, the function calls `updateScaleFactorAndFees()`. This function calculates the interest and delinquency/protocol fees accrued since the last state update and applies them to the cached state. It also provides the rates for base interest and delinquency fees, along with the normalized amount of protocol fees accrued.

    - The function then calls `_processExpiredWithdrawalBatch()` to handle any expired withdrawal batches.

3. Finally, the function checks if `block.timestamp` differs from `state.lastInterestAccruedTimestamp`. If this condition is met, it once again calls `updateScaleFactorAndFees()` to apply interest and fees accrued since the last update (either due to an expiry or a previous transaction).

##### `_writeState()` function flow:
- Writes the cached MarketState to storage and emits an event. Also set if the market is in delinquent state.
- Used at the end of all functions which modify `state`.

#### `_getAccount` function flow:
- return the account by his address (the account is struct of scaledBalance and AuthRole)

#### `_blockAccount` function flow:
- Block an account and transfer its balance of market tokens to an escrow contract.
- If the account is already blocked, this function does nothing.
- During function execution the `scaleBalance` of account is set to 0, `approval` to `Blocked` and the `_accounts[escrow].scaledBalance` is increased by the scaledBalance of account in current market state.

#### `_getAccountWithRole` function flow:
- Retrieve an account from storage and assert that it has at least the required role.
- If the account's role is not set, queries the controller to determine if it is an approved lender; if it is, its role is initialized to DepositAndWithdraw.

#### `coverageLiquidity` function flow:
- Returns the amount of underlying assets the borrower is obligated to maintain in the market to avoid delinquency.
- the function call `liquidityRequired()` function which basically determine how much liquidity must be available within the market to cover pending withdrawals, required reserves, protocol fees, and unclaimed withdrawals

#### `scaleFactor()` Function:
- Returns the scale factor (in ray) used to convert scaled balances to normalized balances.

#### `totalAssets()` Function:
- Total balance in underlying asset.

#### `borrowableAssets()` function flow:
- Returns the amount of underlying assets the borrower is allowed to borrow. This is the balance of underlying assets minus: pending (unpaid) withdrawals, paid withdrawals, reserve ratio times the portion of the supply not pending withdrawal, protocol fees
- The function basically call `borrowableAssets()` with `totalAssets()` view function as a parameter

#### `accruedProtocolFees()` function flow:
- Returns the amount of protocol fees (in underlying asset amount) that have accrued and are pending withdrawal.

#### `previousState()` function flow:
- Returns the state of the market as of the last update.

#### `currentState()` function flow:
- Return the state the market would have at the current block after applying interest and fees accrued since the last update and processing the pending withdrawal batch if it is expired.

#### `scaledTotalSupply()` function flow:
- Returns the scaled total supply the vaut would have at the current block after applying interest and fees accrued since the last update and burning market tokens for the pending withdrawal batch if it is expired.
- `return currentState().scaledTotalSupply;`

#### `scaledBalanceOf()` function flow:
- Returns the scaled balance of `account`

#### `withdrawableProtocolFees` function flow:
- Returns the amount of protocol fees that are currently withdrawable by the fee recipient.
- The function basically calls the `MarketState#withdrawableProtocolFees()` with current market state. The `MarketState#withdrawableProtocolFees()` subtract the `normalizedUnclaimedWithdrawals` from `totalAssets` and return -> `MathUtils.min(totalAvailableAssets, state.accruedProtocolFees)`.   


#### `effectiveBorrowerAPR()` function flow:
- Calculate effective interest rate currently paid by borrower.
- Borrower pays base APR, protocol fee (on base APR) and delinquency fee (if delinquent beyond grace period).

#### `effectiveLenderAPR()` function flow:
- Calculate effective interest rate currently earned by lenders. Lenders earn base APR and delinquency fee (if delinquent beyond grace period)

#### `_calculateCurrentState()` function flow:
- Calculate the current state, applying fees and interest accrued since the last state update as well as the effects of withdrawal batch expiry on the market state.
- Identical to _getUpdatedState() except it does not modify storage or emit events.
- Returns expired batch data, if any, so queries against batches have access to the most recent data.

#### `_applyWithdrawalBatchPayment()` function flow:
- Process withdrawal payment, burning market tokens and reserving underlying assets so they are only available for withdrawals.

---


---

# 3. Code Audit Approach

### After first reading the protocol documentation in the "README.md" file, these possible attack vectors popped into my head:

1. **Can the market enter the "Penalty APR" even if the "reserve ratio" is not actually broken?**

2. **The market cannot trigger the "Penalty APR?"**

3. **Closing a Market without the payment of interest?**

4. **Repayment is paused while the "Penalty APR" can be activated?**

5. **Markets contains tokens that the protocol does not support?**

6. **The Market cannot be closed?**

7. **Broken Protocol Invariants/User Flow Assumptions?**

Protocol Assumptions:

- Any ERC-20 can be used as an underlying asset for a market
- There are no fees on transfer tokens.
- The totalSupply is nowhere near 2^128.
- Arbitrary mints and burns are not possible.
- name, symbol and decimals all return valid results.
- Markets that have already been deployed cannot be shut down, and removing a borrower from the ArchController simply prevents them from deploying anything further
- Lenders cannot be prevented from withdrawing unless they have been flagged by the sentinel (even if they are removed from market by borrower).

8. **Broken Protocol Calculations?**

9. **Examine the logic of `Withdraws` (whether the state of the market is properly handled if withdrawal requests couldn't be honored due to insufficient reserves), `Withdrawal Queues`, `Sanctioned Addresses`!**

10. **Examine all possible flows in the Protocol!**

11. **Breaking of trusted role supposed actions in the Protocol**

- Archcontroller Owner Actions:
  - Can add or remove controller factories to/from the ArchController, dictating who can deploy controllers and markets. 
- Borrower Actions:
  - Can add or remove lenders to/from controllers, permitting them to deposit into markets or restricting their ability to deposit further if they have already deposited before.

### Attacks discussed during the Audit:  
- Is it possible for lenders to accrue additional interest or withdraw funds again after a full or partial withdrawal?
- Which state variables responsible for lenders, depositors, or borrowers undergo changes during non-static function calls, and what potential pitfalls could arise from these changes?
- Can lenders or depositors potentially lose their scoringBalance or market tokens in any way?
- How might the protocol's functions be exploited to cause unexpected denial-of-service (DoS) scenarios, potentially due to improper checks or erroneous protocol assumptions?
- Changing variables that play a crucial role in protocol calculations typically leads to errors; what safeguards are in place to prevent this?
- Let's explore a hypothetical scenario of expected code logic and consider how it could be compromised by testing various scenarios to break it.
- In a protocol with multiple user roles, are there checks in place to ensure that functions designed for specific roles cannot be accessed by unauthorized roles?
- How should borrowers deposit assets to fulfill lender withdrawal requests? Is there a dedicated function for this, or do borrowers need to transfer assets to the market manually?


---

# 4. Codebase Complexity and Codebase Quality

## Codebase Complexity Analysis for WildCat Protocol

1. **Structs and Data Organization:**
   - The codebase relies on structured data using the `MarketState` struct to store and manage various market-related parameters.
   - The use of structs enhances readability and maintains a well-organized data structure.

2. **Modularity and Reusability:**
   - The codebase includes functions like `_getUpdatedState()`, `updateScaleFactorAndFees()`, and `_processExpiredWithdrawalBatch()`, which are modular and designed for reusability.
   - Modular functions promote code maintainability and facilitate testing.

3. **Conditions and Branching:**
   - The codebase contains conditional statements to control program flow based on certain conditions. For instance, checking for pending withdrawal batches and interest accrual.
   - Conditionals help ensure that code executes only when specific requirements are met.

4. **Error Handling:**
   - Error handling is present in the form of conditional checks and reverts. For example, checks for valid timestamps and reverting on certain conditions.
   - Proper error handling enhances the robustness of the protocol.

5. **Mathematical Calculations:**
   - The codebase performs mathematical calculations involving scaling, interest rates, and fees. These calculations are critical for market operations.
   - Careful attention to mathematical accuracy is essential to avoid financial risks.

6. **Use of Comments:**
   - Comments are used effectively to provide explanations of code logic, function purposes, and variable descriptions.
   - Well-commented code enhances readability and understanding.

7. **Complexity Control:**
   - The codebase attempts to control complexity by breaking down complex operations into modular functions, promoting code reuse and maintainability.

## Codebase Quality Analysis for WildCat Protocol

1. **Structural Clarity:**
   - The codebase demonstrates clear structuring with well-defined structs and functions.
   - Structs are used to encapsulate related data fields, improving code clarity.

2. **Modularity and Reusability:**
   - The presence of modular functions like `_getUpdatedState()` and `updateScaleFactorAndFees()` promotes reusability and simplifies code maintenance.

3. **Error Handling:**
   - The codebase incorporates error handling through conditional checks and reverts, ensuring robustness and preventing unintended behaviors.

4. **Code Comments:**
   - The codebase employs comments effectively, providing valuable insights into the purpose and functionality of various components.
   - Well-documented code is crucial for collaboration and future development.

5. **Mathematical Accuracy:**
   - The codebase involves mathematical calculations, especially related to interest rates and fees. It is crucial to ensure the accuracy of these calculations to maintain the financial integrity of the protocol.

6. **Use of Solidity Best Practices:**
   - The codebase adheres to Solidity best practices, such as using structs for data organization and modular functions for code reuse.
   - Following best practices helps mitigate potential vulnerabilities.

7. **Readability and Maintainability:**
   - The codebase exhibits good readability through clear structuring, meaningful variable names, and appropriate commenting.
   - Maintaining code readability is essential for ongoing development and collaboration.

8. **Testing:**
   - It is crucial to have comprehensive unit tests and integration tests to validate the functionality of the protocol.
   - Thorough testing contributes to code quality and reliability.

9. **Security Considerations:**
   - Given the financial nature of the protocol, rigorous security audits and testing are necessary to identify and address vulnerabilities.
   - Security should be a top priority in the development process.

In summary, the WildCat Protocol codebase demonstrates good structural clarity, modularity, and error handling practices.

---

# 5. Centralization Risks
**The protocol gives the borrower too much access and control over the lenders. It's also a very sneaky design that the deployer of a specific contract "WildcatMarketController" has control over all lenders for markets with possibly different borrowers (tuned borrower in specific market cannot have control over lenders in that market, only the deployer of WildcatMarketController contract) . Be afair and definitely write this in the decumentation.**

---

# 6. Systemic Risks 

The WildCat Protocol, like any financial system, has its fair share of potential risks that could affect its stability and the safety of users funds. So bellow I write possible risks.

1. **Smart Contract Hazards:**
   - The protocol depends on smart contracts to handle financial operations. Bugs in these contracts, like reentrancy issues or arithmetic errors, pose a significant risk.
   - Safety Measures: Rigorous audits, testing, code reviews, and bug bounties are essential to catch and fix smart contract vulnerabilities.

2. **Liquidity Concerns:**
   - DeFi platforms like WildCat can face situations where there isn't enough money to meet user demands, especially during wild market swings. This can lead to problems like slippage and liquidity shortages.
   - Safety Measures: Encouraging liquidity providers and maintaining sufficient reserves can help ensure there's enough money to go around.

3. **Interest Rates and Price Swings:**
   - The WildCat Protocol involves lending and borrowing assets, which means it's sensitive to changes in interest rates. Also, the value of collateral can fluctuate wildly.
   - Safety Measures: Adjusting interest rates and collateral ratios smartly can help offset the impact of these ups and downs.

4. **Market Delinquency:**
   - Delinquency occurs when borrowers don't keep enough collateral, making the market undercollateralized. This can lead to forced sell-offs and losses.
   - Safety Measures: Keeping an eye on collateral ratios and setting up mechanisms to trigger liquidations can prevent market delinquency.

5. **Protocol Governance:**
   - Governance decisions made by the community can sometimes lead to disagreements or conflicts that disrupt how the protocol operates.
   - Safety Measures: Creating clear governance processes, involving the community, and having ways to resolve disputes can address these issues.

6. **Economic Incentive Conflicts:**
   - Users and the people running the protocol might not always want the same things. This misalignment can cause actions that hurt the protocol's stability.
   - Safety Measures: Thoughtful design of incentives, governance rules, and penalties can help everyone's interests align better.

7. **Regulatory and Compliance Worries:**
   - Regulations can change or authorities might take action that affects how the protocol runs. Not complying with these rules could lead to legal trouble.
   - Safety Measures: Staying informed about regulations, seeking legal advice, and following the rules can help the protocol stay on the right side of the law.

8. **Connected to Other Protocols:**
   - DeFi protocols often interact with each other. If one protocol has problems, it can affect others in a chain reaction.
   - Safety Measures: Doing thorough checks on connected protocols, having ways to pause operations, and having plans for emergencies can minimize risks from interconnectedness.

### Time spent:
25 hours