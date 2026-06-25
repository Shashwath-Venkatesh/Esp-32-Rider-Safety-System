#include <Wire.h> 
#include <Adafruit_MPU6050.h> 
#include <Adafruit_Sensor.h> 
#include <TinyGPS++.h> 
Adafruit_MPU6050 mpu; 
TinyGPSPlus gps; 
HardwareSerial sim800(2); 
HardwareSerial gpsSerial(1); 
#define OVERRIDE 12 
#define BUZZER 14 
#define FOOTPEG1 32 
#define FOOTPEG2 33 
String riderName = "RIDER NAME"; 
String bloodGroup = "O+"; 
String phoneNumber = "+91XXXXXXXXXX";  // <<< CHANGE NUMBER 
float baseAngle = 102.84;  // Upright normal angle 
float fallThreshold = 45.0; // degrees lt to detect fall 
void setup() { 
Serial.begin(115200); 
pinMode(OVERRIDE, INPUT_PULLUP); 
pinMode(BUZZER, OUTPUT); 
pinMode(FOOTPEG1, INPUT_PULLUP); 
pinMode(FOOTPEG2, INPUT_PULLUP); 
Wire.begin(21, 22); 
mpu.begin(); 
sim800.begin(9600, SERIAL_8N1, 16, 17); 
gpsSerial.begin(9600, SERIAL_8N1, 4, 15); 
delay(2000); 
Serial.println("SYSTEM INITIALIZED"); 
Serial.println("==============================="); 
} 
void loop() { 
unsigned long cycleStart = millis(); 
int fallCount = 0; 
int totalReadings = 0; 
  float initLat = 0, initLng = 0; 
 
  Serial.println("\nReading sensors for 10 seconds..."); 
 
  while (millis() - cycleStart < 10000) { 
 
    readGPS(); 
 
    float angle = getTiltAngle(); 
    float lt = abs(angle - baseAngle); 
 
    int foot1 = digitalRead(FOOTPEG1); 
    int foot2 = digitalRead(FOOTPEG2); 
 
    if (gps.loca on.isValid()) { 
      initLat = gps.loca on.lat(); 
      initLng = gps.loca on.lng(); 
    } 
 
    Serial.print("Angle: "); 
    Serial.print(angle); 
    Serial.print("  Tilt: "); 
    Serial.print( lt); 
    Serial.print("  Foot1: "); 
    Serial.print(foot1); 
    Serial.print("  Foot2: "); 
    Serial.println(foot2); 
 
    if ( lt > fallThreshold) { 
      fallCount++; 
    } 
 
    totalReadings++; 
    delay(500); 
  } 
 
  float fallRa o = (float)fallCount / totalReadings; 
 
  Serial.print("Fall Ra o: "); 
  Serial.println(fallRa o); 
 
  if (fallRa o > 0.6) { 
    triggerAlert("FALL DETECTED", initLat, initLng); 
  } 
 
  Serial.println("Wai ng 20 seconds before next cycle..."); 
  delay(20000);  // completes 30 sec cycle 
} 
 
float getTiltAngle() { 
  sensors_event_t a, g, temp; 
  mpu.getEvent(&a, &g, &temp); 
  float angle = atan2(a.accelera on.y, a.accelera on.z) * 180 / PI; 
  return angle; 
} 
 
void triggerAlert(String type, float initLat, float initLng) { 
 
  Serial.println("\n================================="); 
  Serial.println("ALERT DETECTED: " + type); 
  Serial.println("Press override bu on within 10 seconds to cancel."); 
  Serial.println("================================="); 
 
  for (int i = 0; i < 3; i++) { 
    tone(BUZZER, 2000); 
    delay(300); 
    noTone(BUZZER); 
    delay(300); 
  } 
 
  unsigned long startWait = millis(); 
 
  while (millis() - startWait < 10000) { 
    if (digitalRead(OVERRIDE) == LOW) { 
      delay(200); 
      if (digitalRead(OVERRIDE) == LOW) { 
        Serial.println("ALERT CANCELLED"); 
        return; 
      } 
    } 
  } 
 
  sendSMS(type, initLat, initLng); 
} 
 
void sendSMS(String status, float initLat, float initLng) { 
 
  Serial.println("\n------ SMS PREVIEW ------"); 
 
  readGPS(); 
 
  float finalLat = gps.loca on.lat(); 
  float finalLng = gps.loca on.lng(); 
 
  String message = ""; 
  message += "Rider: " + riderName + "\n"; 
  message += "Blood Group: " + bloodGroup + "\n"; 
  message += status + "\n\n"; 
 
  message += "Ini al Loca on:\n"; 
  if (initLat != 0) { 
    message += "h ps://maps.google.com/?q=" + String(initLat,6) + "," + 
String(initLng,6) + "\n"; 
  } else { 
    message += "GPS Not Fixed\n"; 
  } 
 
  message += "\nFinal Loca on:\n"; 
  if (gps.loca on.isValid()) { 
    message += "h ps://maps.google.com/?q=" + String(finalLat,6) + "," + 
String(finalLng,6) + "\n"; 
  } else { 
    message += "GPS Not Fixed\n"; 
  } 
 
  message += "\nDate & Time:\n"; 
  if (gps.date.isValid() && gps. me.isValid()) { 
    message += String(gps.date.day()) + "/" + 
               String(gps.date.month()) + "/" + 
               String(gps.date.year()) + " "; 
    message += String(gps. me.hour()) + ":" + 
               String(gps. me.minute()); 
  } else { 
    message += "Not Available"; 
  } 
 
  Serial.println(message); 
  Serial.println("-------------------------\n"); 
 
  Serial.print("Sending SMS to: "); 
  Serial.println(phoneNumber); 
 
sim800.println("AT+CMGF=1"); 
delay(1000); 
sim800.print("AT+CMGS=\""); 
sim800.print(phoneNumber); 
sim800.println("\""); 
delay(1000); 
sim800.print(message); 
delay(500); 
sim800.write(26);  // CTRL+Z 
delay(5000); 
Serial.println("SMS SENT SUCCESSFULLY"); 
} 
void readGPS() { 
while (gpsSerial.available()) { 
gps.encode(gpsSerial.read()); 
} 
} 
Code output (Execution): 
