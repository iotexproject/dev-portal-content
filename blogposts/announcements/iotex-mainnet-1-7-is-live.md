---
title: IoTeX Mainnet v1.7 is LIVE!
description: Today marks another major release of the IoTeX blockchain and brings significant improvements to the IoTeX protocol. iotex-core 1.7.0 was released for node operators on March 15th and has just been activated. [Approximately 03/24/2022 around 11pm UTC]. This release brings many API improvements that will speed up dApp development and debugging of complex smart contracts. Node operators will benefit from a simpler architecture of the Ethereum API service and better log management. 
permalink: blogposts/announcements/iotex-mainnet-1-7-is-live.md
---

Today marks another major release of the IoTeX blockchain and brings significant improvements to the IoTeX protocol. [iotex-core 1.7.0](https://github.com/iotexproject/iotex-core) was released for node operators on March 15th and has just been activated. [Approximately 03/24/2022 around 11pm UTC]. This release brings many API improvements that will speed up dApp development and debugging of complex smart contracts. Node operators will benefit from a simpler architecture of the Ethereum API service and better log management. Database and network performance have increased as well. IoTeX is always building, refining, and working iteratively to facilitate easier dApp development and onboarding. In addition to dApps now being simpler to build, the blockchain is more reliable and more secure with enhanced performance. Let's take a closer look at some of the major changes...

## API

### Tracing contract executions

Blockchain contract executions can be complex to debug, especially if they involve calls to other contracts. The standard blockchain API in an EVM compatible platform usually only tells developers if an action was successful or reverted. The new `TraceTransactionStructLogs` API call is the equivalent of the Ethereum `trace_call` and collects low level details during the execution of a single contract call, providing developers with useful insights into what happened during the action execution. This API is available on any IoTeX node where the API gateway service is enabled.

### Contract storage decoding API

This API allows decoding the data in the storage of a smart contract at a specific memory location. It's provided as a native IoTeX GRPc API call (ReadContractStorage) and as an Ethereum JSON API call (eth_getStorageAt).

### Gas fee value in getActions API results

The value of the gas fee actually spent for a specific action was missing in the response object when querying action details. This value is now provided by any native or Ethereum API calls that return transaction details.

### Index values in transaction receipts and EVM logs

Upon 1.7.0 activation, when querying transaction receipts, the `transactionIndex` value as well as  and the `logIndex` value for each log entry in the EVM logs array, is now provided. The `transactionIndex` field provides the position of the transaction in the block and is useful when transactions ordering is important. `logIndex` provides the correct sequence of the EVM logs for a contract execution.

## Node-operation

### Native Ethereum JSON API

The blockchain release 1.2.0 marked a great milestone for IoTeX developers. With native support of Ethereum-signed transactions by the IoTeX node and the release of the Ethereum JSON API service ("Babel"), the IoTeX-Ethereum compatibility was complete. This allows any Ethereum dApp to be ported to IoTeX without requiring any change to the contracts nor to the client code. However, the Ethereum API was implemented as an external service that had to be deployed separately and "pointed" to an actual IoTeX node to make it work. With iotex-core 1.7.0, the Ethereum API server is now integrated natively and exposed directly by the IoTeX nodes. There is no need to run, configure and manage an external service. Simply enable the *Gateway* functionality of your IoTeX node to get both the IoTeX native API and the Ethereum API exposed.

### Log Rotation

Logrotate has been installed into the node Docker image to manage and store the node log files more efficiently. Instead of a single, big log file, the node now creates multiple, smaller files. Those that are too old get deleted. The logrotate configuration is [located in the Docker image](https://github.com/iotexproject/iotex-bootstrap#iotex-delegate-manual). The default settings create new log files daily. Log files get deleted after 30 days.

## Performance

### Separation of the p2p networks for Mainnet and Testnet

With this change, the IoTeX Testnet and Mainnet have been logically separated at the p2p network level based on the value of the ChainID that is now included in all p2p messages. This reduces network traffic interference between Mainnet and Testnet in some special cases and mitigates certain types of attacks.

## Misc

### Blockchain node execution tracing

In release 1.6.0, we introduced tracing code to collect running-time logs on critical execution paths of the node. This tracing code has now been enabled and configured to send the data to a backend where it gets aggregated and analyzed. This is a valuable tool for the IoTeX dev-core team to monitor the execution of the chain and get alerted promptly when weird behaviors or errors are detected. Anyone can take a look at the data by visiting https://tracing.iotex.me.

## More

Many other minor bug fixes and improvements have been implemented in 1.7.0. Check out the [release page](https://github.com/iotexproject/iotex-core/releases) on GitHub for more. We'd love you to be more involved. Please join our [Discord channel](http://iotex.io/devdiscord).
