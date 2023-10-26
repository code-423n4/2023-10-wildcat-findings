## Analysis goal

The  Wildcat Protocol was analysed for common design flaws, such as access control, arithmetic issues, front-running, denial of service, race conditions, and protocol manipulations.  

The analysis specifically explored the following scenarios:
- Can borrowers bypass the whitelisting process?
- Can a lender claim another lender's deposits?
- Can a lender bypass whitelisting process?
- Does market responds appropriately to change in capacity? 
- Does change in reserve ratio correctly trigger delinquency?
- Is penalty tracking working correctly?
- What happens to penalty before grace tracker cools down? Can there be an overlap?
- Is there a way to bypass range of parameters defined by controller?
- Confirm if reserve ratio is linked to supply everywhere.
- Is there any way to stop updates of market calculation from happening?
- Can the market be bricked?
- Can withdrawal cycle be bypassed?
- Can sanctioned wallet access the protocol? By pass the restriction?
- Is mechanism for burning market tokens working correctly? Especially if the withdrawals are queued?
- Can sanction overrides be abused?

Further, the code was reviewed to identify possible improvements. Appendix C briefly covers the methodology.

## Protocol overview

Unlike traditional finance, lending in DeFi is enabled by overcollateralization, given the absence of underlying trust in borrowers' creditworthiness. Wildcat protocol aims to create an undercollateralized, peer-to-peer, fixed-income market  (refer Appendix A for a primer on fixed-income markets) without dictating lending terms. 

Instead, Wildcat provides mechanisms to:
- legally verify borrowers,
- apply penalty rate for borrower delinquency, and 
- apply constraints on the borrower when dropping the APR rate.

The protocol entrusts borrowers to maintain the health of the markets by providing complete control of lending terms such as capacity, rates, underlying asset, penalty terms, withdrawal cycles, and so on.

Wildcat also provides optional, on-chain agreements lending agreements, and let the judicial system resolve legal disputes. By enabling lenders to trust the judicial system and, by proxy, the borrower, the protocol opens lending to a wide array of borrowers to access capital.

Fundamentally, wildcat works by allowing borrowers to deploy market via a preset controller and then authorizing lenders to participate. Lenders can deposit money to earn interest. Market is (under) collaterized by reserving some part of the supply. Violating reserve ratio requirement causes a market to become delinquent. 

![Wildcat architecture](https://i.imgur.com/5DfbrE4.png)

Appendix B lists the user roles for this protocol.
### Onboarding borrower
Borrowers interested in raising capital using the Wildcat protocol are vetted through an offline process before being added to the whitelist on `ArchController` - the registry that tracks permissions and deployments. Borrowers are then expected to follow the terms of the service agreement.

![Borrower onboarding](https://i.imgur.com/S36Awzu.png)

If a borrower is deboarded from the protocol they are still capable of interacting with existing markets.
### Anatomy of a market
A market is created with the following parameters by a whitelisted borrower via controller.  The borrower then authorizes preferred lenders who deposit funds into the market, which can be loaned by the borrower from the protocol.

![Anatomy of a market](https://i.imgur.com/pYhWymY.png)
### Withdrawal-claim cycle
If the market is sufficiently funded, then tokens are moved to unclaimed withdrawal pool and the supply is reduced by burning market tokens. Lenders can then claim tokens at the end of withdrawal cycle. If market does not have sufficient reserves to fulfill the withdrawal, the request is queued until borrower deposits tokens to cure the delinquency.

![Withdrawal](https://i.imgur.com/UqHO8n4.png)
## Comments on code
The codebase is well-written, and sufficiently tested. The documentation is clear and concise, and it provides helpful examples and explanations. The following comments point out possible areas for improvement:

### [C-01] Consider using hooks for managing temporary parameters
To optimize deployment costs as described below in section I-02 of this report, consider using hooks to perform preconditions and reset temporary parameters. This would provide a mechanism to clean up dynamic values before they are read by deployed contracts, making the code more readable and emphasizing the lifecycle of the deployment process. It would also centralize code related to the creation of markets and controllers, making it easier to extend the codebase if needed.

This is partially done in using `WildcatMarketControllerFactory._resetTmpMarketParameters`. A more generic approach could be:

```diff
  function deployController() public returns (address controller) {
    __SNIP__
-   _tmpMarketBorrowerParameter = msg.sender;
+   _onBeforeControllerDeployment();
    __SNIP__
    if (controller.codehash != bytes32(0)) {
      revert ControllerAlreadyDeployed();
    }
    LibStoredInitCode.create2WithStoredInitCode(controllerInitCodeStorage, salt);
-   _tmpMarketBorrowerParameter = address(1);
+   _onAfterControllerDeployment();
    archController.registerController(controller);
    _deployedControllers.add(controller);
  }

+ function _onBeforeControllerDeployment() internal{
+   // Add pre-condtions for creating controllers.
+   _tmpMarketBorrowerParameter = msg.sender;
+ }
+
+ function _onAfterControllerDeployment() internal{
+   _tmpMarketBorrowerParameter = address(1);
+   // Add necessary assertions to verify deployment.
+ }
```

[📄 Source: src/WildcatMarketControllerFactory.sol#L282-L301](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L282-L301)
### [C-02] Consider aligning documentation closer to code 
The following parameter is named differently in the codebase and documentation. While both names are appropriate, using a consistent name in both locations would enhance the correlation between the specification and the code:

| **Parameter in documentation** | **Parameter in code**   |
|--------------------------------|-------------------------|
| Capacity                       | maxTotalSupply          |
| Penalty APR                    | delinquencyFee          |
| Withdrawal Cycle               | withdrawalBatchDuration |
### [C-03] Centralize state variable declarations
Consider declaring the following state variables at one place, ideally at the top of the contract in a Solidity file. This has a number of benefits, including readability, maintainability, and in some cases: gas optimization.

- `src/WildcatMarketControllerFactory.sol`
	- `_tmpMarketBorrowerParameter`

[📄 Source: src/WildcatMarketControllerFactory.sol#L244](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L244)
### [C-04] Increase test coverage
Considering this protocol is designed to be non-upgradable, it is advice to increase test coverage .

| **Ideal coverage (above 90%)**            | - `src/WildcatArchController.sol` (100%) <br> - `src/WildcatSanctionsSentinel.sol` (100%)<br> - `src/WildcatSanctionsEscrow.sol` (100%) <br> - `src/libraries/FeeMath.sol` (100%)<br> - `src/libraries/FIFOQueue.sol` (100%)<br> - `src/libraries/MathUtils.sol` (100%)<br> - `src/libraries/SafeCastLib.sol` (100%)<br> - `src/libraries/Withdrawal.sol` (100%)<br> - `src/market/WildcatMarketToken.sol` (100%)<br> - `src/market/WildcatMarketWithdrawals.sol` (100%)<br> - `src/market/WildcatMarketConfig.sol` (98%)<br> - `src/market/WildcatMarket.sol` (96%)<br> - `src/WildcatMarketController.sol` (95%)<br> - `src/market/WildcatMarketBase.sol` (92%) |
|---------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **High coverage (between 75% and 90%)**   | - `src/libraries/LibStoredInitCode.sol` (88%)<br> - `src/libraries/MarketState.sol` (87%)<br> - `src/ReentrancyGuard.sol` (80%)<br>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| **Medium coverage (between 50% and 75%)** |  -                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| **Low coverage (below 50%)**             | - `src/WildcatMarketControllerFactory.sol` (38%)<br> - `src/libraries/BoolUtils.sol` (0%)<br> - `src/libraries/Errors.sol` (0%)<br> - `src/libraries/StringQuery.sol` (0%)                                                                                                                                                                                                                                                                                                                                                                                                                                              |

At the very least, consider adding coverage for the following critical functions:
- `src/WildcatMarketControllerFactory.sol`
	- `deployController()`
	- `deployControllerAndMarket()`
- `src/libraries/MarketState.sol`
	- `totalDebts()`
### [C-05] Consider adding comments for critical components
The following functions would benefit from comments because they employ novel ways to achieve their goals or play a pivotal role in critical operations. Adding a formal specification would also help document their objective:

- `src/libraries/LibStoredInitCode.sol`
	- `deployInitCode()`
	- `create2WithStoredInitCode()`
## Centralization risks
Wildcat is a permissioned protocol, which means that it is controlled by a central authority (the protocol operator). This gives the operator a great deal of power, including the ability to add new borrowers and manage critical fees. If the operator account is compromised, it could have serious consequences for the protocol, such as preventing new borrowers from joining or contaminating the list of borrowers.

To mitigate this risk, it is recommended that the team use a multi-signature wallet to manage the operator account. A multi-signature wallet requires multiple signatures from authorized users to authorize transactions. This makes it much more difficult for an attacker to compromise the account, even if they have access to one of the signatures.
## Highlights: Innovations and best practices 

### [I-01] Innovative use of `Create2` deployments
The cost of deploying the controller and market contracts is a bottleneck for this protocol. To save gas during deployment, the team uses a clever method of letting the controller and market contracts pull their centralized configuration from the controller factory instead of passing it to them as constructor arguments. `LibStoredInitCode.create2WithStoredInitCode` efficiently creates clones of these contracts.


```solidity
__SNIP__
    function deployController() public returns (address controller) {
        if (!archController.isRegisteredBorrower(msg.sender)) {
            revert NotRegisteredBorrower();
        }
        _tmpMarketBorrowerParameter = msg.sender;
        // Salt is borrower address
        bytes32 salt = bytes32(uint256(uint160(msg.sender)));
        controller = LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt, controllerInitCodeHash);
        if (controller.codehash != bytes32(0)) {
            revert ControllerAlreadyDeployed();
        }
        LibStoredInitCode.create2WithStoredInitCode(controllerInitCodeStorage, salt);
        _tmpMarketBorrowerParameter = address(1);
        archController.registerController(controller);
        _deployedControllers.add(controller);
    }
__SNIP__
```

[📄 Source: src/WildcatMarketControllerFactory.sol#L282-L301](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L282-L301)

[📄 Source: src/libraries/LibStoredInitCode.sol#L99-L118](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/LibStoredInitCode.sol#L99-L118)
### [I-02] Modular constraints
Throughout the codebase, constraints are broken down into modular units, instead of chaining them into one long statement. This greatly improves readability and the ability to test constraints individually.

An example:

```solidity
__SNIP__
    bool hasOriginationFee = originationFeeAmount > 0;
    bool nullFeeRecipient = feeRecipient == address(0);
    bool nullOriginationFeeAsset = originationFeeAsset == address(0);
    if (
      (protocolFeeBips > 0 && nullFeeRecipient) ||
      (hasOriginationFee && nullFeeRecipient) ||
      (hasOriginationFee && nullOriginationFeeAsset)
    ) {
      revert InvalidProtocolFeeConfiguration();
    }
 __SNIP__
```

[📄 Source: /src/WildcatMarketControllerFactory.sol#L195-L217](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L195-L217)
## Appendix A: Fixed-income markets

Fixed-income markets connect borrowers and lenders, allowing borrowers to raise capital from lenders over a specified period of time. Lenders are compensated for providing capital through regular interest payments (hence the term "fixed-income" markets), but there is a risk of delinquency from the borrower, which can mean that the lender does not receive all of the money they are owed.

Delinquency is when a borrower fails to make a payment on a loan. This can happen for a variety of reasons, such as financial hardship or fraud. When a borrower is delinquent, the lender may have to take legal action to recover the money.

For protocol specific terminology [refer this page](https://wildcat-protocol.gitbook.io/wildcat/using-wildcat/terminology).
## Appendix B: User roles
### Operator 
The wallet address that  deploys `Archcontroller`. In charge of whitelisting borrowers, and setting protocol configurations.
### Borrower
Entity authorized by the protocol to deploy markets to use the credit facility.
### Lender
Entity authorized by the borrower to deposit asset in a market.  
## Appendix C:  Methodology  
Project evaluation is a two-part process: understanding and reviewing.

In the understanding phase, the project's background is deeply analyzed, including documentation, previous releases, and similar protocols. This helps to understand the project's core value, identify critical risks, assess user experiences, and evaluate security. Factors like test coverage are also considered.

In the review phase, the project's architecture is systematically assessed. Code duplication, inconsistencies, and areas for improvement are looked for. Best practices are recommended and optimizations are suggested. This comprehensive approach ensures that the project's overall structure and code quality are scrutinized for improvements.

### Time spent:
63 hours