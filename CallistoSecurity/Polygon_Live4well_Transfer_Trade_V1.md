# Live4well Transfer and Trade Security Audit Report

# 1. Summary

[Live4well Transfer and Trade](https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit) smart contract security audit report performed by [Callisto Security Audit Department](https://github.com/CallistoSecurity/Smart-contract-auditing)


# 2. In scope


Commit: 8d1da158a15099cb81e99dcaa5a554a0ae3bae25

- [ERC1155Royalty.sol](https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/ERC1155Royalty.sol)
- [IERC1155_token.sol](https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/IERC1155_token.sol)
- [IERC721-nftPass.sol](https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/IERC721-nftPass.sol)
- [Trade.sol](https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Trade.sol)
- [Transfer.sol](https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Transfer.sol)


# 3. Findings

In total, **1 issue** were reported, including:

- 0 high severity issues.

- 1 medium severity issue.

- 0 low severity issues.

In total, **4 notes** were reported, including:

- 2 minor observations.

- 2 owner privileges.


## 3.1. `TradingERC20` contract accepts only NFTs that implemented the `royaltyInfo()` function

### Severity: medium

### Description

If the NFT contract (ERC721 or ERC1155) does not implement the `royaltyInfo()` function, the calling of this function will revert the transaction.

### Code snippet

- https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Trade.sol#L49
- https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Trade.sol#L81

### Recommendation

if you want to accept any ERC721 and ERC1155, use following code:

```Solidity
        address royaltyRecipient;
        uint256 royaltyPrice;
        try nftContract.royaltyInfo(_tokenId, _erc20Amount) returns (address _royaltyRecipient, uint256 _royaltyPrice) {
            royaltyRecipient = _royaltyRecipient;
            royaltyPrice = _royaltyPrice;
        } catch (bytes memory) {}
```

## 3.2. Owner privileges

### Severity: owner privileges

### Description

1. The owner of `TransferContract` can transfer tokens, which are approved by this contract, to any address or burn it.
2. The owner of the `TradingERC20` contract can grant `TRADER_ROLE` to any address. The address with `TRADER_ROLE` can transfer any approved tokens to any address (using fake NFT or ERC20 contracts).

### Recommendation

Since the owner has unlimited rights to do everything, the ownership must be transferred to a multi-sig contract. Users should trust to owner.


## 3.3. Multiple minor observations

### Severity: minor observation

### Description

1. The `TransferContract` and `TradingERC20` contracts import `@openzeppelin/contracts/access/Ownable.sol` (so during compilation the latest version of `Ownable.sol` is used). But in the latest version of `Ownable.sol` the constructor requires argument [address initialOwner](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4e419d407cb13c4ebd64d3f47faa78964cbbdd71/contracts/access/Ownable.sol#L38). Therefore `TransferContract` and `TradingERC20` contracts can't be compiled.

#### Code snippet

- https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Trade.sol#L5
- https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Transfer.sol#L4
- https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4e419d407cb13c4ebd64d3f47faa78964cbbdd71/contracts/access/Ownable.sol#L38

#### Recommendation

Specify the version of Openzeppelin contracts like: `import @openzeppelin/contracts@4.9.0/ ...` to avoid issues when a new version has incompatible functions.


2. The event `TradingRecord` is declared but never used. Recommend to emit this event in the functions `tradeERC721()` and `tradeERC1155()`.

#### Code snippet

- https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Trade.sol#L15-L25

3. Checking allowance, balances, and approvals of transferred tokens is not necessary, because these checks are performed in the token contract during the transfer call.

Also, ERC721 can be approved in two ways: using functions `approve` or `setApprovalForAll`, but the function `tradeERC721` checks only `getApproved` and omit `isApprovedForAll`.

#### Code snippet

- https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Trade.sol#L39-L40
- https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Trade.sol#L43-L47
- https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Trade.sol#L71-L72
- https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Trade.sol#L75-L79
- https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Transfer.sol#L16-L21
- https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Transfer.sol#L31-L32
- https://github.com/technine-IT/live4well-transfer-and-trade-smartcontractfor-audit/blob/8d1da158a15099cb81e99dcaa5a554a0ae3bae25/smart-contract/Transfer.sol#L42-L46

#### Recommendation

Remove unnecessary checks to reduce transaction costs and improve compatibility.

## 3.4. Follow good coding practice

### Severity: minor observation

### Description

1. Unlocked Pragma.

Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the floating pragma, i.e. by not using ^ in pragma solidity ^0.8.4, ensures that contracts do not accidentally get deployed using a compiler version with unfixed bugs.

2. Missing docstrings.

The contracts in the code base lack documentation. This hinders reviewers’ understanding of the code’s intention, which is fundamental to correctly assess not only security but also correctness. Additionally, docstrings improve readability and ease maintenance. They should explicitly explain the purpose or intention of the functions, the scenarios under which they can fail, the roles allowed to call them, the values returned, and the events emitted.

Consider thoroughly documenting all functions (and their parameters) that are part of the contracts’ public API. Functions implementing sensitive functionality, even if not public, should be documented as well. When writing docstrings, consider following the Ethereum Natural Specification Format (NatSpec).

3. Missing test suite.

The contract is missing a test suite to validate and verify the behavior of the contract functionalities. Add tests are recommended to ensure that the contract functions and behaves as expected.

4. Functions not used internally could be marked as external.

It's a good coding practice to mark a function external when it's not called within the contract but only from outside the contract.

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

The audited contracts are centralized and users should pay attention to the owner's priviliges. 

It is recommended to adhere to the security practices described in pt. 4 of this report to ensure the contract's operability and prevent any issues that are not directly related to the code of this smart contract.
