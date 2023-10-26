## Wrong comment
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L390-L398

## Solidity code optimization

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L182-L190

Change code like this:
```
/**
 * @dev Update lender authorization for a set of markets to the current
 *      status.
 */
function updateLenderAuthorization(address lender, address[] memory markets) external {
    require(_authorizedLenders.contains(lender), "Not an authorized lender");

    for (uint256 i = 0; i < markets.length; i++) {
        address market = markets[i];
        require(_controlledMarkets.contains(market), "Not a controlled market");
        WildcatMarket(market).updateAccountAuthorization(lender, true);
    }
}
```
