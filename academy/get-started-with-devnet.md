W3bstream allows you to create DePIN applications that leverage real world data streams to generate token rewards for users, based on specific application logic. Such an application usually involves a hardware component, an off-chain logic applet, and the on-chain logic/tokeneconomy smart contracts. 

The create-ws-app package allows you to easily get started with any DePIN application built on W3bstream. The package repository can be found [here](https://github.com/machinefi/create-ws-app), and the npm package can be found [here](https://www.npmjs.com/package/create-ws-app).

## Create the project

To create a project, run the following command: 

```bash
npx create-ws-app my-ws-app
```

Just answer each prompt depending on what dependencies you'll need in your application. In this case, we'll use add all dependencies. 

Enter the project directory the package just created and open your favorite code editor

```bash
cd my-ws-app
code .
```

You'll see three directories: `applet`, `blockchain` and `simulator`. The latter contains a simple device simulator written in node.js that will help testing our w3bstream project by sending simulated data messages. 

## Blockchain

There are four contracts to get you started with this application: `DeviceBinding`, `DevicesRegistry`, `NFT` and `Token`. We'll stick with these contracts. Next we'll create a `.env` file in the `blockchain` directory and add the private key of our developement address to deploy the contracts to IoTeX Testnet. 

```bash
cd blockchain && echo IOTEX_PRIVATE_KEY=0xABC...123 > .env 
npm run deploy:testnet
```

We will later have to grant the W3bstream operator the rights to mint erc20 tokens from the contract we just deployed in our application. 

Remember to also fund the W3bstream operator address with some test tokens. The W3bsteam operator address can be found in your project's settings in W3bsteam Studio: See the "**W3bstream Studio**" section below on how to access your project settings. In order to get your operator address you'll have to first create a new W3bstream project, which you'll be able to do in the next section. 

For a more specific tutorial on how to manage device binding and device identity in a W3bstream application, check out this [link](https://developers.iotex.io/posts/manage-device-identity-and-binding-with-w3bstream). 

To fund your account with test tokens, go to the [Developer Portal](https://developers.iotex.io/) and use the public faucet under the "Dev Tools" tab. 

## Applet

Enter the `assembly` folder in the `applet` directory and have a look at the `handlers` directory. You'll find the `binding.ts` file containing the `handle_device_binding` and the `handle_device_registered`, the `start.ts` handler, which we won't use for this project, and a `erc20.ts` file. Let's focus on the `erc20.ts` file which contains a `handle_data` function which will send a device owner 1 token every time a valid message is sent to our W3bstream project. 

In short, this function retrieves the `payload` of the data message sent by a device to to our W3bstream project, parses it and extracts the `public_key` of the device. It then validates and then verifies that the public key has been registered. After that it extracts the owner of the device and mints 1 token for that address. 

Note that the only thing you need to do here, for the purpose of this exercise, is to assign the address of the erc20 token we deployed earlier to the `TOKEN_CONTRACT_ADDRESS` variable, as shown below: 

![token_contract_address](https://github.com/iotexproject/dev-portal-content/assets/77351244/cd48e8d3-d6b1-4551-a8a7-917098392a05)


You can now build the applet with from the `assembly` directory with: 

```bash
npm run asbuild
```

You'll now be able to use the `release.wasm` file when creating your W3bstream project. Remember to grant minting rights to the W3bstream operator address. 

![ws-operator](https://github.com/iotexproject/dev-portal-content/assets/77351244/5f8c1b07-45d6-4f06-92e2-fbea4a517e13)


You can use this command from the `blockchain` directory to do so: 

```bash
npx hardhat add-erc20-minter --address <W3BSTREAM_OPERATOR_ADDRESS> --network testnet
```

Don't forget to also fund this address, as mentioned earlier. 

## W3bstream Studio

It's now time to jump onto W3bstream Studio and create the database tables and the event routing strategy needed for this application. For more detailed information on how to create a project in W3bstream Studio, create data tables, event monitors and event routing strategies, visit the official W3bstream [documentation](https://docs.w3bstream.com/get-started/w3bstream-studio).

Let's create a project using the `release.wasm` file we just deployed, add a device and create the `devices_registry` and `device_binding` tables. 

`devices_registry`: column(device_id, String), column(is_active, Bool)


`device_binding`: column(device_id, String), column(owner_address, String)

You can see below how the `devices_registry` table would look like: 

![devices_registry](https://github.com/iotexproject/dev-portal-content/assets/77351244/19152f34-bb12-46d3-bb82-c7c59c6d0efc)


Once the tables are created, we need to create the contract monitors: 

### Registry Contract Monitor

`Event Type`: OnDeviceRegistered

`Chain Id`: 4690

`Contract address`: Your DeviceRegistry contract address

`Block Start`: Block number where your contract was deployed

`Block End`: 0

`Topic0`: 0x543b01d8fc03bd0f400fb055a7c379dc964b3c478f922bb2e198fa9bccb8e714


### Device Binding Contract Monitor

`Event Type`: OnDeviceBinding

`Chain Id`: 4690

`Contract address`: Your DeviceBinding contract address

`Block Start`: Block number where your contract was deployed

`Block End`: 0

`Topic0`: 0x79e9049c280370b9eda34d20f57456b7dcc94e83ac839777f71209901f780f48

Next, we need to create the routing strategies that will allow us to register a device, bind a device to an owner's address, and then call the `handle_data` function every time a new data message is sent to our project: 

### Routing Startegies

Our project comes with a `DEFAULT` routing strategy that we need to remove. Instead we'll add these 3 strategies: 

`OnDeviceRegistered` --> handle_device_registered

`OnDeviceBinding` --> handle_device_binding

`DATA` --> handle_data

The first routing strategy responds to the `DeviceRegistered` event emitted in the `DeviceRegistry` contract when a new device is registered. The second routing strategy responds to the `OwnershipAssigned` event emitted by the `DeviceBinding` smart contract when a new device is bound to an owner. The third routing strategy will route the event triggered by a new data message to the `handle_data` function in our applet. 

It's now time to use the data simulator and send data to our project. 

## Data Simulator

The data simulator package allows you to configure a script that will periodically send data messages to your W3bstream project. The first step is to copy the `.env.template` file into a `.env` file with the following fields: `PUB_TOKEN`, `PROJECT_NAME`, and `EVENT_TYPE` and replace the default values with your own ones. The `PUB_TOKEN` is found in the  `devices` tab of your project, while the `EVENT_TYPE` is the name of the W3bstream event that you chose to trigger in the contract monitor earlier. The `PROJECT_NAME` is found in the settings tab of your project: 

![project_name](https://github.com/iotexproject/dev-portal-content/assets/77351244/eda17c0c-15ae-4cc9-b5b7-7afc01027637)


```bash
sed -e 's/pub_token/<YOUR_TOKEN>/g' -e 's/project_name/<YOUR_PROJECT_NAME>/g' -e 's/event_type/<W3BSTREAM_EVENT_NAME>/g' .env.template > .env
```

The data message we'll send will look something like this: 

```JSON
{    
  "data": {        
        "sensor_reading": 75,       
        "timestamp": 1682091108    
  },    
  "public_key": "0xabcd...321",    
  "signature": "0432bef...c00"
}
```

It's now time to run the script from the `simulator` directory with: 

```bash
npm start
```

The script will now run periodically (every 10 seconds by default) and each time a new message is sent, your wallet will be rewarded with a token. The only problem is that the first time you send a message, it will fail, because we haven't registered our simulated device and bound it to a valid owner. You'll will see a log similar to this when your messages start being sent: 
  
![message-error](https://github.com/iotexproject/dev-portal-content/assets/77351244/bc6eae34-50f0-415a-8679-1f8f1366a09d)

All you have to do is to copy the `public_key` you got in the log, and use it in these commands that you'll have to run from the `blockchain` folder: 
  
```typescript
// register a new device
npx hardhat register-device --deviceid <YOUR_DEVICE_PUBLIC_KEY> --network testnet

// bind a device with an owner
npx hardhat bind-device --deviceid <YOUR_DEVICE_PUBLIC_KEY> --userid <YOUR_ADDRESS> --network testnet
```

Start the script again to start sending messages to your project. To visualize your new token rewards, simply import the token address in your metamask wallet, and make sure you're on the IoTeX Testnet. 

## Conclusions

Congratulations, you've created an end-to-end DePIN application that rewards device owners for providing real world data to your W3bstream project. 






