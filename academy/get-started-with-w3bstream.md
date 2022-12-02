# Introduction 

import {
  Alert,
  AlertIcon,
  Link,
} from '@chakra-ui/react'

*In this Quickstart, we're going to leverage **W3bstream** to develop a simple **MachineFi** application where a user is rewarded with a token by clicking a button on an IoT device:*

<Alert status='success' variant='solid'>
  <AlertIcon />
Check out this workshop on YouTube that walks you through the tutorial: https://youtu.be/TSXUHIMtrO8
</Alert>

## MachineFi

[MachineFi](https://cdn.iotex.io/machinefi/IoTeX%202.0.pdf) is the methodology developed by IoTeX as a way of incentivizing the deployment of machines, financializing the utility and data stream coming from machines, as well as enabling composable and transparent ways of building innovative applications.

## W3bstream
[W3bstream](https://docs.w3bstream.com/) is the core component of every MachineFi application: An off-chain computing infrastructure serving as an open, **chain-agnostic** and decentralized protocol sitting in between blockchain and devices to convert real-world data streams from devices into verifiable, dApp-ready proofs. 

## Click & Earn

Just like any **MachineFi** application, this *Click & Earn* app is made out of three layers: 

### The Blockchain Layer
Implements a decentralized trusted logic based on blockchain, for example IoTeX, which commonly includes at least an incentivizing tokenomics 

### The IoT logic Layer
An off-chain node or decentralized network, that authorizes, verifies, processes, saves the IoT data, and sends the proofs of real world activities to smart contracts on the blockchain. In this tutorial we will create a single-node architecture using the W3bstream framework.

### Hhe hardware Layer
This is typically a set of smart devices: combination of hardware and dedicated software (*firmware*) that collect sensors data and send messages to the w3bstream node.

![machinefi-animation](https://user-images.githubusercontent.com/11096047/203844543-b358d29c-b3e1-476f-9125-ed5ac9d09fa2.gif)



## Cloning the CLick2Earn code

Let's create a new directory called *"click2earn"*: 

```bash
mkdir click2earn && cd click2earn
```

Use this command to clone the MachineFi get-started repository from IoTeX: 

```bash
git clone https://github.com/machinefi/get-started.git && cd get-started
```

This repository will have three directories, `blockchain`, `w3bstream` and `firmware` representing the three components of our **MachineFi dApp**. 

# The Blockchain Layer

For this application we're going to create an ERC20 mintable token with simple access levels through an `operators` mapping that allows the contract owner to enable certain *operators* to mint tokens. 

If you haven't done so already, use the [Utils](https://developers.iotex.io/utils/iotex-testnet) menu item in the [Developers Portal](https://developers.iotex.io/) to configure the IoTeX testnet in your Metamask wallet. If you [login to](https://developers.iotex.io/user/profile) the portal with your GitHub account, you'll also be able to claim some test **IOTX** which you'll need to deploy your smart contract and test this quickstart. 

## Deploy the CLIK token

Let's get into the `blockchain` folder and create a `.env` file containing the private key of the wallet account you wish to use to deploy the contract.

```bash 
cd blockchain
echo IOTEX_PRIVATE_KEY=<YOUR_PRIVATE_KEY> > .env
# example
# echo IOTEX_PRIVATE_KEY=cdd47d5c266...ed450624b55c4e3a > .env
```

We'll be using [hardhat](https://hardhat.org/tutorial) now to deploy the contract, run this command to deploy the `ClickToken` to the **IoTeX Testnet**.

```bash
make deploy
```

Or, if you don't have *make* installed: 

```bash
npx hardhat run scripts/deploy.js --network testnet
```

You should get a log like this: 

```bash
Deploying contracts with the account: 0x12345abcde.......
Address balance: 127500000 IOTX
adding owner as operator
deployer address:  0x15c....
Contract deployed to 0x8....
```

## Import the token in Metamask

After deploying this contract, remember to import the token contract address in your Metamask wallet. If you'd like the rewards to be sent to a different account then the contract owner's, simply create a new one in Metamask and import the `ClickToEarn` token there:
<img width="1461" alt="image" src="https://user-images.githubusercontent.com/11096047/200410540-7cc5011d-821d-4720-bf8a-d4bb8ea47c11.png"/>


## Create an operator account for the W3bstream node

Since the w3bstream node will be calling this contract to mint token rewards, it's recommended to create a dedicated operator account for it, so as not to expose the contract owner account in the W3bstream configuration. Run this hardhat task in order to do so: 

```bash
npx hardhat addOperator --operatoraddress  <W3BSTREAM_NODE_ADDRESS> --clicktokencontract <CLICKTOEARN_TOKEN_CONTRACT>  --network testnet
```

With the blockchain logic in place, it's now time to get the w3bstream node up and running. 

# The W3bstream IoT Layer: 

## Run the W3bstream node

Let's start by running the w3bstream node. Make sure Docker is installed and running on your machine, and run: 

```bash
cd ../../
curl https://w3bstream.com/dc > docker-compose.yaml
export PRIVATE_KEY=<YOUR_W3BSTREAM_PRIVATE_KEY>
docker-compose up -d
```
<Alert status='info' variant='solid'>
  <AlertIcon />
  Note that, in the command above, you'll have to specify the private key associated with your W3bstream node. If you use an account different from the   one that you used to deploy the token contract, then the address should be added as an authorized "operator" in the token contract as specified above.    Otherwise your W3bstrem node won't be able to mint reward tokens. 
</Alert>

You can now access the **W3bstream Studio** admin dashboard by pointing a browser to port `3000` for the w3bstream node server:

```bash
https://localhost:3000
```
And use the default password to access the dashboard.  

<Alert status='success' variant='solid'>
    <AlertIcon />
    If you'd like to learn more about w3bstream and **W3bstream Studio**, feel free to check out the official documentation <Link _hover={{ color: "brand.500" }} textDecoration={"underline"} isExternal href="https://docs.w3bstream.com/">here.</Link>
</Alert>

## Create a W3bstream project

For a newly installed node, you will see something like this:

![create-project](https://user-images.githubusercontent.com/77351244/200185661-28d38fc5-78db-4e73-b377-93ff48032ea0.png)

Create a new project by clicking the "Create a new project now" button and call it ClicktoEarn. 

## Build and deploy the W3bstream logic

Once we have a w3bstream project, we are ready to deploy the [Applet](https://docs.w3bstream.com/applets-development/basic-concepts#applets) containing the IoT logic of our dApp. 

Use your favorite editor to edit the Applet's source code in `click2earn.go` and replace the `Mintable token contract address` towards the end of the file with the `ClickToEarn` token address you deployed earlier. 

```bash
nano w3bstream/src/click2earn.go  
```

![click-go-file](https://user-images.githubusercontent.com/77351244/200188686-5143bd29-8224-4e73-bde6-53f226e7840a.png)


We'll be using **tinygo** to compile the wasm module. 

```bash
cd w3bstream/src
make 
```
Or, if you don't have *make* installed, just run: 

```bash
 cd w3bstream/src
 tinygo build -o click2earn.wasm -scheduler=none --no-debug -target=wasi click2earn.go
 ```
 
 The WASM module will be created in the `w3bstream/wasm` folder.
 
 Let's go back to **W3bstream Studio** to add, deploy and run our Applet. 

![applet](https://user-images.githubusercontent.com/77351244/200187005-bed52396-e77b-4431-9359-d84550db8f8d.png)

Once prompted, select the `click2earn.wasm` file we just created, and click on the **`Deploy`** button to deploy this logic module into your w3bstream node. 


![deployment](https://user-images.githubusercontent.com/77351244/200187036-6e0acd3a-3499-48bb-9c27-775fe48b460b.png)

If the module is correct, it will be successfully deployed, and you will be able to start/stop the module from the dashboard:


![correct-deployment](https://user-images.githubusercontent.com/77351244/200187091-0390443e-a131-499d-97f1-b30bee4da4d6.png)

You'll also notice a **`Strategy ID`** associated with your **Applet**. This is because, when deploying a logic module to a project, w3bstream configures a default event strategy that just routes any event to the module's `start` handler. The default strategy will work fine for this project. Indeed, we called our handler inside the Applet "`_start`", which the tinygo compiler will export as "`start`" in the WASM module.

## Create a Publisher account

Before we can send messages to the W3bstream node, we should also make sure a Publisher Account is set. 


![add-publisher](https://user-images.githubusercontent.com/77351244/200187173-863d9005-9918-4566-b6b8-ed50349aa55d.png)

Select the project, give a name to the publisher and assign it a unique key, like in the image below. Then click the **`Submit Button`** to confirm:

![create-publisher](https://user-images.githubusercontent.com/77351244/200187269-dd95a1c8-cb76-4dfa-8c0e-c142b3f8f77f.png)

Now that the w3bstream node is up and running, it's time to configure the firmware component of our *ClickToEarn* application. 

# The Hardware Layer

## Using W3bstream Studio to simulate device messages

For the purpose of this quickstart, we will simulate messages sent by a smart device from inside W3bstream Studio itself.

<Alert status='info' variant='solid'>
    <AlertIcon />
If you are familiar with embedded development, you can check out the [next section](https://developers.iotex.io/posts/Deploy-a-MachineFi-Dapp#using-an-arduino-board) where we describe an [Arduino](https://www.arduino.cc) sketch that can be used to actually push a button and notify messages to your W3bstream node.
</Alert>

Before proceeding, you may want to follow the logs of the W3bstream node. In a new terminal, type the following:

```bash
docker container logs w3bstream -f
```
or, if you want a more readable output and you have `jq` installed:

```bash
# sudo apt install jq
docker container logs w3bstream -f | jq
```

In W3bstream Studio, select the Click2Earn project and click the `Send Event` button. In the Send Event dialog, select the Publisher, then edit the event `payload` field with:

```bash
"payload": "{\"Account\" : \"<REWARDS_RECIPIENT_ACCOUNT>}"
```

Just replace the `REWARDS_RECIPIENT_ACCOUNT` placeholder with the Metamask account address where you imported the CLIK token before (that's where rewards will be sent).

Finally click the "Submit" Button to send the message at least 5 times, then check your CLIK balance in Metamask to verify that you actually received 1 CLIK token every 5 "click" messages.


## Using an Arduino board

If you are familiar with embedded development, we present a simple [Arduino](https://arduino.cc) sketch that will work on a few ESP32 and Arduino boards. We suggest using an ESP32 with integrated user buttons like the  ESP-WROOM-32. Alternatively, you will have to connect a button to one of the pins on the board.

The firmware in this project sends a message to w3bstream every time the user clicks a button connected to the board, where the payload just includes the recipient address that will receive the token rewards. 

The firmware is set to send data over HTTP by default, but the MQTT option is also provided. 

Make sure you have installed the [Arduino IDE](https://www.arduino.cc/en/software), and move the sketch folder included in the get-started repository to your Arduino projects directory. In MacOS it is located in your Documents folder by default:

```bash
mv firmware/click2earn ~/Documents/Arduino
```
Inside the IDE open `click2earn.ino` file within the `Documents/Arduino/click2earn` directory. 

Let's now configure the `secrets.h` file:

```bash
nano firmware/click2earn/secrets.h 
```

Update the correct fields with their corresponding values:

1. Set your WIFI Network name and password

![ssid](https://user-images.githubusercontent.com/77351244/200196356-06ae0c53-8151-486c-9d1b-de1e14cf6588.png)

2. Set your W3bstream node ip address to SECRET_WEBSTREAM_HOST 

![host](https://user-images.githubusercontent.com/77351244/200196388-929b7eeb-cc51-468b-afb4-8d02e9cfec3c.png)

3. Set the publisher token as it appears inside W3bstream Studio:

![token](https://user-images.githubusercontent.com/77351244/200196420-15ca3fd7-cd4a-4931-94f7-1e25a0c5a570.png)


![publisher-info](https://user-images.githubusercontent.com/77351244/200187412-9cab92b1-9310-40b9-897e-c7a6dd84feac.png)

Flash the firmware to your Arduino board: 

![flash-firmware](https://user-images.githubusercontent.com/77351244/200187488-242f681e-ec25-4b09-bbb5-de15a9aa248d.png)

This firmware will work out of the box on an ESP32-WROOM-32 dev board: just click the "boot" button to send a click event to W3bstream. Click it at least 5 times and check your Metamask wallet to verify you got 1 CLIK token as a reward:

![metamask](https://user-images.githubusercontent.com/77351244/200187609-38ea6dc2-1a9c-41d6-9f7c-93a1d3cede8d.png)

