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
