
# WILDCAT PROTOCOL - QA REPORT.

| ID    |   Title    |
|:---:  |:---         |
| 1.  | Using an Enumerable set can cause Dos if the set grows big enough and you try to read all the sets in a non-view function.  |
| 2   | You can add names to mapping variables to improve code readability.  |
| 3.  | It’s not recommended to write in the scratch memory, these words are reserved by solidity for special purposes.    |
| 4.  | Emit events in all important functions.   |
| 5.  | Solve the Pending ToDo’s in the Code.   |
| 6.  | A return statement is not necessary if the returns variable is already declared.   |
| 7.  | It’s Recommended to use a fixed pragma version instead of a floating pragma.    |

### 1.- Using an Enumerable set can cause Dos if the set grows big enough and you try to read all the sets in a non-view function.

using an Enumerable set can cause a Dos in the contract if the set grows large enough and it’s used in a function that modifies the state of the contract, this is commented in the openzeppelin documentation and it’s something to keep in mind for future iterations of the contracts.

```solidity
/**
* @dev Return the entire set in an array
*
* WARNING: This operation will copy the entire storage to memory, which can be quite expensive. This is designed
* to mostly be used by view accessors that are queried without any gas fees. Developers should keep in mind that
* this function has an unbounded cost, and using it as part of a state-changing function may render the function
* uncallable if the set grows to a point where copying to memory consumes too much gas to fit in a block.
*/
function _values(Set storage set) private view returns (bytes32[] memory) {
    return set._values;
}
```

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L11-L14


### 2.- You can add names to mapping variables to improve code readability.

The solidity 0.8.20 lets you add names to mapping variables, it’s a good practice to use this feature in all your mappings to improve code readability.

For example, you can change this in the wildcat Market controller contract:
```solidity
mapping(address => TemporaryReserveRatio) public temporaryExcessReserveRatio;
```
To this code:

```solidity
mapping(address market => TemporaryReserveRatio) public temporaryExcessReserveRatio;
```
Code lines that can be improved with this recommendation:
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L76

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/Withdrawal.sol#L33-L34

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L13


### 3.- It’s not recommended to write in scratch memory, these words are reserved by solidity for special purposes.

Solidity reserves the first 4 words of memory for particular computations like hashing functions, memory pointer, and initial values for dynamic arrays for this reason is recommended to avoid using these slots cause this could lead to bugs in your contracts.

Code manipulating solidity reserve memory:

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L375-L385

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/LibStoredInitCode.sol#L59-L80


### 4.- Emit events in all important functions.

it’s recommended to emit events in every important function to have a register of the function called and improve the user experience on the website.

for example, you can add an event in the function that deploys a market controller:

```solidity
function deployController() public returns (address controller) {
    if (!archController.isRegisteredBorrower(msg.sender)) {
        revert NotRegisteredBorrower();
    }


    _tmpMarketBorrowerParameter = msg.sender;
    // Salt is borrower address
    bytes32 salt = bytes32(uint256(uint160(msg.sender)));
    controller = LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt,                 
    controllerInitCodeHash);
    if (controller.codehash != bytes32(0)) {
       revert ControllerAlreadyDeployed();
    }
   LibStoredInitCode.create2WithStoredInitCode(controllerInitCodeStorage, salt);
   _tmpMarketBorrowerParameter = address(1);
   archController.registerController(controller);
  _deployedControllers.add(controller);
}
```
https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L282

also add events in other functions that interact with the user.


### 5.- Solve the Pending ToDo’s in the Code.

there are some pending ToDo comments in the code that should be checked to avoid deploying code that may be shouldn’t be deployed.

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L10-L12


### 6.- A return statement is not necessary if the returns variable is already declared.
 
if a returned variable is declared in the “returns” section at the beginning of the function it’s not necessary to explicitly return that variable at the end of the function cause solidity implicity returns this variable when the execution of the function ends.

for example, in the values function of the FifoQueue library you don’t need to use the return word at the end of the function cause solidity knows that the variable “_values” should be return:

```solidity
function values(FIFOQueue storage arr) internal view returns (uint32[] memory _values) {
    uint256 startIndex = arr.startIndex;
    uint256 nextIndex = arr.nextIndex;
    uint256 len = nextIndex - startIndex;
    _values = new uint32[](len);


    for (uint256 i = 0; i < len; i++) {
        _values[i] = arr.data[startIndex + i];
    }

    return _values; 
}
```
GitHub link:

​​https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FIFOQueue.sol#L42-L53


### 7.- It’s Recommended to use a fixed pragma version instead of a floating pragma.

It’s always a good security practice to use a fixed pragma solidity for your contracts instead of a floating pragma, in this way you’re sure all your contracts adhere to a unique solidity version and it’s easier to know which compiler vulnerabilities can affect your contracts.

so it’s recommended to remove the “>=” in all your contract’s pragma lines and use the same solidity version in all your contracts.

in your contracts code change this code:
```solidity
pragma solidity >=0.8.20;
```
For this code:
```solidity
pragma solidity 0.8.20;
```
Add you some Github line as examples:

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L2

https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/BoolUtils.sol#L2
