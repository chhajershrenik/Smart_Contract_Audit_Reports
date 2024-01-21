# Live4well V2 Security Audit Report (re-audit fixed contracts)

# 1. Summary

[live4well-smartcontractv2-for-audit](https://github.com/technine-IT/live4well-smartcontractv2-for-audit) smart contract security audit report performed by [Callisto Security Audit Department](https://github.com/CallistoSecurity/Smart-contract-auditing)

# 2. In scope

Commit `4101aa7f1ae0f1880caba7f1bfd5f42266173a5d`

- [ERC721_nftPassV2.sol](https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/4101aa7f1ae0f1880caba7f1bfd5f42266173a5d/smart-contract/ERC721_nftPassV2.sol)
- [MintManagerV2.sol](https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/4101aa7f1ae0f1880caba7f1bfd5f42266173a5d/smart-contract/MintManagerV2.sol)

# 3. Findings

In total, **0 issues** were reported, including:

- 0 high severity issues.

- 0 medium severity issues.

- 0 low severity issues.

In total, **13 notes** were reported, including:

- 3 minor observations.

- 10 owner privileges.





## 3.1. `SectionDetail` and `publicMintSection` quotas can exceed `MAX_SUPPLY`.

### Severity: owner privileges

### Description

The functions `initializeSectionInfo()`, `changeSectionQuota()`, and `changePublicMintQuota()` do not check if the allowed number of tokens to be minted (`quota`) in `SectionDetail` and `publicMintSection` exceeds `MAX_SUPPLY`. The function `publicMintERC20()` would revert if there is still a `quota` availability and the amount of `MAX_SUPPLY` tokens are minted.

### Code Snippet

- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/4101aa7f1ae0f1880caba7f1bfd5f42266173a5d/smart-contract/MintManagerV2.sol#L46
- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/4101aa7f1ae0f1880caba7f1bfd5f42266173a5d/smart-contract/MintManagerV2.sol#L70
- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/4101aa7f1ae0f1880caba7f1bfd5f42266173a5d/smart-contract/MintManagerV2.sol#L89

### Recommendation

The owner should take into account `MAX_SUPPLY` so that the sum of quotas does not exceed it. 

**Note** The variable `quota` is redundant in the `SectionDetail` as each whitelisted user is only allowed to mint one token and the user is removed from the whitelist, the owner can control the number of tokens that can be minted by the `SectionDetail` by managing the `addressWhitelist`.


## 3.2. Owner Privileges

### Severity: owner privileges

### Description

The `ERC721-nftpassV2.sol` and `MintManagerV2.sol` contract inherits features of the OpenZeppelin [AccessControl](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/AccessControl.sol) contract, allowing the administrator to manage administrators. This role can be abandoned, and it will result in blocking access to critical functions of the contract.

- [NFTPass](https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/4101aa7f1ae0f1880caba7f1bfd5f42266173a5d/smart-contract/ERC721-nftpassV2.sol)
1. The `safeMint` function allows any user with `MINT_ROLE` permissions to mint an unlimited number of NFTs.
2. The `addMinter` function allows the administrator to grant NFT minter role to any address.
3. Function `setBaseURI` allows the owner to modify the `baseURI_` of the NFT tokens.
4. Function `setTokenURI()` allows owner to override the `baseURI_` of NFT tokens to custom URI.

- [MintManager](https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/4101aa7f1ae0f1880caba7f1bfd5f42266173a5d/smart-contract/MintManagerV2.sol)
1. The `privateMint` function, allows any address with `PRIVATE_MINT` rights, to min any number of NFTs for free, depending on the quota.
2. Functions `changeERC20Price` and `changePublicMintERC20Price`, allow the administrator to add any ERC20 tokens including poisoned tokens as payment for NFTs
3. Function `setPrivateMintRole`, allows the administrator to give rights to any address to allow private minting.
4. The `transferERC20Token` function allows the administrator to sign out any tokens from the contract.
5. Add/remove users to/from the section's whitelist.

### Recommendation

Since the owner has unlimited rights to do everything, the ownership must be transferred to a multi-sig contract. And also the addresses to whom the rights `PRIVATE_MINT`, `MINT_ROLE` are given must be trusted.

## 3.3. Tokens can be read URI even if Blind Box is not opened 

### Severity: minor observations

### Description

The function `tokenURI()` returns `blindboxURI` if the Blind Box is not opened. But all data that are stored in the smart contract can be read from the blockchain. So, if your security model expects that `baseURI_` or `tokenURIOverride[tokenId]` should stay unknown for users until you `openBlindbox` you shouldn't set it before that time.

### Code snippet

- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/4101aa7f1ae0f1880caba7f1bfd5f42266173a5d/smart-contract/ERC721_nftPassV2.sol#L58-L65


## 3.4. Optimization Tips

### Severity: minor observation

### Description

1. Resetting allowance to 0 before setting a new value makes sense only for protection from front-running attacks if another amount is already approved. Since it's the first approval, no sense to reset it. 

Recommend to remove `erc20Token.safeApprove(address(this), 0);` from constructor.

### Code snippet

- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L43


2. A function `transferForm` uses more gas than function `transfer`, so if needs to transfer from the `address(this)` better to use `transfer(to, value)` instead of `transferFrom(address(this), to, value)`.

To reduce gas cost on contract deployment and use the function `transferERC20Token()` you should remove from the constructor:
```Solidity
            IERC20 erc20Token = IERC20(_erc20AddressList[i]);
            erc20Token.safeApprove(address(this), 0);
            erc20Token.safeApprove(address(this), 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF);
```

And replace in the function `transferERC20Token()` 

```Solidity
        erc20Contract.safeTransferFrom(address(this), _to, _amount);
```
with
```Solidity
        erc20Contract.safeTransfer(_to, _amount);
```

### Code snippet

- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L42-L44
- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L156


3. Use custom errors to optimize gas usage and reduce deployment costs.

Starting from [Solidity v0.8.4](https://github.com/ethereum/solidity/releases/tag/v0.8.4), there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., revert("Insufficient funds.");), but they are rather expensive, especially when it comes to deployment cost, and it is difficult to use dynamic information in them.



## 3.5. Follow good coding practice

### Severity: minor observation

### Description

1. Unlocked Pragma.

Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the floating pragma, i.e. by not using ^ in pragma solidity ^0.8.4, ensures that contracts do not accidentally get deployed using a compiler version with unfixed bugs.

2. Missing docstrings.

The contracts in the code base lack documentation. This hinders reviewers’ understanding of the code’s intention, which is fundamental to correctly assessing not only security but also correctness. Additionally, docstrings improve readability and ease maintenance. They should explicitly explain the purpose or intention of the functions, the scenarios under which they can fail, the roles allowed to call them, the values returned, and the events emitted.

Consider thoroughly documenting all functions (and their parameters) that are part of the contracts’ public API. Functions implementing sensitive functionality, even if not public, should be documented as well. When writing docstrings, consider following the Ethereum Natural Specification Format (NatSpec).

3. Missing test suite.

The contract is missing a test suite to validate and verify the behavior of the contract functionalities. Add tests are recommended to ensure that the contract functions and behaves as expected.

4. Functions do not emit Events.

Function `setBaseURI()`, `setTokenURI()`, and `openBlindbox()` in the `NFTPass` contract and functions `initializeSectionInfo()`, `changeSectionPeriod()`, `changeSectionQuota()`, `changeERC20Price()`, `changePublicMintERC20Price()`, `changePublicMintQuota()`, `addSectionWhitelist()`, and `removeSectionWhitelist()` in the `MintManager` contract does not emit any events. Events are a way to log and notify external entities (such as user interfaces or other smart contracts) about specific occurrences within a smart contract. They serve as a mechanism for emitting and recording data onto the blockchain, making it transparent and easily accessible.



# 4. Security practices

- [x] **Open-source contact**.
- [ ] **The contract should pass a bug bounty after the completion of the security audit.**
- [ ] **Public testing.**
- [ ] **Automated anomaly detection systems.** - NOT IMPLEMENTED. A simple anomaly detection algorithm is recommended to be implemented to detect behavior that is atypical compared to normal for this contract. For instance, the contract must halt deposits in case a large amount is withdrawn in a short period until the owner or the community of the contract approves further operations.
- [ ] **Multisig owner account.**
- [ ] **Standard ERC20-related issues.** - NOT IMPLEMENTED. It is known that every contract can potentially receive an unintended ERC20-token deposit without the ability to reject it even if the contract is not intended to receive or hold tokens. As a result, it is recommended to implement a function that will allow extracting any arbitrary number of tokens from the contract.
- [ ] **Crosschain address collisions.** ETH, ETC, CLO, etc. It is possible that a transaction can be sent to the address of your contract at another chain (as a result of a user mistake or some software fault). It is recommended that you deploy a "mock contract" that would allow you to withdraw any tokens from that address or prevent any funds deposits. Note that you can reject transactions of native tokens deposited, but you can not reject the deposits of ERC20 tokens. You can use this source code as a mock contract: [extractor contract source code](https://github.com/EthereumCommonwealth/GNT-emergency-extractor-contract/blob/master/extractor.sol). The address of a new contract deployed using `CREATE (0xf0)` opcode is assigned following this scheme `keccak256(rlp([sender, nonce]))`. Therefore you need to use the same address that was originally used at the main chain to deploy the mock contract at a transaction with the `nonce` that matches that on the original chain. _Example: If you have deployed your main contract with address 0x010101 at your 2021th transaction then you need to increase your nonce of 0x010101 address to 2020 at the chain where your mock contract will be deployed. Then you can deploy your mock contract with your 2021th transaction, and it will receive the same address as your mainnet contract._


# 5. Conclusion
The audited smart contract can be deployed. No security issues were found during the audit.

Users should pay attention to the contract owner's privileges.

It is recommended to adhere to the security practices described in pt. 4 of this report to ensure the contract's operability and prevent any issues that are not directly related to the code of this smart contract.
