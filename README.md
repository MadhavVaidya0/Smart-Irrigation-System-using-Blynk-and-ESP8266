# Smart-Irrigation-System-using-Blynk-and-ESP8266

code

#define BLYNK_TEMPLATE_ID "TMPL34-T5ZU6d"
#define BLYNK_TEMPLATE_NAME "IrrigationSystem"
#define BLYNK_AUTH_TOKEN "MH2eGc_Ko2Sk8aL42SrxiAJPFvJxMWu8"
#define BLYNK_PRINT Serial
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <DHT.h>

char auth[] = "MH2eGc_Ko2Sk8aL42SrxiAJPFvJxMWu8"; // Enter your Auth token
char ssid[] = "Pankaj"; // Enter your WIFI name
char pass[] = "pankaj@007"; // Enter your WIFI password

DHT dht(D4, DHT11); // DHT sensor is connected to pin D4

BlynkTimer timer;

void setup() {
  Serial.begin(9600);
  pinMode(D3, OUTPUT);
  digitalWrite(D3, HIGH);

  Blynk.begin(auth, ssid, pass, "blynk.cloud", 80);

  // Call the functions
  timer.setInterval(100L, soilMoistureSensor);
  timer.setInterval(2000L, readDHTSensor); // Update DHT readings every 2 seconds
  timer.setInterval(1000L, printData); // Print data every 1 second
}

// Get the button value
BLYNK_WRITE(V1) {
  int Relay = param.asInt();

  if (Relay == 1) {
    digitalWrite(D3, LOW);
  } else {
    digitalWrite(D3, HIGH);
  }
}

// Get the soil moisture values
void soilMoistureSensor() {
  int value = analogRead(A0);
  value = map(value, 0, 1024, 0, 100);
  value = (value - 100) * -1;

  Blynk.virtualWrite(V0, value);
}

// Read DHT sensor values
void readDHTSensor() {
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  if (!isnan(temperature) && !isnan(humidity)) {
    // Send temperature in degrees Celsius to Blynk V2
    Blynk.virtualWrite(V2, temperature);

    // Send humidity in percentage to Blynk V3
    Blynk.virtualWrite(V3, humidity);
  } else {
    Serial.println("Failed to read from DHT sensor!");
  }
}

// Print soil moisture, temperature, and humidity
void printData() {
  int soilMoisture = analogRead(A0);
  soilMoisture = map(soilMoisture, 0, 1024, 0, 100);
  soilMoisture = (soilMoisture - 100) * -1;

  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  Serial.print("Soil Moisture: ");
  Serial.print(soilMoisture);
  Serial.print(", Temperature: ");
  Serial.print(temperature);
  Serial.print(" °C, Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");
}

void loop() {
  Blynk.run(); // Run the Blynk library
  timer.run(); // Run the Blynk timer
}
