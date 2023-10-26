## \[L-01\] Insufficient coverage

Description
The test coverage rate of the project is ~90%. Testing all functions is best practice in terms of security criteria.

Due to its capacity, test coverage is expected to be 100%.

## \[L-02\] Stop Using Solidity's transfer() Now

**Proof Of Concept**
https://consensys.io/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

```
_transfer(from, to, amount);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L54

## \[L-03\] One year is not always 365 days

```
uint256 constant SECONDS_IN_365_DAYS = 365 days;
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L14

## \[N-01\] NatSpec comments should be increased in contracts

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## \[N-02\] Function writing that does not comply with the Solidity Style Guide

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## \[N‑03\] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist.

## \[N-04\] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

## \[N-05\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## \[N-06\] Use the delete keyword instead of assigning a value of 0

Using the ‘delete’ keyword instead of assigning a ‘0’ value is a detailed optimization that increases code readability and audit quality, and clearly indicates the intent.

Other hand, if use delete instead 0 value assign , it will be gas saved.

## \[N-07\] Function writing that does not comply with the Solidity Style Guide

Context
All Contracts

Description
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

## \[N-08\] Assembly Codes Specific – Should Have Comments

> Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.
> This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.
> Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.



## \[N-9\] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

```
uint256 constant BIP = 1e4;
uint256 constant HALF_BIP = 0.5e4;

uint256 constant RAY = 1e27;
uint256 constant HALF_RAY = 0.5e27;

uint256 constant BIP_RAY_RATIO = 1e23;
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L6C1-L14C49

## \[N-10\] Don’t use numbers without explanation, it’s not a good code practice. It’s possible to use constants.

```
uint256 constant Panic_CompilerPanic = 0x00;
uint256 constant Panic_AssertFalse = 0x01;
uint256 constant Panic_Arithmetic = 0x11;
uint256 constant Panic_DivideByZero = 0x12;
uint256 constant Panic_InvalidEnumValue = 0x21;
uint256 constant Panic_InvalidStorageByteArray = 0x22;
uint256 constant Panic_EmptyArrayPop = 0x31;
uint256 constant Panic_ArrayOutOfBounds = 0x32;
uint256 constant Panic_MemoryTooLarge = 0x41;
uint256 constant Panic_UninitializedFunctionPointer = 0x51;


uint256 constant Panic_ErrorSelector = 0x4e487b71;
uint256 constant Panic_ErrorCodePointer = 0x20;
uint256 constant Panic_ErrorLength = 0x24;
uint256 constant Error_SelectorPointer = 0x1c;
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/Errors.sol#L4C2-L18C47

## \[S-01\] Generate perfect code headers every time

Description:
I recommend using header for Solidity code layout and readability

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```