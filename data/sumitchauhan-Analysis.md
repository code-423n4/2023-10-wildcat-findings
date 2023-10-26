Risk rating-
High Risk

Title-
Unchecked transfer(return value of transfer call is not checked)

Links to affected code-
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol 

Line number-
38

Vulnerability details-
Impact
The return value of an external transfer call is not checked.

Proof of Concept
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol 
Line 38

Tools Used
Slither, C4udit, soliditycheck, Olympix, ZAN

Recommended Mitigation Steps
Use SafeERC20, or ensure that the transfer return value is checked.



### Time spent:
6 hours