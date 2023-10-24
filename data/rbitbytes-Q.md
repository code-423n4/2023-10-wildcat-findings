solady’s SafeTransferLib  does not verify if the token has code.

This responsibility is delegated to the caller.

@dev Note that none of the functions in this library check that a token has code at all! That responsibility is delegated to the caller.

There are 5 instances of this, that are not included in the bot-report.

WildcatMarket.sol (#L60)

asset.safeTransferFrom(msg.sender, address(this), amount);

WildcatMarket.sol (#L107)

asset.safeTransfer(feeRecipient, withdrawableFees);

WildcatMarket.sol (#L129)

asset.safeTransfer(feeRecipient, withdrawableFees);

WildcatMarket.sol (#L154)

asset.safeTransferFrom(borrower, address(this), totalDebts – currentlyHeld);

WildcatMarket.sol (#L157)

asset.safeTransfer(borrower, currentlyHeld - totalDebts);