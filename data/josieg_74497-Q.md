[Low-01]    Variable checks to avoid unnecessary code execution
File Path: src/market/WildcatMarket.sol
function depositUpTo(
    uint256 amount
  ) public virtual nonReentrant returns (uint256 /* actualAmount */) {…}

    a). ===>  Line No. 45 
    //@audit add check for amount == 0 or any unallowed amount before proceeding, and revert.

    if(amount == 0)
	revert amountNotAllowed;

    if(amount >= unallowedAmount)
	revert unallowedAmount;

    b). ===>  Line No. 60
         //@audit should check before and after balance make sure ‘amount' is transferred

        // Transfer deposit from caller
        asset.safeTransferFrom(msg.sender, address(this), amount);

Alternate recommendation for a). Line No.45:
function deposit(uint256 amount) external virtual {
    ===>  Line No. 86 - 91
    if(amount == 0)
	revert amountNotAllowed;
    if(amount >= someUnallowedAmount)
	revert someUnallowedAmount;

    uint256 actualAmount = depositUpTo(amount);
    if (amount != actualAmount) {
      revert MaxSupplyExceeded();
    }
  }

[Low-02]    Unnecessary no state change to update. 
File Path: src/market/WildcatMarket.sol
Line No. 128   Unnecessary no state change to update.

  function borrow(uint256 amount) external onlyBorrower nonReentrant {
    MarketState memory state = _getUpdatedState();
    if (state.isClosed) {
      revert BorrowFromClosedMarket();
    }

    a). ===> Line No. 124 //@audit don’t need to cache this
    uint256 borrowable = state.borrowableAssets(totalAssets());
    if (amount > borrowable) {
      revert BorrowAmountTooHigh();
    }
    replace with:
    if(amount > state.borrowableAssets(totalAssets())) {
        revert BorrowAmountTooHigh();
    }

    b). ===> Line No. 128  
    _writeState(state);   //@audit remove this line
    asset.safeTransfer(msg.sender, amount);

     c).  ===>  Line No. 130 //@audit this could be renamed, to ‘Borrowed(amount)’
    emit Borrow(amount);
  }

[Low – 3]  Refactor code to avoid unnecessary steps
File Path: 2023-10-wildcat/src/market/WildcatMarketWithdrawals.sol
Line No. 32

Description:
//@audit first just check if expiry is greater than 0 if not why bother checking if expiry == expiredBatchExpiry?

Recommendation:
//@auddit replace with:
if(expiry > 0){
   //no need to execute next line if expiry is > 0
   if(expiry == expiredBatchExpiry) {
   }
}

[Low – 4]    block.timestamp is can be manipulated
File Path: 2023-10-wildcat/src/market/WildcatMarketWithdrawals.sol
Line: 49

Description:
Also on line 141
A block.timestamp value can be manipulated by a dishonest miner. Usually to an extent of few seconds on Ethereum, or generally few percent of the block time on any EVM-compatible PoW network.

Recommendation:
not to use block.timestamp.

[Low – 5]     Unused return value
File Path: 2023-10-wildcat/src/WildcatMarketControllerFactory.sol
Lines: 282 - 301

Description:
also found at File Path: 2023-10-wildcat/src/WildcatMarketControllerFactory.sol
Lines: 300

The return value from an external call is not being retained in either a local or state variable.

Recommendation:
The issue you've identified is that the return value of the `deployController()` function call on line 294 is not stored or used. This could potentially lead to inefficiencies or even vulnerabilities if the return value is important.

If the return value of `deployController()` is important and needs to be used later in the code, it should be stored in a variable. Here's how you can do it:

```solidity
controller = deployController();
```
If the return value is not important and the function is only being called for its side effects, you can ignore the return value. However, it's generally a good practice to handle return values to avoid potential issues.

[Low – 6]    block.timestamp usage is not safe
File Path: 2023-10-wildcat/src/market/WildcatMarketBase.sol
Line: 378

Description:
The utilization of `block.timestamp` presents a potential security risk as it can be manipulated by miners.

Recommendation:
It is advisable to refrain from utilizing block.timestamp in smart contracts

[Low – 7]    Reentrancy events
File Path: 2023-10-wildcat/src/market/WildcatMarketBase.sol
Lines: 163 - 169

Description:
File Path: 2023-10-wildcat/src/market/WildcatMarketBase.sol
Lines: 172 -176

File Path: 2023-10-wildcat/src/market/WildcatMarketBase.sol
Lines: 179 - 183

Reentrancies:
(https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy) that allow manipulation of the order or value of events.

Recommendation:
Apply the [`check-effects-interactions` pattern](https://docs.soliditylang.org/en/latest/security-considerations.html#re-entrancy).

[Low – 8]   Return value not stored in local or state variable and is not returned
File Path: 2023-10-wildcat/src/market/WildcatMarketBase.sol
Lines: 399 - 442

Description:
The return value of an external call is not stored in a local or state variable.

Recommendation:
Remove returns(...), or store value in local or state variable and return as intended in function.

[Low – 9]     Missing zero address check
File Path: 2023-10-wildcat/src/WildcatMarketControllerFactory.sol
Lines: 72 - 104

Description:
An omission in the verification process for null addresses (address(0)).

Recommendation:
Check for zero address validation.


[Low – 10]     Magic Numbers:  ‘9000’
File Path: 2023-10-wildcat/src/WildcatMarketController.sol
Line: 481

Description:
The vulnerability titled "Magic Number 9000" refers to the hard-coded value of 9000 on line 481. This line of code sets the reserve ratio to 90% for the next two weeks whenever the interest rate is reduced. The issue with using a magic number like this is that it lacks context and can make the code harder to understand and maintain. If the logic or business rules of the contract were to change, this magic number would need to be identified and updated, which could be error-prone. It's generally a better practice to use a named constant for such values to provide more context and improve maintainability.

Recommendation:
To resolve this issue, you should replace the magic number with a named constant. This will make the code more readable and maintainable. Here's how you can do it:

1. At the top of your contract, declare a new constant:

```solidity
uint16 private constant TEMPORARY_RESERVE_RATIO_BIPS = 9000;
```

2. Replace the magic number in line 481 with the new constant:

```solidity
WildcatMarket(market).setReserveRatioBips(TEMPORARY_RESERVE_RATIO_BIPS);
```

This way, if the value needs to be changed in the future, you can do it in one place, and the purpose of the value is clear to anyone reading the code.

[Low – 11]     Floating Pragma ^0.8.20
File Path: 2023-10-wildcat/src/interfaces/IWildcatArchController.sol
Line: 2

Description:
It is considered best practice to pick one compiler version and stick with it. With a floating pragma, contracts may accidentally be deployed using an outdated or problematic compiler version which can cause bugs, putting your smart contract's security in jeopardy. For open-source projects, the pragma also tells developers which version to use, should they deploy your contract. The chosen compiler version should be thoroughly tested and considered for known bugs.

Recommendation:
The exception in which it is acceptable to use a floating pragma, is in the case of libraries and packages. Otherwise, developers would need to manually update the pragma to compile locally.

Source:
https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/floating-pragma.md#floating-pragma
https://swcregistry.io/docs/SWC-103
https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/