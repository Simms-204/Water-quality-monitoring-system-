#include <OneWire.h>
#include <DallasTemperature.h>
#include <SoftwareSerial.h>

// === DS18B20 Temp Sensor ===
#define ONE_WIRE_BUS 2
OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature tempSensor(&oneWire);

// === Analog Sensor Pins ===
#define PH_SENSOR_PIN A0
#define TURBIDITY_SENSOR_PIN A1

// === ESP8266 Communication ===
SoftwareSerial espSerial(10, 11); // RX, TX
String ssid = "Your_SSID";
String password = "Your_PASSWORD";
String apiKey = "Your_API_KEY";

void setupWiFi() {
  espSerial.begin(9600);
  delay(1000);

  sendCommand("AT", 2000);
  sendCommand("AT+CWMODE=1", 2000);
  sendCommand("AT+CWJAP=\"" + ssid + "\",\"" + password + "\"", 5000);
}

String sendCommand(String command, const int timeout) {
  String response = "";
  espSerial.println(command);
  long int time = millis();
  while ((time + timeout) > millis()) {
    while (espSerial.available()) {
      char c = espSerial.read();
      response += c;
    }
  }
  Serial.println(response);
  return response;
}

void sendDataToThingSpeak(float temp, int phRaw, int turbidityRaw) {
  String cmd = "AT+CIPSTART=\"TCP\",\"api.thingspeak.com\",80";
  sendCommand(cmd, 3000);

  String getStr = "GET /update?api_key=" + apiKey + 
                  "&field1=" + String(temp) +
                  "&field2=" + String(phRaw) +
                  "&field3=" + String(turbidityRaw) + " \r\n";

  cmd = "AT+CIPSEND=" + String(getStr.length() + 4);
  sendCommand(cmd, 2000);
  espSerial.println(getStr);
}

void setup() {
  Serial.begin(9600);
  tempSensor.begin();
  setupWiFi();
  delay(2000);
  Serial.println("Water Quality Monitoring System Started");
}

void loop() {
  tempSensor.requestTemperatures();
  float temperatureC = tempSensor.getTempCByIndex(0);
  int phValue = analogRead(PH_SENSOR_PIN);
  int turbidityValue = analogRead(TURBIDITY_SENSOR_PIN);

  Serial.print("Temp (°C): "); Serial.println(temperatureC);
  Serial.print("pH Raw: "); Serial.println(phValue);
  Serial.print("Turbidity Raw: "); Serial.println(turbidityValue);
  
  sendDataToThingSpeak(temperatureC, phValue, turbidityValue);

  delay(20000);  // 20-second delay for ThingSpeak
}
