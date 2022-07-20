---
title: Get started with Arduino and IoTeX
description: Learn how to connect an Arduino board to the IoTeX blockchain
path: academy/get-started-with-arduino.md
---

# Introduction

In this geting started guide we will show how you can interact with the IoTeX Blockchain from an embedded device.

# Final result

At the end of this guide you will be able to read the IOTX balance of an IoTeX wallet address from an ESP32 board.

# Core components

We will use the following core concepts and componnets:

- A dev board based on [ESP32](https://www.espressif.com/en/products/socs/esp32)
- The [IoTeX C++ SDK for embedded systems](https://github.com/iotexproject/arduino-sdk)
- [Arduino IDE](https://www.arduino.cc/en/software)
- IoTeX [Smart contracts](https://developers.iotex.io/posts/deploy-smart-contracts-on-iotex-using-hardhat)

# Selecting the board

We will use a development board based on the ESP-WROVER-32 chip. If you have never used an ESP32 board with Arduino before, please see their doc on how to install ESP32 boards in the Arduino IDE.

1. open the `Tools->Board->ESP32 Arduino` menu and select `ESP32 Wrover Module`.
2. open the `Tools->Port`menu and select the serial port of your board (it should look similar to `/dev/cu.SLAB_USBtoUART`)

![](https://user-images.githubusercontent.com/11096047/170865100-0d738ad7-afb3-4faa-8dfd-8b5651b34bd6.png)

# Installing the IoTeX SDK for Arduino

In the Arduino IDE, create and save a new sketch, then open the `Tools->Manage Libraries...` menu item.
In the Library Manager dialog, search for IoTeX-blockchain-client and click `install`.

![](https://user-images.githubusercontent.com/11096047/170863513-500f4dd8-2010-418d-844d-6aff187a3545.png)

# The full Sketch

Copy-paste the following sketch in the IDE (edit your WiFi network configuration), hit the `upload` button in the IDE and wait for the software to be flashed into the board (notice that some boards require you to press the one of the available buttons sometimes called `boot` to allow it to be flashed the software).
Once the flash is complete, select `Tools->Serial monitor`from the menu and configure the monitor speed on 115000 baud to see to output. The onboard led should get activated if the address you checked has some balance, or it will stay off if the balance is 0.

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include "IoTeX-blockchain-client.h"

#define ONBOARD_LED  2

constexpr const char ip[] = "gateway.iotexlab.io";
constexpr const char baseUrl[] = "iotexapi.APIService";
constexpr const int port = 10000;

// Set your Wifi Network name
constexpr const char wifiSsid[] = "<YOUR WIFI NETWORK NAME>";
// Set your WiFi password
constexpr const char wifiPass[] = "<YOUR WIFI PASSWORD>";
// Set the wallet address to check
const char accountStr[] = "io1xkx7y9ygsa3dlmvzzyvv8zm6hd6rmskh4dawyu";

// Create the IoTeX client connection
Connection<Api> connection(ip, port, baseUrl);

void initWiFi()
{
    WiFi.mode(WIFI_STA);
    WiFi.begin(wifiSsid, wifiPass);
    Serial.print(F("Connecting to WiFi .."));
    while (WiFi.status() != WL_CONNECTED) {
        Serial.print('.');
        delay(1000);
    }
    Serial.print(F("\r\nConnected. IP: "));
    Serial.println(WiFi.localIP());
}

void setup() {
    Serial.begin(115200);

    // Connect to the wifi network
    initWiFi();

    // Configure the LED pin
    pinMode(ONBOARD_LED, OUTPUT);
}

void loop() {
    // Query the account metadata
    AccountMeta accountMeta;
    ResultCode result = connection.api.wallets.getAccount(accountStr, accountMeta);

    // Print the result
    Serial.print("Result: ");
    Serial.println(IotexHelpers.GetResultString(result));

    // If the query suceeded, print the account metadata
    if (result == ResultCode::SUCCESS)
    {
        Serial.print("Balance: ");
        Serial.println(accountMeta.balance);
        Serial.print(F("Nonce: "));
        Serial.println(accountMeta.nonce.c_str());
        Serial.print(F("PendingNonce: "));
        Serial.println(accountMeta.pendingNonce.c_str());
        Serial.print(F("NumActions: "));
        Serial.println(accountMeta.numActions.c_str());
        Serial.print(F("IsContract: "));
        Serial.println(accountMeta.isContract ? "\"true\"" : "\"false\"");
    }

    // Enable the onboard led if the balance is > 0.01 IOTX)
    Bignum b = Bignum(accountMeta.balance, NumericBase::Base10);
    Bignum zero = Bignum("0", NumericBase::Base10);
    if (b == zero)
      digitalWrite(ONBOARD_LED, LOW);
    else digitalWrite(ONBOARD_LED, HIGH);

    Serial.println("Program finished");

    while (true) { delay(1000); }
}
```

# Notes on the IoTeX Gateway

The embedded SDK is based on the Native IoTeX API, which is exposed by any blockchain node through the gRPC protocol. Because implementing a gRPC client on embedded boards is not trivial, the IoTeX library requires a JSON over HTTPs translation of the blockchain API. You can either configure your own IoTeX Blockchain Gateway node using a gRPC-JSON transcoder by [following this tutorial], or you can use a community contributed endpoint for development purposes, like in the code above.
