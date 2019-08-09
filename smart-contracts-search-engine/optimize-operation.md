# Optimizing your smart contract search engine's operation

There are many unique use cases for the smart contract search engine. For example, your business may want to deploy a single smart contract which contains all of the logic and data for your DApp. Alteratively, your business may be creating a smart contract template and then allowing end-users to deploy unlimited instances of that smart contract, from within your DApp. Perhaps you may be deploying your DApp on a blokchain with 15 second block intervals, or alternatively you may be deploying it on a blockchain with 1 second intervals. 

## Real-time data

The ultimate goal of the smart contract search engine is to provide your DApp with reliable real-time data. Food for thoughs. If you are indexing thousands of contract instances and have a dozen seperate smart contract ABIs (on a chain with 15 second block intervals) you will require a different optimization than someone else who is indexing only one smart contract ABI on a chain with 1 second block intervals. Similarly, if your enterprise blockchain has a current block height of 10 thousand you may consider a different optimization than if you were say wanting to index the entire Ethereum MainNet which has over 8 million blocks and more than 1 million deployed smart contracts.

## Optimization evaluation

SecondState provides a handy demonstration which assists you in evaluating your usage of the smart contract search engine. The demonstration can be run from inside the [SecondState BUIDL tool](http://buidl.secondstate.io/). The demonstration allows you to manually deploy and then interact with a series of pre-written smart contracts. The SecondState BUIDL tool allows you to set the blockchain RPC endpoint and smart contract search engine endpoint in its providers section. This will facilitate a pragmatic evaluation of the smart contract search engine; evaluating response behaviour in relation to your blockchain's native operation.

## Running the optimization

### Configure the SecondState BUIDL tool

Firstly open the[SecondState BUIDL tool](http://buidl.secondstate.io/). Then go to the DApp tab and paste in [this Javascript](https://raw.githubusercontent.com/second-state/buidl/master/demo/search-engine/demo.js) and [this HTML](https://raw.githubusercontent.com/second-state/buidl/master/demo/search-engine/demo.html).
Open the contract tab and paste in [this smart contract code](https://raw.githubusercontent.com/second-state/buidl/master/demo/search-engine/demo.sol).

From here you will need to configure the provider settings. Specifically pasting in the URL of your smart contract search engine instance and the RPC endpoint of your blockchain. Please note that you can use [SecondState's Blockchain as a Service](http://baas-mvp.secondstate.io/) (BaaS) to create both a new blockchain and a search engine. It is free and is achieved with a few clicks in as little as 5 minutes. Alternatively you can [install your own smart contract search engine](https://docs.secondstate.io/smart-contracts-search-engine/getting-started#installation) and use this in the SecondState BUIDL tool's provider section.


