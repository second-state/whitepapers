---
description: Use the BUIDL IDE to create and deploy applications on the Ethereum blockchain
---

# Develop for Ethereum

The Ethereum blockchain is the most widely used blockchain smart contract platform. However, it is also heavy congested and very expensive to deploy and run decentralized applications \(dapp\). For Ethereum application developers, we recommend you to develop and run your applications on Ethereum Classic or CyberMiles instead.

{% page-ref page="develop-for-ethereum-classic.md" %}

{% page-ref page="develop-for-cybermiles.md" %}

That said, to develop decentralized applications \(dapps\) on Ethereum using BUIDL is as simple as 1-2-3.

## Step1 Configure BUIDL for Ethereum

The easy way is to just click the link below to launch BUIDL in your browser. It pre-loads all the configurations for you. 

[https://buidl.secondstate.io/eth](https://buidl.secondstate.io/eth)

Of course, you can also manually setup the configurations from the **Providers** tab at the lower left panel of BUIDL.

| Setting | Value |
| :--- | :--- |
| ES Provider Endpoint | https://eth.search.secondstate.io |
| Web3 Provider Endpoint | https://mainnet.infura.io |
| Chain ID | 1 |
| Custom Tx Gas | Checked |
| Gas Price | 10000000000 |
| Gas Limit | 8000000 |

The gas price and limit are the default values when you use BUIDL to deploy or call contracts. They are also used when you call `web3.ss` functions in the JavaScript application without specifying gas.

## Step 2 Get some ETHs for gas

In the **Accounts** tab, you will see 5 auto-generated ETH addresses, and you can set any of them as default. The default address is used to sign transactions from both BUIDL and `web3.ss` applications. Because of that, you will need to send a little ETH \(0.1 ETH is enough\) into that account to pay for gas fees.

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

That's it. Happy coding on Ethereum!

