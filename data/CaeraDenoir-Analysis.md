## 1. Summary of Codebase 
### 1.1 Description
Wildcat is a protocol that allows borrowers to deploy lines of credit while being undercollateralised. This is achieved by the protocol itself only allowing certain users to be borrowers.
Note: these lines of credit are called markets by the protocol.

The lenders will have an APR which will be the main motivator for them to lend the assets to the borrower. This APR is determined by the existance of a scalefactor, which increases as time passes and is the relation between the token which the lender receives and the underlying asset.

### 1.2 Roles
There are different actors involved:
- WildcatMarketControllerFactory owner: Wildcat Protocol
- Sentinel: Sentinel contract which receives a list of sanctioned addresses from an oracle, in case some addresses are added to a national sanctions list.
- Borrower: Users allowed by the Wildcat Protocol to deploy markets and have certain control over their own deployed markets. 
- Lender: Users allowed by the borrower to be able to deposit their asset to the lender created markets.

### 1.3 Borrower liability
The protocol is the one in charge of allowing certain users to become borrowers, but it is not responsible for the borrowers. They instead have made sure of the borrower legal status for it to be liable if any bad conduct were to happen.

### 1.4 Market Parameters
Every market has the following parameters:
```
  address asset;
  string namePrefix;
  string symbolPrefix;
  address borrower;
  address controller;
  address feeRecipient;
  address sentinel;
  uint128 maxTotalSupply;
  uint16 protocolFeeBips;
  uint16 annualInterestBips;
  uint16 delinquencyFeeBips;
  uint32 withdrawalBatchDuration;
  uint16 reserveRatioBips;
  uint32 delinquencyGracePeriod;
```
These parameters are what determines the fees, how much asset can a borrower take from the lender, who has the control over the market, the sentinel, etc.
### 1.4.1 Market Logic
Once a market is deployed and an authorized lender deposits a certain amount of asset, the borrower can only withdraw a % of said asset, determined by the reserveRatio. 

No matter if the borrower is using the asset deposited or not, the lender is accruing interest from the moment the market issued token is on his wallet.

If by any chance the reserves remaining are lower than the minimum stablished by the reserveRatio, the market will start a countdown to enter delinquency. This window of time is given for the borrower to be able to solve it without the market going delinquent. 

If a market goes delinquent, the borrower ends up paying a lot more of interest, since the delinquencyFee kicks in, but in exchange the lender's assets are at risk, so it is not a situation desirable by any means.

### 1.5 WildcatArchcontroller
The wildcat protocol will deploy a WildcatArchcontroller to the ethereum mainnet,which will allow it to do the following:
- Authorize or deauthorize borrowers
- Register or remove a factory of controllers
- Store the authorized factories, controllers and markets.

### 1.5.1 WildcatMarketControllerFactory
This contract is mainly used by the authorized borrowers to create new controllers and markets. 
Borrowers can use this contract for the following:
- Deploy a market controller
- Deploy a market controller and a market controlled by the deployed market controller.

The owner of WildcatArchController can use this contract to change the ProtocolFee of the markets and controllers to be deployed by the factory.
### 1.5.2 WildcatMarketController
This contract is the one that the borrower will be using to deploy and control markets.
The borrower is able to do the following:
- Authorize or deauthorize lenders
- Deploy a new market
- Change the Annual Interest of an already deployed market.
In the deployment of markets, the borrower is able to determine the following:
- asset to be lended
- Name and Prefix of the token minted when a lender deposits
- The maximum total supply of the token
- The base annual interest and the interest when the market becomes delinquent
- The reserve ratio
- The amount of time the market can have less reserves than established by the reserve ratio without becoming delinquent


### 1.5.3 WildcatMarket
This contract is the main contract in charge of deposits, withdrawals, borrows and closing itself with the right permissions.
A borrower can do the following:
- Borrow up to what the reserve rate allows
A controller can do the following:
- Close the market
A lender can do the following:
- Deposit an asset and receive the market token
- Queue a withdrawal of underlying asset in exchange of market tokens
Anyone can interact with the token issued by the contract (as in, anyone who has received a market token by any reason is able to send it to another address)

## 2. Observations

### Over-Reliant on off-chain solutions for on-chain bad behavior.
The borrower is able to be a bad actor in some edge cases since the borrower identity is known by the protocol and gives an extra security, but some simple changes can make the borrower not even try to be a bad actor.

Example 1:
A borrower can withdraw the asset and deposit the asset himself by approving himself(using a secondary address), then borrow more asset from the market and loop it until all the assets deposited on the market are on the borrower addresses.

Example 2:

A borrower can lower the annual interest rate and since both the baseRate and the delinquentRate is set by the borrower he could theoretically make the lenders accrue less interest than promised.

Example 3:
A borrower can promise a bond-like token and then call the setAnnualInterestBips() function to lower the reserveRate and borrow more asset than promised.

These examples can be solved with restricting the borrower ability to change the market so arbitrarily or by making the lenders withdrawal ability be a little bit restricted, which would not have a noticeable change on the markets themselves.

It offers a borrower-upgradable terms for the market without making sure that the lenders will not be taken advantage of.

This is done by design since in theory the borrower is someone who is trusted and in case he goes rogue it is legally liable, but it shouldn't hurt to have some obstacles on-chain in case the borrower goes rogue.

In case the borrower keys are compromised, the bad actor is able to take a lot of asset that could be prevented. 

## 3. Approach Taken
I took my time in understanding what the code aims to do, then divided the possible attacks in three different main actors:

- Bad faith borrower
- Bad faith lender
- Anon

Then, after discovering some possible attacks, went back and looked again from scratch with the new vulnerabilities perspective.

Since the protocol itself sacrifices decentralization for a more secure system, the owner of the protocol was ignored on this process as he should be the one that doesn't ever act in bad faith.

## 4. Time Spent
- Day 1: Understanding the protocol, not only the code but what is promised off-chain.
- Day 2: Looking for ways a rogue borrower can take advantage of the lenders and viceversa.
- Day 3: Looking for new ways a rogue borrower can take advantage of the lenders and viceversa from scratch  + creating the analysis report.

### Time spent:
36 hours

### Time spent:
36 hours