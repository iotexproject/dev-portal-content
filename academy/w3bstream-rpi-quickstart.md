import { Alert, AlertIcon, AlertTitle, AlertDescription, Box } from '@chakra-ui/react';

<Alert status='success'>
    <AlertIcon />
	<Box>
	    <AlertTitle>About W3bstream</AlertTitle>
	    <AlertDescription>
		W3bstream is a powerful new framework that enables developers to connect data generated in the physical world to the blockchain world. Using the IoTeX blockchain, W3bstream orchestrates a network of nodes that receive and verify data from IoT devices and machines, and generates proofs of real-world facts that can be used by dApps on different blockchains.  
	    </AlertDescription>
	</Box>
</Alert>
Welcome to this tutorial! Here, we will guide you through the process of using the [W3bstream](https://w3bstream.com/) Client SDK for Linux devices. Specifically, we will show you how to send data from a Raspberry Pi to a W3bstream project. This tutorial will cover the steps involved in importing and using the SDK in C++, building the firmware, publishing a data message, and receiving it in a W3bstream applet.

By the end of this tutorial, you will have the necessary skills and knowledge to either create your own W3bstream-compatible IoT devices or migrate existing ones. Let's get started!

## Creating the w3bstream Project
To begin streaming data from your IoT device, the first step is to deploy a new W3bstream Project. You can deploy the standard "Hello World" project on the W3bstream DevNet, which by default logs each message received by authorized devices to the W3bstream console. To quickly get started with initiating a project and adding a device account, refer to the [Deploy "Hello World" section](https://docs.w3bstream.com/get-started/w3bstream-studio](https://docs.w3bstream.com/get-started/deploying-an-applet) in the W3bstream documentation.

Once you have your W3bstream project set up, it's time to proceed with streaming data from your IoT device.

## Setting up the Environment

Before building and running our application, we need to ensure that our environment is properly configured. Here are the prerequisites:

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

Create a folder for your firmware and clone the Client SDK inside it:

```bash
mkdir my-firmware && cd my-firware
git clone https://github.com/machinefi/w3bstream-iot-sdk 
```

## Create the project file
To configure the basic project settings, create a new CMakeLists.txt file and copy-paste the following code. These settings are mostly standard, but in your own project, you may want to adjust the SOURCES setting, include additional library subdirectories, or rename the generated executable file name.

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
To begin, create a main.cpp file that will contain the source code for our firmware. You can use any text editor of your choice to create this file. For example, you can use the Nano text editor by running the following command:
```bash
nano main.cpp
```
Alternatively, you can use your preferred text editor to create the main.cpp file.

### Header

Let's start by importing some standard libraries:

```cpp
#define _BSD_SOURCE
#include <stdio.h>
#include <sstream>
#include <iomanip>
#include <time.h>
#include <iostream>
#include <curl/curl.h>
```

Next, add the import for the W3bstream Client SDK:

```cpp
#include "psa/crypto.h"
```

Additionally, we can define a custom enumeration to manage return status codes:
 
```cpp
// Custom types
enum ResultCode { SUCCESS, ERROR, ERR_HTTP, ERR_CURL, ERR_FILE_READ, ERR_PSA, };
```

### Endpoint configuration

Let's define some variables that can be used to configure the connection to the W3bstream network, as well as a few others that will help in using the W3bstream SDK:

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

To configure the project_route, make sure you copy the HTTP endpoint of your W3bstream project from the Events section page in the W3bstream Studio UI
<img width="1330" alt="image" src="https://github.com/iotexproject/dev-portal-content/assets/11096047/ee7a171b-cac9-4bab-bcea-6074a76b4108"></img>

Additionally, you need to set the device_token variable to a valid device authorization token that you obtained from the Devices section in the W3bstream Studio UI.

<img width="1330" alt="image" src="https://github.com/iotexproject/dev-portal-content/assets/11096047/16b9b17d-de54-4396-96eb-0616ef3f4e72"></img>

Replace the "project_route" with the actual HTTP endpoint of your W3bstream project, and the "device_token" with the appropriate value you obtained from the W3bstream Studio UI.

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
Let's create some useful functions for payload creation. In this "Get Started" demo, we want our device to send a simple payload to our W3bstream project. The payload should be in the following format:

```json
{
  "data": {
    "value": "0",
    "timestamp": "1666788776"
  },
  "device_id": "1834f959f...29d0fef2557"
}
```

We'll start with the create_payload function. This function takes an integer value, which may represent a sensor reading or a digital I/O input status, and a string that represents a unique device ID. It then builds the W3bstream message payload in the form of the JSON message shown above. Here's the code:

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
The create_payload function converts the value parameter to a string and then builds the payload string by concatenating the necessary JSON elements. It uses the device_id variable defined earlier to include the device ID in the payload.

This function will be useful for creating the payload to be sent by your device to the W3bstream project.

### Public key generation
Now let's focus on how to use the W3bstream Client SDK to generate a public/private key pair. The `generate_key` function below will generate the key pair in the form of a `key_id` and then recover the public key to return it as a string. We will use this public key later to provide a unique device ID to be included in the data message:

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
The generate_key function uses the W3bstream Client SDK to generate an ECDSA key pair. It sets the necessary attributes for the key, generates the key using the PSA API, and then exports the public key from the key pair. The exported public key is stored as a hex string in the public_key parameter.

This function will be used to generate a unique device ID for the W3bstream project.

### Sending the message to W3bstream
The next function we'll explore is `publish_message`. This function takes care of creating the actual message according to the W3bstream protocol and sends it over an HTTP POST request:

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
The publish_message function uses the cURL library to send an HTTP POST request to the specified endpoint with the provided payload. It sets the necessary cURL options such as the URL, payload, headers, and SSL usage. If the request is successful, the function returns `ResultCode::SUCCESS`. Otherwise, it returns an appropriate error code.

This function will be used to publish the message to the W3bstream project endpoint using the generated payload and the device token for authentication.

### Utility functions
The last function is a utility function called get_time_str that retrieves the current time and returns it as a string:

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
The get_time_str function uses the time function to retrieve the current time, and then formats it using the strftime function with the "%X" format specifier. This format specifier represents the time in the "HH:MM:SS" format. The resulting time string is stored in the buf character array and returned as a string.

This utility function can be used to display the current time in log messages or other parts of the code where the timestamp is needed.

## Firmware workflow

The main function of the firmware handles the overall workflow of the application. Here's a breakdown of the steps performed in the main function:

1. Initialize PSA Crypto API:
```cpp
std::string public_key;
ResultCode res = generate_key(public_key);
```
This function initializes the PSA Crypto library provided by the W3bstream Client SDK.

2. Generate a public/private key pair and store the public key in device_id:

```cpp
std::string public_key;
ResultCode res = generate_key(public_key);
```

This step generates a new ECDSA key pair using the W3bstream Client SDK. The public key is extracted from the generated key pair and stored in the device_id variable. The generate_key function is responsible for this process.

3. Create the payload for the message:
```cpp
std::string payload = create_payload(random(), public_key);
```
In this step, the create_payload function is called to create the message payload. It takes a random value (obtained using the random() function) and the public key as inputs. The payload is formatted as a JSON string.

4. Publish the message to W3bstream:
```cpp
res = publish_message(project_route, payload, publisher_token);
```
The publish_message function is called to send the message to the W3bstream project. It takes the project endpoint, the payload, and the publisher token as inputs. The function performs an HTTP POST request to the endpoint, including the payload in the request body.

5. Cleanup
```cpp
psa_destroy_key(key_id);
```
After the message is published, the key pair generated earlier is destroyed to ensure proper cleanup.

The main function handles the key generation, payload creation, and message publishing. It also performs error handling for each step to provide appropriate feedback.

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

<img width="1026" alt="image" src="https://github.com/iotexproject/dev-portal-content/assets/11096047/8fb6d287-d2b6-40b1-893b-af29dbd1b2f6"></img>

