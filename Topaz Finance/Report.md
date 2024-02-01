__Project Name:__ Topaz Finance

__Prepared By:__ bLnk

<br />

__Topaz Finance__ engaged __bLnk__ to review the security of its Smart Contract system. From __11 - December - 2023__ to __31 - December - 2023__ the source code in scope was reviewed. All findings have been recorded in the following report.

Notice that the examined smart contracts are not resistant to external/internal exploit. For a detailed understanding of risk severity, source code vulnerability, and potential attack vectors, refer to the complete audit report below.



# Project Overview

| Topaz Finance | __Project__                                                                                               |
|--------------|----------------------------------------------------------------------------------------------------------|
| Language     | Solidity                                                                                                 |
| Codebase     | https://github.com/owenThurm/TopazFinance                             |
| Commit       | [edd577cd132fba9a304703900bd36bf5dbdb5830](https://github.com/owenThurm/TopazFinance/tree/edd577cd132fba9a304703900bd36bf5dbdb5830) |


| Delivery Date     | DD - Month - YY       |
|-------------------|--------------------------------|
| Audit Methodology | Manual Review |


| Vulnerability Level | Total | Pending | Declined | Acknowledged | Partially Resolved | Resolved |
|---------------------|-------|---------|----------|--------------|--------------------|----------|
| [Critical](#Critical)| 1     | 1       | 0        | 0            | 0                  | 0        |
| [High](#High)        | 2     | 2       | 0        | 0            | 0                  | 0        |
| [Medium](#Medium)    | 3     | 3       | 0        | 0            | 0                  | 0        |
| [Low](#Low)          | 0     | 0       | 0        | 0            | 0                  | 0        |

# Audit Scope & Methodology

## Scope
src/protocol/*

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
| [C-01](#C01)  | SnapshotLib::write() cardinality increased even if a new index was not written | Logic Error | CRITICAL | PENDING |
| [H-01](#H01)  | Snapshot system can be exploited with instant balance change | Manipulation | HIGH | PENDING |
| [H-02](#H02)  | claimBoost can break the user's boost if called before claiming points | Logic Error | HIGH | PENDING |
| [M-01](#M01)  | MarketLiquidation::settleLiquidation() user loses more collateral than intended | Logic Error | MEDIUM | PENDING |
| [M-02](#M02)  | Market::calculateMarketState() state not updated before pause | Logic Error | MEDIUM | PENDING |
| [M-03](#M03)  | Market::setDiscountModel() sets model before liabilities accrue | Logic Error | MEDIUM | PENDING |




## <a id="Critical"></a>Critical

### <a id="C01"></a> C-01 SnapshotLib::write() cardinality increased even if a new index was not written
https://github.com/owenThurm/TopazFinance/blob/edd577cd132fba9a304703900bd36bf5dbdb5830/src/protocol/vault/SnapshotLib.sol#L55

#### Description:
In the write() function the cardinality is always increased as long as the size is bigger than the cardinality. It however doesn't check if a new index was written or not.

-----------------

## <a id="High"></a> High

### <a id="H01"></a> H-01 Snapshot system can be exploited with instant balance change

#### Description:
In order to prevent manipulation through instant balance changes snapshots are used. However the system only looks at the latest snapshot. A user can deposit and manually call `takeSnapshot` and now that will reflect his latest balance.

-----------------

### <a id="H02"></a> H-02 claimBoost can break the user's boost if called before claiming points

#### Description:
The user boost is supposed to be 1x and it can be increased to 1.1x with `claimBoost`. If the user burns the Topaz Token with `burnTokens` before calling `claimPoints` and then they call `claimBoost` the boost will be 0.1x instead of 1x

-----------------

## <a id="Medium"></a> Medium

### <a id="M01"></a> M-01 MarketLiquidation::settleLiquidation() user loses more collateral than intended
https://github.com/owenThurm/TopazFinance/blob/edd577cd132fba9a304703900bd36bf5dbdb5830/src/protocol/market/MarketLiquidation.sol#L233

#### Description:
Settling liquidations is based on the user's liabilities. 

`uint256 repayAmount = Math.min(context.totalAmount, context.liabilities);`

The repay amount can be decreased but the amount withdrawn from the user's supply is still the original supply. This means that theres an additional liquidation fee that the user is not aware of causing loss of funds.

-----------------

### <a id="M02"></a> M-02 Market::calculateMarketState() state not updated before pause

#### Description:
When the market is paused that means that interest is no longer accrued for positions in `calculateMarketState`. Pending interest is also not accrued which means that this accrued interest is basically lost because the following calls to `accrueLiabilities` will update the market state timestamp without accruing interest.

-----------------

### <a id="M03"></a> M-03 Market::setDiscountModel() sets model before liabilities accrue
https://github.com/owenThurm/TopazFinance/blob/edd577cd132fba9a304703900bd36bf5dbdb5830/src/protocol/market/Market.sol#L111

#### Description:
When setting a new discount model before the interest is updated with `accrueLiabilities` then the pending interest is treated as if it had accrued under the new discount model instead of the old one.

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

