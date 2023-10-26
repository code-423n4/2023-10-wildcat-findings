## \[G‑01\] Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L18

## \[G-02\] Use bytes32 rather than string/bytes.

If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable size.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L20C3-L21C23

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L372C4-L373C31

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L24

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L57C1-L60C24

## \[G-03\] Use assembly to validate `msg.sender`

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L302

## \[G-04\] uint8 is not always cheaper than uint256

The EVM only operates on 32 bytes/ 256 bits at a time. This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L54

## \[G-05\] For `uint` use `!= 0` instead of `> 0`

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L79

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L170

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L472

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L112

## \[G-06\] Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry’s script.sol and solmate’s `LibRlp.sol` contracts can help achieve this.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L170

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L154

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L22

## <ins>\[G-07\] Events are not indexed</ins>

The emitted events are not indexed, making off-chain scripts such as front-ends of dApps difficult to filter the events efficiently.

Recommend adding the `indexed` keyword in each event,

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L30C2-L36C43

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L39

## \[G‑08\] Using `bool`s for storage incurs overhead

```
// Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```

Use `uint256(1)` and `uint256(2)` for true/false to avoid a Gwarmaccess (**<ins>100 gas</ins>**) for the extra SLOAD, and to avoid Gsset (**20000 gas**) when changing from `false` to `true`, after having been `true` in the past.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L27

## \[G-10\] Make fewer external calls.

Every call to an external contract costs a decent amount of gas. For optimization of gas usage, It’s better to call one function and have it return all the data you need rather than calling a separate function for every piece of data. This might go against the best coding practices for other languages, but solidity is special.

## \[G-11\] Amounts should be checked for 0 before calling a transfer

It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```

In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```
_transfer(from, to, amount);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L54

```
emit Transfer(from, to, amount);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L81

```
IERC20(asset).transfer(account, amount);
```

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L38

## \[G-12\] Caching global variables is expensive than using the variable itself

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L17