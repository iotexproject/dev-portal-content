---
title: Deploy a MachineFi Dapp
description: Create and deploy the minimal architecture for a MachineFi Dapp on IoTeX
path: academy/deploy-a-machinefi-dapp.md
---

```
# Clone the machinefi-getstarted-preview repository
git clone github.com/iotexproject/w3bstream-get-started

# Set your testnet private key in hardhat.config.ts


# Deploy the DevicesRegistry contract
cd w3bstream-get-started/blockchain
npx hardhat deploy


# Get DevicesRegistry contract address + deploy height and put them in project.yaml
nano projects/app/project.yaml


# Register the device simulator address
npx hardhat registerDevice --deviceaddress 0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A --contractaddress <REGISTRY CONTRACT ADDRESS> --network testnet

# Start webstream
docker-compose up

# Start device simulator script
python3 device-simulator.py
```
