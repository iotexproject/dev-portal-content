*The new version of our IoTeX Analytics is here. Let's find out what Analytics does and what these major improvements mean for our developers.*

*IoTeX Analytics* is an application built upon IoTeX core API which extracts data from IoTeX blockchain and reindexes them for applications to use via a GraphQL web interface. It is used by our internal products as well as many other projects in the IoTeX ecosystem to get a lot of blockchain data quickly and easily.

The new version of *IoTeX Analytics* comes with **increased scalability** and increased support for **larger amounts of transactions and data**. There has been a major architectural upgrade aimed at separating the indexer and API services, which allows for a better handling of erroneous data. *Analytics V2* also supports **multiple API methods** (e.g. REST and GraphQL), a **much higher query speed**, and many new APIs that have been added on top of the ones from V1.

<img width="1131" alt="image" src="https://user-images.githubusercontent.com/11096047/182444522-f434d2db-e54e-4156-ba9a-33de7b1b6003.png"/>


The chart above shows how, for example, many of the requests related to XRC20 or XRC721 tokens, which would *timeout* on V1, are actually handled in V2 in much less than a second. (The horizontal axis on the graph shows requests per second).  

The chart below shows how a lot of the APIs related to the Hermes service have similarly been improved.

<img width="1228" alt="image" src="https://user-images.githubusercontent.com/11096047/182444674-07a57a6c-63ac-4ff9-a049-5f0621737fd4.png"/>

These are just a few examples of how the new version stands out. These improvements have been achieved thanks to new data and index optimization models, as well as SQL query optimization, while some APIs use parallel computing. 

The new Analytics endpoint is: https://analyser-api.iotex.io/graphql - For those devs who'd like to experiment with the API in reference to the Full API documentation, the **GraphQL Playground** can be found at this same address.

The API documentation comes with ready-to-use examples that can be used to query any type of structured data. The advantage is to be able to query data that cannot be extracted directly from the blockchain without indexing it, guaranteeing much simpler use and much faster results. 

Devs can use queries to, for example, build exchanges, dashboards, or display any type of data from the *IoTeX blockchain*. Examples of such queries might include: Token holders for a specific NFT or XRC20 token contract; A list of all the transactions for a given wallet address; Or even a wallet's historical balance for a specific token.

The full API documentation comes with different modules, that already cover most of the queries that a developer would want to perform. Such models are: **Chain**, **Delegate**, **Account**, **Voting**, **Action**, **XRC20**, **XRC721**, and **Hermes**, which gives access to the rewards distributed by the IoTeX official [Hermes](https://hermes.to) System. 

Let's look at some examples:

**List all actions for a specific address**

If you are building an IoTeX Explorer, this is something you really need:

```javascript
query {
  ActionByAddress(address: "io1zqnd7sdppw6s2l20pqnpmyrcj0edtautu9wxss") {
    actions {
      actHash, sender, recipient, amount
    }
  }
}
```

**Get delegate production**

If you want to know the efficiency in block production for a specific delegate in a specific range of epochs:

```js
query {
  Productivity(delegateName: "binancevote", startEpoch:20000, epochCount: 120) {
    productivity {
      production, expectedProduction
    }
  }
}
```

**Historical wallet balance**

You can also get the balance of a wallet at a specific height of the blockchain:

```js
query {
  IotexBalanceByHeight(
    address: "0x6b132450C6988246cf60501f37CdF7eEd5d19176", height:  18777330) {
      balance
  }
}
```

For those developers who are curious to see the new Analytics in action and try it, the full documentation can be found on our [IoTeX Docs](https://docs.iotex.io/reference/analytics). 

The chart below shows the full list of stress tests of Analytics V2 vs V1: 

<img src="https://user-images.githubusercontent.com/11096047/182610944-4196c374-9a7b-4d36-815c-cfc515550a52.png"/>



You can always reach out to our dedicated developers support team on our [Discord](https://discord.gg/3WVZ4Vbs) and keep up-to-date with all the latest news on our [iotex_dev](https://twitter.com/iotex_dev) Twitter account. 

Take full advantage of the IoTeX ecosystem and submit your dream project for our [Halo Grants](https://community.iotex.io/c/halo-grants/61). We can't wait to see where you go from here! 


