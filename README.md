# Classroom-Monitoring
A smart study environment was created by integrating sensors like BH1750, Ultrasonic and DHT11 with ESP32, enabling adaptive learning and seamless control through the BLYNK app.
<br>
Author - Pratiksha Hosalli

https://drive.google.com/drive/folders/19A2TuhRa8wSy4utDoQOLhOICklCC4lg0

#include <WiFi.h>
#include <WiFiClient.h>
#define BLYNK_PRINT Serial

#define BLYNK_TEMPLATE_ID "TMPLWSJgZmOD"
#define BLYNK_DEVICE_NAME "smart study environment"
#define BLYNK_AUTH_TOKEN "ypa3NWiyKM4r8tAZhmjUet8eMTGNm8hH"

#include <BlynkSimpleEsp32.h>

#include "DHT.h"

#define DHTPIN 4
#define DHTTYPE DHT11

int Led_pin1 = 13;
int Led_pin2 = 14;

#include <Wire.h>
#include <BH1750.h>

char auth[] = "V63DJnlbrWQv_XIxYczYhfsrmQmp7mfo";
const char* ssid = "Pranav K";
const char* password = "55550000";

WiFiClient  client;

DHT dht(DHTPIN, DHTTYPE);

BH1750 lightMeter;

const int trigPin = 5;
const int echoPin = 18;

long duration;
int distance;

BLYNK_WRITE(V2)
{
  int pinValue = param.asInt();
  if(pinValue==1)
  Led_glow(Led_pin1);
  else
  Led_noglow(Led_pin1);
}
BLYNK_WRITE(V3)
{
  int pinValue = param.asInt();
  if(pinValue==1)
  Led_glow(Led_pin2);
  else
  Led_noglow(Led_pin2);
}

void Led_glow(int x)
{
  digitalWrite(x, HIGH);
}
void Led_noglow(int x)
{
  digitalWrite(x, LOW);
}

void setup() {
  Serial.begin(9600);

  Serial.println("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
  delay(1000);
  Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected..!");
  Serial.print("Got IP: ");  
  Serial.println(WiFi.localIP());

  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, password);
   
  Serial.println(F("DHTxx test!"));

  dht.begin();

  Wire.begin();
  lightMeter.begin();
  Serial.println(F("BH1750 Test begin"));

  pinMode(trigPin, OUTPUT); 
  pinMode(echoPin, INPUT); 

  pinMode(Led_pin1, OUTPUT);
  pinMode(Led_pin2, OUTPUT);
}
void loop() {

  delay(2000);

  float h = dht.readHumidity();
  float t = dht.readTemperature();

  if (isnan(h) || isnan(t)) 
  {
    Serial.println(F("Failed to read from DHT sensor!"));
    return;
  }

  float lux = lightMeter.readLightLevel();

  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
 
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
 
  duration = pulseIn(echoPin, HIGH);
  
  distance = duration * 0.034 / 2;

  if(distance<40)
  Led_glow(Led_pin1);
  if(lux<40)
  Led_glow(Led_pin1);
  else if(lux>150)
  Led_noglow(Led_pin1);

  Blynk.run();

  Blynk.virtualWrite(V0, lux);
  Blynk.virtualWrite(V1, t);
  
  Serial.print(F("Humidity: "));
  Serial.print(h);
  Serial.print(F("%  Temperature: "));
  Serial.print(t);
  Serial.println(F("°C "));
  
  Serial.print("Light: ");
  Serial.print(lux);
  Serial.println(" lx");

  Serial.print("Distance: ");
  Serial.println(distance);
  Serial.println("");
}
