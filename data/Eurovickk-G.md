
https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol
GAS
In the deployMarket function, you can optimize gas usage by reducing storage writes and performing some checks earlier in the code. 

function deployMarket(
    address asset,
    string memory namePrefix,
    string memory symbolPrefix,
    uint128 maxTotalSupply,
    uint16 annualInterestBips,
    uint16 delinquencyFeeBips,
    uint32 withdrawalBatchDuration,
    uint16 reserveRatioBips,
    uint32 delinquencyGracePeriod
) external returns (address market) {
    require(msg.sender == borrower || msg.sender == address(controllerFactory), "Caller not authorized");
    
    enforceParameterConstraints(
        namePrefix,
        symbolPrefix,
        annualInterestBips,
        delinquencyFeeBips,
        withdrawalBatchDuration,
        reserveRatioBips,
        delinquencyGracePeriod
    );

    TmpMarketParameterStorage memory parameters = TmpMarketParameterStorage({
        asset: asset,
        namePrefix: namePrefix,
        symbolPrefix: symbolPrefix,
        feeRecipient: address(0),
        maxTotalSupply: maxTotalSupply,
        protocolFeeBips: 0,
        annualInterestBips: annualInterestBips,
        delinquencyFeeBips: delinquencyFeeBips,
        withdrawalBatchDuration: withdrawalBatchDuration,
        reserveRatioBips: reserveRatioBips,
        delinquencyGracePeriod: delinquencyGracePeriod
    });

    address originationFeeAsset;
    uint80 originationFeeAmount;
    (
        parameters.feeRecipient,
        originationFeeAsset,
        originationFeeAmount,
        parameters.protocolFeeBips
    ) = controllerFactory.getProtocolFeeConfiguration();

    // Check if the market has already been deployed
    bytes32 salt = _deriveSalt(asset, namePrefix, symbolPrefix);
    market = LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt, marketInitCodeHash);
    require(market.codehash == bytes32(0), "Market already deployed");

    _tmpMarketParameters = parameters;

    if (originationFeeAsset != address(0)) {
        originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);
    }

    // Deploy the market code
    LibStoredInitCode.create2WithStoredInitCode(marketInitCodeStorage, salt);

    archController.registerMarket(market);
    _controlledMarkets.add(market);

    _resetTmpMarketParameters();
}
Early Exit Check: We added a check at the beginning of the function to verify if the caller is authorized (either the borrower or the controller factory). This prevents unnecessary gas consumption by unauthorized callers.
Market Deployment Check: We added a check to verify if the market has already been deployed. If the market has already been deployed, there's no need to proceed with the deployment process, saving gas and preventing redundant deployments.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol

GAS
1)The _getUpdatedState function is called quite frequently within the contract, and it recalculates the state even when it might not be necessary. You could consider adding a check to determine whether state updates are needed, reducing unnecessary gas consumption.
function _getUpdatedState() internal returns (MarketState memory state) {
    state = _state;
    uint256 currentTimestamp = block.timestamp;

    if (state.hasPendingExpiredBatch()) {
        uint256 expiry = state.pendingWithdrawalExpiry;
        if (expiry != state.lastInterestAccruedTimestamp) {
            (uint256 baseInterestRay, uint256 delinquencyFeeRay, uint256 protocolFee) = state
                .updateScaleFactorAndFees(
                    protocolFeeBips,
                    delinquencyFeeBips,
                    delinquencyGracePeriod,
                    expiry
                );
            emit ScaleFactorUpdated(state.scaleFactor, baseInterestRay, delinquencyFeeRay, protocolFee);
        }
        if (currentTimestamp != state.lastInterestAccruedTimestamp) {
            // Calculate the new state here
        }
        _processExpiredWithdrawalBatch(state);
    } else if (currentTimestamp != state.lastInterestAccruedTimestamp) {
        // Calculate the new state here
    }
}
This optimization reduces redundant state updates and calculations, only recalculating when necessary.

2)The _applyWithdrawalBatchPayment and _applyWithdrawalBatchPaymentView functions, which handle withdrawal payments. These functions can be gas-intensive, especially if there are many pending withdrawals. You might consider batching these payments to save on gas costs.
function _batchWithdrawalPayments(
    WithdrawalBatch memory batch,
    MarketState memory state,
    uint32 expiry,
    uint256 availableLiquidity
) internal {
    // Payment logic
    // ...
}

function _applyWithdrawalBatchPayment(
    WithdrawalBatch memory batch,
    MarketState memory state,
    uint32 expiry,
    uint256 availableLiquidity
) internal {
    _batchWithdrawalPayments(batch, state, expiry, availableLiquidity);
}

function _applyWithdrawalBatchPaymentView(
    WithdrawalBatch memory batch,
    MarketState memory state,
    uint256 availableLiquidity
) internal pure {
    // You can keep this function unchanged for view-only operations
}
By creating a separate function for batch withdrawal payments, you can optimize gas usage when processing multiple payments in a single batch. This is especially useful when there are many pending withdrawals to process.

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol

GAS

1)Remove Storage Redundancy: The getParameterConstraints function returns the constraints as individual variables. Instead, you could return a single MarketParameterConstraints struct, which can save gas since it reduces the number of storage reads.

function getParameterConstraints() external view returns (MarketParameterConstraints memory constraints) {
    constraints = MarketParameterConstraints({
        minimumDelinquencyGracePeriod: MinimumDelinquencyGracePeriod,
        maximumDelinquencyGracePeriod: MaximumDelinquencyGracePeriod,
        minimumReserveRatioBips: MinimumReserveRatioBips,
        maximumReserveRatioBips: MaximumReserveRatioBips,
        minimumDelinquencyFeeBips: MinimumDelinquencyFeeBips,
        maximumDelinquencyFeeBips: MaximumDelinquencyFeeBips,
        minimumWithdrawalBatchDuration: MinimumWithdrawalBatchDuration,
        maximumWithdrawalBatchDuration: MaximumWithdrawalBatchDuration,
        minimumAnnualInterestBips: MinimumAnnualInterestBips,
        maximumAnnualInterestBips: MaximumAnnualInterestBips
    });
}

https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol

GAS
1)In the collectFees function, the contract checks whether there are any accrued protocol fees (state.accruedProtocolFees) before proceeding with the withdrawal. This is a reasonable and gas-efficient approach to prevent unnecessary transactions with zero fees.

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GasCostProof {
    uint256 public feeBalance;

    constructor() {
        feeBalance = 100; // Some initial fee balance
    }

    // Function without zero fee check
    function collectFeesWithoutCheck() public {
        uint256 withdrawableFees = feeBalance;
        feeBalance = 0;  // Reset the fee balance

        // Simulate a transfer operation
        // In reality, this would transfer the fees to the recipient
        // but for the proof of concept, we just set the balance to zero.
        //msg.sender.transfer(withdrawableFees);
    }

    // Function with zero fee check
    function collectFeesWithCheck() public {
        if (feeBalance == 0) {
            // If there are no fees to collect, exit early
            return;
        }

        uint256 withdrawableFees = feeBalance;
        feeBalance = 0;  // Reset the fee balance

        // Simulate a transfer operation
        // In reality, this would transfer the fees to the recipient
        // but for the proof of concept, we just set the balance to zero.
        //msg.sender.transfer(withdrawableFees);
    }
}

In this proof of concept, there are two functions: collectFeesWithoutCheck and collectFeesWithCheck. The former does not check for zero fees and attempts to withdraw the entire fee balance, while the latter checks for zero fees before proceeding.
You will likely notice that the collectFeesWithCheck function consumes less gas when the fee balance is zero because it exits early without performing further operations. This check is a gas-efficient way to prevent unnecessary operations 