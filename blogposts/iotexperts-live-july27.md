*In this episode of IoTeXperts Live, **IoTeX** co-founder and CEO **Raullen Chai** talks about the future of W3bStream (IoTeX's Layer-2 solution to empower the development of **MachineFi** dApps), while our **IoTeXperts** (Simone Romano and Discord Jeremi) give some updates on new developer product releases, and share the floor to answer some very interesting questions from the audience.* 

## W3bStream Updates from Raullen Chai

The goal of **MachineFi** - IoTeX's methodology to empower the **Machine Economy** - is to harvest the incredibly large amount of data coming from the real world, and perform validations and computational operations to create proofs that can then be provided to smart contracts on a given blockchain. 

The MachineFi methodology uses three main components in its flow: A machine-integrated **SDK**; **W3bStream**, which is an off-chain computing layer; And a **blockchain layer** that hosts application-specific smart contracts. 

W3bStream is definitely the key component that connects everything together in a MachineFi flow, and it is ideally comprised of different nodes working together, on a specific consensus mechanism, to handle computing from the ground up. These nodes take some input data and perform application-specific validations and computations: A walk-to-earn application will have to implement a different computation logic than a drive-to-earn application. That is why we have a WASM virtual machine embedded in W3BStream as its core computation engine, allowing any developer to be able to program this virtual machine and start funneling the data. The computation then kicks off and starts feeding proofs to the blockchain. 

The IoTeX team has been working really hard on this, and the goal for the end of Q3 is to have three main products rolled out: A technical white paper; A developer MVP, which is being tested at the moment by different teams of developers in order to gather the necessary feedback for an official public release; And the SDK mentioned above. 

In short, the first official version of W3bStream, known as *First Glimpse*, will be out by the end of Q3, while a more comprehensive version will be released by the end of this year. 
This more comprehensive version will support different communication protocols, such as HTTP, HTTP2, web socket, and MQTT, as well as the WASM virtual machine embedded in the system, and full support of the **IoTeX** chain as well as **Ethereum 2.0**. This means that, for example, you could have a drive-to-earn token on Ethereum 2.0, but have all the data come through W3bStream, and the underlying IoTeX chain, which will then send the proof to Ethereum. Q4 will also see the release of different embedded SDKs for open source devices like RaspberryPi and Arduino. 

The longer-term plan is to make this a fully decentralized and scalable infrastructure, by having different entities stake **IOTX** and run the nodes, process streams and generate revenue through our revenue model which is being designed at the moment. 

## Q & A with Raullen:

**Q: Are applications such as Pebble Tracker and Pebble Go still going to be developed?**

The team is working on integrating [Pebble](https://iotex.io/pebble?gclid=Cj0KCQjwuaiXBhCCARIsAKZLt3mCU2LGgsJe0bbIUOwbCXtn4GbMCoqJKtoAp_mt_URRtw4RJux9iuAaAgDzEALw_wcB) and [MetaPebble](https://metapebble.app/) into this new tech stack, which will open up many use cases for many different types of applications. We have different teams currently working on Pebble applications, such as a travel-to-earn project, for example. Many more of these kinds of projects will start entering the market in Q4. 


**Q: So far IoTeX has partnered up with manufacturers to create devices such as the [Ucam](https://iotex.io/ucam) and the Pebble Tracker. Is there anything else, as far as hardware, that you're planning on releasing in the near future?**

We're not really a hardware company, but the reason why we built Ucam and Pebble is to learn how we can help developers with such products. We're now talking with some very promising hardware companies to embed our SDK into their products. Expect some interesting new releases in Q3 and Q4: From drive-to-earn applications built on a device that can be plugged into your car to let your car's data flow through the MachineFi tech stack, to applications that measure your health score and well-being thanks to a new IoTeX SDK embedded smartwatch. 


**Q: Would it be possible to develop the technology further with chip makers, for example, instead of creating separate devices? Is IoTeX working with such partners in order to integrate its technology into already existing hardware?**

This is on our roadmap, but we're starting with hardware that is easy enough to integrate. So we're working our way from the top, down to the chip level. It is definitely one of our long-term goals. 


**Q: What should we expect from IoTeX in the next 5 to 7 years, and what are some key differentiators that IoTeX is focused on, in comparison to other Layer-1 blockchains like Ethereum or Solana?**

We position ourselves as a gateway for machines to enter the world of web3. We are more of a horizontal layer, compared to other Layer-1 solutions, which are competing for the same things, e.g. DeFi or NFTs. All the machines and their data will be connected to IoTeX through W3bStream, but, thanks to integration with Ethereum 2.0, or even Polygon down the road, the actual tokenomics will take place in these other Layer-1 chains. IoTeX will then be the machine layer, bridging the gap between real-world data and web3. 


## Developer Updates: 

### IoTeX Analytics V2

A [new version of IoTeX Analytics](https://developers.iotex.io/blogposts/IoTeX-Analytics-V2-open-to-developers) has recently been released by the IoTeX team. *IoTeX Analytics* is an application built upon IoTeX core API which extracts data from IoTeX blockchain and reindexes them for applications to use via a GraphQL API. It is used by our internal products as well as many other projects in the IoTeX ecosystem to get a lot of blockchain data quickly and easily. 

The new version of *IoTeX Analytics* comes with **increased scalability** and increased support for **larger amounts of transactions and data**. There has been a major architectural upgrade aimed at separating the indexer and API services, which allows for better handling of erroneous data. *Analytics V2* also supports **multiple API methods** (e.g. *REST* and *GraphQL*), a **much higher query speed**, and many new APIs that have been added on top of the ones from V1. 

More details about this release can be found in its dedicated article on our developers portal, [here](https://developers.iotex.io/blogposts/IoTeX-Analytics-V2-open-to-developers). 


### Upcoming iotex-core protocol release

The next update is about the upcoming **IoTeX core software release**, which is going to introduce some new features that might be very interesting for our developers. 

Let's look at some of these improvements in more detail: 

First off is the **EVM upgrade**, in line with the **London release**. The next upgrade is about a custom precompiled smart contract that is added to the EVM, which will be able to run crypto-algorithms that use the **secp256r1 elliptic curve**. Having this elliptic curve implemented in the EVM as a precompiled smart contract allows IoTeX developers to perform cryptography using this primitive in a very cheap and efficient way. Having access to this type of cryptography - used, for example, by many popular IoT boards - will allow developers to easily verify messages signed by these boards in their smart contracts. This is a unique new feature in the next release of the IoTeX protocol, and it's going to be very useful for smart contract developers. 


### Arm Dev Summit

Next up is the [Arm Dev Summit 2022](https://devsummit.arm.com/) scheduled for the end of October, where our **IoTeXperts** will host a **MachineFi** workshop in person in San Josè (CA), for all the devs out there interested in learning about this new methodology and building the new blocks of the machine economy. The workshop's attendees will be able to learn about **DID** (decentralized id), **device ownership binding**, **verifiable data**, and how this data can be eventually used to generate proofs that can be fed to smart contracts and run any type of **x-to-earn application**. The details of the exact topic of this workshop are yet to be determined, but stay assured that it will be something on **IoT+Blockchain**, and something very cool! 



___
**About The IoTeXPerts Livestreams**

*The IoTeXPerts Livestreams are a way for our community to stay in touch and get updated on the latest news from the world of IoTeX. Developers can join and share their projects built on our platform, maybe ask for advice or debugging tips, or even help other developers along the way and engage with other members of the community.*

*The livestreams happen every other Wednesday at 8:15PM UTC and can be accessed through our [Youtube](https://www.youtube.com/c/IoTeXOfficialChannel) channel.*






