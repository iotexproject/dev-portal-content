*This tutorial demonstrates how to remotely control a home automation device using a smart contract. We will be controlling a smart-lock through the popular **Arduino Nano 33 IoT**, which will be connected to the **IoTex blockchain**.*

## Getting Started

Let's go ahead and clone the boilerplate code for this project: 

```bash
 git clone -b smart-lock-with-arduino --single-branch https://github.com/iotexproject/scaffold-iotex.git
```

Then change directory to `scaffold-iotex`:

```bash
cd scaffold-iotex
```

At this point you should see two directories in your project: **`SmartLockBlockchain`** and **`SmartLockDevice`**.

Let's change directory to `SmartLockBlockchain` and install the required dependencies: 

```bash
cd SmartLockBlockchain
```
```bash
npm install
```

If you haven't done so already, install the [Arduino IDE](https://www.arduino.cc/en/software) and the [IoTeX-blockchain-client](https://www.arduino.cc/reference/en/libraries/iotex-blockchain-client/) library, and choose the latest stable version. 

For this specific tutorial we'll use the **Arduino Nano33 IoT**, and we'll therefore need to also install the [FlashStorage](https://www.arduino.cc/reference/en/libraries/flashstorage/) and [WiFiNina](https://www.arduino.cc/reference/en/libraries/wifinina/) Arduino libraries.

## The Smart Contract

The `contracts` folder in the `SmartLockBlockchain` directory contains a file called **`Lock.sol`**. Let's look at it in more detail: 

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.9;

// Import the "Ownable" interface from OpenZeppelin, that implements
// the "onlyOwner" modifier we use to make sure only the contract
// owner can operate the smart-lock
import "@openzeppelin/contracts/access/Ownable.sol";

/**
 * @title Simple Smart-Lock
 * @dev Very simple smart contract to manage the state of a smart-lock
 */
contract Lock is Ownable {
    /** Store the smart lock state */
    bool private _open;
    
    /** Initialise the lock in a closed state */
    constructor (bool open) { _open = open; }

    /** Sets the state of the lock */  
    function setState(bool open) public onlyOwner() { _open = open; }

    /** Returns true if the lock is open */  
    function isOpen() public view onlyOwner() returns (bool) { return _open == true; }
}
```

This contract is quite simple, it only contains two functions:  One to change the state of our smart-lock, and another to retrieve its state. Note the use of the `onlyOwner()` modifiers, which will prevent any user, but the deployer of this contract, to either read or set the state of the lock. 

It is also worth noting that anyone could potentially read the status of the smart-lock, which is something to definitely keep in mind when working on this type of application in production terms. While this discussion is outside the scope of this tutorial, the interested developer is invited to browse through more topical articles that can be found throughout the portal. 

## Deploying the Contract

From the `SmartLockBlockchain` directory run this command to create your own `.env`  file from the `.env-template` file provided: 

```bash
cp .env-template .env
```

At this point your `.env`  file will look like this: 

```bash
IOTEX_PRIVATE_KEY=<your-private-key>
```

Go ahead and replace `<your-private-key>` with the private key corresponding to the account you'd like to use to deploy the contract. 

At this point you'd have to make sure this account has enough **iotx** tokens to deploy this contract. We'll be deploying to the **iotex testnet**. If you'd like to learn how to connect your metamask wallet to the **iotex network**, feel free to check out our [docs](https://docs.iotex.io/get-started/iotex-wallets/metamask). You can then claim some iotx test tokens from your profile page by simply creating an account on our [developers portal](https://developers.iotex.io/).

The deployment script can be found in the `SmartLockBlockchain` directory, in the `scripts` folder. It will look something like this: 

```javascript
// We require the Hardhat Runtime Environment explicitly here. This is optional
// but useful for running the script in a standalone fashion through `node <script>`.
//
// You can also run a script with `npx hardhat run <script>`. If you do that, Hardhat
// will compile your contracts, add the Hardhat Runtime Environment's members to the
// global scope, and execute the script.
const hre = require("hardhat");

async function main() {
  const open = false;

  const Lock = await hre.ethers.getContractFactory("Lock");
  const lock = await Lock.deploy(open);

  await lock.deployed();

  console.log("Lock deployed to: ", lock.address);
  console.log("Lock state: ", open);
}

// We recommend this pattern to be able to use async/await everywhere
// and properly handle errors.
main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

Use the following command to run this script: 

```bash
npx hardhat run scripts/deploy.js --network <NETWORK>
```

You'd have to replace `<NETWORK>` with either `testnet` or `mainnet`. As stated above, we'll go ahead and use `testnet`. 

If the deployment is successful, after running the command above, you should see the following in your console: 

```bash
Compiled 3 Solidity files successfully
Lock deployed to:  0xbB21eD8dB133F0e4216F712D19C68F6b5af6b6e1
Lock state:  false
```

Make sure to take note of this contract address, as we'll be needing it later. 

Now that the contract has been deployed, it's time to work on our smart-lock. 

## The Arduino SmartLock

It's now time to create a DIY smart lock by wiring a magnetic lock to the Arduino board. As said in our introduction, we are going to be moving forward using the **Arduino Nano 33 IoT** but the sketch could also be built for the **ESP32** board.  We will only be using pins GDN, +5V and D21 to drive the Relay module that in turn will enable/disable the smart-lock. 

![arduino](https://user-images.githubusercontent.com/77351244/184433314-cd15b3fd-9162-41df-a415-14b6d7a68aed.png)

![relay](https://user-images.githubusercontent.com/77351244/185755955-aedf1a76-efbd-4e92-8a28-aa5c1289bec4.png)

An example of the wiring is illustarted right below: 

![wiring](https://user-images.githubusercontent.com/77351244/185755629-f2461838-925e-40a2-924f-ac012bace8c8.png)

If you don't have a relay and an actual lock at hand, you can also use the Built-in LED Pin and observe its status instead. 

The sketch for the Arduino board is located in the `SmartLockDevice` directory. 

As you can see in the `SmartLockDevice.ino` file, the sketch "polls" the contract `isOpen()` function to read the lock state from the blockchain. Then it sets the relay pin to either open or close the lock based on the contract state. 

```cpp
#include <Arduino.h>

#ifdef ESP32
    #include <WiFi.h>
#endif
#ifdef ESP8266
    #include <ESP8266WiFi.h>
    #include <ESP8266HTTPClient.h>
    #include <WiFiClient.h>
#endif
#ifdef __SAMD21G18A__
    #include <WiFiNINA.h>
#endif

#include <map>
#include "IoTeX-blockchain-client.h"
#include "secrets.h"
#include "abi.h"

constexpr const char ip[] = IOTEX_GATEWAY_IP;
constexpr const int port = IOTEX_GATEWAY_PORT;
constexpr const char wifiSsid[] = SECRET_WIFI_SSID;
constexpr const char wifiPass[] = SECRET_WIFI_PASS;

// Create the IoTeX client connection
Connection<Api> connection(ip, port, "");

// Enum that represents the status of the lock
enum LockStatus { LOCK_OPEN, LOCK_CLOSED };

// The address
const char contractAddress[] = SECRET_CONTRACT_ADDRESS_IO;

// The address which performs the action
const char fromAddress[] = IOTEX_ADDRESS_IO;

// The contract object
Contract contract(abiJson);

// The call data
String callData = "";
ParameterValuesDictionary params;

// The execution action
Execution execution;

void initWiFi() 
{
    #if defined(ESP32)
        WiFi.mode(WIFI_STA);
        #define LED_BUILTIN 2
    #endif
    WiFi.begin(wifiSsid, wifiPass);
    Serial.print(F("Connecting to WiFi .."));
    while (WiFi.status() != WL_CONNECTED)
    {
        Serial.print('.');
        delay(1000);
    }
    Serial.println(F("Connected. IP: "));
    Serial.println(WiFi.localIP());
}

void setup()
{
    IotexHelpers.setGlobalLogLevel(IotexLogLevel::DEBUG);
    IotexHelpers.setModuleLogLevel("HTTP", IotexLogLevel::DEBUG);
    Serial.begin(115200);

    #if defined(__SAMD21G18A__)
    delay(5000);    // Delay for 5000 seconds to allow a serial connection to be established
    #endif

    // Connect to the wifi network
    initWiFi();

    // Set the lock pin to out
    pinMode(LOCK_PIN, OUTPUT);
    digitalWrite(LOCK_PIN, LOW);

    contract.generateCallData("isOpen", params, callData);
    
    // Print the info
    Serial.print("Calling isOpen on contract ");
    Serial.print(contractAddress);
    Serial.print("with data: 0x");
    Serial.println(callData);

    // Create the execution action
    execution.data = callData;
    strcpy(execution.contract, contractAddress);
}

void loop()
{
    // Read the contract
    ReadContractResponse response;
    ResultCode result = connection.api.wallets.readContract(execution, fromAddress, 200000, &response);
    Serial.print("Result : ");
    Serial.println(IotexHelpers.GetResultString(result));
    Serial.print("Return data: ");
    Serial.println(response.data);

    // Decode the data
    Serial.println("Decoding the data...");
    bool isOpen = decodeBool(response.data.c_str());
    if (result != ResultCode::SUCCESS)
    {
        Serial.println("Failed to decode data");
    }
    else
    {
        String status = isOpen == true ? "OPEN" : "CLOSED";
        Serial.println("Status read from blockchain is: " + status);
        Serial.println("--------------------------------------------------\r\n");
    }

    // Open or close the lock based on the value we have read from the contract
    SetLockStatus(isOpen ? LockStatus::LOCK_OPEN : LockStatus::LOCK_CLOSED);

    // Poll the status every second
    delay(1000);
}

void SetLockStatus(LockStatus status)
{
    int pinStatus = LOW;
    if (status == LockStatus::LOCK_OPEN)
    {
        pinStatus = HIGH;
    }
    digitalWrite(LOCK_PIN, pinStatus);
}
```

At this point, you need to configure the sketch to suit your environment. 

Open the `SmartLockDevice` folder in Arduino IDE, then open the `secrets.h` file, which will look like this: 

```cpp
#ifndef SECRETS_H
#define SECRETS_H

// THe WiFi connection details
#define SECRET_WIFI_SSID   <YOUR_WIFI_SSID>
#define SECRET_WIFI_PASS   <YOUR_WIFI_PASSWORD>

// The contract address in io representation
// You can use https://iotexlab.io/eth2io to convert from 0x to io address
#define SECRET_CONTRACT_ADDRESS_IO   <YOUR_CONTRACT_ADDRESS>

// The address which will send the read action
#define IOTEX_ADDRESS_IO    <YOUR_IOTEX_ADDRESS>

// The digital pin number for the lock
#define LOCK_PIN    <YOUR_PIN>

// IoTeX HTTP gateway
#define IOTEX_GATEWAY_IP    <THE_GATEWAY_IP>
#define IOTEX_GATEWAY_PORT    <THE_GATEWAY_PORT>

#endif
```

Replace the corresponding values with yours. 

Select your board and port in the Arduino IDE and click the upload button. Wait until the firmware is built and flashed to your device. 

## Opening/Closing the smart-lock

It's now time to remotely control the smart-lock through the smart contract we created. All you need to do is to call the `setState()` function in the contract.

Aside from the creation of graphical user interface, that we leave to the interested reader, there are two possible approaches here: One would be to use the iotex [ioctl](https://docs.iotex.io/reference/ioctl-cli-reference) command line tool;  The second approach would be directly through **Hardhat**, by modifying the `hardhat.config.js` file located in the `SmartLockBlockchain` directory. 

We'll look at both of these options: 

### Controlling the smart-lock through the ioctl command line tool

The first thing to do is to go to the [ioctl documentation](https://docs.iotex.io/reference/ioctl-cli-reference) and follow the [installation](https://docs.iotex.io/reference/ioctl-cli-reference/installation) instructions to install this tool on your machine. 

Once you've installed *ioctl*, you need to follow these next four steps in order to configure it for our application: 

- Import the contract owner private key using the following command:

```bash
ioctl account import key smartLock
```

- Set the default signer account to the smart lock owner account: 

```bash
ioctl config set defaultacc smartLock
```

- Point the *ioctl* to the correct network. In this case, we'll point it to the **iotex testnet**:

```bash
ioctl config set endpoint api.testnet.iotex.one:443
```

Should you want to point it to the **iotex mainnet**, simply change from `testnet` to `mainnet` in the command right above. 

You can now use the following two commands to open and close the lock: 

Open the lock with: 

```bash
ioctl contract invoke function io1x6y5gjty378vjwqz7ahdkf5thfeum9q23h394w Lock.abi setState --with-arguments '{"open": true }'
```

And close it with: 

```bash
ioctl contract invoke function io1x6y5gjty378vjwqz7ahdkf5thfeum9q23h394w Lock.abi setState --with-arguments '{"open": false }'
```

After running each command, wait a few seconds to ensure that the transaction is confirmed and that the lock has indeed been opened or closed. 

### Controlling the smart-lock through Hardhat:

As anticipated earlier, the first thing to do here is to modify the `hardhat.config.js` file located in the `SmartLockBlockchain` directory. Our goal here is to create a task to change the state of the lock, which can be called directly from the command line through *Hardhat*.

Add the following task to the `hardhat.config.js` file: 

```javascript
task("setSmartLock", "Opens up the SmartLock")
  .addParam("isOpen", "sets lock to open")
  .addParam("contractAddress", "address of the contract")
  .setAction(async ({ isOpen, contractAddress }) => {
    console.log(
      "Setting lockOpen to: ",
      isOpen,
      " , for contract: ",
      contractAddress
    );

    const Lock = await ethers.getContractFactory("Lock");
    const lock = await Lock.attach(contractAddress);

    let ret = await lock.setState(isOpen);
    console.log("setSmartLock:", ret);
  });
```

The above task - `setSmartLock` - takes in one boolean parameter - `isOpen` - and it allows us to set it to either `true` or `false`. The other parameter - `contractAddress` - will be used to let *Hardhat* know in which contract we want this task to be run. 

The whole `hardhat.config.js` should look like this now: 

```javascript
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

const IOTEX_PRIVATE_KEY = process.env.IOTEX_PRIVATE_KEY;

task("setSmartLock", "Opens up the SmartLock")
  .addParam("isOpen", "sets lock to open")
  .addParam("contractAddress", "address of the contract")
  .setAction(async ({ isOpen, contractAddress }) => {
    console.log(
      "Setting lockOpen to: ",
      isOpen,
      " , for contract: ",
      contractAddress
    );

    const Lock = await ethers.getContractFactory("Lock");
    const lock = await Lock.attach(contractAddress);

    let ret = await lock.setState(isOpen);
    console.log("setSmartLock:", ret);
  });

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: "0.8.9",
  networks: {
    testnet: {
      url: "https://babel-api.testnet.iotex.io",
      accounts: [`${IOTEX_PRIVATE_KEY}`],
    },
    mainnet: {
      url: "https://babel-api.mainnet.iotex.io",
      accounts: [`${IOTEX_PRIVATE_KEY}`],
    },
  },
};
```

Now that the file is ready, we can simply use the following scripts to control the lock: 

```bash
npx hardhat setSmartLock --isOpen <TRUE / FALSE> --contractaddress <CONTRACT_ADDRESS> --network <NETWORK>
```

For example, we'd be using this script to open the lock: 

```bash
npx hardhat setSmartLock --isOpen true --contractaddress 0xbB21eD8dB133F0e4216F712D19C68F6b5af6b6e1 --network testnet
```

And this script to close it: 

```bash
npx hardhat setSmartLock --isOpen false --contractaddress 0xbB21eD8dB133F0e4216F712D19C68F6b5af6b6e1 --network testnet
```

Note that you'd have to add the contract address you got when deploying, and you'd also have to change the network if you decided to deploy to `mainnet`. 

## Conclusions

We have demonstarted how it is possible to use blockchain to remotely control a home automation device, such as our smart-lock. Currently, the majority of people who use home automation devices rely on some sort of subscription-based cloud services, or on local installations of IoT software that allow them to remotely connect and control devices. Using blockchain as a cloud represents a totally different approach and it comes with some drawbacks, but also with some big advantages. Using a smart contract would make the IoT logic of your application far more secure, censorship-resistant, immutable and traceable. These advantages though come at the cost of data storage limitations as well as privacy concerns, given by the open nature of blockchain. Layer-2 solutions such as [W3bStream](https://docs.iotex.io/machinefi/w3bstream-network) would need to be introduced to handle device data verification, complex IoT logic, data storage, data privacy and, possibly, data contribution to certain consumers. 

You can always browse through the portal and learn more about this topic through our dedicated blog articles and tutorials. 


