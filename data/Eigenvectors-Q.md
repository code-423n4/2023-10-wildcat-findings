Link: https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketController.sol#L291-L360

The **`deployMarket`** function lacks validation checks for the **`asset`** and **`maxTotalSupply`** parameters. This oversight allows the potential deployment of markets with a zero-address asset and a **`maxTotalSupply`** of zero.