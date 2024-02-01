__Project Name:__ CDPro

__Prepared By:__ bLnk

<br />

__CDPro__ engaged __bLnk__ to review the security of its Smart Contract system. From __04 - December - 2023__ to __11 - December - 2023__ the source code in scope was reviewed. All findings have been recorded in the following report.

Notice that the examined smart contracts are not resistant to external/internal exploit. For a detailed understanding of risk severity, source code vulnerability, and potential attack vectors, refer to the complete audit report below.



# Project Overview

| CDPro | __Project__                                                                                               |
|--------------|----------------------------------------------------------------------------------------------------------|
| Language     | Solidity                                                                                                 |
| Codebase     | https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz                             |
| Commit       | [5b8d73763dab51c03225f267897127af02412ae9](https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/tree/5b8d73763dab51c03225f267897127af02412ae9) |


| Delivery Date     | 11 - December - 2023       |
|-------------------|--------------------------------|
| Audit Methodology | Manual Review |


| Vulnerability Level | Total | Pending | Declined | Acknowledged | Partially Resolved | Resolved |
|---------------------|-------|---------|----------|--------------|--------------------|----------|
| [Critical](#Critical)| 11     | 11       | 0        | 0            | 0                  | 0        |
| [High](#High)        | 0     | 0       | 0        | 0            | 0                  | 0        |
| [Medium](#Medium)    | 4     | 4       | 0        | 0            | 0                  | 0        |
| [Low](#Low)          | 10     | 10       | 0        | 0            | 0                  | 0        |

# Audit Scope & Methodology

## Scope

| ID | File      |                   |
|---------|------------|---------------------------------------|
| CDP      | CDPro.sol 
| GOV      | Gov.sol
| GVTK      | GovToken.sol
| AGGR      | IAggregatorV3.sol
| ORC      | Oracle.sol
| SC      | StableCoin.sol

## Methodology

The auditing process pays special attention to the following considerations:
- Testing the smart contracts against both common and uncommon attack vectors.
- Assessing the codebase to ensure compliance with current best practices and industry standards.
- Ensuring contract logic meets the specifications and intentions of the client.
- Cross-referencing contract structure and implementation against similar smart contracts produced by industry leaders.
- Thorough line-by-line manual review of the entire codebase by community auditors.

## Vulnerability Classifications

| Vulnerability Level | Classification                                                                               |
|---------------------|----------------------------------------------------------------------------------------------|
| [Critical](#Critical)            | Easily exploitable by anyone, causing causing loss of assets or undermining of the protocol’s goals.                 |
| [High](#High)                | Arduously exploitable by a subset of addresses, causing loss of assets or undermining of the protocol’s goals. |
| [Medium](#Medium)              | Inherent risk of future exploits that may or may not impact the smart contract execution.    |
| [Low](#Low)                 | Minor deviation from best practices.                                                         |



# Findings & Resolutions

| ID      | Title                                                                                     | Category            | Severity | Status  |
|-------|-------------------------------------------------------------------------------------------|---------------------|----------|---------|
| [C-01](#C01)  | CDPro::getUsdValueOfCollateral() calculates USD value incorrectly | Logic Error | CRITICAL | Pending |
| [C-02](#C02)  | CDPro::withdrawCollateral() calculates available collateral incorrectly | Logic Error | CRITICAL | Pending |
| [C-03](#C03)  | CDPro::_updatePositionInterest() calculates interestAmount improperly | Logic Error | CRITICAL | Pending |
| [C-04](#C04)  | CDPro::repay() user can never fully repay their position | Logic Error | CRITICAL | Pending |
| [C-05](#C05)  | CDPro::canLiquidate() users who borrow even a small % of the allowed LTV can make the function revert | Logic Error | CRITICAL | Pending |
| [C-06](#C06)  | CDPro::canLiquidate() will return true to users who are not violating LTV | Logic Error | CRITICAL | Pending |
| [C-07](#C07)  | CDPro::liquidate() is unfinished | Logic Error | CRITICAL | Pending |
| [C-08](#C08)  | Gov::propose() no user can ever propose | Logic Error | CRITICAL | Pending |
| [C-09](#C09)  | Gov::vote() any proposal can be voted on as many times as a user wants | Logic Error | CRITICAL | Pending |
| [C-10](#C10)  | Gov::vote() any proposal can be voted on forever | Logic Error | CRITICAL | Pending |
| [C-11](#C11)  | Gov.sol - no functionality to execute proposals | Logic Error | CRITICAL | Pending 
| [M-01](#M01)  | CDPro::_updatePositionInterest() you can get interest on your interest making it compounding | Logic Error | Medium | Pending |
| [M-02](#M02)  | Owner can change collateral LTVs and existing feed/heartbeat at will | Centralization | Medium | Pending |
| [M-03](#M03)  | CDPro::canLiquidate() it is possible for this function to revert due to sudden price changes | Logic Error | Medium | Pending |
| [L-01](#L01)  | CDPro - If oracle address is set to address(0) getUsdValueOfCollateral() will always revert | Gas | LOW | Pending |
| [L-02](#L02)  | CDPro::getUsdValueOfCollateral() unused variable can be removed to save gas | Gas | LOW | Pending |
| [L-03](#L03)  | CDPro::getTotalMaximumDebtInUsdOfAUser() code can be reused in canBorrow | Gas | LOW | Pending |
| [L-04](#L04)  | CDPro::howMuchCanUserBorrow functions have misleading variable names | Misleading Names | LOW | Pending |
| [L-05](#L05)  | CDPro::totalUsdBorrowableForUser() returns incorrect amount | Logic Error | LOW | Pending |
| [L-06](#L06)  | CDPro::totalUsdBorrowableForUser() does not include fees | Informational | LOW | Pending |
| [L-07](#L07)  | CDPro::canLiquidate() else statement can never be reached | Logic Error | LOW | Pending |
| [L-08](#L08)  | CDPro::liquidate() unnecessary function call return value not used | Gas | LOW | Pending |
| [L-09](#L09)  | Gov::propose() return proposalId++ to save gas | Gas | LOW | Pending |
| [L-10](#L10)  | Gov::propose() unused return variable _proposalId | Gas | LOW | Pending |




## <a id="Critical"></a>Critical


### <a id="C01"></a> C-01 CDPro::getUsdValueOfCollateral() calculates USD value incorrectly
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L444

#### PoC:
The below calculations were done using: https://www.mathsisfun.com/calculator-precision.html

`
amountOfCollateralToken * collateralPrice * collateralTypes[token].decimalAdjustment / 1e30
`

Case 1) USDC - 1 token

    amountOfCollateralToken = 1 * 1e6
    collateralPrice = 1 * 1e8
    collateralTypes[token].decimalAdjustment = 1e34

    1 * 1e6 * 1 * 1e8 * 1e34 / 1e30 = 1e18

The price of 1 USDC is 1e8 not 1e18. 

Case 2) SOL - 1 token (example price used is 60USD per token)

    amountOfCollateralToken = 1 * 1e9
    collateralPrice = 60 * 1e8
    collateralTypes[token].decimalAdjustment = 1e31

    1 * 1e9 * 60 * 1e8 * 1e31 / 1e30 = 60 * 1e18    
The price of 1 SOL is 60 * 1e8 not 60 * 1e18

Case 3) Ether - 1 token (exampe price used is 2200 per token)

    amountOfCollateralToken = 1 * 1e18
    collateralPrice = 2200 * 1e8
    collateralTypes[token].decimalAdjustment = 1e22

    1 * 1e18 * 2200 * 1e8 * 1e22 / 1e30 = 2200 * 1e18    
The price of 1 Ether is 2200 * 1e8 not 2200 * 1e18
#### Description:
The function CDPro::getUsdValueOfCollateral() calculates the value incorrectly, essentially making each asset's price bigger. The decimalAdjustment variable does not have correct values for the different assets.




#### Recommendation:

Change decimalAdjustment to 1e24 for USDC, 1e21 for Solana and 1e12 for Ether.



#### Resolution:


----

### <a id="C02"></a> C-02 CDPro::withdrawCollateral() calculates available collateral incorrectly
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L181

#### PoC:
Exploit Process (USDC):

1) Alice deposit 100 USDC
2) Borrow 80 stablecoin
3) Withdraw 36 stablecoins
4) Alice now has 116 usd, 16 more than she started with

`availableCollateral =
            userCollateralTotalUsdBalance - ((userBorrowedAmount * collateralTypes[collateralToken].ltv) / 100);`

    userCollateralTotalUsdBalance = 100 * 1e18 (value returned by getUsdValueOfCollateral())
    userBorrowedAmount = 80 * 1e18
    collateralTypes[collateralToken].ltv = 80

`availableCollateral = 100 * 1e18 - ((80 * 1e18 * 80) / 100) = 36 * 1e18`

Solana (60$): 
`availableCollateral = 3000 - ((1500 * 50) / 100) = 2250`

Ether (2.2k$):
`availableCollateral = 2200 - ((1540 * 70) / 100) = 1122`


#### Description:

The calculation for available collateral is incorrect. The way the math is written the user can always withdraw up to 51% of their ether collateral, 75% of their solana collateral and 36% of their USDC collateral. Essentially the lower the TLV the more collateral a user can withdraw despite borrowing the maximum amount. No fees are included in these calculations since a malicious user can withdraw with virtually no time in between transactions or even the absolute same transaction.

Note: If getUsdValueOfCollateral() is fixed to return 1e8 then users will not be able to withdraw collateral even without violating TLV because values of 1e8 and 1e18 will be compared inappropriately.


#### Recommendation:

Make corrections to the math AFTER fixing getUsdValueOfCollateral().


#### Resolution:


----


### <a id="C03"></a> C-03 CDPro::_updatePositionInterest() calculates interestAmount improperly
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L468

#### PoC:
1) Alice deposits 100 USDC
2) Alice borrows 40 stablecoin
3) 6 Months pass and Alice attempts to borrow again
4) _updatePositionInterest is entered
5) `interestAmount = userPosition.borrowedAmount * timeSinceLastUpdated * INTEREST_RATE`

    borrowedAmount = 40 * 1e18
    timeSinceLastUpdated = 15778800 (6months in seconds)
    INTEREST_RATE = 3170979198

`interestAmount = 40 * 1e18 * 15778800 * 3170979198 = 2001369862776096000000000000000000000`

That number is approximately equal to 2 * **1e36**. The interest should be 2 * 1e18.
#### Description:
Due to only multiplying you get an extremely big value for the interest. Essentially your interest is 1e18 times bigger than your actual collateral worth for just 6 months. Decimals are not taken into account which balloons this value to the moon.

#### Recommendation:
Divide by /1e18 at the end to get the correct interest amount value


#### Resolution:


----

### <a id="C04"></a> C-04 CDPro::repay() user can never fully repay their position
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L307-L309

#### PoC:
1) Current owed debt 100
2) User calls repay with amount = 100
3) _updatePositionInterest is called changing debt from 100 to 101
4) User repays 100 debt and now has 1 in debt.
5) User attempts to call repay with amount = 2 to cover the future debt
6) require statement reverts

#### Description:
When the user calls repay even if they have the correct amount to repay their interest will be updated after the require statement which means they can never fully repay. If they currently owe 100 and know that after the update they will owe 120 they cant call the function with 120 because the require statement will rever tellig them their amount is larger than their debt. The can only minimize their debt but never pay it out

#### Recommendation:
Allow the user to send an amount bigger than their debt and then just burn only the debt amount.

#### Resolution:


----

### <a id="C05"></a> C-05 CDPro::canLiquidate() users who borrow even a small % of the allowed LTV can make the function revert
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L541

## NOTE - This issue is assuming all issues regarding DECIMALS and FEES are fixed
In case they are not the issue is wrong values

#### PoC:
1) Deposit 100 USDC (borrow limit is 80)
2) Borrow 40 (borrow limit is now 40) 

        maxDebtInUsd = 80 * 1e18
        userBorrowedAmount = 40 * 1e18
        totalCollateralOfUserInUSD = 100 * 1e18
        availableDebtInUSD = 100 - 40 = 60 * 1e18
`availableDebtInUsd - maxDebtInUsd => 60 - 80`

Formula to calculate min amount that reverts:
1) Calculate TotalValue for collateral
2) Calculate MaxDebt
3) TotalValue - MaxDebt + 1 = borrow amount

More complicated example with 100 USDC and 2 Solana (60 USD per sol)
1) 100 + 120 = 220
2) 80% of 100 + 50% of 120 = 140 limit
3) 220 - 140 + 1 = 81.

`availableDebtInUsd = totalCollateralOfUserInUsd - userBorrowedAmount`

`availableDebtInUsd = 220 - 81 = 139`

`139 - 140 < 0`

#### Description:
A user only has to borrow a small percent of the available amount in order to cause the following calculation to be less than 0 and revert
`availableDebtInUsd - maxDebtInUsd`. That would make the user impossible to liquidate. The % needed to trigger that is the remainder of the LTV% + 1. USDC is 80% TLV so if a user borrows 20% + 1 it will revert. For WETH it is 30% + 1. For Solana it would be 50% + 1.

#### Recommendation:
Change how availableDebt is calculated.

#### Resolution:


----

### <a id="C06"></a> C-06 CDPro::canLiquidate() will return true to users who are not violating LTV
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L544

## NOTE - This issue is assuming all issues regarding DECIMALS and FEES are fixed

#### PoC:
Using the formula provided in the previous issue:
1) Calculate TotalValue for collateral
2) Calculate MaxDebt
3) TotalValue - MaxDebt = borrow amount

`100 USDC, 2 Sol (60usd per sol)`
1) 100 + 120 = 220
2) 80% of 100 + 50% of 120 = 140 limit
3) 220 - 140 = 80.

If the user borrows 80 the following elseif will return true

`else if (availableDebtInUsd - maxDebtInUsd == 0)`

#### Description:
Basically the same as C05 however this is assuming the user is on the verge of the LTV's remainder so they get liquidated instead of the function reverting. For Solana if 50% is used a user will be **exactly** on the verge of getting liquidated as they are perfectly using the max borrow amount. They will however be liquidated but for solana specifically this is not too big an issue as even a second longer and the interest would put the user over this limit. For WETH the user will be liquidated for borrowing 30% instead of 70% and for USDC the user will be liquidated for borrowing 20% instead of 80%.

#### Recommendation:
Change how availableDebt is calculated to allow the user to borrow more.

#### Resolution:


----

### <a id="C07"></a> C-07 CDPro::liquidate() is unfinished
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L493

#### PoC:

#### Description:
The function is simply unfinished - no fee handling, no reward for liquidator, attempting to transfer tokens tokens from the contract without checking if the contract has enough. There are many issues but I will simply leave it at unfinished

#### Recommendation:
Finish liquidation() implementation.

#### Resolution:

----

### <a id="C08"></a> C-08 Gov::propose() no user can ever propose
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/Gov.sol#L73

#### PoC:

#### Description:
`userBalance > (totalSupply * 100) / 100`

This means userBalance must be more than 100% of the totalSupply. No user can ever have more than 100% of the totalSupply.

#### Recommendation:
Change the multiplication from 100 to a lower number. If a user has to have 10% to vote then make it totalSupply * 10 / 100

#### Resolution:

----

### <a id="C09"></a> C-09 Gov::vote() any proposal can be voted on as many times as a user wants
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/Gov.sol#L120C13-L120C21

#### PoC:
1) vote(id)
2) vote(id)
3) vote(id)
4) repeat until the user is satisfied

#### Description:
On the given line the entire user's balance is used to vote. However there are no checks if the user has already voted or not so a user can call vote 1, 10, 100 or even a million times.

#### Recommendation:
Add a flag or some kind of check to mark whether or not the user has voted.



#### Resolution:


----

### <a id="C10"></a> C-10 Gov::vote() any proposal can be voted on forever
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/Gov.sol#L108

#### PoC:

#### Description:
There are no checks for whether or not the endTime has been exceeded and there is no functionality in the contract that changes the proposal from active to inactive

#### Recommendation:
Add functionality to the function that checks whether or not endtime has been reached OR add a way to change a proposal from active to inactive.

#### Resolution:

----

### <a id="C11"></a> C-11 Gov.sol - no functionality to execute proposals
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/main/src/Gov.sol

#### PoC:

#### Description:
In the entire contract there is no function that allows for the execution of proposals

#### Recommendation:
Write a function that executes proposals

#### Resolution:

-----------------

## <a id="Medium"></a> Medium

### <a id="M01"></a> M-01 CDPro::_updatePositionInterest() you can get interest on your interest making it compounding
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L471

#### Description:
By adding the interest amount to the user borrow amount the next time the interest is updated you will get interest on your interest amount. As more time passes and with more frequent borrows the difference between a compounding and non compounding interest will only increase. This only affects users who have borrowed more than once.

#### Recommendation:
Change the interest to make it non compounding or if intended add it to the documentation in order to be transparent with the users.

-----------------

### <a id="M02"></a> M-02 Owner can change collateral LTVs and existing feed/heartbeat at will
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L235
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/Oracle.sol#L27

#### Description:
The owner can call the linked functions and liquidate users at will.

#### Recommendation:
Add an alternative where proposals change the LTVs and feed/heartbeats.

-----------------

### <a id="M03"></a> M-03 CDPro::canLiquidate() it is possible for this function to revert due to sudden price changes
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L537

#### Description:
`uint256 availableDebtInUsd = totalCollateralOfUserInUsd - userBorrowedAmount`

If the value of a collateral drops by a lot and the liquidate function is not called in time this can potentially lead to userBorrowedAmount being higher than totalCollateralOfUserInUsd causing any further liquidation attempts to simply revert. Putting this as a medium since it is expected of the owners to keep track and call liquidate well before this point is reached.

#### Recommendation:
Make sure to often check user positions in order to avoid this case.

-----------------

## <a id="Low"></a> Low

### <a id="L01"></a> L-01 CDPro - If oracle address is set to address(0) getUsdValueOfCollateral() will always revert
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L436

#### Description:
There is no way to change the oracle value so if it is set by mistake to address(0) this function and by extension the protocol will never work. If a mistake is NOT made then this if will forever be a waste of gas. I recommend making checks for the oracle value in the constructor and setting it appropriately there.

-----------------

### <a id="L02"></a> L-02 CDPro::getUsdValueOfCollateral() unused variable can be removed to save gas
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L427

#### Description:
address user is not used so it can be removed to save some gas

-----------------

### <a id="L03"></a> L-03 CDPro::getTotalMaximumDebtInUsdOfAUser() code can be reused in canBorrow
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L330-L336

#### Description:
The entire function on this line
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L561

can be called in get borrow instead of the for cycle to replace the code.

-----------------

### <a id="L04"></a> L-04 CDPro::howMuchCanUserBorrow functions have misleading variable names
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L357
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L378
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L399

#### Description:
alreadyBorrowedUSDC, alreadyBorrowedWETH, alreadyBorrowedSOL are all equal to the currently borrowed amount but borrowAmount includes borrows from all token types.

-----------------

### <a id="L05"></a> L-05 CDPro::totalUsdBorrowableForUser() returns incorrect amount
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L413

#### Description:
This function calls howMuchCanUserBorrow for sol weth and usdc. If a user can borrow 50 USD worth of coins all those functions will return 50 and totalUsdBorrowableForUser will then return 150 which will misinform the user on how much they can borrow

-----------------

### <a id="L06"></a> L-06 CDPro::totalUsdBorrowableForUser() does not include fees
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L413

#### Description:
When a user calls this function they would expect to get the maximum amount they can borrow but the function does not include fees which will automatically decrease the max borrow amount.

-----------------

### <a id="L07"></a> L-07 CDPro::canLiquidate() else statement can never be reached
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L546

#### Description:
The if handles the case when the value is positive, the else if handles the == 0 case. Since uint256 values are used there will never be negative values because solidity will revert on those and the else specifically handles the negative value case which will never be reached.

-----------------

### <a id="L08"></a> L-08 CDPro::liquidate() unnecessary function call return value not used
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/CDPro.sol#L504

#### Description:
This function's return value is not used for anything and it is a view function so it doesn't change the state therefore it is currently unneeded

-----------------

### <a id="L09"></a> L-09 Gov::propose() return proposalId++ to save gas
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/Gov.sol#L98-L99

#### Description:
instead of increasing proposalId and then returning it -1 you can just return proposalId++;

-----------------

### <a id="L10"></a> L-10 Gov::propose() unused return variable _proposalId
https://github.com/owenThurm/CDPro-david-dacruz-Soulrealz/blob/5b8d73763dab51c03225f267897127af02412ae9/src/Gov.sol#L64

#### Description:
The variable is not returned in the end so it can be removed to save gas

-----------------

## Disclaimer
> 
> This report is not, nor should be considered, an “endorsement” or “disapproval” of any particular project or team. This report is not, nor should be considered, an indication of the economics or value of any “product” or “asset” created by any team or project that contracts the firm to perform a security assessment. This report does not provide any warranty or guarantee regarding the absolute bug-free nature of the technology analyzed, nor do they provide any indication of the technologies proprietors, business, business model or legal compliance.
> 
> This report should not be used in any way to make decisions around investment or involvement with any particular project. This report in no way provides investment advice, nor should be leveraged as investment advice of any sort. This report represents an extensive assessing process intending to help our customers increase the quality of their code while reducing the high level of risk presented by cryptographic tokens and blockchain technology.
> 
> Blockchain technology and cryptographic assets present a high level of ongoing risk. The firm’s position is that each company and individual are responsible for their own due diligence and continuous security. The firm’s goal is to help reduce the attack vectors and the high level of variance associated with utilizing new and consistently changing technologies, and in no way claims any guarantee of security or functionality of the technology we agree to analyze.
> 
> The assessment services provided by the firm is subject to dependencies and under continuing
> development. You agree that your access and/or use, including but not limited to any services, reports, and materials, will be at your sole risk on an as-is, where-is, and as-available basis. Cryptographic tokens are emergent technologies and carry with them high levels of technical risk and uncertainty. The assessment reports could include false positives, false negatives, and other unpredictable results. The services may access, and depend upon, multiple layers of third-parties.
> 
> Notice that smart contracts deployed on the blockchain are not resistant from internal/external exploit. Notice that active smart contract owner privileges constitute an elevated impact to any smart contract’s safety and security. Therefore, the firm does not guarantee the explicit security of the audited smart contract, regardless of the verdict.

<br/>

