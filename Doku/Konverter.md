
## Zur Konfiguration des Konverters über USB wird das Programm Coolterm empfohlen.

1.) Konverter per USB mit einem Laptop verbinden.

2.) Coolterm starten.

3.) Nach Drücken der Enter Taste erschein das orginale openXsensor on RP 2040 Menü.

## Folgende Einstellungen sind hilfreich:

FV + Enter : Anzeigen der Empfangenen Telemetriewerte

inav = 2   : Senden von Telemetriewerten zum Empfänger ohne INAV

?          : Hielfefunktion

### Befehlsübersich der Einstellungen für INAV

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
klassischen Systemen zu permanenten Checksummenfehlern führt.

