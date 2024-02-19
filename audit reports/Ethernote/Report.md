__Project Name:__ Ethernote

__Prepared By:__ bLnk

<br />

__Ethernote__ engaged __bLnk__ to review the security of its Smart Contract system. From __16.11.2023__ to __24.11.2023__ the source code in scope was reviewed. All findings have been recorded in the following report.

Notice that the examined smart contracts are not resistant to external/internal exploit. For a detailed understanding of risk severity, source code vulnerability, and potential attack vectors, refer to the complete audit report below.



# Project Overview

| Ethernote | __Project__                                                                                               |
|--------------|----------------------------------------------------------------------------------------------------------|
| Language     | Solidity                                                                                                 |
| Codebase     | https://github.com/GuardianAudits/PA-Ethernote-Foundry                             |
| Commit       | [c7fb03531e8bd43c2a2de0f165500106b55a1415](https://github.com/GuardianAudits/PA-Ethernote-Foundry/tree/c7fb03531e8bd43c2a2de0f165500106b55a1415) |


| Delivery Date     | 24.11.2023       |
|-------------------|--------------------------------|
| Audit Methodology | Slither , Manual Review |


| Vulnerability Level | Total | Pending | Declined | Acknowledged | Partially Resolved | Resolved |
|---------------------|-------|---------|----------|--------------|--------------------|----------|
| [Critical](#Critical)| 1     | 1       | 0        | 0            | 0                  | 0        |
| [High](#High)        | 1     | 1       | 0        | 0            | 0                  | 0        |
| [Medium](#Medium)    | 2     | 2       | 0        | 0            | 0                  | 0        |
| [Low](#Low)          | 3     | 3       | 0        | 0            | 0                  | 0        |

# Audit Scope & Methodology

## Scope

| ID | File      |                   |
|---------|------------|---------------------------------------|
| ETH      | src/Ethernote.sol

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
| [C-01](#C01)  | Incorrect fee tracking                                     | Logic Error                 | CRITICAL     | Pending |
| [H-01](#H01)  |  Owner can increase redeemFee so much that users will not receive any amount back when calling redeem     | Centralization         | HIGH     | Pending |
| [H-02](#H02)  |  Contract is pausable but cannot be paused/unpaused     | Logic Error         | HIGH     | Pending |
| [M-01](#M01)  |  wstETH variable can be set to a different implementation address | Rug Pull   | MEDIUM   | Pending |
| [M-02](#M02)  |  User can receive funds minted from _mintEth() but due to update of edition the user may then redeem through _redeemWstEth()                                                     | Logic error         | MEDIUM   | Pending |
| [L-01](#L01)  | createNote and updateNote can be called with an invalid edition                                                                              | Logic Error | LOW      | Pending |
| [L-02](#L02)  | Note can be reactivated after calling ceaseNote on it                                    | Logic Error          | LOW      | Pending |




## <a id="Critical"></a>Critical


### <a id="C01"></a> C-01 Incorrect fee tracking

https://github.com/GuardianAudits/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L197
https://github.com/GuardianAudits/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L186


#### PoC:
1) User A mint 5 with ethereum set to true. 
2) User B mint 50 with ethereum set to false.
3) User B calls redeem. redeemFee set to 100. The following code returns 5

`uint256 redeemFee = (metadata[_tokenId].balance * notes[metadata[_tokenId].note].redeemFee) / 1e4;` 

This leaves the contract with fees = 5 and 5 wstEth in the contract.

4) Owner calls withdraw => drains all 5 ether out of the contract.
5) User A attempts to call redeem but there is no ether to send him back 



#### Description:
Fees for both eth and wstEth are treated the same. Although they have the same value differeng things are provided when mint and redeem are called. When withdraw is called only eth is sent to the payout address. That means that the contract can be drained of its ether while wstEth remains there.




#### Recommendation:
Keep 2 variables for the different fees. uint256 feesEth and uint256 feesWstEth.




#### Resolution:


----


## <a id="High"></a> High

### <a id="H01"></a> H-01 Owner can increase redeemFee so much that users will not receive any amount back when calling redeem

https://github.com/GuardianAudits/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L184C107-L184C107
https://github.com/GuardianAudits/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L195



#### PoC:

1) User A mints 10 ether
2) Owner calls updateNote with redeemFee = 1e4
3) User A attempts to redeem their 10 ether.
4) In the following code we have balance = 10, note.redeemFee = 1e4

`uint256 redeemFee = metadata[_tokenId].balance * notes[metadata[_tokenId].note].redeemFee / 1e4;`

We get:

`redeemFee = 10 * 1e4 / 1e4` => `redeemFee = 10`

`uint256 amount = metadata[_tokenId].balance - redeemFee;`

`amount = 10 - 10` => `amount = 0`

5) User gets back nothing and the owner keeps the 10 ether.


#### Description:

Owner can change the value of notes.redeemFee whenever he wants with updateNote() function. When a user attempts to redeem their eth or wsteth the owner can frontrun that transaction with updateNote and setting redeemFee to a very high value. In the worst case in the following code amount will equal 0. Meaning that the owner gets to keep fees+the entire ether while the user gets nothing effectively getting scammed out completely. This will also work without frontrunning because users that trust the protocol will not check to see if the owner has previously changed the fees without announcing any change. There is an issue in the opposite direction as well. The owner can set the redeemFee so low that after the division of 1e4 redeemFee equals 0.

`uint256 redeemFee = metadata[_tokenId].balance * notes[metadata[_tokenId].note].redeemFee / 1e4;`

`uint256 amount = metadata[_tokenId].balance - redeemFee;`


#### Recommendation:
Set the redeemFee value once and make it a constant. Alternative is to add a limit to how big it can be, preferably a small limit.


### <a id="H02"></a> H-02 Contract is pausable but cannot be paused/unpaused

#### Description:

Ethernote inherits from Pausable but does not have functions that allow the contract to be paused and/or unpaused.

-----------------



## <a id="Medium"></a> Medium

### <a id="M01"></a> M-01 wstETH variable can be set to a different implementation address

https://github.com/GuardianAudits/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L255


#### Description:
Allowing wstEth to be changed creates a backdoor for an owner turned malicious. He can change the variable to point to a contract implemented in such a way that all interactions with it will drain the user's funds, the contract won't actually transfer tokens to the user and so on. The possibilities are endless.




#### Recommendation:
Set the value once and make it immutable. That way users can check the wstETH implementation contract, confirm there are no issues with it and use the protocol without worry.



### <a id="M02"></a> M-02 User can receive funds minted from _mintEth() but due to update of edition the user may then redeem through _redeemWstEth()


#### PoC:
    1) User mints with an edition that has ethereum == true
    2) User is minted token with _mintEth function
    3) Owner calls updateNote() changing that edition ethereum = false
    4) User attempts to redeem and _redeemWstEth() is called instead of _redeemEth()
    5) User receives wstETH instead of ether and that is provided the contract has enough to give
    5.a) If the contract does not have enough wstETH the transaction executes normally leaving the user with nothing


#### Description:

The Owner has the ability to update editions at will thus users who have used _mintEth() or _mintWstEth() may redeem inappropriately from _redeemWstEth() and _redeemEth() respectively. This will lead to loss of funds for the user.




#### Recommendation:
If this functionality is to be kept then pause mints, add checks to make sure all redeems are done and then change the edition.ethereum field. It would be simpler however to have 2 different editions - one with ethereum set to true and one set to false and make the variable immutable.




#### Resolution:

-----------------


## <a id="Low"></a> Low


### <a id="L01"></a> L-01 createNote and updateNote can be called with an invalid edition


https://github.com/GuardianAudits/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L125
https://github.com/GuardianAudits/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L98

#### Description:
There are no checks to see if the added/edited edition exists or not leaving the note unusable until a proper edition is set.




#### Recommendation:
Add checks for an existing edition.




#### Resolution:

-----------------

### <a id="L02"></a> L-02 Note can be reactivated after calling ceaseNote on it


#### Description:

After ceaseNote is called on a note the Owner can later activate that note again with updateNote() function

#### Recommendation:
Remove updateNote()'s ability to change the `active` field




#### Resolution:

____

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

