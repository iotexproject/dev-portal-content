---
title: Query Pebble Tracker Oracle using nodejs
description: Learn how to query the Pebble Tracker Open Oracle using node.js
path: academy/Query-Pebble-Tracker-Oracle-using-nodejs.md
---

## Init the application

```
npm init -y
```

## Install required modules

```
npm install -s apollo-client apollo-link-http apollo-cache-inmemory cross-fetch graphql graphql-tag
```

## Create the script

```
code query.js
```

```
const ApolloClient = require('apollo-client').ApolloClient;
const createHttpLink = require("apollo-link-http").createHttpLink;
const InMemoryCache = require("apollo-cache-inmemory").InMemoryCache;
const crossFetch = require('cross-fetch');
const gql = require('graphql-tag');


const ENDPOINT = "https://pebble.iotex.me/v1/graphql";
const imeiNumber = "351358810263431";

// Create the GraphQL client
const client = new ApolloClient({
	link: createHttpLink({
		uri: ENDPOINT,
		fetch: crossFetch
	}),
	cache: new InMemoryCache()
});

main();

async function main() {
    // Query the most recent latitudes longitudes temperature and timestamps
    const LIST_LOCATIONS_QUERY =
        gql
        `{
            pebble_device_record(
            limit: 1,
            order_by: {timestamp: desc},
            where: {
                imei: {_eq: "${imeiNumber}"},
                latitude: {_neq: "200.0000000"}})
                {
                    latitude, longitude, temperature, timestamp
                }
        }`

    var queryResult;
    console.log("Querying results...");
    try {
        queryResult = (await client.query({
            query: LIST_LOCATIONS_QUERY
        })).data.pebble_device_record;

    } catch (er) { console.log(er);	}

    // Return data to the frontend
    console.log(queryResult);
}
```
