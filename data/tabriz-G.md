
## [G-01] Move storage pointer to top of function to avoid offset calculation
We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.

```
 function executeWithdrawal(
    address accountAddress,
    uint32 expiry
  ) external nonReentrant returns (uint256) {
    if (expiry > block.timestamp) {
      revert WithdrawalBatchNotExpired();
    }
    MarketState memory state = _getUpdatedState();

    WithdrawalBatch memory batch = _withdrawalData.batches[expiry];
    AccountWithdrawalStatus storage status = _withdrawalData.accountStatuses[expiry][
      accountAddress
    ];
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L137C2-L149C7

## [G-02] Events are not indexed
The emitted events are not indexed, making off-chain scripts such as front-ends of dApps difficult to filter the events efficiently.
Recommend adding the `indexed` keyword in each event,

```
  event NewController(address borrower, address controller, string namePrefix, string symbolPrefix);
  event UpdateProtocolFeeConfiguration(
    address feeRecipient,
    uint16 protocolFeeBips,
    address originationFeeAsset,
    uint256 originationFeeAmount
  );
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L16C1-L22C5

```
  event ControllerRemoved(address controller);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L39C1-L39C47

## [G-03] Make for loop unchecked
The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is
limited to the arrays length. Even if the arrays are extremely long, it will take a massive amount of time
and gas to let the for loop overflow.

```
 for (uint256 i = 0; i < count; i++) {
      arr[i] = _authorizedLenders.at(start + i);
    }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L133C4-L135C6

```
  for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.add(lender)) {
        emit LenderAuthorized(lender);
      }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L154C3-L158C8

```
  for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.remove(lender)) {
        emit LenderDeauthorized(lender);
      }
    }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L170C3-L175C6

```
 for (uint256 i; i < markets.length; i++) {
      address market = markets[i];
      if (!_controlledMarkets.contains(market)) {
        revert NotControlledMarket();
      }
      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
    }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L183C4-L189C6

```
 for (uint256 i = 0; i < count; i++) {
      arr[i] = _controlledMarkets.at(start + i);
    }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L212C4-L214C6

```
 for (uint256 i = 0; i < count; i++) {
      arr[i] = _deployedControllers.at(start + i);
    }
  }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L146C4-L149C4

```
 for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }
  }

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L93C4-L97C1


```
 for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllerFactories.at(start + i);
    }
  }

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L136C4-L140C1

```
   for (uint256 i = 0; i < count; i++) {
      arr[i] = _controllers.at(start + i);
    }
  }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L179C2-L182C4

```
 for (uint256 i = 0; i < count; i++) {
      arr[i] = _markets.at(start + i);
    }
  }

```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L222C4-L226C1


```
   for (uint256 i = 0; i < len; i++) {
      _values[i] = arr.data[startIndex + i];
    }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L48C2-L50C6

```
 for (uint256 i = 0; i < n; i++) {
      delete arr.data[startIndex + i];
    }
    arr.startIndex = startIndex + n;
  }
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FIFOQueue.sol#L75C4-L79C4

## [G-04] Use double if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2
if statements.

```
 if (
      (protocolFeeBips > 0 && nullFeeRecipient) ||
      (hasOriginationFee && nullFeeRecipient) ||
      (hasOriginationFee && nullOriginationFeeAsset)
    ) {
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L204C4-L208C8



## [G-05] Use hardcode address instead address(this)
Instead of using address(this) , it is more gas-efficient to pre-calculate and use the hardcoded
address . Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824

```
 parameters.controller = address(this);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L243 


```
 address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
          accountAddress,
          borrower,
          address(this)
        );
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L172C8-L176C11

```
   emit Transfer(msg.sender, address(this), amount);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L91C2-L91C54

```
   asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L154C4-L154C83

```
           bytes1(0xff),
                address(this),
                keccak256(abi.encode(borrower, account, asset)),
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L76C6-L78C65

