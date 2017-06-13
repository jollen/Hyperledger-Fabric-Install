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
export GOROOT=/usr/local/go 
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

You should source the profile by ```$ . ~/.profile``` or logout/login again. Then, create the GOPATH directory:

```
$ mkdir -p $GOPATH
```

Next, install Node.js v6.x:

```
$ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
$ sudo apt-get install -y nodejs
```

## Install Docker CE, 

It's recommended that go through the documents first:

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

Also, it is recommended that go through the documents first:

* https://domsteil.com/2017/04/22/how-to-setup-hyperledger-fabric-v1-0-on-aws/
* https://wiki.hyperledger.org/groups/twgc/fabric-doc/getting_started.md

The command sequence to install Hyperledger Fabric v1.0.0-beta:

```
$ sudo apt-get update
$ sudo apt install libtool libltdl-dev git
```

```
$ cd $GOPATH
$ mkdir src
$ cd src
$ mkdir github.com
$ cd github.com
$ mkdir hyperledger
$ cd hyperledger
```

Download and compile Hyperledger Fabric. You may need to install *make* by ```$ sudo apt-get install make ``` in prior to compile the source:

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

There are the images:

```
$ docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
hyperledger/fabric-tools       latest              ae6b0f53cb70        4 days ago          1.32 GB
hyperledger/fabric-tools       x86_64-1.0.0-beta   ae6b0f53cb70        4 days ago          1.32 GB
hyperledger/fabric-couchdb     latest              31bbbec3d853        4 days ago          1.48 GB
hyperledger/fabric-couchdb     x86_64-1.0.0-beta   31bbbec3d853        4 days ago          1.48 GB
hyperledger/fabric-kafka       latest              c4ac1c9a4797        4 days ago          1.3 GB
hyperledger/fabric-kafka       x86_64-1.0.0-beta   c4ac1c9a4797        4 days ago          1.3 GB
hyperledger/fabric-zookeeper   latest              2c4ebacb6f00        4 days ago          1.31 GB
hyperledger/fabric-zookeeper   x86_64-1.0.0-beta   2c4ebacb6f00        4 days ago          1.31 GB
hyperledger/fabric-orderer     latest              11ff350dd297        4 days ago          179 MB
hyperledger/fabric-orderer     x86_64-1.0.0-beta   11ff350dd297        4 days ago          179 MB
hyperledger/fabric-peer        latest              e01c2b645f11        4 days ago          182 MB
hyperledger/fabric-peer        x86_64-1.0.0-beta   e01c2b645f11        4 days ago          182 MB
hyperledger/fabric-javaenv     latest              61c188dca542        4 days ago          1.42 GB
hyperledger/fabric-javaenv     x86_64-1.0.0-beta   61c188dca542        4 days ago          1.42 GB
hyperledger/fabric-ccenv       latest              7034cca1918d        4 days ago          1.29 GB
hyperledger/fabric-ccenv       x86_64-1.0.0-beta   7034cca1918d        4 days ago          1.29 GB
hyperledger/fabric-ca          latest              e549e8c53c2e        4 days ago          238 MB
hyperledger/fabric-ca          x86_64-1.0.0-beta   e549e8c53c2e        4 days ago          238 MB
```

Or, pull the *fabric-peer* and *fabric-baseimage* needed for the Marble application (described in the following section).

```
$ docker pull hyperledger/fabric-peer:latest
$ docker pull hyperledger/fabric-baseimage:x86_64-0.2.2
$ docker tag \
hyperledger/fabric-baseimage:x86_64-0.2.2 \
hyperledger/fabric-baseimage:latest
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
			"id": "vp0"
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

