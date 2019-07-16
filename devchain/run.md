# Run

This document describes how to run the Second State DevChain.

### Binary

The binary executable `devchain` is the software that runs blockchain nodes.

#### Single node

First, you need to initialize the configurations and settings on the node computer.

```text
$ devchain node init --home $HOME/.devchain
```

The `genesis.json` and `config.toml` files will be created under the `$HOME/.devchain/config` directory. You can make changes to those files to customize your blockchain. Then, `set env` variables for libENI.

```text
$ mkdir -p $HOME/.devchain/eni/lib
$ cd $HOME/.devchain/eni

# Get the lib file. For centos 7 the file name is libeni-1.3.4_centos-7.tgz
$ wget https://github.com/second-state/libeni/releases/download/v1.3.4/libeni-1.3.4_ubuntu-16.04.tgz
$ tar zxvf *.tgz
$ cp libeni-1.3.4/lib/* lib

# For convenience, you should also put these two lines in your .bashrc or .zshrc
export ENI_LIBRARY_PATH=$HOME/.devchain/eni/lib
export LD_LIBRARY_PATH=$HOME/.devchain/eni/lib
```

Now you can start the node.

```text
$ devchain node start --home $HOME/.devchain
```

Next, in a new terminal window, run the following command to connect to the local DevChain node.

```text
$ devchain attach http://localhost:8545
> cmt.syncing
{
  catching_up: false,
  latest_app_hash: "07FA113DF14AAC49773DD7EE2B8418740D9DD552",
  latest_block_hash: "AF1415AF0057C52C4A1F7DC80298217A33291AEE",
  latest_block_height: 23,
  latest_block_time: "2019-05-03T21:41:14.581000291Z"
}
```

#### Multiple nodes

First, you need to initialize the configurations and settings on each of the node computer. Run the following command on each machine.

```text
$ devchain node init --home $HOME/.devchain
```

* Each node has a different `$HOME/.devchain/config/priv_validator.json` key file. Note down the public key for each of them.
* On each node, run command `devchain node show_node_id –home $HOME/.devchain` and note down the seed for each. It is in the format of `seed@ip:26656`

Next, use [this tool](https://github.com/second-state/devchain-config) to generate a new set of `genesis.json` and `config.toml` files for the entire cluster. Enter all the public keys and seeds from the last step into the tool. For example, here is how to create a `genesis.json` with a custom `chain_id`, a custom `gas_price` and public keys from multiple nodes.

```text
$ node index.js --type genesis --config_path ./genesis.json.template --chain_id test --params.gas_price 0 --validators.1.pub_key test1 --validators.1.power 101 --validators.2.pub_key test2 --validators.1.power 102
```

Here is how to create a `config.toml` file with the seeds.

```text
$ node index.js --type config --config_path ./config.toml.template --p2p.seeds seed1@ip1:26656,seed2@ip2:26656
```

Copy the generated `genesis.json` and `config.toml` files back into each node’s `$HOME/.devchain/config` directory.

Next, set env variables for libENI.

```text
$ mkdir -p $HOME/.devchain/eni/lib
$ cd $HOME/.devchain/eni

# Get the lib file. For centos 7 the file name is libeni-1.3.4_centos-7.tgz
$ wget https://github.com/second-state/libeni/releases/download/v1.3.4/libeni-1.3.4_ubuntu-16.04.tgz
$ tar zxvf *.tgz
$ cp libeni-1.3.4/lib/* lib

# For convenience, you should also put these two lines in your .bashrc or .zshrc
export ENI_LIBRARY_PATH=$HOME/.devchain/eni/lib
export LD_LIBRARY_PATH=$HOME/.devchain/eni/lib
```

Now you can start each node, and they will form a cluster.

```text
$ devchain node start --home $HOME/.devchain
```

Next, in a new terminal window, run the following command to connect to a local DevChain node in the cluster.

```text
$ devchain attach http://localhost:8545
> cmt.syncing
{
  catching_up: false,
  latest_app_hash: "07FA113DF14AAC49773DD7EE2B8418740D9DD552",
  latest_block_hash: "AF1415AF0057C52C4A1F7DC80298217A33291AEE",
  latest_block_height: 23,
  latest_block_time: "2019-05-03T21:41:14.581000291Z"
}
```

### Docker

In the previous section, we have built a Docker image for the node software under the name `secondstate/devchain`.

#### Single node

First, you need to initialize the configurations and settings on the node computer.

```text
$ docker run --rm -v $HOME/.devchain:/devchain secondstate/devchain:develop node init --home /devchain
```

The `genesis.json` and `config.toml` files will be created under the `$HOME/.devchain/config` directory. You can make changes to those files to customize your blockchain. You may need to `sudo su -` in order to edit those files since they are created by the root user. The libENI libraries have been built into the docker image so you don’t need to worry about it. Then, you can start the node.

```text
$ docker run --rm -v $HOME/.devchain:/devchain -p 26657:26657 -p 8545:8545 secondstate/devchain:develop node start --home /devchain
```

From a second terminal window, you can get the ID of the running Docker container.

```text
$ docker container ls
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                                                         NAMES
0bcd9da5bf05        secondstate/devchain   "./devchain node sta…"   4 minutes ago       Up 4 minutes        0.0.0.0:8545->8545/tcp, 0.0.0.0:26657->26657/tcp, 26656/tcp   pedantic_mendeleev
```

Next, log into that container.

```text
$ docker exec -i -t 0bcd9da5bf05 bash
root@0bcd9da5bf05:/app# ls
devchain  devchain.sha256  lib
```

Finally, you can attach a console to the node to run web3 commands.

```text
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

#### Multiple nodes

First, you need to initialize the configurations and settings on each of the node computer. Run the following command on each machine.

```text
$ docker run --rm -v $HOME/.devchain:/devchain secondstate/devchain:develop node init --home /devchain
```

* Each node has a different `$HOME/.devchain/config/priv_validator.json` key file. Note down the public key for each of them.
* On each node, run command `devchain node show_node_id –home $HOME/.devchain` and note down the seed for each. It is in the format of `seed@ip:26656`

Next, use [this tool](https://github.com/second-state/devchain-config) to generate a new set of `genesis.json` and `config.toml` files for the entire cluster. Enter all the public keys and seeds from the last step into the tool. For example, here is how to create a `genesis.json` with a custom `chain_id`, a custom `gas_price` and public keys from multiple nodes.

```text
$ node index.js --type genesis --config_path ./genesis.json.template --chain_id test --params.gas_price 0 --validators.1.pub_key test1 --validators.1.power 101 --validators.2.pub_key test2 --validators.1.power 102
```

Here is how to create a `config.toml` file with the seeds.

```text
$ node index.js --type config --config_path ./config.toml.template --p2p.seeds seed1@ip1:26656,seed2@ip2:26656
```

Copy the generated `genesis.json` and `config.toml` files back into each node's `$HOME/.devchain/config` directory.

Now you can start each node, and they will form a cluster.

```text
$ docker run --rm -v $HOME/.devchain:/devchain -p 26657:26657 -p 8545:8545 secondstate/devchain:develop node start --home /devchain
```

On a second terminal window on each node, you can get the ID of the running Docker container.

```text
$ docker container ls
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                                                         NAMES
0bcd9da5bf05        secondstate/devchain   "./devchain node sta…"   4 minutes ago       Up 4 minutes        0.0.0.0:8545->8545/tcp, 0.0.0.0:26657->26657/tcp, 26656/tcp   pedantic_mendeleev
```

Next, log into that container on a node.

```text
$ docker exec -i -t 0bcd9da5bf05 bash
root@0bcd9da5bf05:/app# ls
devchain  devchain.sha256  lib
```

Finally, you can attach a console to the node to run web3 commands.

```text
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



