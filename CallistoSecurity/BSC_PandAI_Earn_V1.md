# PandAIEarn Security Audit Report

# 1. Summary

[PandAI Earn](https://github.com/pandai-corp/pandai-earn-sc/tree/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d) smart contract security audit report performed by [Callisto Security Audit Department](https://github.com/CallistoSecurity/Smart-contract-auditing/)


# 2. In scope

Commit: 5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d

- [PandAI.sol](https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/PandAI.sol)
- [PandAIEarn.sol](https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/PandAIEarn.sol)
- [USDT.sol](https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/USDT.sol)

# 3. Findings

In total, **5 issues** were reported, including:

- 0 critical severity issues.

- 0 high severity issues.

- 3 medium severity issues.

- 2 low severity issues.

In total, **10 notes** were reported, including:

- 5 notes.

- 5 owner privileges.

## 3.1. Claim should be called within depositWithReferral and requestWithdraw

### Severity: medium

### Description

Eventhough the comments mentions the risk that rewards may be lost if claim is not called prior to calling `depositWithReferral` and `requestWithdraw`. Therefor pnding rewards should be stored (for exapmlple, in `userMap[msg.sender].referralPendingReward` or create separate variable) within these function at the beginning so that users 
never experience this loss in rewards. A user might not read carefully/neglect the guidlines/comments and hence lose rewards.

### Code Snippet

- https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/PandAIEarn.sol#L210-L215
- https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/PandAIEarn.sol#L241-L247



## 3.2. Flash loan Price Manipulation attack can Inflate/deflate the value of PandAI/USDT

### Severity: medium

### Description

The function `getPandaiWorthOf` is used to calculate the Pandai worth per USDT, and is used in functions to calculate
fee. 

Since this function checks tokens balances of `lpAddress` it allows attaker to use flash loan price manipulation to pay very small fee.

For example, using intermidiate contract an attaker can:
1. Use flash loan to drain tokens from one side (i.e. Pandai token) of LP pool
2. Call `PandAiEarn` contract to claim or withdraw tokens. Now function `getPandaiWorthOf` returns very high price of Pandai tokens so attaker pay very small amount of fee.
3. Returns flash loan.

This issue has medium severity because an attaker should pay fee for flash loan (depends on DEX it's 0.2%-0.3% of loan amount). So if pool has a lot of liquidity manipulation with price may cost more than pay normal Pandai fee.


### Code Snippet

- https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/PandAIEarn.sol#L373
- https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/PandAIEarn.sol#L259
- https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/PandAIEarn.sol#L423
- https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/PandAIEarn.sol#L438

### Recommendation

Disallow to call functions `claim()` and `requestWithdraw()` from smart contracts adding:
```solidity
require(msg.sender == tx.origin, "Calls from contract disallowed");
```

Or use a decentralised oracle like chainlink to get the price Of Pandai/USDT


## 3.3. Function depositWithReferral() does not check if the referralAddress Tier is initiated

### Severity: low

### Description

Based on the [docs](https://docs.pandai.io/products/pandai-earn#:~:text=PandAI%20Earn%20generates%20a%20unique%20Referral%20Link%20for%20any%20address%20in%20Tier%201%20or%20higher.), the referral address (`referralAddress`) should be in Tier 1 or higher. The function `depositWithReferral()` does not check if the `referralAddress` has made an existing deposit and if a Tier is being assigned to the address. If the Tier is not yet initiated for the `referralAddress` is not allowed to collect the referral bonus locking out the bonus in the contract.

### Code Snippet

- https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/PandAIEarn.sol#L210-L239

### Recommendation

Consider adding checks to verify that the `referralAddress` Tier has already been initialized.

## 3.4. Incorrect assignment of oldLpAddress in setLpAddress function

### Severity: low

### Description

In the `setLpAddress` function, `oldLpAddress` is set with `newLpAddress` instead of `lpAddress` (the current one).

```solidity
function setLpAddress(
    address newLpAddress
) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(newLpAddress != address(0), "empty address");
    require(usdtToken.balanceOf(newLpAddress) > 0, "no usdt in lp");
    require(pandaiToken.balanceOf(newLpAddress) > 0, "no pandai in lp");
    address oldLpAddress = newLpAddress;
    lpAddress = newLpAddress;
    emit LpAddressChanged(oldLpAddress, newLpAddress);
}
```

### Code Snippet

- https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/PandAIEarn.sol#L173

### Recommendation

Assign the value of lpAddress to oldLpAddress before updating lpAddress with newLpAddress.

```solidity
    address oldLpAddress = lpAddress;
```

## 3.5. Function depositWithReferral() when the amount of USDT tokens is less than $100 for any additional deposits

### Severity: medium

### Description

If a user tries to deposit any amount less than $100 the function `depositWithReferral()` reverts even if the total amount of USDT token is greater than the respective tier limit. For example, if a user deposits $500 equivalent USDT tokens in the first deposit and less than $100 worth of USDT tokens in the second deposit. Even though the total amount of USDT token meets the eligibility criteria for the respective Tier the function reverts due to the small deposit check.

### Code Snippet
- https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/PandAIEarn.sol#L216

### Recommendation

Consider adding checks to prevent the function from reverting if a user has an existing deposit and allowing a deposit of less than $100 if the total amount of the deposit meets the respective Tier criteria.


## 3.6. Condition at Line 273 will always hold true

### Severity: note

### Description

`userMap[msg.sender].deposit` increases when a user deposits and so does the `userMap[userMap[msg.sender].referral].referralDeposit` 
with the same value (See line 226 and 232). referralDeposit increases when a user using that referral deposits , this means
referral deposit would at least be equal to `userMap[msg.sender].deposit` (considering only msg.sender made the deposit using the referral).

Here https://github.com/pandai-corp/pandai-earn-sc/blob/main/contracts/PandAIEarn.sol#L248 we check if user's deposit is at least
what the user is trying to withdraw ,  this condition should also automatically imply referral deposit is also atleast the
withdraw amount (according to the above paragraph).

Therefore the condition here https://github.com/pandai-corp/pandai-earn-sc/blob/main/contracts/PandAIEarn.sol#L273 would be 
always true, or in other words no need to check `userMap[userMap[msg.sender].referral].referralDeposit >= usdtWithdrawAmount`

### Code Snippet

- https://github.com/pandai-corp/pandai-earn-sc/blob/main/contracts/PandAIEarn.sol#L273

## 3.7. usdtToken and pandaiToken should be immutable

### Severity: note

### Description

`usdtToken` and `pandaiToken` are declared as public state variables. This means they are stored in the contract's storage. However, since their values are set in the constructor and never changed, they can be declared as `immutable` to save gas on storage reads.

### Code Snippet

- https://github.com/pandai-corp/pandai-earn-sc/blob/5c0fdfb6606ed9d4d86fc27861173d4fa9662f1d/contracts/PandAIEarn.sol#L18-L19

### Recommendation

Improve efficiency by declaring the variables as `immutable`. This will store their values directly in the contract bytecode rather than in storage, saving gas on reads.

```solidity
IERC20 public immutable usdtToken;
IERC20 public immutable pandaiToken;
```

## 3.8. More than 40000 gas can be saved by packing values in struct together

#### Severity: note 

Let's take example of the Tier struct , we know the max value for the minDeposit can be 10000 (tier 5) , so this can be made into
uint16 , then minDeposit and `compoundInterest` can be clubbed together into a single slot (bool takes one bit), thus saving
20000 gas. Similarly more slots can be optimized.

## 3.9. Owner Privileges

### Severity: owner privileges

### Description

1. Contract PandAIEarn inherits the traits of OpenZeppelin [AccessControl](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/AccessControl.sol) contract allowing admin to manage admin's, the role could be renounced leading to locking out of access to critical functions of the contract.
2. Owner completely responisble to maintain an adequate balance on the contract for users' withdrawals and rewards payouts. Otherwise users' will not be able to withdraw or claim.
3. Function `setLpAddress()` allows the admin to change the liquidity pool address to any wallet, the value of PandAi tokens would be affected which is computed using the function `getPandaiWorthOf()` based on the amount of USDT tokens and PandAi tokens available in `newLpAddress` wallet.
4. Function `withdrawTreasury()` allows the admin to withdraw USDT tokens from the contract to the admin address.
5. Function `setUserApprovalLevel()` allows admin to change the claim status of any wallet address to `NotApproved`, `Approved`, and `Forbidden` limiting the user's ability to withdraw or restrict a user from withdrawing USDT tokens deposited in the contract based on the approval status.


### Recommendation
Since the owner has unlimited rights to do everything, the ownership must be transferred to a multi-sig contract.

## 3.10. Follow good coding practice

### Severity: note

### Description

1. Unlocked Pragma

Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the floating pragma, i.e. by not using ^ in pragma solidity ^0.8.9, ensures that contracts do not accidentally get deployed using an older compiler version with unfixed bugs.

2. Typo in the function name

The function name `getDepositUnlokTimestamp()` is misspelled consider correcting the typo to `getDepositUnlockTimestamp()` in the contract.

3. Unorganized and non-standardized docstrings

The contracts in the code base contain unorganized and non-standardized docstring does not explain the contract's state and functionalities in detail. This hinders reviewers’ understanding of the code’s intention, which is fundamental to correctly assess not only security but also correctness. Additionally, detailed docstrings improve readability and ease maintenance. They should explicitly explain the purpose or intention of the functions, the scenarios under which they can fail, the roles allowed to call them, the values returned, and the events emitted.

Consider thoroughly documenting all functions (and their parameters) that are part of the contracts’ public API. Functions implementing sensitive functionality, even if not public, should be documented as well. When writing docstrings, consider following the Ethereum Natural Specification Format (NatSpec).

# 4. Security practices

- [x] **Open-source contact**.
- [ ] **The contract should pass a bug bounty after the completion of the security audit.**
- [ ] **Public testing.**
- [ ] **Automated anomaly detection systems.** - NOT IMPLEMENTED. A simple anomaly detection algorithm is recommended to be implemented to detect behavior that is atypical compared to normal for this contract. For instance, the contract must halt deposits in case a large amount is withdrawn in a short period until the owner or the community of the contract approves further operations.
- [ ] **Multisig owner account.**
- [ ] **Standard ERC20-related issues.** - NOT IMPLEMENTED. It is known that every contract can potentially receive an unintended ERC20-token deposit without the ability to reject it even if the contract is not intended to receive or hold tokens. As a result, it is recommended to implement a function that will allow extracting any arbitrary number of tokens from the contract.
- [ ] **Crosschain address collisions.** ETH, ETC, CLO, etc. It is possible that a transaction can be sent to the address of your contract at another chain (as a result of a user mistake or some software fault). It is recommended that you deploy a "mock contract" that would allow you to withdraw any tokens from that address or prevent any funds deposits. Note that you can reject transactions of native tokens deposited, but you can not reject the deposits of ERC20 tokens. You can use this source code as a mock contract: [extractor contract source code](https://github.com/EthereumCommonwealth/GNT-emergency-extractor-contract/blob/master/extractor.sol). The address of a new contract deployed using `CREATE (0xf0)` opcode is assigned following this scheme `keccak256(rlp([sender, nonce]))`. Therefore you need to use the same address that was originally used at the main chain to deploy the mock contract at a transaction with the `nonce` that matches that on the original chain. _Example: If you have deployed your main contract with address 0x010101 at your 2021th transaction then you need to increase your nonce of 0x010101 address to 2020 at the chain where your mock contract will be deployed. Then you can deploy your mock contract with your 2021th transaction, and it will receive the same address as your mainnet contract._


# 5. Conclusion

The audited smart contract should not be deployed. Reported issues must be fixed prior to the usage of this contract.

It is recommended to adhere to the security practices described in pt. 4 of this report to ensure the contract's operability and prevent any issues that are not directly related to the code of this smart contract.
