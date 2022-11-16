# ERC20_Token

![Screenshot_211](https://user-images.githubusercontent.com/29931071/202281040-5a758ab6-42c9-4fd8-aed5-a4eeb94baa87.png)

![Screenshot_212](https://user-images.githubusercontent.com/29931071/202281069-d8a24a0a-9bd5-40a8-abbb-94fc71a73737.png)


We'll start with spdx license identifier.

Then we'll import openzeppelin ERC20 format from github.. - https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol


```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol';
```

Then we will define our contract, and inherit from ERC20.
We will define a constructor to initialize the initial Supply, the token name and symbol.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol';

contract OceanToken is ERC20 {
    constructor(uint256 initialSupply) ERC20("OceanToken", "OCT") {
        _mint(msg.sender, initialSupply);
    }
}
```


We can now deploy the token smart contract and play with it.

![Screenshot_213](https://user-images.githubusercontent.com/29931071/202281152-b8163511-8484-407a-bcec-12851f6dde22.png)





# Hardhat

Open a new folder.

Open the folder in VScode.

Open the terminal

create a project folder with ```mkdir oceantoken```

cd into the directory

Now do ```npm init```
This will install a package.json file

Now install hardhat with ```npm i --save-dev hardhat```

Now invoke hardhat with ```npx hardhat``` to create the hardhat project scaffold.

Now create a Javascript project

Now install the openzeppelin contract with ```npm i @openzeppelin/contracts```



Now go to the contract folder, open a new .sol file named "OceanToken.sol", copy the solidity code above and paste it.  

We no longer need the remote link pointing to GitHub. We can now reference the locally installed files.


```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

import '@openzeppelin/contracts/token/ERC20/ERC20.sol';

contract OceanToken is ERC20 {
    constructor(uint256 initialSupply) ERC20("OceanToken", "OCT") {
        _mint(msg.sender, initialSupply);
    }
}
```


First let's look at token design

![Screenshot_214](https://user-images.githubusercontent.com/29931071/202281289-f2e2efd7-7c3d-43b0-8396-42e45affffe6.png)


Token design

1. Initial supply - sent to owner = 70% * Max supply = 70,000.

2. Max supply - capped = 100,000

Let's say we want to use the 70k tokens to create a liquidity pool on Uniswap and 30k as reward system.
