Q[0-1]: Numerous casts increase system complexity
using numerous of uint values inside of structs to save on storage costs. However, projects must carefully weigh the realized gas savings against the additional complexity such a design decision introduces.to reduce complexity of the code, consider using non-uint256 values only when necessary. 

Q[0-2]: In WildcatMarketBase.sol::_applyWithdrawalBatchPaymentView Function name mentioned as view but this function is pure and not view so need to fix function name. WildcatMarketBase.sol#L528-L532
