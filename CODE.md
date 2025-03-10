```
#include <DHT.h>

// Define the pin connected to the DHT11 sensor
#define DHTPIN 15
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE); // Initialize DHT sensor

// Relay pin definitions
const int relay1 = 23;
const int relay2 = 22;
const int relay3 = 21;
const int relay4 = 19;

float targetTemperature = -1; // User-specified target temperature
unsigned long lastReadTime = 0; // Last sensor read time
const unsigned long readInterval = 5000; // 5-second interval for sensor reading

void setup() {
  // Configure relay pins as output
  pinMode(relay1, OUTPUT);
  pinMode(relay2, OUTPUT);
  pinMode(relay3, OUTPUT);
  pinMode(relay4, OUTPUT);

  // Ensure all relays are off initially
  resetAllRelays();

  // Start Serial communication
  Serial.begin(115200);
  Serial.println("System initialized.");

  // Start DHT sensor
  dht.begin();
}

void loop() {
  // Handle periodic sensor reading
  unsigned long currentMillis = millis();
  if (currentMillis - lastReadTime >= readInterval) {
    lastReadTime = currentMillis;
    readSensorData();
  }

  // Handle Serial commands
  if (Serial.available() > 0) {
    handleSerialCommands();
  }
}

void readSensorData() {
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.print("%  Temperature: ");
  Serial.print(temperature);
  Serial.println(" °C");

  // Auto-control relay4 based on target temperature
  if (targetTemperature >= 0) {
    digitalWrite(relay4, (temperature < targetTemperature) ? HIGH : LOW);
    Serial.print("Relay 4 set to: ");
    Serial.println((temperature < targetTemperature) ? "ON" : "OFF");
  }
}

void handleSerialCommands() {
  String command = Serial.readStringUntil('\n');
  command.trim(); // Remove any extra whitespace

  if (command == "ON L") {
    setRelayState(HIGH, LOW, LOW);
    Serial.println("Relay 1 (LOW) ON");
  } else if (command == "ON M") {
    setRelayState(LOW, HIGH, LOW);
    Serial.println("Relay 2 (MIDDLE) ON");
  } else if (command == "ON H") {
    setRelayState(LOW, LOW, HIGH);
    Serial.println("Relay 3 (HIGH) ON");
  } else if (command == "OFF") {
    resetAllRelays();
    Serial.println("All relays OFF");
  } else if (command.startsWith("SET_TEMP")) {
    if (command.length() > 9) {
      targetTemperature = command.substring(9).toFloat();
      Serial.print("Target temperature set to: ");
      Serial.println(targetTemperature);
    } else {
      Serial.println("Error: No temperature value provided.");
    }
  } else if (command == "STATUS") {
    reportStatus();
  } else {
    Serial.println("Invalid Command!");
  }
}

void setRelayState(int relay1State, int relay2State, int relay3State) {
  // Safely toggle relay states
  digitalWrite(relay1, LOW);
  digitalWrite(relay2, LOW);
  digitalWrite(relay3, LOW);
  delay(50); // Ensure relays are fully reset before setting new states

  digitalWrite(relay1, relay1State);
  delay(50);
  digitalWrite(relay2, relay2State);
  delay(50);
  digitalWrite(relay3, relay3State);
}

void resetAllRelays() {
  digitalWrite(relay1, LOW);
  digitalWrite(relay2, LOW);
  digitalWrite(relay3, LOW);
  digitalWrite(relay4, LOW);
}

void reportStatus() {
  Serial.println("----- System Status -----");
  Serial.print("Target Temperature: ");
  Serial.println((targetTemperature >= 0) ? targetTemperature : -1);

  Serial.print("Relay States: ");
  Serial.print("Relay1: "); Serial.print(digitalRead(relay1));
  Serial.print(", Relay2: "); Serial.print(digitalRead(relay2));
  Serial.print(", Relay3: "); Serial.print(digitalRead(relay3));
  Serial.print(", Relay4: "); Serial.println(digitalRead(relay4));

  Serial.println("-------------------------");
}
