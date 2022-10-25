*Head of DevRel at IoTeX, Simone Romano, stars in the second episode of this series of lectures about the MachineFi methodology developed by IoTeX, in front of Prof. Zhenhua Liu's students of the Stony Brook University of New York.*

## Quick Overview

The second lecture of this series will mainly focus on the core component of a MachineFi dApp: **W3bstream**, the off-chain computing infrastructure, developed by IoTeX, serving as an open, chain-agnostic and decentralized protocol sitting in between blockchain and devices to convert real-world data streams from devices into verifiable, dApp-ready proofs.

![w3bstream-flow](https://user-images.githubusercontent.com/77351244/195899352-f40cfeca-dfcc-46ff-892d-16b5d4dfa90f.png)

Whilst the first lecture focused on the firmware aspect of this application, in this lecture students are guided through the configuration of a simple W3bstream node (though the configuration shown will be greatly updated and will differ significantly compared to the **W3bstream 1.0 Alpha release** expected by the end of October 2022). 

W3bstream will be able to monitor the specified contracts, and handle the corresponding events appropriately, by feeding the walk-to-earn contract the correct proofs. 

The architecture of this simple flow is illustrated in the image below: 

![walk-to-earn-flow](https://user-images.githubusercontent.com/77351244/197878601-af10ee4e-f3b7-478f-b91e-90f80d1bea25.png)

## What's Next?

The next part of the lecture will focus on the smart contracts that will be used to:

- Register a device 
- Bind a device to a blockchain account
- Create the token that will be used to reward users
- Handle the bulk of the logic of the application: Sending requests to W3bstream, processing the proofs and allowing users to claim their rewards.

Feel free to have a look at the full video to gain a more in-depth understanding of the overall flow of this application and learn how to build your next MachineFi dApp with IoTeX.  
