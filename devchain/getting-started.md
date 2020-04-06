# Getting started

This document describes how to get started with application development on the Second State DevChain. The Second State DevChain features a powerful and easy-to-use virtual machine that can quickly get you started with smart contract and DApp development.

{% hint style="info" %}
With the [Second State Blockchain as a Service \(BaaS\)](baas.md), you can start a new Second State blockchain with its innovative virtual machines and smart contract search engines with one click of the mouse.
{% endhint %}

In this document, we will explain how to create and run the Second State blockchain. You can then connect and test basic features such as coin transactions and smart contract functions.

The easiest way to get started is to use our pre-build Docker images. Please make sure that you have [Docker installed](https://docs.docker.com/install/) and that your [Docker can work without sudo](https://docs.docker.com/install/linux/linux-postinstall/).

For example, on Ubuntu, you can use the following commands.

```bash
$ sudo apt install docker.io
$ sudo usermod -a -G docker $USER
$ docker pull secondstate/devchain:devchain
```

### Initialize

Let’s initialize the DevChain configuration and genesis settings.

```bash
$ docker run --rm -v $HOME/.devchain:/devchain secondstate/devchain:devchain node init --home /devchain
```

Note: If you are running a cluster, you should now copy over the cluster wide `genesis.json` and `config.toml` files to the `$HOME/.devchain/config` directory.

### Run

Now you can start the DevChain node in docker.

```bash
$ docker run --rm -v $HOME/.devchain:/devchain -p 26657:26657 -p 8545:8545 secondstate/devchain:devchain node start --home /devchain
```

You should see blocks like the following in the log.

```text
INFO [07-14|07:23:05] Imported new chain segment               blocks=1 txs=0 mgas=0.000 elapsed=431.085µs mgasps=0.000 number=163 hash=05e16c…a06228
INFO [07-14|07:23:15] Imported new chain segment               blocks=1 txs=0 mgas=0.000 elapsed=461.465µs mgasps=0.000 number=164 hash=933b97…0c340c
```

### Connect

Next, open another terminal window to interact with the running node.

You can get the ID of the running Docker container.

```bash
$ docker container ls
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                                                         NAMES
0bcd9da5bf05        secondstate/devchain   "./devchain node sta…"   4 minutes ago       Up 4 minutes        0.0.0.0:8545->8545/tcp, 0.0.0.0:26657->26657/tcp, 26656/tcp   pedantic_mendeleev
```

Next, log into that container.

```bash
$ docker exec -i -t 0bcd9da5bf05 bash
root@0bcd9da5bf05:/app# ls
devchain  devchain.sha256  lib
```

Finally, you can attach a console to the node to run web3 commands.

```bash
root@0bcd9da5bf05:/app# ./devchain attach http://localhost:8545
...
> cmt.syncing
{
  catching_up: false,
  latest_app_hash: "C7D8AECE081DF06FFC9BF6144A50B37CA5DD8A8E",
  latest_block_hash: "B592D63AB78C571E0FB695A052681E65F6DFE15B",
  latest_block_height: 35,
  latest_block_time: "2019-05-04T02:59:30.542783017Z"
}
```

### Test transactions

You can now send a transaction between accounts like the following.

```typescript
personal.unlockAccount("from_address")
cmt.sendTransaction({"from": "from_address", "to": "to_address", "value": web3.toWei(0.001, "cmt")})
```

Next, you can paste the following script into the client console, at the &gt; prompt.

```typescript
function checkAllBalances() {
  var totalBal = 0;
  for (var acctNum in cmt.accounts) {
      var acct = cmt.accounts[acctNum];
      var acctBal = web3.fromWei(cmt.getBalance(acct), "cmt");
      totalBal += parseFloat(acctBal);
      console.log("  cmt.accounts[" + acctNum + "]: \t" + acct + " \tbalance: " + acctBal + " CMT");
  }
  console.log("  Total balance: " + totalBal + "CMT");
};
```

You can now run the script in the console, and see the results.

```typescript
> checkAllBalances();
cmt.accounts[0]:      0x6....................................230      balance: 466.798526 CMT
cmt.accounts[1]:      0x6....................................244      balance: 1531 CMT
Total balance: 1997.798526CMT
```







