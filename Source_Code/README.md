#include <WiFi.h>
#include "ThingSpeak.h"
#include "HX711.h"

const char* ssid = "Wokwi-GUEST";
const char* password = "";

WiFiClient client;

unsigned long channelID = 3418063;
const char * writeAPIKey = "L7ZGTJLCK7YY6WC2";

const int DT_PIN = 4;
const int SCK_PIN = 5;

HX711 scale;

void setup() {

  Serial.begin(115200);

  WiFi.begin(ssid, password);

  Serial.print("Connecting to WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("WiFi Connected!");

  ThingSpeak.begin(client);

  scale.begin(DT_PIN, SCK_PIN);
  scale.set_scale(420.0);
  scale.tare();

  Serial.println("Smart Load Monitoring System Started");
}

void loop() {

  float weight = scale.get_units(10);

  String status;
  int statusCode;

  if (weight > 40) {
    status = "OVERLOAD";
    statusCode = 2;
  }
  else if (weight < 5) {
    status = "LOW STOCK";
    statusCode = 0;
  }
  else {
    status = "NORMAL";
    statusCode = 1;
  }

  Serial.print("Weight: ");
  Serial.print(weight);
  Serial.print(" kg   Status: ");
  Serial.println(status);

  ThingSpeak.setField(1, weight);
  ThingSpeak.setField(2, statusCode);

  int response = ThingSpeak.writeFields(channelID, writeAPIKey);

  if(response == 200){
    Serial.println("Data uploaded to ThingSpeak");
  } else {
    Serial.print("Upload failed. Error code: ");
    Serial.println(response);
  }

  Serial.println("-------------------------");

  delay(16000); // ThingSpeak requires at least 15 seconds
}
