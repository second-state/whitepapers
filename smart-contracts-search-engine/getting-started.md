# Getting started

## Overview

The following image illustrates how the smart contract search engine integrates with a) a blockchain, b) a decentralized application (DApp) and any potential c) third-party application and service.

![](../.gitbook/assets/SCSE-getting_started-01.png)

The smart contract search engine can:
 * index any Ethereum compatible blockchain
 * index any number of smart contracts
 * index any number of blocks
 
## Installation

You can quickly install and use your very own smart contract search engine via [a Docker instance](https://github.com/second-state/whitepapers/blob/master/smart-contracts-search-engine/start-a-search-engine-docker.md). Alternatively, you can install and configure the smart contract search engine for use on [your own Ubuntu Server instance](https://github.com/second-state/whitepapers/blob/master/smart-contracts-search-engine/start-a-search-engine.md).

## Usage

### Configuring the smart contract search engine

All of the configuration for the harvesting is stored in [a single, config.ini, Python configuration file](https://github.com/second-state/smart-contract-search-engine/blob/master/python/config.ini).

### Harvesting smart contract data (writing blockchain data to the indices)

The core of the smart contract search engine is [a single, harvest.py, Python file](https://github.com/second-state/smart-contract-search-engine/blob/master/python/harvest.py). All of the harvesting functionality resides in the harvest.py file. 

The file can be utilized as a library in the following way.
```bash
# Bash syntax
cd /home/smart-contract-search-engine/python
python3.6
```
```python
# Python syntax
from harvest import Harvest
```
harvester = Harvest()



 
### Consuming smart contract data (reading blockchain data from the indices)

 * can be queried via client-side JS or server-side NodeJS
