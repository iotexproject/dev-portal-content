import { Alert, AlertTitle, AlertDescription } from '@chakra-ui/react'

This is a quick start tutorial on deploying the [Simple Smart Grid](https://developers.iotex.io/posts/building-an-energy-efficient-smart-grid-that-rewards-responsible-users-with-w3bstream-and-the-iotex-blockchain) W3bstream example.

## Prerequisites

### Testnet Account

Make sure you have a Metamask Wallet configured and some IOTX test tokens. Here is a guide on how to do it: [Create an IoTeX Developer Account and fund it with test IOTX tokens](https://developers.iotex.io/community-posts/create-an-iotex-developer-account-and-fund-it-with-test-iotx-tokens)

### Software

You will need Node.js, git, and Assemblyscript installed in your system

## Part 1: Deploy blockchain contracts

Clone the repository containing W3bstream examples, cd into the smart grid example and open it in your favorite code editor:

```bash
git clone https://github.com/machinefi/w3bstream-examples
cd w3bstream-examples/simple-smart-grid
code .
```

In the same directory, install node.js dependencies:

```bash
npm install
```

Export your developer wallet private key (get it from Metamask):

```bash
export PRIVATE_KEY=<your key here>
```

Run the deployment script

```bash
npx hardhat run scripts/deploy.js  --network testnet
```

Cat .env to see deployment data

```bash
cat .env
```

outputs:

```bash
REGISTRY_CONTRACT=0xcf10260Ec867d6B71b05F1C7d9FA66AAdf969Ac2
BINDING_CONTRACT=0xB8c4b966A7E5134CdeCAE36924f367F97Cd6b161
TOKEN_CONTRACT=0x941D2B113983060dc0Fe42C5C1d89ad5a3eDd89f
DEPLOYED_HEIGHT=20021882
DEPLOYER_ADDRESS=0x00e27ACAF1d3D58861DF710719fc97C43fC976f6
```

## Part 2: Create the W3bstream project

Enter the W3bstream folder containing the W3bstream applet and install dependencies:

```bash
cd ../w3bstream
npm install
```

Edit the assembly/constants.ts file and replace the contract addresses with those you just deployed:

```typescript
export const ZERO_ADDRESS = "0000000000000000000000000000000000000000";
export const FOUR_TOKENS_HEX = "3782DACE9D900000";

// Customize with your own contracts
export const REGISTRY_CONTRACT = "0xcf10260Ec867d6B71b05F1C7d9FA66AAdf969Ac2";
export const BINDING_CONTRACT = "0xB8c4b966A7E5134CdeCAE36924f367F97Cd6b161";
export const TOKEN_CONTRACT = "0x941D2B113983060dc0Fe42C5C1d89ad5a3eDd89f";
```

Build the applet:

```bash
npm run asbuild
```

Find the applet file `release.wasm` in `simple-smart-grid/w3bstream/build` and use it to create a new project in W3bstream: call it simple_smart_grid:

<Alert>
  <AlertTitle>🎬</AlertTitle>
  <AlertDescription>Creating Projects: https://docs.w3bstream.com/get-started/w3bstream-studio/creating-projects</AlertDescription>
</Alert>

Open the project in W3bstream Studio, go to Settings and find your W3bstream operator account:

![w3bstream_screenshot](https://user-images.githubusercontent.com/64008830/235380407-c5738be4-5eea-4421-a780-955ba637559c.png)

Get back to the blockchain folder and add that account as a minter in the ECO Token contract:

```typescript
cd blockchain
npx hardhat addMinter --minteraddress YOUR_OPERATOR_ADDRESS --network testnet
```

Use MetaMask to send some test IOTX to the operator address (10 IOTX will allow for over 300 rewards transactions).

In your W3bstream project, delete the `DEFAULT` event route, and create two contract monitors with the following data:

<Alert>
  <AlertTitle>🎬</AlertTitle>
  <AlertDescription>Event routes: https://docs.w3bstream.com/get-started/w3bstream-studio/creating-strategies</AlertDescription>
  <AlertDescription>Smart contract monitors: https://docs.w3bstream.com/get-started/w3bstream-studio/triggering-events/monitoring-smart-contracts</AlertDescription>
</Alert>

**Registry Contract Monitor**

`Event Type` OnDeviceRegistered

`Chain Id` 4690

`Contract address` your DeviceRegistry contract address

`Block Start` find the deployment height in your .env into the blockchain folder

`Block End` 0

`Topic` 0x05d7f0c690676ba31675b45bcdb9ff4c34bb10744ec89d329eacd93c79ecc029



**Device Binding Contract Monitor**

`Event Type` OnDeviceBinding

`Chain Id` 4690

`Contract address` your DeviceBinding contract address

`Block Start` find the deployment height in your .env into the blockchain folder

`Block End` 0

`Topic` 0x9fd2c28ce9affee8592933156880418279ba95f7c71e344a71d1928a7c982979



**Create these 3 event routes:**

`DATA` --> handle_data

`OnDeviceRegistered` --> handle_device_registered

`OnDeviceBinding` --> handle_device_binding



**Create the database tables:**

`device_registry`: column(device_id, String), column(is_active, string)

`device_bindings`: column(device_id, String), column(owner_address, string)

`data_table`: column(public_key, String), column(sensor_reading, String), column(timestamp, String)

Send the following test message from the Log section:

<Alert>
  <AlertTitle>🎬</AlertTitle>
  <AlertDescription>Send test data message: https://docs.w3bstream.com/get-started/w3bstream-studio/triggering-events/testing-events</AlertDescription>
</Alert>

```JSON
[
    {
      "header": {
        "event_type": "DATA",
        "pub_id": "meter",
        "token": "",
        "pub_time": 1682616302032
      },

      "payload": {
        "data":{
          "sensor_reading": 5,
          "timestamp": 1682616723
        },
        "signature": "006f5adc7f000000906f5adc7f000000602a00a87f0000002e000000000000003c00000000000000906f5adc7f000000d06e5adc7f0000000100000000000000",
        "public_key":"046d443995cbaf0c4fdbb9163136ebcead9ee9c32023b7668384647a950fb0ca2450e8369f062720d91601fc0027373bd937e7ee59f019612b880e085b95cde3bc"
      }
    }
  ]
```

Check out the log, and it should fail with "Device not authorized". From the blockchain folder, let's register the device id in the DeviceRegistry contract:

```typescript
npx hardhat registerDevice --deviceid 0x6d443995cbaf0c4fdbb9163136ebcead9ee9c320 --network testnet
```

Let's also bind the device to our account as the owner:

```typescript
npx hardhat bindDevice --deviceid 0x6d443995cbaf0c4fdbb9163136ebcead9ee9c320 --owneraddress YOUR_METAMASK_ADDRESS --network testnet
```

And finally, find the token contract address into the .env file and import it in Metamask so that you will be able to check the balance of the token:

![metamask](https://user-images.githubusercontent.com/64008830/235380640-bf4667f7-dea2-4d4d-80c4-949e57b67e70.png)

Now, send the test message again, make sure the sensor_reading is "3.5" and check the logs: rewards should have been sent, and the balance in Metamask updated.

![logs](https://user-images.githubusercontent.com/64008830/235380684-0a79be5d-d4fb-47bf-af7b-6ebca214e8d2.png)
