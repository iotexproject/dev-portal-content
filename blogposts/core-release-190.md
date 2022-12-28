import {
  Alert,
  AlertIcon,
  Link,
} from '@chakra-ui/react'

## 1.9.0 Hard Fork

v1.9.0 has a **hardfork**, which will be activated at **block height 21,542,761** (ETA is 1/11/2023 at 12am UTC) on IoTeX mainnet. 

## Main Features

v1.9.0 enables 2 important features:

### Zero-nonce

The zero-nonce feature: Namely, a newly created account will have 0 initial nonce. The IoTeX mainnet was launched with the initial nonce value set to 1, which is different from Ethereum's nonce start, conventionally set to 0. Starting from v1.9.0, this behavior alligns with Ethereum's convention, **further enhancing** the IoTeX chain's compatibility with the Ethereum eco-system. In particular, the Gnosis Safe protocol hinges on this zero-nonce property and, with the v1.9.0 launch, the IoTeX chain will then be able to be successfully integrated with it.

### EVM upgrade to London
Secondly, the **EVM** has been upgraded to both **Berlin** and **London**, enabling many important EIPs. For example, EIP-2565 lowers **ModExp** opcode gas cost and EIP-2930 that mitigates contract breakage risks introduced by EIP-2929 as well as "unstucking" any contracts that become stuck due to EIP-1884. 


For a complete list of EIPs enabled, please check the sections "London" and "Berlin" at https://ethereum.org/en/history/.


Here's a list of EIPs enabled in London:

1. [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559): Fee market change for ETH 1.0 chain
2. [EIP-3198](https://eips.ethereum.org/EIPS/eip-3198): BASEFEE opcode
3. [EIP-3529](https://eips.ethereum.org/EIPS/eip-3529): Reduction in refunds
4. [EIP-3541](https://eips.ethereum.org/EIPS/eip-3541): Reject new contract code starting with the 0xEF byte
5. [EIP-3554](https://eips.ethereum.org/EIPS/eip-3554): Difficulty Bomb Delay to December 2021

and EIPs enabled in Berlin:

1. [EIP-2565](https://eips.ethereum.org/EIPS/eip-2565): ModExp Gas Cost
2. [EIP-2718](https://eips.ethereum.org/EIPS/eip-2718): Typed Transaction Envelope
3. [EIP-2929](https://eips.ethereum.org/EIPS/eip-2929): Gas cost increases for state access opcodes
4. [EIP-2930](https://eips.ethereum.org/EIPS/eip-2930): Optional access lists

## Upgrade Priority 

v1.9.0 comes with a hardfork, so all nodes must upgrade in order to keep syncing with the IoTeX blockchain. 

If you are running an IoTeX Delegate or a full node, please refer to the [Bootstrap repository](https://github.com/iotexproject/iotex-bootstrap#iotex-delegate-manual) for detailed instructions on how to update.

## Other Minor Improvements

v1.9.0 has also come with a couple of other minor improvements and fixes, including the final fix to [the issue patched by v1.8.4](https://developers.iotex.io/posts/IoTeX-Core-Release-1.8.4).

## Full release Notes

<Alert status='success' variant='solid'>
  <AlertIcon />
If you're interested in exploring the full release notes, feel free to checkout the iotex core github repository: https://github.com/iotexproject/iotex-core/releases/tag/v1.9.0
</Alert>


___ 


## Interested in running an IoTeX node?
If you are interested in setting up an IoTeX delegate from scratch, please refer to the official documentation: 

📚 [delegates.iotex.io](https://delegates.iotex.io)

