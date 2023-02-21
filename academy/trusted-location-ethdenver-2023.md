*Injecting trusted location data into dApps opens the door for several new types of applications. The application we're going to be building allows users to create location based Airdrops that can be unlocked only by those devices that were in the Airdrop's specified location area within the time range determined by the Airdrop's creator.*

## Architecture

Let's look at the architecture of a typical dApp that would make use of the trusted location API and the on-chain Verifier contract, developed by the IoTeX core team. 

![geo-location-flow](https://user-images.githubusercontent.com/77351244/218823510-44d3e15e-3396-4456-881b-1ea7e2c26fe8.png)

The image above describes the flow of an application where enabled devices can send their GPS locations to a W3bstream server, and device owners can then allow the dApp to query that W3bstream server to obtain a proof of their presence in a certain location at a certain time. This proof is then used by the dApp to, in this case, create a digital asset, such as an airdrop. The final step before the airdrop is actually minted is to verify the proof on the IoTeX chain using the "Verifier Contract"

## App Setup

The only pre-requisite here is to be able to send a trusted location to a W3bstream node. In order to do so, you can check out [this tutorial on the IoTeX Developer Portal](https://developers.iotex.io/posts/enable-trusted-geolocation-tutorial). 

Once you're able to successfully send a trusted location, open a new project folder, fork and clone the *trusted-location-starting-template*, like this: 

```bash
git clone -b starting-template --single-branch https://github.com/machinefi/trusted-location-airdrop-dapp.git
cd trusted-location-airdrop-dapp
```

Inside the project you'll find the `frontend` and the `web3` directories. Enter each directory and install the respective dependencies with the `npm install` command. 

```
cd web3 && npm install
cd ../frontend && npm install
```

Now, from the `frontend` directory run the `npm run dev` command and open your browser at http://localhost:3000/ - you will see the app's landing page. Nothing too exciting there, our app is currently reading from a blank smart contract. 

```
npm run dev
```

Let's go ahead and take care of this first. 

## The LocationAirdrop Contract

Inside your code editor, get to the `contracts` folder in the `web3` directory and open the `LocationAirdrop.sol` file.

Let's add these 2 lines, to import the `LocationNFT` and the `VerifierInterface`.

``` solidity
import "./LocationNFT.sol";
import "./VerifierInterface.sol";
```

We'll use the NFT contract to mint our Airdrop once certain conditions are met, and we'll use `VerifierInterface.sol` to interact with the on-chain Verifier suite of contracts developed by the IoTeX core team, to verify the validity of the proofs forwarded to the contarct. 

Let's go ahead and start coding the `LocationAirdrop` contract by adding our global scope:

```solidity
    event Claimed(address indexed holder, bytes32 indexed deviceHash);

    VerifierInterface public verifier;
    LocationNFT public NFT;

    address public feeReceiver;
    uint256 public baseFee;
    uint256 public feeCliff; // value after which creating more tokens will be free

    struct AirDrop {
        int256 lat;
        int256 long;
        uint256 maxDistance;
        uint256 time_from;
        uint256 time_to;
        uint256 tokens_count;
        uint256 tokens_minted;
    }

    bytes32[] public airDropsHashes;
    mapping(bytes32 => AirDrop) public airDrops;
    mapping(address => mapping(bytes32 => bool)) public claimedAirDrops;

    uint256 private _tokenId;
    
    constructor(
        address _verifier,
        address _nft,
        uint256 _baseFee,
        address _feeReceiver,
        uint256 _feeCliff
    ) {
        verifier = VerifierInterface(_verifier);
        NFT = LocationNFT(_nft);
        baseFee = _baseFee;
        feeReceiver = _feeReceiver;
        feeCliff = _feeCliff;
    }
```

This is pretty straightforward: Upon deployment, we'l have to provide this contract with the Verifier and NFT contract addresses, as well as a series of fees, that just make the whole app a little more fun. 

Notice the `Airdrop` struct. We'll use this a lot while building this app: Each Airdrop will have a `latitude`, `longitude`, a `maxDistance` (out of which the Airdrop won't be valid), a time interval - `time_from` and `time_to` -  and it will keep track of how many tokens have been created in general, and how many have been minted upon before each call. 

The on-chain storage is going to be split between an array of bytes32 called `airDropsHashes` which will keep track of the hashes of all the Airdrops created, a mapping that maps a hash to its corresponding Airdrop - `airDrops` - and finally a double mapping which we'll use to figure out if a specific address has already claimed an Airdrop or not. 

Let's add a few auxiliary functions: 

```solidity
    function airDropsCount() external view returns (uint256) {
        return airDropsHashes.length;
    }

    function getAllHashes() public view returns (bytes32[] memory) {
        return airDropsHashes;
    }

    function generateHash(
        int256 _lat,
        int256 _long,
        uint256 _maxDistance,
        uint256 _time_from,
        uint256 _time_to
    ) internal pure returns (bytes32 _hash) {
        _hash = keccak256(
            abi.encodePacked(_lat, _long, _maxDistance, _time_from, _time_to)
        );
    }
```

This function checks for the input values to be valid, then generates a hash for the Airdrop the user is about to create, makes sure it's not a duplicate and updates out on-chain storage: The `airDrops` mapping for the generated hash and the `airDropsHashes` array. 

The next step is to create a function to add a new AirDrop to the Dapp:

```solidity
 function addAirDrop(
        int256 _lat,
        int256 _long,
        uint256 _maxDistance,
        uint256 _time_from,
        uint256 _time_to,
        uint256 _tokens_count
    ) external payable {
        require(
            _lat >= -90_000000 && _lat <= 90_000000,
            "Invalid latitude value"
        );
        require(
            _long >= -180_000000 && _long <= 180_000000,
            "Invalid longitude value"
        );
        require(_maxDistance > 0, "Invalid max distance");
        require(
            msg.value >= calculateFee(_tokens_count),
            "Value sent with tx is not sufficient based on the tokens count"
        );
        bytes32 airDropHash = generateHash(
            _lat,
            _long,
            _maxDistance,
            _time_from,
            _time_to
        );
        require(airDrops[airDropHash].maxDistance == 0, "Duplicated airDrop");
        airDrops[airDropHash] = AirDrop({
            lat: _lat,
            long: _long,
            maxDistance: _maxDistance,
            time_from: _time_from,
            time_to: _time_to,
            tokens_count: _tokens_count,
            tokens_minted: 0
        });
        airDropsHashes.push(airDropHash);
        _tokenId++;
    }
```

The last important function is `claim()`, which does the bulk of the work in our contract: 

```solidity
function claim(
        int256 _lat,
        int256 _long,
        uint256 _distance,
        uint256 _time_from,
        uint256 _time_to,
        bytes32 _deviceHash,
        bytes memory signature
    ) external payable nonReentrant {
        // verify proof of location
        bytes32 digest = verifier.generateLocationDistanceDigest(
            msg.sender,
            _lat,
            _long,
            _distance,
            _deviceHash,
            _time_from,
            _time_to
        );
        require(
            verifier.verify{value: msg.value}(digest, signature),
            "Invalid proof of location"
        );

        // verify that an AirDrop claim doesn't exists for this address
        bytes32 airDropHash = generateHash(
            _lat,
            _long,
            _distance,
            _time_from,
            _time_to
        );
        require(
            claimedAirDrops[msg.sender][airDropHash] == false,
            "AirDrop has been already claimed for this address"
        );
        // verify that the Airdrop exists and it's available
        AirDrop memory airDrop = airDrops[airDropHash];
        require(airDrop.maxDistance > 0, "AirDrop does not exist");
        if (airDrop.tokens_count > 0) {
            require(
                airDrop.tokens_count - airDrop.tokens_minted > 0,
                "No more tokens left for this AirDrop"
            );
        }

        //update the mapping
        claimedAirDrops[msg.sender][airDropHash] = true;

        // mint the location NFT
        NFT.safeMint(msg.sender);

        // emit the event
        emit Claimed(msg.sender, airDropHash);

        // update the number of tokens minted for this airdrop
        airDrops[airDropHash].tokens_minted++;
    }
```

Besides taking `latitude`, `longitude`, `distance`, `time_from` and `time_to` values, notice that this function will also take in the `deviceHash` and the `signature` which will be returned by the trusted location API upon querying the W3bstream server. These values are important because they'll be used to generate a `digest` which will be passed, together with the `signature`, onto the Verifier contract, to determine the validity of the proof received, i.e. make sure that this proof has been generated by a valid W3bstream node. Other than this, the rest is very clear. We'll then mint the Airdrop NFT (line 51), emit the `Claimed` event and update the number of tokens minted for this Airdrop.

Add these last three functions to complete the contract: 

```solidity
    function setFeeReceiver(address _receiver) public onlyOwner {
        feeReceiver = _receiver;
    }

    function setFeeCliff(uint256 cliff) public onlyOwner {
        feeCliff = cliff;
    }

    function calculateFee(uint256 _tokens_count) public view returns (uint256) {
        uint256 returnFee;

        if (_tokens_count >= feeCliff) {
            returnFee = feeCliff * baseFee;
        } else {
            returnFee = _tokens_count * baseFee;
        }
        return returnFee;
    }
```

Now that the contract is complete, create a `.env` file with your private key inside the `web3` disrectory, like this: 

```bash
IOTEX_PRIVATE_KEY = <YOUR-PRIVATE-KEY-HERE>
```

Then, from the web3 directory, run this command to deploy the contracts to the **IoTeX Testnet**. 

```bash
cd web3
npm run deploy:testnet
```

This command will deploy to the IoTeX Testnet and export the deployed contracts' addresses and ABIs to the `frontend` directory automatically, for us to use later. 

Make sure you have **IOTX** test tokens before deploying! The easiest way to obtain some is by creating an account on the IoTeX [devloper portal](https://developers.iotex.io/) and navigating to your dashboard. 

## Frontend Overview

Now that the `web3` folder has been taken care of, it's time to work on the `frontend`. In its entirety, the `frontend` directory has numerous components, hooks, utilities and typescript types, but for the purposes of this tutorial we'll be focusing on three components: the `useClaimDrop` hook, the `ClaimButton` and the `ClaimVerifier` components. 

## The useClaimDrop Hook

From the code editor, navigate to the `hooks` directory and open the `useClaimDrop.ts` file. Let's start by importing the tools we'll need: 

```typescript
import {
  useContractWrite,
  usePrepareContractWrite,
  useWaitForTransaction,
} from "wagmi";
import { LocationAirdrop } from "../config/contracts";
import { useRouter } from "next/router";
import { VerifiedLocation } from "../types/VerifiedLocation";
import { useToast } from "@chakra-ui/react";
import { ethers } from "ethers";
```

We're going to be using wagmi to handle contract interactions. We'll need the `LocationAirdrop` contract address and ABI, `useRouter()` from NextJS, a `VerifiedLocation` type from the `types` folder, the `useToast()` hook from *Chakra-UI* and, finally, the *ethers* library. 

The `useClaimDrop` hook is going to use some arguments, update it like this: 

```typescript
export const useClaimDrop = ({
  scaled_latitude,
  scaled_longitude,
  distance,
  from,
  to,
  devicehash,
  signature,
  isReadyToClaim,
}: VerifiedLocation & { isReadyToClaim: boolean }) => {

}
```

Now add the following code inside the hook: 

```typescript
  const router = useRouter();
  const { config, isError: isPrepareError } = usePrepareContractWrite({
    ...LocationAirdrop,
    functionName: "claim",
    args: [
      scaled_latitude,
      scaled_longitude,
      distance,
      from,
      to,
      devicehash,
      signature,
    ],
    enabled: isReadyToClaim,
    overrides: {
      value: ethers.utils.parseEther("2"),
    },
    onError: (error) => console.log("error", error),
  });
  ```
  
Let's focus on the `usePrepareContractWrite()` hook from *wagmi*: `usePrepareContractWrite` gives back a "prepared config" to be sent through to `useContractWrite`. More information can be found on wagmi's page [here](https://wagmi.sh/react/prepare-hooks/usePrepareContractWrite). This hook needs a few params: First it will need the contract address and ABI it needs to call, the name of the function to be called and the arguments that need to be used. Note that we're only enabling this contract to be called only when `isReadyToClaim` (one of the arguments passed earlier in this component) is `true`. Finally, we'll need to send some test IOTX with this transaction to cover the value required by the Verifier contract (lines 15-17). 

Let's finish building this hook by adding the following code: 

```typescript
  const { data, isLoading, isSuccess, isError: isWriteError, write } = useContractWrite({
    ...config,
    onSuccess: () => router.push("/"),
  });
  const toast = useToast();
  const { isLoading: isWaiting, isSuccess: isGood, isError: isWaitError } = useWaitForTransaction({
    confirmations: 1,
    hash: data?.hash,
    onSuccess: () => toast({ title: "successful claim", status: "success" }),
  });

  return {
    data,
    isLoading: isLoading || isWaiting,
    isSuccess: isSuccess && isGood,
    isError: isWaitError || isPrepareError || isWriteError,
    claimAirdrop: write,
  };
```

Now, `useContractWrite` will actually call the contract using the `config` from the `usePrepareContractWrite()` hook we just wrote and will redirect the user home upon success. We're also using the `useWaitForTransaction` hook, which comes with a series of handy properties to make up a better user experience. Upon success, we'll simply display a toast with a **"successful claim"** message and return all the values you see above, most importantly though are `data` and `claimAirdrop` which will be used in the next component. 

## The ClaimButton Component

The `ClaimButton` handles the actual contract call by leveraging the `useClaimDrop` we just created. 

Let's paste this code in the `ClaimButton.tsx` file, under the Components directory:

```typescript
import { useClaimDrop } from "../hooks/useClaimDrop";
import { Button, ButtonProps, Spinner } from "@chakra-ui/react";
import { VerifiedLocation } from "../types/VerifiedLocation";

type ClaimButtonProps = {
    isReadyToClaim: boolean,
    verifiedLocations: VerifiedLocation[],
}

export const ClaimButton = ({isReadyToClaim, verifiedLocations, ...rest} : ClaimButtonProps & ButtonProps) => {
    const { isLoading, isSuccess, isError, claimAirdrop } = useClaimDrop({
        scaled_latitude: isReadyToClaim ? verifiedLocations[0].scaled_latitude : 0,
        scaled_longitude: isReadyToClaim
            ? verifiedLocations[0].scaled_longitude
            : 0,
        distance: isReadyToClaim ? verifiedLocations[0].distance : 0,
        from: isReadyToClaim ? verifiedLocations[0].from : 0,
        to: isReadyToClaim ? verifiedLocations[0].to : 0,
        devicehash: isReadyToClaim ? verifiedLocations[0].devicehash : "",
        signature: isReadyToClaim ? verifiedLocations[0].signature : "",
        isReadyToClaim,
    });

    function handleClaim() {
        claimAirdrop?.()
    }

    if (isError) {
        return (
            <Button
            {...rest}
            isDisabled={true}
            >
                Unable to Claim
            </Button>
        )
    }

    return (
        <Button
            onClick={handleClaim}
            {...rest}
        >
            {isLoading ? <Spinner size="xs" /> : `Claim`}
        </Button>
    )
}
```

If the value of `isReadyToClaim`, which we pass into this component, is set to `true`, then we'll be able to call the `useClaimDrop` hook with the appropriate parameters. The `handleClaim` function triggers the `claimAirdrop` method we established earlier. The rest is about rendering the correct button/message based on the outcome of the contract call. The last question to answer is: "When is this contract call triggered?". The answer depends on the response we get from querying the trusted location API in the next component.

## The ClaimVerifier Component

This component is in charge of orchestrating the API call and, in turn, determining whether or not to call the smart contract. 

Let's open `ClaimVerifier.tsx` in the `Components` directory and start by importing everything we need to make this work: 

```typescript
import { useSignMessage, useAccount } from "wagmi";
import { useRef, useState } from "react";
import { VerifiedLocation } from "../types/VerifiedLocation";
import { Airdrop } from "../types/Airdrop";
import { Button } from "@chakra-ui/react";
import { ConnectButton } from "./User/ConnectButton";
import { ClaimButton } from "./ClaimButton";
import { GeolocationVerifier } from "@w3bstream/geolocation-light";
```

Note that, instead of directly calling the API and manually handling the scaling of coordinates and dates as well as generating the correct SIWE message, we'll be using the GeolocationVerifier package from IoTeX, which will make the whole process faster and easier. For the full API example, feel free to checkout the trusted location documentation. More information about the package, in return, can be found here. 

Replace the template code with this:

```typescript
export const ClaimVerifier = ({ airdrop }: { airdrop: Airdrop }) => {
  const geolocation = useRef<GeolocationVerifier>(new GeolocationVerifier());
  const { address, isConnected } = useAccount();

  const [verifiedLocations, setVerifiedLocations] = useState<
    VerifiedLocation[]
  >([]);
  const [isReadyToClaim, setIsReadyToClaim] = useState(false);
  const [verificationUnsuccessful, setVerificationUnsuccessful] = useState(false);
}
```

This component takes in an `airdrop`. Feel free to dig deeper and learn more about how all the components come into play. For the purposes of this tutorial, it'll suffice to know that this airdrop represents the one the user is about to claim, and it mirrors the struct we looked at in the `LocationAirdrop` contract. 

We're creating a new instance of the `GeolocationVerifier` using the `useRef()` hook. 

Wagmi's `useAccount()` will give us access to the `address` and `isConnected` values, while the `useState()` hook will allow us to render the right message, later on, based on the response we'll get from querying the trusted location API. 

Next, add the following code to the component: 

```typescript
  async function sendQuery() {
    const verifiedLocations = await geolocation.current.verifyLocation();
    if (!!verifiedLocations && verifiedLocations.length > 0) {
      setVerifiedLocations([...verifiedLocations]);
      setIsReadyToClaim(true);
    } else {
      setVerifiedLocations([]);
      setIsReadyToClaim(false);
      setVerificationUnsuccessful(true);
    }
  }

  const { signMessage } = useSignMessage({
    onSuccess: (data) => {
      geolocation.current.signature = data;
      sendQuery()
    }
  });

  function handleUnlock() {
    if (!address) return;
    geolocation.current.scaledLocation = {
      scaled_latitude: Number(airdrop.lat),
      scaled_longitude: Number(airdrop.long),
      distance: Number(airdrop.max_distance),
      from: Number(airdrop.time_from),
      to: Number(airdrop.time_to),
    };

    const message = geolocation.current.generateSiweMessage({
      address,
      domain: globalThis.location.host,
      uri: globalThis.location.origin,
    });
    signMessage({ message });
  }
```

This is the bulk of the logic for this component, and it can be divided into three steps: 

The `handleUnlock()` function will use the `scaledLocation` method to create a location from the value we got from the airdrop prop and will generate a SIWE message, which will also include the user's `address`, the `domain` and `uri` of the application, amongst other values (which are taken care of by the geolocation package under the hood). For a more in depth view check out the trusted location docs, [here](https://docs.iotex.io/).

Once the message has been successfully signed and the signature retrieved (line 15), the program will call the `sendQuery()` function, which will call the API using the `verifyLocation()` method from the package, and will set the state of `verifiedLocations`, `isReadyToClaim`, and `verificationUnsuccessful` accordingly. 

Add this last bit of code: 

```typescript
  if (!isConnected) {
    return <ConnectButton />;
  }

  return isReadyToClaim ? (
    <ClaimButton
      isReadyToClaim={isReadyToClaim}
      verifiedLocations={verifiedLocations}
      mt={12}
      size={'xs'}
    />
  ) : (
    <Button size={'xs'} mt={12} onClick={handleUnlock}>
      {verificationUnsuccessful ? "Verification Unsuccessful" : "Unlock"}
    </Button>
  );
```

If the user has not yet connected, we'll render the `ConnectButton` component. 

We'll now render the "Unlock" button (like 14) which will call the trusted location API and determine if the user is ready to claim an airdrop. If so, we'll then render the `ClaimButton` with the corresponding appropriate props. 

Check out `localhost:3000` in your browser and start using the Dapp!

## Conclusion

Congratulations on building a dApp on trusted real-world data! The fully-working repository for this application can be found [here](https://github.com/machinefi/trusted-location-airdrop-dapp/tree/main). 

Trusted location is only one of the multitude of implementations for trusted data in smart contracts, and within this vertical there are certainly so many use cases: Proof of presence, scavanger hunts, and GameFi, just to name a few.  

If you'd like to know more about IoTeX's geo location API, you can check out the official W3bstream documentation [here](https://docs.w3bstream.com/introduction/readme). More info on the geo location package can be found [here](https://github.com/machinefi/geolocation-sdk-demo/tree/main/packages/geolocation). 
