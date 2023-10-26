## Approach taken in evaluating the codebase

Everyone in this system is somehow "whitelisted", so the risk is limited. For this review, I treated borrowers as admin and lenders as classic users.

## Codebase quality analysis

The overall codebase is well built and it was a pleasure to review it. But I would have designed the protocol differently, as I explained in the "Architecture Recommendations" section.

For a contract running on the Ethereum mainnet, contracts are not really gas optimized. There are simple and safe gas optimization techniques that can be used here.

## Architecture recommendations / Mechanism review

I would have built this protocol by copying the design of government bonds. The design would have been much simpler and broken down into 3 phases:

1. the funding : lenders give money
2. the operation : once the maximum supply is reached, the borrower can take the money up to a certain maturity. Before the maturity, the borrower has to refund the market.
3. the refunding : the borrower have to refund + pay interest. Once done lenders can harvest there money.

As you have noticed, the withdrawal mechanism is removed, instead we can imagine an optimized secondary market with a small AMM.

A liquidity pool must be created for each market and by default 20% of the liquidity is given to the pool to allow the lender to trade there shares (so the borrower can borrow up to 80%). This liquidity belongs to the market: it is added during the **operation** phase and removed when the market is refund.

This way, if a borrower is no longer trusted, the share price will fall and reach a market value that corresponds to the risk people are willing to pay for the reward.

This is a proper application of capitalism. In this way, if confidence in the borrower is lost, people can sell their shares and new lenders can come in to buy the higher risk, higher reward share. Just like bonds in traditional finance (you don't have to be "banking but worse": sometimes the old world is right).

If the borrower needs more time to repay his debt, he can open a new market to refinance his debt. (Just like the current system)

Why do I think the current system is not ideal?

- The batch payoff mechanism introduces a lot of overhead and code complexity.
- Lenders are incentivized to always request a withdrawal to be liquid. If there is no problem, the user can reinvest there stake, if there is a problem, the lender keeps the asset.
- In case the borrower does not refund the market, first lenders who asked the withdrawal will be paid and the others will never be.
- This brings so much incertity for the borrower: he must keep a certain collateral ratio to not be delinquent and at the same time anyone can freely request a withdrawal. The borrower will have to constantly monitor his markets.

Some people will say that this is the current broken traditional system, and they will be right. But if the trusted debt system works this way, maybe it's because we've found through iteration that this is the best way to deal with debt.

## Centralization Risks / Systemic Risks

There is a lot of trust in the borrower. Rug can happen, and to make people come, I think it can be good to create an insurance reserve that comes from fees. This way if a bad actor rug or backrupt Wildcat can try to refund users.

Also, as I said, everyone who interacts with Wildcat is "whitelisted", which means there is a huge centralization risk on the arc controller owner. Make sure this wallet uses multisig with as many signers as possible.

Since the entire platform is not upgradable, migration in the event of a vulnerability. The team should prepare multiple recovery scenarios and set up appropriate action channels to prepare for such eventualities.

### Time spent:
30 hours