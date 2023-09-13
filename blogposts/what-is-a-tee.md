import {
  Alert,
  AlertIcon,
} from '@chakra-ui/react'

A **Trusted Execution Environment** is one of those fundamental topics that doesn’t get enough attention in discussions on IoT / connected devices. There are many areas where the integrity of data and the ability to execute programs in a protected environment is of paramount importance. Having highly sensitive information such as passwords, private keys, etc. processed and stored in a Trusted Execution Environment (TEE) provides the most reliable IoT data. 

Data that needs the highest level of security should come from a highly tamper resistant device, i.e. one that has a TEE. There are IoT data use cases that fall on a spectrum of required data integrity. These include financial transactions and health data at one end of that sprectrum. Sometimes the degree of integrity isn’t as important. But in some cases, it's mission critical and a TEE is warranted.

<Alert status='info'>
    <AlertIcon />
   A trusted execution environment (TEE) is a secure area of a device's hardware and software, designed to protect certain data and processes from external tampering or interference.
</Alert>


The **Pebble Tracker**, from IoTeX, uses a TEE and thus can be used when the greatest degree of data integrity is required. 

## What happens in this environment?

The [IoTeX](https://iotex.io/) Pebble Tracker uses a Trusted Execution Environment to ensure the integrity of the data by capturing and cryptographically signing real-world data. It's similar to the chips in your smartphone that manage FaceID/fingerprints and to crypto hardware wallets to manage private keys. The signature Pebble Tracker produces is then attached to the IoT data message. This allows it to be verified against tampering and enables the identification of the exact device that generated the data. Every Pebble Tracker, when manufactured, is registered on the [IoTeX](https://iotex.io/) blockchain.

With the blockchain + IoT technology companies [enviroBLOQ](https://envirobloq.io/) and AhoyDAO, third parties are paying money for the data from these devices these companies use. Those third parties rely on the fact that the data, or proofs based on that data, are completely reliable. That’s what makes the entire process from trusted device to any dApp groundbreaking. The integrity of the device data can be proven. This data integrity is foundational to a MachineFi economy.

For a general explanation, we found this Wikipedia entry on Trusted Execution Environment informative. (excerpt)


> *A trusted execution environment (TEE) is a secure area of a main processor. It guarantees code and data loaded inside to be protected with respect to confidentiality and integrity. Data integrity — prevents unauthorized entities from altering data when any entity outside the TEE processes data ... A TEE as an isolated execution environment provides security features such as isolated execution, integrity of applications executing with the TEE, along with confidentiality of their assets. In general terms, the TEE offers an execution space that provides a higher level of security for trusted applications running on the device than a rich operating system (OS) and more functionality than a 'secure element' (SE).*
   

## Where are TEEs used?

It turns out, Trusted Execution Environments are found everywhere in our lives. They're in smartphones, laptops, and other types of computing devices. TEEs are what allow these devices to safely store your fingerprint or use your face to Identify you.

TEEs can also be found in some types of smart cards, such as credit cards and passports, which use TEEs to store and process sensitive data in a secure environment. How else are they useful?

1. Protecting sensitive data: TEEs can be used to store and process sensitive data, such as passwords, financial information, and personal identification documents, in a secure environment, isolated from the rest of the device.
2. Enabling secure transactions to facilitate secure transactions, such as online banking and e-commerce transactions. A secure environment for processing sensitive financial information helps protect against fraud and ensures the integrity of financial transactions.
3. Enforcing security policies: TEEs can be used to enforce security policies, such as access controls, on a device. This prevents unauthorized access to sensitive data and ensures only authorized users can access certain resources.
4. Protecting against malware: TEEs can be used to protect against malware. By isolating certain processes from the rest of the device, TEEs can help prevent malware from spreading and infecting the device.
5. Ensuring the integrity of software updates: TEEs can be used to ensure the integrity of software updates. By verifying that software updates have not been tampered with before they are installed, TEEs can help prevent malware from being introduced through software updates.


## How will the data from these tamper resistant devices (using TEEs) be used in the days to come?

We checked in with **Simone Romano**, Head of Developer Relations, IoTeX and he had this to say: 


> *While ubiquitous in IoT devices, TEEs have virtually no uses in the blockchain industry, except for hardware wallets that use them to store users' private keys. But signing blockchain transactions offline is a very limited use of this technology. With a TEE in almost every smart device, they will certainly be able to sign and send transactions autonomously in the future, interacting with each other on the blockchain. Yet this is just the tip of the iceberg of what a TEE can enable when leveraged into blockchain applications. By registering a decentralized on-chain identity for each device, they will be able to use their TEE to autonomously generate and sign all the captured IoT data, which can then be used as the basis for generating evidence of real facts that smart contracts can verify on-chain, giving rise to the true decentralized economy based on trusted IoT data.*

