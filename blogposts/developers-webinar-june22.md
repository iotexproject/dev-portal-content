*Our June 22nd's Webinar focused on three very exciting things happening in the **IoTeX** ecosystem. 
First off, the release of the new iotex-core **v1.8.1**. We also have an update for our developers on the status of **W3bStream**, and a sneak peak at our very own **Developers Portal**. 
Let's look at all these one by one.*


## Upcoming Iotex-Core Release v1.8.1

Ethreum compatibility has always been an important theme for IoTex: the consensus mechanism was already integrating the native Ethereum EVM since the initial preview back in 2018, (so it has always been compatible with Ethereum Smart Contracts). However it hasn't been compatible with the Ethereum Gateway API until the release of the IoTeX Babel service in early 2021.  

![BabelNode](https://user-images.githubusercontent.com/77351244/176372038-38447aa2-40ef-4bc7-9b12-c520232c3826.png)

The Babel Service runs in the middle between an IoTeX full node and an Ethereum client and wraps Ethereum API calls into IoTeX native transactions - talking to this Babel Service was like talking to the Ethereum Node. 

This Babel Service would implement both Ethreum HTTPS API for any Ethereum call, and **Web Socket API** to subscribe to any blockchain event. 

One of the main points of focus of iotex-core from **v1.7.0** has been to bring this Babel Service **inside the full node**, and **v1.8.1** completed this implementation by also bringing the Web Socket Server inside the node's code. Operators can now run an IoTeX full node which is natively exposing both the IoTeX native API **and the Ethereum HTTPS API** as well as the Web Socket API, by simply starting it, configuring it as an IoTeX gateway and just talking to it directly through any ethereum client.

![newNode](https://user-images.githubusercontent.com/77351244/176623286-ce3d4d8c-a643-467c-a615-efd4f6d3b27d.png)


## W3bStream Status Update

*IoTeX is not only a Layer1 blockchain, but it also acts as a trusted logic and authorization layer into a bigger context enabled by MachineFi.*

![W3bStreamFlow](https://user-images.githubusercontent.com/77351244/176372360-9ed4fa0d-d704-4343-b36f-d73119ee36be.png)

**MachineFi** is IoTeX's methodology to jump-start machine economies. The IoTeX blockchain lies at its center, while developers, device owners, W3bstream nodes and machines revolve around it. 

**W3bStream** is the Layer2 solution that enables these components to interact with each other. Let's look at some of the main points of focus of this flow: 

- New web3 machines could have their identity registered into the IoTeX blockchain by the manufacturer. 
- Machine owners, once they acquire a device, could associate them to their own blockchain account.
- Machines could then send some telemetry to W3bStream layer-2 nodes. 
- Blockchain Dapps could use "Proofs of real-world facts" provided by W3bStream nodes
- Developers have several entry points: they could either build **dApps** that make use of these proofs; program the **W3bStream** node to run the IoT logic for the data; design the firmware for existing web3 devices; design a new web3 device itself, or any combinantion of the above, depending on how big of a part they want to have in the MachineFi development phases. 


It's worth mentioning, at this point, what is meant by **machine**: 

We can think of a "machine" as an object that is programmable and can generate some sort of telemetry autonomously: it could be as small as a programmable sensor and as big as a car or a smart city. 

We can see that in this flow the machine's entry point is through W3bStream, and should therefore be designed **for it**, meaning that its hardware and firmware must have certain features, designed with blockchin in mind. This cannot always be the case though, as most smart devices are currently built on top of a web2 architecture. 


## Developers Portal Preview 

The IoTeX developers portal is about to officially launch. Our goal is to create a one-stop-shop for any developer interested in building on IoTeX, and taking advantage of our community resources. 

The portal will feature some content on the Layer1 (deploying smart contracts) aspect of IoTeX. The more interesting aspect, though, will be about deploying MachineFi architectures, as well as simpler machine-to-machine communication designs where devices are directly orchestrated by the smart contracts deployed on the blockchain. 

Some of the portal's initial content could be summarized into these categories: 

- W3bStream Preview Experiments.
- Firmware for Popular Devices.
- Smart Contracts and MachineFi-Oriented Guides.

The Developers Portal is meant to be a product created through the shared contributions of the IoTeX community. Developers will, in fact, be able to contribute educational or demonstrational content, **and be rewarded for it** by anyone in the IoTeX community. 

The portal will also feature a **blog**, where IoT or blockchain developers could contribute with articles on their subjects of expertise. 


## Q & A from Livestream participants

**Can legacy devices then be part of the MachineFi economy, or not?**

Yes, they can, and this is part of where the complexity of W3Bstream lies in. W3bStream has to, in fact, allow for the new machines specifically designed for web3, such as the **Pebble tracker** to be onboarded on the new web3 architecture, but also account for legacy devices to be transitioned to W3bStream possibly without any change to the existing architecture nor to the device's firmware.

Dealing with traditional IoT devices is just one of the many issues faced by W3bStream. Other issues are related to data privacy and security, as well as *centralization vs decentralization* and the trust management issues that go with it. 

In order to account for all these possible use cases and issues, the W3bStream architecture needs to be established as a set of architectures, and will eventually be a network of subnets, each with its own architecture to account for different use cases and different devices. 

The W3bStream Architecture will soon be featured on our [official documentation portal](https://docs.iotex.io/machinefi/w3bstream-network). 

**Will the Pebble Tracker be produced for retail users and the general public?**

Pebble was initially designed to be a developer's device, and evolved during the [Crowdsupply campaign](https://www.crowdsupply.com/iotex/pebble-tracker) to become a retail product. Once bought, though, every device has a default firmware installed by IoTeX, and it therefore needs to be either configured or re-programmed to accommodate for an application's specific use case. *Pebble is therefore a programmable finished product*. As it is the case for apps on our smart phones, there will be, for example, MachineFi applications built for Pebble tracker, which will be made out of a firmware part, an IoT logic deployed on a W3bStream node to collect data and generate proofs of facts, and smart contracts to implement user rewards or the desired financial logic. 

**Does IoTeX expect that carrying a Pebble tracker will be as normal as carrying a smart watch or a smart phone?**

We hope this won't be the case. The current issue is that these devices, such as smartphones, are not web3 native. The Pebble tracker was designed to counter security limitations that are inherent to today's smart phones. Pebble is a design that is **blockchain native**, in terms of security, providing tamper-proof hardware and tamper-proof data, but it's intended to be the first step toward a future where these features will be embedded into our everyday devices, possibly also smartphones. 


**Is there anything you'd like to add to the concept of Machine Identity?**

IoTeX already has decentralized identity specifications that can be used for people, companies and devices. In the MachineFi design, every entity must have its own decentralized identity (DID) to openly interconnect with the others. As an example: Your W3bStream node will have a DID. You - as the owner of the node - will have your own personal DID. Each device sending data to the node will have its own DID, and so will the device manufacturer. These concepts are already implemented by IoTeX through standard specifications. 


**Why would someone build a DeFi protocol on IoTeX instead of on another chain?**

The quick answer is: *"You should not build DeFi, you should build **MachineFi** on IoTeX".* 
The reason for this is that MachineFi is an evolution of DeFi. When DeFi will have access to the value that real world assets bring, when you can prove that a machine you own is doing some work, with verifiable data and a verifiable identity, that data can then be tokenized on the IoTeX blockchain and be injected into DeFi protocols, even on different networks thanks to our [decentralized bridge IoTuBe](https://iotube.org/) - that's the evolution of DeFi into MachineFi. The reason to deploy a DeFi application on IoTeX is that in the future such application will get access to a much bigger set of use cases than just web3 finance. 


___
**About The IoTeXPerts Livestreams**

*The IoTeXPerts Livestreams are a way for our community to stay in touch and get updated on the latest news from the world of IoTeX. Developers can join and share their projects built on our platform, maybe ask for advice or debugging tips, or even help other developers along the way and engage with other members of the community.*


*The livestreams happen every other Wednesday at 8:15PM UTC and can be accessed through our [Youtube](https://www.youtube.com/c/IoTeXOfficialChannel) channel.*








