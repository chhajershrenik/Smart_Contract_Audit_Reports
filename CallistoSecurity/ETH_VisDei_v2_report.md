# VIS-DEI Security Audit Report v2

# 1. Summary

[VIS-DEI](https://github.com/VIS-DEI/smart-contracts-public/tree/main/contracts) smart contract security audit report performed by [Callisto Security Audit Department](https://github.com/CallistoSecurity/Smart-contract-auditing)

# 2. In scope

Commit: `93135602752eff72097f602dae44b0d1bed5986d`

generic:
- [TimeMultisig.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/generic/TimeMultisig.sol)
- [Pausable.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/generic/Pausable.sol)

staking:
- [Staking.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/staking/Staking.sol)
- [IStaking.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/staking/IStaking.sol)

token:
- [Launchpad.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/token/Launchpad.sol)
- [Vis.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/token/Vis.sol)

vesting:
- [VestingFactory.sol](https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/vesting/VestingFactory.sol)


# 3. Findings

In total, **1 issue** were reported, including:

- 0 high severity issues.

- 1 medium severity issue.

- 0 low severity issues.

In total, **5 notes** were reported, including:

- 1 minor observation.

- 4 owner privileges.


## 3.1. Remove owner issues

### Severity: medium

### Description

There are multiple issues in the function `removeOwner`:
1. The `ownersLength` initiated before removing the owner, so condition `if (requiredApprovals > ownersLength)` will be always `false` since you compare `requiredApprovals` with the old `ownersLength`. 
2. To change requirements you call `changeRequirement(ownersLength);` which will fail, because it requires enough approvals.

### Code snippet

- https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/generic/TimeMultisig.sol#L159
- https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/generic/TimeMultisig.sol#L167
- https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/generic/TimeMultisig.sol#L168

### Recommendation

Replace this part https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/generic/TimeMultisig.sol#L166-L169 with:
```Solidity
        ownersLength--;
        if (requiredApprovals > ownersLength){
            _changeRequirement(ownersLength);
        }
```

## 3.2. For dust transactions, the current price would return zero allowing arbitrary wallets to buy tokens without paying.

### Severity: minor observation

### Description

For dust amount of tokens, the function `currentPrice()` would always be zero allowing wallets to buy tokens without paying any USDT tokens. 

### Code Snippet
- https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/token/Launchpad.sol#L33-L43

### Recommendation

Add in the and of function [currentPrice](https://github.com/VIS-DEI/smart-contracts-public/blob/93135602752eff72097f602dae44b0d1bed5986d/contracts/token/Launchpad.sol#L43) following requirement:
```Solidity
    require(price != 0, "Too small amount");
```

## 3.3. Owner privileges

### Severity: owner privileges

### Description

1. Any owner can Pause/unpause Staking and Launchpad contracts.
2. Set interest in a Staking contract.
3. Withdraw tokens from the Staking contract (including users' deposits).
4. Withdraw tokens from Launchpad contract.

# 4. Security practices

- [x] **Open-source contact**.
- [ ] **The contract should pass a bug bounty after the completion of the security audit.**
- [ ] **Public testing.**
- [ ] **Automated anomaly detection systems.** - NOT IMPLEMENTED. A simple anomaly detection algorithm is recommended to be implemented to detect behavior that is atypical compared to normal for this contract. For instance, the contract must halt deposits in case a large amount is withdrawn in a short period until the owner or the community of the contract approves further operations.
- [ ] **Multisig owner account.**
- [x] **Standard ERC20-related issues.** - IMPLEMENTED. It is known that every contract can potentially receive an unintended ERC20-token deposit without the ability to reject it even if the contract is not intended to receive or hold tokens. As a result, it is recommended to implement a function that will allow extracting any arbitrary number of tokens from the contract.
- [ ] **Crosschain address collisions.** ETH, ETC, CLO, etc. It is possible that a transaction can be sent to the address of your contract at another chain (as a result of a user mistake or some software fault). It is recommended that you deploy a "mock contract" that would allow you to withdraw any tokens from that address or prevent any funds deposits. Note that you can reject transactions of native tokens deposited, but you can not reject the deposits of ERC20 tokens. You can use this source code as a mock contract: [extractor contract source code](https://github.com/EthereumCommonwealth/GNT-emergency-extractor-contract/blob/master/extractor.sol). The address of a new contract deployed using `CREATE (0xf0)` opcode is assigned following this scheme `keccak256(rlp([sender, nonce]))`. Therefore you need to use the same address that was originally used at the main chain to deploy the mock contract at a transaction with the `nonce` that matches that on the original chain. _Example: If you have deployed your main contract with address 0x010101 at your 2021th transaction then you need to increase your nonce of 0x010101 address to 2020 at the chain where your mock contract will be deployed. Then you can deploy your mock contract with your 2021th transaction, and it will receive the same address as your mainnet contract._


# 5. Conclusion

The audited smart contract must not be deployed. Reported issues must be fixed before the usage of this contract.

Staking contract do not guarantee users that they will be able to withdraw their deposits if tokens for interest are not added by the owner.

It is recommended to adhere to the security practices described in pt. 4 of this report to ensure the contract's operability and prevent any issues that are not directly related to the code of this smart contract.
