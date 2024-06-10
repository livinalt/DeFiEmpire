# DeFi Empire on Avalanche Subnet

Welcome to DeFi Empire clone, a decentralized finance application inspired by DeFi Kingdoms, built on an Avalanche Subnet. This guide will walk you through setting up your environment, deploying your Avalanche subnet, deploying smart contracts, and interacting with them.


## Introduction

DeFi Empire is a blockchain-based gaming platform where players can collect, build, and battle with their digital assets, earning rewards through various game activities. This guide focuses on setting up an EVM subnet on Avalanche and deploying the smart contracts for DeFi Empire clone.

## Prerequisites

Before starting, you need the following:

- [Avalanche-CLI](https://github.com/ava-labs/avalanche-cli)
- [Node.js](https://nodejs.org/)
- [NPM](https://www.npmjs.com/)
- [Metamask](https://metamask.io/)
- [Remix IDE](https://remix.ethereum.org/)

## Setting Up the Avalanche Subnet

### Step 1: Install Avalanche-CLI

First, install the Avalanche-CLI tool:

```sh
curl -sSfL https://raw.githubusercontent.com/ava-labs/avalanche-cli/main/scripts/install.sh | sh -s
```

### Step 2: Create a Subnet

Create a subnet called `mySubnet`:

```sh
avalanche subnet create mySubnet
```

Follow the prompts to configure your subnet.

### Step 3: Deploy the Subnet Locally

Deploy your subnet locally:

```sh
avalanche subnet deploy mySubnet
```

### Step 4: Start the Avalanche Node with Subnet

Start the Avalanche node and include your subnet in its configuration:

```sh
avalanche network start --subnet mySubnet
```

### Step 5: Add Your Subnet to Metamask

- Open Metamask.
- Click on the network dropdown and select "Custom RPC".
- Enter the RPC URL of your local Avalanche node, which is usually `http://localhost:9650/ext/bc/YOUR_SUBNET_ID/rpc`.
  

## Deploying Smart Contracts

### Step 1: Connect Remix to Your Metamask

- Open [Remix IDE](https://remix.ethereum.org/).
- Click on the "Deploy & Run Transactions" tab.
- Select "Injected Web3" as the environment. Ensure Metamask is connected to your custom Avalanche subnet.

### Step 2: Deploy ERC20 Contract 
Deployed ERC20 Contract Address on the Fuji Network: 0xB5Cc0eA09Ea45a317D1c155461EB8d65b973D2c1
Create a new file in Remix and paste the ERC20 contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract ERC20 {
    uint256 public totalSupply;
    mapping(address => uint) public balanceOf;
    mapping(address => mapping(address => uint)) public allowance;
    string public name = "Defi Empire";
    string public symbol = "EMP";
    uint8 public decimals = 18;

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);

    function transfer(address recipient, uint amount) external returns (bool) {
        require(balanceOf[msg.sender] >= amount, "Insufficient balance");
        balanceOf[msg.sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(msg.sender, recipient, amount);
        return true;
    }

    function approve(address spender, uint amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(address sender, address recipient, uint amount) external returns (bool) {
        require(balanceOf[sender] >= amount, "Insufficient balance");
        require(allowance[sender][msg.sender] >= amount, "Allowance exceeded");
        balanceOf[sender] -= amount;
        balanceOf[recipient] += amount;
        allowance[sender][msg.sender] -= amount;
        emit Transfer(sender, recipient, amount);
        return true;
    }

    function mint(uint amount) external {
        balanceOf[msg.sender] += amount;
        totalSupply += amount;
        emit Transfer(address(0), msg.sender, amount);
    }

    function burn(uint amount) external {
        require(balanceOf[msg.sender] >= amount, "Insufficient balance");
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        emit Transfer(msg.sender, address(0), amount);
    }
}
```

Compile the contract and deploy it using Remix.

### Step 3: Deploy Vault Contract 
Deployed Vault Contract Address on the Fuji Network: 0x1552d249aAebe7d0883452721E49074c791e1d81
Create another file in Remix and paste the vault code:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import {IERC20} from "IERC20.sol";

contract Vault {
    IERC20 public immutable token;

    uint256 public totalSupply;
    mapping(address => uint256) public balanceOf;

    event Deposit(address indexed user, uint256 amount, uint256 shares);

    constructor(address _token) {
        token = IERC20(_token);
    }

    function _mint(address _to, uint256 _shares) private {
        totalSupply += _shares;
        balanceOf[_to] += _shares;
    }

    function _burn(address _from, uint256 _shares) private {
        totalSupply -= _shares;
        balanceOf[_from] -= _shares;
    }

    function deposit(uint256 _amount) external {
        require(_amount > 0, "Deposit amount must be greater than 0");
        uint256 shares;
        uint256 currentBalance = token.balanceOf(address(this));

        if (totalSupply == 0) {
            shares = _amount;
        } else {
            shares = (_amount * totalSupply) / currentBalance;
        }

        uint256 allowance = token.allowance(msg.sender, address(this));
        require(allowance >= _amount, "Check the token allowance");

        uint256 senderBalance = token.balanceOf(msg.sender);
        require(senderBalance >= _amount, "Sender has insufficient balance");

        _mint(msg.sender, shares);
        
        bool success = token.transferFrom(msg.sender, address(this), _amount);
        require(success, "Token transfer failed");

        emit Deposit(msg.sender, _amount, shares);
    }

    function withdraw(uint256 _shares) external {
        require(_shares > 0, "Withdraw amount must be greater than 0");
        uint256 amount = (_shares * token.balanceOf(address(this))) / totalSupply;
        _burn(msg.sender, _shares);
        bool success = token.transfer(msg.sender, amount);
        require(success, "Token transfer failed");
    }
}
```

### Step 4: Create a file for the ERC20 Interface
Create a new file in Remix and paste the IERC20 code:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address recipient, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}

```

Compile the contract and deploy it using Remix. Provide the ERC20 token contract address as the constructor argument.

## Interacting with the Contracts

After deploying the contracts, you can interact with them using Remix. Here are some basic interactions:

### Mint Tokens

Mint tokens by calling the `mint` function on the ERC20 contract.

### Approve and Deposit Tokens

Approve the Vault contract to spend your tokens and then call the `deposit` function on the Vault contract.

### Withdraw Tokens

Withdraw your shares by calling the `withdraw` function on the Vault contract.

## Contribution
You are welcome to clone and make changes to this project. 

## Authors
This project was competed by [Jeremiah Samuel](livinalt@gmail.com)

License
This project is licensed under the MIT License