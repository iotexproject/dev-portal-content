In one of our latest tutorial series on a [Blockchain-powered smart-lock with Arduino](https://developers.iotex.io/posts/Blockchain-Powered-Smart-Lock), we looked at how a simple home automation device could be controlled remotely through a smart contract. In this case, the Arduino board controlling the smart-lock was directly communicating with the *Layer-1* blockchain (that was being used as a decentralized IoT cloud), by reading the state of the smart contract and opening/closing the lock accordingly.

Currently, the majority of people who use home automation devices rely on some sort of subscription-based cloud services, or on local installations of IoT software that allow them to remotely connect and control devices. Using blockchain as an IoT cloud represents a totally different approach which offers some remarkable advantages, as well as some drawbacks.

## Benefits of Blockchain

Blockchain has some intrinsic properties that, when inherited by an IoT application, could, in some cases, make a significant difference compared to centralized cloud services or open source home automation softwares: Just having the IoT logic of a specific application deployed as a smart contract (which can only be interacted with by blockchain accounts), would make it military-grade secure, censorship-resistant, immutable and traceable.

Despite being very secure, most popular centralized cloud services are a constant target for hackers; users need to blindly trust their security systems and rely entirely on these third party services, which could sell their data or even hide a data breach.

Centralized cloud services are no strangers to major hacks, proving that these security systems are often not enough. There are many examples of IoT devices being compromised or even controlled by malicious attackers. As the number of interconnected devices grows every day, while smart homes and OTA updates become the norm, so does the risk of potential cyber attacks. 

[These are only some of the biggest attacks that happened last year.](https://firedome.io/blog/top-cyber-attacks-on-iot-devices-in-2021/) 

Open source home automation softwares, on the other hand, require maintanance time, backup servers and backup network connectivity. In addition, users would have to open their home or office network to external incoming connections to talk directly to the IoT software that, in turn, controls the devices. This poses security concerns that are not easy to tackle without professional knowledge and a solid security policy. 

Using a smart contract would, instead, be much easier, and much safer from a security standpoint: The device would access the blockchain network as if it were a cloud, avoiding the need to receive incoming connections. Nobody would, furthermore, be able to take control of devices remotely, because they would somehow need to either crack the owner's private key or hack a blockchain (both things deemed not feasible, or almost impossible as of today).

One final benefit of using a public blockchain as a secure IoT cloud, is the fact that there is no limit on the number of *"Things"* that could be controlled.


## The Limitations

Blockchain benefits do not come for free: While reading the state of public blockchains is always free, modifying their state requires users to pay a *network fee*. Even when the fee is very low, this model may become expensive, depending on the use case and number of devices. However, when the number of devices is very high and the number of interactions is relatively low, the blockchain *pay-per-use* model may become even cheaper then some centralized services. Using the blockchain to control IoT devices might also cause potential latency issues. Unlike centralized, cloud-based IoT applications, blockchain is a shared underlying infrastructure for many dApps. As a result, users might experience significant delays when some dApps become very popular and the network gets clogged with a high volume of requests. In the case of our smart-lock application, this would mean that the lock might take several minutes to open!

It's also worth noting that, regardless of the state of a smart contract or the visibility level of its functions, reading the state of public blockchains is always possible just by running a full-node (given their nature, public blockchains don't inherently come with any storage encryption features as there is no centralized entity that would maintain a secret encryption key).

In terms of privacy, this is usually a big issue for most use cases and can pose a security threat: The state of one of our home automation devices could be exposed to the whole network! The only way to solve this issue, is to implement encryption features in the device firmware that would only allow device owners to decrypt the status of their devices. However, these techniques are hard to implement, especially when there are different state/data representation for different devices, since the user would have to make sure that it could never be possible for anybody to read the state of their devices by looking at the contract's history, or they would have to prevent replay attacks, etc...

Lastly, while smart contracts are a great way of executing trusted logic, they simply become too cumbersome or even prohibitively expensive as the complexity of IoT data logic increases. Blockchain is also not a good computational tool when certain complex IoT logic has to be applied to the data being fed by various smart devices.

## Layer-2 Scaling

As the diagram below illustrates, using blockchain as a cloud to establish a direct device-to-blockchain connection, is only one of the possible **IoT/Blockchain** integration patterns. 

![iot/blockchain integration](https://user-images.githubusercontent.com/77351244/187288280-a218a09c-2dab-4cb8-99c8-22f4c0da7c07.jpeg)

Ultimately, due to intrinsic blockchain limitations such as data storage cost, limited data processing options and lack of data privacy described above, applications that make use of blockchain as an IoT cloud are a viable solution only in a small set of use cases, like some machine-to-machine communications or low-frequency real-world public data feeds (e.g. weather data). For all the other IoT applications where data volumes are considerable, privacy is required, and blockchain fees are a problem, a different integration pattern that uses a Layer-2 network would have to be introduced. 

This Layer-2 network orchestrates the interaction between IoT devices and blockchain, and can be used to handle device data verification, complex IoT logic, data storage, data privacy and, possibly, data contribution to certain consumers. 

[W3bStream](https://docs.iotex.io/machinefi/w3bstream-network) is the Layer-2 solution developed by [MachineFi Lab](https://machinefi.com/), and is essentially a powerful real-world data computation and proof generation layer, for connecting the physical world to web3 with an innovative combination of blockchain and IoT. In a nutshell, W3bStream uses the *IoTeX blockchain* to manage device ownership and orchestrate a decentralized network of nodes to establish two-way communications between IoT devices in the physical world and decentralized web3 applications. 


Once a Layer-2 network is introduced in the IoT+Blockchain architecture to handle IoT storage, processing scalability as well as data privacy, the blockchain layer acts as a source of trusted authorization, trusted logic and handles the token economy based on the "proofs-of-real-world-facts" provided by the Layer-2 network.

![w3bstream roles](https://user-images.githubusercontent.com/77351244/187995484-b00b7bc6-7cd7-4714-9271-a17d18617879.png)

The diagram above illustrates how these different components come into play thanks to W3bStream:

-  **IoT devices** equipped with sensors are responsible for collecting data from the physical world.
-  **Users** are the owners of one or multiple IoT devices.
-  Each **W3bStream** node is an IoT gateway with computing, connectivity and storage capabilities.
-  The **Node Operators** are the administrators of the W3bStream nodes.  
-  **MachineFi** dApps represent a category of dApps which leverage IoT data and apply financial instruments to power the machine economy.

The diagram illustrates how a [MachineFi](https://machinefi.com/) application would generally work by leveraging all these different components, orchestrated by W3bStream. It shows the possibilities that a Layer-2 solution like W3bStream could open up for any type of IoT+Blockchain implementation. For the reasons explained throughout this article it's clear how, despite being a good starting point, the smart-lock example of an M2M application, just like other simple pay-per-use applications, represents only a limited use case and cannot exactly be considered a MachineFi dApp without the implementation of a Layer-2 network like W3bStream. 

The interested developer is encouraged to find out more in our [documentation](https://docs.iotex.io/machinefi/w3bstream-network). 

## Conclusions

Home automation is just one example of the myriad of verticals that could benefit from the intersection of blockchain and IoT. 

After the explosion and worldwide adoption of cryptocurrency, NFT's and the subsequent birth of decentralized finance, a new wave of web3 applications that connect to real world data is emerging even more rapidly. However, connecting IoT data to the blockchain **requires much more than just sending data from a device to a smart contract**. 

The *MachineFi* methodology developed by **IoTeX** aims to empower developers and organizations alike to be part of the **machine economy** and leverage the immense value of machine-generated IoT data. IoTeX provides all the developer tools and required elements for a successful design: From a fast, cheap and reliable Layer-1 blockchain to the most advanced Layer-2 IoT-data oracle in the world. 
