# VIS-DEI Security Audit Report

# 1. Summary

[VIS-DEI](https://github.com/VIS-DEI/smart-contracts-public/tree/main/contracts) smart contract security audit report performed by [Callisto Security Audit Department](https://github.com/CallistoSecurity/Smart-contract-auditing)

# 2. In scope

Commit: `78c02637bc376e5b7a109f180928080fd082ea19`

generic:
- [TimeMultisig.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/TimeMultisig.sol)
- [Pausable.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/Pausable.sol)

staking:
- [Staking.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/staking/Staking.sol)
- [IStaking.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/staking/IStaking.sol)

token:
- [Launchpad.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/token/Launchpad.sol)
- [Vis.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/token/Vis.sol)

vesting:
- [VestingFactory.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/vesting/VestingFactory.sol)


# 3. Findings

In total, **7 issues** were reported, including:

- 2 high severity issues.

- 4 medium severity issues.

- 1 low severity issues.

In total, **8 notes** were reported, including:

- 4 minor observations.

- 4 owner privileges.

## 3.1. Known vulnerabilities of ERC-20 token

### Severity: low

### Description

Lack of transaction handling mechanism issue. [WARNING!](https://gist.github.com/Dexaran/ddb3e89fe64bf2e06ed15fbd5679bd20)  This is a very common issue, and it already caused millions of dollars in losses for lots of token users! More details [here](https://docs.google.com/document/d/1Feh5sP6oQL1-1NHi-X1dbgT3ch2WdhbXRevDN681Jv4/edit).

### Recommendation

Add the following code to the [Vis.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/token/Vis.sol)

```Solidity
    function transfer(address to, uint256 value) public override returns (bool) {
        require( to != address(this) );
        supper.transfer(to, value);
    }
```

## 3.2. Owners should vote for specific action

### Severity: medium

### Description

When owners "vote" they call the function `approve()` but do not specify what exactly they approve. If there are enough approves (required number - 1), the last owner can call any function that requires `enoughApprovals`.

It will allow any owner (or hacker if he gets the owner's private key) to execute any restricted function or gain full control over all contracts if he `changeRequirement` to 1 vote, and then remove all other owners. 

### Code snippet

- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/TimeMultisig.sol#L121
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/TimeMultisig.sol#L56

### Recommendation

Each owner should specify for what action they vote.

## 3.3. Any owner can disrupt the vote

### Severity: medium

### Description

Any owner (or hacker if he gets the owner's private key) can disrupt the vote by calling the function `revokeAll()` before reaching the required number of approvals.

### Code snippet

- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/TimeMultisig.sol#L133

## 3.4. There is no protection from withdrawing other users' tokens

### Severity: high

### Description

When users call functions `claimInterest()` or `withdrawStaking()` they withdraw tokens from contract balance if it has enough. But there is no checking if the contract has enough tokens to allow all users to withdraw their stakings. So, if the contract does not receive enough tokens for interest, the interest will be paid from other users' deposits, like in [Ponzi scheme](https://en.wikipedia.org/wiki/Ponzi_scheme). 

### Code snippet

- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/staking/Staking.sol#L109-L123
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/staking/Staking.sol#L125-L142

### Recommendation

Should be counted amount available for paying interest and bonuses, and don't allow to receive more rewards the available in the rewards pool.


## 3.5. The USDT tokens on ETH chain can't be transferred using ERC20 functions

### Severity: medium

### Description

The USDT token on ETH chain does not follow ERC20 standard functions declaration. So it can't be transferred using `transfer` and `transferFrom` from `IERC20.sol`.
Instead of it, you should use `safeTransfer` and `safeTransferFrom` from [SafeERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol) library.

Also, recommended to use `SafeERC20` wherever you transfer third-party tokens.

### Code snippet

- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/token/Launchpad.sol#L40
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/token/Launchpad.sol#L63
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/staking/Staking.sol#L167


## 3.6. Wrong price calculation

### Severity: high

### Description

When the user buys a token, the amount of USDT he should pay is calculated in the function `currentPrice`.
But this function does not take into account token decimals. Since Vis token has 18 decimals the function `currentPrice` returns a number with at least 18 decimals as well, but USDT token has 6 decimals. It means that 1 Vis token will cost 1 000 000 000 000 USDT (at least). The actual price will be higher because `startPrice` and `endPrice` also should have decimals (you should add in the description how many decimals have `startPrice` and `endPrice`).

### Code snippet

- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/token/Launchpad.sol#L39-L40
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/token/Launchpad.sol#L35


## 3.7. No upper boundary of the `gracePeriod`

### Severity: medium

### Description

In the `_changeGracePeriod` function, there is no check of the upper boundary of the `_newGracePeriod`. In case the `gracePeriod` is too large (bigger than `block.timestamp`) the approval check `if (approvals[owners[i]] > block.timestamp - gracePeriod)` will revert due to underflow protection. It will cause "deadlock" making it unable to call any function with `enoughApprovals` modifier.


### Code Snippet

- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/TimeMultisig.sol#L72
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/TimeMultisig.sol#L42

### Recommendation
It is recommended to add a restriction on the upper value of the `gracePeriod`.


## 3.8. The vesting contract does not incentivize the wallet which locks the funds.

### Severity: minor observation

### Description

The `VestingFactory` contract allows wallets to lock `vis` tokens of a wallet for up to a maximum period of 4 years.
Upon locking the tokens the tokens are unlocked after the `unlockTimestamp`, but the tokens are withdrawn to `withdrawalAddress` and not to the wallet which is locking the tokens. Additionally, wallets are not Incentivized for long-term participation and the amount of tokens which is released is equal to the amount of tokens unlocked after the vesting period.

### Code Snippet

- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/vesting/VestingFactory.sol#L31-L32
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/vesting/VestingFactory.sol#L79-L83

### Recommendation

Consider implementing a function to allow wallets to withdraw the tokens directly after the vesting period is ended, also consider incentivizing wallets based on the vesting period and re-implement `withdraw()` to be able to gradually unlock the token's withdrawable amount based on the elapsed time since the lock, the unlock rate (e.g., percentage unlocked per year), and the locked amount.

## 3.9. There isn't a description of interest decimals

### Severity: minor observation

### Description

Specified interests do not have a description of what they should be, (50% and 75%) or (5% and 7.5%).

```Solidity
    uint public YEAR_1_INTEREST = 50;
    uint public YEAR_2_INTEREST = 75;
```

Accordingly contract logic is 5% and 7.5%, but you should check if it correct value.

### Code snippet

- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/staking/Staking.sol#L13-L14
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/staking/Staking.sol#L57
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/staking/Staking.sol#L62


## 3.10. Multiple minor observations

### Severity: minor observation

### 1. Current `gracePeriod` conditions could lead to unexpected reverts and require re-approvals.

In the contract `TimeMultisig` the variable `gracePeriod` is the period within which approvals must be submitted. Based on the required statement in the function `_changeGracePeriod()` the `gracePeriod` can be set to any arbitrary value above 60 seconds and based on the dev comments the current recommended `gracePeriod` is 10 minutes (600 seconds).

In Ethereum transactions can take several minutes to hours even with high gas prices during high congestion periods. The `gracePeriod` is a fixed value set by the contract and doesn't adapt to dynamic network conditions and the total number of `requiredApprovals` required could result in previous approvals by owners being discarded reverting the approved transaction by the owners since the confirmation time for transactions could exceeds the `gracePeriod`.

### Code Snippet
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/TimeMultisig.sol#L42-L44
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/TimeMultisig.sol#L72

### Recommendation

Consider updating the minimum requirement for `gracePeriod` to 30 mins (1800) in the require statement in the function `_changeGracePeriod()` to allow sufficient time for approvals and executing the transaction.



### 2. Wasting gas in `for` loop in TimeMultisig contract

Using `owners.length` in `for` condition read data from storage on each iteration. To save gas better to use a local variable for length.
Also, removing the initialization `uint i = 0` saves a little bit of gas too. 

### Code snippet

- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/TimeMultisig.sol#L41
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/TimeMultisig.sol#L90
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/TimeMultisig.sol#L108
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/generic/TimeMultisig.sol#L134

### Recommendation

Use
```Solidity
    uint256 length = owners.length;
    for (uint i; i<length - 1; i++)
```

### 3. Wasting gas in `for` loop in Staking contract

Using `lastDepositedBonusIndex` in `for` condition read data from storage on each iteration. The calculation `uint stakingEndsAt = acc.depositTimestamp + (YEAR * acc.period);` should be done out of the loop.

### Code snippet
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/staking/Staking.sol#L84-L90

### Recommendation

Use
```Solidity
        uint256 _lastDepositedBonusIndex = lastDepositedBonusIndex;
        uint stakingEndsAt = acc.depositTimestamp + (YEAR * acc.period);
        for (uint i = acc.lastWithdrawnBonusIndex + 1; i <= _lastDepositedBonusIndex; i++) {
```

### 4. ReentrancyGuard can be omitted

Vis and USDT token is ERC20 token and doesn't contain a callback, so reentrancy is not possible. You can remove `ReentrancyGuard` to reduce deployment costs. 

### Code snippet
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/staking/Staking.sol#L5
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/token/Launchpad.sol#L5


### 5. The vesting contract does not incentivize the wallet which locks the funds.

The `VestingFactory` contract allows wallets to lock `vis` tokens of a wallet for up to a maximum period of 4 years.
Upon locking the tokens the tokens are unlocked after the `unlockTimestamp`, but the tokens are withdrawn to `withdrawalAddress` and not to the wallet which is locking the tokens. Additionally, wallets are not Incentivized for long-term participation and the amount of tokens which is released is equal to the amount of tokens unlocked after the vesting period.

### Code Snippet

- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/vesting/VestingFactory.sol#L31-L32
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/vesting/VestingFactory.sol#L79-L83

### Recommendation

Consider implementing a function to allow wallets to withdraw the tokens directly after the vesting period is ended, also consider incentivizing wallets based on the vesting period and re-implement `withdraw()` to be able to gradually unlock the token's withdrawable amount based on the elapsed time since the lock, the unlock rate (e.g., percentage unlocked per year), and the locked amount.

### 6. AccessControl is unused

`AccessControl` is imported but never used for access restriction. To reduce deployment cost is better to remove it. 

### Code snippet

- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/vesting/VestingFactory.sol#L5
- https://github.com/VIS-DEI/smart-contracts-public/blob/78c02637bc376e5b7a109f180928080fd082ea19/contracts/vesting/VestingFactory.sol#L25



## 3.11. Owner privileges

### Severity: owner privileges

### Description

1. Any owner can Pause/unpause Staking and Launchpad contracts.
2. Set interest in Staking contract.
3. Withdraw tokens from Staking contract (including users' deposits).
4. Withdraw tokens from Launchpad contract.

Following owner privileges is dangerous due to the issue in TimeMultisig.
- Set withdrawal address in VestingFactory contract.
- Change approval requirement.
- Change grace period.
- Add/remove/replace owner.


## 3.12. Follow good coding practice

### Severity: minor observation

### Description

1. Unlocked Pragma.

Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the floating pragma, i.e. by not using ^ in pragma solidity ^0.8.4, ensures that contracts do not accidentally get deployed using a compiler version with unfixed bugs.

2. Missing docstrings.

The contracts in the code base lack documentation. This hinders reviewers’ understanding of the code’s intention, which is fundamental to correctly assessing not only security but also correctness. Additionally, docstrings improve readability and ease maintenance. They should explicitly explain the purpose or intention of the functions, the scenarios under which they can fail, the roles allowed to call them, the values returned, and the events emitted.

Consider thoroughly documenting all functions (and their parameters) that are part of the contracts’ public API. Functions implementing sensitive functionality, even if not public, should be documented as well. When writing docstrings, consider following the Ethereum Natural Specification Format (NatSpec).

3. Missing test suite.

The contract is missing a test suite to validate and verify the behavior of the contract functionalities. Add tests are recommended to ensure that the contract functions and behaves as expected.



# 4. Security practices

- [x] **Open-source contact**.
- [ ] **The contract should pass a bug bounty after the completion of the security audit.**
- [ ] **Public testing.**
- [ ] **Automated anomaly detection systems.** - NOT IMPLEMENTED. A simple anomaly detection algorithm is recommended to be implemented to detect behavior that is atypical compared to normal for this contract. For instance, the contract must halt deposits in case a large amount is withdrawn in a short period until the owner or the community of the contract approves further operations.
- [ ] **Multisig owner account.**
- [ ] **Standard ERC20-related issues.** - NOT IMPLEMENTED. It is known that every contract can potentially receive an unintended ERC20-token deposit without the ability to reject it even if the contract is not intended to receive or hold tokens. As a result, it is recommended to implement a function that will allow extracting any arbitrary number of tokens from the contract.
- [ ] **Crosschain address collisions.** ETH, ETC, CLO, etc. It is possible that a transaction can be sent to the address of your contract at another chain (as a result of a user mistake or some software fault). It is recommended that you deploy a "mock contract" that would allow you to withdraw any tokens from that address or prevent any funds deposits. Note that you can reject transactions of native tokens deposited, but you can not reject the deposits of ERC20 tokens. You can use this source code as a mock contract: [extractor contract source code](https://github.com/EthereumCommonwealth/GNT-emergency-extractor-contract/blob/master/extractor.sol). The address of a new contract deployed using `CREATE (0xf0)` opcode is assigned following this scheme `keccak256(rlp([sender, nonce]))`. Therefore you need to use the same address that was originally used at the main chain to deploy the mock contract at a transaction with the `nonce` that matches that on the original chain. _Example: If you have deployed your main contract with address 0x010101 at your 2021th transaction then you need to increase your nonce of 0x010101 address to 2020 at the chain where your mock contract will be deployed. Then you can deploy your mock contract with your 2021th transaction, and it will receive the same address as your mainnet contract._


# 5. Conclusion

The audited smart contract must not be deployed. Reported issues must be fixed before the usage of this contract.

TimeMultisig contract does not provide the required security and allows any owner to get full control over important functions.

Staking contract do not guarantee users that they will be able to withdraw their deposits, at least. 

It is recommended to adhere to the security practices described in pt. 4 of this report to ensure the contract's operability and prevent any issues that are not directly related to the code of this smart contract.
