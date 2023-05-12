W3bstream allows you to create DePIN applications that leverage real world data streams to generate token rewards for users, based on specific application logic. Such an application usually involves a hardware component, an off-chain logic applet, and the on-chain logic/tokeneconomy smart contracts. 

The create-ws-app package allows you to easily get started with any DePIN application built on W3bstream. The package repository can be found [here](https://github.com/machinefi/create-ws-app), and the npm package can be found [here](https://www.npmjs.com/package/create-ws-app).

## Create the project

To create a project, run the following command: 

```bash
npx create-ws-app
```

Just answer each prompt depending on what dependencies you'll need in your application. In this case, we'll use add all dependencies. 

Enter the project directory the package just created and open your favorite code editor

```bash
cd my-ws-app
code .
```

You'll see three directories: `applet`, `blockchain` and `simulator` which will act as our device, periodically sending a simulated data message to our W3bsream project. 

## Blockchain

There are four contracts to get you started with your application: `DeviceBinding`, `DevicesRegistry`, `NFT` and `Token`. We'll stick with these contracts. Next we'll create a `.env` file in the `blockchain` directory and add our private key to deploy the contracts to IoTeX Testnet. 

```bash
cd blockchain && nano IOTEX_PRIVATE_KEY=<YOUR_PRIVATE_KEY> > .env 
npm run deploy:testnet
```

We will later use the following command: 

```bash
npx hardhat add-erc20-minter --address <W3BSTREAM_OPERATOR_ADDRESS> --network testnet
```

To grant W3bstream the rights to mint erc20 tokens from the contract we just deployed in our application. 

Remember to fund the W3bstream operator address (which can be found in your project's settings, see "**W3bstream Studio**" section below on how to access your project) with enough IOTX test tokens to mint each time a data message is received by your project. In order to get your operator address you'll have to create a new W3bstream project, and in order to do that, you'll need an applet. Which you'll be able to do in the next section. 

For a more specific tutorial on how to manage device binding and device identity in a W3bstream application, check out this [link](https://developers.iotex.io/posts/manage-device-identity-and-binding-with-w3bstream). 

To fund your account with test tokens, go to the [Developer Portal](https://developers.iotex.io/) and use the public faucet under the "Dev Tools" tab. 

## Applet

Enter the `assembly` folder in the `applet` directory and have a look at the `handlers.ts` file. 

```bash
cd applet/assembly && cat handlers.ts
```

You'll see the `handle_device_registered` and the `handle_device_binding` functions. Our next step is to add one more handler to handle the data stream from the simulated device to our W3bstream project. We simply want to mint 1 token for the owner of a valid device, every time a new data message is sent to our project.

Add this import at the top of the file: 

```typescript
import { mintRewards } from "./rewards/mint-rewards";
```

And then add the following function instead of the `start` handler: 

```typescript
// This handler will be executed each time a new data message 
// is sent to our W3bstream project
export function handle_data(rid: i32): i32 {
  Log("Hello W3bstream!");
  const deviceMessage = GetDataByRID(rid);
  Log("device message: " + deviceMessage);
  const message_json = JSON.parse(deviceMessage) as JSON.Obj;
  const pub_key = message_json.get("public_key") as JSON.Str;
  if (pub_key == null) {
    Log("Public key is null");
    return 1;
  }
  // Check if the public key is registered in the W3bstream DB
  const query_pub_key = `SELECT is_active FROM "devices_registry" WHERE public_key = ?;`;
  const result_pub_key = QuerySQL(query_pub_key, [new String(pub_key.valueOf())]);
  if (result_pub_key == "") {
    Log("Public key not found in DB");
    return 1;
  }
  // Find the owner address for the device
  const query_owner_address = `SELECT owner_address FROM "device_binding" WHERE public_key = ?;`;
  const result_owner = QuerySQL(query_owner_address, [new String(pub_key.valueOf())]);
  if (result_owner == "") {
    Log("Owner address not found in DB");
    return 1;
  }
  // Log the owner address and send 1 token
  const owner_address_json = JSON.parse(result_owner) as JSON.Obj;
  const owner_address = owner_address_json.getString("owner_address") as JSON.Str;
  Log("Owner address: " + owner_address.valueOf());
  Log("Sending tokens to owner address...");
  mintRewards("<YOUR_CONTRACT_ADDRESS>", owner_address.valueOf(), "1");
  return 0;
}
```

In short, this function retrieves a data message, parses it and extracts the `public_key` of the device that sent the data. It then checks if the device is actually registered (lines 13-19) and then retrieves the `owner_address` by querying the DB. Once the `owner_address` is found, it then mints one token to it.

Note that the only thing you need to modify in the function is the Token contract address we deployed earlier. 

You can now build the applet with:

```bash
npm run asbuild
```

You'll now be able to use the `release.wasm` file when creating your W3bstream project, and remember to go to settings, grant minting rights to the W3bstream operator address, and fund that address with some tokens. 

## W3bstream Studio

It's now time to jump onto W3bstream Studio and create the database tables and the event routing strategy needed for this application. For more detailed information on how to create a project in W3bstream Studio, create data tables, event monitors and event routing strategies, visit the official W3bstream [documentation](https://docs.w3bstream.com/get-started/w3bstream-studio).

Let's create a project, add a device and create the `devices_registry` and `device_bindings` tables. 

`device_registry`: column(device_id, String), column(is_active, string)
`device_bindings`: column(device_id, String), column(owner_address, string)

Once the tables are created, we need to create the contract monitors: 

### Registry Contract Monitor

`Event Type`: OnDeviceRegistered
`Chain Id`: 4690
`Contract address`: your DeviceRegistry contract address
`Block Start`: Block number where your contract was deployed
`Block End`: 0
`Topic0`: 0x543b01d8fc03bd0f400fb055a7c379dc964b3c478f922bb2e198fa9bccb8e714


### Device Binding Contract Monitor

`Event Type`: OnDeviceBinding
`Chain Id`: 4690
`Contract address`: your DeviceBinding contract address
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

Let's go ahead and now run these tasks to register the device and bind our address to the device we previously created: 

```typescript
// register a new device
npx hardhat register-device --deviceid <YOUR_DEVICE_PUBLIC_KEY> --network testnet

// bind a device with an owner
npx hardhat bind-device --deviceid <YOUR_DEVICE_PUBLIC_KEY> --userid <YOUR_ADDRESS>
```

t's now time to use the data simulator and send data to our project. 

## Data Simulator

The data simulator package allows you to configure a script that will periodically send data messages to your W3bstream project. The first step is to copy the `.env.template` file into a `.env` file with the following fields: `PUB_TOKEN`, `PROJECT_NAME`, and `EVENT_TYPE` and replace the default values with your own ones. 

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

It's now time to run the script with: 

```bash
npm run start
```

The script will now run periodically (every 10 seconds by default) and each time a new message is sent, your wallet will be rewarded with a token. To visualize your new token rewards, simply import the token address in your metamask wallet, and make sure to be on the IoTeX Testnet. 

## Conclusions

Congratulations, you've created an end-to-end DePIN application that rewards device owners for providing real world data to your W3bstream project. 






