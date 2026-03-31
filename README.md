
# IR Remote Control for PC via ESP32

Control your PC using an IR remote via ESP32 and Bluetooth.

---

##  Required Hardware

- ESP32 (e.g., DevKit V1)  
- IR Sensor (e.g., VS1838B)  
- IR Remote Control  
- Jumper wires  

**Connections:**

| IR Sensor | ESP32 Pin |
|-----------|-----------|
| VCC       | 3.3V      |
| GND       | GND       |
| OUT       | D23       |

---

##  Required Software

- Arduino IDE or PlatformIO  
- Libraries:
  - `IRremoteESP8266` – to read IR signals  
  - `BluetoothSerial` – to send data to PC via Bluetooth  
  - Optional: `ESP32 BLE Keyboard` – to send key presses directly  

---

##  Usage

### 1. Reading IR Codes

Upload this sketch to your ESP32 to detect IR codes:

```cpp
#include <IRremote.h>
#include <BleKeyboard.h>

#define IR_RECEIVE_PIN 4

BleKeyboard bleKeyboard("ESP32 Remote", "ESP32", 100);

uint32_t lastCode = 0;
unsigned long lastTime = 0;

void setup() {
  Serial.begin(115200);
  delay(2000);

  Serial.println("PROGRAM STARTED");

  IrReceiver.begin(IR_RECEIVE_PIN, ENABLE_LED_FEEDBACK);
  bleKeyboard.begin();

  Serial.println("Bluetooth Ready");
}

void loop() {

  if (IrReceiver.decode()) {

    uint32_t code = IrReceiver.decodedIRData.decodedRawData;

    // anti-repeat
    if (code != lastCode || millis() - lastTime > 300) {

      Serial.print("Received code: ");
      Serial.println(code, HEX);

      if (bleKeyboard.isConnected()) {

        // ===== VOLUME =====
        if (code == 0xEA15FF00 || code == 0x00FF15EA) {
          bleKeyboard.write(KEY_MEDIA_VOLUME_UP);
        }

        if (code == 0xF807FF00 || code == 0x00FF07F8) {
          bleKeyboard.write(KEY_MEDIA_VOLUME_DOWN);
        }

        // ===== ARROW KEYS =====
        if (code == 0xE718FF00) {
          bleKeyboard.write(KEY_UP_ARROW);
        }

        if (code == 0xAD52FF00) {
          bleKeyboard.write(KEY_DOWN_ARROW);
        }

        if (code == 0xA55AFF00) {
          bleKeyboard.write(KEY_RIGHT_ARROW);
        }

        if (code == 0xF708FF00) {
          bleKeyboard.write(KEY_LEFT_ARROW);
        }

        // ===== ENTER =====
        if (code == 0xE31CFF00) {
          bleKeyboard.write(KEY_RETURN);
        }

        // ===== SUBTITLES (Ctrl + L) =====
        if (code == 0xB54AFF00) {
          bleKeyboard.press(KEY_LEFT_CTRL);
          bleKeyboard.press('l');
          delay(100);
          bleKeyboard.releaseAll();
        }

        // ===== F11 =====
        if (code == 0xA15EFF00) {
          bleKeyboard.write(KEY_F11);
        }

        // ===== WINDOWS KEY =====
        if (code == 0xBD42FF00) {
          bleKeyboard.write(KEY_LEFT_GUI);
        }

        // ===== ALT + F4 =====
        if (code == 0xBB44FF00) {   // set your desired code here
          bleKeyboard.press(KEY_LEFT_ALT);
          bleKeyboard.press(KEY_F4);
          delay(100);
          bleKeyboard.releaseAll();
        }
      }

      lastCode = code;
      lastTime = millis();
    }

    IrReceiver.resume();
  }
}
