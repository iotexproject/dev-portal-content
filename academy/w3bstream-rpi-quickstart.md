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

## Creating the firmware
Create a `main.cpp` file that will contain the source code of our firmware

```bash
nano main.cpp
```

### Header

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
// Custom types
enum ResultCode { SUCCESS, ERROR, ERR_HTTP, ERR_CURL, ERR_FILE_READ, ERR_PSA, };
```

### Endpoint configuration

Go ahead with some variables that can be used to configure the connection to the W3bstream network and a few others that help using the W3bstream SDK:

```cpp
// User configuration and constants.
namespace
{
	// Connection details.
	std::string publisher_token = "";   // The publisher token.
	std::string project_route = "";     // The w3bstream endpoint.

	// Constants.
	const size_t public_key_size = 65;
	const size_t private_key_size_bits = 256;

	// Variables to hold global state, etc.
	psa_key_id_t key_id;		    // The PSA API key slot id.
	std::string device_id = "";	    // The device id. In our case, we will use a public key generated using the SDK.
}
```

Make sure you copy the HTTP endpoint of your W3bstream project from the Events section page in the W3bstream Studio UI:
<img width="1330" alt="image" src="https://github.com/iotexproject/dev-portal-content/assets/11096047/ee7a171b-cac9-4bab-bcea-6074a76b4108"></img>

copy a device auth token from the devices section:

<img width="1330" alt="image" src="https://github.com/iotexproject/dev-portal-content/assets/11096047/16b9b17d-de54-4396-96eb-0616ef3f4e72"></img>

and use them to set the `project_route` and `device_token` variables respectively. 

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

### Payload creation
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

So let's start with the `create_payload` function: this function takes an integer value that may represent a sensor reading or a digital I/O input status, a string that represents a unique device id, and builds the W3bstream message payload in the form of the stringyfied JSON message above:
```cpp
std::string create_payload(int value)
{
    // Convert value to a string
    std::string value_str = std::to_string(value);

    std::string payload = "{";
    std::string data = "";
    data = "{";
	data += "\"status\":" + value_str;
    data += ",\"timestamp\":" + std::to_string(time(NULL));
    data += "}";

    payload += "\"data\":" + data + ",";
    payload += "\"device_id\":\"" + device_id + "\"";
    payload += "}";

    return payload;
}
```

### Public key generation
Now let's focus on how to use the W3bstream Client SDK to generate a public/private key pair. The `generate_key` below will generate the pair in the form of a `key_id`, and then recover the public key only to return it as a string. We will use this public key later to providea unique device id to be included in the data message:

```cpp
static ResultCode generate_key(std::string &public_key)
{
	std::cout << "Generating a new ECDSA key pair."  << std::endl;

	psa_key_attributes_t attributes = PSA_KEY_ATTRIBUTES_INIT;
	psa_set_key_usage_flags(&attributes, PSA_KEY_USAGE_SIGN_HASH | PSA_KEY_USAGE_VERIFY_HASH | PSA_KEY_USAGE_EXPORT);
	psa_set_key_algorithm(&attributes, PSA_ALG_ECDSA(PSA_ALG_SHA_256));
	psa_set_key_type(&attributes, PSA_KEY_TYPE_ECC_KEY_PAIR(PSA_ECC_FAMILY_SECP_K1));
	psa_set_key_bits(&attributes, private_key_size_bits);
	
	auto status = psa_generate_key(&attributes, &key_id);
	if (status != PSA_SUCCESS)
	{
		std::cerr << "Failed to generate keys: " << status << std::endl;
		return ResultCode::ERR_PSA;	
	}

	// Query the public key from the PSA API slot and return it as a string.
	uint8_t exported_public_key[public_key_size] = {0};
	size_t exported_public_key_len = 0;
	
	status = psa_export_public_key(key_id, exported_public_key, sizeof(exported_public_key), &exported_public_key_len);
	if (status != PSA_SUCCESS)
	{
		std::cerr << "Public key not returned by the SDK: " << status << std::endl;
		return ResultCode::ERR_PSA;	
	}

	// Store  the public key as a hex string
	std::stringstream ss;
	for (int i = 0; i < public_key_size; i++) { 
		ss << std::hex << std::setw(2) << std::setfill('0') << (int)exported_public_key[i]; 
	}
	
	public_key = ss.str();

	std::cout << "Public key is: " << public_key << std::endl;

	return ResultCode::SUCCESS;
}
```
### Sending the message to W3bstream
the third function we explore is `publish_message`. This function takes care of creating the actual message according to the W3bstream protocol, and sends it over a HTTP POST request:

```cpp
ResultCode publish_message(std::string endpoint, std::string payload, std::string device_token)
{
	std::cout << get_time_str() << ": Publishing message: " << payload << std::endl <<" to " << endpoint << std::endl;
    CURL *curl;
    CURLcode res;
    curl = curl_easy_init();
    if(curl)
	{
        curl_easy_setopt(curl, CURLOPT_URL, endpoint.c_str());
        curl_easy_setopt(curl, CURLOPT_POSTFIELDS, payload.c_str());
        struct curl_slist *headers = NULL;
        headers = curl_slist_append(headers, "Content-Type: application/json");
		headers = curl_slist_append(headers, ("Authorization: Bearer " + device_token).c_str());
        curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);
        curl_easy_setopt(curl, CURLOPT_USE_SSL, (long)CURLUSESSL_TRY);
        res = curl_easy_perform(curl);
        if(res != CURLE_OK)
        {
            std::cerr << "Failed to publish message: " << curl_easy_strerror(res) << std::endl;
            return ResultCode::ERR_HTTP;
        }

        curl_easy_cleanup(curl);
    }
    else
    {
        std::cerr << "Failed to initialize curl" << std::endl << std::endl;
        return ResultCode::ERR_CURL;
    }
	std::cout << std::endl;
    return ResultCode::SUCCESS;
}
```
### Utility functions
The last function is just a utility function that gets the current and returns it as a string:
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

## Firmware workflow

At this point we are ready to analyze the main() function and go through the firmware workflow:

```cpp
int main(int argc, char* argv[])
{
	// Initialize PSA Crypto.
    psa_status_t status = psa_crypto_init();
    if (status != PSA_SUCCESS)
	{
        std::cerr << "Failed to initialize PSA crypto library: " << status << std::endl;
        return (-1);
    }

	// Generate a public/private key pair and store the public key in device_id
	std::string public_key;
	ResultCode res = generate_key(public_key);
	if (res != ResultCode::SUCCESS)
	{
		std::cerr << "Failed to generate keys. " << std::endl;
		return 1;
	}

    // Publish a message to W3bstream 
    std::string payload = create_payload(random(), public_key);

	res = publish_message(project_route, payload, publisher_token);
	if (res != ResultCode::SUCCESS)
	{
		std::cerr << "Failed to publish message. " << std::endl;
	}
	else 
	{
		std::cout << "Successfully published message. " << std::endl;

	}

	// Cleanup
	psa_destroy_key(key_id);

	return 0;
}
```

The example application firmware first initializes the PSA Crypto API provided by the W3bstream Client SDK. A single function call to  `psa_crypto_init()` is required for this:

```c++
psa_status_t status = psa_crypto_init();
```

We used an ECDSA key pair to obtain a public key and use it to serve as a unique device_id to be included in the message. So we create a payload according to our W3bstream project expectations, using a random value and the device id.

Finally, the message W3bstream message is constructed merging our payload with some protocol data like the project auth token and a W3bstream event id, ans is sent to the network. 


## Building the Applictaion Firmware

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

### Running the Application

At this point you can run the firmware by just typing

```bash
buiuld-out/my-firmware
```
the firmware will just send a single message to your W3bstream project: open the logs section of the project and find the output of the applet:

<img width="1026" alt="image" src="https://github.com/iotexproject/dev-portal-content/assets/11096047/8fb6d287-d2b6-40b1-893b-af29dbd1b2f6">

## What's next?

