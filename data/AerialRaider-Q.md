wildcatMarketBase.sol 

Conditional statements can be costly in terms of gas. Minimize their use if possible. In the wildcatMarketBase.sol contract the conditional statements in modifiers like onlyBorrower and onlyController. These modifiers are relatively simple, but you can make them even more efficient. Here's an example for onlyBorrower:

modifier onlyBorrower() {
    require(msg.sender == borrower, "NotApprovedBorrower");
    _;
}

modifier onlyController() {
    require(msg.sender == borrower, "NotController");
    _;
}



instead of using: 
  modifier onlyBorrower() {
    if (msg.sender != borrower) revert NotApprovedBorrower();
    _;
  }

  modifier onlyController() {
    if (msg.sender != controller) revert NotController();
    _;
  }
