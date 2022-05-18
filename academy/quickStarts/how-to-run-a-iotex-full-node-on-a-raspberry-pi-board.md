---
title: How to run a IoTeX Full Node on a Raspberry Pi Board
description: Learn how to run and IoTeX full node as a network gateway on a Raspberry Pi
permalink: academy/quickStarts/how-to-run-a-iotex-full-node-on-a-raspberry-pi-board.md
---

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

## Raspberry Pi 3 b+
[Download Link](http://cdimage.ubuntu.com/releases/18.04/release/ubuntu-18.04.3-preinstalled-server-arm64+raspi3.img.xz)

**Default login:** ubuntu

**Default password:** ubuntu

## Raspberry Pi 4 b
[Download Link](https://github.com/TheRemote/Ubuntu-Server-raspi4-unofficial/releases/download/v12/ubuntu-18.04.3-preinstalled-server-arm64+raspi4.img.xz)

This is currently the only (unofficial) version of Ubuntu for the Raspberry Pi 4 b

**Default login:** ubuntu

**Default password:** ubuntu

## Odroid N2
[Download Link](http://de.eu.odroid.in/ubuntu_18.04lts/N2/ubuntu-18.04.3-4.9-minimal-odroid-n2-20190806.img.xz)

**Default login:** odroid

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
## Configure your router
If you don't know how to do that, check out this [YouTube Video](https://youtu.be/xWD4UhGGCyo?t=246) for an example.
- Access your router configuration page from a browser
- Locate your board among the connected devices
- Configure your router to provide a fixed IP address for your board
- Also, configure the router "Virtual server" port 4689 by redirecting it to your board IP address

# 4. Configure Ubuntu

## Login in Ubuntu
Please use the IP address you assigned for your board. In the example it will be 192.168.1.105, while yours could be different.
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

## Clone the IoTeXLab iotex-arm scripts
```
git clone https://github.com/IoTeXLab/iotex-arm.git
```
ifgitis not installed in your Ubuntu image, install it with:
```
sudo apt install git
```

## Set the path to the scripts folder
Edit the file.profile
```
nano .profile
```
add this line at the end of the file:
```
PATH="$PATH:$HOME/iotex-arm/bin"
```
typectrl-XandYto save, then type
```
source .profile
```
to apply the change.

# 5. Optional: configure a USB hard drive
Using a USB hard drive is recommended to store the IoTeX blockchain Data. You can avoid it, but adding a USB hard disk would make the raspberry more responsive, and the sync process/node updates will run much faster.
Create theiotex-varfolder in your home directory
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
[IoTeXLab Delegate](https://iotexlab.io/) provides, an updated **Docker image** for the latest IoTeX node release, specifically built for **ARM boards**. You can browse the available versions at the [theIoTeXLab public Docker Repository](https://hub.docker.com/r/iotexlab/iotex-core-arm/tags).

## The quick way
The IoTeXLab GitHub repository that you just cloned in the previous step, includes a convenient script that will just perform all the required operations, from installing Docker up to configuring the node for you.
To run it, just type the following:
```
setup_iotex_node
```
## The standard way
Having the Docker image for ARM, you can install the IoTeX full node by simply installing Docker:
```
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $(whoami)
```
and following the official instructions available at [IoTeX Bootstrap Repository](https://github.com/iotexproject/iotex-bootstrap#status) just replace the official IoTeX Docker repository `iotex/iotex-core` with the IoTeXLab repository `iotexlab/iotex-core-arm` (replace the release with the current one):
```
docker pull iotexlab/iotex-core-arm:v0.9.2
```
# 7. Interact with the node
The IoTeXLab scripts folder includes a few convenient scripts tools. After the node is downloaded and configured, you can run the following commands:
to show the log from the node
```
logNode
```
to kill the currently running node
```
killNode 
```
to start the node
```
startNode 
```
It also includes the latest IoTeX `ioctl` command-line  tool to interact with the IoTeX blockchain:
First, configure it to interact with our local node:
```
ioctl config set endpoint localhost:14014--insecure
```
now you can use it to query the node or execute actions:
Get basic info about the blockchain
```
ioctl bc info
```
List the current consensus delegates
```
ioctl node delegate
```


