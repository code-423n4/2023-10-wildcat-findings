## Impact
Steal funds
A state variable is changed after a contract uses call.value. The attacker uses a fallback function—which is automatically executed after Ether is transferred from the targeted contract—to execute the vulnerable function again, before the state variable is changed.
Reentrancy in WildcatMarketBase._blockAccount(MarketState,address) (src/market/WildcatMarketBase.sol#163-187):
External calls:
- escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(accountAddress,borrower,address(this)) (src/market/WildcatMarketBase.sol#172-176)
State variables written after the call(s):
- _accounts[escrow].scaledBalance += scaledBalance (src/market/WildcatMarketBase.sol#178)
WildcatMarketBase._accounts (src/market/WildcatMarketBase.sol#68) can be used in cross function reentrancies:
- WildcatMarketBase.getAccountRole(address) (src/market/WildcatMarketBase.sol#299-301)
- WildcatMarketBase.scaledBalanceOf(address) (src/market/WildcatMarketBase.sol#292-294)
- _accounts[accountAddress] = account (src/market/WildcatMarketBase.sol#185)
WildcatMarketBase._accounts (src/market/WildcatMarketBase.sol#68) can be used in cross function reentrancies:
- WildcatMarketBase.getAccountRole(address) (src/market/WildcatMarketBase.sol#299-301)
- WildcatMarketBase.scaledBalanceOf(address) (src/market/WildcatMarketBase.sol#292-294)

Description:

Detection of the [reentrancy bug](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy). Do not report reentrancies that involve Ether (see `reentrancy-eth`).



## Proof of Concept
https://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit

function splitDAO(
  uint _proposalID,
  address _newCurator
) noEther onlyTokenholders returns (bool _success) {

  ...
  // XXXXX Move ether and assign new Tokens.  Notice how this is done first!
  uint fundsToBeMoved =
      (balances[msg.sender] * p.splitData[0].splitBalance) /
      p.splitData[0].totalSupply;
  if (p.splitData[0].newDAO.createTokenProxy.value(fundsToBeMoved)(msg.sender) == false) // XXXXX This is the line the attacker wants to run more than once
      throw;

  ...
  // Burn DAO Tokens
  Transfer(msg.sender, 0, balances[msg.sender]);
  withdrawRewardFor(msg.sender); // be nice, and get his rewards
  // XXXXX Notice the preceding line is critically before the next few
  totalSupply -= balances[msg.sender]; // XXXXX AND THIS IS DONE LAST
  balances[msg.sender] = 0; // XXXXX AND THIS IS DONE LAST TOO
  paidOut[msg.sender] = 0;
  return true;
}

## Tools Used

Slither

## Recommended Mitigation Steps

bool private locked = false;

modifier noReentrancy() {
    require(!locked, "Reentrant call detected!");
    locked = true;
    _;
    locked = false;
}

function _blockAccount(MarketState memory state, address accountAddress) internal noReentrancy {
    Account memory account = _accounts[accountAddress];
    if (account.approval != AuthRole.Blocked) {
        uint104 scaledBalance = account.scaledBalance;
        account.approval = AuthRole.Blocked;
        emit AuthorizationStatusUpdated(accountAddress, AuthRole.Blocked);

        if (scaledBalance > 0) {
            account.scaledBalance = 0;
            _accounts[accountAddress] = account;

            address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
                accountAddress,
                borrower,
                address(this)
            );
            emit Transfer(accountAddress, escrow, state.normalizeAmount(scaledBalance));
            _accounts[escrow].scaledBalance += scaledBalance;
            emit SanctionedAccountAssetsSentToEscrow(
                accountAddress,
                escrow,
                state.normalizeAmount(scaledBalance)
            );
        }
    }
}

Apply the [`check-effects-interactions` pattern](http://solidity.readthedocs.io/en/v0.4.21/security-considerations.html#re-entrancy).

### Time spent:
36 hours