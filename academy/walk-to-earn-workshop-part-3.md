This is Part III of a series of 3 tutorials:

[Part I](https://developers.iotex.io/posts/Walk-to-Earn-Workshop-Part-1) | [Part II](https://developers.iotex.io/posts/Walk-to-Earn-Workshop-Part-2) | [Part III](https://developers.iotex.io/posts/Walk-to-Earn-Workshop-Part-3)

# Part III
In the second tutorial of this series we left off at deploying the smart contracts that are required for the data oracle to do its job of serving "Proofs of Walking" to the WalkToEarn Dapp, and for step-count owners to actually claim their rewards. 

In this third and last part we will talk about the IoTeX Computational Data Oracle "W3bstream". Because the first usable release of W3bstream is expected by Q4 2022, for this workshop we will use some *experimental code* that was created to run Proof of Concepts in the past year while designing the oracle architecture. This code behaves like a computatiuonal data oracle - but it's not programmable and it's not modular, like W3bstream will be. Nonetheless, we will use this code to complete the Walk to Earn project, as an opportunity to introduce the concepts behing W3bstream and MachineFi. When the firest release of W3bstream will be out, we will replace this code with the actual oracle node.

## The Data Oracle code
This "Experimental oracle code" is already included in the repository that we cloned in previous Part II, which is the "MachineFi get started". It included a DataRegistry contract that we overwrote with our DataRegistry and we also added more contracts fo our WalkToEarn. The repository also includes a Python script to simulate a device sending some "verifiable" data, but we are using a real device here so we'll ignore it. 
And the repository also contains this experimental data oracle code that we will now configure and build to suit our Walk to Earn app.

This oracle code is made of the following services:
- Postgres Database (for IoT data storage)
- GraphQl API (if we want to expose the Data to our Frontend)
- Mosquitto service (to receive IoT data using the MQTT protocol)
- Oracle (the actual service that will "watch" our smart contracts for interaction events, will watch the MQTT service for incoming data events, and will react to all these events implementing our MachineFi logic).

So in order to make it work we need to:
1. Define the database model
2. Configure the incoming data events handler
3. Configure the blockchain events handler 

The configuration of our oracle "app" is located in the folder:
```
src/projects/app
```

## Create the device auth model
Let's start from the database part: we said before that our oracle will keep a database table synced with the DeviceRegistry contract, so basically we want to add a new device id to our oracle database every time the manufacturer whitelists a new device into the DeviceRegistry contract. So let's create the device.model.ts in the `models` folder of our app to define this table:

[models/device.model.ts](https://github.com/simonerom/walk-to-earn-arduino/tree/main/src/projects/app/models/device.model.ts)

The table is pretty simple: only one column and it just stores the list of all device IDs.

## Create the device data model
We also want to store incoming data from our step counter devices. As we anticipated, the Arduino board is sending data messages that look like this 

```json
{
    message: {
        steps: 123,
        timestamp: 16587458
    },
    signature: "..."
    ...
}
```

For every message that we receive from a device, we want tto store the **total steps**, the respective **timestamp** and the **id** of the device that generated it. We will also store the signature for that data message so that we keep the data verifiable (in case we need to re-verify that the database was not tampered with when we generate the "proofs of walking").

So let's create our `device_data.model.ts` that defines a table with these four columns:

[models/device_data.model.ts](https://github.com/simonerom/walk-to-earn-arduino/tree/main/src/projects/app/models/device_data.model.ts)

The rest of the files in that folder is used for storing some Oracle settings in the database, but we are not required to manage them. 

## Provide smart contracts ABIs
Since the oracle will have to interact with our smart contracts, we should provide it the ABIs of these contracts. Given that we have already built and deployed these contracts in Part II, the ABIs can be fount in the folder below:
```bash
blockchain/artifacts/contracts/CONTRACT_NAME/CONTRACT_NAME.json
```
Since the oracle will only interact with the `DeviceRegistry` contract (to sync the list of whitelisted devices with the database) and with the `WalkToEarn` contract (to detect proof requests and call back with the replay), then we only need the ABI for these two contracts. 

Just open `blockchain/artifacts/contracts/DeviceBinding.sol/DeviceBinding.json`, copy **only the ABI array object**:

```json
"abi": [
    {
      "inputs": [],
      "stateMutability": "nonpayable",
      "type": "constructor"
    },

    ...

    "type": "bool"
        }
      ],
      "stateMutability": "nonpayable",
      "type": "function"
    }
  ]
  ```
  and paste it in the ABI folder of the Oracle app:

  ```
  src/projects/app/abis/DeviceRegistry.json
  ```

  do the same for the `WalkToEarn.json` ABI.

## Configure the oracle
Now it's time to configure the oracle behavior: what contracts should the oracle watch? How should it receive the data from devices? And what should the oracle do when new data is received or when a contract event is detected?

### Base settings
The oracle application settings are located in the file `project.yaml`in the app folder. Here is the full content:

`src/projects/app/`[project.yaml](https://github.com/simonerom/walk-to-earn-arduino/blob/main/src/projects/app/project.yaml)

the first section of the project (*features*) tells the oracle which modules to run: we want the MQTT service to receive IoT data and we want the Blockchain service to watch our smart contracts:
```yaml
features:
  - mqtt
  - blockchain
```
Then you may want to change the private key of the oracle in the `host` section: that will be used to call back the WalkToEarn contract to reply with Proofs of Walking, and it should be the same key used to deploy the contracts (the contracts "owner" account).
```yaml
host:
  privatekey: '0xbd8ddc8086f99c2221ece7c0c2e13d25456a38ac487ef9e73e9d3b0697732e$
```
Finally you want to set the blockchain haight (or _block number_) when you have deployed your contracts: this way the oracle will start indexing the blockchain from that block on, no need to index prior than that. That information is logged on the screen when you deploy the contracts in the hardhat folder.
```
startHeight: 15400834
```
### Blockchain events settings
In the dataSource section, we can configure the different modules (MQTT and Blockchain). We add 2 `ethereum/contract`subsections one for each of the contracts we want to watch, and we tell the oracle what's the contract address and what's the name of the event we want to watch, as well as what function it should call when such event is emitted by the contract:

```yaml
dataSources:
  - kind: ethereum/contract
    name: DevicesRegistry
    source:
      address: 094b57BEFd1151B4b7dd43f9DCABCbc8F144A44a
      abi: DevicesRegistry
    eventHandlers:
      - event: DeviceRegistered
        handler: onDeviceRegistered
  - kind: ethereum/contract
    name: WalkToEarn 
    source:  
      address: 2BdF71d56B20F89959F4869D27f5A7b812461B31
      abi: WalkToEarn
    eventHandlers:
      - event: ActivityRequested
        handler: onActivityRequested
```


### Data events settings
Now we got to tell the oracle the MQTT "topic" to subscribe to: if you are not familiar with MQTT, it's enough to know that each MQTT client (the Arduino board) will send the data to a certain topic, sort of a path, that is composed as this:

```
device/DEVICE_ID
```

So in the dataSource section, under the "Mqtt" data source, we are telling the oracle to only conside data that is sent to one of those topics, so we use a regex (we assume the device ID can be anything). We are also telling the oracle what to do when new data is received on the MQTT protocol (call the `onMqttData` function, we will add it soon):

```yaml
handlers:
      - topicReg: ^/device\/[a-fA-F0-9]+\/data$
        handler: onMqttData
```

## Define the handlers
Now the real stuff: we must implement the logic of our oracle! We have 3 handler functions to implement:

**onMqttData**: this will get the received data message, extract the signature, verify who this signer is, check in the database that signer is a registered device and, if so, store the steps data, the time stamp and the data signature in the data base.

**onDeviceRegistered**: this will just add to the database the new device id just registered on the blockchain, as provided by the event data.

**onActivityRequested**: this does the real job! It looks in the database for all data that belongs to the device in the specific timeframe as per the blockchain event data, and will call back the WalkToEarn smart contract to send the total number of steps in the timeframe.

All handlers are defined in the `handlers.ts` file in `src/projects/app`:

src/projects/app/[handlers.ts](https://github.com/simonerom/walk-to-earn-arduino/blob/main/src/projects/app/handlers.ts)

## Start the services
First let's install required node packages:
```
npm install
```
Create your .env config file from the template
```
cp .env.template .env
```

We now start the accessory services: the database, the GarphQl API and the MQTT server. It's all configured in a Docker-compose

```
# On MacOs
docker-compose up
# On Ubuntu
docker compose up
```

Since it's the first time we run the database, we should create our database, and we have a script for that:
```
./create-db.sh
```

## Initialize the database
We are ready to build the oracle service with
```
# make sure you have typescript installed in your system
# sudo npm install
npm run build
```

then we need to initialize the DB: the command below will just "wipe" all database tables and re-create them based on our model definitions: you want to do this the first time and every time you re-deploy the smart contracts:

```
node dist/projects/app/initdb.js
```

## Run the oracle and send test data
We can run the oracle from inside VS code, or from the command line:

```
npm run start
```

Here we are: the oracle is finally running and ready to accept MQTT data from devices, and detect requests from our smart contracts.

Let's reset the Arduino board and simulate some steps: from the oracle log we should see this data incoming (make sure the oracle server has the port 1883 open for incoming connections!) and also we should see it being discarded by the oracle!

In fact, when the oracle receives the data it recovers the public key that made the signature and the first 20 bytes are supposed to be the device ID: and because we have not whitelisted any device yet, the oracle cannot find that in the devices table and rejects the data. 

## Authorize and bind the step counter
So time to interact with our smart contracts: get back to the blcockain project folder and let's use our Tasks/scripts we prepared to easily interact with our contracts.

As the device manufacturer, I want o do two things:

1. Whitelist the device in the DevicesRegistry:
```
npx hardhat registerDevice --deviceid 04b687e298ad52eec4fe32b27af45247f3659062 --registrycontract 3828fC74E1c4C57E353AB99FC5B3fF2A89ef6720  --network testnet
```
2. Make sure that device is bound to the user who bought it! We can use a wallet account for the user other then the one we used to deploy the contracts, just to make things more real. We can also generate a new account and send it some test tokes from our other occount.

```
npx hardhat bindDevice --deviceid 04b687e298ad52eec4fe32b27af45247f3659062 --owneraddress 0x169dc1Cfc4Fd15ed5276B12D6c10CE65fBef0D11 --contractaddress 0x242688423D4DDA708642e4b32Ab72a5b23b7D86f  --network testnet
```

Check the oracle logs and you should see that it detected the device registration, and now the data from our step counter should be accepted and stored by the oracle.

## Claim rewards
We reached the end of this long workshop! Time to do some walking activity and get our deserved rewards!

In the blockchain project we can run the script to claim the walking activity:

```
npx hardhat claimActivityRequest --deviceid 04b687e298ad52eec4fe32b27af45247f3659062 --contractaddress 0x401677815F75026C2328E723202983D2232671c7 --network testnet
```

then run the contract call to claim our rewards

```
npx hardhat claimRewards --owneraddress 0x169dc1Cfc4Fd15ed5276B12D6c10CE65fBef0D11 --contractaddress 0x401677815F75026C2328E723202983D2232671c7 --network testnet 
```

Check our blcockain wallet and we should have earned some STP tokens!
