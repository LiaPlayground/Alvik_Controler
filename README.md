<!--
author:   Ihr Name
email:    ihre.email@beispiel.de
version:  1.0.0
language: de
narrator: Deutsch Female

comment:  Regelungstechnik mit dem Arduino Alvik - Ein praktischer Einblick in Python, Sensortechnik und Regelkreise

import: https://github.com/liascript/CodeRunner


@style
.flex-container {
    display: flex;
    flex-wrap: wrap; /* Allows the items to wrap as needed */
    align-items: stretch;
    gap: 20px; /* Adds both horizontal and vertical spacing between items */
}

.flex-child { 
    flex: 1;
    margin-right: 20px; /* Adds space between the columns */
}

@media (max-width: 600px) {
    .flex-child {
        flex: 100%; /* Makes the child divs take up the full width on slim devices */
        margin-right: 0; /* Removes the right margin */
    }
}
@end


-->

[![LiaScript](https://raw.githubusercontent.com/LiaScript/LiaScript/master/badges/course.svg)](https://liascript.github.io/course/?https://raw.githubusercontent.com/liaScript/ArduinoEinstieg/master/Parcel_monitor.md#1)


# Mikrocontrollerprogrammierung und Datenanalyse 

Einstieg in die Regelungstechnik mit dem Arduino Alvik Roboter und Python
================================

<section class="flex-container">

<!-- class="flex-child" style="min-width: 250px;" -->
> **Tage der Naturwissenschaften, Johannes-Keppler-Gymnasium, 2025**
> 
> Prof. Dr. Sebastian Zug
>
> Technische Universität Bergakademie Freiberg
>
> <h2>Herzlich Willkommen!</h2>

<!-- class="flex-child" style="min-width: 250px;" -->
![Welcome](https://github.com/LiaPlayground/ArduinoEinstieg/blob/master/images/WorkingDesk.jpg?raw=true)

</section>

Die interaktive Ansicht dieses Kurses ist unter folgendem [Link](https://liascript.github.io/course/?https://raw.githubusercontent.com/LiaPlayground/Alvik_Controler/main/README.md) verfügbar.
Der Quellcode der Materialien ist unter https://github.com/LiaPlayground/Alvik_Controler zu finden.

## Kennenlernen

> __Wer bin ich und was macht man so an der TUBAF?__

![Husky](https://github.com/LiaPlayground/ArduinoEinstieg/blob/master/images/Husky_on_tour.png?raw=true "Autonomer Roboter der Arbeitsgruppe")
![Husky](https://github.com/LiaPlayground/ArduinoEinstieg/blob/master/images/Schwimmroboter.png?raw=true "Schwimmroboter mit Windmessungsaufsatz")

> __Wer sind Sie? Was erwarten Sie vom heutigen Tutorial?__

## Die Aufgabe

> __Ziele des Kurses:__ 

1. Sie verstehen das Grundlegende Prinzip der Regelungstechnik
2. Lernen die Vielfalt der Nutzungsmöglichkeiten von Python kennen.

Dazu werden wir uns gemeinsam in folgendes Problem einarbeiten:

> **Der Roboter folgt der Hand**: Ein Arduino Alvik Roboter soll immer in einem festen Abstand (z.B. 20 cm) zu Ihrer Hand bleiben, wenn Sie diese vor den Roboter halten.

> **Aufgabe:**  Welchen Anwendungsfall sehen Sie darin? Wo könnte so etwas nützlich sein?

### Der Roboter: Arduino Alvik

![](https://www.datocms-assets.com/32427/1715680599-alvik-callouts_1522x1080-white.jpg?auto=format&max-w=980 "Dokumentenation: https://docs.arduino.cc/hardware/alvik/")

> Die Programmierung kann in C++ (Arduino IDE) oder MicroPython erfolgen. Es gibt auch grafische programmierbare Umgebungen wie mBlock.

### Unsere Regelungsaufgabe etwas abstrakter

Unsere Aufgabe integriert alle Komponenten eines klassischen **Regelkreises**:

```ascii

    Sollwert  ┌─────────────┐      ┌──────────────┐      ┌─────────────┐
        ─────>│   Fehler    │─────>│    Regler    │─────>│  Stellglied │
              │             │      │   Software   │      │   Motoren   │
              └─────────────┘      └──────────────┘      └──────┬──────┘
                    ^                                           │
                    │                                           v
                    │                                    ┌─────────────┐
                    │                 "$Regelkreis$"     │   Strecke   │
                    │                                    │ Roboter&Hand│
                    │                                    └──────┬──────┘
                    │              ┌──────────────┐             │
                    └──────────────│  Messglied   │<────────────┘
                                   │Distanz-Sensor│
                                   └──────────────┘                                                 .
```

> **Aufgabe:** Nehmen wir an, wir wollen einen Regelkreis aufbauen, so dass ein autonomes Automobile immer in der Mitte der Straße bleibt. Wie würden wir das umsetzen? 

> Was brauchen wir für unser "Der Roboter folgt der Hand"-Projekt?
>
> | Komponente      | Beschreibung                          | Aufgabe |
> |-----------------|---------------------------------------|---------|
> | 1. Messglied      | ToF-Sensor misst aktuelle Distanz zur Hand | Messung ins Programm einbauen |
> | 2. Sollwert / Fehler       | Gewünschter Abstand zur Hand (z.B. 20 cm) | Definieren und Berechnung umsetzen |
> | 3. Stellglied     | Motoren, die den Roboter bewegen       | im Programm ansteuern  |
> | 4. Regler        | Software, die den Fehler verarbeitet   | Entwerfen und implementieren |
>
> Danach testen wir unsere Regelung und optimieren die Parameter mit der Strecke - also unserem Roboter auf dem Tisch. 

## 1. Messglied - Distanzmessung mit ToF-Sensoren

![](https://docs.arduino.cc/static/ce29ac06a54005218e6763f0594d29da/4ef49/image.png "ToF-Sensoren am Alvik-Roboter Dokumentation des Projektes unter https://docs.arduino.cc/tutorials/alvik/user-manual/#deep-dive-programming-alvik")

**ToF-Sensoren** messen die Entfernung durch Laufzeitmessung von Lichtsignalen:

1. Sensor sendet einen kurzen Lichtimpuls aus
2. Licht wird vom Objekt reflektiert
3. Sensor misst die Zeit bis zum Empfang
4. Distanz = (Lichtgeschwindigkeit × Zeit) / 2

**Vorteile:**
- ✅ Präzise Messung (±3%)
- ✅ Funktioniert bei verschiedenen Materialien
- ✅ Messbereich: 5-200 cm
- ✅ Schnelle Messung (bis 50 Hz)

```python
from arduino_alvik import ArduinoAlvik
import time

# Alvik initialisieren
alvik = ArduinoAlvik()
alvik.begin()

print("Bereit. Tippe 'OK' (check) auf das Touch-Panel, um die Messung zu starten...")

# --- Programmstart per OK-Touch ---
while not alvik.get_touch_ok():
    time.sleep(0.05)

# --- Hauptloop ---
running = True
while running:

    # --- Abbruch per CANCEL-Touch ---
    if alvik.get_touch_cancel():
        print("Messung manuell gestoppt (CANCEL).")
        running = False
        break

    # Distanzwerte holen
    distance = alvik.get_distance(unit='cm')
    print(distance[0], distance[1], distance[2], distance[3], distance[4])

    time.sleep(0.1)

# --- Aufräumen ---
alvik.stop()
print("Roboter gestoppt.")
```

> **Aufgabe:** Erklären Sie den Code, was passiert in der Hauptschleife?

## 2. Sollwert und Fehlerberechnung

> **Aufgabe:** Was müssen wir abändern, damit wir überprüfen können, ob der mittlere Sensor (Index 2) einen Abstand von größer oder kleiner 10 cm misst?

> Wir entwerfen gemeinsam einen Code!

## 3. Stellglied - Ansteuerung der Motoren

Der Alvik-Roboter hat zwei Motoren, deren Geschwindigkeit wir separat steuern können. 

> **Aufgabe:** Welche Bewegungen erwarten Sie bei folgenden Funktionsaufrufen?

| Funktion                           | Beschreibung                                   |
| ---------------------------------- | ---------------------------------------------- |
| `alvik.set_wheels_speed(10, 10)`   | Vorwärtsbewegung mit mittlerer Geschwindigkeit |
| `alvik.set_wheels_speed(0, 0)`     |                                                |
| `alvik.set_wheels_speed(-10, -10)` |                                                |
| `alvik.set_wheels_speed(10, -10)`  |                                                |

Wie sieht ein Programm für die Ansteuerung der Motoren dann konkret aus?

```python Motoransteuerung.py
from arduino_alvik import ArduinoAlvik
import time

# Alvik initialisieren
alvik = ArduinoAlvik()
alvik.begin()

print("Bereit. Tippe 'OK' (check) auf das Touch-Panel, um den Motortest zu starten...")

# --- Programmstart per OK-Touch ---
while not alvik.get_touch_ok():
    time.sleep(0.05)

# --- Motoren ansteuern ---
alvik.set_wheels_speed(10, 10)

# --- Hauptloop ---
running = True
while running:

    # --- Abbruch per CANCEL-Touch ---
    if alvik.get_touch_cancel():
        print("Messung manuell gestoppt (CANCEL).")
        running = False
        break

    time.sleep(0.1)

# --- Aufräumen ---
alvik.stop()
print("Roboter gestoppt.")
```

> **Aufgabe:** Die Geschwindigkeit soll langsam erhöht werden. Wie würden Sie das umsetzen? 

## 4. P-Regler

Wir wollen, dass der Roboter **immer einen konstanten Abstand** zu Ihrer Hand hält:

- Hand kommt näher → Roboter fährt rückwärts
- Hand entfernt sich → Roboter fährt vorwärts
- Hand bleibt auf Zielabstand → Roboter steht still

Die einfachste Form der Regelung ist der **P-Regler**:

$$
  u(t) = K_p \cdot e(t) = K_p \cdot (Sollwert - Istwert)
$$

Dabei ist:

- $u(t)$ = Stellgröße (Motorgeschwindigkeit)
- $K_p$ = Proportionalfaktor (Verstärkung)
- $e(t)$ = Regelfehler = Sollwert - Istwert

### Implementation

```python  Regler.py
from arduino_alvik import ArduinoAlvik
import time

# Alvik initialisieren
alvik = ArduinoAlvik()
alvik.begin()

print("Bereit. Tippe 'OK' (check) auf das Touch-Panel, um die Messung zu starten...")

# --- Programmstart per OK-Touch ---
while not alvik.get_touch_ok():
    time.sleep(0.05)

# --- Hauptloop ---
start = time.ticks_ms()

running = True
while running:

    # --- Abbruch per CANCEL-Touch ---
    if alvik.get_touch_cancel():
        print("Messung manuell gestoppt (CANCEL).")
        running = False
        break

    # Distanzwerte holen
    distance = alvik.get_distance(unit='cm')
    center_distance = distance[2]

    # Regelfehler berechnen (10 cm Zielabstand)
    error = (center_distance - 10)

    # Stellgröße berechnen (P-Regler)
    speed = 100 * error

    # Zeitmessung für Datenaufzeichnung
    elapsed = time.ticks_diff(time.ticks_ms(), start)
    print(elapsed, center_distance, error, speed)
  
    # Motoren ansteuern
    alvik.set_wheels_speed(speed, speed)

    time.sleep(0.05)

# --- Aufräumen ---
alvik.stop()
print("Roboter gestoppt.")

```

> **Beobachtung:** Wie verhält sich der Roboter? Sind Sie zufrieden mit der Regelung?

### Was bewirkt Kp?

Der Parameter $K_p$ bestimmt, wie "aggressiv" der Regler reagiert:

| $K_p$            | Verhalten                     | Vor-/Nachteile               |
| ---------------- | ----------------------------- | ---------------------------- |
| Klein (z.B. 10)  | Langsame, sanfte Annäherung   | ✅ Stabil, ❌ Langsam        |
| Mittel (z.B. 30) | Moderate Reaktion             | ✅ Guter Kompromiss          |
| Groß (z.B. 100)  | Schnelle, aggressive Reaktion | ✅ Schnell, ❌ Überschwingen |


**Ursachen:**

1. **Überschießen**: Roboter reagiert zu stark → fährt über Ziel hinaus
2. **Gegenreaktion**: Regler erkennt negativen Fehler → fährt in Gegenrichtung
3. **Oszillation**: System schwingt um den Sollwert

Ein gut eingestellter Regler sollte:

- ✅ **Keine Dauerschwingung** zeigen
- ✅ **Schnell** den Sollwert erreichen
- ✅ **Minimal überschwingen** (< 10%)
- ✅ **Stabil** bei Störungen bleiben

> **Wie können wir diesen Wert sinnvoll festlegen? Wir brauchen eine systematische Methode**

## Datenvisualisierung mit Python

Unsere Daten sind zeilenweise gespeichert worden. Beim bloßen draufschauen, ist es schwer, das Verhalten des Reglers zu beurteilen.

```text data.csv 
time distance error speed
0 13.4 3.400001 340.0001
54 13.5 3.5 350.0
110 13.3 3.3 330.0
165 13.2 3.2 320.0
219 13.2 3.2 320.0
274 13.3 3.3 330.0
331 13.0 3.0 300.0
```

Diagramme helfen uns:

- 📈 Das Systemverhalten zu verstehen
- 🔍 Probleme zu erkennen
- ⚙️ Parameter optimal einzustellen

> Sie können natürlich die Daten auch in Excel oder Google Sheets importieren und dort Diagramme erstellen. Aber ... Python ist hier viel mächtiger und flexibler!

> Visualisierung ist ein wichtiger Anwendungsfall von Python. Bibliotheken dafür sind **Pandas** (Datenstrukturierung) und **Matplotlib** (Plots). 

### Laden der Daten

Pandas macht es uns sehr einfach csv-Dateien zu laden: Dazu brauchen wir nur wenige Zeilen Code.

> Im Unterschied zum bisherigen Programmcode, der auf dem Arduino Alvik lief, wird dieser Code lokal ausgeführt. Damit das hier funktioniert haben wir eine browserbasierte Variante für Sie vorbereitet. Klicken Sie auf den Button unter dem Codefenster.

```text -data.csv 
time distance error speed
0 13.4 3.400001 340.0001
54 13.5 3.5 350.0
110 13.3 3.3 330.0
165 13.2 3.2 320.0
219 13.2 3.2 320.0
274 13.3 3.3 330.0
331 13.0 3.0 300.0
386 12.8 2.8 280.0
442 12.2 2.2 220.0
498 11.6 1.6 160.0
554 11.2 1.2 120.0
609 10.6 0.6000004 60.00004
664 10.6 0.6000004 60.00004
719 9.5 -0.5 -50.0
775 8.8 -1.2 -120.0
828 8.5 -1.5 -150.0
884 7.7 -2.3 -230.0
940 7.8 -2.2 -220.0
```
```python readCSV.py
import pandas as pd

df = pd.read_csv('data.csv', header = 0, sep=" ")  
print(df)
```
@LIA.eval(`["data.csv", "main.py"]`, `none`, `python3 main.py`)


### Visualisierung des schwingenden Reglers

> Schauen wir uns zunächst einen Schwingenden Regler (Kp = 100) an. Hinweis: Das Hindernis stand bei dieser Messung still!

```python readCSV.py
import pandas as pd
import matplotlib.pyplot as plt
import pandas as pd

url = "https://raw.githubusercontent.com/LiaPlayground/Alvik_Controler/refs/heads/main/data/critical.csv"

df = pd.read_csv(url, header = 0, sep=" ")  
plt.plot(df["time"], df["distance"])
plt.axhline(y=10, color='r', linestyle='--', label='Sollwert (10 cm)')
plt.xlabel("Time (s)")
plt.ylabel("Distance (cm)")

plt.savefig('foo.png')
```
@LIA.eval(`["main.py"]`, `none`, `python3 main.py`)

>  Bestimmen Sie die maximale Überschwingung im Diagramm.

### Optimierter Regler

```python readCSV.py
import pandas as pd
import matplotlib.pyplot as plt
import pandas as pd

url = "https://raw.githubusercontent.com/LiaPlayground/Alvik_Controler/refs/heads/main/data/optimal.csv"

df = pd.read_csv(url, header = 0, sep=" ")  
plt.plot(df["time"], df["distance"])
plot.axhline(y=10, color='r', linestyle='--', label='Sollwert (10 cm)')
plt.xlabel("Time (s)")
plt.ylabel("Distance (cm)")

plt.savefig('foo.png')
```
@LIA.eval(`["main.py"]`, `none`, `python3 main.py`)

> Wie können wir die Geschwindigkeiten im Diagramm darstellen, um das Regelverhalten besser zu verstehen?

## Zusammenfassung und Ausblick

Dieser Kurs zeigt die **Bandbreite von Python**:

- 🔧 **Hardware-Programmierung** (MicroPython auf ESP32)
- 📊 **Datenanalyse** (Pandas, NumPy)
- 📈 **Visualisierung** (Matplotlib)
- 🤖 **Robotik** (Echtzeit-Regelung)

Wir kennen die grundlegenden Konzepte der **Regelungstechnik**:

- Messglied
- Sollwert und Regelfehler
- Stellglied
- Regler (P-Regler)

> Vielen Dank für Ihr Interesse und viel Erfolg bei Ihren eigenen Projekten!

## Ressourcen und weiterführende Links

- [Offizielle Arduino Alvik Docs](https://docs.arduino.cc/hardware/alvik/)
- [MicroPython Bibliothek](https://github.com/arduino/arduino-alvik-mpy)

- [PID-Regler erklärt](https://de.wikipedia.org/wiki/Regler#PID-Regler)
- [Control Systems mit Python](https://python-control.readthedocs.io/)

- [Pandas Tutorial](https://pandas.pydata.org/docs/getting_started/intro_tutorials/)
- [Matplotlib Gallery](https://matplotlib.org/stable/gallery/index.html)


## Quiz: Testen Sie Ihr Wissen!

**Frage 1:** Was misst ein ToF-Sensor?

[( )] Temperatur
[(X)] Distanz durch Laufzeitmessung
[( )] Helligkeit
[( )] Geschwindigkeit

**Frage 2:** Was passiert, wenn Kp zu groß gewählt wird?

[( )] Der Roboter wird langsamer
[(X)] Das System beginnt zu schwingen
[( )] Der Sensor funktioniert nicht mehr
[( )] Nichts

**Frage 3:** Was ist der Regelfehler?

[( )] Die Motorgeschwindigkeit
[(X)] Die Differenz zwischen Sollwert und Istwert
[( )] Die Sensormessgenauigkeit
[( )] Die Rechenzeit

**Frage 4:** Welche Python-Bibliothek nutzen wir zur Visualisierung?

[( )] NumPy
[(X)] Matplotlib
[( )] TensorFlow
[( )] Flask

**Frage 5:** Was ist ein gutes Zeichen für einen stabilen Regler?

[(X)] Der Fehler konvergiert gegen Null ohne Dauerschwingung
[( )] Der Roboter fährt sehr schnell
[( )] Die Motoren sind immer an
[( )] Der Sensor zeigt immer denselben Wert
