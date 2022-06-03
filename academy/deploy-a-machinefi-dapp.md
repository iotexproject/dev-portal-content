---
title: Deploy a MachineFi Dapp
description: Create and deploy the minimal architecture for a MachineFi Dapp on IoTeX
path: academy/deploy-a-machinefi-dapp.md
---

```
# Clone the w3bstream-get-started repository
git clone github.com/iotexproject/w3bstream-get-started

# Set your testnet private key in hardhat.config.ts
<img width="1051" alt="image" src="https://user-images.githubusercontent.com/11096047/171911545-2c064288-873c-4f83-842d-db3aa5c4dbe0.png">


# Deploy the DevicesRegistry contract
cd w3bstream-get-started/blockchain
npx hardhat deploy

<img width="1154" alt="image" src="https://user-images.githubusercontent.com/11096047/171911935-16147cd0-1364-4437-86a8-747eb0e7ea07.png">

# Get DevicesRegistry contract address + deploy height and put them in project.yaml
nano projects/app/project.yaml

<img width="1385" alt="image" src="https://user-images.githubusercontent.com/11096047/171911288-d207f8a7-5629-4615-95b7-c136c37352d3.png">

# Register the device simulator address
npx hardhat registerDevice --deviceaddress 0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A --contractaddress <REGISTRY CONTRACT ADDRESS> --network testnet

# Start webstream
docker-compose up

# Start device simulator script
python3 device-simulator.py
```
