---
description: >-
  Use the BUIDL IDE to create and deploy applications on the CyberMiles public
  blockchain
---

# Develop for CyberMiles

The CyberMiles blockchain is fully compatible with the Ethereum protocol. It is very fast and very cheap to use. To develop decentralized applications \(dapps\) on CyberMiles using BUIDL is as simple as 1-2-3.

## Step1 Configure BUIDL for CyberMiles

The easy way is to just click the link below to launch BUIDL in your browser. It pre-loads all the configurations for you. 

[https://buidl.secondstate.io/?es\_provider=https%3A%2F%2Fcmt.search.secondstate.io&web3\_provider=https%3A%2F%2Frpc.cybermiles.io%3A8545&web3\_chainId=18&gas\_price=5000000000&gas\_limit=8000000](https://buidl.secondstate.io/?es_provider=https%3A%2F%2Fcmt.search.secondstate.io&web3_provider=https%3A%2F%2Frpc.cybermiles.io%3A8545&web3_chainId=18&gas_price=5000000000&gas_limit=8000000)

Of course, you can also manually setup the configurations from the **Providers** tab at the lower left panel of BUIDL.

| Setting | Value |
| :--- | :--- |
| ES Provider Endpoint | https://cmt.search.secondstate.io |
| Web3 Provider Endpoint | https://rpc.cybermiles.io:8545 |
| Chain ID | 18 |
| Custom Tx Gas | Checked |
| Gas Price | 5000000000 |
| Gas Limit | 8000000 |

The gas price and limit are the default values when you use BUIDL to deploy or call contracts. They are also used when you call `web3.ss` functions in the JavaScript application without specifying gas.

## Step 2 Get some CMTs for gas

In the **Accounts** tab, you will see 5 auto-generated CMT addresses, and you can set any of them as default. The default address is used to sign transactions from both BUIDL and `web3.ss` applications. Because of that, you will need to send a little CMT \(1 CMT is enough\) into that account to pay for gas fees.

If you do not have CMT and do not know where to buy it, send an email with your **Accounts** screenshot and your default address to cmt@secondstate.io, and we will send you 1 CMT.

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

That's it. Happy coding on CyberMiles!

