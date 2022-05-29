---
title: Get started with Arduino and IoTeX
description: Learn how to connect an Arduino board to the IoTeX blockchain
---
# Introduction
In this geting started guide we will show how you can interact with the IoTeX Blockchain from an embedded device.

# Final result
At the end of this guide you will be able to send an IOTX token transafer from an ESP32 board.

# Core components
We will use the following core concepts and componnets:
- A TTGO T1 board based on [ESP32](https://www.espressif.com/en/products/socs/esp32) 
- The [IoTeX C++ SDK for embedded systems](https://github.com/iotexproject/arduino-sdk)
- [Arduino IDE](https://www.arduino.cc/en/software)
- IoTeX [Smart contracts](https://developers.iotex.io/posts/deploy-smart-contracts-on-iotex-using-hardhat)

# Selecting the board
We will use a development board based on the ESP-WROOM-32 chip, but you can use any [supported board](https://github.com/iotexproject/arduino-sdk/blob/main/README.md#supported-boards).
If you have never used an ESP32 board with Arduino before, you will have to first install the support for ESP32 in th board manager, alternatively you can skip steps 1. and 2. below: 

1. open the `Tools->Board->Board manager` menu
2. search for ÈSP32` and install the support for `esp32`boards
3. finally, open the `Tools->Board->ESP32 Arduino` and select `TTGO T1`

<img width="2470" alt="image" src="https://user-images.githubusercontent.com/11096047/170865100-0d738ad7-afb3-4faa-8dfd-8b5651b34bd6.png">

# Installing the IoTeX SDK for Arduino
In the Arduino IDE, create and save a new sketch, then open the `Tools->Manage Libraries...` menu item. 
In the Library Manager dialog, search for IoTeX-blockchain-client and click install 

<img width="2084" alt="image" src="https://user-images.githubusercontent.com/11096047/170863513-500f4dd8-2010-418d-844d-6aff187a3545.png">

# The full Sketch
Copy-paste the following sketch in the IDE:

```c++
#include <Arduino.h>
#include <WiFi.h>
#include <map>
#include "IoTeX-blockchain-client.h"

constexpr const char ip[] = "gateway.iotexlab.io";
constexpr const char baseUrl[] = "iotexapi.APIService";
constexpr const int port = 10000;
constexpr const char wifiSsid[] = "<YOUR WIFI NETWORK NAME>";
constexpr const char wifiPass[] = "<YOUR WIFI PASSWORD>";

// Create the IoTeX client connection
Connection<Api> connection(ip, port, baseUrl);

void initWiFi() 
{
    #if defined(ESP8266) || defined(ESP32)
    WiFi.mode(WIFI_STA);
    #endif
    WiFi.begin(wifiSsid, wifiPass);
    Serial.print(F("Connecting to WiFi .."));
    while (WiFi.status() != WL_CONNECTED) {
        Serial.print('.');
        delay(1000);
    }
    Serial.println(F("\r\nConnected. IP: "));
    Serial.println(WiFi.localIP());
}

void setup() {
    Serial.begin(115200);

    #if defined(__SAMD21G18A__)
    delay(5000);    // Delay for 5000 seconds to allow a serial connection to be established
    #endif

    // Connect to the wifi network
    initWiFi();
}

void loop() {
    // Private key of the origin address
    const char pkString[] = SECRET_PRIVATE_KEY;

    // Recipient IoTeX address
    char destinationAddr[IOTEX_ADDRESS_C_STRING_SIZE] = "destination-address";
    
    // Amount of RAU to transfer
    char amount[IOTEX_ADDRESS_C_STRING_SIZE] = "1";

    // Convert the private key and address hex strings to byte arrays
    uint8_t pkBytes[IOTEX_PRIVATE_KEY_SIZE];
    signer.str2hex(pkString, pkBytes, IOTEX_SIGNATURE_SIZE);

    // Create the account object
    Account originAccount(pkBytes);
    
    // Get the IoTeX address
    char originIotexAddr[IOTEX_ADDRESS_C_STRING_SIZE] = "";
    originAccount.getIotexAddress(originIotexAddr);

    // Get the account nonce
    AccountMeta accMeta;
    ResultCode result = connection.api.wallets.getAccount(originIotexAddr, accMeta);
    IotexString nonceString = accMeta.pendingNonce;
    uint64_t nonce = atoi(nonceString.c_str());
    if (result != ResultCode::SUCCESS)
    {
        Serial.print("Error getting account nonce: ");
        Serial.println(IotexHelpers.GetResultString(result));
    }

    // Send the transfer 
    uint8_t hash[IOTEX_HASH_SIZE] = {0};
    result = originAccount.sendTokenTransferAction(connection, nonce, 20000000, "1000000000000", amount, destinationAddr, hash);

    // Print the result and the hash if successful
    if (result == ResultCode::SUCCESS)
    {
        Serial.print("Transfer sent. Hash: ");
        for (int i=0; i<IOTEX_HASH_SIZE; i++)
        {
            char buf[3] = "";
            sprintf(buf, "%02x", hash[i]);
            Serial.print(buf);
        }
        Serial.println();
    }
    else
    {
        Serial.print("Error sending transfer: ");
        Serial.println(IotexHelpers.GetResultString(result));
    }

    Serial.println("Program finished");
    while(true)
    {
        delay(1000);
    }
}
```

# Sketch review
Let's review
## Imports
We must of course import the Arduino standerd library. For WiFi connectivity on ESP32 it's enough to just import the standard Arduino WiFi library, for other boards you can find the full code for this example in the [GitHub repository](https://github.com/iotexproject/arduino-sdk/blob/main/examples/SendTransfer/SendTransfer.ino). Finally, we will include the IoTeX Arduino SDK

```c++
#include <Arduino.h>
#include <WiFi.h>
#include "IoTeX-blockchain-client.h"
```

# Connect to the IoTeX Gateway
The embedded SDK is based on the Native IoTeX API, which is exposed by any blockchain node through the gRPC protocol. Because implementing a gRPC client on embedded boards is not trivial, the IoTeX library requires a JSON over HTTPs translation of the blockchain API. You can either configure your own IoTeX Blockchain Gateway node and a gRPC-JSON transcoder, or you can use a community contributed endpoint for development purposes like below:

```c++
constexpr const char ip[] = "gateway.iotexlab.io";
constexpr const char baseUrl[] = "iotexapi.APIService";
constexpr const int port = 10000;
```
