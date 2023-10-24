`src/WildcatMarketController.sol:125`
`getAuthorizedLenders` function. Variable `_authorizedLenders` read 1 time at the beginning of the function and reading on loop. Could be move to a function variable to save a gas.

`src/WildcatMarketController.sol:182`
`updateLenderAuthorization` function has the reading `_controlledMarkets` during the loop. Better to make a new instance at the beginning of the function and read from there.

`src/WildcatMarketController.sol:394`
function `enforceParameterConstraints` could be refactored, too many arguments. Better to split a multiple functions.

`src/WildcatArchController.sol:128`
the function `getRegisteredControllerFactories` is the reading value of `_controllerFactories` to get the length and value in the loop. Better to make a copy of the storage variables inside of the function and read from there.

`src/WildcatArchController.sol:171`
the function `getRegisteredControllers` is the reading value of `_controllers` to get the length and value in the loop. Better to make a copy of the storage variables inside of the function and read from there.

`src/WildcatArchController.sol:214`
the function `getRegisteredMarkets` is the reading value of `_markets` to get the length and value in the loop. Better to make a copy of the storage variables inside of the function and read from there.

`src/market/WildcatMarketWithdrawals.sol:137`
the function `executeWithdrawal` creates a storage variable and then reads a couple of times and after that assigns value. To save gas better to move it to a variable and read from there