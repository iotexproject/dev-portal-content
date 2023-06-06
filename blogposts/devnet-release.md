[W3bstream](https://w3bstream.com/), developed by the [IoTeX](https://iotex.io/) core team, is a cutting-edge computational infrastructure designed to revolutionize the way applications can verify and utilize real-world data to tokenize user behaviors in the physical world. W3bstream effectively acts as the underlying infrastructure for DePIN (Decentralized Physical Infrastructure Network) projects, driving innovation and collaboration in the web3 landscape.

## W3bstream Devnet 

We are excited to announce the W3bstream Devnet release, a significant milestone aimed at providing developers with the tools and resources needed to build dApps on real-world data. This release is intended as a public node for developers who are interested in getting early access to this technology. 

The Devnet release comes together with [W3bstream Studio](https://devnet.w3bstream.com/), a powerful control center and user interfacing tool that allows developers to orchestrate every aspect of their W3bstream project from data reception to application logic and interaction with the blockchain. Key features of W3bstream Studio include:

1. Wallet Login: Developers can now log in to the Studio using their MetaMask wallet, allowing the same w3bstream node to be shared across different users
2. Trusted Metrics: Trusted Metrics provide a seamless way for developers to establish and monitor their project metrics, enabling the creation of insightful dashboards, offering flexibility in dashboard selection, and providing convenient access through an API.
3. [Database Creation](https://docs.w3bstream.com/get-started/video-how-to/create-database-tables): Studio enables developers to create database tables for storing and querying data streamed to their project, and for any other application-specific data.
4. [Applets & Triggers](https://docs.w3bstream.com/get-started/basic-concepts/applets-and-handlers): Applets define the logic of a project in the form of event handlers, while Triggers orchestrate the timing of handlers execution actions based on either on-chain events or incoming data messages. 
5. Chain agnosticism: W3bstream, and therefore the Devnet release, is chain agnostic. Besides IoTeX Mainnet and Testnet, there will be gradual support for the main L1s and L2s, such as Polygon Mainnet/Mumbai Testnet, Ethereum Mainnet/Goerli testnet. 
6. [Client SDKs](https://docs.w3bstream.com/client-device-sdks/introduction): The devnet release comes with four client SDKs to send IoT data to w3bstream, for Linux-based devices, ESP32, and Android/iOS devices, while Arduino boards will be supported soon. 


For device communication, W3bstream implements a messaging protocol that can be accessed on both HTTP and MQTT, with a dedicated endpoint for each user's project.

The big question at this point is: "What can I actually build right now?". The answer is: "Anything!". The W3bstream Devnet release actually allows you to build an end-to-end DePIN application, and, as mentioned earlier, control every step of the process through W3bstream Studio. 

## A typical flow of a W3bstream application

Let's dive deeper into what the flow of a possible application could be. Suppose you're building an application to manage a smart energy grid project that rewards or charges users based on the amount of energy that they contribute to or absorb from the grid. 

1. Data from a user's energy meter is sent to W3bstream
2. W3bstream will receive a data message, which is signed by the energy meter device.
3. W3bstream will route the message to the relevant handler function based on the W3bstream event that gets raised when a data message is received. 
4. Different handlers can then be called when either a certain blockchain event is emitted or periodically triggered by cron jobs.
5. The handler function will process the data message to verify data integrity and device identity, and optionally store the data in W3bstream's SQL database. 
6. Based on data received and on project specific requirements, a handler will generate a proof of some real world facts and submit it to a smart contract to trigger application specific tokeneconomy.

As previously mentioned, W3bstream Devnet currently operates as a single-node system. However, future upgrades will transition it towards a decentralized infrastructure. This will allow developers to determine the specific number of nodes required for their applications to run and select that consensus mechanism necessary to trigger on-chain actions based on their application's logic.

## What's Next?  

Besides decentralization, there are a few more interesting features that that devs can look forward to. Some might be on a closer timeline than others, but nevertheless a quick list of upcoming features is right below: 

- DePIN Dev Kits: IoTeX and Seeed studio are partnering to create DePIN dev kits to allow builders to directly plug in different sensors, visualize their data and seamlessly connect to W3bstream and start building. 
- Better developer experience and smoother on-chain interactions: With future iterations, on-chain interactions, and other features that are considered "standard" in a DePIN application will be streamlined. More and more applet templates will also be available. 
- More chains: With next iterations, more and more Ethereum compatible chains, such as Binance, will be supported. We might also see support for non-ethereum-like chains like Solana. 
- More modularity: In future iterations, developers will be able to use W3bstream as the central brain to orchestrate different components/services of their choice, such as storage, connectivity, etc... 

## Conclusion 

The W3bstream Devnet release brings a wealth of new possibilities for developers looking to create dApps that bridge the gap between the physical world and the blockchain ecosystem. By offering an easy-to-use, powerful backend solution through W3bstream Studio, we're taking a significant step towards realizing the full potential of DePIN and fostering innovation in the Web3 space.

We encourage developers to dive into the W3bstream Devnet release, explore its capabilities, and share their feedback with the IoTeX team. [Start building with us!](https://developers.iotex.io/) Together, we can shape the future of decentralized applications and unlock the power of real-world data in the Web3 ecosystem.
