#include <SoftwareSerial.h>

// Define pins for LiDAR communication (if using software serial)
SoftwareSerial LiDarSerial(2, 3); // RX, TX

int distance = 0; // Distance measured by LiDAR in cm
int strength = 0; // Signal strength of LiDAR
unsigned char tfData[9]; // Data buffer from TF-Luna

// Target safety distance in cm (e.g., don't drop below 50cm)
const int safetyDistance = 50; 

void setup() {
  Serial.begin(115200);      // Hardware serial for debugging
  LiDarSerial.begin(115200); // TF-Luna defaults to 115200 baud
  
  Serial.println("LiDAR Microdrone Brain Initialized...");
}

void loop() {
  if (readLiDAR()) {
    Serial.print("Distance: ");
    Serial.print(distance);
    Serial.println(" cm");

    // --- DECISION MAKING LOGIC ---
    if (distance < safetyDistance && distance > 0) {
      Serial.println("⚠️ TOO CLOSE! Sending emergency throttle/pitch to Flight Controller...");
      
      // Calculate how hard to push back based on proximity
      int correctionFactor = map(distance, 10, safetyDistance, 255, 0);
      
      sendCorrectionToFlightController(correctionFactor);
    } else {
      // Allow normal pilot control
      maintainNormalFlight();
    }
  }
  delay(20); // LiDAR updates at roughly 50Hz-100Hz
}

// Function to parse TF-Luna LiDAR standard data packet (9 bytes)
bool readLiDAR() {
  while (LiDarSerial.available() >= 9) {
    if (LiDarSerial.read() == 0x59) { // First header byte
      if (LiDarSerial.read() == 0x59) { // Second header byte
        
        for (int i = 2; i < 9; i++) {
          tfData[i] = LiDarSerial.read();
        }
        
        // Calculate distance (Low Byte + High Byte)
        distance = tfData[2] + tfData[3] * 256;
        // Calculate signal strength (indicates if reading is reliable)
        strength = tfData[4] + tfData[5] * 256; 
        
        return true;
      }
    }
  }
  return false;
}

void sendCorrectionToFlightController(int correction) {
  // In a finished build, you would write MSP protocol packets here
  // to fake a high-throttle or reverse-pitch command over Serial 
  // to Betaflight/Cleanflight.
}

void maintainNormalFlight() {
  // Pass-through regular receiver signals to the Flight Controller
}
