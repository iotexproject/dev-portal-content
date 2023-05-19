## Introduction

W3bstream is a powerful new framework that enables developers to connect data generated in the physical world to the blockchain world. Using the IoTeX blockchain, W3bstream streams orchestrates a network of nodes that receive and verify data from IoT devices and machines, and generates proofs of real-world facts that can be used by dApps on different blockchains.  

In this article, we'll walk you through how to use the W3bstream Client SDK for Linux devices to send data from a Raspberry Pi to a W3bstream project. We'll cover how to importt and use the SDK in C++, build the firmware, publish a data message and receiving it in a W3bstream applet. 

By the end of this article, you'll have the skills and knowledge you need to start building your own W3bstream-compatible IoT devices.

## Creating the w3bstream Project

To start streaming data from your IoT device, you'll first need to deploy a new W3bstream Project. You can deploy the standard "Hello World" project on the W3bstream DevNet that by default will log each message received by authorized devices. Checkout the [Deploy "Hello World" section]([https://docs.w3bstream.com/get-started/w3bstream-studio](https://docs.w3bstream.com/get-started/deploying-an-applet)) in the W3bstream documentation to learn how to quickly get started initiating a project, adding a device account. 

With your w3bstream project set up, it's time to start streaming data from your IoT device.

## Sending data from Raspberry Pi

### Prerequisites: Setting up the Environment

Before we can build and run our application, we need to make sure our environment is properly set up. Here are the prerequisites. 

Connect to your Raspberry Pi using ssh:
```
ssh pi@raspberrypi.local
```

update the system and install the required system packages:

    ```bash
    sudo apt update
    sudo apt upgrade
    sudo apt-get install -y python3-pip build-essential cmake libcurl4-openssl-dev git
    ```

    These packages are necessary for building the w3bstream IoT SDK.

2. Clone the W3bstream Client SDK for Linux:
Let's create a folder for our firmware and clone the Client SDK inside:

    ```bash
    mkdir my-firmware && cd my-firware
    git clone https://github.com/machinefi/w3bstream-iot-sdk 
    ```

### Create the project file
Create a new `CMakeLists.txt` and copy paste this basic project configuration:
```
# CMake 3.10: Ubuntu 20.04.
# https://cliutils.gitlab.io/modern-cmake/chapters/intro/dodonot.html
cmake_minimum_required(VERSION 3.16) 

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 11)

project(
  TEST
  VERSION 1.0
  LANGUAGES C CXX)

set(SOURCES 
    main.cpp
)

add_executable(my-firmware ${SOURCES})

target_link_libraries(my-firmware
    PRIVATE ws_iot_sdk
    PRIVATE curl
)

add_subdirectory(w3bstream-iot-sdk)
```
### Creating the firmware code
Create a `main.cpp`file where we will code our firmware

```
nano main.cpp
```

Start with the required imports:
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sstream>
#include <cstdio>
#include <iomanip>
#include <fcntl.h>
#include <unistd.h>
#include <time.h>
#include <string>
#include <algorithm>
#include <iostream>
#define _BSD_SOURCE
#include <sys/time.h>
#include <thread>
#include <curl/curl.h>
#include <fstream>
```
and add the import for the W3bstream Client SDK and a custom enum for return codes:
```cpp
#include "psa/crypto.h"
// Custom types.
enum ResultCode
{ SUCCESS, ERROR, ERR_HTTP, ERR_CURL, ERR_FILE_READ, ERR_PSA, };
```
Go ahead with some constants:
```cpp
// User configuration and constants.
namespace
{
    // Publisher details.
    // The publisher token.
    std::string device_token = "";
    // The project name.
    std::string project_name = "";

    // HTTP server details.
    std::string host = "devnet-staging-api.w3bstream.com";  // The w3bstream server ip or hostname.
    std::string publish_url = "https://" + host + "/srv-applet-mgr/v0/event/" + project_name;

    // Constants for using the SDK.
    const size_t public_key_size = 65;
    const size_t private_key_size = 32;
    const size_t private_key_size_bits = private_key_size*8;

    // Variables to hold global state, etc.
    psa_key_id_t key_id;                                    // The PSA API key slot id.
    std::string device_id = "";                                // The device id. In our case, the public key.
}
```

Make sure you fetch the W3bstream project name from the settings page of your project and the device token from the devices section, and use them to set the `project_name` and `device_token` variables. 



### Building the Application: Using the W3bstream IoT SDK

To build the application using the W3bstream IoT SDK, follow these steps:

1. Run the following command to configure the build:

```bash
cmake -DGIT_SUBMODULE_UPDATE=ON -S ./ -B ./build-out
```

    This will clone the required Git submodules and generate the build files in the `build-out` directory.

3. Run the following command to build the example Quick Start application:

```bash
cmake --build build-out --target my-firmware
```

This will compile the source code and generate an executable in the `build-out` directory called `example-quick-start-rpi`.

### Running the Application: Sending Data to w3bstream

At this point you can run the firmware by just typing

```bash
buiuld-out/my-firmware
```
the firmware will just send a single message to your W3bstream project: open the logs section of the project and find the output of the applet:

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
