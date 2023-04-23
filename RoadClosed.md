# [Road Closed Challenge](https://quillctf.super.site/challenges/quillctf-challenges/road-closed)

- Objective of C.T.F.:
1. Become the owner of the smart contract.
2. Change the value of hacked to true.

## Analisys and Solution:

```solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity 0.8.7;

contract RoadClosed {

    bool hacked;
    address owner;
		address pwner;
    mapping(address => bool) whitelistedMinters;


    function isContract(address addr) public view returns (bool) {
        uint size;
        assembly {
            size := extcodesize(addr)
            }
        return size > 0;
    }

    function isOwner() public view returns(bool){
        if (msg.sender==owner) {
            return true;
        }
        else return false;
    }

    constructor() {
        owner = msg.sender;
    }

    function addToWhitelist(address addr) public {
        require(!isContract(addr),"Contracts are not allowed"); // @audit Step #1: Call this Function with your own Wallet Address and get in the WhiteList.
        whitelistedMinters[addr] = true;
    }
    

    function changeOwner(address addr) public {
        require(whitelistedMinters[addr], "You are not whitelisted");
	require(msg.sender == addr, "address must be msg.sender"); // @audit Step #2: Call this Function with your own Wallet Address to become the Owner.
        require(addr != address(0), "Zero address");
        owner = addr;
    }

    function pwn(address addr) external payable{
        require(!isContract(msg.sender), "Contracts are not allowed");
	require(msg.sender == addr, "address must be msg.sender");
        require (msg.sender == owner, "Must be owner"); // @audit Step #3: Call this Function with your own Wallet Address to change the value "hacked = true;".
        hacked = true;
    }

    function pwn() external payable {
        require(msg.sender == pwner);
        hacked = true;
    }

    function isHacked() public view returns(bool) {
        return hacked;
    }
}

```
