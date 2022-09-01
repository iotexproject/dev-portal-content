## Introduction
This tutorial will show you how to create and deploy an ERC20 token on an IoTeX public network (testnet or mainnet). 
IoTeX implements a fully-featured EVM allowing you to use all your [favorite dev tools](https://docs.iotex.io/dapp-development/web3-development): deploying an ERC20 token on the IoTeX public blockchain is as direct as doing so on any Ethereum-compatible network. 

For this project we’ll use [Truffle](https://trufflesuite.com/), one of the most widely used development suites for Ethereum. 

We’ll also use OpenZeppelin’s smart contract library to import a [Mintable-Pausable ERC20 token](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/presets/ERC20PresetMinterPauser.sol), that will grant the deployer the ability to mint new tokens, pause the contract, and burn tokens. 

## Prerequisites
This is a quick checklist of the minimum tech prerequisites to follow along: 
1. npm package manager
2. Truffle installed on your machine
3. nano (Or any code editor of your choice)
4. Metamask wallet connected with the IoTeX blockchain (check out the docs)

## Packages Installation
Let’s start by creating a new npm project in a new project directory called IoTeXTruffleProject:

```bash
mkdir IoTeXTruffleProject
```
```bash
cd IoTeXTruffleProject
``` 

In IoTeXTruffleProject initialize a nodejs application

```bash
 npm init -y
```

Run this command in order to create a new Truffle project: 

```bash
truffle init
```

You’ll now see Truffle’s project directory triad in your project: `contracts`, `migrations`, `test`.

It’s now time to install Truffle's HDWallet provider package, run this command: 

```bash
npm install @truffle/hdwallet-provider --save
```

If you'd like to get into a little more detail on how to deploy your smart contracts with Truffle on IoTeX feel free to also check out our IoTeX docs [here](https://docs.iotex.io/dapp-development/web3-development/truffle).

We're also going to install another useful package called `dotenv`, that loads environment variables from a `.env` file into `process.env` 

```bash
npm install dotenv --save
```

The last thing to do is to import OpenZeppelin’s smart contracts Library to create our Token and inherit its preset functionalities:

```bash
npm i @openzeppelin/contracts
```

## Mnemonic Phrase

We'll configure Truffle using the mnemonic phrase from our development Metamask account.  

Let's create an environment configuration file:

```bash
nano .env
```

And paste our account's mnemonic phrase inside it:

```bash
// .env content: replace with your mnemonic phrase
MNEMONIC='imitate laptop vital today false cash orphan lucky beef practice today pattern force risk draw pipe mutual ball sleep wet orbit badge song trophy'
```

This ensures we do not expose our mnemonic phrase into Truffle configuration files that could be published on Github.

Make sure that your account has some test IoTX tokens, which you'll need when deploying on IoTeX testnet. 

You can always claim some from your profile on the IoTeX [developers portal](https://developers.iotex.io/). 

## Truffle-Config

The first thing to do is configure Truffle so that it knows the network we’d like to deploy to. 

In this case, we’ll use the IoTeX testnet. 

Modify the `truffle-config.js` file in your project to look like this: 

```javascript
require('dotenv').config();
const { MNEMONIC } = process.env;
const HDWalletProvider = require('@truffle/hdwallet-provider');
module.exports = {
  compilers: {
    solc: {
      version: "^0.8.2",
    }
  },
  networks: {
    testnet: {
      provider: () =>
        new HDWalletProvider({
          mnemonic: {
            phrase: MNEMONIC,
          },
          providerOrUrl: "https://babel-api.testnet.iotex.io",
          shareNonce: true
        }),
      network_id: 4690,    // IOTEX mainnet chain id 4689, testnet is 4690
      gas: 8500000,
      gasPrice: 1000000000000,
      skipDryRun: true
    }
   }
}
```

Notice that we only specified one network object (testnet). If you'd like to develop on IoTeX mainnet you can also add the mainnet configuration by changing providerOrUrl to "https://babel-api.mainnet.iotex.io" and the network_id to 4689. 

## Creating the contract 

Let’s go ahead and create a new file in the contracts directory called: IoToken.sol. 

```bash
cd contracts
```
```bash
nano IoToken.sol
```

Now  add the following code to the file and save it when done. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.2;

import "@openzeppelin/contracts/token/ERC20/presets/ERC20PresetMinterPauser.sol";

contract IoToken is ERC20PresetMinterPauser {
    constructor() ERC20PresetMinterPauser("IOTOKEN", "IOK") {}
}
```

As far as our ERC20 goes, you’re practically done! All we're doing here is using OpenZeppelin's ERC20PresetMinterPauser, which comes in with all the ERC20 specs, plus the ability for the owner to Pause, Mint or Burn tokens. 

We just need to decide the name and symbol of our token and put them in the constructor. 

## Creating the migration script

All that’s left for us to do now is to create the migration script to deploy our IoToken. 

Create a new file in the migrations folder, call it however you want, just make sure that it starts with the number "2" since the migrations are run by Truffle in chronological order. 

In this case, we’re calling this file 2_iotoken_deploy.js: 

```bash
cd ..
cd migrations
nano 2_iotoken_deploy.js
```

And we use the following code as the migration script:

```javascript
const IoToken = artifacts.require("IoToken");

module.exports = async function (deployer, network, accounts) {
  await deployer.deploy(IoToken);
  const iotoken = await IoToken.deployed();
  console.log("deployed at", iotoken.address);
}
```

## Deployment
It's now time to deploy our ERC20 token on IoTeX. Before running the migration script make sure that you have some test IOTX in your metamask account. 

When you're ready to go, run the following command: 

```bash
truffle migrate --reset --network testnet
```

Congratulations! You just created and deployed your ERC20 token on IoTeX! 
