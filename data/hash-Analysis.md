## Approach
Apart from the case of `borrower fails to repay what is owed`, which is handled via off-chain signed agreements, all security vulnerabilities in the in-scope contracts was taken as in-scope. Since the most complex logic of the system is present in the Market, majority of the time was spent analyzing this part. 

## Mechanism review
The implementation is a generalized, permissioned, no-collateral lending protocol where trust is generated in means outside of on-chain code. Borrowers have to be registered by the main owner while borrowers can registers lenders in their own markets. Conditions for lending is preset by the borrower but can be changed in a limited way taking care of not adversely affecting the lenders. Although the idea is `a protocol for fixed-rate, undercollateralised credit facilities`, the interests are compounded at every state update making the actual interval interest rate dependent on the number of state updates.
Withdrawals occur in batches and lenders have to wait till the expiry time of the batch to withdraw their deposited funds assuming that there are funds available to withdraw. In case there aren't enough funds for withdrawal, the burden is put on the borrower by inducing a delinquency fee.
Lenders are monitored using Chainalysis to avoid poisoning of the system. 

## Systemic risks
1. The most important trust aspect is kept off-chain. The agreements made by the participants and the judging system of the agreement will bear all the expect-able risks associated with off-chain entities.

## Centralization risks
1. Apart from the entry to the system, the `archController` owner has the ability to remove a registered borrower. But this only disallows creation of new markets and doesn't interfere with the operations of already deployed markets. 

### Time spent:
25 hours