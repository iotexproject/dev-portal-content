import { Alert, AlertIcon, AlertTitle, AlertDescription, Box } from '@chakra-ui/react';

## Introduction
Building a DePIN project necessitates a cohesive team with proficient skills in both Web3, IoT, and hardware development. The creation of an infrastructure that seamlessly integrates users, devices, data, and dapps in a secure and trusted manner is crucial. As the landscape evolves, the advent of Layer-2 open networks tailored for trusted machine data processing, such as [W3bstream](https://w3bstream.com), enriches the DePIN arena with enhanced composability.

<Alert>
Web3 and DePIN developers are now presented with two opportunities to bypass the intricacies of device design and production.
</Alert>

## Leveraging Web3 ready devices

Developers can opt for leveraging data from established, reliable web3-ready devices already compatible with W3bstream, and simply process the data emanating from such devices into their W3bstream projects to construct a DePIN application.

[Pebble Tracker](https://iotex.io/pebble), is a great example of a Web3-ready device. However, owing to the nascent state of decentralized Layer-2 infrastructures dedicated to IoT, devices of this nature remain uncommon.

## Solution: Leveraging traditional IoT devices

An alternative approach leverages existing IoT devices that feature an API for their users to get access to the data. With a streamlined sequence of steps, its easy to "convert" these devices to become "Web3 devices" and use their data into a DePIN project.

Within this guide, we delve into this method by harnessing the extensive network of DIMO-equipped vehicles and W3bstream. Our objective is to validate specific driving behaviors through W3bstream and prove them to a dapp that, in this example, would mint NFTs based on the cumulative distance driven by the user.

## The code

The project depicted in this guide is open source, and the complete code can be accessed on GitHub at github.com/machinefi/dimo-demo.

For a live demonstration, please visit [dimows.vercel.app](dimows.vercel.app).

## The final project

Let's take a glimpse into the appearance of this example DePIN app. The initial step involves enabling a DIMO user (i.e., an individual who possesses a car equipped with a [DIMO device](https://dimo.zone)) to register for our DePIN project. To achieve this, users are required to provide their DIMO API token within the Registration section of the application:

<img width="1490" alt="image" src="https://github.com/simonerom/dev-portal-content/assets/11096047/c60dfe8c-32d2-4343-ba6a-5e237ca82d62"/>

Once a user is registered, they can log in at any time by providing their API token again. The application will then display all the DIMO-equipped cars owned by that user.

![devices-list](https://github.com/simonerom/dev-portal-content/assets/11096047/3f92becc-32ca-41d8-8347-0c8155f59a34)

Subsequently, the user can initiate the data fetching from the DIMO API:

![sync-data](https://github.com/simonerom/dev-portal-content/assets/11096047/9ba28ada-6fb6-4814-bd20-ec5445faca72)

after retrieving data from the DIMO API and transmitting it to W3bstream, data processing takes place in our W3bstream project's applet. Finally, the app presents a collection of minted NFTs, giving users the ability to initiate the NFT withdrawal process:

![available-claims](https://github.com/simonerom/dev-portal-content/assets/11096047/cd3d2d5b-c7a2-4ead-a97b-7b3dbfb42668)


Now let's get into the details of the main steps of this application.

## User Registration and Device Binding in DePIN Projects

Similar to any Internet of Things (IoT) system, DePIN projects require the registration of authorized devices, linking each to a specific user who assumes the role of the device's "owner."

In a DePIN application, this process goes beyond traditional IoT setups. Apart from utilizing blockchain technology to securely store these device-user associations, we also establish a connection between users and their blockchain wallets. This linkage ensures that users can seamlessly receive any digital assets minted based on the data generated by their devices.

<Alert>
To delve deeper into these vital concepts, explore the [Trusted IoT Data](https://docs.w3bstream.com/applets-development/advanced-concepts/trusted-iot-data) section within the W3bstream documentation, and take a look at this insightful [Tutorial](https://developers.iotex.io/posts/manage-device-identity-and-binding-with-w3bstream) available on the IoTeX academy.
</Alert>

## The registration process

In the context of our example, the registration process includes the following actions:

1. Utilizing the DIMO API token provided by the user, access the user's data and retrieve the list of registered DIMO device IDs.
2. Register each distinct device ID within a designated registry contract on the IoTeX blockchain.
3. Obtain the user's wallet address through Metamask login and generate binding records within the binding registry contract. These records solidify the connection between each device ID owned by the user and their respective wallet address.

For detailed insights, the Device Registry, Device Binding, and SBT Token contracts can be located within the blockchain hardhat project directory of the project repository.

The following snippet of code showes how the Device Registry contract gets called to register a new device id on the blockchain:
```Typescript
// adapter/features/web3/services/viem/registrationContract.ts

export async function registerDevice(deviceIds: string[]) {

  /* ... */

  const { request } = await publicClient.simulateContract({
    account: walletClient.account,
    address: registryConfig.address as `0x${string}`,
    abi: registryConfig.abi,
    functionName: "registerDevices",
    args: [devicesToRegister],
  });
  const hash = await walletClient.writeContract(request);
  return publicClient.waitForTransactionReceipt({ hash, confirmations: 1 });
}
```

## Data fetching

Once a user has successfully registered their DIMO devices within our project, they can log in to the web application providing their DIMO API Token. (Note: In an ideal scenario, further enhancement could be achieved by incorporating an authentication mechanism such as OAuth 2.0, if supported by the utilized API).

Ideally, our web application would securely store the user's API token, orchestrating periodic data retrieval and seamless forwarding to our W3bstream logic. To provide clarity and facilitate the step-by-step process, we've introduced a dedicated "Sync Data" page. Upon logging in, users initiate a request to fetch the latest driving data. This acquired data is then seamlessly channeled into the W3bstream project, where it undergoes meticulous processing and analysis.

The following code snippets shows the DIMO client code that fetches the device data:
```typescript
// adapter/features/data/services/dimo/client.ts

/* ... */

// Create the Axios instance 
export const dimoDeviceDataAxiosInstance = (token: string) =>
  axios.create({
    baseURL: DIMO_DEVICE_DATA_API_BASE_URL,
    headers: {
      Accept: "application/json",
      Authorization: `Bearer ${token}`,
    },
  });
```
```Typescript
// adapter/features/data/services/dimo/get-driven-distance.ts

/* ... */

// Fetch the daily distance data from the DIMO API
const res = await dimoDeviceDataAxiosInstance(
    token
  ).get<DrivenDistanceResponse>(
    DIMO_DRIVEN_DISTANCE_API +
      `/${deviceId}/daily-distance?time_zone=America/Los_Angeles`
);
```



For each DIMO device associated with the user, the daily distance traveled since the last data fetch is requested to the API. Ultimately, this results in the generation of a JSON message, which the web application dispatches to W3bstream using the [W3bstream Client SDK for Node JS](https://docs.w3bstream.com/client-sdks/pc-client-sdks/node-js). The message structure is exemplified below:

```JSON
{
  "deviceId": "0xbca809a5...18ec722ed",
  "distances": [
    {
      "date": "2023-06-15",
      "distance": 28.796875
    },
     // ...
    {
      "date": "2023-06-19",
      "distance": 5.796875
    }
  ]
}
```

The message includes the unique DeviceId so that the W3bstream logic will be able to establish the device-owner correlation, as stored in the device binding contract. Subsequently, NFTs are minted and attributed to the wallet address of the respective owner. The next section illustrates this process.

## Inside the W3bstream Logic

Let's now move to the core of our DePIN project: the W3bstream logic that breaths life to our ecosystem.

While we won't delve into the details of creating and deploying a W3bstream applet, comprehensive quick start and developer documentation await at [docs.w3bstream.com](https://docs.w3bstream.com).

Contained within the repository's `applet/assembly` folder, the `handlers`` subfolder holds the different W3bstream handler functions for the project. We will skip the binding-related handlers (check out this [Academy Tutorial](https://developers.iotex.io/posts/manage-device-identity-and-binding-with-w3bstream) for a detailed explanation of these handlers).

Let's direct our attention to the `handle_receive_data` data handler, that fires when a fresh data message arrives from our web application:

```typescript
// data-receiver.ts

export function handle_receive_data(rid: i32): i32 {
  const deviceMessage = GetDataByRID(rid);
  const payload = getPayloadValue(deviceMessage);

  const deviceId = getDeviceId(payload);
  const auth = checkDeviceRegistrationAndActivity(deviceId);
  if (!auth) return 1;

  const distances = getDistances(payload);

  handleDistances(deviceId, distances);
  return 0;
}
```

The handler logic is self-explanatory:

1. Obtain the JSON of the message
2. Get the DeviceId from the JSON
3. Authenticate the device (i.e. make sure it's registered)
4. Get the distances array from the JSON
5. Store the distance data in the Project DB

The second relevant handler in the Applet is the `handle_analyze_data` function, that is executed periodically by our W3bstream project:

```Typescript
export function handle_analyze_data(rid: i32): i32 {
  const drivenDistances = getDrivenDistance();
  processDistances(drivenDistances);
  cleanDB();
  return 0;
}
```

The scope of this function is also pretty clear:

1. Gether recently added data from the Project DB
2. Process the data and, when required, mint an NFT
3. Clean the DB from just processed data

Step number two checks each data point - a reflection of daily distance traveled. When a certain distance threshold is crossed, an NFT gets minted for that day and attributed to the user's wallet, ready to be withdrawn.

## The NFT Contract

Although a DePIN project may have one or more complex smart contracts implementing different features required to run the ecosystem, in this simple example we are just using a stadard ERC1155 NFT contract that does not implement any custom logic. The Applet deployed to W3bstream is where the data processing happens and the user behavior is proven to directly mint an NFT. The source code of the NFT rewards contract can be found here:  
[https://github.com/nicky-ru/dimo-demo/blob/main/blockchain/contracts/DeviceRewards.sol](https://github.com/nicky-ru/dimo-demo/blob/main/blockchain/contracts/DeviceRewards.sol)

## Final considerations

In wrapping up this guide, it's essential to emphasize an interesting perspective. While the "API fetching pattern" demonstrated here does grant a DePIN project the versatility to easily leverage existing devices through their public APIs (including fetching data from data aggregation services like Google Fit or Apple Health), it's important to note that this approach need not be obligatory. A more seamless and robust solution resides in encouraging established IoT companies to seamlessly embed their systems within W3bstream's framework.

Empowering device owners to willingly channel their device data to a designated W3bstream project would improve user experience and serve as an althernative way to add value to existing IoT products.

To embark on this transformative path, IoT companies can harness the capabilities of [W3bstream Client SDKs](https://docs.w3bstream.com/client-sdks/pc-client-sdks), effectively streamlining the integration of their backend systems with W3bstream and open their devices to be used into any DePIN project.