*During this episode of **IoteXperts Live**, we walked through the new **IoTeX Developer Portal**, looked at upcoming events for developers, and had amazing discussions about the tech behind the **MachineFi** methodology.*


## The IoTeX Developers Portal 
The *Developers Portal* is meant to be the go-to place for the growing IoTeX developers community to meet, learn and keep up-to-date with the latest news from the IoTeX world. It's constructed around three main sections.

The **Home** section which allows developers to immediately get started building on IoTeX and get updated on the latest news, the **Academy**, and the **Blog**. Whilst more sections are expected to be added over time, it is worth mentioning that the next addition to the portal will be about the [**Halo Grants**](https://community.iotex.io/c/halo-grants/61). By doing so, this platform will encompass each developer's experience from start to end: Devs will be able to get quickly started with IoTeX, learn more in depth through our Academy and the Blog and finally submit their own projects and get funded through the Halo Grants program.

The Portal's homepage is intedended to be the most immediate tool for devs to stay up-to-date with what's going on at **IoTeX**. The homepage has a live feed with IoTeX chain stats (which are always useful for devs) such as current block height, epoch, and transactions per second. For those who'd like to also be updated on any developer related content, the homepage will show the latest tweets from the our developers [Twitter account](https://twitter.com/iotex_dev), as well as the latest blog articles. 

Besides being a quick and useful tool to be kept updated, the homepage section also allows devs to immediately start experimenting with **IoTeX**. There are three prominent quickstarts on the page: The main one is about deploying a simple architecture to get started with a [**MachineFi**](https://machinefi.com/) dApp on IoTeX, while the other two are meant to give web3 devs and IoT devs an idea of what it's like to build on IoTeX. Web3 devs will be able to use the quickstart to deploy their EVM contracts and interact with the IoTeX blokchain, while hardware enthusiasts will get an idea of how to connect a device as a client of the IoTeX blockchain.

Next up is the Academy section, which devs can use to find and contribute educational content. Throughout the Academy, content can be filtered by category (*Quick Starts, Tutorials, Guides*) and by difficulty level (*Beginner, Intermediate, Advanced*). 

As mentioned above, developers contributions are extremely valued and are a very important aspect of the Academy. In this regard, the Academy can be seen as an archive of projects, demos, tutorials, quick starts, examples or even ideas that contributors might find worth sharing with the other members of the IoTeX developers community. Contributing to the Academy will allow any developer to build up their profile on the platform over time. 

Contributions to the portal are incentivized by a tipping mechanism. Developers have the possibility to connect their wallet to their profile and immediately receive tips for their content, while also supporting content created by other members of the community by tipping them in return. Through their profile page, devs will also be able to request test IOTX tokens for development purposes. If you'd like to know more about how to connect your MetaMask wallet to the IoTeX network, check the developers docs [here](https://app.gitbook.com/o/-MQ9LhchTp7_QJr-AYG0/s/-MUPHwAAaa4_zIrX70rA/get-started/iotex-wallets/metamask).

The tipping mechanism is not only a feature of the Academy, but pertains to the Blog section as well. Our goal with the blog is to allow experts in relevant fields to contribute educational articles to nurture the community and grow within it. 

Contributing to the platform is very easy, and it's clearly explained on our dev-portal-content [repository](https://github.com/iotexproject/dev-portal-content). While all the contributions to the platform are assessed internally at the moment, our goal is to start implementing DAO concepts in the future to make the developers portal fully driven and managed by our community. 



## Q & A from Livestream Participants

**Any news about the Pebble Tracker?**

Our main point of focus at the moment is to start teaching developers how to build using the [Pebble Tracker](https://metapebble.app/). As the portal grows, it will inevitably provide developers with more tools and knowledge to build applications that everybody can use. The increase in Pebble-related content for interested builders is strictly connected to the release of [**W3bStream**](https://app.gitbook.com/o/-MQ9LhchTp7_QJr-AYG0/s/-MUPHwAAaa4_zIrX70rA/machinefi/w3bstream-network), the Layer2 component of the IoTeX tech stack. After the release of **W3bStream**, expected by the end of the year, Pebble owners will be able to mine tokens, and use their devices to build applications. The standard cycle of a possible do-something-to-earn MachineFi application is illustrated below. 

![do-something-to-earn app cycle](https://user-images.githubusercontent.com/77351244/180025197-eeaefe06-9634-4906-9afa-b1abe28cffe3.png)



**Why does IoTeX use Microsoft Bing for the Pebble Tracker Map feature?**

That is just the component that's used on the MachineFi portal to display the map. It has nothing to do with the accuracy and the actual location of the device. The location is sent directly from the device itself, it's read from the GPS antenna, and it's not handled by any third party service. 
Currently the default firmware in the Pebble tracker will send verifiable data (including location) to a Layer2 node, which handles the device authorization and data integrity verification based on the device's identity stored on the blockchain. Upon success, the node will then save this data in a database. This databse is not only queried by the MachineFi portal, which then displays this information, but can also be openly accessed by any developer for testing purposes. The reason why the default firmware is sending a rounded GPS location is to avoid accurately tracking a person's location based on their device. The default firmware cycle is illustrated right below: 


![Default Pebble firmware cycle](https://user-images.githubusercontent.com/77351244/180023722-98523d72-b388-4d3b-82e2-ee6b8d589b4a.png)


Should a developer need an accurate location, for example, we have released a new feature for the Pebble tracker that allows an owner to redirect the data to a specific Dapp's node, choose which sensor/sensors to use and determine the accuracy of the GPS data that is sent by the device, as erquired by that Dapp. The Dapp developers can then implement their own IoT logic on their server, which will then interact with custom tokenomics on the IoTeX blockchain. 

![Custom configuration](https://user-images.githubusercontent.com/77351244/180024507-483488b8-6afb-47d1-8a48-0456e6b273ec.png)


It's worth mentioning that the Pebble tracker is already wrapping all the complexity of designing verifiable hardware and secure firmware. Each device has already been registered on the IoTeX blockchain, it already has tamper-proof hardware and provides tamper-proof data.  When a Dapp W3bStream node receives data, it can easily verify the signature associated with it, check against the public key that has been registered on the IoTeX blockchain, and thus ensure that this data is indeed coming from the Pebble tracker, and is not fake. Pebble is a finished, retail Web3 device and those Dapps for which Pebble's data is suitable, can be already developed by leveraging the exixting community of Pebble owners: they don't have to worry about how to design a secure device and can focus on their applicatuons logic.


**When will it be possible to connect other IoT sensors like a weather station?**

The Pebble tracker's sensors are pretty accurate for commercial use: They can detect temperature, humidity, air pressure and indoor air quality, so the Pebble tracker could already be used as a weather station. While the device's casing clearly makes it idoneous for personal use, the Pebble tracker is built so that its board can be opened and integrated into any specific product in order to allow for a wider range of use cases. 


**What is the importance of blockchain in IoT?**

The blockchain layer is essentially an authorization layer, which happens to be one of the biggest weaknesses of IoT. This layer could be used to store machine identities, for example, or allow the ownership of a machine to be verified and stored in a smart contract. Blockchain allows for a much greater security then what regular IoT solutions usually provide, just think about the more and more broadly used OTA firmware updates and what kind of security threats could arise with them. On top of security, blockchain opens IoT device owners to all the possibilities that MachineFi applications can provide by tokenizing real world resources, data and assets. Blockchain is going to have a huge role in the evolution of IoT and every single vertical in IoT could benefit from it. 


**Will there be a waterproof Ucam for outdoor use?**

We are not device manufacturers, but we lead by example. Based on the ongoing research, there could be a future product release that might be related to Ucam, but this is not guaranteed. 


___
**About The IoTeXPerts Livestreams**

*The IoTeXPerts Livestreams are a way for our community to stay in touch and get updated on the latest news from the world of IoTeX. Developers can join and share their projects built on our platform, maybe ask for advice or debugging tips, or even help other developers along the way and engage with other members of the community.*

*The livestreams happen every other Wednesday at 8:15PM UTC and can be accessed through our [Youtube](https://www.youtube.com/c/IoTeXOfficialChannel) channel.*

