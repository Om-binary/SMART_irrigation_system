#define BLYNK_TEMPLATE_ID "TMPL3f2OlgjwK"

#define BLYNK_TEMPLATE_NAME "SMART Irrigation"

#define BLYNK_AUTH_TOKEN "u9cymSYbYmI8S9jBx0pZBR4eKC66Z5yR"

#include <ESP8266WiFi.h>

#include <BlynkSimpleEsp8266.h>

#define SOIL_SENSOR_PIN A0 

#define RELAY_PIN D1 

char auth[] = "u9cymSYbYmI8S9jBx0pZBR4eKC66Z5yR";

char ssid[] = "SVERI-CISCO";

char pass[] = "Cisco@123";

#define VIRTUAL_MOISTURE_PERCENT V0 

#define VIRTUAL_MANUAL_MODE V1 

#define VIRTUAL_MANUAL_CONTROL V3 

#define VIRTUAL_SOIL_MOISTURE V2 

int soilMoistureValue = 0;

int soilMoisturePercent = 0;

bool isManualMode = false;

bool isMotorOn = false;

int moistureThresholdLow = 30; 

int moistureThresholdHigh = 70; 

BlynkTimer timer;

void readSoilMoisture() {

soilMoistureValue = analogRead(SOIL_SENSOR_PIN); 

soilMoisturePercent = map(soilMoistureValue, 1023, 0, 0, 100); 

Blynk.virtualWrite(VIRTUAL_MOISTURE_PERCENT, soilMoisturePercent);

Blynk.virtualWrite(VIRTUAL_SOIL_MOISTURE, soilMoistureValue);if (!isManualMode) {

if (soilMoisturePercent < moistureThresholdLow && !isMotorOn) {

digitalWrite(RELAY_PIN, LOW); 

isMotorOn = true;

} else if (soilMoisturePercent > moistureThresholdHigh && isMotorOn) {

digitalWrite(RELAY_PIN, HIGH); 

isMotorOn = false;

}

}

}

BLYNK_WRITE(VIRTUAL_MANUAL_MODE) {

isManualMode = param.asInt(); 

if (!isManualMode && isMotorOn) {

digitalWrite(RELAY_PIN, HIGH); 

isMotorOn = false;

}}

BLYNK_WRITE(VIRTUAL_MANUAL_CONTROL) {

if (isManualMode) {

int manualMotorState = param.asInt(); 

if (manualMotorState == 1 && !isMotorOn) {

digitalWrite(RELAY_PIN, LOW); 

isMotorOn = true;

} else if (manualMotorState == 0 && isMotorOn) {

digitalWrite(RELAY_PIN, HIGH); 

isMotorOn = false;

}

}}void setup() {

Serial.begin(115200);

Blynk.begin(auth, ssid, pass);

pinMode(RELAY_PIN, OUTPUT);

digitalWrite(RELAY_PIN, HIGH); 

timer.setInterval(2000L, readSoilMoisture);

}

void loop() {

Blynk.run(); 

timer.run(); 

}
