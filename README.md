# Flood-detector-with-sms-alert
#include <SoftwareSerial.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// Define GSM module pins
SoftwareSerial gsmSerial(D7, D8); // RX, TX

// LCD setup (I2C address 0x27)
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Float Sensor pin
const int floatSensorPin = D5;

// State tracking
bool alertSent = false;

void setup() {
  pinMode(floatSensorPin, INPUT);
  Serial.begin(9600);
  gsmSerial.begin(9600);

  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Flood Monitor");
  delay(2000);
}

void loop() {
  int waterLevel = digitalRead(floatSensorPin);
  
  lcd.clear();
  lcd.setCursor(0, 0);

  if (waterLevel == HIGH) {
    lcd.print("Water Level: HIGH");
    if (!alertSent) {
      sendSMS("ALERT: Water level HIGH! Possible Flood Risk!");
      alertSent = true;
    }
  } else {
    lcd.print("Water Level: LOW ");
    alertSent = false; // Reset when level is safe
  }

  delay(2000); // Check every 2 seconds
}

void sendSMS(String message) {
  gsmSerial.println("AT+CMGF=1"); // Set SMS mode to text
  delay(1000);
  gsmSerial.println("AT+CMGS=\"+91XXXXXXXXXX\""); // Replace with your phone number
  delay(1000);
  gsmSerial.println(message);
  delay(500);
  gsmSerial.write(26); // ASCII code of CTRL+Z to send SMS
  delay(3000);
}
