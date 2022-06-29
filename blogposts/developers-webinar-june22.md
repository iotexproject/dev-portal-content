*Our June 22nd's Webinar focused on three very exciting things happening in the **IoTeX** ecosystem. 
First off, the release of the new iotex-core **v1.8.1**. We also have an update for our developers on the status of **W3bStream**, and a sneak peak at our very own **Developers Portal**. 
Let's look at all these one by one.*


## Upcoming iotex-core release v1.8.1

Since the initial preview of the IoTeX consensus back in 2018, Ethreum compatibility has always been an important theme for IoTex. 
The conseus was already integrating the native Ethereum EVM (so it has always been compatible with Ethereum Smart Contracts) yet it hasn't been Ethereum API compatible until now.  

<img width="900" alt="" src="https://user-images.githubusercontent.com/77351244/176372038-38447aa2-40ef-4bc7-9b12-c520232c3826.png">

We initially created a Babel Service, which had an IoTeX full node on one side and an ethereum client on the other - talking to this Babel Service was like talking to the Ethereum Node. 

This Babel Service would implement both Ethreum HTTPS API for any Ethereum call, and Web Socket API to subscribe to any blockchain event. 

One of the main points of focus of **v1.8.1** has been to bring this Babel Service **inside the full node**. Operators can now run an IoTeX full node (which is natively exposing both the IoTeX native API and the Ethereum HTTPS API as well as the Web Socket API) by starting it, configuring it as a gateway and just talking to it directly through any ethereum client.


## W3bStream Status Update

*IoTeX is not only a Layer1 blockchain, but it also acts as a trusted logic and authorization layer into a bigger context enabled by MachineFi.*

<img width="900" alt="" src="https://user-images.githubusercontent.com/77351244/176372360-9ed4fa0d-d704-4343-b36f-d73119ee36be.png">

**MachineFi** is a pattern, a methodology to jump-start machine economies. The IoTeX blockchain lies at its center, while developers, device owners and machines revolve around it. 

**W3bStream** is the Layer2 solution in between these entities and enables them to interact with each other. Let's look at some of the main points of focus of this flow: 

- New web3 machines could be acquired and registered into the IoTeX blockchain. 
- Machines could then send some telemetry received through W3bStream. 
- Developers have two entry points: either building **dApps** that make use of this data, or programming the **W3bStream** Virtual Machine to run the IoT logic for the data. 


It's worth mentioning, at this point, what is meant by **machine**: 

"A machine can be something programmable that generates telemetry autonomously, and it could be as small as a programmable sensor and as big as a car or a smart city". 

We can see that in this flow the machine's entry point is through W3bStream, and should therefore be designed for it, meaning that its hardware and firmware must have certain features. 

*What about legacy devices, can they not be part of the MachineFi economy?*

Yes, they can, and this is part of where the complexity of W3Bstream lies in. W3bStream has to, in fact, allow legacy devices to be onboarded on the new web3 architecture, but also account for the new machines specifically designed for web3, such as the **Pebble tracker** (an IoTeX native web3 device with firmware and hardware features that allow it to talk directly to W3bStream). 

The device legacy is just one of the many issues faced by W3bStream. Other issues are related to data privacy and security, as well as *centralization vs decentralization* and the trust management issues that go with it. 

In order to account for all these possible use cases and issues, the W3bStream architecture needs to be established as a set of architectures, and will eventually be a set of subnets, each with its own architecture to account for different use cases and different devices. 

The W3bStream Architecture will soon be featured on our [official documentation](https://docs.iotex.io/). 


## Developers Portal Preview 

The IoTeX developers portal is about to officially launch. Our goal is to create a one-stop-shop for any developer interested in building on IoTeX, and taking advantage of our community resources. 

The portal will feature some content on the Layer1 aspect of IoTeX. The more interesting aspect, though, will be about the MachineFi architecture, and how devices can directly interact with each other through smart contracts in a machine-to-machine communication network orchestrated by the blockchain's trusted logic. 

Some of the portal's initial content could be summarized into these categories: 

- W3bStream Preview Experiments.
- Firmware for Popular Devices.
- Smart Contracts and MachineFi-Oriented Guides.

The Developers Portal is meant to be a product created through the shared contributions of the IoTeX community. Developers will, in fact, be able to contribute educational or demonstrational content, and be rewarded for it. 

The portal will also features a blog, where IoT or blockchain developers could contribute with articles on their subjects of expertise. 


## Q & A

***Will the Pebble Tracker be produced for retail, retail users and the general public?***

Pebble was initially designed to be a developer's device, but it is intended to be a retail product. Once bought, though, a device has a default firware installed by IoTeX, and it therefore needs to be programmed to accommodate for a user's specific use case. *Pebble is therefore a programmable finished product*. As it is the case for apps on our smart phones, there will be, for example, MachineFi applications built for Pebble tracker, which will be made out of a firmware part, a W3bStream node to collect data and generate proofs of things, and smart contracts to manage the rewards and tokenomics. 


***Does IoTeX think that carrying a Pebble tracker will be as normal as carrying a smart watch or a smart phone?***

We hope this won't be the case. The current issue is that these devices, such as smartphones, are not web3 native. The Pebble tracker was designed to counter security limitations that are inherent to today's smart phones. Pebble is a design that is blockchain native, in terms of security, identity and tamper-proof hardware. The vision is to embed all these functionalities in our everyday devices, such as smartphones. 


***Is there anything you'd like to add to the concept of Machine Identity?***

IoTeX already has decentralized identity specifications that can be used for people, companies and devices. In the MachineFi design, every entity must have its own decentralized identity (DID) to openly interconnect with the others. As an example: Your node will have a DID. You - as the owner of the node - will have your own DID. Each device sending data to the node will have its own DID, and so will the device manufacturer. These concepts are already implemented by IoTeX through standard specifications. 


***Why would someone build a DeFi protocol on IoTeX instead of on another chain?***

The quick answer is: *"you should not build DeFi, you should build **MachineFi** on IoTeX".* 
The reason for this is that MachineFi is an evolution of DeFi. When DeFi will have access to the value that real world assets bring, when you can prove that a machine you own is doing some work, with verifiable data and a verifiable identity, then that data can then be tokenized through blockchain - that's the evolution of DeFi into MachineFi. The reason to deploy a DeFi application on IoTeX is that in the future such application will get access to a much bigger set of use cases than just web3 finance. 


***
***About The IoTeXPerts Livestreams***

*The IoTeXPerts Livestreams are a way for our community to stay in touch and get updated on the latest news from the world of IoTeX. Developers can join and share their projects built on our platform, maybe ask for advice or debugging tips, or even help other developers along the way and engage with other members of the community. 
The livestreams happen every Wednesday at 7:00PM UTC and can be accessed through our [Discord](https://discord.gg/bSGzaSvg) channel.*
***







