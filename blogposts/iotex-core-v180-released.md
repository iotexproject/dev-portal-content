Two months after the release of version 1.7 of the IoTeX protocol, the core team of developers has just released the new version 1.8. 

This release represents a "hardfork" of the current protocol. It will activate at block **17,662,681** (approximately, 05/30/2022 UTC 11pm), and all block producers and delegates should upgrade the node software to latest official release [v1.8.0](https://github.com/iotexproject/iotex-bootstrap/releases/tag/v1.8.0). Full node installation instructions can be found [here](https://delegates.iotex.io/get-started/node-configuration/install-the-node-software).

Here's a detailed breakdown what’s new in this release.

## IoTeX Staking from Metamask

Let’s start with the most important feature: the ability to access the IoTeX staking from all **Ethereum wallets and software libraries**. An important goal, which required months of development and accurate testing by core developers, but absolutely necessary to complete Ethereum compatibility by exposing IoTeX custom staking transactions to Ethereum clients.

But what is it all about? To understand this, it's important to know that IoTeX staking actions, from the creation of the stake, to the choice of the delegated node, to the transfer of deposits, are implemented as **custom transactions that are part of the native protocol of the blockchain** and not, as some might think, as a smart contract.  

This choice provides great scalability to IoTeX staking, which can easily handle hundreds of thousands of staking deposits. It is also extremely cost-effective. Each staking action does not involve the execution of expensive smart contract calls, but just sending a **native transaction**, which only costs 0.01 IOTX, exactly like a simple IOTX token transfer transaction. On the other hand, these particular types of "staking transactions" are not part of the Ethereum protocol. They are **not known to Ethereum clients**. For this reason, until now it was not possible to interact with the IoTeX staking using an Ethereum wallet like Metamask, or Trust Wallet, but only through [ioPay](https://iopay.me/), the native wallet of the IoTeX blockchain.

So, how does it work? While the implementation details are not trivial, the solution is simple at a high level. With this update, the IoTeX blockchain has now a special "recipient address" (`0x04C22AfaE6a03438b8FED74cb1Cf441168DF3F12`) that is **hardcoded and controlled by the protocol**. It allows any Ethereum client to send staking transactions by **means of normal IOTX transfer transactions where the staking action data is encoded in the payload of the transaction**. Internally, the IoTeX protocol intercepts all transactions to this special address, decodes the staking action from the payload, and converts it into the native staking transactions to perform the action.

All the details about this new feature can be found in the [IIP-12 proposal for improvements](https://github.com/iotexproject/iips/blob/master/iip-12.md).


## Bug Fixes and more

Release v1.8.0 contains several additional fixes:

1. Improved the p2p network connection robustness to resolve the issue that a full-node could not join the Mainnet as occasionally reported by some delegates after upgrading to v1.7.1
2. Introduced chainservice builder to better manage service start-up and shut-down
3. Multiple code refactors and improvements for API module

You can find more detailed information at the official repository on GitHub: https://github.com/iotexproject/iotex-core

