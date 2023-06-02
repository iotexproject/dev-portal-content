import { Button, Alert, AlertIcon, AlertTitle, AlertDescription, Link, Box, flexWrap } from '@chakra-ui/react'

## Introduction

<Alert status='success'>
    <AlertIcon />
    <AlertTitle>About W3bstream</AlertTitle>
    <AlertDescription>
        W3bstream is a powerful new framework that enables developers to connect data generated in the physical world to the blockchain world. Using the IoTeX blockchain, W3bstream streams orchestrates a network of nodes that receive and verify data from IoT devices and machines, and generates proofs of real-world facts that can be used by dApps on different blockchains.  
    </AlertDescription>
</Alert>

In this article, we'll walk you through how to use the W3bstream Client SDK for Linux devices to send data from a Raspberry Pi to a W3bstream project. We'll cover how to importt and use the SDK in C++, build the firmware, publish a data message and receiving it in a W3bstream applet. 

By the end of this article, you'll have the skills and knowledge you need to start building your own W3bstream-compatible IoT devices or migrate existing ones.

## Creating the w3bstream Project

To start streaming data from your IoT device, you'll first need to deploy a new W3bstream Project. You can deploy the standard "Hello World" project on the W3bstream DevNet that by default will log each message received by authorized devices to the W3bstream console. Checkout the [Deploy "Hello World" section]([https://docs.w3bstream.com/get-started/w3bstream-studio](https://docs.w3bstream.com/get-started/deploying-an-applet)) in the W3bstream documentation to learn how to quickly get started initiating a project, adding a device account. 

With your w3bstream project set up, it's time to start streaming data from your IoT device.

## Setting up the Environment

Before we can build and run our application, we need to make sure our environment is properly set up. Here are the prerequisites: 

1. Connect to your Raspberry Pi using ssh:
```bash
ssh pi@raspberrypi.local
```

2. Update the system and install the required system packages:

```bash
sudo apt update
sudo apt upgrade
sudo apt-get install -y python3-pip build-essential cmake libcurl4-openssl-dev git
```

These packages are necessary for building the w3bstream IoT SDK.

3. Clone the W3bstream Client SDK for Linux:

Let's create a folder for our firmware and clone the Client SDK inside:

    ```bash
    mkdir my-firmware && cd my-firware
    git clone https://github.com/machinefi/w3bstream-iot-sdk 
    ```

## Create the project file
Create a new `CMakeLists.txt` and copy paste this basic project configuration. These are mostly standard settings, however, in your own project you may want to configure the `SOURCES` setting, include more libraries subdirectories, or rename the generated executable file name.

```text
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

## Creating the firmware code
Create a `main.cpp` file that will contain the source code of our firmware

```
nano main.cpp
```

Start with some standard imports:
```cpp
#define _BSD_SOURCE
#include <stdio.h>
#include <sstream>
#include <iomanip>
#include <time.h>
#include <iostream>
#include <curl/curl.h>
```

then add the import for the W3bstream Client SDK:

```cpp
#include "psa/crypto.h"
```
 and a custom enum that we can use to manage return status codes:
 
```cpp
// Custom types.
enum ResultCode
{ SUCCESS, ERROR, ERR_HTTP, ERR_CURL, ERR_FILE_READ, ERR_PSA, };
```

Go ahead with some variables that can be used to configure the connection to the W3bstream network and a few others that help using the W3bstream SDK:

```cpp
// User configuration and constants.
namespace
{
	// Connection details.
	std::string publisher_token = "";           // The publisher token.
	std::string project_route = "";     		// The w3bstream endpoint.

	// Constants.
	const size_t public_key_size = 65;
	const size_t private_key_size_bits = 256;

	// Variables to hold global state, etc.
	psa_key_id_t key_id;						// The PSA API key slot id.
	std::string device_id = "";					// The device id. In our case, the public key.
}
```

Make sure you copy the HTTP endpoint of your W3bstream project from the Events section page in the W3bstream Studio UI:
<img width="1330" alt="image" src="https://github.com/iotexproject/dev-portal-content/assets/11096047/07e62b14-2def-459f-b6cd-02cfe8c2fe95"></img>

copy a device auth token from the devices section:
<img width="1330" alt="image" src="https://github.com/iotexproject/dev-portal-content/assets/11096047/16b9b17d-de54-4396-96eb-0616ef3f4e72"></img>


and use them to set the `project_route` and `device_token` respectively. 

```cpp
namespace
{
    // The device auth token.
    std::string device_token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJQYXlsb2FkIjoiOTAyNzcxMjI4MDMyMzA3NiIsImlzcyI6InczYnN0cmVhbSJ9.RCOIGP6JtanL9JzBakxpFf7JVSQmYAd7tanhHJK5MFc";
    // The project name.
    std::string project_name = "eth_0x2c37a2cbcfaccdd0625b4e3151d6260149ee866b_smart_energy";
...
}
```

Let's go ahead and create some useful functions. In this get started demo we want our device to send a simple payload to our W3bstream project, like this:
```json
{
  "data": {
    "value": "0",
    "timestamp": "1666788776"
  },
  "device_id": "1834f959f...29d0fef2557"
}
```

So let's start from the create_payload function:
```cpp


So we can create a function that gets the current timestamp as a string:
```cpp
std::string get_time_str()
{
	time_t now = time(0);
	struct tm tstruct;
	char buf[80];
	tstruct = *localtime(&now);
	strftime(buf, sizeof(buf), "%X", &tstruct);
	return buf;
}
```

and the function that creates the actual message IoT message, that will be the payload of the W3bstream message:

```cpp



## Building the Applictaion Firmware: Using the W3bstream IoT SDK

To build the application using the W3bstream IoT SDK, you can follow these steps:

1. Run the following command to configure the build:

```bash
cmake -DGIT_SUBMODULE_UPDATE=ON -S ./ -B ./build-out
```

This will clone the required Git submodules and generate the build files in the `build-out` directory.

3. Run the following command to build the example application:

```bash
cmake --build build-out --target my-firmware
```

This will compile the source code and generate an executable in the `build-out` directory called `my-firmware` (or any oter name you configured in your CMake.txt.

### Running the Application: Sending Data to w3bstream

At this point you can run the firmware by just typing

```bash
buiuld-out/my-firmware
```
the firmware will just send a single message to your W3bstream project: open the logs section of the project and find the output of the applet:


#### Firmware workflow

The example application firmware first initializes the PSA Crypto API provided by the W3bstream Client SDK. A single function call to  `psa_crypto_init()` is required for this:

```c++
psa_status_t status = psa_crypto_init();
```

The application uses an ECDSA key pair to sign the data. 

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
