# Stoppuhr
## Ziel
Das Ziel dieses Projekts war die Entwicklung einer einfachen Stoppuhr mit einem LCD-Display und einem Taster, um die Zeit zu starten und zu stoppen. Die gemessene Zeit wird auf dem Display angezeigt.

![IMG_3820](https://github.com/user-attachments/assets/e2a04965-98d8-48d0-af25-18f7268cd66f)
Abbildung: Mein fertiges Projekt

## Benötigte Hardware
- Arduino Uno (Funduino)
- LCD 1602 mit I2C-Adapter
- Taster (für Start/Stopp-Funktion)
- Jumper-Kabel zur Verbindung der Komponenten

## Umsetzung
### Verkabelung des LCD-Displays
| LCD-Pin (I2C-Modul) | Arduino-Pin | Funktion |
|-----------------|--------------|----------------------|
| **VCC** | **5V** | Stromversorgung |
| **GND** | **GND** | Masse |
| **SDA** | **A4** | I2C-Datenleitung |
| **SCL** | **A5** | I2C-Taktleitung |

### Verkabelung des Tasters
| Taster-Pin | Arduino-Pin | Funktion |
|------------|--------------|----------------------|
| **1. Pin** | **D2** | Digitaleingang für Start/Stopp |
| **2. Pin** | **GND** | Masse |

### Verwendete Bibliotheken & ihre Funktionen

| **Bibliothek** | **Funktion** |
|---------------|-------------|
| `Wire.h` | Ermöglicht die Kommunikation über den **I2C-Bus** (wird für das LCD benötigt). |
| `LiquidCrystal_I2C.h` | Steuert das **LCD 1602 mit I2C-Adapter** und erlaubt das Anzeigen von Text. |

### Code
#### Testcode für Anzeige des Displays
```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
lcd.init();
lcd.backlight();
}

void loop() {
lcd.setCursor(0, 0);
lcd.print("Chiara");
lcd.setCursor(0, 1);
lcd.print("Bachmann");
}
```

#### Code für die Stoppuhr
```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

const int buttonPin = 2;
bool running = false;
unsigned long startTime = 0;
unsigned long elapsedTime = 0;
bool lastButtonState = HIGH;  // Speichert den letzten Zustand des Tasters
unsigned long buttonPressTime = 0;  // Speichert den Zeitpunkt, wann der Knopf gedrückt wurde

void setup() {
    pinMode(buttonPin, INPUT_PULLUP);
    lcd.init();
    lcd.backlight();
    lcd.setCursor(0, 0);
    lcd.print("Stoppuhr bereit!");
}

void loop() {
    bool buttonState = digitalRead(buttonPin);  // Knopfzustand lesen

    if (buttonState == LOW && lastButtonState == HIGH) {  // Knopf wird gedrückt
        buttonPressTime = millis();  // Speichert die Zeit des Tastendrucks

        if (!running) {
            startTime = millis() - elapsedTime;  // Falls Pause, setze die Zeit korrekt
            running = true;
        } else {
            elapsedTime = millis() - startTime;  // Speichere aktuelle Zeit
            running = false;
        }
    }

    if (buttonState == LOW && millis() - buttonPressTime >= 2000) {  // Knopf wird 2 Sekunden gedrückt gehalten
        running = false;  // Stoppuhr stoppen
        elapsedTime = 0;  // Zeit auf 0 zurücksetzen
        lcd.setCursor(0, 1);
        lcd.print("Zeit: 0:00.00  ");  // Anzeige zurücksetzen
    }

    lastButtonState = buttonState;  // Speichert den aktuellen Zustand für den nächsten Loop

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
    if (seconds < 10) lcd.print("0");
    lcd.print(seconds);
    lcd.print(".");
    if (milliseconds < 10) lcd.print("0");
    lcd.print(milliseconds);
}
```
#### Funktionalität
Der Code steuert die Stoppuhr basierend auf Knopf-Interaktionen:
- Kurzes Drücken: Startet oder stoppt die Zeit.
- Langes Drücken (2 Sekunden): Setzt die Zeit auf 0 zurück.
- Die Zeit wird im Format Minuten:Sekunden.Millisekunden auf dem LCD ausgegeben.

### Probleme und Herausforderungen
- Das LCD-Display zeigte anfangs nur Kästchen, da die Hardware defekt war. Nach dem Austausch funktionierte es.

### Benutzte Quellen
- "Arduino lernen" Arbeitsheft
- https://hartmut-waller.info/arduinoblog/stoppuhr-lcd/
- https://funduino.de/wp-content/uploads/2021/01/Stoppuhr.pdf
