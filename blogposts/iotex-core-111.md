The IoTeX Core Release 1.11.0 opens a new frontier for developers, introducing the much-anticipated IIP-13 (**[Represent Staking Buckets As Non-fungible Tokens](https://github.com/iotexproject/iips/blob/master/iip-13.md)**) and IIP-14 (**[Account Abstraction via EntryPoint Contract](https://github.com/iotexproject/iips/blob/master/iip-14.md)**) features. With overwhelming support from our community, these updates are set to revolutionize the way staking mechanisms and account operations are handled in the IoTeX blockchain. This new version features a crucial hardfork, set to be activated on the IoTeX mainnet at block height 24,838,201 (anticipated to be around 07/20/2023, 11pm UTC). It is crucial for all nodes to upgrade to this latest release; those failing to do so will face sync issues with the IoTeX blockchain after the activation block.

Take note: before restarting your node, ensure you've upgraded to the latest config.yaml file, essential for v1.11.0's proper functionality. Detailed instructions for node configuration can be found [here](https://github.com/iotexproject/iotex-bootstrap#join-mainnet).

## Significant Developments in v1.11.0

Version 1.11.0 introduces three key enhancements:

- IIP-13: Staking Buckets as Non-fungible Tokens 
- IIP-14: Account Abstraction
- ChainID Enforcement in transactions
  
For a deeper technical understanding, we will soon publish two separate blogs focusing on IIP-13 and IIP-14. 

## IIP-13: Staking Buckets as Non-Fungible Tokens

IIP-13 is a breakthrough in the representation of staking buckets, now modeled as Non-fungible Tokens (NFTs) on the IoTeX blockchain. This transformation paves the way for innovative applications such as Liquid Staking Derivatives (LSD), enabling the trading of these buckets or using them as collateral in other DeFi protocols. As a result, we expect an increase in the overall staking ratio, enhancing the security and decentralization of the IoTeX blockchain. 

## IIP-14: Account Abstraction

The IIP-14 update focuses on improving the IoTeX platform's user experience by abstracting various account operations and properties, including authentication, authorization, replay protection, gas payment, batching, and atomicity. Based on EIP-4337, IIP-14 promises to make IoTeX more user-friendly and secure, overcoming the constraints of externally owned accounts (EOAs). 

## ChainID Enforcement: Enhanced Transaction Security

Building on the v1.8 release, which introduced ChainID into transactions to differentiate between networks, v1.11 now mandates that each transaction must carry the correct ChainID (1 for mainnet, and 2 for testnet), rejecting the default ChainID value 0. This further strengthens the security of the IoTeX blockchain.

## Additional Updates

Besides the major improvements, v1.11 includes several minor enhancements:

- An "ioctl bc delegate" command for delegate information retrieval.
- Console log output now provides a summary of node status.
- Enabled message batch in API service for increased network data efficiency.
- An "ioctl did service" command.
- System action validation is added to the block validation process.

The full release notes can be found [here](https://github.com/iotexproject/iotex-core/releases/tag/v1.11.0).

## Urgency to Upgrade

Since v1.11.0 introduces a hardfork, it is vital for all nodes to upgrade to keep in sync with the IoTeX blockchain.


## Stay Tuned

This blog post provides a general overview of the updates that come with v1.11. Stay tuned for the upcoming posts on both IIP-13 and IIP-14 as they will be particularly beneficial for developers, offering specific examples and comprehensive analysis of each improvement. Remember to [register to our Developer Portal](https://developers.iotex.io/) to get notified of new developer-oriented articles, Academy content and learn how to build real world-based projects with the IoTeX tech stack. 
