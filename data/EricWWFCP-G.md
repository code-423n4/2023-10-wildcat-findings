1) src/ReentrancyGuard.sol - The types for the _reentrancyGuard, _NOT_ENTERED, _ENTERED variables don't need to be uint256 as it is overkill.

```
  uint256 private _reentrancyGuard;

  uint256 private constant _NOT_ENTERED = 1;
  uint256 private constant _ENTERED = 2;
```

We can change these to uint8 or boolean type to save storage space and gas.