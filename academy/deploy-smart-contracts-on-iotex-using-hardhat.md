---
title: Deploy smart contracts on IoTeX
description: Deploy a smart contracts on IoTeX in a few steps using Hardhat
path: academy/deploy-smart-contracts-on-iotex-using-hardhat.md
---

# Introduction

The IoTeX Blockchain implements a full-featured Ethereum Virtual Machine (EVM), allowing you to use Solidity as a programming language to create smart contracts on IoTeX or port any existing Ethereum smart contract to IoTeX without changes to the source code.

In addition to that, any IoTeX gateway node provides a full Ethereum API, so that any Ethereum client can also interact with the IoTeX blockchain without any change to the code.

# Core information

This tutorial teach you how to:

1. Create an IoTeX smart contract using [Solidity](https://docs.soliditylang.org/en/v0.8.14/)

2. Deploy a smart contract using the official IoTeX endpoint for Ethereum clients: https://babel-api.testnet.iotex.io

# Tools that you need

We will use the popular [Hardhat](https://hardhat.org) developer environment to deploy a very simple "Hello World" Solidity contract on IoTeX. You can follow the ["Setting up the environment](https://hardhat.org/tutorial/setting-up-the-environment.html) HardHat tutorial to set up NodeJS on your system before getting started.

We will also use [VS Code](https://code.visualstudio.com) as the editor, but you can use any other editor that you like.

# Create a testnet wallet in Metamask

Make sure you create a new wallet account in Metamask that you will use exclusively for testnet development. You also need some testnet IOTX tokens in your account, that you can claim directly from your profile on [developers.iotex.io](https://developers.iotex.io)

# Creating a new Hardhat project

Let's start with creating an empty Hardhat project:

```bash
mkdir hardhat-tutorial
cd hardhat-tutorial
npm init --yes
npm install --save-dev hardhat

npx hardhat
```

# Install plugins

You will typically install some Hardhat plugins make Solidity development and testing much easier:

```bash
npm install --save-dev @nomiclabs/hardhat-ethers ethers @nomiclabs/hardhat-waffle ethereum-waffle chai
```

# Create a simple contract in Solidity

Let's create a simple Hello World smart contract in the "contracts" folder of our project, then open it inside the editor:

```bash
mkdir contracts
code contracts/hellowowrld.sol
```

The source code of the contract is extremely simple: we only have a public string variable initialized with "Hello Blockchain World!" value.

```solidity
// SPDX-License-Identifier: MIT
// carbon.sol

pragma solidity ^0.8.9;
contract HelloWorld {

    string public message;

    constructor() {
        message = "Hello Blockchain World!";
    }
}
```

# Edit hardhat.config.js to deploy to IoTeX Testnet

Now comes the most relevant part: adding the IoTeX Testnet endpoint to the Hardhat networks.

```js
require("@nomiclabs/hardhat-waffle");

const IOTEX_PRIVATE_KEY = "<YOUR PRIVATE KEY HERE>";

module.exports = {
  solidity: "0.8.9",
  networks: {
    testnet: {
      // These are the official IoTeX endpoints to be used by Ethereum clients
      // Testnet https://babel-api.testnet.iotex.io
      // Mainnet https://babel-api.mainnet.iotex.io
      url: `https://babel-api.testnet.iotex.io`,

      // Input your Metamask testnet account private key here
      accounts: [`${IOTEX_PRIVATE_KEY}`],
    },
  },
};
```

# Edit the deploy script

Let's now create a default Hardhat deploy script for the contract in the scripts folder of our project:

```
mkdir scripts
code scripts/deploy.js
```

The script will just load the account corresponding to the private key we defined in the Hardhat configuration, load the Hello World smart contract, and use the account to deploy it:

```js
async function main() {
  const [deployer] = await ethers.getSigners();

  console.log("Deploying contracts with the account:", deployer.address);

  console.log("Account balance:", (await deployer.getBalance()).toString());

  const HelloWorld = await ethers.getContractFactory("HelloWorld");
  const helloWorld = await HelloWorld.deploy();

  console.log("Contract address:", helloWorld.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

# Deploy

Now that everything is in place, we can finally deploy the contract to the IoTeX testnet with:

```
npx hardhat run scripts/deploy.js --network testnet
```
