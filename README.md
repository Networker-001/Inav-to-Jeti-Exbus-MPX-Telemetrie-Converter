# Universal iNav LTM-zu-Telemetrie Konverter (RP2040)

Diese Firmware ermöglicht die Übersetzung des push-basierten iNav LTM-Protokolls (Lightweight Telemetry) in die Sender-Rückkanäle von JETI (EXBUS/Standard), Multiplex MSB und Graupner HoTT über ein eigenständiges RP2040-Board.

---

## DER STRATEGISCHE HINTERGRUND

Die direkte Nutzung des Jeti EXBUS-Protokolls in der iNav-Firmware führt zu Instabilitäten. Durch Fehler im iNav-Quellcode und Inkompatibilitäten bei der Kompilierung mit neueren Compiler-Versionen kommt es bei aktiver EXBUS-Verbindung regelmäßig zu Systemausfällen (Freezes) des Flight Controllers. Diese treten vor allem bei schnellen Steuerbewegungen oder bei der Übertragung von mehr als 16 Kanälen auf. Zudem ist die Rückrichtung für Telemetriedaten fehlerhaft implementiert, was zu einfrierenden oder fehlenden Werten auf dem Sender führt. Multiplex-Protokolle werden von iNav nicht nativ unterstützt.

### DIE SYSTEMLÖSUNG DURCH DIESEN KONVERTER

Dieser Konverter löst das Problem, indem er die kritische Flugsteuerung und die Telemetrieverarbeitung physisch vollständig voneinander trennt. Dadurch haben Telemetrie-Verarbeitungsfehler im Flight Controller keine Auswirkungen auf die Steuerbarkeit des Modells.

Das Sicherheitskonzept basiert auf drei Säulen:

1. **Flugsteuerung über SBUS:** 
   Die Übertragung der Steuerbefehle vom Empfänger zum Flight Controller erfolgt über das standardisierte SBUS-Protokoll. SBUS arbeitet als reiner, robuster Einweg-Kanal ohne Rückrichtung. Da iNav bei diesem Protokoll keine Telemetriedaten verarbeiten muss, läuft die Flugsteuerung fehlerfrei. Ein Absturz der Steuerung durch Telemetrie-Überlastung ist ausgeschlossen.
2. **Autarke Telemetrie-Übersetzung (LTM):** 
   Der Flight Controller gibt seine Sensordaten über das ressourcenschonende LTM-Protokoll (Lightweight Telemetry) aus. Der eigenständige RP2040-Konverter fängt diesen Datenstrom an GPIO 5 ab und übernimmt die Übersetzung in das Zielformat (JETI EXBUS oder Multiplex) auf einem separaten Prozessor.
3. **Unabhängiger Zusatzkanal:** 
   Der Konverter speist die aufbereiteten Werte direkt in den Telemetrie-Eingang des Empfängers ein. Die Berechnung der Telemetrie erfolgt somit außerhalb des Flight Controllers. Selbst bei einer Störung auf der Telemetrieleitung bleibt der SBUS-Steuerkanal davon unberührt, und das Modell lässt sich sicher steuern.

---

## 1. PHYSISCHER HARDWARE-AUFBAU & SYMBOLZEICHNUNG

Das System ist in drei logische Funktionsebenen unterteilt. Der RC-Empfänger bildet die obere Ebene, der RP2040-Konverter vermittelt in der mittleren Ebene, und der iNav Flight Controller schließt das System als breite Basis nach unten ab. Der Aufbau ist hardwareseitig fest auf GPIO 5 als LTM-Eingang fixiert.

### Physikalische Signal- und Verdrahtungs-Matrix (Top-Down)

    +---------------------------------------------------------------+

    |                          RC-EMPFÄNGER                         |
    |                     (z.B. Jeti / Multiplex)                   |
    |                                                               |
    |  [SBUS Out] [+5V]  [GND]                    [GND]    [TLM]    |
    +-----+---------+-----+------------------------+-----------+----+

          |         |     |                        |           |
          |         |     |                        |           |
          |         |     |                        |           |
          |         |     |                        |       [1 kOhm]
          |         |     |                        |           |
          |         |     |                        |           |
          |         |     |                        v           |
          |         |     |                   +----+-----------+----+
          |         |     |                   |    [GPIO 0]  MPX    |
          |         |     |                   |    [GPIO 9]  Jeti   |         
          |         |     |                   |                     |
          |         |     |                   |     RP2040          |
          |         |     |                   |   KONVERTER         |
          |         |     |                   |                     |
          |         |     |                   |                     |
          |         |     |                   | [5V] [GND] [GPIO 5] |
          |         |     |                   +----+---+---+--------+
          |         |     |                        ^   ^   ^
          |         |     |                        |   |   |
          |         |     |                        |   |   [1 kOhm]
          |         |     |                        |   |   |
          v         v     v                        |   |   |
    +-----+---------+-----+------------------------+---+---+--------+

    |  [RC-SBUS In][+5V] [GND]              [+4,5V] [GND] [UART4 TX]|
    |                                                               |
    |                     FLIGHT CONTROLLER (FC)                    |
    |                             (iNAV)                            |
    +---------------------------------------------------------------+

```text
========================================================================
2. ZULEITUNGEN AM WAVESHARE RP2040-ZERO
========================================================================

                                +-------------------------------------+

                                |             [ USB-C ]               | <-- USB-Anschluss oben
                                +-------------------------------------+
Empfänger [GND] --------------> | [GND]                          [5V] | <--- FC [UART4 +4,5V]

                                | [GND]                         [GND] | <--- FC [UART4 GND]
Empfänger [TLM] <--- [1 kOhm] <-| [GP0] (TLM-Ausgang)           [3V3] | Hott, Multiplex

                                |                                     |
                                | [GP1]                        [GP29] |
                                | [GP2]          +-------+     [GP28] |
                                | [GP3]          |  BOOT |     [GP27] |
                                | [GP4]          +-------+     [GP26] |
FC [UART4 TX] ----> [1 kOhm] -> | [GP5] (LTM-Eingang)          [GP15] |

                                |                                     |
                                | [GP6]          +-------+     [GP14] |
                                | [GP7]          | RESET |      [RX0] | (GP13)
                                |                +-------+      [TX0] | (GP12)
                                |          (RGB) <-- Status-LED       |
                                +-------------------------------------+

                                   |   |   |   |   |   |   |   |   |
                                  GP8 GP9 GP10 GP11 GND 3V3 GP22 GP21 GP20
                                       |
Empfänger [TLM] <--- [1 kOhm] <--------+ Jeti

```



### Strukturierte Leitungsführung & Schutzbeschaltung:
1. **Verbindung zwischen Empfänger und Flight Controller (Hauptstrom & Steuerung):** 
   * **SBUS:** Das Signalkabel führt als direkte, eng geführte Linie vom **SBUS Out** des oberen Empfängers nach unten an den **RC-SBUS In** des Flight Controllers.
   * **Hauptstromversorgung:** Eine durchgehende **+5V**-Leitung und eine gemeinsame **GND**-Masseleitung führen direkt vom Empfänger nach unten zum Flight Controller.

2. **Verbindung zwischen Flight Controller (UART4) und Konverter (Unterseite):** 
   * **LTM-Signal:** Das Kabel führt vom Ausgang **UART4 TX** des Flight Controllers nach oben an den Pin **GPIO 5** des Konverters. In diese Signalleitung ist ein **1-kOhm-Widerstand** zur Strombegrenzung in Reihe einzulöten.
   * **Schnittstellen-Spannung:** Für die Logikversorgung greift an dieser Stelle eine Versorgungsleitung mit **+4,5V** und der dazugehörigen **GND**-Masse vom Flight Controller zu den Eingängen **5V** und **GND** an der Unterseite des Konverters.

3. **Verbindung zwischen Konverter und Empfänger (Oberseite):** 
   * **Multiplex , Hott:** 
   * **Telemetrie-Signal:** Das Kabel führt vom Ausgang **GPIO 0** des Konverters nach oben zurück an den **TLM**-Eingang des Empfängers. In diese Signalleitung ist ein **1-kOhm-Widerstand** zur Schnittstellenentkopplung in Reihe einzulöten.
   * **Jeti:** 
   * **Telemetrie-Signal:** Das Kabel führt vom Ausgang **GPIO 9** des Konverters nach oben zurück an den **TLM**-Eingang des Empfängers. In diese Signalleitung ist ein **1-kOhm-Widerstand** zur Schnittstellenentkopplung in Reihe einzulöten.


   * **Logik-Masse:** Zwischen dem **GND**-Anschluss des Empfängers und dem Konverter wird eine separate, direkte Masseleitung gezogen, um den Potenzialausgleich des Telemetriekreises sicherzustellen.

---

## 2. INAV-EINSTELLUNGEN (FLIGHT CONTROLLER)

Damit Daten aus dem TX4-Pad fließen, müssen im iNav-Configurator folgende Parameter aktiv sein:

1. **Ports-Tab:** Am genutzten Hardware-UART (UART 4) die Option **Telemetry** auf **LTM** stellen (Geschwindigkeit schaltet iNav automatisch fest auf **19200 Baud**).
2. **Configuration-Tab:** Das System-Feature **SOFTSERIAL** muss global **deaktiviert (AUS)** sein, um den stabilen Betrieb über den Hardware-UART zu erzwingen.
3. **CLI-Tab / Abschaltsperren:** iNav blockiert die Telemetrie standardmäßig, wenn kein RC-Empfänger aktiv ist. Um den Datenfluss auch am Tisch ohne Empfänger zu garantieren, müssen folgende Befehle im CLI eingegeben und mit `save` bestätigt werden:
   ```text
   set telemetry_disabled_before_arming = OFF
   save
   ```

---

## 3. DAS SERIAL-KONFIGURATIONSMENÜ

Das integrierte Terminal-Menü wird am PC über die USB-Schnittstelle aufgerufen (**Baudrate: 115200**). Eingaben im Terminal müssen zwingend mit **Leerzeichen um das Gleichheitszeichen** erfolgen.

### Kompakt-Übersicht für die Befehlseingabe (Aufruf mit "?"):

--- INAV TELEMETRIE-KONVERTER BEFEHLE ---
  HINWEIS: Eingaben muessen mit Leerzeichen um das '=' erfolgen (z.B. INAV = 1)

  1. Eingangs-Modus (iNav LTM an GPIO 5):
     INAV = 1           : Schaltet den Konverter-Modus ein
     INAV = 2           : Schaltet den Konverter-Modus mit simulierten Festwerten ein
     INAV = 0           : Deaktiviert den iNav-Eingang

  2. Ausgangs-Protokoll (Auswahl fuer Empfaenger):
     PROTOCOL = M       : Schaltet den Ausgang auf MULTIPLEX um
     PROTOCOL = E       : Schaltet den Ausgang auf JETI (EXBUS) um
     PROTOCOL = H       : Schaltet den Ausgang auf Graupner HoTT um

  3. Ausgangs-Port (Auswahl fuer Empfaenger):
     TLM = 0            : TLM Port für Hott oder MULTIPLEX 
     TLM = 255          : TLM Port für Jeti
     PRI = 255          : Port für Hott oder MULTIPLEX 
     PRI = 9            : Port für Jeti

  3. Einstellungen dauerhaft sichern:
     SAVE               : Speichert alle Parameter im Flash-Speicher des Pico
------------------------------------------

---

[Das originale OPENXSENSOR Projekt](oxs.md)

