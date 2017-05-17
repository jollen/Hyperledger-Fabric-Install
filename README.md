The step-by-step hangout to guide installing pre-requisites and setting up the Hyperledger Fabric 1.0 enviornment.

# The Hyperledger InstallFest Guide

## Environment

Ubuntu 16.04 AWS instance.

## Install GO 1.7+ and Node.js 6.x

Install GO:

```
$ sudo curl -O https://storage.googleapis.com/golang/go1.8.linux-amd64.tar.gz
$ tar zxvf go1.8.linux-amd64.tar.gz
$ sudo mv go /usr/local
```

Put these three lines to ```~/.profile```:

```
$ export GOROOT=/usr/local/go 
$ export GOPATH=$HOME/go
$ export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

Install Node.js:

```
$ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
$ sudo apt-get install -y nodejs
```

## Install Docker CE, 

Go through the documents first:

* https://docs.docker.com/engine/installation/linux/ubuntu/#install-using-the-repository
* https://docs.docker.com/engine/installation/linux/linux-postinstall/#manage-docker-as-a-non-root-user
* https://docs.docker.com/compose/install/

Install Docker CE:

```
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88    
```

```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

```
$ sudo apt-get update
$ sudo apt-get install docker-ce
```

Config the Docker envorinment as a non-root user by adding a new ```docker``` group, and put the logined user to this group. **You have to log out and log in again**:

```
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
```

Install Docker Compose:

```
# curl -L https://github.com/docker/compose/releases/download/1.13.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# chmod a+x /usr/local/bin/docker-compose 
```

## Install Fabric 1.0

Go through the documents first:

* https://domsteil.com/2017/04/22/how-to-setup-hyperledger-fabric-v1-0-on-aws/
* https://wiki.hyperledger.org/groups/twgc/fabric-doc/getting_started.md

```
$ sudo apt-get update
$ sudo apt install libtool libltdl-dev git
```

```
$ cd <yourgodirectoryname>
$ mkdir src
$ cd src
$ mkdir github.com
$ cd github.com
$ mkdir hyperledger
$ cd hyperledger
```

```
$ git clone https://github.com/hyperledger/fabric.git
$ cd fabric
$ make configtxgen
```

At the end of ```make```, you can see:

```
binary available as build/bin/configtxgen``
```

Subsequently, you can download **all** docker images by ```e2e_cli``` example:

```
cd examples
cd e2e_cli
chmod +x download-dockerimages.sh
./download-dockerimages.sh
```

```
$ docker images
REPOSITORY                     TAG                  IMAGE ID            CREATED             SIZE
hyperledger/fabric-couchdb     x86_64-1.0.0-alpha   f3ce31e25872        2 months ago        1.51 GB
hyperledger/fabric-kafka       x86_64-1.0.0-alpha   589dad0b93fc        2 months ago        1.3 GB
hyperledger/fabric-zookeeper   x86_64-1.0.0-alpha   9a51f5be29c1        2 months ago        1.31 GB
hyperledger/fabric-orderer     x86_64-1.0.0-alpha   5685fd77ab7c        2 months ago        182 MB
hyperledger/fabric-peer        x86_64-1.0.0-alpha   784c5d41ac1d        2 months ago        184 MB
hyperledger/fabric-javaenv     x86_64-1.0.0-alpha   a08f85d8f0a9        2 months ago        1.42 GB
hyperledger/fabric-ccenv       x86_64-1.0.0-alpha   91792014b61f        2 months ago        1.29 GB
```

## Create Custom Docker Compose

Simply create ```docker-compose-peers.yaml```, and start the Fabric peers:

```
docker-compose -f docker-compose-peers.yaml up -d
```

Now, you can try Marbles v2.0 demo.

# Use Marbles Demo

Download Marbles v2.0:

```
$ sudo npm install gulp -g
$ git clone http://gopkg.in/ibm-blockchain/marbles.v2
$ cd marbles.v2
$ npm install
```

Inspect peer IP address:

```
$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' vp0
```

Edit ```mycreds_docker_compose.json ```:

```
{
	"credentials": {
		"peers": [{
			"api_host": "172.17.0.2",
			"api_port": 7050,
			"type": "peer",
			"id": "ubuntu_vp0_1"
		}],
		"users": [
		]
	}
}
```

```
$ gulp
```

Use a browser to open ```http://localhost:3000```

# Create Orderer Service

## Generate crypto-config

Use ```samples/e2e```:

```
$ cd ~/e2e
$ $GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/bin/cryptogen generate --config=./crypto-config.yaml
```

The certs are then parked into a ```crypto-config``` folder that is generated when you run the tool.

## Generate orderer.block

Generate the orderer genesis block:

```
$ export FABRIC_CFG_PATH=$PWD
$ $GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/bin/configtxgen -profile TwoOrgs -outputBlock orderer.block
```

## Generate channel.tx

Generate the channel configuration artifact ```channel.tx```:

```
$ $GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/bin/configtxgen -profile TwoOrgs -outputCreateChannelTx channel.tx -channelID mychannel
```

# Next

Go through the ```learn-chaincode```:

* learn-chaincode, https://github.com/IBM-Blockchain/learn-chaincode.git

# Where to Join

* [Hyperledger Taipei: Chaincode Night](https://www.meetup.com/Hyperledger-Taipei/events/239564790/)

