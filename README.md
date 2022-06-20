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
9. Deploy the price feed contract for each asset, test the interface and save their addresses.
10. After a contract is successfully deployed, you can see the instance under *Deployed Contracts*, where you can get your contract address and interact with the contract manually (e.g. if you call the `balanceOf` function of XUSD and enter your account address, you will get the number of XUSD tokens as output). 

</details>
