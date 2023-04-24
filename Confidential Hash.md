# [Confidential Hash Challenge](https://quillctf.super.site/challenges/quillctf-challenges/ctf02)

- Objective of C.T.F.:
1. Find the keccak256 hash of aliceHash and bobHash.

## Code Analisys:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.7;

contract Confidential {
    string public firstUser = "ALICE";
    uint public alice_age = 24;
		bytes32 private ALICE_PRIVATE_KEY; //Super Secret Key
    bytes32 public ALICE_DATA = "QWxpY2UK";
    bytes32 private aliceHash = hash(ALICE_PRIVATE_KEY, ALICE_DATA);

    string public secondUser = "BOB";
    uint public bob_age = 21;
    bytes32 private BOB_PRIVATE_KEY; // Super Secret Key
    bytes32 public BOB_DATA = "Qm9iCg";
    bytes32 private bobHash = hash(BOB_PRIVATE_KEY, BOB_DATA);
		
		constructor() {}

    function hash(bytes32 key1, bytes32 key2) public pure returns (bytes32) {
        return keccak256(abi.encodePacked(key1, key2));
    }

    function checkthehash(bytes32 _hash) public view returns(bool){
        require (_hash == hash(aliceHash, bobHash));
        return true;
    }
}

```

## Hack and Solution(Using Ethers.js):

```typescript
describe('QuillCTF 3: Confidential Hash', () => {
  let contract: ConfidentialHash;
  let owner: SignerWithAddress;

  before(async () => {
    [owner] = await ethers.getSigners();
    contract = await ethers.getContractFactory('ConfidentialHash', owner).then(f => f.deploy());
    await contract.deployed();
  });

  it('should find the private variables', async () => {
    const aliceHash: string = await ethers.provider.getStorageAt(contract.address, ethers.utils.hexValue(4));

    const bobHash: string = await ethers.provider.getStorageAt(contract.address, ethers.utils.hexValue(9));

    // construct the hash as done in the contract via ethers.utils.solidityKeccak256
    const hash = ethers.utils.solidityKeccak256(['bytes32', 'bytes32'], [aliceHash, bobHash]);

    expect(await contract.checkthehash(hash)).to.be.true;
  });
});

```
