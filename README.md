//# Microgreen_Farming_Using_IOT_and_its_application
//The development of an IoT-enabled automation system for microgreen farming aims to improve efficiency and productivity, reduce manual effort, and increase yield and quality of microgreens using hydroponic methods. Incorporating IoT //technology in agriculture can enhance crop production, minimize reso
//This Code is used for micogreen farming using iot and its application .
#include <DHT.h>//Code Written by Abhishek Singh
#include <ThingSpeak.h>
#include <ESP8266WiFi.h>
String apiKey = "1G8PO42RMLPB39EF"; // Enter your Write API key here
const char* server = "api.thingspeak.com";
const char *ssid = "Abhisingh"; // Enter your WiFi Name
const char *pass = "Abhi@2002"; // Enter your WiFi Password
#define DHTPIN D3 // GPIO Pin where the dht11 is connected
DHT dht(DHTPIN, DHT11);
WiFiClient client;

const int moisturePin = A0; // moisteure sensor pin
const int motorPin = D0;
unsigned long interval = 1000;
unsigned long previousMillis = 0;
unsigned long interval1 = 1000;
unsigned long previousMillis1 = 0;
float moisturePercentage; //moisture reading
float h; // humidity reading
float t; //temperature reading

void setup()
{
Serial.begin(115200);
delay(10);
pinMode(motorPin, OUTPUT);
digitalWrite(motorPin, LOW); // keep motor off initally
dht.begin();
Serial.println("Connecting to ");
Serial.println(ssid);
WiFi.begin(ssid, pass);
while (WiFi.status() != WL_CONNECTED)
{
delay(500);
Serial.print("."); // print ... till not connected
}
Serial.println("");
Serial.println("WiFi connected");
}

void loop()
{
unsigned long currentMillis = millis(); // grab current time

h = dht.readHumidity(); // read humiduty
t = dht.readTemperature(); // read temperature

if (isnan(h) || isnan(t))
{
Serial.println("Failed to read from DHT sensor!");
return;
}

moisturePercentage = 2*( 100.00 - ( (analogRead(moisturePin) / 1023.00) * 100.00 ) );

if ((unsigned long)(currentMillis - previousMillis1) >= interval1) {
Serial.print("Soil Moisture is = ");
Serial.print(moisturePercentage);
Serial.println("%");
previousMillis1 = millis();
}

if (moisturePercentage < 30) {
digitalWrite(motorPin, HIGH); // tun on motor

}
if (moisturePercentage > 30 && moisturePercentage < 35) {
digitalWrite(motorPin, HIGH); //turn on motor pump

}
if (moisturePercentage > 36) {
digitalWrite(motorPin, LOW); // turn off mottor

}

if ((unsigned long)(currentMillis - previousMillis) >= interval) {

sendThingspeak(); //send data to thing speak
previousMillis = millis();
client.stop();
}

}

void sendThingspeak() {
if (client.connect(server, 80))
{
String postStr = apiKey; // add api key in the postStr string
postStr += "&field1=";
postStr += String(moisturePercentage); // add mositure readin
postStr += "&field2=";
postStr += String(t); // add tempr readin
postStr += "&field3=";
postStr += String(h); // add humidity readin
postStr += "\r\n\r\n";

client.print("POST /update HTTP/1.1\n");
client.print("Host: api.thingspeak.com\n");
client.print("Connection: close\n");
client.print("X-THINGSPEAKAPIKEY: " + apiKey + "\n");
client.print("Content-Type: application/x-www-form-urlencoded\n");
client.print("Content-Length: ");
client.print(postStr.length());           //send lenght of the string
client.print("\n\n");
client.print(postStr);                      // send complete string
Serial.print("Moisture Percentage: ");
Serial.print(moisturePercentage);
Serial.print("%. Temperature: ");
Serial.print(t);
Serial.print(" C, Humidity: ");
Serial.print(h);
Serial.println("%. Sent to Thingspeak.");
}
delay(500);
client.stop();
Serial.println("Waiting...");

// thingspeak needs minimum 15 sec delay between updates.
delay(1500);
}
