# Synthetic Dapp

<!--step0-->

- **Who is this for**: Dapp developers, DeFi learners, XinFin users.
- **What you'll learn**: DeFi concepts such as synthetic assets, decentralized exchange protocols; how to create a full-stack decentralized applications using solidity.
- **What you'll build**: A decentralized trading platform to enable exchanges among synthetic assets on the XinFin Testnet "Apothem network".
- **Prerequisites**: We assume you are familiar with basic blockchain primitives and solidity programming skills.
- **How long**: This course contains 4 parts and takes about 4 weeks to complete.

## How to start this course

1. Above these instructions, right-click **Use this template** and open the link in a new tab.
2. In the new tab, follow the prompts to create a new repository.
3. After your new repository is created, follow the step-by-step instructions in the README.

<details id=0>
<summary><h2>Step 0: Overview</h2></summary>
   
 **Synthetic assets** are tokenized derivatives that produce the same value as another asset. It enables you to trade some assets without holding the asset itself. For instance, on XinFin, you can trade synthetic assets representing fiat currencies (e.g. synthetic USD), other cryptocurrencies like ETH, even stocks (e.g. synthetic TSLA), which behave like the underlying asset by tracking its price using **data oracles** (will be explained later). 

We want to create a decentralized system to mint, manage, exchange synthetic assets, here are several example use cases to illustrate the application:

1. Marry wants to invest in Tesla stock on XinFin network, she mints synthetic Tesla (sTSLA) tokens by sending XUSD (XinFin stable coin) as collateral.
2. Linda owns lots of synthetic Ethers (sETH) and sTSLA tokens and puts these tokens in a liquidity pool for rewards.
3. Tom owns lots of sETH and wants to exchange them for sTSLA.


### Participants

In the above examples, three people involved represent three participants in our system, they are

* **Minter**: create Collateralized debt positions (CDP) in order to obtain newly minted tokens of a synthetic asset. CDPs can accept collateral in the form of XUSD and must maintain a collateral ratio above the minimum rate.
* **Liquidity provider**: add tokens to the corresponding pool, which increases liquidity for that market. 
* **Trader**: buy and sell synthetic tokens through a Uniswap-like protocol

Do not worry if you can not fully understand the role of each participant and some terminologies now, they will be introduced in more detail in subsequent parts. 

### Tokens

As suggested in the examples, three tokens will be used in our system. (XDC is not listed, but will also be used to pay gas fees.)

| Token Name | Token Symbol | Function | Type |
| -------- | -------- | -------- | -------- |
|XinFin USD| XUSD | stable coin | XRC-20 token|
|Synthetic ETH| sETH | synthetic asset | XRC-20 token|
|Synthetic TSLA| sTSLA| synthetic asset | XRC-20 token|

### Smart Contracts

The whole system consists of three smart contracts, each for one part. 

| Contract | Function |  |
| -------- | -------- |-------- |
|PriceFeed     | An interface to get prices for synthetic assets from oracle     | Step 1|
|Mint | For CDP creation, management, and liquidation |Step 2|
|SynthSwap | A Uniswap-like automated market maker (AMM) protocol |Step 3|

After completing the above three parts, the last part will integrate three contracts with a web client to finish an end product.

</details>

<!--endstep0-->

<details id=1>
<summary><h2>Step 1: Deploy Token Contracts and Price Feeds</h2></summary>

In this part, you need to set up the environment and create three tokens introduced before. These tokens all follow the XRC-20 standards  (standard similar to ERC-20 on Ethereum) and use the interface provided by [OpenZeppelin](https://docs.openzeppelin.com/contracts/4.x/erc20). The smart contracts are provided in the following files:

```
contracts/XUSD.sol
contracts/sAsset.sol
```
   
where XUSD is the stable coin and sAsset is the base struct for synthethic assets (sETH and sTSLA). To reflect the value of these assets, we need to provide price feeds. In this project we use static centralized price feeds that rely on a trusted third-party to report prices manually for simplicity. In the case when data consumers may not want to trust any single data provider, data oracles provide a decentralized and trustworthy way for blockchains to access external data sources. 

The data feeds interface is provided in `interfaces/IPriceFeed.sol`, you need to implement your `PriceFeed.sol` and deploy one instance for each synthetic asset to provide their prices with respect to USD.

   
### Instructions
To deploy these contracts on XinFin testnet, you first need to create some public accounts using **Metamask** and connect to XinFin Apothem testnet. Here are the step-by-step instuctions:
   
1. Install [MetaMask](https://chrome.google.com/webstore/detail/metamask/nkbihfbeogaeaoehlefnkodbefgpgknn) on Chrome, follow the instructions on the app to create a new wallet. After entering the correct phrases, a new account will be created automatically. You can create any number of accounts by clicking the upper right icon and *Create Account*.
2. Manually add XDC testnet into your network with the following parameters: 
   
	```
	Network Name: XinFin Apothem Testnet
	New RPC URL: https://apothemxdcpayrpc.blocksscan.io/
	Chain ID: 51
	```  
3. Get some free XDC: go to a [faucet](https://faucet.apothem.network/) and enter your address, you will get 1000 XDC for testing.
4. Open [Remix](https://remix.xinfin.network/) in your web browser, this is an online IDE where you will write, test and deploy your smart contracts.
5. Add provided contracts in Remix and compile them in the *Solidity compiler* tab. 
6. In the *Deploy & run transactions* tab, set the environment to *Injected Web3*. This will launch a popup page to connect with your wallet. 
7. Create XUSD by deploying `XUSD.sol`, this will create a contract deployment transaction. The information and status of the transaction will be displayed in the terminal. If the deployment is successful, you can check the transaction by hash like [this](https://explorer.apothem.network/txs/0x621f1dd45aa0edef87a8d2fe3d6c0ed3bc4687cda22d6d517fdf9d322711d5d4).
8. Create sETH and sTSLA by deploying `sAsset.sol` with corresponding parameters ``(name, symbol, initialSupply)``, `name` and `symbol` are provided in the token table, `initialSupply = 0`.
9. Deploy the price feed contract for each asset (parameters `t0`, `t1` are the name of base token and quote token, in our example, `t0` is `ETH` and `TSLA`, `t1` is `USD`), test the interface and save their addresses.
10. After a contract is successfully deployed, you can see the instance under *Deployed Contracts*, where you can get your contract address and interact with the contract manually (e.g. if you call the `balanceOf` function of XUSD and enter your account address, you will get the number of XUSD tokens as output). 

</details>


<details id=2>
<summary><h2>Step 2: Mint Synthetic Assets </h2></summary>

As we know, synthetic assets are created to mimic the value of other assets, but value cannot be created out of nothing. To mint some amount of sAsset, a certain amount of existing tokens (usually at a higher value) need to be collateralized. Such binding commitments are represented by collateralized debt positions (CDP).

**Collateralized debt positions (CDP)** is the position created by locking collateral to generate synthetic assets. For instance, you hold stablecoin EUSD worth $10,000. You could collateralize these tokens in a CDP to mint $5,000 worth of the sAsset sTSLA. At this point, your CDP holds $10,000 worth of EUSD as collateral and owes $5,000 worth of sTSLA.

The minted sTSLA tokens are freely tradeable to exploit the price changes. For example, to short TSLA, you can sell tokens in the DEX market and wait until the price drops, then you buy tokens back to close the position. 


The value of the collateral must exceed the value of synthetic assets to avoid debt risk. The ratio of the value of a CDP's locked collateral to the value of its current minted tokens is called **collateral ratio** (e.g. in the above example, the collateral ratio is `$10,000 / $5,000 = 2`). Each asset has a **minimum collateral ratio (MCR)** (in this project, we define `MCR=2`),  and the CDP is required to always maintain the ratio above its MCR. If the price of a synthetic asset rises resulting in a collateral ratio lower than MCR, minters of associated CDP would be pressured to deposit more collateral to maintain the MCR. Otherwise, the protocol will initiate a margin call to liquidate collateral. 

In this part, you need to implement the `contracts/Mint.sol` contract which allows anyone to 

1. open a CDP by sending EUSD as collateral and mint sAsset (`openPosition`)
2. close a CDP to withdraw EUSD and burn sAsset (`closePosition`)
3. deposit EUSD to an existing CDP (`deposit`)
4. withdraw EUSD from an existing CDP (`withdraw`)
5. mint sAsset from an existing CDP (`mint`)
6. return and burn sAsset to an existing CDP (`burn`)

The interfaces of these functions are defined in `interfaces/IMint.sol`. Your task is to implement these functions in `Mint.sol` according to the specifications below. The liquidation function is not required in this project.

### Struct
There are two predefined structs. `Asset` represents information related to each sAsset such as token contract address, price feed contract address and MCR. `Position` represents an active CDP including an index, the owner of the position, the amount of the collateral tokens, and the address and the amount of the asset tokens.

### State variables
Four variables are used to record the states of the contract.

* `_assetMap` is a mapping from sAsset token address to an Asset struct, which includes all registered asset users can mint.
* `_currentPositionIndex` is an incremental integer starting from 0, whenever a new position is opened, the current index will be assigned to the position and increase by 1.
* `_idxPositionMap` is a mapping from position index to a Position struct used to store all existing positions.
* `collateralToken` is the address of the collateral token contract, in this project, we only accept one type of token (EUSD) as collateral whose address is specified in the constructor.

### Functions
There are some functions already implemented for the initial setup.

* `registerAsset` is used to register a new asset given the address of the new asset, MCR and the price feed contract address. It can only be called by the owner of the Mint contract.
* `getPosition` is a view function that returns the information about a position at the given index.
* `checkRegistered` is a view function that checks whether the input asset token is registered.
* `getMintAmount` is a helper function that calculates the amount of asset token minted given the amount of collateral and ratio.

The remaining functions are left for you to implement.

* `openPosition`: Create a new position by transferring `collateralAmount` EUSD tokens from the message sender to the contract. Make sure the asset is registered and the input collateral ratio is not less than the asset MCR, then calculate the number of minted tokens to send to the message sender.
* `closePosition`: Close a position when the position owner calls this function. Burn all sAsset tokens and transfer EUSD tokens locked in the position to the message sender. Finally, delete the position at the given index.
* `deposit`: Add collateral amount of the position at the given index. Make sure the message sender owns the position and transfer deposited tokens from the sender to the contract.
* `withdraw`: Withdraw collateral tokens from the position at the given index. Make sure the message sender owns the position and the collateral ratio won't go below the MCR. Transfer withdrawn tokens from the contract to the sender.
* `mint`: Mint more asset tokens from the position at the given index. Make sure the message sender owns the position and the collateral ratio won't go below the MCR. 
* `burn`: Contract burns the given amount of asset tokens in the position. Make sure the message sender owns the position.


</details>
