# Start a search engine \(Ubuntu\)

This documentation details how you can start, and host, your own [smart contract search engine](https://github.com/second-state/smart-contract-search-engine) using Ubuntu 18.04LTS.

### Apache 2

Update system and install Apache2

```bash
sudo apt-get update
sudo apt-get -y upgrade
sudo apt-get -y install apache2
```

List the firewall rules and add Apache2 on port 80 only

```text
sudo ufw app list
sudo ufw allow 'Apache'
```

Check that Apache is running

```text
sudo systemctl status apache2
```

Enable modules for the proxy

```bash
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo systemctl restart apache2
```

Get the domain name \(as discussed at the start of this page\) and use it in the following steps i.e. search-engine.com or 13.236.179.58 \(just the IP without the protocol\). In this example, we are just using fictitious search-engine.com

Set up Virtual Host

```text
sudo mkdir -p /var/www/search-engine.com/html
sudo chown -R $USER:$USER /var/www/search-engine.com/html
sudo chmod -R 755 /var/www/search-engine.com
```

Create the following configuration file

```text
sudo vi /etc/apache2/sites-available/search-engine.com.conf
```

Add the following content to the file which you just created \(note: we will explain the Proxy component a little later in this document\). Obviously you will need to replace search-engine.com with your public IP/Domain

```text
<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass /api http://127.0.0.1:8080/api
    ProxyPassReverse /api http://127.0.0.1:8080/api
    ServerAdmin admin@search-engine.com
    ServerName search-engine.com
    ServerAlias www.search-engine.com
    DocumentRoot /var/www/search-engine.com/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the new site

```text
sudo a2ensite search-engine.com.conf
```

Disable the original Apache2 site

```text
sudo a2dissite 000-default.conf
```

Test the configuration which we just created

```text
sudo apache2ctl configtest
```

Reload the system to show the new site

```text
sudo systemctl reload apache2
```

Here is a quick reference of commands which you will find usefull in the future

```text
## Stop and start
sudo systemctl stop apache2
sudo systemctl start apache2
## Restart
sudo systemctl restart apache2
## Reload without interuption
sudo systemctl reload apache2
```

### Search engine source code

```bash
cd ~
git clone https://github.com/second-state/smart-contract-search-engine.git
```

Place the code in the appropriate directories

```bash
cp -rp ~/smart-contract-search-engine/* /var/www/search-engine.com/html/
```

Set final permissions on all files

```bash
sudo chown -R $USER:$USER /var/www/search-engine.com/*
```

#### Javascript

This system uses a single Javascript file which passes events and data back and forth between the HTML and Python. The code repository currently has one Javascript file `secondStateJS.js` which services the [FairPlay - Product Giveaway site](https://cmt.search.secondstate.io/) and one Javascript file `ethJS.js` which services the [Ethereum Search Engine Demonstration](https://ethereum.search.secondstate.io/). One of the strong points of this search engine is that it allows you to create your own custom HTML/JS so that you can render your data in any way.

##### publicIp
**publicIp** If running this in global mode, please make sure that the `var publicIp = "";` in the [secondStateJS.js file](https://github.com/second-state/whitepapers/tree/47778a98d431c2d32819ec8b263d8094a73bf390/js/secondStateJS.js) is set to the public domain name of the server which is hosting the search engine \(including the protocol\) i.e.

```text
var publicIp = "https://www.search-engine.com"; //No trailing slash please
```

##### searchEngineNetwork
**searchEngineNetwork** in secondStateJS.js Please ensure that the correct network id is set in the "searchEngineNetwork" variable in the secondStateJS.js file i.e.

```text
var searchEngineNetwork = "18"; // CyberMiles MainNet
```

##### esIndexName
**esIndexName** name in secondStateJS.js Please ensure that the appropriate index name will be set \(depending on which network you selected in the previous step\) The logic is as follows.

```text
if (searchEngineNetwork == "19") {
    blockExplorer = "https://testnet.cmttracking.io/";
    esIndexName = "testnet";
}

if (searchEngineNetwork == "18") {
    blockExplorer = "https://www.cmttracking.io/";
    esIndexName = "cmtmainnetmultiabi";
}
```

Just please make sure that you set the esIndexName to the same value as the config.ini \(i.e. note how the below config.ini common index and the above secondStateJS.js esIndexName are both set to testnet\).

**This Javascript configuration will be made part of the global configuration as per the GitHub Issue**

```text
[commonindex]
network = testnet
```

```text
if (searchEngineNetwork == "19") {
    blockExplorer = "https://testnet.cmttracking.io/";
    esIndexName = "testnet";
}

if (searchEngineNetwork == "18") {
    blockExplorer = "https://www.cmttracking.io/";
    esIndexName = "cmtmainnetmultiabi";
}
```

#### Configuring the Python harvester \(a single config.ini file\)

It is important that the search engine is pointing to the correct RPC endpoint i.e. CMT TestNet vs MainNet. It is also important that you set the average block time \(this will make the system run more efficiently\).

```text
[blockchain]
rpc = https://testnet-rpc.cybermiles.io:8545
seconds_per_block = 1
```

**Elasticsearch** Please also put in your Elasticsearch URL and region.

```text
[elasticSearch]
endpoint = search-smart-contract-search-engine-abcdefg.es.amazonaws.com
aws_region = ap-southeast-2
```

**Index names** The masterindex, abiindex and bytecode index can all stay as they are below. You might just want to change the commonindex to be more descriptive i.e. mainnet, testnet etc.

```text
# Stores every transaction in the blockchain which has a contractAddress (an instantiation of a contract as apposed to a transaction which just moved funds from one EOA to another)
[masterindex]
all = all

# Just stores ABI data (an index which holds every ABI that could match up with a contract address and make up a contract instantiation)
[abiindex]
abi = abi

# Just stores a contracts bytecode with the appropriate key to find said bytecode
[bytecodeindex]
bytecode = bytecode

# Stores all of the smart contract instance details, ABI hashes, function data etc.
[commonindex]
network = network

# Ignore stores ABI and contract address Sha3 values which are not contract instantiations. This improves performance greatly because an ABI hash mixed with a contract address hash will either be a contract instance or not and this will never change once set.
[ignoreindex]
ignore = ignore

# The default number of threads is set to 500. However, this value can be raised if you also raise the ulimit when starting the harvest.py scripts i.e. adding ulimit -n 1000 will facilitate a setting here of max_threads = 1000
[system]
max_threads = 500

# Please provide a raw GitHub URL which contains your first ABI. This is required for the search engine to initialize
[abi_code]
initial_abi_url = https://raw.githubusercontent.com/tpmccallum/test_endpoint2/master/erc20_transfer_function_only_abi.txt
```

### SSL \(HTTPS\) using "lets encrypt"

```text
sudo wget https://dl.eff.org/certbot-auto -O /usr/sbin/certbot-auto
```

```text
sudo chmod a+x /usr/sbin/certbot-auto
```

```text
sudo certbot-auto --apache -d search-engine.com  -d www.search-engine.com
```

### Harvesting

Please follow the instructions below so that your system can automatically execute all of the search engine's scripts.

### Operating system libraries

Python3

```bash
sudo apt-get -y update
sudo apt-get -y upgrade

# If using Ubuntu 18.04LTS Python 3.6 will already be installed

# If using older Ubuntu, you will need to install Python3.6 and Python3.6-dev 
#sudo add-apt-repository ppa:jonathonf/python-3.6
#sudo apt-get -y update
#sudo apt-get install python3.6-dev
```

Pip3

```bash
sudo apt-get -y install python3-pip
```

Web3

```text
python3.6 -m pip install web3 --user
```

Boto3

```bash
python3.6 -m pip install boto3 --user
```

AWS Requests Auth

Note: this particular implementation of the smart contract search engine uses AWS Elasticsearch and as such there is a small amount of Amazon and Elasticsearch specific configuration. You can choose to use your own local installation of Elasticsearch if you prefer that.

```text
python3.6 -m pip install aws_requests_auth --user
```

AWS Command Line Interface \(CLI\)

```text
sudo apt-get install awscli
```

Configuring AWS CLI See [official AWS documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) as required

```text
aws configure
```

Elasticsearch

```text
python3.6 -m pip install elasticsearch --user
```

### Elasticsearch

**AWS provides Elasticsearch as a service. To set up an AWS Elasticsearch instance visit your AWS console using the following URL.**

```text
https://console.aws.amazon.com/console/home
```

### Amazon Web Services \(AWS\)

**Authentication and access control**

Please read the [Amazon Elasticsearch Service Access Control](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-ac.html) documentation. This very flexible authentication and access control can be set up after the fact by writing a policy.

### Recommended usage - Run once at startup!

**Run at startup**

Technically speaking you will just want to run all of these commands the **one** time, at startup!. The system will take care of itself. Here is an example of how to run this once at startup.

Create a bash file, say, `~/startup.sh` and make it executable with the `chmod a+x` command. Then put the following code in the file. **Please** be sure to replace `https://testnet-rpc.cybermiles.io:8545` with that of your RPC.

```bash
#!/bin/bash
while true
do
  STATUS=$(curl --max-time 30 -s -o /dev/null -w '%{http_code}' https://YOUR RPC NODE GOES HERE)
  if [ $STATUS -eq 200 ]; then
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m init >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m abi >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m full >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m topup >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m tx >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m state >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m bytecode >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m indexed >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m faster_state >/dev/null 2>&1 &
    break
  else
    echo "Got $STATUS please wait"
  fi
  sleep 10
done
```

Add the following command to cron using `crontab -e` command.

```bash
@reboot ~/startup.sh
```

The smart contract search engine will autonomously harvest upon bootup.

### Detailed explanation of harvesting modes

#### Full

The `harvest.py -m full` mode operates in the following way. It divides the number of blocks in the blockchain by the `max_threads` setting in the [config.ini](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L27) file, to create chunks of blocks. It then starts a separate thread for each of those chunks. Each chunk is harvested in parallel. For example, if the blockchain has 1 million blocks and the `max_threads` value is 500, there will be 500 individual threads processing 2, 000 blocks each.

The `harvest.py -m full` mode quickly and efficiently traverses the entire blockchain with its sole purpose being to find transactions which involve smart contracts. Transactions which involve smart contract creation i.e. have a contract address in the transaction receipt are stored in the smart contract search engine's [masterindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L9).

#### Topup

The `harvest.py -m topup` mode operates in the following way. It uses the following formular to determine how many of the most recent blocks to harvest.

```python
stopAtBlock = latestBlockNumber - math.floor(100 / int(self.secondsPerBlock))
```

For example if the blockchain has 1 million blocks and the `seconds_per_block` [in the config.ini](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L3) is set to 10, the system will process the most recent block `1000000` and stop at block `999990` \(harvest only the 10 most recent blocks\). Once executed, this topup will run repeatedly i.e. it does not have to be run using cron because it already uses Python `time.sleep` and will repeat as required.

The `harvest.py -m topup` mode quickly and efficiently traverses only a few of the latest blocks with its sole purpose being to find only the most recent transactions which involve smart contracts. These are stored in the smart contract search engine's [masterindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L9).

Both of the above modes have only identified \(and saved to the masterindex\) transactions which involve the creation of smart contracts. None of this data is searchable via the API. These full and topup modes are run at all times as they provide the definitive list of transactions which the search engine has to process on an ongoing basis.

#### Transaction \(tx\)

The `harvest.py -m tx` mode operates in the following way. It takes all of the known ABIs which are stored in the [abiindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L15) and all of the known smart contract related transactions which are stored in the smart contract search engine's [masterindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L9). It then creates a web3 smart contract instantiation for every combination and tests to see if web3 can sucessfully call all of the contract's public view functions.

If the contract address is _not_ already in the commonindex, and the contract instance returns valid data for all of the public/view functions defined in the ABI, then a new entry is created in the [commonindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L12).

If the contract address is already indexed in the commonindex, there is still a chance that this particular ABI and smart contract combination is new. Therefore the code will go ahead and try to instantiate a web3 contract instance with the ABI and address at hand. If the public view functions of the contract instance are all returned perfectly then the code will assume that this is an associated ABI i.e. a valid ABI which is part of Solidity inheritance etc. The outcome of this process will include the abiShaList being updated as well as the functionDataList seeing the addition of the new data.

If the contract instance is unable to return valid data for each of the public view functions of the ABI in question, then it is assumed that the contract address and the ABI were never related. This combination goes into the [ignoreindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L21) because this will never change.

#### Faster State

The `harvest.py -m faster_state` mode operates in the following way. It traverses only the most recent blocks, calls the public/view functions of the contracts in those blocks and updates the [commonindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L12). Remembering that the commonindex is the index which provides the smart contract state data to the API.

_Note_ This `-m faster_state` mode requires that the smart contract instantiation \(both the transaction hash and all associated ABIs\) is already known to the smart contract search engine indices. This requires work. This mode was created for a special case whereby the search engine was required to provide real-time data for a blockchain with 1 second block intervals. Part of this special use case required that the software which was responsible for instantiating new contracts explicitly indexed the contract's ABIs and also the transaction hash of the contract instantiation. This was achieved via the [submitManyAbis](https://github.com/second-state/es-ss.js#submit-many-abis-in-conjuntion-with-a-single-tx-hash-for-indexing) API call.

This mode provides the fastest data updates available. However, as mentioned above, it also needs to have each contract's ABIs and transaction hash to be purposely indexed asap. This mode is not about self discovery, but rather about explicit indexing in real-time. The system can perform self discovery but the self discovery process \(testing combinations of many ABIs and many contract addresses i.e. millions of combinations\) takes longer than 1 second.

#### ABI

The `harvest.py -m abi` mode operates in the following way. It fetches the already indexed records from the [commonindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L12) and also fetches all of the already indexed ABIs from the [abiindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L15). It then creates web3 contract instantiations for each of the combinations. Then in relation to each combination, if the contract instance is unable to return the public view function data perfectly then the ABI and address combiination is added to the ignoreindex. This prevents the abi mode from ever checking that particular ombination out again. On the other hand, if the public view functions of the contract instance are all returned perfectly, the code will assume that this is an associated ABI i.e. a valid ABI which is part of Solidity inheritance etc. The outcome of this process will include the abiShaList being updated as well as the functionDataList seeing the addition of the new data.

The primary purpose of the ABI mode is to introduce newly uploaded ABIs to pre-existing contract addresses of which they may be associated with.

#### State

The `harvest.py -m state` mode operates in the following way. It fetches all indexed contracts from the [commonindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L12). It reads their abiShaList and creates a web3 contract instance for each of the ABI / address combinations. Now the ABI address combinations are not questionable because they have already been through a process of making sure that they are a real relationship which can yield real data. The state mode fetches the public view data from the contract and updates the index if the data is different to what was originally stored. It also creates a local hash of the data so that when it repeats this process over and over it can compare hashes on local disk rather than remotely issuing queries to the index.

#### Bytecode

The `harvest.py -m bytecode` mode operates in the following way. It fetches all indexed contracts and matches their bytecode \(which is in theie transaction instance input\) with any individual bytecode entries in the [bytecodeindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L18)

#### Indexed

The `harvest.py -m indexed` mode operates in the following way. It loops through all of the indexed contracts from the [commonindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L12) and sets the `indexed` value of any item in the [masterindex](https://github.com/second-state/smart-contract-search-engine/blob/45ea54d0217fff0973a40f95c688ac03eedc2e1c/python/config.ini#L9) to `true` if the contract addresses from the two indices match. This is an independent script which does not have to be run, however it does help speed up performance slightly by excluding any already indexed items when the tx mode is executed.

### Flask

```text
python3.6 -m pip install Flask --user
```

### Python Flask / Apache2 Integration

```bash
sudo ufw allow ssh
sudo ufw enable
sudo ufw allow 8080/tcp
sudo ufw allow 443/tcp
```

Open crontab for editing

```bash
crontab -e
```

Add the following line inside crontab

```bash
@reboot sudo ufw enable
@reboot cd /var/www/search-engine.com/html/python && nohup /usr/bin/python3.6 io.py >/dev/null 2>&1 &
```

### CORS \(Allowing Javascript, from anywhere, to access the API\)

To enable CORS please following these instructions.

Ensure that Apache2 has the mod\_rewrite enabled

```text
sudo a2enmod rewrite
```

Ensure that Apache2 has the headers library enabled by typing the following command.

```text
sudo a2enmod headers
```

Open the `/etc/apache2/apache2.conf` file and add the following.

```text
<Directory /var/www/search-engine>
     Order Allow,Deny
     Allow from all
     AllowOverride all
     Header set Access-Control-Allow-Origin "*"
</Directory>
```

Then in addition to this, please open the `/etc/apache2/sites-enabled/search-engine-le-ssl.conf` file \(which was created automatically by the above "lets encrypt" command\) and add the following code inside the `VirtualHost` section.

```text
Header always set Access-Control-Allow-Origin "*"
Header always set Access-Control-Allow-Methods "POST, GET, OPTIONS"
Header always set Access-Control-Max-Age "1000"
Header always set Access-Control-Allow-Headers "x-requested-with, Content-Type, origin, authorization, accept, client-security-token"
RewriteEngine On
RewriteCond %{REQUEST_METHOD} OPTIONS
RewriteRule ^(.*)$ $1 [R=200,L]
```

Also once all of this is done, please just give the server a quick reboot; during this time all of the processes will fire off as per the cron etc.

```text
sudo shutdown -r now
```

## Talking to the smart contract search engine from your DApp

The [es-ss.js](https://github.com/second-state/es-ss.js) data services library provides a simple way for your DApp to talk to the data \(using native Javascript\(client side\) and/or Node\(server side\)\) which is indexed in this system.

