### Overview

The Wildcat protocol facilitates the creation of credit lines that require less collateral from borrowers compared to the usual solutions available in web3. The protocol provides significant control to borrowers. These are able to control key parameters like interest rates, underlying assets, penalties, reserve ratios etc.

The protocol is responsible for vetting the borrowers and in turn borrowers are responsible for vetting their lenders, everything being conducted under the umbrella of a `Service Agreement` concluded between the protocol and all of its users (both lenders and borrowers). The relation between borrowers and lenders is governed by a `Master Loan Agreement` that needs to be signed by the borrower when a new market is deployed and by the lender at the first interaction they have with the market.

After a market is deployed Wildcat has no control over it.

### Understanding the Ecosystem

Borrowers go through a thorough vetting process carried by Wildcat and could be considered trusted accounts.
Borrowers use approved factory contracts to deploy market controllers.
Borrowers have control over market creation and can set all the initial market parameters:
- asset
- namePrefix
- symbolPrefix
- maxTotalSupply
- annualInterestBips
- delinquencyFeeBips
- withdrawalBatchDuration
- reserveRatioBips
- delinquencyGracePeriod

Borrowers can deploy multiple markets and whitelist lenders for each market.
Lenders receive market tokens based on the amount they deposit.
Special protections are in place to prevent APR manipulation by borrowers.
Withdrawals are carried from the reserve amount, established through the `reserveRatio`, pro-rated in case the amount in sufficient.
Borrowers are considered delinquent if they fail to maintain the reserve ratio. There is a `grace period` in which the borrower needs to deposit funds to replenish the reserve. In case the borrower doesn't do that, they'll have to pay a penalty APR. The delinquency is tracked using a `grace tracker` that functions as a rolling limit (once delinquency has been cured within a market, the grace tracker will count back down to zero from whatever value it had reached),

### Codebase Analysis

The codebase is well-structured and follows best practices for smart contract development. There are some comments attached that go hand in hand with the great documentation in facilitating a great understanding of the project. It would have been better if the protocol provided some form of flow charts, which could have been very helpful to someone with a limited time or background in auditing this type of protocols. I took the liberty of drafting one to get a better grasp of the project.

<img src="https://user-images.githubusercontent.com/95440897/277175194-7f25912d-b4ce-432c-9552-ab91a278d270.png" alt="A nice sketch representing contracts and functions">

### Architecture Recommendations

The overall architecture is great. I was surprised to see critical protocol variables `withdrawableFees`, `borrowableAssets`, `availableLiquidity` based on `IERC20(asset).balanceOf(address(this))` which can obviously be inflated by anyone. The project does this because transferring the underlying token to the market contract is the only way a `borrower` can repay his debt. I advise creating a function that can be used by the `borrower` to repay his debt and change the way `totalAssets()` is derived by tracking inflows and outflows from the authorized parties.

Moreover, the protocol should limit the tokens that can be used. It’s more cautious to ensure that borrowers aren’t asking for weird ERC20 tokens at protocol level by making a list of approved tokens (one that doesn’t include rebasing tokens for example).

### Centralization Risks

The overall centralization risk is low. Indeed, the protocol vets every single user and can take back any permission at any moment, but the markets are not controlled by the protocol after deployment and they are not upgradable. The owner of the protocol doesn’t have access to the borrower’s/lender’s funds.

### Systemic Risks

Even if this is out of scope and the protocol is fully aware of this, we cannot simply ignore the risk of having a bad borrower.

The protocol has an onboarding process that requires different documents from the borrowers, which will be assessed by lawyers and 3rd party services. The borrowers will have a process to vet the lenders, and both parties will sign a `Service Agreement` and at least one `Master Loan Agreement`.

1. People steal, scam and hack not because there isn’t a legal biding document that prevents them from doing it, the law establishes the illegality of those acts anyway, but people keep doing it because they don’t care. If the amount is large enough there’s always a chance a borrower will run with the money. Getting money back is always difficult.
2. In order to do this, borrowers will be inclined to provide fake documents. Given that the stealing opportunity is high, people will invest in high quality fakes in order to scam the protocol and lenders.
3. In the Master Loan Agreement (MLA) terminology definition from the [documentation](https://wildcat-protocol.gitbook.io/wildcat/using-wildcat/terminology#master-loan-agreement-mla) it is specified that MLA is ` offered by Wildcat as protection for lenders`, those kinds of statements invite damaged lenders to sue the protocol for compensation when they realize that Master Loan Agreement didn’t quite protected their funds from getting stolen.

### Recommendations

Consider an even more modular architecture, some of the functions are really big. Breaking them into smaller functions could help with the maintainability, readability and ease of building on top of the project.

Limit the number of tokens.

Create a repay method that can be called only by the lender to avoid using the current way of calculating `totalAssets`.

### Conclusion

Overall Wildcat is a unique and innovative project that attempts to bring undercollateralized lending into the web3 space. It was a fun project to audit. I sincerely hope this shift from `trustless` to `trustful` will succeed proving everyone that trust between two parties still has a place in web3.


### Time spent:
48 hours