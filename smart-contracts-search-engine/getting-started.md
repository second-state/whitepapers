# Getting started

## Overview

The following image illustrates how the smart contract search engine integrates with a) a blockchain, b) a decentralized application (DApp) and any potential c) third-party application and service.

![](../.gitbook/assets/SCSE-getting_started-01.png)

The smart contract search engine can:
 * index any Ethereum compatible blockchain
 * index any number of smart contracts
 * index any number of blocks
 * be called via Curl
 * be queried via client side Javascript
 * be queried via server side NodeJS
 
## Installation

You can quickly install and use your very own smart contract search engine via [a Docker instance](https://github.com/second-state/whitepapers/blob/master/smart-contracts-search-engine/start-a-search-engine-docker.md). Alternatively, you can install and configure the smart contract search engine for use on [your own Ubuntu Server instance](https://github.com/second-state/whitepapers/blob/master/smart-contracts-search-engine/start-a-search-engine.md).

## Usage

### Configuring the smart contract search engine

All of the configuration for the harvesting is stored in [a single, config.ini, Python configuration file](https://github.com/second-state/smart-contract-search-engine/blob/master/python/config.ini). The explaination for each of the configuration settings i.e. rpc endpoint, index names etc. can be found [here](https://github.com/second-state/whitepapers/blob/master/smart-contracts-search-engine/start-a-search-engine.md#configuring-the-python-harvester-a-single-configini-file).

### Harvesting smart contract data (writing blockchain data to the indices)

The core of the smart contract search engine is [a single, harvest.py, Python file](https://github.com/second-state/smart-contract-search-engine/blob/master/python/harvest.py). All of the harvesting functionality resides in the harvest.py file. 


#### Using the harvest.py file as a library (useful for testing in certain circumstances)
Whilst technically not the recommended usage, the harvest.py file can also be utilized as a library in the following way. Firstly by starting a Python terminal.

```bash
# Bash syntax
cd /home/smart-contract-search-engine/python
python3.6
```
Then by importing the Harvest code

```python
# Python syntax
from harvest import Harvest
```
Then, the following code will automatically read all of the required configuration from the config.ini file (as the Harvest() object is instantiated as the variable `harvester`).

```python
harvester = Harvest()
```
The above code will allow any of the Harvest functions to be called individually. For example.

```python
_addressHash = "0x123...567"
harvester.getDataUsingAddressHash(_addressHash)
```

 
### Consuming smart contract data (reading blockchain data from the indices)

 * can be queried via client-side JS or server-side NodeJS
