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


Now let's go back to the code, set an owner variable and set it in the constructor.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol';

contract OceanToken is ERC20 {

    address payable public owner;
    constructor(uint256 initialSupply) ERC20("OceanToken", "OCT") {
        owner = payable(msg.sender);
        _mint(owner, initialSupply);
    }
}
```

Now let's work on the initial supply..

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol';

contract OceanToken is ERC20 {

    address payable public owner;
    constructor() ERC20("OceanToken", "OCT") {
        owner = payable(msg.sender);
        _mint(owner, 70000 ** decimals());
    }
}
```

Now let's implement the max supply.

To do that we import the ERC20 capped contract from openzeppelin.

Because ERC20Capped, inherits from ERC20, we can replace it.

We will also pass in a new constructor parameter.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol';
import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Capped.sol';


contract OceanToken is ERC20Capped {

    address payable public owner;
    constructor(uint256 cap) ERC20("OceanToken", "OCT") ERC20Capped(cap * (10 ** decimals())) {
        owner = payable(msg.sender);
        _mint(owner, 70000 ** decimals());
    }
}
```

Now let's make the token burnable.

We will import ERC20Burnable from openzeppelin.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol';
import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Capped.sol';
import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Burnable.sol';

contract OceanToken is ERC20Capped, ERC20Burnable {

    address payable public owner;
    constructor(uint256 cap) ERC20("OceanToken", "OCT") ERC20Capped(cap * (10 ** decimals())) {
        owner = payable(msg.sender);
        _mint(owner, 70000 ** decimals());
    }
}
```


Now let's create block reward to distribute new supply to miners..


We're going to have a new variable called blockReward(then set it to the constructor) and a hook called beforeTokenTransfer.

We can extend any functionality to whatever you want to happen before any transfer.

 We will create variable mintMinerReward - mint new tokens depending on blockReward.
 
 We will create a function called setBlockReward aand set an onlyOwner modifier, so only the owner can set block reward.
 
 Then we will create a mintMinerReward function
 
 
 ```
 // SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol';
import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Capped.sol';
import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Burnable.sol';

contract OceanToken is ERC20Capped, ERC20Burnable {

    address payable public owner;
    uint256 public blockReward;


    constructor(uint256 cap, uint256 reward) ERC20("OceanToken", "OCT") ERC20Capped(cap * (10 ** decimals())) {
        owner = payable(msg.sender);
        _mint(owner, 70000 ** decimals());
        blockReward = reward ** decimals();
    }

    modifier onlyOwner {
        require(msg.sender == owner, "only the owner can call this function");
        _; 
    }

    function setBlockReward(uint256 reward) public onlyOwner {
        blockReward = reward * (10 ** decimals());
    }

    function _mintMinerReward() internal {
        _mint(block.coinbase, blockReward);
    }

    function _beforeTokenTransfer(address from, address to, uint256 value) internal virtual override {
        if(from != address(0) && to != block.coinbase && block.coinbase != address(0)){
            _mintMinerReward();
        }
        super._beforeTokenTransfer(from, to, value);
    }

    function destroy() public onlyOwner {
        selfdestruct(owner);
    }


}
```

That concludes our token design.

Now we can compile our code and test.

Compile with ```npx hardhat compile```

By now you would have discovered that the ```_mint``` function is being called from ERC20Capped and ERC20Burnable.]]

Now go to ERC20Capped in openzeppelin on Hardhat, copy the mint function,and paste under the constructor of our contract.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol';
import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Capped.sol';
import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Burnable.sol';

contract OceanToken is ERC20Capped, ERC20Burnable {

    address payable public owner;
    uint256 public blockReward;


    constructor(uint256 cap, uint256 reward) ERC20("OceanToken", "OCT") ERC20Capped(cap * (10 ** decimals())) {
        owner = payable(msg.sender);
        _mint(owner, 70000 ** decimals());
        blockReward = reward ** decimals();
    }

    function _mint(address account, uint256 amount) internal virtual override(ERC20Capped, ERC20) {
        require(ERC20.totalSupply() + amount <= cap(), "ERC20Capped: cap exceeded");
        super._mint(account, amount);
    }

    modifier onlyOwner {
        require(msg.sender == owner, "only the owner can call this function");
        _; 
    }

    function setBlockReward(uint256 reward) public onlyOwner {
        blockReward = reward * (10 ** decimals());
    }

    function _mintMinerReward() internal {
        _mint(block.coinbase, blockReward);
    }

    function _beforeTokenTransfer(address from, address to, uint256 value) internal virtual override {
        if(from != address(0) && to != block.coinbase && block.coinbase != address(0)){
            _mintMinerReward();
        }
        super._beforeTokenTransfer(from, to, value);
    }

    function destroy() public onlyOwner {
        selfdestruct(owner);
    }


}
```
 
 
 Now the next step is to write unit tests.
 
 Now go to the test folder on Hardhat and create a new OceanToken.js file..
 
 
 
 ```
 //We will import chai and hardhat hre(it contains a version of ethers.js to interact with 
//our smart contract) library


const { expect } = require("chai");
const hre = require("hardhat");

//We will use a describe block to describe our test
describe("OceanToken contract", function() {
  // designate global varaiables we will use throughout the test
  let Token;
  let oceanToken;
  let owner;
  let addr1;
  let addr2;
  let tokenCap = 100000000;
  let tokenBlockReward = 50;

  //To instantiate a contract to create an instance of it before each test runs.
  beforeEach(async function () {
    // Get the ContractFactory and Signers here.
    Token = await ethers.getContractFactory("OceanToken");
    [owner, addr1, addr2] = await hre.ethers.getSigners(); //setting up wallets for test

    oceanToken = await Token.deploy(tokenCap, tokenBlockReward);  //deploy contract
  });


  // The first block of test
  describe("Deployment", function () {
    it("Should set the right owner", async function () {
      expect(await oceanToken.owner()).to.equal(owner.address);
    });

    it("Should assign the total supply of tokens to the owner", async function () {
      const ownerBalance = await oceanToken.balanceOf(owner.address);
      expect(await oceanToken.totalSupply()).to.equal(ownerBalance);
    });

    it("Should set the max capped supply to the argument provided during deployment", async function () {
      const cap = await oceanToken.cap();
      expect(Number(hre.ethers.utils.formatEther(cap))).to.equal(tokenCap);
    });

    it("Should set the blockReward to the argument provided during deployment", async function () {
      const blockReward = await oceanToken.blockReward();
      expect(Number(hre.ethers.utils.formatEther(blockReward))).to.equal(tokenBlockReward);
    });
  });

  describe("Transactions", function () {
    it("Should transfer tokens between accounts", async function () {
      // Transfer 50 tokens from owner to addr1
      await oceanToken.transfer(addr1.address, 50);
      const addr1Balance = await oceanToken.balanceOf(addr1.address);
      expect(addr1Balance).to.equal(50);

      // Transfer 50 tokens from addr1 to addr2
      // We use .connect(signer) to send a transaction from another account
      await oceanToken.connect(addr1).transfer(addr2.address, 50);
      const addr2Balance = await oceanToken.balanceOf(addr2.address);
      expect(addr2Balance).to.equal(50);
    });

    it("Should fail if sender doesn't have enough tokens", async function () {
      const initialOwnerBalance = await oceanToken.balanceOf(owner.address);
      // Try to send 1 token from addr1 (0 tokens) to owner (1000000 tokens).
      // `require` will evaluate false and revert the transaction.
      await expect(
        oceanToken.connect(addr1).transfer(owner.address, 1)
      ).to.be.revertedWith("ERC20: transfer amount exceeds balance");

      // Owner balance shouldn't have changed.
      expect(await oceanToken.balanceOf(owner.address)).to.equal(
        initialOwnerBalance
      );
    });

    it("Should update balances after transfers", async function () {
      const initialOwnerBalance = await oceanToken.balanceOf(owner.address);

      // Transfer 100 tokens from owner to addr1.
      await oceanToken.transfer(addr1.address, 100);

      // Transfer another 50 tokens from owner to addr2.
      await oceanToken.transfer(addr2.address, 50);

      // Check balances.
      const finalOwnerBalance = await oceanToken.balanceOf(owner.address);
      expect(finalOwnerBalance).to.equal(initialOwnerBalance.sub(150));

      const addr1Balance = await oceanToken.balanceOf(addr1.address);
      expect(addr1Balance).to.equal(100);

      const addr2Balance = await oceanToken.balanceOf(addr2.address);
      expect(addr2Balance).to.equal(50);
    });
  });
  
});
```

Now let's deploy our smart contract to the blockchain..

First let's configure our project for deployment with a ```.env``` file.

This file will hold 2 variables we will use for deployment.

```
PRIVATE_KEY=
INFURA_RINKEBY_ENDPOINT=
```

Now install the dot.env package with ```npm i dotenv```

Also go to hardhat.config.js and add a require statement for dotenv:

```
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: "0.8.17",
};
```

Now under module.exports, we need to create a new module called networks - which is going to contain our network configuration.

```
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: "0.8.17",
  networks: {
    rinkeby: {
      url: process.env.INFURA_RINKEBY_ENDPOINT,
      accounts: [process.env.PRIVATE_KEY]![Screenshot 2022-11-19 at 15 04 30](https://user-images.githubusercontent.com/29931071/202858313-24ad78b1-4fc0-494d-bd18-9fdf1066c319.png)

    }
  }
};
```

Now let's look at how to get the private key and endpoint for the env file.

The first one is the private key of our metamask wallet

![Screenshot 2022-11-19 at 15 04 30](https://user-images.githubusercontent.com/29931071/202858315-12d29e05-6b49-4ca5-b2ae-020002cb3554.png)

![Screenshot 2022-11-19 at 15 04 30](https://user-images.githubusercontent.com/29931071/202858314-bbdb2dee-0c4c-40ee-848f-b02c149deb18.png)

![Screenshot 2022-11-19 at 15 04 51 copy](https://user-images.githubusercontent.com/29931071/202858317-7e5a2c90-e33f-4c29-a595-2e6ea65dc5f7.png)



Next is the Infura Rinkeby endpoint.

First to go the rinkeby faucet to get test ether - https://faucet.rinkeby.io/

Go create an Infura account and hit "CREATE NEW KEY"

![Screenshot 2022-11-19 at 15 14 35](https://user-images.githubusercontent.com/29931071/202858577-169dda55-12ab-40f9-858b-bd882a6c3b15.png)


Copy the API key..


# Deploy Script


```
const hre = require("hardhat");

async function main() {
  const OceanToken = await hre.ethers.getContractFactory("OceanToken");
  const oceanToken = await OceanToken.deploy(100000000, 50);

  await oceanToken.deployed();

  console.log("Ocean Token deployed: ", oceanToken.address);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

Now save this file and attempt to deploy to Goerli.

Now we've decided to deploy to Goerli since Rinkeby has been deprecated. So here's our config.js and env

config.js:

```
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: "0.8.17",
  networks: {
    goerli: {
      url: process.env.INFURA_GOERLI_ENDPOINT,
      accounts: [process.env.PRIVATE_KEY]
    }
  }
};
```

env:
```
PRIVATE_KEY=
INFURA_GOERLI_ENDPOINT=
```


Now deploy with ```npx hardhat run --network goerli scripts/deploy.js```

