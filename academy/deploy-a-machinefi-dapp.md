---
title: Deploy a MachineFi Dapp
description: Create and deploy the minimal architecture for a MachineFi Dapp on IoTeX
path: academy/deploy-a-machinefi-dapp.md
---

# Overview
This guide provides a starting point for the development of a [MachineFi](https://machinefi.com/) Dapp on IoTeX.  

MachineFi is a new
paradigm fueled by Web3 that underpins the new machine economy, whereby machine resources and intelligence can be financialized to deliver value and ownership to the people, not centralized corporations.

A MachineFi Dapp can be broken into 3 main components:

1. Smart devices / Machines that generate data 

2. An off-chain, Layer-2 compute network that verifies, stores and generates "Proofs" of real-world facts on top of the data

3. An on-chain, Layer-1 set of contracts that manages authorizations and the token-economy based on the real world facts.

![img](https://user-images.githubusercontent.com/11096047/174353904-52b5869b-ed67-4ae9-b32d-30b17130d9df.png)


At the end of this quick start you will have a working infrastructure including a device data simulator, an off-chain data layer that receives, verifies and stores data, and a simple Layer-1 device authorization contract. 


## Requirements

- NodeJS: tested using version 14
- Python 3
- Npm
- Docker and docker-compose

# Installation

## Clone the repository and install the data layer dependencies

```shell
git clone https://github.com/iotexproject/machinefi-getstarted-preview.git
cd machinefi-getstarted-preview
npm install
```
## Setup the environment configuration

Create your `.env` config file from the template    
```shell
cp .env.template .env
```
You can modify the configuration or use the provided one  

## Start the required services

The db, graphql and mqtt services can be started in docker containers using docker-compose.  
Make sure you have `docker` and `docker-compose` installed and running in your machine.  
Run the command below to launch the services  
```shell
docker-compose up
```

After a while, all containers should be up. You should see a log similar to
```shell
graphql-engine_1  | {"type":"startup","timestamp":"2022-06-08T13:24:33.916+0000","level":"info","detail":{"kind":"server","info":{"time_taken":1.3997461,"message":"starting API server"}}}
```

Leave these terminal window open and running and proceed with the next steps in a separate terminal.  

## Create the database

Before starting the data layer, you must create the database and schema.  
Make sure the database and schema names matches your `DB_NAME` and `PROJECT` values respectively, in your `.env`.  
You also need to make sure `psql` is installed in your machine. You can install psql as follows:  
- On OS X: `brew install libpq; brew link --force libpq`  
- On Linux: `sudo apt-get install postgresql-client`  

See here for more: https://www.compose.com/articles/postgresql-tips-installing-the-postgresql-client/  

Create the database using the provided script
```shell
./create-db.sh
```

Example:  
```shell
$ ./createDb.sh
Creating database datalayerdb and schema datalayerdb
CREATE DATABASE
You are now connected to database "datalayerdb" as user "postgres".
CREATE SCHEMA
Database created
```

## Deploy the device registry contract on layer 1

The device registry contract should be deployed on Layer 1. It is used by the application to determine which devices are registered (ie. which devices are valid data sources). If data is received from an unregistered device, it will not be processed.  

Follow the below steps to deploy the device registry contract:

Change directory to blockchain  
```shell
cd blockchain
```

Install required dependencies
```shell
npm install
```

Replace the private key in `blockchain/.env` with your own private key.  
```shell
echo IOTEX_PRIVATE_KEY=<YOUR_PRIVATE_KEY> > .env
# Eg: echo IOTEX_PRIVATE_KEY=1111111111111111111111111111111111111111111111111111111111111111 > .env
```

Deploy the contract using the following command. Replace `NETWORK` with either `testnet` or `mainnet`  
```shell
# Make sure you are on node 14: 
# nvm use 14
npx hardhat run scripts/deploy.js --network <NETWORK>
Eg: npx hardhat run scripts/deploy.js --network testnet
``` 

You should get some output like below  
```shell
Compiled 5 Solidity files successfully
Deploying contracts with the account: 0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A
Account balance: 1.7413600000001  IOTX
DevicesRegistry Contract
address: 0x47eb28D8139A188C5686EedE1E9D8EDE3Afdd543
block: 14737849
```

Take note of the contract address and block height. They will be needed later to configure the application.  

## Configure the data layer blockchain interface to use the contract you just deployed

The configuration file for the data layer application is located in `src/projects/app/project.yaml`. You must edit it to suit your deployment as follows:

- Replace the `startHeight` value with the block previous to the block where the contract was deployed.  

- Replace the `DataSourceRegistry` address with the address of the deployed contract. **Note:** omit the "0x" prefix.  

## Build the data layer

Run the provided npm script to build the data layer  
```shell
cd ..
npm run build
```

## Initialize the database

The database table definitions are application specific and they need to be initialized before we can run the application.  
The `initdb` script must be run to initialize the database and create the tables defined in the `src/projects/app/models` directory.  

Run the following to initialize the database:  
```shell
node dist/projects/app/initdb.js
```

You should get output similar to below  
```shell
> node dist/projects/app/initdb.js

[DB] Connecting to app, DB: localhost, DB_PORT: 5432, DB_USERNAME: postgres, DB_PASSWORD: postgrespassword, DB_NAME: datalayerdb
Initialized DB with the following:  14737849
```

**Note:** You should run `initdb` every time you need to reset db data, e.g. when you redeploy the smart contracts or modify the models.  

# Start the layer-2 data service

## Option 1 - Start the data layer service using npm:

```shell
npm run app
```

## Option 2 - Start the data layer service using PM2:

Ensure pm2 is installed in your system. You can install pm2 with   
```shell
yarn global add pm2
```

or  
```shell
or npm install -g pm2
```

Run the following command to run the app using pm2  
```shell
pm2 start all.yml && pm2 log 0
```

Keep this terminal open and running and proceed with the following steps in a new terminal.  

## Verify the data layer is running

The data layer service should start indexing a few blocks of the IoTeX blockchain (see the endpoint set in .env to switch chain) before reaching the current tip, then it will keep scanning the chain on every new block.  
When a new event is detected (ie. a device registration) some logs will be printed and the event will be processed.  
In the the case of a device registration event, the device will be added to the database.  

When a mqtt message is received, it will be also processed as follows:  
- If the topic matches `/device/<address>/data` then this is some data coming from a device.  
- If the device is not registered, the data is discarded.  
- If the device is registered, the signature of the data is checked.  
- If the signature is valid, the data is inserted into the database.  

The data layer output should be similar to below:
```shell
> node dist/projects/app/main.js

[DB] Connecting to app, DB: localhost, DB_PORT: 5432, DB_USERNAME: postgres, DB_PASSWORD: postgrespassword, DB_NAME: datalayerdb
catchUp start at: 14737849, end at: 14737992
indexing from: 14737850, to: 14737860
indexing from: 14737861, to: 14737871
indexing from: 14737872, to: 14737882
indexing from: 14737883, to: 14737893
indexing from: 14737894, to: 14737904
indexing from: 14737905, to: 14737915
indexing from: 14737916, to: 14737926
indexing from: 14737927, to: 14737937
indexing from: 14737938, to: 14737948
indexing from: 14737949, to: 14737959
indexing from: 14737960, to: 14737970
indexing from: 14737971, to: 14737981
indexing from: 14737982, to: 14737992
catchUp end at 14737993
```

# Send simulated data
## Run the device simulator

A device simulator script is provided that sends randomly generated data that are signed by a private key, supposed to the the device's securely generated key.

The simulator is located in the `simulator` directory. Create an environment file at `simulator/.env` from the template to configure the simulator:

```bash
cp simulator/.env.template simulator/.env
```

```bash
# The private key of the device
PRIVATE_KEY=0x1111111111111111111111111111111111111111111111111111111111111111

# MQTT settings
MQTT_BROKER_HOST="localhost"
MQTT_BROKER_PORT=1883
MQTT_USE_AUTHENTICATION=False
MQTT_USER=user
MQTT_PASSWORD=password

# Interval in seconds between data messages
SEND_INTERVAL_SECONDS=60
```

In order to send test data using the simulator run the following:  
```shell
pip3 install -r simulator/requirements.txt
python3 simulator/simulator.py
```

You should get some output similar to below  
```shell
Connected to MQTT Broker!
Sent {"message": {"address": "0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A", "heartRate": 96, "timestamp": 1654696521}, "signature": "AH3QBnFMxYQHI7p8J69uCMLhoFD9xJM4ovyioaZ7O0lIz7J/Zx4r2FLp5cNJuaXlER4CG4c9OMS6DQb2/F/5Ths="} to topic /device/0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A/data
```

Now check the data layer terminal, you should see the data being received and rejected because the device is not registered.  You should see some output similar to below
```shell
indexing from: 14738081, to: 14738081
topic: /device/0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A/data
Received a message on topic:  /device/0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A/data
WARNING: Dropping data message: Device 0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A is
not registered
```

Follow the next steps in order to register your device.  

## Authorize the device simulator in the smart contract

You can use the provided `registerDevice` hardhat script to register your own device. In order to do it follow the commands below with changes that apply to your system:  

```shell
cd blockchain
npx hardhat registerDevice --deviceaddress <DEVICE_ADDRESS> --contractaddress <CONTRACT_ADDRESS> --network <NETWORK>
# Eg: npx hardhat registerDevice --deviceaddress 0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A --contractaddress 0x4fb87c52Bb6D194f78cd4896E3e574028fedBAB9  --network testnet
```

Replace `DEVICE_ADDRESS` with your 0x prefixed simulated device address.  
Replace `CONTRACT_ADDRESS` with the 0x prefixed address of your registru contract.  
Replace `NETWORK` with either testnet or mainnet.  

You should get some output similar to below  
```shell
Registering device: 0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A ,to contract:  0x47eb28D8139A188C5686EedE1E9D8EDE3Afdd543
registerDevice: {
  hash: '0x3a0188c80fa96a9b498a706bfe4c49970c33ec963c8ea08a431760970dc4eeac',
  type: 0,
  accessList: null,
  blockHash: '0xf47f8052d4faf8fb34ca1a930c56d0276fc10009189e59c66c2233a9f73969e8',
  blockNumber: 14738120,
  transactionIndex: 0,
  confirmations: 1,
  from: '0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A',
  gasPrice: BigNumber { value: "1000000000000" },
  gasLimit: BigNumber { value: "44143" },
  to: '0x47eb28D8139A188C5686EedE1E9D8EDE3Afdd543',
  value: BigNumber { value: "0" },
  nonce: 12,
  data: '0x6dc7b9a500000000000000000000000019e7e376e7c213b7e7e7e46cc70a5dd086daff2a',
  r: '0xd560fb6917411363d0b44e3cc20beba84e3d64498dc0f307d67317d4f38297a6',
  s: '0x0b35ec4e3fd95c66af05dbd921ef5a27b785a5db5196457570fc5e35b34f302c',
  v: 27,
  creates: null,
  chainId: 0,
  wait: [Function (anonymous)]
}
```

Check the data layer logs. After a few seconds the smart contract event should be detected. You should see a log similar to below  
```shell
indexing from: 14738120, to: 14738120
got event DeviceRegistered
Registered new device:  0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A
```   

Once the device is registered, you can use the simulator to send data again. The data should now be accepted and stored in the database.  
You should see some output similar to below when sending data from a registered device  
```shell
indexing from: 14738148, to: 14738148
topic: /device/0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A/data
Received a message on topic:  /device/0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A/data
Device is registered. Processing data
{
  message: {
    address: '0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A',
    heartRate: 126,
    timestamp: 1654696872
  },
  signature: 'BT+ej7RLxPrJbhycgisiB6i1Gx+f+UdVLkZocXzxvLUb7cvnmGm0aln8aQh2tm2B9qijX/yPpRe7moM1yEROsBs='
}
```

Now the data should be stored in the database.  

## Visualizing the data in Hasura

The graphql image provides a Hasura web interface that can be used to visualize the data.  
In order to do this, navigate to http://localhost:9090/ . Make sure you use the port you set in docker-compose if you have modified it.  
You should see the Hasura interface  

![](https://user-images.githubusercontent.com/82106612/172644818-5004bc18-994a-4100-8256-8317203f17b7.png)

Click on Connect your first database  

Enter the database name and url. If you haven't modified the database name, the default url is `postgres://postgres:postgrespassword@postgres:5432/datalayerdb`. If you've modified it, then the url would be `postgres://postgres:postgrespassword@postgres:5432/<YOUR DATABASE NAME>`. 


![](https://user-images.githubusercontent.com/82106612/172646367-3923bbbc-ef16-4e67-b7ee-0a500e7c2d20.png)

Click Connect database. Once connected click the `app` database on the left panel  

![](https://user-images.githubusercontent.com/82106612/172646787-0118fdd7-7845-461b-83dd-332839e5d9dc.png)

Click Track all beside Untracked tables or views  
You can now visualize the database tables. The device data will be stored in the `device_data` table. See an example below  

![](https://user-images.githubusercontent.com/82106612/172647369-5a020341-78d9-4f42-91c5-b114c611fcc6.png)
