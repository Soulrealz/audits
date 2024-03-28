__Project Name:__ HalbornCTF_Solidity_Ethereum

__Prepared By:__ bLnk

<br />

Notice that the examined smart contracts are not resistant to external/internal exploit. For a detailed understanding of risk severity, source code vulnerability, and potential attack vectors, refer to the complete audit report below.



# Project Overview

| HalbornCTF_Solidity_Ethereum | __Project__                                                                                               |
|--------------|----------------------------------------------------------------------------------------------------------|
| Language     | Solidity                                                                                                 |
| Codebase     | https://github.com/HalbornSecurity/CTFs                             |
| Commit       | [6bc8cc1c8f5ac6c75a21da6d5ef7043f0862603b](https://github.com/HalbornSecurity/CTFs/tree/6bc8cc1c8f5ac6c75a21da6d5ef7043f0862603b) |


| Delivery Date     | 24 - 02 - 2024       |
|-------------------|--------------------------------|
| Audit Methodology | Manual Review |


| Vulnerability Level | Total | Pending | Declined | Acknowledged | Partially Resolved | Resolved |
|---------------------|-------|---------|----------|--------------|--------------------|----------|
| [Critical](#Critical)| 8     | 8       | 0        | 0            | 0                  | 0        |
| [High](#High)        | 3     | 3       | 0        | 0            | 0                  | 0        |
| [Medium](#Medium)    | 2     | 2       | 0        | 0            | 0                  | 0        |
| [Low](#Low)          | 0     | 0       | 0        | 0            | 0                  | 0        |
| [Informational](#Info)| 2     | 2       | 0        | 0            | 0                  | 0        |

# Audit Scope & Methodology

## Scope

| File |       |                   |
|---------|------------|---------------------------------------|
|   HalbornCTF_Solidity_Ethereum/src    |
|       | HalbornLoans.sol
|       | HalbornNFT.sol
|       | HalbornToken.sol
|       | libraries/Multicall.sol

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
| [Critical](#Critical)            | Easily exploitable by anyone, causing loss of assets or undermining of the protocol’s goals.                 |
| [High](#High)                | Arduously exploitable by a subset of addresses, causing loss of assets or undermining of the protocol’s goals. |
| [Medium](#Medium)              | Inherent risk of future exploits that may or may not impact the smart contract execution.    |
| [Low](#Low)                 | Minor deviation from best practices.                                                         |
| [Informational](#Info)                 | Various things that can may or may not have an impact depending on the context.                                                         |



# Findings & Resolutions

| ID      | Title                                                                                     | Category            | Severity | Status  |
|-------|-------------------------------------------------------------------------------------------|---------------------|----------|---------|
| [C-01](#C01)  | `HalbornLoans::returnLoan` increases user debt instead of decreasing it | Logic Error | CRITICAL | Pending |
| [C-02](#C02)  | `HalbornLoans::getLoan` lets users only get loan bigger than their collateral deposited | Logic Error | CRITICAL | Pending |
| [C-03](#C03)  | `HalbornNFT::setMerkleRoot` missing authorization | Authorization | CRITICAL | Pending |
| [C-04](#C04)  | `HalbornLoans::depositNFTCollateral` nfts cannot be deposited to the contract | Logic Error | CRITICAL | Pending |
| [C-05](#C05)  | User can exit the protocol with both their nft and a loan | Reentrancy | CRITICAL | Pending |
| [C-06](#C06)  | `HalbornNFT::mintAirdrops` reverts on non minted tokens | Logic Error | CRITICAL | Pending |
| [C-07](#C07)  | `HalbornNFT::mintBuyWithEth` will eventually be unable to mint NFTs | Logic Error | CRITICAL | Pending |
| [C-08](#C08)  | `_authorizeUpgrade` has no authorization | Authorization | CRITICAL | Pending |
| [H-01](#H01)  | Token loan amount treated as always equal value to `collateralPrice` | Logic Error | High | Pending |
| [H-02](#H02)  | Loans have an LTV of 100% which can lead to bad debt | Missing Logic | High | Pending |
| [H-03](#H03)  | Missing Liquidation Logic | Missing Logic | High | Pending |
| [M-01](#M01)  | `HalbornLoans::collateralPrice` is a static amount | Logic Error | Medium | Pending |
| [M-02](#M02)  | Second preimage attack in merkle tree possible | Logic Error | Medium | Pending |
| [I-01](#I01)  | Upgradeable contracts will have a missing storage gap if OZ version >5.0 | OZ Package Version | Informational | Pending |
| [I-02](#I02)  | `Multicall` does not identify non-canonical context | Missing Logic | Informational | Pending |




## <a id="Critical"></a>Critical


### <a id="C01"></a> C-01 `HalbornLoans::returnLoan` increases user debt instead of decreasing it
https://github.com/HalbornSecurity/CTFs/blob/6bc8cc1c8f5ac6c75a21da6d5ef7043f0862603b/HalbornCTF_Solidity_Ethereum/src/HalbornLoans.sol#L70

#### PoC:
This test will not work without first fixing different issues. See [C-02](#C02) & [C-04](#C04).

![alt text](image.png)

#### Description:
The `returnLoan` function is used so that a user may repay their owed debt and become eligible to withdraw their collateral. However the variable that tracks the user's debt is incorrectly increased instead of decreased.

`usedCollateral[msg.sender] += amount;`

This will lead to users being unable to withdraw their collateral effectively losing their nft forever. This will also cause problems in future calls to `getLoan` as a user may take out half a loan, repay it and then he will be unable to take out a loan again as `usedCollateral` will be maxed out for him.

#### Recommendation:
Substract the amount from the mapping instead of adding it

```diff
+ usedCollateral[msg.sender] += amount
- usedCollateral[msg.sender] -= amount
```



#### Resolution:


-----------------

### <a id="C02"></a> C-02 `HalbornLoans::getLoan` lets users only get loan bigger than their collateral deposited
https://github.com/HalbornSecurity/CTFs/blob/6bc8cc1c8f5ac6c75a21da6d5ef7043f0862603b/HalbornCTF_Solidity_Ethereum/src/HalbornLoans.sol#L60
#### PoC:
This test will not work without first fixing different issues. See [C-04](#C04).

![alt text](image-1.png)

#### Description:
The `getLoan` function performs the following check upon being called:

`totalCollateral[msg.sender] - usedCollateral[msg.sender] < amount`

This is incorrectly checking if the user's available collateral is smaller than the amount that the user wants to withdraw. A user can only ever get a loan that is bigger than their deposited collateral. There is also no upper limit so a user can theoretically mint `type(uint256).max`

#### Recommendation:
Change the `less than <` operator to a `greater or equals >=` operator in order to the allow the user to get a loan that is less or equal to their provided collateral.

```diff
+ totalCollateral[msg.sender] - usedCollateral[msg.sender] >= amount
- totalCollateral[msg.sender] - usedCollateral[msg.sender] < amount
```



#### Resolution:


-----------------

### <a id="C03"></a> C-03 `HalbornNFT::setMerkleRoot` missing authorization
https://github.com/HalbornSecurity/CTFs/blob/6bc8cc1c8f5ac6c75a21da6d5ef7043f0862603b/HalbornCTF_Solidity_Ethereum/src/HalbornNFT.sol#L41

#### PoC:

![alt text](image-2.png)

#### Description:
There is no functionality that checks whether `msg.sender` is an authorized user or not so any user can set a new `merkleRoot`

#### Recommendation:
Consider adding the `onlyOwner` modifier inherited by `OwnableUpgradeable`

```diff
+ function setMerkleRoot(bytes32 merkleRoot_) public onlyOwner {
- function setMerkleRoot(bytes32 merkleRoot_) public {
```



#### Resolution:


-----------------

### <a id="C04"></a> C-04 `HalbornLoans::depositNFTCollateral` nfts cannot be deposited to the contract
https://github.com/HalbornSecurity/CTFs/blob/6bc8cc1c8f5ac6c75a21da6d5ef7043f0862603b/HalbornCTF_Solidity_Ethereum/src/HalbornLoans.sol#L39
#### PoC:

![alt text](image-3.png)

#### Description:
`safeTransferFrom` first checks if the receiver is a contract. If that is true then the following check is performed `_checkOnERC721Received`. Since the contract `HalbornLoans` does not comply with that check the call will revert and no one can ever deposit an NFT. 



#### Recommendation:
Make the contract extend `IERC721Receiver` from OZ and implement `onERC721Received`.

```diff
+ contract HalbornLoans is Initializable, UUPSUpgradeable, MulticallUpgradeable, IERC721ReceiverUpgradeable {
- contract HalbornLoans is Initializable, UUPSUpgradeable, MulticallUpgradeable {
```
and
```diff
+ function onERC721Received(address, address, uint256, bytes calldata) external returns (bytes4) {
+        return this.onERC721Received.selector;
+    }
```

Additional checks may be added if needed.

#### Resolution:


-----------------

### <a id="C05"></a> C-05 User can exit the protocol with both their nft and a loan
https://github.com/HalbornSecurity/CTFs/blob/6bc8cc1c8f5ac6c75a21da6d5ef7043f0862603b/HalbornCTF_Solidity_Ethereum/src/HalbornLoans.sol#L53
#### PoC:
This test assumes that [C-04](#C04) & [C-02](#C02) are fixed with the given recommendations. For C-04 `safeTransferFrom` can be replaced with `transferFrom` and the test will work as well.

![alt text](image-4.png)

![alt text](image-5.png)

#### Description:
The `withdrawCollateral` function doesn't implement the CEI (checks - effects - interactions) pattern and the `safeTransferFrom` call gives control back to the user if it is a contract. Using that opportunity a malicious user can get a loan before their `totalCollateral` gets decreased.

#### Recommendation:

```diff
+ totalCollateral[msg.sender] -= collateralPrice;
+ delete idsCollateral[id];
+ nft.safeTransferFrom(address(this), msg.sender, id);

- nft.safeTransferFrom(address(this), msg.sender, id);
- totalCollateral[msg.sender] -= collateralPrice;
- delete idsCollateral[id];
```



#### Resolution:


-----------------

### <a id="C06"></a> C-06 `HalbornNFT::mintAirdrops` reverts on non minted tokens
https://github.com/HalbornSecurity/CTFs/blob/6bc8cc1c8f5ac6c75a21da6d5ef7043f0862603b/HalbornCTF_Solidity_Ethereum/src/HalbornNFT.sol#L46
#### PoC:

![alt text](image-7.png)
#### Description:
The call to `_exists(id)` in the function will return true if the id has been minted. This means that the following require statement will revert if the id does not exist.

`require(_exists(id), "Token already minted");`

This means that no user will ever be able to mint as the only way to pass this check is to attempt to mint an already minted nft.



#### Recommendation:

```diff
+ require(!_exists(id), "Token already minted");
- require(_exists(id), "Token already minted");
```



#### Resolution:


-----------------

### <a id="C07"></a> C-07 `HalbornNFT::mintBuyWithEth` will eventually be unable to mint NFTs
https://github.com/HalbornSecurity/CTFs/blob/6bc8cc1c8f5ac6c75a21da6d5ef7043f0862603b/HalbornCTF_Solidity_Ethereum/src/HalbornNFT.sol#L59-L67
#### PoC:
This test will not work without first fixing different issues. See [C-06](#C06).

![alt text](image-6.png)
#### Description:
The function `mintBuyWithEth` mints an nft with an id equal to `idCounter` that gets incremented beforehand in the same function. However it does not take into account that an NFT with that id might already exist due to it being minted from `mintAirdrops`. If `mintAirdrops` creates an nft with id = 1 then `mintBuyWithEth` will be unusable as `idCounter` will get increased to 1 and it will be unable to mint due to the nft already existing. There is no other way to increase the counter so the function will be bricked forever. 

#### Recommendation:
There is no easy fix as `mintAirdrops` can mint nfs with any id such as `150, 999, 1, 50, 3` and so on. Keeping track of which id was minted and which wasn't will not be a trivial task with the way the contract is right now. Minted ids can be kept offchain in a DB and have users interact with an API that gets them a free id. If that solution is not acceptable then both minting functions have to be rewritten in a way that avoids this issue.


#### Resolution:


-----------------

### <a id="C08"></a> C-08 `_authorizeUpgrade` has no authorization
https://github.com/HalbornSecurity/CTFs/blob/master/HalbornCTF_Solidity_Ethereum/src/HalbornLoans.sol

https://github.com/HalbornSecurity/CTFs/blob/master/HalbornCTF_Solidity_Ethereum/src/HalbornNFT.sol

https://github.com/HalbornSecurity/CTFs/blob/master/HalbornCTF_Solidity_Ethereum/src/HalbornToken.sol
#### Description:
As per OZ's official documentation:
https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/789ba4f167cc94088e305d78e4ae6f3c1ec2e6f1/contracts/proxy/utils/UUPSUpgradeable.sol#L122-L131

`_authorizeUpgrade` has to be overriden with some kind of check that makes sure that `msg.sender` is an authorized user. Since no such check has been added anyone can perform an upgrade through the `upgradeTo` inherited from `UUPSUpgradeable`.

#### Recommendation:

```diff
+ function _authorizeUpgrade(address) internal override onlyOwner {}
- function _authorizeUpgrade(address) internal override {}
```



#### Resolution:


-----------------


## <a id="High"></a> High

### <a id="H01"></a> H-01 Token loan amount treated as always equal value to `collateralPrice`
https://github.com/HalbornSecurity/CTFs/blob/master/HalbornCTF_Solidity_Ethereum/src/HalbornLoans.sol

#### Description:
The `getLoan` function assume that `collateralPrice` (2 ether) will always equal that many amount of tokens 1:1 (2 ethers worth of tokens). With decimals = 18 (default) if a user deposits an nft they can get 2 HalbornTokens from the `getLoan` function. With current ETH price = 2928$ (24.02.2024 02:25AM +2 UTC) that means that each token is worth ~1464$. If the value of those tokens fluctuates due to outside factors (such as trading on dexes) then those tokens might be worth more than the collateral and it might get left behind as the users will have no incentive to return their loan when it is worth more money. This will be a loss for the protocol. Alternatively if the value of the tokens drop then users will not have a reason to deposit and get a loan as they will get less value than intended. This also applies for fluctuations in the NFT price too. The NFT costs 1 ether but if the market decides to trade the distributed supply for a different price the protocol might suffer from that.


#### Resolution:


-----------------

### <a id="H02"></a> H-02 Loans have an LTV of 100% which can lead to bad debt
https://github.com/HalbornSecurity/CTFs/blob/master/HalbornCTF_Solidity_Ethereum/src/HalbornLoans.sol

#### Description:
If a user has taken out a loan and the value of his loan goes up then he will never return his loan. If the value of a user's loan is 1000$ and the tokens' price rises to 1010$ then the user will have effectively made money and will never return the loan as it is not profitable for him. The protocol will have accrued bad debt as the nft will be stuck in the protocol because it still belongs to the original user and the protocol will have minted out tokens to said user.

#### Recommendation:
Add an LTV ratio that the protocol team considers reasonable. Other protocols usually have ether based collaterals at 70~80% LTV.

#### Resolution:


-----------------

### <a id="H03"></a> H-03 Missing Liquidation Logic
https://github.com/HalbornSecurity/CTFs/blob/master/HalbornCTF_Solidity_Ethereum/src/HalbornLoans.sol

#### Description:
If the protocol starts to accrue bad debt there is no logic to remediate the losses. Liquidation logic can help acquire NFTs from users who's loan value is bigger than their provided collateral. NFTs can later on be sold to recuperate some/all of the losses.

#### Recommendation:
Add logic that liquidates users that accrue bad debt. This will be more effective if [H-02](#H02) is fixed as well.

#### Resolution:


-----------------

## <a id="Medium"></a> Medium

### <a id="M01"></a> M-01 `HalbornLoans::collateralPrice` is a static amount
https://github.com/HalbornSecurity/CTFs/blob/master/HalbornCTF_Solidity_Ethereum/src/HalbornLoans.sol

#### Description:
`collateralPrice` unlike `price` in `HalbornNFT` is a static amount. This means that the NFT is always treated as if it provides the same collateral value irregardless if it's cost was 1 ether or 10 ether. Calls to `setPrice` can subsequently result in loss of value as the loaned tokens will have been minted against different prices. When the NFT price is 1 ether the user will be able to take out a loan of 2 tokens. When the NFT price is 0.1 ether (10 times less) the user will again be able to take out a loan of 2 tokens. This will cause significant fluctuations in the price of the tokens. Some users will profit off of this and some will lose value. 

#### Recommendation:
There are 2 possible solutions.
1) Make `price` immutable like `collateralPrice` as this will allow it to preserve its ratio.
2) Remove immutable from `collateralPrice` and add a setter for it too. All calls to one setter should also call the other setter and set the same ratio as before.

-----------------

### <a id="M02"></a> M-02 Second preimage attack in merkle tree possible
https://github.com/HalbornSecurity/CTFs/blob/6bc8cc1c8f5ac6c75a21da6d5ef7043f0862603b/HalbornCTF_Solidity_Ethereum/src/HalbornNFT.sol#L45

#### Description:
Second preimage attack is when an attacker attempts to create a new piece of data (a leaf node) that produces the same hash value as an existing leaf node in the Merkle tree. In other words, the attacker is trying to find a different input that hashes to the same value as the original data without modifying the original data itself.

#### Recommendation:
The following [article](https://www.rareskills.io/post/merkle-tree-second-preimage-attack#935a00_f4837d828df44f6f923b590367d7119e~mv2.png) by Rareskills goes in depth about what the attack is and how to prevent it.

-----------------

## <a id="Info"></a> Informational

### <a id="I01"></a> I-01 Upgradeable contracts will have a missing storage gap if OZ version >5.0
#### Description:
Upgradeable OZ Contracts in versions >5.0 use [Namespaced Storage](https://docs.openzeppelin.com/contracts/5.x/upgradeable#storage_gaps) while older versions have [storage gaps](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps). If deployed with a newer version then the protocol will have to add a storage gap to the upgradeable contracts.

-----------------

### <a id="I02"></a> I-02 `Multicall` does not identify non-canonical context 
#### Description:
The OZ implementation of MulticallUpgradeable has the following [comments](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/789ba4f167cc94088e305d78e4ae6f3c1ec2e6f1/contracts/utils/MulticallUpgradeable.sol#L17-L20). The Halborn contracts interact with the caller's address `msg.sender` so preserving specific context with `_msgSender` might not be necessary however all the contracts are upgradeable. In order to ensure maximum safety for future versions too context handling should be implemented.

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

