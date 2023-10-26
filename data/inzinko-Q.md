## OpenZepplin Enumerable set Might Not Work as intended 

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L11C3-L14C50

The OpenZepplin Eneumerable set does not not mantain order, which means if the protocol for what ever reason is relying on the order of the sets in the future, to find out maybe which borrowers, markets or controllers were added to the protocol first, it may not be particularly accurate, since the main usage of the Enumerable Set is to check if a particular element exists in the Sets, This makes the impact to the protocol operations low. 

```solidity
  EnumerableSet.AddressSet internal _markets;
  EnumerableSet.AddressSet internal _controllerFactories;
  EnumerableSet.AddressSet internal _borrowers;
  EnumerableSet.AddressSet internal _controllers;
```

More information about the Enumerable Set are as follows 

They Consume a lot of Gas if the data Grows higher
The library uses the Solidity array under the hood so they is a limit on how big they can be.

## Borrowers can avoid paying a high fee for a period of time

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L468C3-L489C1

A borrower that knows all lenders in their market and their behaviours, can avoid paying a high intrest fee for two weeks, the borrower reduces the intrest rate, then of course the 90% reserve ration kicks in, all in their plan through having the 90% limit they return the amount, and set the intrest rate to the lowest it can go, they can do this as they know the lender is a big spender that does not notice the change in intrest fee, they can do this preodically as they want to avoid the initial fee. 

```solidity
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

## Lenders should be checked before being added by the Borrower to avoid protocol contamination

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L153C2-L160C4

```solidity
 function authorizeLenders(address[] memory lenders) external onlyBorrower {
    for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.add(lender)) {
        emit LenderAuthorized(lender);
      }
    }
  }
```

A check for if the Lender is already a wanted person around the world, which resulted in them being sanctioned should be made when lenders are being added to a market, which will reduce the chances of contamination with other market participants and also the borrower themselves, and prevent unwanted authorities situations should be implemented in the authorizelenders function, a simple check to the sentinel contract to check the chainalysis oracle should mitigate the risk. Only the already made implementation in the protocol that transfers the user funds to an escrow, should only be made to do work if the Lender is sanctioned when they have already been added and already earning interests in markets
