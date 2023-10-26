# [L-1] Borrower cannot close the market

        function closeMarket() external onlyController nonReentrant {

https://github.com/code-423n4/2023-10-wildcat/blob/c0e8f1b785659fe58ac43fb61e859b4da0b643d5/src/market/WildcatMarket.sol#L142

The function `closeMarket` is used when borrower would not like to sets the market APR to 0% and marks market as closed. This function can only be called by controller. However in the controller, there is no function to `closeMarket`. It means that borrower has no way to close the market.

# [L-2] Wrong multiply function

The protocolFeeRay is in ray, however the function use `bipMul`. The `bipMul` function is used to multiple two bip. The result will be wrongly rounded.

        uint256 protocolFeeRay = protocolFeeBips.bipMul(baseInterestRay);

https://github.com/code-423n4/2023-10-wildcat/blob/c0e8f1b785659fe58ac43fb61e859b4da0b643d5/src/libraries/FeeMath.sol#L46


    /**
    * @dev Multiplies two bip, rounding half up to the nearest bip
    *      see https://twitter.com/transmissions11/status/1451131036377571328
    */
    function bipMul(uint256 a, uint256 b) internal pure returns (uint256 c) {
        assembly {
        // equivalent to `require(b == 0 || a <= (type(uint256).max - HALF_BIP) / b)`
        if iszero(or(iszero(b), iszero(gt(a, div(sub(not(0), HALF_BIP), b))))) {
            // Store the Panic error signature.
            mstore(0, Panic_ErrorSelector)
            // Store the arithmetic (0x11) panic code.
            mstore(Panic_ErrorCodePointer, Panic_Arithmetic)
            // revert(abi.encodeWithSignature("Panic(uint256)", 0x11))
            revert(Error_SelectorPointer, Panic_ErrorLength)
        }

        c := div(add(mul(a, b), HALF_BIP), BIP)
        }
    }

https://github.com/code-423n4/2023-10-wildcat/blob/dfbeba095582b30491708e0be1f87a4974fcf10d/src/libraries/MathUtils.sol#L81-L99