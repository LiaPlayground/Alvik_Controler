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

Die interaktive Ansicht dieses Kurses ist unter folgendem [Link](https://liascript.github.io/course/?https://raw.githubusercontent.com/liaScript/ArduinoEinstieg/master/Parcel_monitor.md#1) verfügbar.
Der Quellcode der Materialien ist unter https://github.com/liaScript/ArduinoEinstieg/blob/master/Parcel_monitor.md zu finden.

## Kennenlernen

> __Wer bin ich und was macht man so an der TUBAF?__

![Husky](https://github.com/LiaPlayground/ArduinoEinstieg/blob/master/images/Husky_on_tour.png?raw=true "Autonomer Roboter der Arbeitsgruppe")
![Husky](https://github.com/LiaPlayground/ArduinoEinstieg/blob/master/images/Schwimmroboter.png?raw=true "Schwimmroboter mit Windmessungsaufsatz")

> __Wer sind Sie? Was erwarten Sie vom heutigen Tutorial?__

## Die Aufgabe

> __Ziele des Kurses:__ 

1. Sie verstehen das Grundlegende Prinzip der Regelungstechnik
2. Lernen die vielfalt der Nutzungsmöglichkeiten von Python kennen.

Dazu werden wir uns gemeinsam in folgendes Problem einarbeiten:

> **Der Roboter folgt der Hand**: Ein Arduino Alvik Roboter soll immer in einem festen Abstand (z.B. 20 cm) zu Ihrer Hand bleiben, wenn Sie diese vor den Roboter halten.

> **Aufgabe:**  Welchen Anwendungsfall sehen Sie darin? Wo könnte so etwas nützlich sein?

### Der Roboter: Arduino Alvik

![](https://www.datocms-assets.com/32427/1715680599-alvik-callouts_1522x1080-white.jpg?auto=format&max-w=980 "Dokumentenation: https://docs.arduino.cc/hardware/alvik/")

> Die Programmierung kann in C++ (Arduino IDE) oder MicroPython erfolgen. Es gibt auch grafische progrommierbare Umgebungen wie mBlock.

### Unsere Regelungsaufgabe etwas abstrakter

Unsere Aufgabe integrierte alle Komponenten eines klassischen **Regelkreises**:

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

> **Aufgabe:** Die Geschwindigkeit soll langsam erhöt werden. Wie würden Sie das umsetzen? 

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

> Im unterschied zum bisherigen Programmcode, der auf dem Arduino Alvik lief, wird dieser Code auf lokal ausgeführt. Damit das hier funktioniert haben wir eine Browserbasierte Variante für Sie vorbereitet. Klicken Sie auf den Button unter dem Codefenster.



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
993 8.8 -1.2 -120.0
1047 8.8 -1.2 -120.0
1102 9.3 -0.6999998 -69.99998
1158 9.7 -0.3000002 -30.00002
1216 9.7 -0.3000002 -30.00002
1271 10.8 0.8000002 80.00002
1326 11.4 1.400001 140.0001
1382 11.7 1.7 170.0
1437 11.7 1.7 170.0
1492 12.2 2.2 220.0
1549 12.2 2.2 220.0
1604 12.4 2.400001 240.0001
1660 12.4 2.400001 240.0001
1716 12.2 2.2 220.0
1771 11.8 1.8 180.0
1827 11.8 1.8 180.0
1882 11.8 1.8 180.0
1937 10.0 0.0 0.0
1992 9.0 -1.0 -100.0
2048 9.0 -1.0 -100.0
2102 8.7 -1.3 -130.0
2155 8.5 -1.5 -150.0
2212 8.6 -1.4 -140.0
2267 8.6 -1.4 -140.0
2322 8.900001 -1.099999 -109.9999
2378 9.6 -0.3999996 -39.99996
2433 9.6 -0.3999996 -39.99996
2487 10.1 0.1000004 10.00004
2552 10.1 0.1000004 10.00004
2608 10.1 0.1000004 10.00004
2666 11.2 1.2 120.0
2721 11.3 1.3 130.0
2774 11.3 1.3 130.0
2830 11.2 1.2 120.0
2886 11.2 1.2 120.0
```
```python readCSV.py
import pandas as pd

df = pd.read_csv('data.csv', header = 0, sep=" ")  
print(df)
```
@LIA.eval(`["data.csv", "main.py"]`, `none`, `python3 main.py`)

### Visualisierungsbseispiel

```python readCSV.py
import pandas as pd
import matplotlib.pyplot as plt

# Beispiel-Daten erzeugen
df = pd.DataFrame({
    "x": [0, 1, 2, 3, 4, 5],
    "y": [0, 1, 4, 9, 16, 25]
})

# Plot erstellen
plt.plot(df["x"], df["y"])
plt.xlabel("x")
plt.ylabel("y")
plt.title("Einfaches XY-Diagramm aus Pandas-Daten")

plt.savefig('foo.png')
```
@LIA.eval(`[""main.py"]`, `none`, `python3 main.py`)


### Visualisierung unserer Reglerdaten

> Schauen wir uns zunächst einen Schwingenden Regler (Kp = 100) an. Hinweis: Das Hindernis stand bei dieser Messung still!

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
993 8.8 -1.2 -120.0
1047 8.8 -1.2 -120.0
1102 9.3 -0.6999998 -69.99998
1158 9.7 -0.3000002 -30.00002
1216 9.7 -0.3000002 -30.00002
1271 10.8 0.8000002 80.00002
1326 11.4 1.400001 140.0001
1382 11.7 1.7 170.0
1437 11.7 1.7 170.0
1492 12.2 2.2 220.0
1549 12.2 2.2 220.0
1604 12.4 2.400001 240.0001
1660 12.4 2.400001 240.0001
1716 12.2 2.2 220.0
1771 11.8 1.8 180.0
1827 11.8 1.8 180.0
1882 11.8 1.8 180.0
1937 10.0 0.0 0.0
1992 9.0 -1.0 -100.0
2048 9.0 -1.0 -100.0
2102 8.7 -1.3 -130.0
2155 8.5 -1.5 -150.0
2212 8.6 -1.4 -140.0
2267 8.6 -1.4 -140.0
2322 8.900001 -1.099999 -109.9999
2378 9.6 -0.3999996 -39.99996
2433 9.6 -0.3999996 -39.99996
2487 10.1 0.1000004 10.00004
2552 10.1 0.1000004 10.00004
2608 10.1 0.1000004 10.00004
2666 11.2 1.2 120.0
2721 11.3 1.3 130.0
2774 11.3 1.3 130.0
2830 11.2 1.2 120.0
2886 11.2 1.2 120.0
2941 11.2 1.2 120.0
2994 10.9 0.9000006 90.00006
3050 10.4 0.4000006 40.00006
3106 9.7 -0.3000002 -30.00002
3163 9.5 -0.5 -50.0
3217 8.400001 -1.599999 -159.9999
3271 8.400001 -1.599999 -159.9999
3329 8.3 -1.7 -170.0
3385 8.400001 -1.599999 -159.9999
3440 8.2 -1.8 -180.0
3497 8.2 -1.8 -180.0
3552 8.2 -1.8 -180.0
3606 8.2 -1.8 -180.0
3661 8.5 -1.5 -150.0
3716 9.5 -0.5 -50.0
3770 10.1 0.1000004 10.00004
3825 10.4 0.4000006 40.00006
3882 10.4 0.4000006 40.00006
3945 11.3 1.3 130.0
4000 11.2 1.2 120.0
4056 11.0 1.0 100.0
4111 11.0 1.0 100.0
4167 10.7 0.6999998 69.99998
4223 10.0 0.0 0.0
4280 9.0 -1.0 -100.0
4337 9.0 -1.0 -100.0
4393 8.5 -1.5 -150.0
4448 8.6 -1.4 -140.0
4504 8.6 -1.4 -140.0
4558 8.900001 -1.099999 -109.9999
4613 8.900001 -1.099999 -109.9999
4668 8.900001 -1.099999 -109.9999
4723 9.6 -0.3999996 -39.99996
4777 10.3 0.3000002 30.00002
4832 10.6 0.6000004 60.00004
4886 11.2 1.2 120.0
4943 11.2 1.2 120.0
4998 11.2 1.2 120.0
5053 11.8 1.8 180.0
5107 11.9 1.900001 190.0001
5162 12.0 2.0 200.0
5217 12.0 2.0 200.0
5271 12.0 2.0 200.0
5326 11.5 1.5 150.0
5382 11.1 1.1 110.0
5436 10.7 0.6999998 69.99998
5492 9.6 -0.3999996 -39.99996
5547 9.6 -0.3999996 -39.99996
5602 8.8 -1.2 -120.0
5658 8.8 -1.2 -120.0
5713 8.7 -1.3 -130.0
5769 8.7 -1.3 -130.0
5824 9.2 -0.8000002 -80.00002
5879 9.2 -0.8000002 -80.00002
5935 10.3 0.3000002 30.00002
5989 11.1 1.1 110.0
6044 11.3 1.3 130.0
6100 11.7 1.7 170.0
6154 11.7 1.7 170.0
6210 11.5 1.5 150.0
6265 11.1 1.1 110.0
6320 11.1 1.1 110.0
6376 10.1 0.1000004 10.00004
6431 9.2 -0.8000002 -80.00002
6490 9.0 -1.0 -100.0
6543 8.5 -1.5 -150.0
6598 8.6 -1.4 -140.0
6654 8.5 -1.5 -150.0
6710 8.3 -1.7 -170.0
6767 8.6 -1.4 -140.0
6822 8.8 -1.2 -120.0
6876 8.8 -1.2 -120.0
6930 9.7 -0.3000002 -30.00002
6984 9.7 -0.3000002 -30.00002
7041 10.5 0.5 50.0
7097 11.1 1.1 110.0
7151 11.1 1.1 110.0
7205 11.6 1.6 160.0
7260 12.0 2.0 200.0
7315 12.1 2.1 210.0
7371 12.1 2.1 210.0
7427 11.9 1.900001 190.0001
7483 12.0 2.0 200.0
7538 11.6 1.6 160.0
7593 10.8 0.8000002 80.00002
7647 10.8 0.8000002 80.00002
7700 10.2 0.1999998 19.99998
7754 9.7 -0.3000002 -30.00002
7810 9.1 -0.8999996 -89.99996
7865 8.400001 -1.599999 -159.9999
7921 8.400001 -1.599999 -159.9999
7977 8.2 -1.8 -180.0
8032 8.3 -1.7 -170.0
8086 8.3 -1.7 -170.0
8150 8.3 -1.7 -170.0
8204 8.400001 -1.599999 -159.9999
8259 9.1 -0.8999996 -89.99996
8313 9.7 -0.3000002 -30.00002
8367 9.7 -0.3000002 -30.00002
```
```python readCSV.py
import pandas as pd
import matplotlib.pyplot as plt
import pandas as pd

df = pd.read_csv('data.csv', header = 0, sep=" ")  
plt.plot(df["time"], df["distance"])
plt.xlabel("Time (s)")
plt.ylabel("Distance (cm)")

plt.savefig('foo.png')
```
@LIA.eval(`["data.csv", "main.py"]`, `none`, `python3 main.py`)

>  Bestimmen Sie die maximale Überschwingung im Diagramm.

### Optimierter Regler

```text -data.csv 
time distance error speed
0 12.2 2.2 44.0
55 12.2 2.2 44.0
110 12.2 2.2 44.0
166 12.2 2.2 44.0
220 12.2 2.2 44.0
274 11.9 1.900001 38.00001
330 11.5 1.5 30.0
385 11.5 1.5 30.0
439 11.0 1.0 20.0
496 11.0 1.0 20.0
550 11.1 1.1 22.00001
605 11.2 1.2 24.0
660 11.2 1.2 24.0
716 11.1 1.1 22.00001
771 11.4 1.400001 28.00001
824 11.4 1.400001 28.00001
880 11.4 1.400001 28.00001
933 11.2 1.2 24.0
987 11.1 1.1 22.00001
1042 11.1 1.1 22.00001
1097 11.1 1.1 22.00001
1153 11.2 1.2 24.0
1209 11.2 1.2 24.0
1264 11.2 1.2 24.0
1319 11.3 1.3 26.0
1375 11.1 1.1 22.00001
1431 11.1 1.1 22.00001
1487 10.9 0.9000006 18.00001
1543 10.9 0.9000006 18.00001
1598 10.9 0.9000006 18.00001
1655 11.6 1.6 32.00001
1709 11.6 1.6 32.00001
1764 12.0 2.0 40.0
1820 12.1 2.1 42.00001
1875 12.3 2.3 46.0
1931 12.3 2.3 46.0
1986 12.4 2.400001 48.00001
2041 12.6 2.6 52.00001
2095 12.7 2.7 54.0
2148 12.7 2.7 54.0
2212 13.3 3.3 66.0
2268 13.7 3.7 74.0
2324 14.3 4.3 86.0
2379 14.7 4.7 94.0
2434 14.8 4.8 96.0
2489 15.0 5.0 100.0
2543 15.3 5.3 106.0
2598 15.3 5.3 106.0
2652 15.9 5.900001 118.0
2708 15.9 5.900001 118.0
2763 16.8 6.800001 136.0
2819 17.4 7.4 148.0
2875 17.6 7.6 152.0
2930 17.5 7.5 150.0
2984 17.4 7.4 148.0
3040 17.0 7.0 140.0
3095 16.3 6.300001 126.0
3150 16.0 6.0 120.0
3204 15.4 5.400001 108.0
3259 14.7 4.7 94.0
3316 14.7 4.7 94.0
3369 14.0 4.0 80.0
3424 13.9 3.900001 78.00002
3479 13.6 3.6 72.00001
3532 13.2 3.2 64.0
3596 12.8 2.8 56.0
3650 12.8 2.8 56.0
3705 12.8 2.8 56.0
3761 11.2 1.2 24.0
3818 10.3 0.3000002 6.000004
3873 10.3 0.3000002 6.000004
3927 9.8 -0.1999998 -3.999996
3981 9.6 -0.3999996 -7.999992
4035 9.6 -0.3999996 -7.999992
4092 9.6 -0.3999996 -7.999992
4146 9.5 -0.5 -10.0
4201 9.400001 -0.5999994 -11.99999
4259 9.400001 -0.5999994 -11.99999
4314 9.5 -0.5 -10.0
4370 9.6 -0.3999996 -7.999992
4425 9.6 -0.3999996 -7.999992
4480 9.6 -0.3999996 -7.999992
4537 9.7 -0.3000002 -6.000004
4592 9.8 -0.1999998 -3.999996
4646 9.8 -0.1999998 -3.999996
4701 9.900001 -0.09999943 -1.999989
4756 9.900001 -0.09999943 -1.999989
4813 9.8 -0.1999998 -3.999996
4867 9.8 -0.1999998 -3.999996
4924 9.900001 -0.09999943 -1.999989
4978 9.900001 -0.09999943 -1.999989
5034 9.900001 -0.09999943 -1.999989
5089 9.900001 -0.09999943 -1.999989
5144 9.900001 -0.09999943 -1.999989
5197 10.0 0.0 0.0
5252 10.0 0.0 0.0
5307 10.0 0.0 0.0
5361 10.0 0.0 0.0
5416 10.0 0.0 0.0
5471 10.0 0.0 0.0
5526 10.0 0.0 0.0
5581 9.900001 -0.09999943 -1.999989
5637 10.0 0.0 0.0
5692 10.1 0.1000004 2.000008
5746 10.0 0.0 0.0
5800 10.0 0.0 0.0
5854 10.0 0.0 0.0
5909 9.900001 -0.09999943 -1.999989
5965 10.1 0.1000004 2.000008
6020 10.0 0.0 0.0
6075 10.0 0.0 0.0
6130 10.0 0.0 0.0
6184 10.0 0.0 0.0
6239 10.1 0.1000004 2.000008
6293 10.0 0.0 0.0
6348 10.0 0.0 0.0
6404 10.0 0.0 0.0
6459 10.0 0.0 0.0
6515 10.1 0.1000004 2.000008
6568 10.0 0.0 0.0
6623 9.900001 -0.09999943 -1.999989
6678 9.900001 -0.09999943 -1.999989
6731 10.1 0.1000004 2.000008
6786 9.900001 -0.09999943 -1.999989
6842 10.0 0.0 0.0
6898 10.1 0.1000004 2.000008
6953 10.1 0.1000004 2.000008
7009 10.0 0.0 0.0
7065 10.0 0.0 0.0
7120 10.0 0.0 0.0
7174 10.1 0.1000004 2.000008
7227 10.1 0.1000004 2.000008
7283 9.8 -0.1999998 -3.999996
7339 10.0 0.0 0.0
7393 10.0 0.0 0.0
7447 10.0 0.0 0.0
7504 10.1 0.1000004 2.000008
7558 10.1 0.1000004 2.000008
7614 10.1 0.1000004 2.000008
7668 10.1 0.1000004 2.000008
7723 10.0 0.0 0.0
7788 10.0 0.0 0.0
7843 10.0 0.0 0.0
7899 10.0 0.0 0.0
7954 10.2 0.1999998 3.999996
8011 9.900001 -0.09999943 -1.999989
8067 9.900001 -0.09999943 -1.999989
8120 9.8 -0.1999998 -3.999996
8176 9.8 -0.1999998 -3.999996
```
```python readCSV.py
import pandas as pd
import matplotlib.pyplot as plt
import pandas as pd

df = pd.read_csv('data.csv', header = 0, sep=" ")  
plt.plot(df["time"], df["distance"])
plt.xlabel("Time (s)")
plt.ylabel("Distance (cm)")

plt.savefig('foo.png')
```
@LIA.eval(`["data.csv", "main.py"]`, `none`, `python3 main.py`)


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
