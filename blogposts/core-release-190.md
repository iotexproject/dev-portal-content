import {
  Alert,
  AlertIcon,
  Link,
} from '@chakra-ui/react'

The IoTeX core team is about to release the last major core upgrade of the year, before shifting to the IoTeX Core v2.0 expected in Q1 2023. Let's look at some of the key features that come with this core release. 

## 1.9.0 Hard Fork

v1.9.0 has a **hardfork**, which will be activated at **block height 21,542,761** (ETA is 1/11/2022 at 12am UTC) on IoTeX mainnet. 

## Main Features

v1.9.0 enables 2 important features:

- The zero-nonce feature: Namely, a newly created account will have 0 nitial nonce. For historic reasons, our mainnet is launched with that initial nonce value set to 1, which is different from Ethereum's nonce start, which is conventionally set to 0. Starting from v1.9.0, this behavior alligns with Ethereum's convention, **further enhancing** our chain's compatibility with the Ethereum eco-system. In particular, the Gnosis Safe protocol hinges on this zero-nonce property and, with the v1.9.0 launch, the IoTeX blockchain will then be able to be successfully integrated with it.
- Secondly, the **EVM** has been upgraded to **London** (and including Berlin), which enables many important EIPs. For example, EIP-2565 lowers **ModExp** gas cost and EIP-2930 provides an optional access list (a list of addresses and storage keys) to reduce gas costs when accessing these addresses and keys during contract execution. For a complete list of EIPs enabled, please check the sections "London" and "Berlin" at https://ethereum.org/en/history/

## Other Minor Improvements

v1.9.0 has also come with a couple of other minor improvements and fixes, including a complete fix to [the issue patched by v1.8.4](https://developers.iotex.io/posts/IoTeX-Core-Release-1.8.4).

## Full release Notes

<Alert status='success' variant='solid'>
  <AlertIcon />
The interested developer is encouraged to checkout the full release notes in the iotex core github repository, here: 
</Alert>


___ 


## Interested in running an IoTeX node?
If you are interested in setting up an IoTeX delegate from scratch, please refer to the official documentation: 

📚 [delegates.iotex.io](https://delegates.iotex.io)

