---
description: Deploy and test ewasm smart contracts via an interactive web3 console
---

# CyberMiles ewasm testnet

The CyberMiles Public Blockchain runs an Ewasm testnet based on its blockchain software and Second State's SSVM. The easiest way to access the testnet is via Docker.

![https://asciinema.org/a/321762?speed=8](../../.gitbook/assets/cm_ewasm_testnet_connect.gif)

Start by pulling the Second State DevChain Docker image.

```text
$ docker pull secondstate/devchain:devchain
```

You can now start the interactive console via Docker. Please note that the password is subject to change. Do not abuse it!

```text
$ docker run --rm -it secondstate/devchain:devchain attach http://ewasm:3WAeT4CYSkMTAPuF@23.98.151.156
```

We recommend you to create your own account and then give yourself a little CMTs from our faucet account \(`0x1bba632055efb57aa991a9b0e900194e2ea037ad`\).

```text
// Create a new account
> personal.newAccount("mypass");
"0xMY_ACCOUNT_ADDRESS"

// Unlock the faucet account
> personal.unlockAccount("0x1bba632055efb57aa991a9b0e900194e2ea037ad", "3WAeT4CYSkMTAPuF");
true

// Transfer 5 CMTs from the facuet account to the newly created account
> cmt.sendTransaction({"from": "0x1bba632055efb57aa991a9b0e900194e2ea037ad", "to": "0xMY_ACCOUNT_ADDRESS", "value": web3.toWei(5, "cmt")})

// Unlock the new account
> personal.unlockAccount("0xMY_ACCOUNT_ADDRESS", "mypass");
```

Now you can [follow the tutorial](run-an-ewasm-smart-contract.md) to deploy and test your ewasm smart contracts.

{% page-ref page="run-an-ewasm-smart-contract.md" %}

![https://asciinema.org/a/321767?speed=8](../../.gitbook/assets/cm_ewasm_testnet_deploy.gif)

{% embed url="https://asciinema.org/a/321767?speed=8" caption="Watch the screencast" %}





