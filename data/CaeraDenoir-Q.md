# QA Findings

These QA Findings are based on markets set on specific conditions by the borrower, allowing some bad-faith borrowers to bypass some market configs. (Not listing a full 100% borrow by a bad-faith borrower here since it is of a higher severity, even though it is based on the same principles of a bad-faith borrower)

## First QA Finding
### Borrower is able to use `setAnnualInterestBips` to skew the Reserve Ratio

The logic behind is: a borrower can deploy a market with a high Reserve ratio, then call setAnnualInterestBips to lower the ratio and be able to get access to more asset without making the market delinquent.

#### Code Involved

A function in src/WildcatMarketController.sol


```sol
function setAnnualInterestBips(
    address market,
    uint16 annualInterestBips
  ) external virtual onlyBorrower onlyControlledMarket(market) {
    // If borrower is reducing the interest rate, increase the reserve
    // ratio for the next two weeks.
    if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {
      TemporaryReserveRatio storage tmp = temporaryExcessReserveRatio[market];

      if (tmp.expiry == 0) {
        tmp.reserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips());

        // Require 90% liquidity coverage for the next 2 weeks
        WildcatMarket(market).setReserveRatioBips(9000);
      }

      tmp.expiry = uint128(block.timestamp + 2 weeks);
    }

    WildcatMarket(market).setAnnualInterestBips(annualInterestBips);
  }
```
###  Explanation
When the borrower changes the annual interest, the function checks if the rate is being reduced, and in case it is, it sets the reserve ratio to 90%.

This behavior can be exploited by a bad borrower, by attracting lenders with a very high reserve ratio, and then lowering it by using the function.

It is worth to take into consideration that the bad actor can in fact promise a high Annual Interest with a high reserve ratio, and benefit both from lowering the interest and being able to borrow more of the asset lended.

### Example
Assume a new market is deployed, Ana is an approved lender, the reserve ratio is 99%, and has a 10% Annual Interest Rate.

Ana supplies the market with 50,000 of asset.
```
Supply 50,000, Reserve Ratio 99%, In Reserves: 50,000, Reserves Required: 49,500, Annual Interest : 10%

Borrower can withdraw up to 500 asset at 10% annual interest without making the market delinquent.
 
```
Borrower then calls setAnnualInterestBips, and lowers the interest to 1%.

```
Supply 50,000, Reserve Ratio 90%, In Reserves: 50,000, Reserves Required: 45,000, Annual Interest : 1%

Borrower can withdraw up to 5000 asset at 1% annual interest without making the market delinquent.
 
```
Borrower is able to withdraw more asset at a lower rate.

### Recommendation
Don't allow setAnnualInterestBips to lower the Reserve Ratio under any circumstances.

### Severity: Low
This is a very niche situation that compromises lender assets, but there are a lot of things that must align for this to be possible, and should in theory be covered by other off-chain methods.


## Second QA Finding
### Borrower is able to use `setAnnualInterestBips` to borrow at a lower interest.

The logic behind is: a borrower can deploy a market with a high base Interest and a low delinquent interest, then borrow whatever is allowed by the Reserve Rate follower by a setAnnualInterestBips call to lower the base Interest.
#### Code Involved

A function in src/WildcatMarketController.sol


```
function setAnnualInterestBips(
    address market,
    uint16 annualInterestBips
  ) external virtual onlyBorrower onlyControlledMarket(market) {
    // If borrower is reducing the interest rate, increase the reserve
    // ratio for the next two weeks.
    if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {
      TemporaryReserveRatio storage tmp = temporaryExcessReserveRatio[market];

      if (tmp.expiry == 0) {
        tmp.reserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips());

        // Require 90% liquidity coverage for the next 2 weeks
        WildcatMarket(market).setReserveRatioBips(9000);
      }

      tmp.expiry = uint128(block.timestamp + 2 weeks);
    }

    WildcatMarket(market).setAnnualInterestBips(annualInterestBips);
  }
```
###  Explanation
When the borrower changes the annual interest while borrowing asset, it is possible for the new Annual Interest rate + Deliquency rate to be lower than the promised % for the lenders, and by being delinquent the lenders are unable to exit either.

It is worth considering that this behavior makes the market go delinquent, so it raises some flags instantly.

### Example
Assume a new market is deployed, Ana is an approved lender, the reserve ratio is 20%, and has a 30% Annual Interest Rate with a 10% Delinquent Rate.

Ana supplies the market with 50,000 of asset.
```
Supply 50,000, Reserve Ratio 20%, In Reserves: 50,000, Reserves Required: 10,000, Actual Annual Interest : 30%

Borrower: 0 asset

Market: Not Delinquent

 
```
Borrower then calls borrows the maximum amount he is allowed

```
Supply 50,000, Reserve Ratio 20%, In Reserves: 10,000, Reserves Required: 10,000,Actual Annual Interest : 30%

Borrower: 40,000 asset

Market: Not Delinquent
 
```
Borrower then calls setAnnualInterestBips, and lowers the interest to 10%.


```
Supply 50,000, Reserve Ratio 20%, In Reserves: 10,000, Reserves Required: 45,000
Actual Annual Interest : 10%(new Annual Interest) + 10%(Delinquency Rate being applied) = 20%

Borrower: 40,000 asset

Market: Delinquent
 
```


### Recommendation
There are a few ways to solve this issue. I would advise not to allow the borrower to lower the Interest if the market would enter delinquency after the function call.

### Severity: Low
This is a very niche situation that compromises lender interest, but there are a lot of things that must align for this to be possible, and should in theory be covered by other off-chain methods.
