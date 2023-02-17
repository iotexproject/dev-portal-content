import { Button, Alert, Link } from '@chakra-ui/react'

One possible use case for W3bstream is a layer-2 network that provides trusted GPS location of blockchain users to Web3 dApps. IoTeX implementedthe first of his kind trusted location service the form of a W3bstream-based API for Dapps. A corresponding feature in te ioPay wallet allows users to contribute their trusted locations to the API, while keeping the data transmission fully customizable and confidential.

In this document, we explain how to join the trusted location network from your iOS or Android device using ioPay, and monetize your data in dApps that use the API.

# 1. Download ioPay Wallet
Make sure you have ioPay Wallet installed in your mobile phone. ioPay is the official wallet for the IoTeX Blockchain.

[Go to the ioPay website](https://iopay.me)

Once you installed ioPay, you'll have to go through the quick settings of your first wallet (or import the private key of a wallet you already own).

# 2. Enable W3bstream applications in ioPay
Access to W3bstream applications in ioPay is still limited to beta testers. You will have to enable it in ioPay to get access: make sure you access this tutorial from your smartphone with ioPay installed, then click the button below:


<Button colorScheme='pink'>
  <Link href='iopay://w3bstream_enable' isExternal>
    Enable W3bstream
  </Link>
</Button>

# 3. Open the Geo Location module

In ioPay, select `Settings`-->`WebStream`-->`Geo Location`

<img width="1874" alt="image" src="https://user-images.githubusercontent.com/11096047/209715536-d6283558-787f-4fb2-a4d6-7b2183813c3a.png"></img>

# 4. Bind geo location to wallet

Once the geo location module is opened for the first time, you will be asked to "Connect" the geo location of your smartphone to the currenly selected ioPay wallet.
<Alert>
  Make sure you selected the right wallet in ioPay before continuing. That wallet must be used to interact with supported dapp. e.g. to claim location-based NFTs or token rewards.
</Alert>

Click `Connect`, wait for the service to complete the setup process, then sign the action when the request pops up:


<img width="1874" alt="image" src="https://user-images.githubusercontent.com/11096047/209716376-936ea9a9-d506-4982-b913-c216f12d73a3.png"></img>

# 5. Activate trusted geo location data

Make sure you activate the geo location data, allow ioPay to access GPS data, finally open the ioPay browser to join your favorite dapp:

<img width="1874" alt="image" src="https://user-images.githubusercontent.com/11096047/209721749-a7532db2-3981-4292-a46c-9332f48eab30.png"></img>