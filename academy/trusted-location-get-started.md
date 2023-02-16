*The **trusted location API** developed by **IoTeX** allows developers to build a whole new set of dApps: Upon a user's approval, an application can query the API for a proof of the user's presence in a certain area within a specified time range. This get-started will show how to interact with the trusted location API through the iotex geolocation package. More detailed information can be found in the trusted location docs, [here]()*. 
 
 ## Send a trusted location through ioPay
 
 The first step is to send a trusted location to a [w3bstream](https://w3bstream.com/) server through IoTeX's native wallet, [ioPay](https://iopay.me/). Follow the steps in this tutorial to [enable trusted GPS location data in ioPay](https://developers.iotex.io/posts/enable-trusted-geolocation-tutorial). 
 
 ## Create a trusted location dApp
 
 Create a NextJS project in typescript called *trusted-location*: 
 
 ```bash
 npx create-next-app@latest --typescript
 ```
 
 Let's enter the project and install the tools we'll need to proceed: the *wagmi-ethers* package and the *iotex-geolocation* package. 
 
 ```bash
 cd trusted-location && npm i wagmi ethers@^5 @w3bstream/geolocation-light
 ```
 
 Open the project in your favorite code editor. 
 
 The first step is to replace the default content of the `index.tsx` file in the `pages` directory with the following: 
 
 ```typescript
 import Head from 'next/head'
import dynamic from "next/dynamic";

const GeoButton = dynamic(() => import("../components/GeoButton")
.then(module => module.GeoButton), {
  ssr: false,
});

export default function Home() {
  return (
    <>
      <Head>
        <title>Trusted Location </title>
      </Head>
      <main>
        <GeoButton />
      </main>
    </>
  )
}
```

The only component to render is the `GeoButton` which will be in charge of creating the message to be signed by the user and querying the trusted location API. In this case, the `GeoButton` is imported dynamically by NextJS in order to avoid discrepancies between the server-side and client-side rendering when connecting to Metamask. More information on dynamic imports can be found [here](https://nextjs.org/docs/advanced-features/dynamic-import).

Next, we'll replace the default content of the `_app.tsx` file in the `pages` directory with the following: 

```typescript
import { WagmiConfig, createClient } from 'wagmi'
import { getDefaultProvider } from 'ethers'

const client = createClient({
    provider: getDefaultProvider('testnet'),
    autoConnect: true
})

export default function App({ Component, pageProps } : any) {
    return (
        <WagmiConfig client={client}>
            <Component {...pageProps} />
        </WagmiConfig>
    )
}
```

Here, we're creating a wagmi `client` and passing it to the `WagmiConfig` React Context. The client is set up to use the ethers Default Provider and automatically connect to previously connected wallets. We'll then wrap our application with the `WagmiConfig` component. More information can be found in the [wagmi docs](https://wagmi.sh/).

Next, we'll create a `components` directory where we'll have two components: the `ConnectButton` and the `GeoButton` mentioned above. 

```bash
mkdir components && cd components
```

Create a file called `ConnectButton.tsx` and paste the following code: 

```typescript
import { useConnect } from "wagmi";

export const ConnectButton = () => {
  const { connect, connectors } = useConnect();

  return (
    <div>
      {connectors.map((connector) => (
        <button key={connector.id} onClick={() => connect({ connector })}>
          Connect
        </button>
      ))}
    </div>
  );
};
```

This component uses the `useConnect()` hook by *wagmi* to allow a user to connect to the application with their browser wallet. More info on the use of this hook can be found [here](https://wagmi.sh/react/hooks/useConnect). 

Next, in this same directory, create a new file called `GeoButton.tsx`. Let's start by importing the tools needed to build this component. Paste this code: 

```typescript
import { useSignMessage, useAccount } from "wagmi";
import { useRef, useState } from "react";
import { ConnectButton } from "./ConnectButton";
import { GeolocationVerifier } from "@w3bstream/geolocation-light";
```

Let's now create a `locationObject` represeting the location we'll be querying the API with. If you have already successfully sent your location via *ioPay* earlier, you should use those values: 

```typescript
const locationObject = {
    latitude: 40.572349, 
    longitude: -60.536486, 
    distance: 100, 
    from: new Date("2023-01-03").getTime() / 1000, 
    to: new Date("2023-01-28").getTime() / 1000
}
```

A location object is comprised of the keys you see above: `latitude`, `longitude`, `distance`, `from` and `to`. The `from` and `to` timestamps are expressed in seconds, while the other values are integers. 

Next, add the following code: 

```typescript
export const GeoButton = () => {

    const geolocation = useRef<GeolocationVerifier>(new GeolocationVerifier());

    const { address, isConnected } = useAccount();

    const [verificationSuccessful, setVerificationSuccessful] = useState(false);
    
}
```

We're creating a new instance of the `GeolocationVerifier` using the `useRef()` hook.

Wagmi's `useAccount()` will give us access to the `address` and `isConnected` values, while the `useState()` hook will allow us to render the right message, later on, based on the response we'll get from querying the trusted location API. 

Now add the following code to the component: 

```typescript
    const { signMessage } = useSignMessage({
        onSuccess: async (data) => {
            geolocation.current.signature = data;
            sendQuery()
        },
    });
    
    async function sendQuery() {
        const verifiedLocations = await geolocation.current.verifyLocation();
        if (!!verifiedLocations && verifiedLocations.length > 0) {
            setVerificationSuccessful(true);
        } else {
            setVerificationSuccessful(false);
        }
    }

    function handleQuery() {
        if (!address) return;

        geolocation.current.location = (locationObject)

        const message = geolocation.current.generateSiweMessage({
            address,
            domain: globalThis.location.host,
            uri: globalThis.location.origin,
        });
        signMessage({ message });
    }
```

The `handleQuery()` function will use the `locationObject` created above to generate a *SIWE* message, which will also include the user's `address`, the `domain` and `uri` of the application, amongst other values (which are taken care of by the geolocation package under the hood). For a more in depth view check out the trusted location docs, [here](https://docs.iotex.io/). 

Once the message has been successfully signed and the signature retrieved (line 3), the program will call the `sendQuery()` function, which will call the API using the `verifyLocation()` method from the package. 

Based on the API's response, the value of `verificationSuccesful` will be set to `true` or `false`.

Now add the following code: 

```typescript
    if (!isConnected) {
        return <ConnectButton />;
    }

    return (
        <div>
            <p>
                {address}
            </p>
            <button
                onClick={handleQuery}>
                Send Query
            </button>
            <p>
                {
                    verificationSuccessful ?
                        `Valid Proof` : `No Proof Returned`
                }
            </p>

        </div>
    )    
```

If the user has not yet connected, we'll display the `ConnectButton`, otherwise we'll show the user's address, the button to handle the query, and a message based on the query's response.


## Conclusion

Congratulations, you've created a simple dApp based on real world data. 

Go ahead and test your app by running `npm run dev` from your project's root and opening your browser at http://localhost:3000/ 

Trusted location has many benefits, and a multitude of use cases, some of which are described in the docs, [here](https://docs.iotex.io/). 

The trusted location package is very useful and will quickly get you up to speed. More info on it can be found [here](https://github.com/nicky-ru/g3o). However, if you'd like to dig deeper, the documentation is a great place to learn about what's happening under the hood. 
