# Description
WildCat is banking but with a twist, as the developers explained. Overall, it operates like a traditional bank, where borrowers and lenders are known and trust the system. It's an undercollateralized (or uncollateralized) lending platform where borrowers can open a market, and lenders will supply the funds.

Every pool has some rules:
- It employs a 2-step withdrawal process where you need to wait for the withdrawal request to "expire" before you can access the funds.
- There are three types of APR, all determined by the borrower, with two of them generating profits for the lenders.
- Delinquency is a key factor, where if the borrower is unable to repay their debts, their paying APR increases dramatically, leading to increased profits for the lenders.

The borrower incurs certain fees, which are calculated as follows:
- Annual APR: This is the primary way for lenders to earn money, with lenders earning this APR constantly for their assets.
- Protocol fee APR: This is how WildCat generates its revenue. It's a percentage on top of the annual APR, calculated as a percentage of the annual rate. For example, when the annual rate is 10% and the protocol fee is 30%, the APR that the borrower needs to pay is 10 * 30% = 3% on their borrowed assets.
- Delinquency fee: This is the APR that the borrower pays every time they cannot afford to maintain the minimum ratio of the pool.

| Severity | Occurrences |
|----------|-------------|
| High     | 1           |
| Medium   | 4           |
| Low      | 3           |
| NC       | 2           |
| GAS      | 3           |
| Analysis | 3 hours     |


# Findings
The codebase was remarkably simple and straightforward, just as I expected. There weren't many bugs, most of the issues I reported can be classified as 'medium' since their impact is not significant. The report also contains a few 'low' and 'non-critical' issues. Most of my time was spent trying to break the lending and borrowing mechanisms. Although these mechanisms make up half of the code, the other half, including deployments and controllers, were straightforward and simple. They mainly consist of getters and setters, which are accessible to administrators.


# Codebase Quality
The code quality was excellent, featuring simple yet proficient code. Many explanations were provided in the GitBook and could be inferred between the lines. Additionally, the developers were highly proficient in offering more in-depth details on how things are done and why they are done that way.

1. **Code Comments:** The comments were excellent in providing more information about the specific logic.

2. **Documentation:** The documentation was well-written, engaging, and packed with information.

3. **Organization:** The codebase is very mature and well-organized, with clear distinctions between functions and their purposes.

4. **Testing:**
   - **Unit Tests:** The unit tests were sophisticated. Despite the codebase's simplicity without major complications or unnecessary complexity, the tests were comprehensive, covering various attack angles. I also wrote some tests in Foundry, and the code behaved as expected.

   - **Fuzz Tests:** There were numerous fuzz tests covering most of the functions, indicating the developers' commitment to the protocol's stability. They were willing to invest extra effort, which some other projects might consider unnecessary."


# Positive Suggestions

- When setting `setAnnualInterestBips`, don't use 90% as the required reserve ratio for that period.90% means that the borrwer needs 100%, as some lenders might withdraw and make him delinquent. I suggest that the borrower, when creating the market, can set their own ratio, with a minimum of 40% or 50%.

# Systemic Risks

- `processUnpaidWithdrawalBatch` needs to be manually triggered by the borrower or another entity. However, if the withdrawal waiting period is short (or 0), a newly scheduled batch can be executed before any of the unpaid batches.

- When an account is blocked, there is no way to retrieve the funds. I would suggest implementing an admin function or some mechanism that would allow retrieving the funds from the escrow when they have been sitting there for an extended period. This would enable the return of stolen or hacked funds to the protocol from which they were stolen.

- `_getUpdatedState` seems vulnerable and could potentially cause issues in the future. This vulnerability arises from the possibility of updating the time into the past, as demonstrated by the condition: `if (state.hasPendingExpiredBatch())`, which updates the state up to the point where the batch expired. Although I haven't found a way to exploit it and update the `lastInterestAccruedTimestamp` to a past point (backwards) at the moment, it may become a problem in the future.

# Conclusion
Wildcat code was good and functioning well. As you have already read, there are some recommendations on what is potentially risky and what can be improved. This analysis was intentionally kept small to enhance its value for the reader, prioritizing the eficiency between 'time spent on reading' to 'quality suggestions.'"


### Time spent:
20 hours