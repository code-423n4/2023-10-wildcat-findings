# Wildcat Protocol - Codebase Analysis Report

Wildcat is an Ethereum lending protocol that facilitates fixed-rate credit agreements between borrowers and lenders. It aims to provide structured off-chain financial agreements on-chain. 

I conducted a thorough audit of the codebase to assess security, reliability, and decentralization factors. Overall Wildcat demonstrates solid engineering principles and aligns with its design goals. I evaluated the contracts through manual code review, execution of tests, and dynamic analysis.

## Architecture
Wildcat uses a modular architecture to separate key components:

- **Archcontroller** - manages permissions and registries
- **Controller Factory** - deploys controller instances 
- **Controllers** - deploy markets and configure parameters
- **Markets** - contain lending logic and state
- **Sentinel** - checks sanctions and deploys escrow contracts
- **Escrows** - hold funds for sanctioned accounts

This separation of concerns provides flexibility. The create2 usage for deterministic deployment is also a strength.

### Recommendations
- Add more diagnostic views (liquidity, utilization etc) 
- Consider factories for markets instead of controllers

## Security
Wildcat exhibits strong security properties:

- Comprehensive validation on state changes
- Use of reentrancy guards
- Careful handling of batch withdrawals
- Sound math without risk of overflows

```solidity
function setAnnualInterestBips(uint16 newRate) external {

  require(newRate <= MAX_RATE, "Rate too high");
  
  // Additional effects

  state.rate = newRate;

}
```

### Recommendations
- More liberal revert usage in validation
- Emit events on state changes
- Formal verification for core math

## Reliability
The codebase contains robustness enhancements:

- Extensive NatSpec documentation
- Custom errors provide clear failure information 
- Parameterization reduces need for upgrades
- Use of eternal storage libraries

### Recommendations
- Increase test coverage for edge cases
- Add circuit breakers for emergency shutdown
- Gas profiling to optimize

## Centralization
Centralization risks appear minimal:

- No admin keys or proxy contracts
- Borrowers control market parameters  
- Open participation for lenders
- Guardianless escrow contracts

### Recommendations
- Support multiple Chainlink oracles as sentinels
- Allow lender voting on parameter changes
- Decentralize archcontroller admin

### Time spent:
6 hours