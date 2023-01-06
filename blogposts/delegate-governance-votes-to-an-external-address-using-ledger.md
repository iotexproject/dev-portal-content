---
title: Delegate Governance Votes Using ioPay Desktop and Ledger
description: Learn how to delegate your governance votes using your Ledger, so that you can vote on new governance proposals using web3 or ioPay mobile wallets.
path: delegate-governance-votes-to-an-external-address-using-ledger.md
---

This short guide will help you vote on governance proposals in the event that you keep your stake buckets on your hardware Ledger wallet using ioPay desktop.

Currently, the governance portal https://gov.iotex.io/ allows you to vote on new governance proposals, however it is limited to web3 compatible wallets only (like Metamask or ioPay mobile). If you use ioPay desktop as your main staking bucket wallet, you can also participate in the voting, but you will need to complete a couple additional steps in order to vote.


# What you will need:

- ioPay desktop (latest version at time of writing is 1.0.19)
- Ledger Nano S / Nano S Plus / Nano X
- Ledger USB connection cable
- Computer with Operating System macOS / Windows / Linux
- ioPay mobile or a compatible web3 wallet (Metamask / Trust Wallet / etc)
- at least 10 IOTX in your ioPay desktop Ledger address

# Get ioPay mobile:

In order to complete the voting process, you will need to delegate your voting power from your ioPay Desktop Ledger wallet to a web3 compatible wallet, such as ioPay mobile. If you do not have a web3 wallet yet, you can install ioPay mobile on your mobile device by going to https://iopay.me/ and create a new wallet. 

[<img src="https://user-images.githubusercontent.com/63042547/190892685-a96be1f0-2d34-4700-9c5f-c2f02d57adb3.png">](https://iopay.me/)

If you need assistance in using ioPay mobile, you can look here for a guide https://iotex.io/blog/iopay-crypto-wallet-tutorial/.

# Connect Ledger to ioPay desktop 

Follow these basic steps to connect your Ledger to ioPay desktop. You can find more detailed instructions in the following link https://onboard.iotex.io/engage-with-iotex/ledger-nano-app

1. Connect your Ledger to your computer
2. Open ioPay desktop on your computer
3. Unlock Ledger
4. Open the IoTeX app on your Ledger
5. Unlock ioPay desktop using Ledger connection

![Ledger-ioPayDesktop](https://user-images.githubusercontent.com/63042547/190891897-d6a357d7-ef9f-4abb-9a82-55a8d6c9defd.JPG)

# Connect your ioPay desktop wallet to the staking portal

Once you have connected your Ledger to your computer and unlocked it, you are ready to connect ioPay desktop to the staking portal.

1. Go to https://stake.iotex.io/governance-delegation
2. Connect your ioPay desktop wallet to the staking portal

![Untitled](https://user-images.githubusercontent.com/63042547/190893559-45a5299f-a0d3-4a98-a34e-c9b280541e2e.png)

# Delegate Voting Power to your ioPay mobile address

Now that you have connected your ioPay desktop to the staking portal, you can delegate your voting power to another address. Make sure the address you delegate your votes to is a the address of your web3 wallet, otherwise you will not be able to vote. Please be advised that even if you make a mistake and delegate teh voting power to an incorrect address, you can always take back the delegate voting power action via the staking portal under the Governance Delegate section.

1. Enter in your ioPay mobile wallet address which you will use to vote for governance voting proposals
2. Click Submit to transfer the voting power
3. Approve the action in ioPay desktop
4. Approve the action in the IoTeX Ledger application

![VMwDYK](https://user-images.githubusercontent.com/63042547/190894068-b89851c9-e294-47a7-ba45-9c42aa2b8a44.png)

![Qceoec](https://user-images.githubusercontent.com/63042547/190894157-4b981d78-3f4b-48f8-89fd-28662f164c5a.png)

# Revoke the voting power delegation

You can always cancel an existing voting delegation by pressing the "Cancel this delegation" button and approving the action in ioPay desktop and your Ledger

![Screenshot from 2022-09-18 10-50-30](https://user-images.githubusercontent.com/63042547/190893964-786bbce2-1753-446b-853a-26f7772ca7a9.png)

# Notes

If after approving the transfer voting power action on your Ledger the transaction fails, it is likely due to the fact that you don't have enough native IOTX tokens in your ioPay desktop Ledger wallet for the gas fee. The gas fee will likely not be 10 IOTX, but that is the gas limit set on the staking portal for this action. When performing any action on the blockchain, the system checks if you have the minimum amount of tokens available in your balance to cover the gas limit. Therefore, if the transaction fails, check that you have at least the minimum amount of IOTX available on your ioPay desktop Ledger wallet balance.
