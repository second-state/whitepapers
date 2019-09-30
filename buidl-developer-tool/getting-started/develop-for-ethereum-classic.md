---
description: >-
  Use the BUIDL IDE to create and deploy applications on the Ethereum Classic
  blockchain
---

# Develop for Ethereum Classic

The Ethereum Classic blockchain is ideal for application developers looking for a reliable, fast, and low cost public blockchain to deploy decentralized applications \(dapps\). Ethereum Classic is a secured by a large global network of miners and is one of the most highly valued blockchain networks. The native cryptocurrency, ETC, is a well-known store-of-value, which makes it a great DeFi platform. Compared with Ethereum \(ETH\), Ethereum Classic \(ETC\) is much less congested and hence much faster. For developers and users, paying blockchain "gas" using ETC costs only about 5% of ETH gas. 

To develop decentralized applications \(dapps\) on Ethereum Classic using BUIDL is as simple as 1-2-3.

## Step1 Configure BUIDL for Ethereum Classic

The easy way is to just click the link below to launch BUIDL in your browser. It pre-loads all the configurations for you. 

[https://buidl.secondstate.io/etc](https://buidl.secondstate.io/etc)

Of course, you can also manually setup the configurations from the **Providers** tab at the lower left panel of BUIDL.

| Setting | Value |
| :--- | :--- |
| ES Provider Endpoint | [https://etc.search.secondstate.io](https://etc.search.secondstate.io) |
| Web3 Provider Endpoint | [https://www.ethercluster.com/etc](https://www.ethercluster.com/etc
) |
| Chain ID | 61 |
| Custom Tx Gas | Checked |
| Gas Price | 15000000000 |
| Gas Limit | 7000000 |

The gas price and limit are the default values when you use BUIDL to deploy or call contracts. They are also used when you call `web3.ss` functions in the JavaScript application without specifying gas.

Furthermore, since the Ethereum Classic requires bytecode generated from Solidity compiler version 0.4.2, you will need to append a `/?s042` URL parameter to BUIDL. All these are done for you at [https://buidl.secondstate.io/etc](https://buidl.secondstate.io/etc)

## Step 2 Get some ETCs for gas

In the **Accounts** tab, you will see 5 auto-generated ETC addresses, and you can set any of them as default. The default address is used to sign transactions from both BUIDL and `web3.ss` applications. Because of that, you will need to send a little ETC \(0.1 ETC is enough\) into that account to pay for gas fees.

If you do not have ETC and do not know where to buy it, send an email with your **Accounts** screenshot and your default address to etc@secondstate.io, and we will send you 0.1 ETC.

## Step 3 Develop and deploy

You can now follow the [Getting Started guide](./) to develop and deploy your smart contract and dapp. The `web3.ss` package is a fully featured replacement for the `web3.eth` package. Of course, in your JavaScript code, you can still use `web3.eth` if you wish.

{% page-ref page="./" %}

The contract creation, and function calls are all executed with the gas price and gas limit you set. You can still specify the gas price and limit on a per transaction basis. Here is an example.

```text
instance.set(n, {
  gas: 100000,
  gasPrice: 10000000000
}, function (e, result) {
  // ... ...
});
```

That's it. Happy coding on Ethereum Classic!

