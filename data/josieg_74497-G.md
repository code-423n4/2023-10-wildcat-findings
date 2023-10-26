[Gas Optimization – 1]    ++ costs less than i++. Also use unchecked {} to increment index in for loop
File Path: 2023-10-wildcat/src/WildcatMarketController.sol
Lines: 133 - 135

Description:
for (uint256 i = 0; i < count; ++i) {}; should be replaced with for (uint256 i; i < count; ++i) {};

Recommendation:
++i costs 5 gas less than i++
use ++i

use unchecked increment index to save gas further
Replaced line 133-135 with
for (uint256 i; i < count;) {
arr[i] = _authorizedLenders.at(start + i);
unchecked {
++i;
}
};


[Gas Optimization – 2]     Cache line No 81 after Line No. 86
File Path: 2023-10-wildcat/src/market/WildcatMarketWithdrawals.sol
Lines: 77 - 86

Description:
function queueWithdrawal(uint256 amount) external nonReentrant {
MarketState memory state = _getUpdatedState();

// Cache account data and revert if not authorized to withdraw.

//@audit cache this 'account' variable after line No. 86 just incase line No. 85 reverts
Account memory account = _getAccountWithRole(msg.sender, AuthRole.WithdrawOnly);

uint104 scaledAmount = state.scaleAmount(amount).toUint104();
if (scaledAmount == 0) {
revert NullBurnAmount();
}

Recommendation:
move Line No. 81 after Line No. 68

function queueWithdrawal(uint256 amount) external nonReentrant {
MarketState memory state = _getUpdatedState();

uint104 scaledAmount = state.scaleAmount(amount).toUint104();
if (scaledAmount == 0) {
revert NullBurnAmount();
}

// Cache account data and revert if not authorized to withdraw.
Account memory account = _getAccountWithRole(msg.sender, AuthRole.WithdrawOnly);
..
}