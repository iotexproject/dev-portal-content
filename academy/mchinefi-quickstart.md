---
title: Get started with MachineFi
description: Learn how to deploy a running MachineFi architecture in a few steps.
path: academy/test.md
---

```
git clone github.com/iotexproject/machine-get-started

cd machinefi-get-started

cd blockchain

npx hardhat deploy

# Get DevicesRegistry contract address and deploy height and put them in project.yaml
nano projects/app/project.yaml

# Register the device simulator address
npx hardhat registerDevice --deviceaddress 0x19E7E376E7C213B7E7e7e46cc70A5dD086DAff2A --contractaddress <REGISTRY CONTRACT ADDRESS> --network testnet

# Start webstream
docker-compose up

# Start device simulator script
python3 device-simulator.py
```
