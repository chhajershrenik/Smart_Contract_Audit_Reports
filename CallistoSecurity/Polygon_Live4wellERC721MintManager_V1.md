# Live4well ERC721 token and Mint Manager Security Audit Report

# 1. Summary

[Live4well](https://github.com/technine-IT/live4well-smartcontract-for-audit) smart contract security audit report performed by [Callisto Security Audit Department](https://github.com/CallistoSecurity/Smart-contract-auditing)

This is re-audit of smart contracts that were fixed by developer according our recommendation.

# 2. In scope

Commit `467a0fcbc46b2cc94d9ae9ce63d6d565f318a2eb`

- [MintManager.sol](https://github.com/technine-IT/live4well-smartcontract-for-audit/blob/467a0fcbc46b2cc94d9ae9ce63d6d565f318a2eb/smart-contract/MintManager.sol)
- [ERC721-nftpass.sol](https://github.com/technine-IT/live4well-smartcontract-for-audit/blob/467a0fcbc46b2cc94d9ae9ce63d6d565f318a2eb/smart-contract/ERC721-nftpass.sol)
- [IERC721-nftpass.sol](https://github.com/technine-IT/live4well-smartcontract-for-audit/blob/467a0fcbc46b2cc94d9ae9ce63d6d565f318a2eb/smart-contract/IERC721-nftpass.sol)

# 3. Findings

In total, **0 issues** were reported, including:

- 0 high severity issues.

- 0 medium severity issues.

- 0 low severity issues.

In total, **12 notes** were reported, including:

- 2 minor observations.

- 10 owner privileges.



## 3.1. Owner Privileges

### Severity: owner privileges

### Description

The `ERC721-nftpass.sol` and `MintManager.sol` contract inherits features of the OpenZeppelin [AccessControl](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/AccessControl.sol) contract, allowing the administrator to manage administrators. This role can be abandoned, and it will result in blocking access to critical functions of the contract.

- [NFTPass](https://github.com/technine-IT/live4well-smartcontract-for-audit/blob/467a0fcbc46b2cc94d9ae9ce63d6d565f318a2eb/smart-contract/ERC721-nftpass.sol)
1. The `safeMint` function allows any user with `MINT_ROLE` permissions to mint an unlimited number of NFTs.
2. The `addMinter` function allows the administrator to grant NFT minter role to any address.
3. Function `setBaseURI` allows the owner to modify the `baseURI_` of the NFT tokens.
4. Function `setTokenURI()` allows owner to override the `baseURI_` of NFT tokens to custom URI.

- [MintManager](https://github.com/technine-IT/live4well-smartcontract-for-audit/blob/467a0fcbc46b2cc94d9ae9ce63d6d565f318a2eb/smart-contract/MintManager.sol)
1. The `privateMint` function, allows any address with `PRIVATE_MINT` rights, to min any number of NFTs for free, depending on the quota.
2. The `changeLevelPeriod` function, allows the administrator to change the time range for a mint
3. Function `changeLevelquota`, allows the administrator to change the quota for the NFT mint
4. Function `addNewERC20Price`, allows the administrator to add any ERC20 tokens including poisoned tokens as payment for NFTs
5. Function `setPrivateMintRole`, allows the administrator to give rights to any address to allow private minting.
6. The `transferERC20Token` function allows the administrator to sign out any tokens from the contract.

### Recommendation

Since the owner has unlimited rights to do everything, the ownership must be transferred to a multi-sig contract. And also the addresses to whom the rights `PRIVATE_MINT`, `MINT_ROLE` are given must be trusted.


## 3.2. The ERC-20 tokens price is set manually

### Severity: minor observation

### Description

The functions `initializeNFTLevelInfo()` and `addNewERC20Price()` are used by the owner to set the price of ERC-721 tokens in ERC-20 tokens. For non-stable coins, the value might fluctuate and the prices are required to be updated in case of a change in value. Allowing users to mint the ERC-721 token at an increased or decreased value depending on the price fluctuation.

### Code Snippet

- https://github.com/technine-IT/live4well-smartcontract-for-audit/blob/467a0fcbc46b2cc94d9ae9ce63d6d565f318a2eb/smart-contract/MintManager.sol#L36
- https://github.com/technine-IT/live4well-smartcontract-for-audit/blob/467a0fcbc46b2cc94d9ae9ce63d6d565f318a2eb/smart-contract/MintManager.sol#L66

### Recommendation

Consider using a price oracle to get the current value of the ERC-20 tokens and convert the value to a stablecoin to determine the amount of ERC-20 tokens required to mint an NFT by a user.

## 3.3. Follow good coding practice

### Severity: minor observation

### Description

1. Unlocked Pragma.

Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the floating pragma, i.e. by not using ^ in pragma solidity ^0.8.4, ensures that contracts do not accidentally get deployed using a compiler version with unfixed bugs.

2. Missing docstrings.

The contracts in the code base lack documentation. This hinders reviewers’ understanding of the code’s intention, which is fundamental to correctly assess not only security but also correctness. Additionally, docstrings improve readability and ease maintenance. They should explicitly explain the purpose or intention of the functions, the scenarios under which they can fail, the roles allowed to call them, the values returned, and the events emitted.

Consider thoroughly documenting all functions (and their parameters) that are part of the contracts’ public API. Functions implementing sensitive functionality, even if not public, should be documented as well. When writing docstrings, consider following the Ethereum Natural Specification Format (NatSpec).

3. Missing test suite.

The contract is missing a test suite to validate and verify the behavior of the contract functionalities. Add tests are recommended to ensure that the contract functions and behaves as expected.

# 4. Security practices

- [ ] **Open-source contact**.
- [ ] **The contract should pass a bug bounty after the completion of the security audit.**
- [ ] **Public testing.**
- [ ] **Automated anomaly detection systems.** - NOT IMPLEMENTED. A simple anomaly detection algorithm is recommended to be implemented to detect behavior that is atypical compared to normal for this contract. For instance, the contract must halt deposits in case a large amount is withdrawn in a short period until the owner or the community of the contract approves further operations.
- [ ] **Multisig owner account.**
- [ ] **Standard ERC20-related issues.** - NOT IMPLEMENTED. It is known that every contract can potentially receive an unintended ERC20-token deposit without the ability to reject it even if the contract is not intended to receive or hold tokens. As a result, it is recommended to implement a function that will allow extracting any arbitrary number of tokens from the contract.
- [ ] **Crosschain address collisions.** ETH, ETC, CLO, etc. It is possible that a transaction can be sent to the address of your contract at another chain (as a result of a user mistake or some software fault). It is recommended that you deploy a "mock contract" that would allow you to withdraw any tokens from that address or prevent any funds deposits. Note that you can reject transactions of native tokens deposited, but you can not reject the deposits of ERC20 tokens. You can use this source code as a mock contract: [extractor contract source code](https://github.com/EthereumCommonwealth/GNT-emergency-extractor-contract/blob/master/extractor.sol). The address of a new contract deployed using `CREATE (0xf0)` opcode is assigned following this scheme `keccak256(rlp([sender, nonce]))`. Therefore you need to use the same address that was originally used at the main chain to deploy the mock contract at a transaction with the `nonce` that matches that on the original chain. _Example: If you have deployed your main contract with address 0x010101 at your 2021th transaction then you need to increase your nonce of 0x010101 address to 2020 at the chain where your mock contract will be deployed. Then you can deploy your mock contract with your 2021th transaction, and it will receive the same address as your mainnet contract._

# 5. Conclusion

The audited smart contract can be deployed. No security issues were found during the audit.

Users should pay attention to unlimited owner's rights.

It is recommended to adhere to the security practices described in pt. 4 of this report to ensure the contract's operability and prevent any issues that are not directly related to the code of this smart contract.
