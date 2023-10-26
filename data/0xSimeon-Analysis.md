### Any comments for the judge to contextualize your findings
Context was added where it was relevant to aid the judges. 

###  Approach taken in evaluating the protocol

Since this audit was focused on the Wildcat protocol, here are the steps taken to complete this audit. 

1. We began with the Wildcat [Manifesto](https://medium.com/@wildcatprotocol/the-wildcat-manifesto-db23d4b9484d) before moving to the [ whitepaper](https://github.com/wildcat-finance/wildcat-whitepaper/blob/main/whitepaper_v0.2.pdf) and gitbook. 
2.  Started to read the codebase high-level, starting with the archController and its role in the protocol. Next, We moved on to the controllerFactories and how controllers can be deployed, borrowers and sentinel amongst other components of the system. 
4.  With the docs in mind, We started to read each contract and examine if they match their supposed functionalities, functions calls (depending on how they are chained) do what they are supposed to do. 
5. When in doubt or confused, We consult the docs or ask in the discord channel dedicated for the audit. The sponsor and other wardens were kind enough to answer my questions.   
6.  Next, We evaluate the contracts's logic, following the chained internal functions. e.g: How only a registered Borrower can deploy a new controller but the controller creation can only be done via a controllerFactory. 
7.  Over the course of the above processes, I took notes of our observations and possible attack vectors/paths. 
8. Last but not least, we review the notes and examine observations/attack vectors with more depth. 

### Observations 
- The Wildcat protocol engineering team did a good work with codebase. we encountered the usage of the concept of using temporary storage during deployment of controllers.  
- Integration with ChainAnalysis follows best practice and it checks out. 
- The wildcat team violoted some of the DRY principles. for instance, here in https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatArchController.sol#L85

```
 function getRegisteredBorrowers(
    uint256 start,
    uint256 end
  ) external view returns (address[] memory arr) {
    uint256 len = _borrowers.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }
  }
```

similiar code block is observed on other places in the codebase. it could be refactored and the array is passed to the function and return some data. 


- Lastly, while the codebase don't have full test coverage, we confirmed some components were working as expected by writing additional tests like this one: 
```
    function test_deployController() public {
        vm.prank(archController.owner());
        archController.registerControllerFactory(address(controllerFactory));
        bool success = archController.isRegisteredControllerFactory(address(controllerFactory));
        // console.log("Factory added: ", success);
        // vm.prank(address(controllerFactory)); 
        address newBorrower = makeAddr("newBorrower");
        archController.registerBorrower(newBorrower); 
        vm.prank(newBorrower);
        address addressForController = controllerFactory.deployController();
        console.log(addressForController);
    }
```


### Architecture recommendations

- Minor but, error handling is good but it could be better if it includes the contract where the error originated from. e.g `error MarketToken__InvalidInputArrays(); or ControllerFactory__CallerMustBeBorrower()` instead of generic `error InvalidInputArrays();`
	

### Systemic and/or Centralization risks
The protocol is not designed to be decentralized have pose some centralization risks. 


### Time spent:
40 hours.




### Time spent:
30 hours