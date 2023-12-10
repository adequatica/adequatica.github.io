---
layout: post
title: "How I Tried to Add TRX to MetaMask Wallet"
date: 2022-04-01 08:34:54 +0300
tags: crypto metamask
---

My first dive into cryptocurrency revealed the pitfalls that are difficult to suspect until you encounter them.

What I had at the beginning:

- USDT and TRX tokens at [Binance](https://www.binance.com);
- [MetaMask](https://metamask.io/) crypto wallet.

Why MetaMask?

- A few friends advised that this is «a simple wallet for beginners, but works only with [Ethereum](https://ethereum.org)»;
- Googling about USDT shows that it is [hosted on the Ethereum blockchain](<https://en.wikipedia.org/wiki/Tether_(cryptocurrency)>), so it should work.

But reality was not so easy, for USDT (Tether) and especially for TRX (Tronix).

Why did I decide to transfer currency from a cryptocurrency exchange into a wallet?

1. Your assets in the exchange are just rows in a database (not real assets);
2. Your assets in the exchange are stored in a [custodial wallet](https://academy.binance.com/en/articles/custodial-vs-non-custodial-wallets-what-s-the-difference), which means a third party (e.g., the exchange) has access to your account — you must be 100% confident in the reliability of the cryptocurrency exchange;
3. No matter how reliable the exchange is, you never know what might come from a big regulators’ side (bans on transfers for certain citizens or disabling certain banks).

---

When you decide to withdraw crypto from Binance you’ll see the select «Network» where is a list of networks indicating the amount of the commission:

![Select network](/assets/2022-04-01/01-select-network.png)

To transfer USDT by ETH (Ethereum) costs 10 USDT [per transaction](https://academy.binance.com/en/articles/what-are-blockchain-transaction-fees) — it is too expensive, while transfer by TRX or BSB is around 1 USDT.

Because I already had prepared TRX tokens, it would be convenient for me to make a transaction by choosing this network ([Tron](https://tron.network)). But there are no networks in MetaMask, that support transfer by BSB or TRX — only ETH by Ethereum Main Network.

After a few dozen minutes of [lurking through the Internet I found out](https://bitcointalk.org/index.php?topic=5343213.20) that **TRON blockchain is not supported by MetaMask**, but it is possible to add manually [Binance Smart Chain](https://coinmarketcap.com/alexandria/article/what-is-binance-smart-chain) (BSC) as a workaround.

> My initial idea was to have USDT as a payment currency and TRX as a transport (a currency for transaction fees), but it failed at the start. Despite everything, I still decided to check how to transfer TRX tokens through BSC to MetaMask.

## 1. Add BSC (BEP20) Network to MetaMask

There is detailed instruction on [how to add Binance Smart Chain to MetaMask in a browser extension](https://docs.binance.org/smart-chain/wallet/metamask.html). If you have the app (as I did), it is almost the same:

1\. Go to: Settings > Networks > [Add Network]

2\. Fill out the form for «New RPC Network»:

![New RPC Network](/assets/2022-04-01/02-new-rpc-network.png)

- Network Name (can be arbitrary): Binance Smart Chain
- RPC Url: [https://bsc-dataseed.binance.org](https://docs.binance.org/smart-chain/developer/rpc.html)
- ChainID: 56
- Symbol: BNB
- Block Explorer URL: [https://bscscan.com](https://bscscan.com)

3\. [Add] new network;

4\. Tap on the «Wallet» and choose network: Binance Smart Chain.

![Other Networks](/assets/2022-04-01/03-other-networks.png)

Now you are able to test receiving tokens through this network.

![Account](/assets/2022-04-01/04-account.png)

## 2. Add TRX tokens to MetaMask

Before transferring the real TRX tokens, you need to add these kinds of tokens to MetaMask:

1. Select [Import Tokens] in your account;
2. Fill out the form for «Custom Token»:

- Token Address: contract address can be obtained from the [CoinMarketCap](https://coinmarketcap.com/currencies/tron/):

![CoinMarketCap: TRON](/assets/2022-04-01/05-coinmarketcap-tron.png)

- Token Symbol and Token of Precision will automatically be filled in:

![Custom token](/assets/2022-04-01/06-custom-token.png)

After import, you will get TRX token in your wallet:

![Acount](/assets/2022-04-01/07-account.png)

## 3. Withdraw TRX tokens from Binance to MetaMask

The last step is to [withdraw TRX from Binance](https://www.binance.com/en/support/faq/115003670492) to MetaMask wallet:

![Withdraw Crypto](/assets/2022-04-01/08-withdraw-crypto.png)

- Address: your MetaMask’s public address;
- Network: BSC — it will be the only choice because other networks will be removed if they are unmatched with your address:

![Select network](/assets/2022-04-01/09-select-network.png)

After withdrawing, you will get your TRX tokens into your wallet in less than a few minutes:

![Account](/assets/2022-04-01/10-account.png)

USDT tokens could be transferred in the same way (by BSC network).

### And one more thing (pitfall)…

I immediately decided to check how transferable TRX tokens from MetaMask to another wallet: inside the wallet, I selected TRX > Send, and on the «Sent to» screen, I saw that there is only one type of tokens to transfer — BNB — default token for BSC network:

![Send to](/assets/2022-04-01/11-send-to.png)

At least, you can add TRX tokens to MetaMask, but for sending them from MetaMask, you have to convert (swap) them to other tokens. And for swapping, you will need more tokens [for the fees](https://metamask.zendesk.com/hc/en-us/articles/360022895972).

Then I thought that maybe the functionality in the application is limited and there are more options in the [browser extension](https://metamask.io/download/). I chose Firefox’s extension, imported my existing account, and saw 0 TRX tokens. It turns out that to connect the app and the extension, you had to make a «Sync with mobile», but [it does not work](https://metamask.zendesk.com/hc/en-us/articles/360032378452-How-to-Sync-Mobile-with-MetaMask-Extension):

![Sync with mobile](/assets/2022-04-01/12-sync-with-mobile.png)

_Mobile Sync is disabled in MetaMask_

So, your MetaMask wallet is rigidly tied to the app on the phone (or to the extension in the browser).

Summary:

- If you are a noobie in crypto, be prepared for the unexpected WTFs;
- If your friend says that it is easy, be prepared that it is not;
- Maybe you should choose another crypto wallet ([Exodus](https://www.exodus.com) or [Trust Wallet](https://trustwallet.com) [works with TRX](https://tron.network/wallet) «out of the box»), then you would not have faced all these difficulties.

But I learned a lot:

- MetaMask works only with[ EVM-compatible blockchains](https://metamask.zendesk.com/hc/en-us/sections/4442278744475-Network-Profiles) ([Ethereum Virtual Machine](https://coinmarketcap.com/alexandria/glossary/ethereum-virtual-machine-evm)) — Ethereum for ETH tokens by default;
- Other networks could be[ manually added to MetaMask](https://satochip.medium.com/metamask-how-to-add-custom-network-binance-smart-chain-polygon-avalanche-43c1c25afd88). But only certain: Binance Smart Chain for BNB tokens, Polygon for MATIC tokens and Avalanche for AVAX tokens);
- Despite[ TRON is based on Ethereum’s code](https://en.wikipedia.org/wiki/Tron_%28cryptocurrency%29), it runs on its own TVM network ([Tron Virtual Machine](https://medium.com/@TronDotLive/what-exactly-is-tron-virtual-machine-tvm-1424ac17b2c7)) and therefore is not supported by MetaMask;
- [Custom non-EVM tokens can still be added to MetaMask](https://consensys.net/blog/metamask/how-to-add-your-custom-tokens-in-metamask/);
- But for further operations with non-EVM tokens in MetaMask, they must be swapped into supported tokens — default tokens of the chosen network (e.g., BNB for BSC);
- To transfer USDT to MetaMask it is easy (and cheaper) to use BNB as a currency for transaction fees;
- More lore through Ethereum’s community guides and resources.

Copy @ [Medium](https://adequatica.medium.com/how-to-add-trx-to-metamask-wallet-f5ff83bb1005)
