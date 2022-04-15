-
// Include Libraries
#include "Arduino.h"
#include "DHT.h"
#include "ESP8266.h"
#include "Relay.h"
#include "SoilMoisture.h"
#include "Pump.h"


// Pin Definitions
#define DHT_PIN_DATA	2
#define WIFI_PIN_TX	11
#define WIFI_PIN_RX	10
#define RELAYMODULE_PIN_SIGNAL	3
#define SOILMOISTURE_5V_PIN_SIG	A3
#define WATERPUMP_PIN_COIL1	5



// Global variables and defines
// ====================================================================
// vvvvvvvvvvvvvvvvvvvv ENTER YOUR WI-FI SETTINGS  vvvvvvvvvvvvvvvvvvvv
//
const char *SSID     = "WIFI-SSID"; // Enter your Wi-Fi name 
const char *PASSWORD = "PASSWORD" ; // Enter your Wi-Fi password
//
// ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
// ====================================================================
char* const host = "www.google.com";
int hostPort = 80;
// object initialization
DHT dht(DHT_PIN_DATA);
ESP8266 wifi(WIFI_PIN_RX,WIFI_PIN_TX);
Relay relayModule(RELAYMODULE_PIN_SIGNAL);
SoilMoisture soilMoisture_5v(SOILMOISTURE_5V_PIN_SIG);
Pump waterpump(WATERPUMP_PIN_COIL1);


// define vars for testing menu
const int timeout = 10000;       //define timeout of 10 sec
char menuOption = 0;
long time0;

// Setup the essentials for your circuit to work. It runs first every time your circuit is powered with electricity.
void setup() 
{
    // Setup Serial which is useful for debugging
    // Use the Serial Monitor to view printed messages
    Serial.begin(9600);
    while (!Serial) ; // wait for serial port to connect. Needed for native USB
    Serial.println("start");
    
    dht.begin();
    wifi.init(SSID, PASSWORD);
    menuOption = menu();
    
}

// Main logic of your circuit. It defines the interaction between the components you selected. After setup, it runs over and over again, in an eternal loop.
void loop() 
{
    
    
    if(menuOption == '1') {
    // DHT22/11 Humidity and Temperature Sensor - Test Code
    // Reading humidity in %
    float dhtHumidity = dht.readHumidity();
    // Read temperature in Celsius, for Fahrenheit use .readTempF()
    float dhtTempC = dht.readTempC();
    Serial.print(F("Humidity: ")); Serial.print(dhtHumidity); Serial.print(F(" [%]\t"));
    Serial.print(F("Temp: ")); Serial.print(dhtTempC); Serial.println(F(" [C]"));

    }
    else if(menuOption == '2') {
    // ESP8266-01 - Wifi Module - Test Code
    //Send request for www.google.com at port 80
    wifi.httpGet(host, hostPort);
    // get response buffer. Note that it is set to 250 bytes due to the Arduino low memory
    char* wifiBuf = wifi.getBuffer();
    //Comment out to print the buffer to Serial Monitor
    //for(int i=0; i< MAX_BUFFER_SIZE ; i++)
    //  Serial.print(wifiBuf[i]);
    //search buffer for the date and time and print it to the serial monitor. This is GMT time!
    char *wifiDateIdx = strstr (wifiBuf, "Date");
    for (int i = 0; wifiDateIdx[i] != '\n' ; i++)
    Serial.print(wifiDateIdx[i]);

    }
    else if(menuOption == '3') {
    // Relay Module - Test Code
    // The relay will turn on and off for 500ms (0.5 sec)
    relayModule.on();       // 1. turns on
    delay(500);             // 2. waits 500 milliseconds (0.5 sec). Change the value in the brackets (500) for a longer or shorter delay in milliseconds.
    relayModule.off();      // 3. turns off.
    delay(500);             // 4. waits 500 milliseconds (0.5 sec). Change the value in the brackets (500) for a longer or shorter delay in milliseconds.
    }
    else if(menuOption == '4') {
    // Soil Moisture Sensor - Test Code
    int soilMoisture_5vVal = soilMoisture_5v.read();
    Serial.print(F("Val: ")); Serial.println(soilMoisture_5vVal);

    }
    else if(menuOption == '5') {
    // Submersible Pool Water Pump - Test Code
    // The water pump will turn on and off for 2000ms (4 sec)
    waterpump.on(); // 1. turns on
    delay(2000);       // 2. waits 500 milliseconds (0.5 sec).
    waterpump.off();// 3. turns off
    delay(2000);       // 4. waits 500 milliseconds (0.5 sec).

    }
    
    if (millis() - time0 > timeout)
    {
        menuOption = menu();
    }
    
}



// Menu function for selecting the components to be tested
// Follow serial monitor for instrcutions
char menu()
{

    Serial.println(F("\nWhich component would you like to test?"));
    Serial.println(F("(1) DHT22/11 Humidity and Temperature Sensor"));
    Serial.println(F("(2) ESP8266-01 - Wifi Module"));
    Serial.println(F("(3) Relay Module"));
    Serial.println(F("(4) Soil Moisture Sensor"));
    Serial.println(F("(5) Submersible Pool Water Pump"));
    Serial.println(F("(menu) send anything else or press on board reset button\n"));
    while (!Serial.available());

    // Read data from serial monitor if received
    while (Serial.available()) 
    {
        char c = Serial.read();
        if (isAlphaNumeric(c)) 
        {   
            
            if(c == '1') 
    			Serial.println(F("Now Testing DHT22/11 Humidity and Temperature Sensor"));
    		else if(c == '2') 
    			Serial.println(F("Now Testing ESP8266-01 - Wifi Module"));
    		else if(c == '3') 
    			Serial.println(F("Now Testing Relay Module"));
    		else if(c == '4') 
    			Serial.println(F("Now Testing Soil Moisture Sensor"));
    		else if(c == '5') 
    			Serial.println(F("Now Testing Submersible Pool Water Pump"));
            else
            {
                Serial.println(F("illegal input!"));
                return 0;
            }
            time0 = millis();
            return c;
        }
    }
}
