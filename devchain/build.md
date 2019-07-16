# Build

This document describes how to build the Second State DevChain.

### Binary build

Binary builds of the Second State DevChain are limited to Ubuntu 16.04 and CentOS 7.

#### Ubuntu 16.04

First, let’s install and update necessary packages on a clean Ubuntu 16.04 install.

```text
$ sudo apt update -y
$ sudo apt install -y curl wget git bison build-essential
```

Next, you must have GO language version 1.10+ installed. The easiest way to get GO 1.10 is through the GVM. Below are the commands.

```text
$ bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
$ gvm install go1.10.3 -B
$ gvm use go1.10.3 --default
```

Now, pull the devchain source code from Github, and then build the binary executable for Ubuntu 16.04.

```text
$ go get github.com/second-state/devchain
$ cd $GOPATH/src/github.com/second-state/devchain
$ make all
```

Once successful, the binary executable from the build is `$GOPATH/bin/devchain`.

```text
$ which devchain
/home/ubuntu/.gvm/pkgsets/go1.10.3/global/bin/devchain
```

#### CentOS 7

TBD

### Docker build

First, install Docker.

```text
$ sudo apt install docker.io
$ sudo usermod -a -G docker $USER
```

Log out and then log back in. Clone the repository from Github to your build machine.

```text
$ git clone https://github.com/second-state/devchain.git
$ cd devchain
```

Next, build a Docker image for the build environment.

```text
$ docker build -t secondstate/devchain-build -f Dockerfile.build.ubuntu .
...
Successfully tagged secondstate/devchain-build:latest
```

Finally, you can build the Docker image of the node.

```text
$ docker build -t secondstate/devchain .
...
Successfully tagged secondstate/devchain:latest
```

You can now see the Docker image you just built.

```text
$ docker image ls
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
secondstate/devchain         latest              c800759441b6        5 minutes ago       246MB
secondstate/devchain-build   latest              364c7a4700b9        4 hours ago         732MB
ubuntu                        16.04               a3551444fc85        7 days ago          119MB
```



