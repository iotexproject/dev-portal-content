*[A Blockchain Powered Smart-Lock with Arduino Nano IoT 33](https://developers.iotex.io/posts/Blockchain-Powered-Smart-Lock) demonstrated how to remotely control a home automation device using a smart contract. With this tutorial we'll be adding a new feature to our project: A button that will change the status of the lock and update the smart-contract accordingly.*

## Get Started

As we mentioned in the introduction, this tutorial is built on top of the [Blockchain Powered Smart-Lock with Arduino Nano IoT 33](https://developers.iotex.io/posts/Blockchain-Powered-Smart-Lock). If you haven't done so already, go ahead and complete that tutorial before proceeding with this one. 

Once completed, you'll have a remotely controlled smart-lock that is directly connected to a smart contract on the **IoTeX** blockchain. At this point, you're able to control your lock by directly calling the smart contract. But what if you wanted to physically interact with the lock? Let's go ahead and see how this feature can easily be implemented by adding a button to the wiring and modifying our sketch. 

## Modify the Sketch

The first thing to do is to update the sketch to toggle the status of the lock in the smart contract. Let's look at the `SmartLockDevice.ino` in the `SmartLockDevice` directory. 

The first thing to do here is to add a global variable to help us store the previous status of the lock, as well defining the button pin. 

Add this code after the execution action: 

```cpp
// Global variable to store previous status for toggling the lock
static bool previousStatus = false;

// The button pin
#if defined(__SAMD21G18A__)
#define BUTTON_PIN 3    // Pin D3
#else
#define BUTTON_PIN 18
#endif
```
At this point, we need to create a function to toggle the status of the lock in the smart contract: 

```cpp
// Toggles the status of the lock in the smart contract
void toggleStatusOnBlockchain()
{
    Serial.println("Toggling lock status");
    // Convert the privte key to a byte array
    const char pK[] = SECRET_PRIVATE_KEY;
    uint8_t pk[IOTEX_PRIVATE_KEY_SIZE];
    signer.str2hex(pK, pk, IOTEX_PRIVATE_KEY_SIZE);

    // Create the account and get the nonce
    Account originAccount(pk);
    AccountMeta accMeta;
    ResultCode result = connection.api.wallets.getAccount(fromAddress, accMeta);
    if (result != ResultCode::SUCCESS)
    {
        Serial.print("Error getting account meta: ");
        Serial.print(IotexHelpers.GetResultString(result));
    }
    int nonce = atoi(accMeta.pendingNonce.c_str());

    // Construct the action - Create the parameters
    ParameterValue paramOpen;
        paramOpen.value.boolean = !previousStatus;
        paramOpen.type = EthereumTypeName::BOOL;
    ParameterValuesDictionary params;
    params.AddParameter("open", paramOpen);

    // Contruct the action - Generate contract call data
    String callData = "";
    contract.generateCallData("setState", params, callData);

    // Send the action and store it's hash for printing it to the console
    uint8_t hash[IOTEX_HASH_SIZE] = {0};
    result = originAccount.sendExecutionAction(connection, nonce, 20000000, "1000000000000", "0", contractAddress, callData, hash);

    // If successful print the action has, otherwise print an error message
    if (result == ResultCode::SUCCESS)
    {
        Serial.print("Hash: ");
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
        Serial.println("Failed to toggle lock status");
    }
}
```

Now add the following bit of code to keep track of when the button is pressed, and to interrupt the service runtime accordingly:

```cpp
// Flag that is set on button press
volatile bool buttonPressed = false;
// Interrupt service routine that is triggered when the button is pressed 
#if defined(ESP32)
void IRAM_ATTR isr() {
#else
void isr() {
#endif
    detachInterrupt(BUTTON_PIN);
        buttonPressed = true;
    attachInterrupt(BUTTON_PIN, isr, FALLING);
}
```
In the code above, we used an interrupt to detect button press. The following line was used to enable the interrupt on the button pin and configure the `isr()` function as the interrupt service routine:

```cpp
attachInterrupt(BUTTON_PIN, isr, FALLING);
```

When the button is pressed, the interrupt service routine is executed. It is considered good practice to do as little work as possible from within the `isr()` in order not to hang the main execution process. In our case, we simply set the `buttonPressed` flag and return to the main execution process, which will take care of checking this flag and act accordingly.

We now need a function to set the pin status of the lock: 

```cpp
// Sets the pin status of the lock
void SetLockPinStatus(bool open)
{
    digitalWrite(LOCK_PIN, open);
}
```

Once this is taken care of, it's time to modify the `setup()` and the `loop()` functions: 

The `setup()` function will look like this: 

```cpp
void setup()
{
    Serial.begin(115200);

    #if defined(__SAMD21G18A__)
    delay(5000);    // Delay for 5000 seconds to allow a serial connection to be established
    #endif

    // Connect to the wifi network
    initWiFi();

    // Create the execution action for calling the "isOpen" function
    contract.generateCallData("isOpen", params, callData);
    execution.data = callData;
    strcpy(execution.contract, contractAddress);

    // Configure the lock pin as an output
    pinMode(LOCK_PIN, OUTPUT);
    digitalWrite(LOCK_PIN, LOW);
    
    // Setup the interrupt on the button
    pinMode(BUTTON_PIN, INPUT_PULLUP);
        attachInterrupt(BUTTON_PIN, isr, FALLING);
}
```

And the `loop()` function will look like this: 

```cpp
void loop()
{
    // First check if the button has been pressed and update the lock pin and the smart contract
    if (buttonPressed)
    {
        toggleStatusOnBlockchain();
        SetLockPinStatus(previousStatus);
        buttonPressed = false;
        previousStatus = !previousStatus;
        String statusStr = previousStatus == true ? "OPEN" : "CLOSED";
        Serial.println("Button was pressed. Status changed to: " + statusStr);
        // Delay 7.5 seconds which is the avg confirmation time time, to ensure we don't read stale data on the next read
        delay(7500);
    }
    // If the buton wasn't pressed check if the status has changed in the blockchain
    else
    {
        // Read the contract
        ReadContractResponse response;
        ResultCode result = connection.api.wallets.readContract(execution, fromAddress, 200000, &response);
        if (result != ResultCode::SUCCESS)
        {
            Serial.println("Failed to read contract");
            return;
        }

        // Decode the data into a boolean value where 0 = closed and 1 = open
        bool newStatus = decodeBool(response.data.c_str());
        
        // If we read the contract successfully, update the lock status. Otherwise print an error message
        if (result != ResultCode::SUCCESS)
        {
            Serial.println("Failed to decode data");
        }
        else
        {
            if (newStatus != previousStatus)
            {
                String statusStr = newStatus == true ? "OPEN" : "CLOSED";
                Serial.println("Status read from blockchain has changed to: " + statusStr);
                previousStatus = newStatus;
            }
        }
        // Wait 1 second before polling again
        delay(1000);
    }
}
```

The complete SmartLockDevice.inowill now look like this: 


```cpp
#include <Arduino.h>

#ifdef ESP32
    #include <WiFi.h>
#endif
#ifdef ESP8266
    #include <ESP8266WiFi.h>
    #include <ESP8266HTTPClient.h>
    #include <WiFiClient.h>
#endif
#ifdef __SAMD21G18A__
    #include <WiFiNINA.h>
#endif

#include <map>
#include "IoTeX-blockchain-client.h"
#include "secrets.h"
#include "abi.h"

// Server details
constexpr const char ip[] = IOTEX_GATEWAY_IP;
constexpr const int port = IOTEX_GATEWAY_PORT;
constexpr const char wifiSsid[] = SECRET_WIFI_SSID;
constexpr const char wifiPass[] = SECRET_WIFI_PASS;

// Create the IoTeX client connection
Connection<Api> connection(ip, port, "");

// Enum that represents the status of the lock
enum LockStatus { LOCK_OPEN, LOCK_CLOSED };

// The address
const char contractAddress[] = SECRET_CONTRACT_ADDRESS_IO;

// The address which performs the action
const char fromAddress[] = IOTEX_ADDRESS_IO;

// The contract object
Contract contract(abiJson);

// The call data
String callData = "";
ParameterValuesDictionary params;

// The execution action
Execution execution;

// Global variable to store previous status for toggling the lock
static bool previousStatus = false;

// The button pin
#if defined(__SAMD21G18A__)
#define BUTTON_PIN 3    // Pin D3
#else
#define BUTTON_PIN 18
#endif

// Toggles the status of the lock in the smart contract
void toggleStatusOnBlockchain()
{
    Serial.println("Toggling lock status");
    // Convert the privte key to a byte array
    const char pK[] = SECRET_PRIVATE_KEY;
    uint8_t pk[IOTEX_PRIVATE_KEY_SIZE];
    signer.str2hex(pK, pk, IOTEX_PRIVATE_KEY_SIZE);

    // Create the account and get the nonce
    Account originAccount(pk);
    AccountMeta accMeta;
    ResultCode result = connection.api.wallets.getAccount(fromAddress, accMeta);
    if (result != ResultCode::SUCCESS)
    {
        Serial.print("Error getting account meta: ");
        Serial.print(IotexHelpers.GetResultString(result));
    }
    int nonce = atoi(accMeta.pendingNonce.c_str());

    // Construct the action - Create the parameters
    ParameterValue paramOpen;
        paramOpen.value.boolean = !previousStatus;
        paramOpen.type = EthereumTypeName::BOOL;
    ParameterValuesDictionary params;
    params.AddParameter("open", paramOpen);

    // Contruct the action - Generate contract call data
    String callData = "";
    contract.generateCallData("setState", params, callData);

    // Send the action and store it's hash for printing it to the console
    uint8_t hash[IOTEX_HASH_SIZE] = {0};
    result = originAccount.sendExecutionAction(connection, nonce, 20000000, "1000000000000", "0", contractAddress, callData, hash);

    // If successful print the action has, otherwise print an error message
    if (result == ResultCode::SUCCESS)
    {
        Serial.print("Hash: ");
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
        Serial.println("Failed to toggle lock status");
    }
}

// Flag that is set on button press
volatile bool buttonPressed = false;
// Interrupt service routine that is triggered when the button is pressed 
#if defined(ESP32)
void IRAM_ATTR isr() {
#else
void isr() {
#endif
    detachInterrupt(BUTTON_PIN);
        buttonPressed = true;
    attachInterrupt(BUTTON_PIN, isr, FALLING);
}

// Sets the pin status of the lock
void SetLockPinStatus(bool open)
{
    digitalWrite(LOCK_PIN, open);
}

// Connects to the Wifi network
void initWiFi() 
{
    #if defined(ESP32)
        WiFi.mode(WIFI_STA);
        #define LED_BUILTIN 2
    #endif
    WiFi.begin(wifiSsid, wifiPass);
    Serial.print(F("Connecting to WiFi .."));
    while (WiFi.status() != WL_CONNECTED)
    {
        Serial.print('.');
        delay(1000);
    }
    Serial.println(F("Connected. IP: "));
    Serial.println(WiFi.localIP());
}

void setup()
{
    Serial.begin(115200);

    #if defined(__SAMD21G18A__)
    delay(5000);    // Delay for 5000 seconds to allow a serial connection to be established
    #endif

    // Connect to the wifi network
    initWiFi();

    // Create the execution action for calling the "isOpen" function
    contract.generateCallData("isOpen", params, callData);
    execution.data = callData;
    strcpy(execution.contract, contractAddress);

    // Configure the lock pin as an output
    pinMode(LOCK_PIN, OUTPUT);
    digitalWrite(LOCK_PIN, LOW);
    
    // Setup the interrupt on the button
    pinMode(BUTTON_PIN, INPUT_PULLUP);
        attachInterrupt(BUTTON_PIN, isr, FALLING);
}

void loop()
{
    // First check if the button has been pressed and update the lock pin and the smart contract
    if (buttonPressed)
    {
        toggleStatusOnBlockchain();
        SetLockPinStatus(previousStatus);
        buttonPressed = false;
        previousStatus = !previousStatus;
        String statusStr = previousStatus == true ? "OPEN" : "CLOSED";
        Serial.println("Button was pressed. Status changed to: " + statusStr);
        // Delay 7.5 seconds which is the avg confirmation time time, to ensure we don't read stale data on the next read
        delay(7500);
    }
    // If the buton wasn't pressed check if the status has changed in the blockchain
    else
    {
        // Read the contract
        ReadContractResponse response;
        ResultCode result = connection.api.wallets.readContract(execution, fromAddress, 200000, &response);
        if (result != ResultCode::SUCCESS)
        {
            Serial.println("Failed to read contract");
            return;
        }

        // Decode the data into a boolean value where 0 = closed and 1 = open
        bool newStatus = decodeBool(response.data.c_str());
        
        // If we read the contract successfully, update the lock status. Otherwise print an error message
        if (result != ResultCode::SUCCESS)
        {
            Serial.println("Failed to decode data");
        }
        else
        {
            if (newStatus != previousStatus)
            {
                String statusStr = newStatus == true ? "OPEN" : "CLOSED";
                Serial.println("Status read from blockchain has changed to: " + statusStr);
                previousStatus = newStatus;
            }
        }
        // Wait 1 second before polling again
        delay(1000);
    }

    
}
```

It's now time to modify a bit of code in the `secrets.h` file.

Open the `SmartLockDevice` folder in **Arduino IDE**, then open the secrets.h file and add this line with your private key: 


```cpp
// Wallet private key, used to set the state in the contract
#define SECRET_PRIVATE_KEY    <your_private_key>
```

The complete `secrets.h` will now look like this (but with your appropriate values): 


```cpp
#ifndef SECRETS_H
#define SECRETS_H

// THe WiFi connection details
#define SECRET_WIFI_SSID   <YOUR_WIFI_SSID>
#define SECRET_WIFI_PASS   <YOUR_WIFI_PASSWORD>

// The contract address in io representation
// You can use https://iotexlab.io/eth2io to convert from 0x to io address
#define SECRET_CONTRACT_ADDRESS_IO   <YOUR_CONTRACT_ADDRESS>

// The address which will send the read action
#define IOTEX_ADDRESS_IO    <YOUR_IOTEX_ADDRESS>

// The digital pin number for the lock
#define LOCK_PIN    <YOUR_PIN>

// IoTeX HTTP gateway
#define IOTEX_GATEWAY_IP    <THE_GATEWAY_IP>
#define IOTEX_GATEWAY_PORT    <THE_GATEWAY_PORT>

// Wallet private key, used to set the state i the contract
#define SECRET_PRIVATE_KEY    <your_private_key>

#endif
```

By adding the private key to the `secrets.h` file, we make sure that the corresponding wallet will sign the transaction that gets triggered once we click the button and toggle the state of the lock. (Make sure to use the same wallet that was used to deploy the smart contract, since those were `onlyOwner` functions). 

## Wiring

We'll now add the button to toggle the state of the lock. The wiring is exactly like the previous one, except that we'll now add the toggle button and connect it to **D3** and **GND** (make sure to use the opposite ends of the button). 


![Wiring](https://user-images.githubusercontent.com/77351244/188516279-ef196215-af33-41c2-b578-46350803caef.png)

## Opening / Closing the Lock 

All you'll have to do is press the button to toggle the status of the lock, the device will take care of changing the status of the physical lock and updating its corresponding value in the smart contract.

## Conclusions

These few modifications have allowed us to complete the smart-lock series. We can now control our device either remotely through the smart contract, or physically by using the toggle button. 

Applications like this, that are based on a direct connection between a device and a Layer-1 blockchain, come with some pros and cons. Feel free to browse through our developer portal to learn more about this and other related topics. 
