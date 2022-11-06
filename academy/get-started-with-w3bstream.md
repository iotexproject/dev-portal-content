# Introduction 

*In this Quickstart, we're going to leverage **W3bstream** develop a simple **MachineFi** application where a user is rewarded with a token by clicking a button on an IoT device:*

[MachineFi](https://cdn.iotex.io/machinefi/IoTeX%202.0.pdf) is the methodology developed by IoTeX as a way of incentivizing the deployment of machines, financializing the utility and data stream coming from machines, as well as enabling composable and transparent ways of building innovative applications.

[W3bstream](https://docs.w3bstream.com/) is the core component of every MachineFi application: An off-chain computing infrastructure serving as an open, **chain-agnostic** and decentralized protocol sitting in between blockchain and devices to convert real-world data streams from devices into verifiable, dApp-ready proofs. 

Just like any **MachineFi** application, this *ClickToEarn* app is made out of three components: 

- The blockchain trusted logic, which commonly includes at least an incentivizing tokenomics 
- A w3bstream node, with its own logic, that verifies, processes, saves the IoT data, and sends the proofs of real world activities to smart contracts
- The IoT device's firmware that sends messages to the w3bstream node

![w3bstream-animation](https://user-images.githubusercontent.com/77351244/200186494-85ed9aec-5f96-43a0-9c6e-e230002dd49a.png)


## Cloning the CLick2Earn code

Let's create a new project called *"click2earn"*: 

```bash
mkdir click2earn && cd click2earn
```

Use this command to clone the MachineFi get-started repository from IoTeX: 

```bash
git clone https://github.com/machinefi/get-started.git && cd get-started
```

This repository will have three directories, `blockchain`, `w3bstream` and `firmware` representing the three components of our **MachineFi dApp**. 

# The Blockchain Component: 

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
npx hardhat run scripts/deploy.js
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

After deploying this contract, remember to import the token contract address in your Metamask wallet. If you'd like the rewards to be sent to a different account then the contract owner's, simply create a new one in Metamask and import the `ClickToEarn` token there.

## Create an operator account for the W3bstream node

Since the w3bstream node will be calling this contract to mint token rewards, it's recommended to create a dedicated operator account for it, so as not to expose the contract owner account in the W3bstream configuration. Run this hardhat task in order to do so: 

```bash
npx hardhat addOperator --operatoraddress  <W3BSTREAM_NODE_ADDRESS> --clicktokencontract <CLICKTOEARN_TOKEN_CONTRACT>  --network testnet
```

With the blockchain logic in place, it's now time to get the w3bstream node up and running. 

# The W3bstream  Component: 

## Run the W3bstream node

Let's start by running the w3bstream node. Make sure Docker is installed and running on your machine, and run: 

```bash
cd ../
curl https://raw.githubusercontent.com/machinefi/w3bstream/main/docker-compose.yaml > docker-compose.yaml
export WS_WORKING_DIR=$PWD/w3bstream_image
export PRIVATE_KEY=<YOUR_PRIVATE_KEY>
docker-compose -p w3bstream -f ./docker-compose.yaml up -d
```

Note that, in the command above, you'll have to specify the private key associated with the address you just added as "operator". 

You can now access the **W3bstream Studio** admin dashboard by pointing a browser to port `3000` for the w3bstream node server:

```bash
https://localhost:3000
```
And use the default password to access the dashboard. 

> ✅ If you'd like to learn more about w3bstream and **W3bstream Studio**, feel free to check out the official documentation [here](https://docs.w3bstream.com/). 

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

# The Firmware Component:

## Using W3bstream Studio to simulate device messages

For the purpose of this quickstart, we will simulate messages sent by a smart device from inside W3bstream Studio itself.

> ✅ If you are familiar with embedded development, you can check out the next section where we describe an [Arduino](https://www.arduino.cc) sketch that can be used to actually push a button and notify messages to your W3bstream node.



## Use an Arduino board

For the device part, we will use Arduino to flash firmware that will work on a few ESP32 and Arduino dev boards. We suggest using an ESP32 with integrated user buttons like the  ESP-WROOM-32. Alternatively, you will have to connect a button to one of the pins on the board.

The firmware in this project sends an event to w3bstream every time the user clicks on a button connected to the board. The firmware is set to work with an Arduino ESP32 board, but could also work on an Arduino Nano 33 IoT, and is set to send data over HTTP by default. 

The first thing to do is to [install the Arduino IDE](https://www.arduino.cc/en/software), create a new project and paste the sketch found in  the `click2earn.ino` file within the `firmware/click2earn` directory. 

By default, the firmware uses the PIN 35 on the ESP32 board. Simply connect this PIN to the 3.3V PIN on your board. 

Let's now configure the `secrets.h` file:

Update the correct fields with their corresponding values. Note that the `SECRET_PUBLISHER_ID` and `SECRET_PUBLISHER_TOKEN` can be found in your **W3bstream Studio** project in the **`Publishers`** section. 

![publisher-info](https://user-images.githubusercontent.com/77351244/200187412-9cab92b1-9310-40b9-897e-c7a6dd84feac.png)

Now add the `Secrets.h` file in your **Arduino IDE** project, and flash the firmware:

![f;ash-firmware](https://user-images.githubusercontent.com/77351244/200187488-242f681e-ec25-4b09-bbb5-de15a9aa248d.png)

##Putting everything together 

If you have configured the board, connect it to your computer and click the button. Remember that w3bstream will call the `mint()` function in the smart contract after every 5 clicks. Click the button 5 times, wait a few seconds and you should see **1 CLIK** token in your wallet. 


![metamask](https://user-images.githubusercontent.com/77351244/200187609-38ea6dc2-1a9c-41d6-9f7c-93a1d3cede8d.png)

As mentioned earlier, if you haven't configured the board, you can still send messages to your w3bstream node through the **W3bstream** Studio interface under the "[Running a Project](https://docs.w3bstream.com/get-started/running-a-project)" section. Refer to the official [w3bstream documentation](https://docs.w3bstream.com/) for any further information. 


