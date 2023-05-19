# W3bstream Quick Start - Sending data from Raspberry Pi

## Introduction

W3bstream is a powerful new framework that enables developers to connect data generated in the physical world to the blockchain world. Using the IoTeX blockchain, w3bstream streams data from IoT devices and machines, and generates proofs of real-world facts that can be used by dApps on different blockchains.  

In this article, we'll walk you through how to use w3bstream to send data from a Raspberry Pi to a w3bstream node. We'll cover how to create and configure the w3bstream project in the w3bstream web interface and how to build a C++ program that uses the W3bstream IoT SDK to publish data from a Raspberry Pi. By the end of this article, you'll have the skills and knowledge you need to start building your own IoT projects on the w3bstream network.

## Creating the w3bstream Project

To start streaming data from your IoT device to the w3bstream network, you'll first need to create a new project on the w3bstream developer portal. Checkout the "how to" section of the [W3bstream documentation](https://docs.w3bstream.com/get-started/w3bstream-studio) to learn how to quickly get started initiating a project, adding devices and creating event routing strategies. 

With your w3bstream project set up, it's time to start streaming data from your IoT device.

## Sending data from Raspberry Pi

### Introduction

In this section, we will be using the w3bstream IoT SDK to publish data from a Raspberry Pi to the w3bstream network. The w3bstream IoT SDK provides an easy-to-use interface for sending data from IoT devices to the w3bstream network. This will allow us to monitor and analyze the data in real-time using the w3bstream web UI.

Before we begin, it is assumed that you have a basic understanding of the Raspberry Pi and C++ programming. Additionally, you will need to have a project set up on the w3bstream web UI. If you have not already done so, please refer to the previous section on creating a w3bstream project.

### Prerequisites: Setting up the Environment

Before we can build and run our application, we need to make sure our environment is properly set up. Here are the prerequisites:

1. Install the required system packages:

    ```bash
    sudo apt-get install -y python3-pip build-essential cmake libcurl4-openssl-dev
    ```

    These packages are necessary for building the w3bstream IoT SDK.

2. Clone the repository:

    ```bash
    git clone https://github.com/machinefi/w3bstream-iot-sdk && cd w3bstream-iot-sdk
    ```

    This will download the w3bstream IoT SDK and change directory into the project directory.

### Building the Application: Using the W3bstream IoT SDK

To build the application using the W3bstream IoT SDK, follow these steps:

1. Create a directory to store the build output:

    ```bash
    mkdir ./build-out
    ```

2. Run the following command to configure the build:

    ```bash
    cmake -DBUILD_EXAMPLE_QUICK_START_RPI=ON -DGIT_SUBMODULE_UPDATE=ON -S ./ -B ./build-out
    ```

    This will clone the required Git submodules and generate the build files in the `build-out` directory.

3. Run the following command to build the example Quick Start application:

    ```bash
    cmake --build build-out --target example-quick-start-rpi
    ```

    This will compile the source code and generate an executable in the `build-out` directory called `example-quick-start-rpi`.

### Running the Application: Sending Data to w3bstream

This section provides a detailed guide on how to run the example application on Raspberry Pi, which sends data to the w3bstream platform using the IoT SDK. We will outline how the application uses the IoT SDK to establish a connection and transmit verifiable data.

The application can be run with the following command:  

```bash
cd build-out/examples/quick-start-rpi
./example-quick-start-rpi -t "publisher_token"
```

More command line options can be passed to the application. Run the following to see all available options: 

```bash
./example-quick-start-rpi -h
```

#### Application workflow

The example application first initializes the PSA Crypto API provided by the IoT SDK. A single function call to  `psa_crypto_init()` is required for this:

```c++
psa_status_t status = psa_crypto_init();
```

The application uses an ECDSA key pair to sign the data. The user can provide the key storage using the `keystore_path` command line option. If the keystore_path is not specified, the running directory is used. If the keystore does not contain any key information, a new key is generated. If a key is already present, then that key will be used, unless the `generate_key` option is used.  
The key material is stored in the keystore directory. Two files will be used: `private.key` and `public.key`.  

The application needs to import the key into the PSA Crypto API, so it can be used for signing. The `import_key()` funtion takes care of this.  

Then the application enters a loop where it sends a message periodically to the w3bstream node. In order for the w3bstream node to verify the data is sent from our device, the message contains a signature. It also includes the public key, so the signature can ve verified in w3bstream.

We tried keeping the logic as simple as possible for this example. So the data contained in the message is just a random value and a timestamp.  

The `create_payload()` function is used to build and sign message.  

### Observing the Data: Verifying Data Reception in w3bstream

1. Verify the console output of the application in the Raspberry Pi. It should display output similar to the following:
    
    ```bash
    17:12:11: Publishing message: {"data":{"status":1804289383,"timestamp":1683907931},"signature": "00000000000000004020000100000040ffffffeaffffffbfffffffeffffffffe7f000031000000000000002c00000000000000ffffffe041300001000000006075ffffff98ffffffff7f0000400000000000000002","public_key":"04b5cdfa25aaa1e724d27ce0d928ca146d18be5f43b28b5ca1642075ae7d0007d7d777f0d1160840044e9021b09c8224ff652d7262dad2a25c39e025ee498b8dee"}
    to <http://104.198.23.192:8889/srv-applet-mgr/v0/event/eth_0x00000000000000000000000000000_quick_start>
    {"channel":"eth_0x00000000000000000000000000000_quick_start","publisherID":"9025854401981442","eventID":"cb2a7a66-bf81-47f3-8cd1-4dabcf33abad_w3b","results":[{"appletName":"9025851227716615","instanceID":"9025851227721735","handler":"start","returnValue":null,"code":1712}]}
    ```
    
2. On the w3bstream web interface, go to the Log tab. You should see an entry in the log with the message sent from the Raspberry Pi. Note that you may need to refresh the page in order for it to appear.

## Conclusions

In this project, we explored the process of setting up and running a w3bstream application on a Raspberry Pi. We also covered how to use the w3bstream web interface to create a project, register a publisher and view the data.

This article also gave an overview of how to use IoT SDK to build a C++ application that sends data to w3bstream. We also showed how to verify the successful reception of data by monitoring the console output on the Raspberry Pi and checking the logs on the w3bstream web interface.

Overall, this project provides a solid foundation for anyone interested in developing and deploying w3bstream applications on Raspberry Pi devices. By following the steps outlined in this guide, users can easily set up and run their own w3bstream applications and begin streaming data over the internet.
