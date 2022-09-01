*In this episode of IoTeXperts Live, Robert hosts from an undisclosed mysterious location, Simone and Giuseppe share some updates on the upcoming IoTeX 1.9.0 core release, a new tutorial series on our Developer Portal and a quick overview of the next release of W3bStream. Stick around till the end for our community's favorite 'Discord Updates by Discord Jeremi'.*

## Developer Updates

First off an update on the IoTeX core protocol, which can can found on our [GitHub](https://github.com/iotexproject/iotex-core). The EVM will be updated in accordance with the London EIPs: Improved transaction fee market, reduced gas refunds for EVM operations and prevented deployment for contracts starting with the *0xEF* bytecode.

There is also going to be a nonce update for new wallet accounts, which will now start with **0 nonce**, as opposed to **1** (as it was previously the case). The goal of this update is to be even more compatible with Ethereum (which starts from **0 nonce**) and make it easier for various projects to integrate with the **IoTeX** chain. Already existing accounts will be managed in a backward compatible way.

Developers can expect these, and more features with the **1.9.0** core release that is planned for mid September. 

The IoTeX **[Developer Portal](https://developers.iotex.io/)** official release is approaching, and in the meantime the team is constantly working on new content for our developers. The latest tutorial, **[A Blockchain-Powered Smart-Lock with Arduino](https://developers.iotex.io/posts/Blockchain-Powered-Smart-Lock)**, is the first chapter of a series of tutorials and blog articles that will focus on connecting IoT devices to our Layer-1 blockchain. 

As part of this tutorial series, we have also released a blog article that dives a little deeper into advantages and limitations of using blockchain as an IoT cloud, and introduces the numerous advantages of [W3bStream](https://docs.iotex.io/machinefi/w3bstream), IoTeX's Layer-2 solution to orchestrate a decentralized gateway network (i.e., W3bStream nodes) that streams encrypted data from IoT devices and machines to generate proofs to blockchains and streamline the development of [MachineFi](https://machinefi.com/) dApps.

## Dev Portal Improvements

The **Dev Portal News Letter** is coming soon! In the meantime, developers who sign up to our dev portal can subscribe to their content of preference and stay up-to-date as new material gets published by the team. The notification system will be improving over time to also account for other upcoming features, such as the developer public profile. These and more improvements are scheduled to be rolled out during the month of September, so stay tuned for more! 

## W3bStream Updates

The core team is working on multiple tracks ahead of the upcoming W3bStream release. W3bStream 0.1 and 1.0 are being developed in parallel. Version 0.1 will be a quick glance for developers who want to connect their data to the blockchain and start to experience some of the features that will appear on the official 1.0 release by the end of the year. 

The core team is also working on some SDKs. The hardware SDK will allow devices to easily communicate with the W3bStream node with just a few API calls. The mobile SDK, on the other hand, allows mobile applications to send data to W3bStream and it's expected to be release as a preview in the very near future. 

There is also a Demo Package in the works, that is built on top of what we have created for [MetaPebble](https://metapebble.app/), and it will be a working W3bStream application for mobile, that developers can use as a starting point to build their W3bStream applications. 

Due to the complexity of this decentralized architecture, IoTeX is also going to release the **W3bStream Whitepaper**.

Lastly, the creation of the MachineFi alliance, which will definitely improve the ecosystem and create an entry point for future companies to join in and access expertise and knowledge from the main actors in the IoT+Blockchain sphere. Through the Alliance, interested web2 companies could find an effective strategy to migrate to web3 and become MachineFi companies.  

## Live Audience Q&A

**Q: We are seeing some traction in MachineFi, but are there any updates on GameFi partners and developers projects?**

There have been a few projects that implemented Pebble trackers. We might have to wait a little bit longer for the release of W3bStream and its documentation and SDKs in order to see more GameFi projects surface in the IoTeX ecosystem. 

**Q: When can we expect an IoTeX smart-watch?**

When you build it! Jokes aside, we're not a hardware company, but we're very happy to help hardware companies to design these devices, and developers who are interested in experimenting. We have a few more hardware tutorials coming up on our portal, such as a Helium-like network on IoTeX built using an ESP32 or one of the latest Arduino boards. The goal is to explain how IoTeX is positioned compared to these other single purpose IoT applications that we see on the market. 

**Q: What is the ideal hardware for local home IoTeX nodes?**

RaspberryPi, as it is affordable, it's accessible, and there is a lot of material on it. Our [developers portal](https://developers.iotex.io/) already has some very useful tutorials on it. 

**Q: Any new hardware based on the smart lock concept or any other hardware or hardware updates?**

The idea behind these examples, where you have devices directly connected to the blockchain, is mostly done for educational purposes. While you could easily create a program for smart devices to talk directly to the blockchain, a Layer-2 network will always be needed when creating a MachineFi application for scalability and privacy reasons, and for more complex IoT logic. 

**Q: Are there any new ways to use the Pebble?**

Pebble tracker is still being used the way it was designed to, with a firmware that sends out data with a certain frequency to a W3bStream node, and it's then used to prove something to blockchain applications. If you want to use the Pebble tracker in a totally different way, then you could think of completely erasing the default firmware and using it as a development retail device. You could then program the device with your custom firmware and use it in whichever way it'd fit your application logic. 

## Discord Updates from Discord Jeremi

Our discord serves has seen about 240 active visitors per month, with about 25% of these visitors actually communicating. We therefore encourage you to join our [Discord](https://discord.gg/UNGadkTm) and be an active part of the community. Of these  visitors, 41% use only mobile, while 25% use desktop only and the rest use both.

We have received some messages regarding bugs with IoPay desktop, specifically related to connecting the IoPay desktop to the staking portal, and some issues connecting the IoPay desktop with the ledger. These issues have since been fixed! IoPay desktop is also being deprecated and it will no longer be developed with new features. 

There have been some questions about sending NFTs to IoPay desktop. Some users might have, for example, sent some NFTs from IoPay mobile to the hardware ledger, and are asking how to get them out of that ledger. There are a couple of workarounds: 

- Use your recovery phrase to export your private key and import it to Metamask and send the NFT that way. 
- Directly interact with the smart contract through IoPay desktop. 
- Import the NFT address as an XRC20 token. 

Stay tuned for more Discord updates on the next IoTeXperts Livestream.

___
**About The IoTeXPerts Livestreams**

*The IoTeXPerts Livestreams are a way for our community to stay in touch and get updated on the latest news from the world of IoTeX. Developers can join and share their projects built on our platform, maybe ask for advice or debugging tips, or even help other developers along the way and engage with other members of the community.*

*The livestreams happen every other Wednesday at 8:15PM UTC and can be accessed through our [Youtube](https://www.youtube.com/c/IoTeXOfficialChannel) channel.*











