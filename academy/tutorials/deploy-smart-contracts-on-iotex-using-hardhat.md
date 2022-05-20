---
title: Deploy smart contracts on IoTeX using Hardhat
description: Learn how to deploy smart contracts on IoTeX using Hardhat
permalink: academy/tutorials/deploy-smart-contracts-on-iotex-using-hardhat.md
---

#### Install node.js

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.39.1/install.sh | bash
nvm install 16
nvm use 16
nvm alias default 16
npm install npm --global # Upgrade npm to the latest version
```

#### Creating a new Hardhat project

```
mkdir hardhat-tutorial
cd hardhat-tutorial
npm init --yes
npm install --save-dev hardhat
```

#### Create a new hardhat project

```
npx hardhat
```

#### Install plugins

```
npm install --save-dev @nomiclabs/hardhat-ethers ethers @nomiclabs/hardhat-waffle ethereum-waffle chai
```

#### Edit hardhat.config.js

```
code .
```

```
require("@nomiclabs/hardhat-waffle");

/**
 * @type import('hardhat/config').HardhatUserConfig
 */

module.exports ={solidity:"0.8.4",};
```

#### Create the rewards token "Carbon"

```
mkdir contracts
touch contracts/carbon.sol
```

```
// SPDX-License-Identifier: MIT
// carbon.sol

pragma solidity ^0.8.4;
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
contract Carbon is ERC20 {
    constructor() ERC20("Carbon", "CARB") {
        _mint(msg.sender, 1000000 * 10 ** decimals());
    }
}
```

#### Install openzeppelin contracts

```
npm install @openzeppelin/contracts
```

#### Create the temperature rewarding contract

```
touch contracts/temperature.sol
```

```
// SPDX-License-Identifier: MIT
// temperature.sol

pragma solidity ^0.8.4;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "hardhat/console.sol";

// Receives device data, collects rewards and allow to claim them
contract Temperature is Ownable {
    using SafeERC20 for IERC20;

    event Data(string indexed deviceId, int temperature);

    event RewardsClaimed(address indexed deviceOwner, uint256 amount);

    // The token used as a reward (Carbon)
    address public rewardTokenAddress;
    // How much we give per data point
    uint256 public rewardPerDataPoint;
    // Minimum temperature to get a reward
    int public minTemperature;
    // Maximum temperature to get a reward
    int public maxTemperature;

    mapping(address => uint256) public Rewards;

    // Expect temperature values to be in Celsius, over -30 and under 45
    // Temperature values are to be divided by 100 (i.e. 24,23 deg Celsius is 2423)
    modifier onlyValidTemperature(int temperature) {
        require(temperature > -30 * 100, "Temperature out of range");
        require(temperature > -30 * 100, "Temperature out of range");
        _;
    }

    constructor(
        uint256 _tokenRewardsPerDataPoint,
        address _rewardTokenAddress,
        int _minTemperature,
        int _maxTemperature       ) {
        console.log("Deploying Temperature contract");
        rewardPerDataPoint = _tokenRewardsPerDataPoint;
        rewardTokenAddress = _rewardTokenAddress;
        minTemperature = _minTemperature;
        maxTemperature = _maxTemperature;
    }

    function submitData(string memory imei, address deviceOwner, int temperature)
        public
        onlyOwner
        onlyValidTemperature(temperature)
    {
        if (temperature >= minTemperature && temperature <= maxTemperature) {
            emit Data(imei, temperature);
            _accumulateRewards(deviceOwner);
        }
    }

    function claimRewards() public {
        address deviceOwner = msg.sender;
        uint256 tokensToSend = Rewards[deviceOwner];
        delete Rewards[deviceOwner];
        IERC20(rewardTokenAddress).safeTransfer(deviceOwner, tokensToSend);
        emit RewardsClaimed(deviceOwner, tokensToSend);
    }

    function _accumulateRewards(address deviceOwner)
        internal
    {
        Rewards[deviceOwner] = Rewards[deviceOwner] + rewardPerDataPoint;
    }
}
```

#### Edit the deploy script

```
mkdir scripts
touch scripts/deploy.js
```

```
async function main() {
    const [deployer] = await ethers.getSigners();

    console.log("Deploying contracts with the account:", deployer.address);

    console.log("Account balance:", (await deployer.getBalance()).toString());

    const Token = await ethers.getContractFactory("Carbon");
    const token = await Token.deploy();

    console.log("Token address:", token.address);

    const Temperature = await ethers.getContractFactory("Temperature");
    const temperature = await Temperature.deploy(
                            10 * 10000000000, token.address, -30, 18);

    console.log("Temperature monitor address:", temperature.address);
  }

  main()
    .then(() => process.exit(0))
    .catch((error) => {
      console.error(error);
      process.exit(1);
    });
```

#### Edit hardhat.config.js to deploy to IoTeX Testnet

```
require("@nomiclabs/hardhat-waffle");

const IOTEX_PRIVATE_KEY = "c5bebf4685acfa8998986214497dcc4006a9405ccdeccdd9840b6784236b8e30";

module.exports = {
  solidity: "0.8.4",
  networks: {
    testnet: {
      url: `https://babel-api.testnet.iotex.io`,
      accounts: [`${IOTEX_PRIVATE_KEY}`]
    }
  }
};
```

#### Deploy

```
npx hardhat run scripts/deploy.js --network testnet
```

#### Redeploy and test with remix ide
