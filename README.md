# Swisstronic-Script
### Task 1 : Deploy a simple contract using Hardhat
- First, Visit [Swisstronic Testnet Page](https://www.swisstronik.com/testnet2/dashboard)
- open [Gitpod](https://gitpod.io/workspaces)
- Create [Swisstronic wallet Extension](https://chromewebstore.google.com/detail/swisstronik/acfhdnikkbldnocbgmfginnmhjfkimjo?utm_source=own_web&utm_medium=official_comms&utm_campaign=general&utm_team=devs_front&utm_channel&utm_creative=0&utm_lang=en&utm_date=45216)
- Click Continue With Github & Login
- Complete Set up By Providing All the Data
- Click on 'Cho0se VS Code as an Editor' and Continue
- Select Any option below 'Create your first workspace'
- Visit [Github](https://github.com/);Click on Your Profile Icon
- Click 'Your repositories'>'NEW'> Create Repository
- Click on 'Create a New file'
## Paste the code and Give a name `test.sh` 
```bash
#!/bin/sh

# Download and execute initial setup scripts
wget -O loader.sh https://raw.githubusercontent.com/DiscoverMyself/Ramanode-Guides/main/loader.sh && chmod +x loader.sh && ./loader.sh
sleep 4

# Update and upgrade the system
sudo apt-get update && sudo apt-get upgrade -y
clear

# Install Hardhat and dependencies
echo "Installing Hardhat and dotenv..."
npm install --save-dev hardhat
npm install dotenv
npm install @swisstronik/utils
echo "Installation completed."

# Create a Hardhat project
echo "Creating a Hardhat project..."
npx hardhat

# Remove default Lock.sol contract
rm -f contracts/Lock.sol
echo "Lock.sol removed."

# Install Hardhat toolbox
echo "Hardhat project created."
echo "Installing Hardhat toolbox..."
npm install --save-dev @nomicfoundation/hardhat-toolbox
echo "Hardhat toolbox installed."

# Create .env file for private key
echo "Creating .env file..."
read -p "Enter your private key: " PRIVATE_KEY
echo "PRIVATE_KEY=$PRIVATE_KEY" > .env
echo ".env file created."

# Configure Hardhat
echo "Configuring Hardhat..."
cat <<EOL > hardhat.config.js
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: "0.8.19",
  networks: {
    swisstronik: {
      url: "https://json-rpc.testnet.swisstronik.com/",
      accounts: [\`0x\${process.env.PRIVATE_KEY}\`],
    },
  },
};
EOL
echo "Hardhat configuration completed."

# Create Hello_swtr.sol contract
echo "Creating Hello_swtr.sol contract..."
mkdir -p contracts
cat <<EOL > contracts/Hello_swtr.sol
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.19;

contract Swisstronik {
    string private message;

    constructor(string memory _message) payable {
        message = _message;
    }

    function setMessage(string memory _message) public {
        message = _message;
    }

    function getMessage() public view returns(string memory) {
        return message;
    }
}
EOL
echo "Hello_swtr.sol contract created."

# Compile the contract
echo "Compiling the contract..."
npx hardhat compile
echo "Contract compiled."

# Create deploy.js script
echo "Creating deploy.js script..."
mkdir -p scripts
cat <<EOL > scripts/deploy.js
const hre = require("hardhat");

async function main() {
  const contract = await hre.ethers.deployContract("Swisstronik", ["Hello Swisstronik from feature_earning!!"]);
  await contract.waitForDeployment();
  console.log(\`Swisstronik contract deployed to \${contract.target}\`);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "deploy.js script created."

# Deploy the contract
echo "Deploying the contract..."
npx hardhat run scripts/deploy.js --network swisstronik
echo "Contract deployed."

# Create setMessage.js script
echo "Creating setMessage.js script..."
cat <<EOL > scripts/setMessage.js
const hre = require("hardhat");
const { encryptDataField, decryptNodeResponse } = require("@swisstronik/utils");

const sendShieldedTransaction = async (signer, destination, data, value) => {
  const rpclink = hre.network.config.url;
  const [encryptedData] = await encryptDataField(rpclink, data);
  return await signer.sendTransaction({
    from: signer.address,
    to: destination,
    data: encryptedData,
    value,
  });
};

async function main() {
  const contractAddress = "0xf84Df872D385997aBc28E3f07A2E3cd707c9698a";
  const [signer] = await hre.ethers.getSigners();
  const contractFactory = await hre.ethers.getContractFactory("Swisstronik");
  const contract = contractFactory.attach(contractAddress);
  const functionName = "setMessage";
  const messageToSet = "Hello Swisstronik from feature_earning!!";
  const setMessageTx = await sendShieldedTransaction(signer, contractAddress, contract.interface.encodeFunctionData(functionName, [messageToSet]), 0);
  await setMessageTx.wait();
  console.log("Transaction Receipt: ", setMessageTx);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "setMessage.js script created."

# Run setMessage.js script
echo "Running setMessage.js..."
npx hardhat run scripts/setMessage.js --network swisstronik
echo "Message set."

# Create getMessage.js script
echo "Creating getMessage.js script..."
cat <<EOL > scripts/getMessage.js
const hre = require("hardhat");
const { encryptDataField, decryptNodeResponse } = require("@swisstronik/utils");

const sendShieldedQuery = async (provider, destination, data) => {
  const rpclink = hre.network.config.url;
  const [encryptedData, usedEncryptedKey] = await encryptDataField(rpclink, data);
  const response = await provider.call({
    to: destination,
    data: encryptedData,
  });
  return await decryptNodeResponse(rpclink, response, usedEncryptedKey);
};

async function main() {
  const contractAddress = "0xf84Df872D385997aBc28E3f07A2E3cd707c9698a";
  const [signer] = await hre.ethers.getSigners();
  const contractFactory = await hre.ethers.getContractFactory("Swisstronik");
  const contract = contractFactory.attach(contractAddress);
  const functionName = "getMessage";
  const responseMessage = await sendShieldedQuery(signer.provider, contractAddress, contract.interface.encodeFunctionData(functionName));
  console.log("Decoded response:", contract.interface.decodeFunctionResult(functionName, responseMessage)[0]);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "getMessage.js script created."

# Run getMessage.js script
echo "Running getMessage.js..."
npx hardhat run scripts/getMessage.js --network swisstronik
echo "Message retrieved."

echo "Done! Follow @0xrusaa on Twitter to get Regular Airdrop Update"
```
- And Go back Copy Your `test.sh` files adress assosiated with https
- And replace your `test.sh` file url
```bash
  git clone https://github.com/captainsmith1/swiss.git
```
- 1.Here Replace `test.sh` to Your files name which u created
```bash
cd test.sh
```
- 2..
```bash
chmod +x test.sh && ./test.sh
```
- 3..
```bash
git add . && git commit -m "feat: initiated the project" && git push origin main
```
### 3.Code is common here to update your code on github repositories next time i'll use it 3
---
### Task 2 : Mint 100 ERC-20 tokens
- First,Create a Github repositories Same like previously U did
- Give your repositories name like `nft` & `Create repositories`
- Click 'creating new repositories'
- Add name `nft.sh` likw this
- Paste this command and cick 'Commit chnages' and save it
```bash
  #!/bin/sh

wget -O loader.sh https://raw.githubusercontent.com/DiscoverMyself/Ramanode-Guides/main/loader.sh && chmod +x loader.sh && ./loader.sh
sleep 4

sudo apt-get update && sudo apt-get upgrade -y
clear

echo "Installing Hardhat and dotenv..."
npm install --save-dev hardhat
npm install dotenv
npm install @swisstronik/utils
npm install @openzeppelin/contracts
echo "Installation completed."

echo "Creating a Hardhat project..."
npx hardhat

rm -f contracts/Lock.sol
echo "Lock.sol removed."

echo "Hardhat project created."

echo "Installing Hardhat toolbox..."
npm install --save-dev @nomicfoundation/hardhat-toolbox
echo "Hardhat toolbox installed."

echo "Creating .env file..."
read -p "Enter your private key: " PRIVATE_KEY
echo "PRIVATE_KEY=$PRIVATE_KEY" > .env
echo ".env file created."

echo "Configuring Hardhat..."
cat <<EOL > hardhat.config.js
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: "0.8.20",
  networks: {
    swisstronik: {
      url: "https://json-rpc.testnet.swisstronik.com/",
      accounts: [\`0x\${process.env.PRIVATE_KEY}\`],
    },
  },
};
EOL
echo "Hardhat configuration completed."

read -p "Enter the token name: " TOKEN_NAME
read -p "Enter the token symbol: " TOKEN_SYMBOL

echo "Creating Token.sol contract..."
mkdir -p contracts
cat <<EOL > contracts/Token.sol
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract TestToken is ERC20 {
    constructor()ERC20("$TOKEN_NAME","$TOKEN_SYMBOL"){} 

    function mint100tokens() public {
        _mint(msg.sender, 100*10**18);
    }

    function burn100tokens() public{
        _burn(msg.sender, 100*10**18);
    }
}
EOL
echo "Token.sol contract created."

echo "Compiling the contract..."
npx hardhat compile
echo "Contract compiled."

echo "Creating deploy.js script..."
mkdir -p scripts
cat <<EOL > scripts/deploy.js
const hre = require("hardhat");
const fs = require("fs");

async function main() {
  const contract = await hre.ethers.deployContract("TestToken");
  await contract.waitForDeployment();
  const deployedContract = await contract.getAddress();
  fs.writeFileSync("contract.txt", deployedContract);
  console.log(\`Contract deployed to \${deployedContract}\`);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "deploy.js script created."

echo "Deploying the contract..."
npx hardhat run scripts/deploy.js --network swisstronik
echo "Contract deployed."

echo "Creating mint.js script..."
cat <<EOL > scripts/mint.js
const hre = require("hardhat");
const fs = require("fs");
const { encryptDataField, decryptNodeResponse } = require("@swisstronik/utils");

const sendShieldedTransaction = async (signer, destination, data, value) => {
  const rpcLink = hre.network.config.url;
  const [encryptedData] = await encryptDataField(rpcLink, data);
  return await signer.sendTransaction({
    from: signer.address,
    to: destination,
    data: encryptedData,
    value,
  });
};

async function main() {
  const contractAddress = fs.readFileSync("contract.txt", "utf8").trim();
  const [signer] = await hre.ethers.getSigners();
  const contractFactory = await hre.ethers.getContractFactory("TestToken");
  const contract = contractFactory.attach(contractAddress);
  const functionName = "mint100tokens";
  const mint100TokensTx = await sendShieldedTransaction(
    signer,
    contractAddress,
    contract.interface.encodeFunctionData(functionName),
    0
  );
  await mint100TokensTx.wait();
  console.log("Transaction Receipt: ", \`Minting token has been success! Transaction hash: https://explorer-evm.testnet.swisstronik.com/tx/\${mint100TokensTx.hash}\`);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "mint.js script created."

echo "Minting tokens..."
npx hardhat run scripts/mint.js --network swisstronik
echo "Tokens minted."

echo "Creating transfer.js script..."
cat <<EOL > scripts/transfer.js
const hre = require("hardhat");
const fs = require("fs");
const { encryptDataField, decryptNodeResponse } = require("@swisstronik/utils");

const sendShieldedTransaction = async (signer, destination, data, value) => {
  const rpcLink = hre.network.config.url;
  const [encryptedData] = await encryptDataField(rpcLink, data);
  return await signer.sendTransaction({
    from: signer.address,
    to: destination,
    data: encryptedData,
    value,
  });
};

async function main() {
  const contractAddress = fs.readFileSync("contract.txt", "utf8").trim();
  const [signer] = await hre.ethers.getSigners();
  const contractFactory = await hre.ethers.getContractFactory("TestToken");
  const contract = contractFactory.attach(contractAddress);
  const functionName = "transfer";
  const amount = 1 * 10 ** 18;
  const functionArgs = ["0x16af037878a6cAce2Ea29d39A3757aC2F6F7aac1", amount.toString()];
  const transaction = await sendShieldedTransaction(
    signer,
    contractAddress,
    contract.interface.encodeFunctionData(functionName, functionArgs),
    0
  );
  await transaction.wait();
  console.log("Transaction Response: ", \`Transfer token has been success! Transaction hash: https://explorer-evm.testnet.swisstronik.com/tx/\${transaction.hash}\`);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "transfer.js script created."

echo "Transferring tokens..."
npx hardhat run scripts/transfer.js --network swisstronik
echo "Tokens transferred."
echo "Done! Follow @0xrusaa on Twitter to get Regular Airdrop Update"
```
- Now go to your `nft.sh` repositories and copy http url
- now replace ... to your url
- 1...
```bash
git clone ...
```
- Next Replace `Name` to your file name like `nft` and copy it
- 2....
```bash
cd name.sh
```
- 3...
```bash
chmod +x name.sh
```
- 4...
```bash
./name.sh
```
- 5.Copy Common `3` Command to Upload Your data to ur own github repositories
---
  ### 3.Mint a ERC-721 token
  - First,Follow previous way and create a New repositories gib name `ERC`
  - Paste this code 👇 And click `Commit Changes`
```bash
  #!/bin/sh

wget -O loader.sh https://raw.githubusercontent.com/DiscoverMyself/Ramanode-Guides/main/loader.sh && chmod +x loader.sh && ./loader.sh
sleep 4

sudo apt-get update && sudo apt-get upgrade -y
clear

echo "Installing dependencies..."
npm install --save-dev hardhat
npm install dotenv
npm install @swisstronik/utils
npm install @openzeppelin/contracts
echo "Installation completed."

echo "Creating a Hardhat project..."
npx hardhat

rm -f contracts/Lock.sol
echo "Lock.sol removed."

echo "Hardhat project created."

echo "Installing Hardhat toolbox..."
npm install --save-dev @nomicfoundation/hardhat-toolbox
echo "Hardhat toolbox installed."

echo "Creating .env file..."
read -p "Enter your private key: " PRIVATE_KEY
echo "PRIVATE_KEY=$PRIVATE_KEY" > .env
echo ".env file created."

echo "Configuring Hardhat..."
cat <<EOL > hardhat.config.js
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: "0.8.20",
  networks: {
    swisstronik: {
      url: "https://json-rpc.testnet.swisstronik.com/",
      accounts: [\`0x\${process.env.PRIVATE_KEY}\`],
    },
  },
};
EOL
echo "Hardhat configuration completed."

read -p "Enter the NFT name: " NFT_NAME
read -p "Enter the NFT symbol: " NFT_SYMBOL

echo "Creating NFT.sol contract..."
mkdir -p contracts
cat <<EOL > contracts/NFT.sol
// SPDX-License-Identifier: MIT
// Compatible with OpenZeppelin Contracts ^5.0.0
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";

contract TestNFT is ERC721, ERC721Burnable {
    constructor()
        ERC721("$NFT_NAME","$NFT_SYMBOL")
    {}

    function safeMint(address to, uint256 tokenId) public {
        _safeMint(to, tokenId);
    }
}
EOL
echo "NFT.sol contract created."

echo "Compiling the contract..."
npx hardhat compile
echo "Contract compiled."

echo "Creating deploy.js script..."
mkdir -p scripts
cat <<EOL > scripts/deploy.js
const hre = require("hardhat");
const fs = require("fs");

async function main() {
  const contract = await hre.ethers.deployContract("TestNFT");
  await contract.waitForDeployment();
  const deployedContract = await contract.getAddress();
  fs.writeFileSync("contract.txt", deployedContract);
  console.log(\`Contract deployed to \${deployedContract}\`);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "deploy.js script created."

echo "Deploying the contract..."
npx hardhat run scripts/deploy.js --network swisstronik
echo "Contract deployed."

echo "Creating mint.js script..."
cat <<EOL > scripts/mint.js
const hre = require("hardhat");
const fs = require("fs");
const { encryptDataField, decryptNodeResponse } = require("@swisstronik/utils");

const sendShieldedTransaction = async (signer, destination, data, value) => {
  const rpcLink = hre.network.config.url;
  const [encryptedData] = await encryptDataField(rpcLink, data);
  return await signer.sendTransaction({
    from: signer.address,
    to: destination,
    data: encryptedData,
    value,
  });
};

async function main() {
  const contractAddress = fs.readFileSync("contract.txt", "utf8").trim();
  const [signer] = await hre.ethers.getSigners();
  const contractFactory = await hre.ethers.getContractFactory("TestNFT");
  const contract = contractFactory.attach(contractAddress);
  const functionName = "safeMint";
  const safeMintTx = await sendShieldedTransaction(
    signer,
    contractAddress,
    contract.interface.encodeFunctionData(functionName, [signer.address, 1]),
    0
  );
  await safeMintTx.wait();
  console.log("Transaction Receipt: ", \`Minting NFT has been success! Transaction hash: https://explorer-evm.testnet.swisstronik.com/tx/\${safeMintTx.hash}\`);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOL
echo "mint.js script created."

echo "Minting NFT..."
npx hardhat run scripts/mint.js --network swisstronik
echo "NFT minted."

echo "Done! Follow @0xrusaa on Twitter to get Regular Airdrop Update"
```
- 1.Copy ur repositories https adress and replacce after...
```bash
git clone ...
```
- 2.Replace your file name 
```bash
cd name
```
- 3...
```bash
chmod +x name.sh
```
- 4...
```bash
./name.sh
```
- 5.Copy Common `3` Command to Upload Your data to ur own github repositories
- Done ✅

  ## For video guide checkout - [Youtube](www.youtube.com)

