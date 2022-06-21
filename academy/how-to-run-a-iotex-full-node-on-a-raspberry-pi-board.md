# 1. Overview

## What You'll Learn

- how to install Ubuntu on your Raspberry Pi (or compatible board)
- how to install an IoTeX full Node on the board
- how to configure and run the full node
- how to interact with the node

## What You Will Need

- an ARM board like: Raspberry 3b+ (not recommended), Raspberry 4b, Odroid N2or similar ARM boards
- a power supply compatible with your board
- a PC with an SD card reader
- a 32GB Micro SD card (consider an adapter if requred by your PC)
- an internet connection
- optionally: a USB hard-drive

# 2. Download Ubuntu for your ARM board

Before installing the full node on our ARM board, we need to install a Ubuntu OS image. Depending on the board you are using, you should download the correct OS image. Below are some examples:

## Raspberry Pi 3 & 4
See [this page](https://ubuntu.com/download/raspberry-pi) for more supported boards.

Download [Ubuntu 22.04](https://ubuntu.com/download/raspberry-pi/thank-you?version=22.04&architecture=server-arm64+raspi).

**Default login:** ubuntu

**Default password:** ubuntu

## Odroid N2

Download [Ubuntu 18.04](http://de.eu.odroid.in/ubuntu_18.04lts/N2/ubuntu-18.04.3-4.9-minimal-odroid-n2-20190806.img.xz).

**Default login:** root

**Default password:** odroid

# 3. Configure the board

## Download Etcher

[Download Link](https://www.balena.io/etcher/)

## Flash the OS image

- Insert your micro sd card in your PC card reader
- Open Etcher
- Select the OS image you downloaded
- Flash it!

## Power on your board

- Connect the board to your router with an ethernet cable
- Insert the Micro SD card in the card reader of the board
- Power on the board by connecting the power supply

## Find the IP address of your board

You have different ways to find the IP address of your board, you can [follow Ubuntu's instructions](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#4-boot-ubuntu-server), or you can find the information from your router if you have access:

- Access your router configuration page from a browser
- Locate your board among the connected devices
- Configure your router to provide a fixed IP address for your board
- Also, configure the router "Virtual server" port 4689 by redirecting it to your board IP address

If you don't know how to do that, check out this community [YouTube Video](https://youtu.be/xWD4UhGGCyo?t=246) for an example.


# 4. Configure Ubuntu

## Login into Ubuntu

In this tutorial the board's address is 192.168.1.105, while in your case it could be different.
From your PC, type the following:

For Raspberry boards:

```
ssh ubuntu@192.168.1.105
password: ubuntu
```

For the Odroid N2 board

```
ssh odroid@192.168.1.105
password: odroid
```

If requested, update your password and log in again using the new password.

# 5. Optional: configure a USB hard drive

Using a USB hard drive is highly recommended to store the Blockchain Database. You can avoid it and use the SD card instead, but adding a USB hard disk would make the system more responsive, and the sync process as well as node updates will run much faster. Also SD cards may not provide enough disk space to store the entire blockchain DB.

Create the iotex-var folder in your home directory

```
cd ~
mkdir iotex-var
```

List the current disks

```
lsblk
```

Connect your hard disk to a USB port, and list the disks again: you should see a new sdadisk with one or more partitions (sda1, sda2....) depending on the disk you connected.
Mount the partition

```
sudo mount /dev/sda1 ~/iotex-var
```

# 6. Install the IoTeX full node

## Install Docker
The preferred way to run an IoTeX full node is through a Docker image. You can install simply install Docker with:

```
curl -sSL https://get.docker.com | sh

sudo usermod -aG docker $(whoami)
```

logout and then login again to make sure all changes are applied. 

## Pull an existing Docker image
You will have to build your own Docker image of the IoTeX node **to run it on an ARM machine**. Alternatively, you can check a Docker repository provided by a trusted IoTeX community member, like this one: [the IoTeXLab public Docker Repository](https://hub.docker.com/r/iotexlab/iotex-core-arm/tags), and pull the latest image with:

```
docker pull iotexlab/iotex-core-arm:v1.8.1-rc0
```

## Build your Docker image
You can also build a Docker image directly on your system ([full instructions](https://github.com/iotexproject/iotex-core#building-the-source-code)):

You will need Golang >= 1.17.3 installed:
```
wget https://dl.google.com/go/go1.17.3.linux-armv64.tar.gz
sudo tar -C /usr/local -xzf go1.17.3.linux-arm64.tar.gz
rm  go1.17.3.linux-arm64.tar.gz
nano ~/.profile
```
Scroll all the way down to the end of the file and add the following:
```
PATH=$PATH:/usr/local/go/bin
GOPATH=$HOME/golang
```

then you can build the image with
```
git clone https://github.com/iotexproject/iotex-core.git
cd iotex-core

make docker
```

It's also convenient to buid the official IoTeX command line client `ioctl` at this point, as it will become useful later to interact with the blockchain:

```
make ioctl
sudo cp bin/ioctl /usr/local/bin
```

# 7. Configure and run the node

Once you have a docker image of the full node for ARM64 systems, just follow the official IoTeX instructions to configure and run the node available at [IoTeX Bootstrap Repository](https://github.com/iotexproject/iotex-bootstrap#status).

If you are using a third party image, just replace the official IoTeX Docker repository `iotex/iotex-core` with the IoTeXLab repository `iotexlab/iotex-core-arm` (replace the release with the current one):

```
docker pull iotexlab/iotex-core-arm:v1.17.0
```

if you built your docker image then you don't need to pull, just heck out the image name/tag by typing:
```
docker image ls
```

# 8. Manage the node

You can create a few convenient scripts to manage the node (make sure you replace `iotex/iotex-core-arm` with the correct tag for your docker image and that you are using the latest version of the ):

Start the node
```
lastRelease=$(curl --silent "https://api.github.com/repos/iotexproject/iotex-core/releases/latest" | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')
docker run -d --restart on-failure --name iotex \
        -p 4689:4689 \
        -p 8080:8080 \
	      -p 14014:14014 \
        -v=$HOME/iotex-var/data:/var/data:rw \
        -v=$HOME/iotex-var/log:/var/log:rw \
        -v=$HOME/iotex-var/etc/config.yaml:/etc/iotex/config_override.yaml:ro \
        -v=$HOME/iotex-var/etc/genesis.yaml:/etc/iotex/genesis.yaml:ro \
        iotex/iotex-core-arm:$lastRelease \
        iotex-server \
        -config-path=/etc/iotex/config_override.yaml \
        -genesis-path=/etc/iotex/genesis.yaml \
	      -plugin=gateway
```

to kill a running node
```
docker kill iotex
docker rm iotex
```

to show the log (requires `jq`installed)
```
docker logs -f --since 1m iotex | jq
```

# 9. Interact with the blockchain
[iotcl](https://docs.iotex.io/get-started/iotex-wallets/command-line-client) is the officilal command line client to manage IoTeX blockchain accounts and interact with a full node. You should have built and installed it at the end of previous step:

First, configure ioctl to interact with the local node:
```
ioctl config set endpoint localhost:14014 --insecure
```

now you can use it to query or broadcast transactions to the blockachain:

**Get basic info about the blockchain**
```
ioctl bc info
```

**List the current consensus delegates**
```
ioctl node delegate
```

Check out [the official documentation](https://docs.iotex.io/get-started/iotex-wallets/command-line-client) for a full ioctl tutorial and the list of commands.
