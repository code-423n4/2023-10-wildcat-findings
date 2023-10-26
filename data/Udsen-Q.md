## 1. `WildcatMarketControllerFactory.deployController` FUNCTION DOES NOT CHECK WHETHER `_tmpMarketBorrowerParameter` IS IN THE CORRECT STATE BEFORE THE `WildcatMarketController` CONTRACT DEPLOYMENT

The `WildcatMarketControllerFactory.deployController` function is used to deploy a `WildcatMarketController` contract using the `create2`. Before the contrat deplployment the `_tmpMarketBorrowerParameter` state variable is updated as follows:

    _tmpMarketBorrowerParameter = msg.sender;

Once the `WildcatMarketController` contract is deployed the `_tmpMarketBorrowerParameter` state variable is reset to `address(1)` as shown below:

    LibStoredInitCode.create2WithStoredInitCode(controllerInitCodeStorage, salt);
    _tmpMarketBorrowerParameter = address(1);

The issue here is before the `_tmpMarketBorrowerParameter` is set to `msg.sender` in the `WildcatMarketControllerFactory.deployController` function before the `WildcatMarketController` contract deployment there is no parameter validation to check whether `WildcatMarketController == address(1)`.

Hence it is recommended to add the following `require` statement before the ` _tmpMarketBorrowerParameter = msg.sender` assignment to ensure that the `_tmpMarketBorrowerParameter` state variable is in the correct state.

    require(_tmpMarketBorrowerParameter == address(1), "State variable not in the correct state");
    _tmpMarketBorrowerParameter = msg.sender;

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L286
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L298
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L244

## 2. THE PRE-CALCULATED `WildcatMarketController` CONTRACT ADDRESS AND ACTUALLY DEPLOYED ADDRESS ARE NOT CHECKED FOR EQUALITY IN THE `WildcatMarketControllerFactory.deployController` FUNCTION

The `WildcatMarketControllerFactory.deployController` function is used to deploy a `WildcatMarketController` contract using the `create2`.

The `deployController` function initially calculates the address of the `WildcatMarketController` contract using `create2` as shown below:

    controller = LibStoredInitCode.calculateCreate2Address(
      ownCreate2Prefix,
      salt,
      controllerInitCodeHash
    ); 

Subsequently the `WildcatMarketController` contract is deployed using the `LibStoredInitCode.create2WithStoredInitCode` as shown below:

    LibStoredInitCode.create2WithStoredInitCode(controllerInitCodeStorage, salt);

The `LibStoredInitCode.create2WithStoredInitCode` function returns the address of the deployed `WildcatMarketController` contract. But this returned address is not checked against the previously calculated `controller` address. 

Similar issue is found in the `WildMarketController.deployMarket` function while deploying `WildcatMarket` contracts.

Hence it is recommended to implement the following `require` statement to check whether the `pre-cacluated` `controller` address is equal to the actually deployed `WildcatMarketController` contract address.

    address deployedController = LibStoredInitCode.create2WithStoredInitCode(controllerInitCodeStorage, salt);
    require(controller == deployedController, "Calculated and Deployed Contract Addresses are Different");

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L289-L293
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L297
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol#L99-L104
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L351-L354

## 3. THERE IS NO INPUT VALIDATION CHECK ON `address(0)` FOR THE `lender` ADDRESS WHICH IS ADDED TO THE `_authorizedLenders` ADDRESSSET IN THE `WildcatMarketController.authorizeLenders` FUNCTION

The `WildcatMarketController.authorizeLenders` function is used to authorize a new set of `lenders` to the `WildcatMarketController`. 

Each of the lenders given in the `lenders` address array are added to teh `_authorizedLenders` AddressSet as shown below:

    for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      if (_authorizedLenders.add(lender)) {
        emit LenderAuthorized(lender);
      }
    }

But the issue here is that the `lender` is not checked for `address(0)` before adding to the `_authorizedLenders` AddressSet. 

Hence it is recommended to add the following `require` statement inside the `for` loop of the `WildcatMarketController.authorizeLenders` function to ensure address(0) is not added as a `lender` to the `_authorizedLenders` AddressSet.

    for (uint256 i = 0; i < lenders.length; i++) {
      address lender = lenders[i];
      require(lender != address(0))
      if (_authorizedLenders.add(lender)) {
        emit LenderAuthorized(lender);
      }
    }

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L154-L159

## 4. THE `MarketState.pendingWithdrawalExpiry` IS DEFINED AS A `uint32` AND AS A RESULT WILL LIMIT THE TIME DURATION FOR WHICH THE PROTOCOL CAN EXECUTE WITHDRAWAL REQUESTS BY LENDERS WITHOUT REVERT

The `MarketState.pendingWithdrawalExpiry` is declared as an `uint32` datatype. The maximum `block.timestamp` value a `uint32` can hold is upto `Tuesday, February 7, 2106, 06:28:15 AM UTC`.

The `WildcatMarketWithdrawals.queueWithdrawal` is used to create a withdrawal request for a lender and its function execution `state.pendingWithdrawalExpiry` is updated as follows, if there is no pending withdrawal batch.

    if (state.pendingWithdrawalExpiry == 0) {
      state.pendingWithdrawalExpiry = uint32(block.timestamp + withdrawalBatchDuration);
      emit WithdrawalBatchCreated(state.pendingWithdrawalExpiry);
    }

Hence as a result after the `Tuesday, February 7, 2106, 06:28:15 AM UTC` date the withdrawal request for a lender will not able to be executed since the `uint32` datatype will overflow and the withdrawal request transaction will revert (Given there is no pending withdrawal batch). 

Hence it is recommended to update the `state.pendingWithdrawalExpiry` variable to a higher datatype such as `uint128` to resolve the above limitation related to time duration for which this protocol is valid for.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol94-L97

## 5. IT IS RECOMMENDED TO CHECK FOR THE EQUALITY BETWEEN NEW VALUE AND EXISTING VALUE OF `annualInterestBips` BEFORE CHANGING THE STATE IN THE `WildcatMarketController.setAnnualInterestBips` FUNCTION

The `WildcatMarketController.setAnnualInterestBips` function is used to modify the interest rate for a market.

There is no logical check to verify if the new `annualInterestBips` value passed in is equal to the existing value returned from `WildcatMarket(market).annualInterestBips()` call. Even if the new value is equal to the existing value still the following function will be executed.

    WildcatMarket(market).setAnnualInterestBips(annualInterestBips);

This will still update the `annualInterestBips` to the same value. Hence it is recommended to add a logic in the `setAnnualInterestBips` function to `return` if the new value of `annualInterestBips` is equal to the existing `annualInterestBips` value.

The recommended `if` conditional statement is shown below:

    if(annualInterestBips == WildcatMarket(market).annualInterestBips()){
        return;
    } else if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {
        ...
    }

This will ensure if the new value of `annualInterestBips` is equal to the existing `annualInterestBips` value, then the function will revert without any state change.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L468-L488

## 6. RECOMMENDED TO ADD NATSPEC COMMENTS TO THE SMART CONTRACTS IN THE `wildcat` PROTOCOL

It is recommended to add comprehensive `Natspec` comments to the smart contracts in the `wildcat` protocol. This is because most of the smart contracts in the project has general `Natspec` comments for each of its functions and does not have any `definition` on the `input variables` or `return values` and any required `intrusctions` for the `devs` as well.

By adding proper `Natspec` comments will make the project easy to understand and work on for both developers and auditors.

This issue is found in all the smart contracts found in this project.