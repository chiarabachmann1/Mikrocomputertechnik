# 4.1 Timing
### Wie ist die Zahl in der Funktion delay() zu verstehen?
Die Zahl in delay(x) gibt an, wie viele Millisekunden das Programm an dieser Stelle pausiert. Beispiel: delay(1000); bedeutet, dass das Programm für 1 Sekunde (1000 ms) angehalten wird.

### Welche weiteren Funktionen stehen in Zusammenhang mit dem Timing zur Verfügung?
- millis(): Gibt die Zeit in Millisekunden seit dem Start des Programms zurück.
- micros(): Gibt die Zeit in Mikrosekunden zurück.
- delayMicroseconds(x): Verzögert das Programm um x Mikrosekunden.

### Wie schnell ist der Arduino?
- Der Arduino UNO hat eine Taktfrequenz von 16 MHz.
-  Ein einfacher Befehl (z. B. digitalWrite()) dauert typischerweise einige Mikrosekunden.

### Wie lange braucht der Arduino für eine Schleife (loop())
```cpp
unsigned long startTime = millis();
for (int i = 0; i < 1000; i++) {
    loop();
}
unsigned long duration = millis() - startTime;
Serial.println(duration / 1000.0); // Durchschnittliche Laufzeit pro loop()
```

# 4.2 Dynamischer Blinker
Die LED bleibt zuerst lange aus, dann ändert sich das Verhältnis bis zum Gegenteil.
```cpp
const int ledPin = 5;
int onTime = 100;
int offTime = 1900;
int step = 50;

void setup() {
    pinMode(ledPin, OUTPUT);
}

void loop() {
    digitalWrite(ledPin, HIGH);
    delay(onTime);
    digitalWrite(ledPin, LOW);
    delay(offTime);

    onTime += step;
    offTime -= step;

    if (onTime >= 1900 || onTime <= 100) {
        step = -step; // Richtung umkehren
    }
}
```

# 4.3 Schaltung mit pull-up Widerstand
```cpp
const int ledPin = 5;
const int buttonPin = 2;

void setup() {
    pinMode(ledPin, OUTPUT);
    pinMode(buttonPin, INPUT_PULLUP); // Interner Pull-Up Widerstand wird aktiviert
}

void loop() {
    int state = digitalRead(buttonPin);
    digitalWrite(ledPin, state == LOW ? HIGH : LOW); // Invertierte Logik
}
```

# 4.4 Sind alle Pins Pull-Up fähig?
- Ja, alle digitalen Pins des Arduino UNO haben einen internen Pull-Up-Widerstand.
- Nein, es gibt keine internen Pull-Down Widerstände – man muss externe Widerstände verwenden.

# 4.5 Taster zum Ein-/Ausschalten der LED
```cpp
const int ledPin = 5;
const int buttonPin = 2;
bool ledState = false;
bool lastButtonState = HIGH;

void setup() {
    pinMode(ledPin, OUTPUT);
    pinMode(buttonPin, INPUT_PULLUP);
}

void loop() {
    bool buttonState = digitalRead(buttonPin);

    if (buttonState == LOW && lastButtonState == HIGH) {
        ledState = !ledState;
        digitalWrite(ledPin, ledState ? HIGH : LOW);
        delay(50); // Entprellung
    }

    lastButtonState = buttonState;
}
```

# 4.6 Fade-in, Fade-out
```cpp
const int ledPin = 5;
const int buttonPin = 2;
int brightness = 0;
bool fadeIn = true;

void setup() {
    pinMode(ledPin, OUTPUT);
    pinMode(buttonPin, INPUT_PULLUP);
}

void loop() {
    if (digitalRead(buttonPin) == LOW) {
        for (int i = 0; i <= 255; i++) {
            analogWrite(ledPin, fadeIn ? i : 255 - i);
            delay(5);
        }
        fadeIn = !fadeIn; // Umschalten für nächsten Druck
        while (digitalRead(buttonPin) == LOW); // Warten, bis der Knopf losgelassen wird
        delay(50); // Entprellung
    }
}
```

# 4.7 Interrupt mit Fade-in/Fade-out
```cpp
const int ledPin = 5;
const int buttonPin = 2;
bool fadeIn = true;
volatile bool buttonPressed = false;

void InterruptHandler() {
    buttonPressed = true;
}

void setup() {
    pinMode(ledPin, OUTPUT);
    pinMode(buttonPin, INPUT_PULLUP);
    attachInterrupt(digitalPinToInterrupt(buttonPin), InterruptHandler, FALLING);
}

void loop() {
    if (buttonPressed) {
        buttonPressed = false;
        for (int i = 0; i <= 255; i++) {
            analogWrite(ledPin, fadeIn ? i : 255 - i);
            delay(5);
        }
        fadeIn = !fadeIn;
    }
}
```
# 4.8 Fade-In/Fade-Out, per Interrupt initiiert & entprellt
```cpp
const int ledPin = 5;
const int buttonPin = 2;
volatile bool buttonPressed = false;
bool fadeIn = true;

void IRAM_ATTR InterruptHandler() {
    buttonPressed = true;
}

void fadeLED(bool up) {
    int step = up ? 5 : -5;
    for (int i = (up ? 0 : 255); (up ? i <= 255 : i >= 0); i += step) {
        analogWrite(ledPin, i);
        delay(10);
    }
}

void setup() {
    pinMode(ledPin, OUTPUT);
    pinMode(buttonPin, INPUT_PULLUP);
    attachInterrupt(digitalPinToInterrupt(buttonPin), InterruptHandler, FALLING);
}

void loop() {
    if (buttonPressed) {
        buttonPressed = false;
        fadeLED(fadeIn);
        fadeIn = !fadeIn;
    }
}
```

# 5.1 Unterschied zwischen Präprozessor #define und C++-Variable
#### Unterschiede:
| Aspekt            | `#define`                 | `const bool`            |
|-------------------|-------------------------|-------------------------|
| **Speicherverbrauch** | Kein Speicher belegt, reine Textersetzung | Belegt Speicher (1 Byte für `bool`) |
| **Datentypprüfung**  | Keine, reine Textersetzung | Compiler prüft Datentypen |
| **Debugging**       | Keine Fehlerprüfung durch Compiler | Compiler erkennt falsche Typen |
| **Sichtbarkeit**    | Gilt für gesamte Datei (global) | Kann auf Block-/Dateiebene begrenzt werden |
| **Änderung zur Laufzeit** | Nein | Ja, aber nur in Ausnahmefällen |

