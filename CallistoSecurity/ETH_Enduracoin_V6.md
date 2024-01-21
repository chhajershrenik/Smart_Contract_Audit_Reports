# EnduracoinToken v1.0.5 Security Audit Report

# 1. Summary

[EnduracoinToken](https://github.com/Enduracoin/EnduracoinToken/tree/main) smart contract security audit report performed by [Callisto Security Audit Department](https://github.com/CallistoSecurity/Smart-contract-auditing)

# 2. In scope

Commit [b9a32fc37a5a81748f68e42a3dcd6590cee08035](https://github.com/Enduracoin/EnduracoinToken/blob/b9a32fc37a5a81748f68e42a3dcd6590cee08035/)

- EnduracoinToken.sol
- EnduracoinValue.sol
- ManageApprovers.sol
- StringHelpers.sol
- ChangeRequests.sol

# 3. Findings

In total, **0 issues** were reported, including:

- 0 high severity issues.

- 0 medium severity issues.

- 0 low severity issues.

In total, **2 notes** were reported, including:

- 1 minor observation.

- 1 owner privilege.



## 3.1. Owner privileges

### Severity: owner privileges

### Description

50 Billion Enduracoin will be pre-minted to the owner's wallet.


## 3.2. A few minor observation

### Severity: minor observation

### Description

1. The `getPendingChangeRequest()` is a view function, so does not require `onlyApprovers` modifier.
- https://github.com/Enduracoin/EnduracoinToken/blob/b9a32fc37a5a81748f68e42a3dcd6590cee08035/ChangeRequests.sol#L152

2. Owner must be the final approver in voting for `transfer`, `approve`, `transferFrom`, `increaseAllowance`. If the owner votes earlier the change request can't be approved and all other approvers should vote against it to be able to create a new request. 
It creates an opportunity for a frontrunning attack by a malicious approver, where he can flip his vote just before the owner's vote (so the owner will not be the last approver).



# 4. Security practices

- [ ] **Open-source contact**.
- [ ] **The contract should pass a bug bounty after the completion of the security audit.**
- [ ] **Public testing.**
- [ ] **Automated anomaly detection systems.** - NOT IMPLEMENTED. A simple anomaly detection algorithm is recommended to be implemented to detect behavior that is atypical compared to normal for this contract. For instance, the contract must halt deposits if a large amount is being withdrawn quickly until the owner or the project community approves further operations.
- [ ] **Multisig owner account.**
- [ ] **Standard ERC20-related issues.** - NOT IMPLEMENTED. Every contract can potentially receive an unintended ERC20-token deposit without the ability to reject it, even if the contract is not intended to receive or hold tokens. As a result, it is recommended to implement a function that will allow extracting any arbitrary number of tokens from the contract.
- [ ] **Crosschain address collisions.** ETH, ETC, CLO, etc. It is possible that a transaction can be sent to the address of your contract at another chain (as a result of a user mistake or some software fault). It is recommended that you deploy a "mock contract" that would allow you to withdraw any tokens from that address or prevent any funds deposits. Note that you can reject transactions of native tokens deposited, but you can not reject the deposits of ERC20 tokens. You can use this source code as a mock contract: [extractor contract source code](https://github.com/EthereumCommonwealth/GNT-emergency-extractor-contract/blob/master/extractor.sol). The address of a new contract deployed using `CREATE (0xf0)` opcode is assigned following this scheme `keccak256(rlp([sender, nonce]))`. Therefore you need to use the same address that was originally used at the main chain to deploy the mock contract at a transaction with the `nonce` that matches that on the original chain. _Example: If you have deployed your main contract with address 0x010101 at your 2021st transaction, then you need to increase your nonce of 0x010101 address to 2020 at the chain where your mock contract will be deployed. Then you can deploy your mock contract with your 2021st transaction, and it will receive the same address as your mainnet contract._

# 5. Conclusion

The audited smart contract can be deployed. No security issues were found during the audit.

It is recommended to adhere to the security practices described in pt. 4 of this report to ensure the contract's operability and prevent any issues that are not directly related to the code of this smart contract.
