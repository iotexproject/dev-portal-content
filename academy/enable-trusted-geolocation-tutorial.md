import { Button, Alert, Link } from '@chakra-ui/react'

One possible use case for W3bstream is a layer-2 network that provides trusted GPS location of blockchain users to Web3 dApps. IoTeX implementedthe first of his kind trusted location service the form of a W3bstream-based API for Dapps. A corresponding feature in te ioPay wallet allows users to contribute their trusted locations to the API, while keeping the data transmission fully customizable and confidential.

In this document, we explain how to join the trusted location network from your iOS or Android device using ioPay, and monetize your data in dApps that use the API.

# Download ioPay Wallet
Make sure you have ioPay Wallet installed in your mobile phone. ioPay is the official wallet for the IoTeX Blockchain.

[Go to the ioPay website](https://iopay.me)

Once you installed ioPay, you'll have to go through the quick settings of your first wallet (or import the private key of a wallet you already own).

# Open W3bstream's geolocation module

In ioPay, select `Settings`-->`WebStream`-->`Geo Location`

<img width="1874" alt="image" src="https://user-images.githubusercontent.com/11096047/209715536-d6283558-787f-4fb2-a4d6-7b2183813c3a.png"></img>

# Bind the geolocation to your wallet

Once the geolocation module is opened for the first time, you will be asked to "Connect" the geolocation of your smartphone to the currenly selected ioPay wallet.
<Alert>
Make sure you selected the right wallet in ioPay before continuing. That wallet must be used to interact with supported dapp. e.g. to claim location-based NFTs or token rewards.
</Alert>

Click `Connect`, wait for the service to complete the setup process, then sign the action when the request pops up:


<img width="1874" alt="image" src="https://user-images.githubusercontent.com/11096047/209716376-936ea9a9-d506-4982-b913-c216f12d73a3.png"></img>

# Activate geolocation data

Make sure you activate the geolocation data, allow ioPay to access GPS data, finally open the ioPay browser to join your favorite dapp:

<img width="1874" alt="image" src="https://user-images.githubusercontent.com/11096047/209721749-a7532db2-3981-4292-a46c-9332f48eab30.png"></img>

# Final notes
When the geolocation module is "ON", ioPay will read your mobile GPS data and securely send your location to a W3bstream node with the frequency you configured in the settings. These location data are bound to the specific device, guaranteed to be tamper-proof and therefore ideal to be used inside dapps by means of the IoTeX unique trusted location API service based on W3bstream.
<Alert status="success">
Please notice that dapps will never be able to query your location. Dapps must in fact themselves **provide** a region within which they want to query a proof of location for one of your devices. You will be also requested to **sign** the query with your associated wallet for it to return any result.
</Alert>
The geolocation service can be shut off at any moment from ioPay, and you can also switch it on/off instantly to send just a single location when you are in a specific location that you intend to proof in the future.
