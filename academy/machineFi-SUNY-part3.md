*Giuseppe De Luca, DevRel at IoTeX, stars in the third episode of this series of lectures about the MachineFi methodology developed by IoTeX, in front of Prof. Zhenhua Liu's students of the Stony Brook University of New York.*

## A Quick Recap: 

Let's quickly look at what we covered during the first two lectures of this series, before moving on: 

The first lecture of this series focused on a brief introduction to the fundamentals of blockchain, as well as its limitations when working with real-world data; an overview of the **MachineFi** methodology developed by IoTeX; and the hardware component of this **walk-to-earn** application.

The second lecture focused on the core component of a MachineFi dApp: **W3bstream**, the off-chain computing infrastructure, developed by IoTeX, serving as an open, chain-agnostic and decentralized protocol sitting in between blockchain and devices to convert real-world data streams from devices into verifiable, dApp-ready proofs. 


![w3bstream](https://user-images.githubusercontent.com/77351244/197877078-ea26ed26-d9dc-4e77-9192-71d0246a4d7d.png)


In the second lecture we also saw how W3bstream was able to monitor the specified contracts, and handle the corresponding events appropriately, by feeding the walk-to-earn contract the correct proofs. 

The architecture of this flow was then illustrated in the image below: 

![architecture](https://user-images.githubusercontent.com/77351244/197877244-db84f1dd-5b09-4ff6-88de-4749d4e81f69.png)


## Part 3 of 3

In this lecture, the focus will shift to the smart contracts used to register a device, bind a device to a blockchain account, create the token that will be used to reward users, and handle the bulk of the logic of the application: Sending requests to W3bstream, processing the proofs and allowing users to claim their rewards.

Once the smart contracts have been deployed through [Hardhat](https://hardhat.org/) (one of the most popular development environment tools for blockchain applications), it'll be finally time to put everything together by starting the accessory services (database, GraphQL API, and MQTT server), initializing the database and running W3bstream.

## Time to Walk

We'll see how "walking" at this point will result in W3bstream correctly monitoring the data message from our Arduino board, but disregarding it due to the fact that the device has not yet been registered on the blockchain. 

The students are then shown how to use Hardhat to create "tasks" that can be used to directly call on certain functions in the smart contracts. By calling these tasks we'll be registering the device on the IoTeX blockchain, and we'll then be binding this device to our Metamask wallet address. 

In doing so, we'll finally be ready to take a few more steps, make an "activity request", and claim our rewards that will directly show on our wallet. 
