Low[0-1]:Function Calls in Loop Could Lead to Denial of Service

Impact:

In WildcatMarketController.sol::updateLenderAuthorization 
and WildcatMarketController.sol::deauthorizeLenders can lead to Dos attack a moment may come when lenders.length be big enough
so more gas is required than it is allocated to record one block.

Recommended Mitigation Steps:

To avoid this attack makeing a require() check in WildcatMarketController.sol::updateLenderAuthorization 
and WildcatMarketController.sol::deauthorizeLenders.
that the lenders.length does not exceed the maximum number of lenders in one bloch(maxlengthlimitation).
 WildcatMarketController.sol#L169-L190

       +     require(
                 lenders.length<maxlengthlimitation,
                " The lenders.length must be less than maxlengthlimitation" 
                 );



Q[0-1]: Numerous casts increase system complexity
using numerous of uint values inside of structs to save on storage costs. However, projects must carefully weigh the realized gas savings against the additional complexity such a design decision introduces.to reduce complexity of the code, consider using non-uint256 values only when necessary. 

Q[0-2]: In WildcatMarketBase.sol::_applyWithdrawalBatchPaymentView Function name mentioned as view but this function is pure and not view so need to fix function name. WildcatMarketBase.sol#L528-L532
