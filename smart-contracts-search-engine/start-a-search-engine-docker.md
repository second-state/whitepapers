# Start a search engine \(Docker\)

This documentation details how you can start, and host, your own [smart contract search engine](https://github.com/second-state/smart-contract-search-engine) using [Docker](https://www.docker.com/).

## awscli

Install and configure [awscli](https://github.com/aws/aws-cli):

```bash
pip install awscli
aws configure
```

After configuration, awscli config and credentials are placed in `~/.aws/`.

## Get source

```bash
git clone https://github.com/second-state/smart-contract-search-engine.git
cd smart-contract-search-engine
```

## Configure search engine

- `ServerName` in apache config `config/site.conf`.
- `blockchain`, `elasticsearch` configs in `python/config.ini`.
- `publicIp` in `js/secondStateJS.js`.
- Check [here](./start-a-search-engine#javascript) for details about configurations.

## Build Docker image

```bash
docker build -f docker/Dockerfile -t search-engine .
```

## Run Docker container

```bash
docker run -it --rm -p 80:80 -v $HOME/.aws:/root/.aws search-engine
```

Now you can visit `http://<your_host>` to check your smart contract search engine.
