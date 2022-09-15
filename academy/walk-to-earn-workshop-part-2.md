This is Part II of a series of 3 tutorials:

[Part I](https://developers.iotex.io/posts-7walk-to-earn-workshop-part-1) | [Part II](https://developers.iotex.io/posts-7walk-to-earn-workshop-part-2) | [Part III](https://developers.iotex.io/posts-7walk-to-earn-workshop-part-3)

# Part II
In the first tutorial of this series we left off at the step counter device that was sending steps data to the data oracle node, that we will configure in part III of this workshop.

In this second part we will focus on the blockchain component of our MachineFi application. To save some time, let's enter the blockchain project folder and install npm packages:

```bash
cd blockchain
npm install
```

For the sake of simplicity, we will use the ownable interface for all contracts and we will use a single private key to deploy all the contracts and this private key will be associated to the data oracle too. So that we (and the data oracle) are the only allowed to make changes to contracts state (in a production application you may want to implement access levels, at least to distinguish from the data oracle account and the actual contracts owner).

## The Data Oracle "service" contracts
We have already mentioned in the introduction that the data oracle needs two "service contracts" deployeo on the blockchain:

The DeviceRegistry and the DeviceBinding contracts. For the blockchain part we have a blockchain folder ready in the quick start that includes a simple HardHat project. So let's go ahead and quickly describe these two contracts that we will add to the contracts folder of the HArdhat project:

**DeviceRegistry.sol**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/access/Ownable.sol";
import "hardhat/console.sol";

import "./interfaces/IDevicesRegistry.sol";

contract DevicesRegistry is Ownable, IDevicesRegistry {
    event DeviceRegistered(address indexed _deviceId);

    event DeviceDeleted(address indexed _deviceId);
    event DeviceSuspended(address indexed _deviceId);
    event DeviceActivated(address indexed _deviceId);

    struct Device {
        bool isRegistered;
        bool isActive;
    }

    mapping(address => Device) public AuthorizedDevices;

    constructor() {
        console.log("Deploying DevicesRegistry contract");
    }

    modifier onlyRegisteredDevice(address _deviceId) {
        require(
            AuthorizedDevices[_deviceId].isRegistered,
            "Data Source is not registered"
        );
        _;
    }

    modifier onlyUnregisteredDevice(address _deviceId) {
        require(
            !AuthorizedDevices[_deviceId].isRegistered,
            "Data Source already registered"
        );
        _;
    }

    modifier onlyActiveDevice(address _deviceId) {
        require(
            AuthorizedDevices[_deviceId].isActive,
            "Data Source is suspended"
        );
        _;
    }

    modifier onlySuspendedDevice(address _deviceId) {
        require(!AuthorizedDevices[_deviceId].isActive, "Data Source is active");
        _;
    }

    function registerDevice(address _newDeviceId)
        public
        onlyOwner
        onlyUnregisteredDevice(_newDeviceId)
    {
        AuthorizedDevices[_newDeviceId] = Device(true, true);
        emit DeviceRegistered(_newDeviceId);
    }

    function removeDevice(address _deviceIdToRemove)
        public
        onlyOwner
        onlyRegisteredDevice(_deviceIdToRemove)
    {
        delete AuthorizedDevices[_deviceIdToRemove];
        emit DeviceDeleted(_deviceIdToRemove);
    }

    function suspendDevice(address _deviceIdToSuspend)
        public
        onlyOwner
        onlyRegisteredDevice(_deviceIdToSuspend)
        onlyActiveDevice(_deviceIdToSuspend)
    {
        AuthorizedDevices[_deviceIdToSuspend].isActive = false;
        emit DeviceSuspended(_deviceIdToSuspend);
    }

    function activateDevice(address _deviceIdToActivate)
        public
        onlyOwner
        onlyRegisteredDevice(_deviceIdToActivate)
        onlySuspendedDevice(_deviceIdToActivate)
    {
        AuthorizedDevices[_deviceIdToActivate].isActive = true;
        emit DeviceActivated(_deviceIdToActivate);
    }

    function isAuthorizedDevice(address _deviceId)
        public
        view
        override
        onlyRegisteredDevice(_deviceId)
        onlyActiveDevice(_deviceId)
        returns (bool)
    {
        return true;
    }
}
```

This contract is straightforward: it just stores a mapping that associates a unique ID (we use the "address" type for the ID) with a device data structure that keeps the status of the device: if it's registered (i.e. "Whitelisted" and if it's active or suspended - we are not using these anyway). The contract allows to register, unregister, suspend a device by its ID, and emit the respective events. We will only use the "DeviceRegistered" event, that is emitted when the device manufacturer (us!) manufacture a new device and registers (or "whitelists") it in the contract: this event is caught by the data oracle to sync a local database with the list of authorized device for best performance when Authorizing data.

Moving on to the next contract of the data oracle:
**DeviceBinding.sol**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/access/Ownable.sol";
import "hardhat/console.sol";


contract DeviceBinding is Ownable {

    // Devices ownership management
    struct Device {
        address ownerAddress;
        uint arrayIndex;
    }
    address[] public DeviceIds;
    mapping (address => Device) public OwnedDevices;

    // Keep track of how many devices an owner owns
    mapping(address => uint) public DevicesCount;

    // Events
    event OwnershipAssigned (address _deviceId, address _ownerAddress);
    event OwnershipRenounced (address _deviceId);

    constructor() {
        console.log("Deploying DeviceBinding contract");
    }

    function bindDevice(address _deviceId, address _ownerAddress) public onlyOwner returns (bool) {
        require(OwnedDevices[_deviceId].ownerAddress == address(0), "device has already been bound");
        
        AddDevice(_deviceId, _ownerAddress);

        emit OwnershipAssigned(_deviceId, _ownerAddress);
        return true;
    }

    function unbindDevice(address _deviceId) public returns (bool) {
        require(
            (OwnedDevices[_deviceId].ownerAddress == msg.sender) ||
            (msg.sender == this.owner()), 
            "not the device owner");

        removeDevice(_deviceId);

        emit OwnershipRenounced(_deviceId);
        return true;
    }

    function getDevicesCount() public view returns (uint) {
        return DeviceIds.length;
    }

    function getDeviceOwner(address _deviceId) public view returns (address) {
        return OwnedDevices[_deviceId].ownerAddress;
    }

    function getOwnedDevices(address _ownerAddress) public view returns (address[] memory) {
        address[] memory foundDevices = new address[](DevicesCount[_ownerAddress]);
        uint count = 0;
        Device memory device;

         for (uint i=0; i<DeviceIds.length; i++) {
            device = OwnedDevices[DeviceIds[i]];
            if (device.ownerAddress == _ownerAddress) {
                foundDevices[count] = DeviceIds[i];    
                count++;      
            }      
        }
       
        return foundDevices;
    }

    function AddDevice(address _deviceId, address _ownerAddress) private {
        OwnedDevices[_deviceId] = Device(_ownerAddress, DeviceIds.length);        
        DeviceIds.push(_deviceId);
        DevicesCount[_ownerAddress]++;
    }

    function removeDevice(address _deviceId) private {
        [ ... ]
    }
}
```

Again, this is a simple contract: it keeps track of what blockchain account "owns" a certain device. The most relevant functions are the **BindDevice** (which should be improved a bit, for example it should prevent binding a device that is not registered in the DeviceRegistry contract), and the **getOwnedDevices** that returns the list of devices owned by a certain account. This contract also has events that we could watch for in the data oracle to index the binding status in a local database, but for the sake of simplicity we will oly use this contract in the main Dapp when a user claims her rewards to make sure she actually owns a device (but we will not manage the case of multiple owned device).

## The WalkToEarn Dapp contracts
Time to take a look at our WalkToEarn Dapp, which basically implements the token economy of our MachineFi, and it's made of 2 smart contracts:

**STEP ERC20 Token**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract StepToken is ERC20 {
    constructor() ERC20("StepToken", "STP") {
        _mint(msg.sender, 10000000000 * 10 ** decimals());
    }
}
```

This is the token we'll use to reward users. It's a basic pre-minted ERC20 token and in a serious token economy this should be something more complex (e.g. a mintable/burnable token for example).

Finally, here comes the central contract of our Dapp: the WalkToEarn contract:

**WalkToEarn**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "hardhat/console.sol";
import "./DeviceBinding.sol";
import "./StepToken.sol";

contract WalkToEarn is Ownable {
    using Counters for Counters.Counter;
    
    uint16 public REQUEST_TIMEOUT_SECONDS = 60;
    // As an incentive, we can start with rewarding 1 token for each valid 
    // step walked. Should be lowered over time.
    uint public REWARDS_FACTOR;

    Counters.Counter private _currentReqId;

    DeviceBinding private _deviceBindingContract;
    StepToken private _stepTokenContract;
      
    struct PendingRequest {
        uint256 id;
        address deviceOwner;
        address deviceId;
        uint256 fromTime;
        uint256 toTime; // This also is the time when the request is created.
    }
    
    // Keep track of all requests waiting for a reply from W3bStream
    mapping (uint => PendingRequest) PendingRequests;
    
    struct User {
        uint256 lastClaimedTime;
        uint256 balance;
        uint256 pendingRequestId;
        bool hasPendingRequest;
    }

    // Keep track of every user's unclaimed balance 
    // and their last claimed time
    mapping (address => User) Users;

    // This event serves as a request for walking activity
    event ActivityRequested (
        uint256 _requestId, 
        address _userAddress,
        address _deviceId, 
        uint256 _fromTime, 
        uint256 _toTime); 

    constructor(address _deviceBindingContractAddress, address _stepTokenAddress) {
        console.log("Deploying WalkToEarn contract");
        _deviceBindingContract = DeviceBinding(_deviceBindingContractAddress);
        _stepTokenContract = StepToken(_stepTokenAddress);
        setRewardsFactor(10 ** _stepTokenContract.decimals());
    }

    // Attempt to claim rewards for walking activity for a specific device
    // We specify the device here to make things simpler: we could
    // just have w3bstream extract all devices owned by the user and return
    // aggergated rewards.
    function claimActivityRequest(address _deviceId) public returns (bool) {
        // Let's ignore any request from accounts that do not own a device
        address[] memory ownedDevices = _deviceBindingContract.getOwnedDevices(msg.sender);
        require(ownedDevices.length > 0, "no devices owned"); 
        bool isDeviceOwned = false;
        for (uint i=0; i<ownedDevices.length; i++) {
            if (ownedDevices[i] == _deviceId) {
                isDeviceOwned = true;
                break;
            }
        }
        require(isDeviceOwned, "device not owned");
        // One request at a time per user
        if (Users[msg.sender].hasPendingRequest) {
            uint256 lastRequest = Users[msg.sender].pendingRequestId;        
            require(isExpiredRequest(lastRequest), "already have a pending request");
            delete PendingRequests[lastRequest]; // Remove the expired request
        }
        // Emit a new W3bStream request to get an index of user's walking activity
        // from last claim time to current time
        uint256 reqId = _currentReqId.current();        
        uint256 lastClaimedTime = Users[msg.sender].lastClaimedTime;
       
        // Create a pending request for later processing        
        PendingRequest memory req = 
            PendingRequest(reqId, msg.sender, _deviceId, lastClaimedTime, block.timestamp);
        
        PendingRequests[reqId] = req;
        Users[msg.sender].pendingRequestId = reqId;
        Users[msg.sender].hasPendingRequest = true;
        // Emit the request to W3bStream
        emit ActivityRequested(req.id,  req.deviceOwner, req.deviceId, req.fromTime, req.toTime);

        _currentReqId.increment();
        return true;
    }

    // This function is called by W3bStream when a user's walking activity 
    // has been requested by the contract. We only want W3bStream to reply, 
    // for simplicity we assume the w3bstream server is the owner of this contract
    function claimActivityReply(
        uint256 _requestId, uint steps, bool _success, string memory _error) 
        public onlyOwner {        
        // Revert if there is no matching request pending (the server should 
        // use getPendingRequest to check if the request has been fulfilled)
        require(isPendingRequest(_requestId), "no pending request with this id (or it's expired)");
        // Revert on W3bstream failure
        require(_success, _error);
        // If all good, W3bStream returned the number of steps walked that are
        // worth of rewards. Let's update the user's balance accordingly.
        Users[PendingRequests[_requestId].deviceOwner].balance += steps * REWARDS_FACTOR;
        Users[PendingRequests[_requestId].deviceOwner].lastClaimedTime = 
            PendingRequests[_requestId].toTime;
        // Request fulfilled, remove it from the pending requests list
        Users[PendingRequests[_requestId].deviceOwner].hasPendingRequest = false;
        delete PendingRequests[_requestId];
    }

    function claimRewards() public payable {
        require(Users[msg.sender].balance > 0, "no activity");
        
        uint balanceToTransfer = Users[msg.sender].balance;
        Users[msg.sender].balance = 0;
        _stepTokenContract.transfer(msg.sender, balanceToTransfer);
    }

    // Check if a request is pending
    function isPendingRequest(uint256 _reqId) public view returns (bool) {
        if (PendingRequests[_reqId].deviceOwner == address(0)) return false;

        return 
            (block.timestamp - PendingRequests[_reqId].toTime)
            < REQUEST_TIMEOUT_SECONDS; // request is not expired
    }

    function isExpiredRequest(uint256 _reqId) public view returns (bool) {
        return !isPendingRequest(_reqId);
    }

    function getPendingRequest(uint256 _reqId) public view returns (PendingRequest memory) {
        if (isPendingRequest(_reqId)) {
            return PendingRequests[_reqId];
        }

        return PendingRequest(0, address(0), address(0), 0, 0);
    }

    function getUserSteps(address _userAddress) public view returns (uint256) {
        return Users[_userAddress].balance;
    }

    function getUserLastClaimedTime(address _userAddress) public view returns (uint256) {
        return Users[_userAddress].lastClaimedTime;
    }

    function setRequestTimeout(uint16 _timeout) public onlyOwner {
        REQUEST_TIMEOUT_SECONDS = _timeout;
    }

    function getRequestTimeout() public view returns (uint32) {
        return REQUEST_TIMEOUT_SECONDS;
    }

    function setRewardsFactor(uint _factor) public onlyOwner {
        REWARDS_FACTOR = _factor;
    }

    function getRewardsFactor() public view returns (uint) {
        return REWARDS_FACTOR;
    }

    function getBalance() public view returns (uint256) {
        return _stepTokenContract.balanceOf(address(this));
    }
}
```

This one is a bit longer, however the concept is very simple. The goal of this contract is to send rewards to users that have done walking activity. In order for the contract to know if a specific user who is attempting to claim rewards did actually walk and how many activity worth of rewards she's done, it will "query" the data oracle node.

This is managed as a simple request-reply cycle using contract events: if you are familiar with how the Chainlink oracle works, this concept is not new, it's just a simpler implementation. 

So let's start from the ClaimActivityRequest: basically this function is called by the device owner who wants to get "credited" the last activity she did from the last time the same function was called. But first, before querying the oracle, it'll make sure that this user (the caller) does actually own the step counter device she's attempting to get proof of activity for. This is done by reading the DeviceBinding contract. 

The user has to specify which device here, to make things simpler, but we could have the Data Oracle index the DeviceBinding contract and do the logic to get all devices for a user and return aggregated rewards.

Then we make sure there is not another pending request for the same user. And finally, it creates a new request for the oracle and keeps track of it. The request is simply "sent" to the oracle by emitting a request event.

At that point, the data oracle will detect the request, will extract the data for the requested device for the requested timeframe, add up all the steps and call back the WalkToEarn contract on the ClaimActivityReply function where it will provide the "proof" of steps in the specifict timeframe.

Now even if we have not implemented and real cryptographic proof, we can still trust this call because our data oracle is not decentralized but rather it's a single node that's under our control and it's also the owner of the contract and the only one who can reply to the WalkToEarn requests. 

In the future, when the computational data oracle will be a permissionless decentralized network, you will not be able to "trust" the reply of a node, and this "proof" will have to be more "convincing" (from a merkletree hash, to zero knowledge proofs).

So the rest of the contract is pretty straightforward: the ActivityRequestReply function will do some checks on the reply (I've also implemented a timeout, just to stay on the safe side), and eventually will credit the STEP tokens due to the user based on the number of steps received by the Data Oracle. Here we implement a simple "factor" that can be used to implement some kind of decreasing rewards to help bootstrap the ecosystem, but it could also be anything else more complicated.

For security reasons, we do not transfer the tokens directly but we just credit them to the user balance in the contract: We expect the users to explicitly withdraw his balance whenever they want to.

## Deploy the contracts
So time to deploy our contracts, first we add them to the blockchain/contracts folder:

blockchain/contracts/[DeviceRegistry.sol](https://github.com/simonerom/walk-to-earn-arduino/blob/main/blockchain/contracts/DevicesRegistry.sol)

blockchain/contracts/[DeviceBindig.sol](https://github.com/simonerom/walk-to-earn-arduino/blob/main/blockchain/contracts/DeviceBinding.sol)

blockchain/contracts/[StepToken.sol](https://github.com/simonerom/walk-to-earn-arduino/blob/main/blockchain/contracts/StepToken.sol)

blockchain/contracts/[WalkToEarn.sol](https://github.com/simonerom/walk-to-earn-arduino/blob/main/blockchain/contracts/WalkToEarn.sol)

Also, we need a deploy script for Hardhat:

blockchain/scripts/[deploy.js](https://github.com/simonerom/walk-to-earn-arduino/blob/main/blockchain/scripts/deploy.js)

And finally, I've also prepared a few Hardhat tasks in the hardhat config file to easily interact with the smart contracts (since we don't have any front end for the workshop)

**hardhat.config.ts**
```javascript
require("@nomiclabs/hardhat-waffle");
require("@nomiclabs/hardhat-web3");
require('dotenv').config()
const fs = require('fs');

const IOTEX_PRIVATE_KEY = process.env.IOTEX_PRIVATE_KEY;

task("registerDevice", "Authorize a new device by adding it to the DevicesRegistry contract")
  .addParam("deviceid", "The device id (can be anything of the size of an EVM address)")
  .addParam("registrycontract", "The DevicesRegistry contract address.")
  .setAction(async ({deviceid,registrycontract}) => {
    console.log("Registering device:", deviceid, ", to registry contract: ", registrycontract);

    const DevicesRegistry = await ethers.getContractFactory("DevicesRegistry");
    const devicesRegistry = await DevicesRegistry.attach(registrycontract);
    let ret = await devicesRegistry.registerDevice(deviceid);
    console.log ("registerDevice:", ret);
  });

task("bindDevice", "Binding device to an owner's account")
  .addParam("deviceid", "The device id")
  .addParam("owneraddress", "The device owner address.")
  .addParam("contractaddress", "The DevicesRegistry contract address.")
  .setAction(async ({deviceid,owneraddress, contractaddress}) => {
    console.log("Binding device:", deviceid, ",to owner: ", owneraddress);

    const DeviceBinding = await ethers.getContractFactory("DeviceBinding");
    const deviceBinding = await DeviceBinding.attach(contractaddress);
    let ret = await deviceBinding.bindDevice(deviceid, owneraddress );
    console.log ("bindingDevice:", ret);
  });

  task("claimActivityRequest", "Query W3bStream for my activity")
  .addParam("deviceid", "The device to request activity for. Must be owned by the caller..")
  .addParam("contractaddress", "The WalkToEarn contract address.")
  .setAction(async ({deviceid, contractaddress}) => {
    console.log("Requesting walking activity");

    const WalkToEarn = await ethers.getContractFactory("WalkToEarn");
    const walkToEarn = await WalkToEarn.attach(contractaddress);
    let ret = await walkToEarn.claimActivityRequest(deviceid);
    console.log ("claimRewards:", ret);
  });

  task("claimRewards", "Claim rewards for a device owner")
  .addParam("owneraddress", "The device owner address.")
  .addParam("contractaddress", "The WalkToEarn contract address.")
  .setAction(async ({owneraddress, contractaddress}) => {
    console.log("Claiming rewards for:", owneraddress,);

    const WalkToEarn = await ethers.getContractFactory("WalkToEarn");
    const walkToEarn = await WalkToEarn.attach(contractaddress);
    let ret = await walkToEarn.claimRewards();
    console.log ("claimRewards:", ret);
  });

  task("getUserSteps", "Chek the steps of a user")
  .addParam("owneraddress", "The device owner address.")
  .addParam("contractaddress", "The WalkToEarn contract address.")
  .setAction(async ({owneraddress, contractaddress}) => {
    console.log("Querying steps for user:", owneraddress);

    const WalkToEarn = await ethers.getContractFactory("WalkToEarn");
    const walkToEarn = await WalkToEarn.attach(contractaddress);
    let ret = await walkToEarn.getUserSteps(owneraddress);
    console.log ("STEPS:", ret);
  });


module.exports = {
  solidity: "0.8.4",
  networks: {
    hardhat: {
      gas: 8500000,
    },
    testnet: {
      // These are the official IoTeX endpoints to be used by Ethereum clients
      // Testnet https://babel-api.testnet.iotex.io
      // Mainnet https://babel-api.mainnet.iotex.io
      url: `https://babel-api.testnet.iotex.io`,

      // Input your Metamask testnet account private key here
      accounts: [`${IOTEX_PRIVATE_KEY}`],
    },
  },
};
````
The relevant part of this config file is the Hardhat configuration at the end: it's basically the exact same configuration you would do for Ethereum, but you use the IoTeX Testnet RPC as the network endpoint - and we caled this HArdhat network "testnet".

Before we can deploy, we need to create our private key for owning these contracts. You can create one in Metamask (and you can easily configure Metamask to connect to the IoTeX testnet by [adding the IoTeX RPC endpoint as a new network in Metamask](https://docs.iotex.io/get-started/iotex-wallets/metamask).

When the contract owner account is created we put in in a .env file to that it can be used by Hardhat scripts and tasks:

```
echo IOTEX_PRIVATE_KEY=0x1234ABCD > .env
```
You should also fund this wallet with some test IOTX tokens: you can access your [Developer Profile](https://developers.iotex.io) on the IoTeX developer portal, connect your Metamask wallet and hit the "Claim Test Tokens" button.

Ready to deploy?

```bash
cd blockchain
npx hardhat deploy --network=testnet
```
Here is the output:
```bash
Compiled 12 Solidity files successfully
Deploying contracts with the account: 0x169dc1Cfc4Fd15ed5276B12D6c10CE65fBef0D11
Account balance: 1684.895518  IOTX

Deploying DeviceBinding contract
DeviceBinding Contract
address: 0xBe88Eb846f186724d2908c52b618a3Ee675c31aE
block: 16349843

Deploying DevicesRegistry contract
DevicesRegistry Contract
address: 0xeA58CF82F82FC6e851DA0b54ef5C0E8E84e897cB
block: 16349844

Deploying StepToken contract
StepToken Contract
address: 0x696B5Bbfa1537C91380a9A096Ec70B4e3120e4f8
block: 16349845

Deploying WalkToEarn contract
WalkToEarn Contract
address: 0x80FF4c7CC31d27DC95ea3836CbCE31D363bD380f
block: 16349853

Funding the WalkToEran contract
Balance:  BigNumber { value: "10000000000000000000000000000" }  STP
```

Notice how we **transfer all the pre-minted STEP tokens** to the WalkToEarn contract to fund rewards distribution!

We have deployed the Layer-1 smart contracts for the oracle to function and for the users to get rewardsd. With all that, Part II is over and in Part III we will configure the data oracle to put everything together and see if we can get some STP tokens for our walking activity using the Arduino board!