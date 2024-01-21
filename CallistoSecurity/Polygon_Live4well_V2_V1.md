# Live4well V2 Security Audit Report

# 1. Summary

[live4well-smartcontractv2-for-audit](https://github.com/technine-IT/live4well-smartcontractv2-for-audit) smart contract security audit report performed by [Callisto Security Audit Department](https://github.com/CallistoSecurity/Smart-contract-auditing)

# 2. In scope

Commit `d31fd40e51037948abdd558d9f13a3187036d7ea`

- [ERC721_nftPassV2.sol](https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/ERC721_nftPassV2.sol)
- [MintManagerV2.sol](https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol)

# 3. Findings

In total, **2 issues** were reported, including:

- 1 high severity issue.

- 0 medium severity issues.

- 1 low severity issue.

In total, **15 notes** were reported, including:

- 4 minor observations.

- 11 owner privileges.


## 3.1. Token can be minted for free

### Severity: high

### Description

The function `publicMintERC20()` allows to minting NFTPass without payment if the user specifies in `_erc20Address` a token address for which there was not set price. In this case `targetSection.unitPriceERC20[_erc20Address]` is equal to 0, so the user will mint NFTPass paying 0 tokens.

### Code snippet

- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L149-L150

### Recommendation

Add checking `targetSection.unitPriceERC20[_erc20Address] > 0` before transfer tokens from user.

```Solidity
        require(targetSection.unitPriceERC20[_erc20Address] > 0, "This token is not added");
        erc20Contract.safeTransferFrom(msg.sender, address(this), targetSection.unitPriceERC20[_erc20Address]);
        erc721Contract.safeMint(_toAddress);
```

## 3.2. MAX_SUPPLY can't be reached 

### Severity: low

### Description

In the function `safeMint()` the requirement `require(tokenId + 1 < MAX_SUPPLY, "Exceed Max Supply.");` allow to mint less tokens then `MAX_SUPPLY`. So the maximum supply of tokens can be `MAX_SUPPLY - 1`.

### Code snippet

- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/ERC721_nftPassV2.sol#L48

### Recommendation

Use the following requirement:
```Solidity
require(tokenId < MAX_SUPPLY, "Exceed Max Supply.");
```

## 3.3. The section's period overlapping

### Severity: owner privileges

### Description

The owner can set sections `startTime` and `endTime`, and public mint `startTime`. There is no check for overlapping those periods. If the owner sets overlapped periods it will cause the following issues:

1. In the function `publicMintERC20()` if sections have overlapped periods (`startTime` of the next section is lower than the `endTime` of the previous section) the last section will be used. 

2. If `isPublicMint == true` and `publicMintSection` overlapped with one of a section in the `sectionDetailList` then `targetSection` will be a pointer to a section in the `sectionDetailList` and further code will use it instead of `publicMintSection`.

### Code snippet

- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L119-L124 
- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L125-L127

### Recommendation

The owner should carefully set up periods avoiding overlaps. 

Better to rewrite the section selection part like this:

```Solidity
        bool isWithinPeriod = false;
        SectionDetail storage targetSection = publicMintSection;
        if (isPublicMint && publicMintSection.startTime <= block.timestamp) {
            isWithinPeriod = true;
        } 
        else {
            for(uint i = 0; i < sectionDetailList.length; i++) {
                if (sectionDetailList[i].startTime < block.timestamp && sectionDetailList[i].endTime > block.timestamp) {
                    targetSection = sectionDetailList[i];
                    isWithinPeriod = true;
                    break;
                }
            }
        }
        require(isWithinPeriod, "It is not in the period of minting");
```

## 3.4. `SectionDetail` and `publicMintSection` quotas can exceed `MAX_SUPPLY`.

### Severity: owner privileges

### Description

The functions `initializeSectionInfo()`, `changeSectionQuota()`, and `changePublicMintQuota()` do not check if the allowed number of tokens to be minted (`quota`) in `SectionDetail` and `publicMintSection` exceeds `MAX_SUPPLY`. The function `publicMintERC20()` would revert if there is still a `quota` availability and the amount of `MAX_SUPPLY` tokens are minted.

### Code Snippet

- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L55
- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L77
- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L94

### Recommendation

The owner should take into account `MAX_SUPPLY` so that the sum of quotas does not exceed it. 

**Note** The variable `quota` is redundant in the `SectionDetail` as each whitelisted user is only allowed to mint one token and the user is removed from the whitelist, the owner can control the number of tokens that can be minted by the `SectionDetail` by managing the `addressWhitelist`.


## 3.5. Owner Privileges

### Severity: owner privileges

### Description

The `ERC721-nftpassV2.sol` and `MintManagerV2.sol` contract inherits features of the OpenZeppelin [AccessControl](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/AccessControl.sol) contract, allowing the administrator to manage administrators. This role can be abandoned, and it will result in blocking access to critical functions of the contract.

- [NFTPass](https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/ERC721-nftpassV2.sol)
1. The `safeMint` function allows any user with `MINT_ROLE` permissions to mint an unlimited number of NFTs.
2. The `addMinter` function allows the administrator to grant NFT minter role to any address.
3. Function `setBaseURI` allows the owner to modify the `baseURI_` of the NFT tokens.
4. Function `setTokenURI()` allows owner to override the `baseURI_` of NFT tokens to custom URI.

- [MintManager](https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol)
1. The `privateMint` function, allows any address with `PRIVATE_MINT` rights, to min any number of NFTs for free, depending on the quota.
2. Functions `changeERC20Price` and `changePublicMintERC20Price`, allow the administrator to add any ERC20 tokens including poisoned tokens as payment for NFTs
3. Function `setPrivateMintRole`, allows the administrator to give rights to any address to allow private minting.
4. The `transferERC20Token` function allows the administrator to sign out any tokens from the contract.
5. Add/remove users to/from the section's whitelist.

### Recommendation

Since the owner has unlimited rights to do everything, the ownership must be transferred to a multi-sig contract. And also the addresses to whom the rights `PRIVATE_MINT`, `MINT_ROLE` are given must be trusted.

## 3.6. Tokens can be read URI even if Blind Box is not opened 

### Severity: minor observations

### Description

The function `tokenURI()` returns `blindboxURI` if the Blind Box is not opened. But all data that are stored in the smart contract can be read from the blockchain. So, if your security model expects that `baseURI_` or `tokenURIOverride[tokenId]` should stay unknown for users until you `openBlindbox` you shouldn't set it before that time.

### Code snippet

- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/ERC721_nftPassV2.sol#L57-L64

## 3.7. Missing zero address validation 

### Severity: minor observation

### Description

1. Missing zero address validation in the `addMinter` function

### Code Snippet
- https://github.com/CallistoSecurity/live4well-smartcontractv2-for-audit-main/blob/main/smart-contract/ERC721_nftPassV2.sol#L53

2. Missing zero address validation for the `erc721Address` variable in constructor

### Code Snippet
- https://github.com/CallistoSecurity/live4well-smartcontractv2-for-audit-main/blob/main/smart-contract/MintManagerV2.sol#L36

### Recommendation
Check that the address is not zero.

## 3.8. Optimization Tips

### Severity: minor observation

### Description

1. The `totalNumberOfMint` variable is not used and can be removed

### Code Snippet
- https://github.com/CallistoSecurity/live4well-smartcontractv2-for-audit-main/blob/main/smart-contract/MintManagerV2.sol#L15


2. Variables `erc721Address` and `MAX_SUPLY` that are not updated following deployment should be declared immutable to save gas.

### Code Snippet
- https://github.com/CallistoSecurity/live4well-smartcontractv2-for-audit-main/blob/main/smart-contract/MintManagerV2.sol#L14


3. Resetting allowance to 0 before setting a new value makes sense only for protection from front-running attacks if another amount is already approved. Since it's the first approval, no sense to reset it. 

Recommend to remove `erc20Token.safeApprove(address(this), 0);` from constructor.

### Code snippet

- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L43


4. A function `transferForm` uses more gas than function `transfer`, so if needs to transfer from the `address(this)` better to use `transfer(to, value)` instead of `transferFrom(address(this), to, value)`.

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


5. In the `MintManager` contract there are multiple checks of tokens allowance and balance before calling the transfer token function. It's not necessary, because these checks are performed in token contracts. To reduce transaction gas usage it may be removed.

### Code snippet

- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L131-L136
- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L138-L143
- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L155


6. Use custom errors to optimize gas usage and reduce deployment costs.

Starting from [Solidity v0.8.4](https://github.com/ethereum/solidity/releases/tag/v0.8.4), there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., revert("Insufficient funds.");), but they are rather expensive, especially when it comes to deployment cost, and it is difficult to use dynamic information in them.

7. Redundant check.

The condition `_price.length > 0` would never revert, and can safely be removed.

### Code snippet
- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L34
- https://github.com/technine-IT/live4well-smartcontractv2-for-audit/blob/d31fd40e51037948abdd558d9f13a3187036d7ea/smart-contract/MintManagerV2.sol#L50


## 3.9. Follow good coding practice

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
- [x] **Standard ERC20-related issues.** - IMPLEMENTED. It is known that every contract can potentially receive an unintended ERC20-token deposit without the ability to reject it even if the contract is not intended to receive or hold tokens. As a result, it is recommended to implement a function that will allow extracting any arbitrary number of tokens from the contract.
- [ ] **Crosschain address collisions.** ETH, ETC, CLO, etc. It is possible that a transaction can be sent to the address of your contract at another chain (as a result of a user mistake or some software fault). It is recommended that you deploy a "mock contract" that would allow you to withdraw any tokens from that address or prevent any funds deposits. Note that you can reject transactions of native tokens deposited, but you can not reject the deposits of ERC20 tokens. You can use this source code as a mock contract: [extractor contract source code](https://github.com/EthereumCommonwealth/GNT-emergency-extractor-contract/blob/master/extractor.sol). The address of a new contract deployed using `CREATE (0xf0)` opcode is assigned following this scheme `keccak256(rlp([sender, nonce]))`. Therefore you need to use the same address that was originally used at the main chain to deploy the mock contract at a transaction with the `nonce` that matches that on the original chain. _Example: If you have deployed your main contract with address 0x010101 at your 2021th transaction then you need to increase your nonce of 0x010101 address to 2020 at the chain where your mock contract will be deployed. Then you can deploy your mock contract with your 2021th transaction, and it will receive the same address as your mainnet contract._


# 5. Conclusion

The audited smart contract must not be deployed. Reported issues must be fixed before the usage of this contract.

It is recommended to adhere to the security practices described in pt. 4 of this report to ensure the contract's operability and prevent any issues that are not directly related to the code of this smart contract.
