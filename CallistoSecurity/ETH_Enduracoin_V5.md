# EnduracoinToken v1.0.5.10152023 Security Audit Report

# 1. Summary

[EnduracoinToken v1.0.5.10152023](https://github.com/Enduracoin/EnduracoinToken/tree/main) smart contract security audit report performed by [Callisto Security Audit Department](https://github.com/CallistoSecurity/Smart-contract-auditing)

# 2. In scope

Commit [f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b](https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/)

- EnduracoinToken.sol
- EnduracoinValue.sol
- ManageApprovers.sol
- StringHelpers.sol
- ChangeRequests.sol

# 3. Findings

In total, **2 issues** were reported, including:

- 0 high severity issues.

- 1 medium severity issue.

- 1 low severity issue.

In total, **2 notes** were reported, including:

- 1 minor observation.

- 1 owner privilege.



## 3.1. Token functionality may be blocked for all users for the voting period

### Severity: medium

### Description

All major functions (`transfer`, `approve`, `transferFrom`, etc) of EnduracoinToken have the modifier `requiresMultiSig()`. 
This modifier applies to check vote results if `pendingChange.addrValueToChange == owner()`. It means that any approver who creates `newChangeRequest` with the parameter `addressToUse` equal to the owner address will block all tokens functionality until voting is finished.

### Code snipped

- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinToken.sol#L112
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/ChangeRequests.sol#L69
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinToken.sol#L72


## 3.2. Owner privileges

### Severity: owner privileges

### Description

50 Billion Enduracoin will be pre-minted to the owner's wallet. If tokens are burnt, the owner has the right to mint new tokens up to 50 Billion in total supply.


## 3.3. The transfer ownership does not remove the old owner's address from the approver list  

### Severity: low

### Description

In the function `execChangeRequest` when `ChangeType` is `transferOwnership` and if a new owner address has already been added to the approver list, the old address will not be removed from approvers. Only if a new owner isn't approved it will be added to the list and the old address will be removed.

### Code snipped

- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinToken.sol#L45-L47
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinToken.sol#L49-L50

### Recommendation

Rewrite this part: https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinToken.sol#L44-L53

like this:

```Solidity
            else if(pendingChangeType == uint8(ChangeType.transferOwnership)) { 
                if(validateApproverExists(owner())) 
                    _removeApprover(owner());
                if(!validateApproverExists(pendingChange.addrValueToChange)) 
                    _setApprover(pendingChange.addrValueToChange);
                transferOwnership(pendingChange.addrValueToChange); 
            }
```

## 3.4. Multiple minor observation

### Severity: minor observation

### Description

1. The modifier `requiresMultiSig` restricts `owner` from calling the function until voting is finished, but allows anybody else to call the function without restriction. Therefore, in the context of the contract `EnduracoinValue` it does not make any sense and can be removed or replaced by the modifier `onlyApprovers`.

- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinValue.sol#L166
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinValue.sol#L180
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinValue.sol#L189

2. The `getPendingChangeRequest()` is a view function, so does not require `onlyApprovers` modifier.
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/ChangeRequests.sol#L152

3. Owner can't create a change request for `transfer`, `approve`, `transferFrom`, `increaseAllowance` and must be the last approver in voting. If the owner votes earlier the change request can't be approved and all other approvers should vote against it to be able to create a new request. 
It creates an opportunity for a frontrunning attack by a malicious approver, where he can flip his vote just before the owner's vote (so the owner will not be the last approver).

4. In the `EnduracoinValue` contract the comment says: "`LASTDAY_OF_YEAR21` represents the last day of year 21 from the start date.", but value 8030 = 22 * 365. So it's the last day of year 22. Also, this number does not take into account years with 366 days.
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinValue.sol#L16-L17

5. In the `EnduracoinValue` contract the `currentDayInPeriod` should be in range from 0 to 364. Therefore condition `if(daysSinceStart > 365)` should be changed to `if(daysSinceStart >= 365)`. or can be removed at all.
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinValue.sol#L60

6. The modifier `requiresMultiSig()` has a check `(msg.sender == owner() && approverAddresses[msg.sender])`. The condition `approverAddresses[msg.sender]` will always be `True` if the `msg.sender` is the `owner()` address. Therefore the condition `approverAddresses[msg.sender]` can safely be removed.
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/ChangeRequests.sol#L69C12-L69C68

7. Change request types `setBaseValueAdjustment`, and `setDailyValueGainAdjustment` are unused in the contracts and can be removed.
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/ChangeRequests.sol#L42-L43

8. The `require` statement would never revert and is redundant since the contract would never be in a state where the function `transferOwnership()` would be directly accessible.
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinValue.sol#L168

9. Condition in the require statement would always be `False`, as the contract cannot access the function `removeApprover()` for the change type `ChangeType.renounceOwnership`.
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinValue.sol#L192C12-L192C68
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinToken.sol#L162C12-L162C68

9. Use custom errors to optimize gas usage and reduce deployment costs.

Starting from [Solidity v0.8.4](https://github.com/ethereum/solidity/releases/tag/v0.8.4), there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., revert("Insufficient funds.");), but they are rather expensive, especially when it comes to deployment cost, and it is difficult to use dynamic information in them.

10. When a pending change is executed in `EnduracoinValue` contract the `executionResult` string is set to "Change Opposed!", but even if the change was successfully executed the event `ChangeRequestExecutedEvent` would be emitted with the opposed string instead of a success string. 
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinValue.sol#L147
- https://github.com/Enduracoin/EnduracoinToken/blob/f1a046ffb4a9fc42236b4b80b1de5b8bb84e034b/EnduracoinValue.sol#L154


# 4. Security practices

- [ ] **Open-source contact**.
- [ ] **The contract should pass a bug bounty after the completion of the security audit.**
- [ ] **Public testing.**
- [ ] **Automated anomaly detection systems.** - NOT IMPLEMENTED. A simple anomaly detection algorithm is recommended to be implemented to detect behavior that is atypical compared to normal for this contract. For instance, the contract must halt deposits if a large amount is being withdrawn quickly until the owner or the project community approves further operations.
- [ ] **Multisig owner account.**
- [ ] **Standard ERC20-related issues.** - NOT IMPLEMENTED. Every contract can potentially receive an unintended ERC20-token deposit without the ability to reject it, even if the contract is not intended to receive or hold tokens. As a result, it is recommended to implement a function that will allow extracting any arbitrary number of tokens from the contract.
- [ ] **Crosschain address collisions.** ETH, ETC, CLO, etc. It is possible that a transaction can be sent to the address of your contract at another chain (as a result of a user mistake or some software fault). It is recommended that you deploy a "mock contract" that would allow you to withdraw any tokens from that address or prevent any funds deposits. Note that you can reject transactions of native tokens deposited, but you can not reject the deposits of ERC20 tokens. You can use this source code as a mock contract: [extractor contract source code](https://github.com/EthereumCommonwealth/GNT-emergency-extractor-contract/blob/master/extractor.sol). The address of a new contract deployed using `CREATE (0xf0)` opcode is assigned following this scheme `keccak256(rlp([sender, nonce]))`. Therefore you need to use the same address that was originally used at the main chain to deploy the mock contract at a transaction with the `nonce` that matches that on the original chain. _Example: If you have deployed your main contract with address 0x010101 at your 2021st transaction, then you need to increase your nonce of 0x010101 address to 2020 at the chain where your mock contract will be deployed. Then you can deploy your mock contract with your 2021st transaction, and it will receive the same address as your mainnet contract._

# 5. Conclusion

The audited smart contract can't be deployed. Pointed issues should be fixed before deployment.

Users should pay attention to the contract owner's rights.

It is recommended to adhere to the security practices described in pt. 4 of this report to ensure the contract's operability and prevent any issues that are not directly related to the code of this smart contract.
