## Overview
Totally spent over 20 hours on this project, I am happy to acknowledge that the overall codebase and documentation are high quality, clear to read, and easy to understand. Architecture is also clearly described in the whitepaper. I would give the codebase 8/10, it would be good to have more comments on each function. For the documentation also 8/10 for the same reason, however, most functions can be intuitively understood. 

## Findings
The main issues that I mostly focus on: are the loss of funds due to ERC20 rebasing tokens as underlying assets and lack of deployment success validation.
Also, wasn't sure about the severity of "`releaseEscrow()` does not return any value" so submitted it as medium because it is most likely going to be used by other smart contracts that would need the validation functionality.

## Methodology
In order to evaluate the codebase, I first checked the whitepaper, then the documentation, and finally, what was audited already by aleph_v (src/libraries/, src/market/, src/ReentrancyGuard.sol). After that, I first started line-by-line auditing everything that was not in the scope before, and then, the rest.


## Centralization risks:
The main centralization risk is split between the Archcontroller Owner and Borrowers. Even though, the Wildcat team claims them to be trusted roles, the issue still has to be acknowledged.
Another centralization risk would be in Chainalysis oracle, however, the issue is mitigated with the `overrideSanction` function


### Time spent:
21 hours