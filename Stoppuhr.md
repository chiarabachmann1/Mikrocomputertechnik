# Stoppuhr
### Ziel
Das Ziel dieses Projekts war die Entwicklung einer einfachen Stoppuhr mit einem LCD-Display und einem Taster, um die Zeit zu starten und zu stoppen. Die gemessene Zeit wird auf dem Display angezeigt.

### Hardware
#### Benötigte Hardware
- Arduino Uno (Funduino)
- LCD 1602 mit I2C-Adapter
- Taster (für Start/Stopp-Funktion)
- Jumper-Kabel zur Verbindung der Komponenten

#### Verkabelung des LCD-Displays
| LCD-Pin (I2C-Modul) | Arduino-Pin | Funktion |
|-----------------|--------------|----------------------|
| **VCC** | **5V** | Stromversorgung |
| **GND** | **GND** | Masse |
| **SDA** | **A4** | I2C-Datenleitung |
| **SCL** | **A5** | I2C-Taktleitung |

#### Verkabelung des Tasters
| Taster-Pin | Arduino-Pin | Funktion |
|------------|--------------|----------------------|
| **1. Pin** | **D2** | Digitaleingang für Start/Stopp |
| **2. Pin** | **GND** | Masse |

### Code
#### Testcode für Anzeige des Displays
```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

#define LCD_ADDR 0x27  // Bestätigte I2C-Adresse

LiquidCrystal_I2C lcd(LCD_ADDR, 16, 2);

void setup() {
    Serial.begin(9600);
    Serial.println("LCD Test gestartet...");

    delay(2000);  // 2 Sekunden warten
    lcd.begin(16, 2);
    lcd.backlight();  
    lcd.clear();  

    lcd.setCursor(0, 0);
    lcd.print("LCD Funktioniert!");
}

void loop() {
}
```

#### Code für die Stoppuhr
```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

#define LCD_ADDR 0x27  // Falls 0x27 nicht funktioniert, probiere 0x3F

LiquidCrystal_I2C lcd(LCD_ADDR, 16, 2);  // LCD 16x2 erstellen

const int buttonPin = 2;
bool running = false;
unsigned long startTime = 0;
unsigned long elapsedTime = 0;

void setup() {
    pinMode(buttonPin, INPUT_PULLUP);
    Serial.begin(9600);
    
    lcd.begin(16, 2);  // Hier die richtige Anzahl Parameter übergeben!
    lcd.backlight();  // Hintergrundbeleuchtung einschalten
    
    lcd.setCursor(0, 0);
    lcd.print("Stoppuhr bereit!");
}

void loop() {
    if (digitalRead(buttonPin) == LOW) {  // Knopf gedrückt
        delay(200);  // Entprellung
        while (digitalRead(buttonPin) == LOW);  // Warten, bis losgelassen
        
        if (!running) {
            startTime = millis() - elapsedTime;  // Falls Pause, setze die Zeit korrekt
            running = true;
        } else {
            elapsedTime = millis() - startTime;  // Speichere aktuelle Zeit
            running = false;
        }
    }

    if (running) {
        elapsedTime = millis() - startTime;
    }

    int minutes = (elapsedTime / 60000) % 60;
    int seconds = (elapsedTime / 1000) % 60;
    int milliseconds = (elapsedTime % 1000) / 10;

    lcd.setCursor(0, 1);
    lcd.print("Zeit: ");
    lcd.print(minutes);
    lcd.print(":");
    if (seconds < 10) lcd.print("0");  // Führende Null für Sekunden
    lcd.print(seconds);
    lcd.print(".");
    if (milliseconds < 10) lcd.print("0");  // Führende Null für Millisekunden
    lcd.print(milliseconds);
}
```
![IMG_3820](https://github.com/user-attachments/assets/e2a04965-98d8-48d0-af25-18f7268cd66f)

### Probleme und Herausforderungen
- Das LCD-Display zeigte anfangs nur Kästchen, da die Hardware defekt war. Nach dem Austausch funktionierte es.
- Die I2C-Adresse musste zuerst mit einem Scanner-Code ermittelt werden (0x27 oder 0x3F).
- Die Verkabelung musste mehrfach überprüft und korrigiert werden, um eine fehlerfreie Kommunikation zwischen dem LCD und dem Arduino zu gewährleisten

