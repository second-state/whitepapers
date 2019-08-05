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

**publicIp** If running this in global mode, please make sure that the `var publicIp = "";` in the [secondStateJS.js file](https://github.com/second-state/whitepapers/tree/47778a98d431c2d32819ec8b263d8094a73bf390/js/secondStateJS.js) is set to the public domain name of the server which is hosting the search engine \(including the protocol\) i.e.

```text
var publicIp = "https://www.search-engine.com"; //No trailing slash please
```

**searchEngineNetwork in secondStateJS.js** Please ensure that the correct network id is set in the "searchEngineNetwork" variable in the secondStateJS.js file i.e.

```text
var searchEngineNetwork = "18"; // CyberMiles MainNet
```

**esIndexName name in secondStateJS.js** Please ensure that the appropriate index name will be set \(depending on which network you selected in the previous step\) The logic is as follows.

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

#### Configuring the Python harvester (a single config.ini file)

It is important that the search engine is pointing to the correct RPC endpoint i.e. CMT TestNet vs MainNet. It is also important that you set the average block time (this will make the system run more efficiently).

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
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m tx >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m abi >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m state >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m bytecode >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m full >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m topup >/dev/null 2>&1 &
    cd /var/www/search-engine.com/html/python && ulimit -n 10000 && nohup /usr/bin/python3.6 harvest.py -m indexed >/dev/null 2>&1 &
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

## Uploading your first ABI

The system requires that at least one ABI is uploaded before the harvesting is started. Please perform the following Python code from within the `/var/www/ethereum.search.secondstate.io/html/python` directory \(the same directory as the harvest.py file\). Please note, you will need your [ABI to be stored in a file as raw text like this](https://raw.githubusercontent.com/tpmccallum/test_endpoint2/master/erc20abi.txt).

```python
import re
import json
import time
import requests
from harvest import Harvest

harvester = Harvest()

abiUrl1 = "http://A_raw_text_file_which_contains_only_an_abi's_text"
abiData1 = requests.get(abiUrl1).content
abiData1JSON = json.loads(abiData1)
theDeterministicHash1 = harvester.shaAnAbi(abiData1JSON)
cleanedAndOrderedAbiText1 = harvester.cleanAndConvertAbiToText(abiData1JSON)

data1 = {}
data1['indexInProgress'] = "false"
data1['epochOfLastUpdate'] = int(time.time())
data1['abi'] = cleanedAndOrderedAbiText1
harvester.es.index(index=harvester.abiIndex, id=theDeterministicHash1, body=data1)
```

Put the above content into a file called `init_abi.py` and then run

```text
python3.6 init_abi.py
```

Also once all of this is done, please just give the server a quick reboot; during this time all of the processes will fire off as per the cron etc.

```text
sudo shutdown -r now
```

## Talking to the smart contract search engine from your DApp

The [es-ss.js](https://github.com/second-state/es-ss.js) data services library provides a simple way for your DApp to talk to the data \(using native Javascript\(client side\) and/or Node\(server side\)\) which is indexed in this system.

