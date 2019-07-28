---
description: Start a smart contract search engine for your own DApp
---

# Start a search engine \(Docker\)

This documentation details how you can start, and host, your own [smart contract search engine](https://github.com/second-state/smart-contract-search-engine) using [Docker](https://www.docker.com/). If you would like to build from scratch from a fresh Ubuntu install, please refer to [this document](start-a-search-engine.md).

## Prerequisite

We start from a fresh install of Ubuntu 18.04. You should first follow [the instructions here to install Docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04).

Next install the Python pip and AWS CLI utilities as follows. The AWS CLI is required to access AWS ElasticSearch services.

```bash
$ sudo apt update
$ sudo apt install python-pip
$ sudo apt  install awscli
```

## ElasticSearch

We use the AWS ElasticSearch services to run the search engine. You should [create a new ES cluster here](https://console.aws.amazon.com/es/home). For now, a single machine development cluster would suffice. In the Access Policy section, please select IAM users. You will need [an IAM user already set up](https://console.aws.amazon.com/iam/home?#/users) to access AWS ES services. Here is an example.

```text
arn:aws:iam::522901590065:user/secondstatesearch
```

Once the ElasticSearch service is up and running, you should have an ES endpoint like the following.

```text
search-smart-contract-search-engine-3paomceha6u4qzchkmbgsjdcqa.us-east-1.es.amazonaws.com
```

## Docker

Now, go back to the Ubuntu 18.04 machine.

### AWS credentials

Configure AWS CLI to access the ElasticSearch engine.

```bash
$ aws configure
```

It requires four pieces of information. The access keys are found in the [IAM user console](https://console.aws.amazon.com/iam/home?#/users) for the user you configured to access the ElasticSearch engine you just created.

```text
AWS Access Key ID [None]: [IAM user console]
AWS Secret Access Key [None]: [IAM user console]
Default region name [None]: [Region for ES instance. eg us-east-1]
Default output format [None]: json
```

After configuration, AWS config and credentials are placed in `~/.aws/`.

### Configure search engine

Next, get the source code for the search engine.

```bash
$ git clone https://github.com/second-state/smart-contract-search-engine.git
$ cd smart-contract-search-engine
```

Fill in the following configuration options.

* `ServerName` in apache config `config/site.conf`. This could be your public IP address for now.
* `blockchain`, `elasticsearch` configs in `python/config.ini`.
* `publicIp` in `js/secondStateJS.js`. This could be your IP address for now.
* Check [here](https://github.com/second-state/whitepapers/tree/2d68282b29af48f62e2075a36bd229f10fe51aa3/smart-contracts-search-engine/start-a-search-engine/README.md#javascript) for details about configurations.

### Build Docker image

```bash
$ docker build -f docker/Dockerfile -t search-engine .
```

### Run Docker container

```bash
$ docker run -d -it --rm -p 80:80 -v $HOME/.aws:/root/.aws search-engine
```

Now you can visit `http://<your_host>` to check your smart contract search engine.

## Upload an ABI

You can find the `container_id` for your docker instance on your host OS, by running

```bash
$ docker container ls
```

Next, logging into your docker container using the `container_id`

```bash
$ docker exec -it container_id bash
```

Once logged, in the `/app` directory, create a file `init_abi.py` like the following.

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

Then run

```bash
$ python3.6 init_abi.py
```

Also once all of this is done, please just exit docker and give it a reboot.

```bash
$ docker restart container_id
```

## Check progress

Now, you should be able to go to `http://<your ip address>` to check the progress of the indexing. Be patient, as it may take hours before the results show up on that page.

## Enable SSL and HTTPS

Finally, in many applications, you will need to use HTTPS to access the search engine. First, you must map the domain name, `search.domain.com`, to the public IP address of the host running docker. Then, log into docker.

```bash
$ docker exec -it container_id bash
```

Next, use Let's Encrypt services to setup SSL.

```bash
$ sudo wget https://dl.eff.org/certbot-auto -O /usr/sbin/certbot-auto
$ sudo chmod a+x /usr/sbin/certbot-auto
$ sudo certbot-auto --apache -d search.domain.com
```

Next, please open the `/etc/apache2/sites-enabled/*-ssl.conf` file \(which was created automatically by the above command\) and add the following code inside the `VirtualHost` section.

```text
<VirtualHost *:443>
    ... ...
    <Location "/">
        Header always set Access-Control-Allow-Origin "*"
        Header always set Access-Control-Allow-Methods "POST, GET, OPTIONS"
        Header always set Access-Control-Max-Age "1000"
        Header always set Access-Control-Allow-Headers "x-requested-with, Content-Type, origin, authorization, accept, client-security-token"
        RewriteEngine On
        RewriteCond %{REQUEST_METHOD} OPTIONS
        RewriteRule ^(.*)$ $1 [R=200,L]
    </Location>
</VirtualHost>
```

Replace the HTTP IP address below with your new HTTPS domain name.

* `ServerName` in apache config `config/site.conf`. 
* `publicIp` in `js/secondStateJS.js`.

Exit docker and give it a reboot.

```bash
$ docker restart container_id
```

Now, you should be able access the search engine from `https://search.domain.com` now.

