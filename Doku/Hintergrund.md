
## DER STRATEGISCHE HINTERGRUND

Die direkte Nutzung des Jeti EXBUS-Protokolls in der iNav-Firmware führt zu Instabilitäten. Durch Fehler im iNav-Quellcode und Inkompatibilitäten bei der Kompilierung mit neueren Compiler-Versionen kommt es bei aktiver EXBUS-Verbindung regelmäßig zu Systemausfällen (Freezes) des Flight Controllers. Diese treten vor allem bei schnellen Steuerbewegungen oder bei der Übertragung von mehr als 16 Kanälen auf. Zudem ist die Rückrichtung für Telemetriedaten fehlerhaft implementiert, was zu einfrierenden oder fehlenden Werten auf dem Sender führt. Multiplex-Protokolle werden von iNav nicht nativ unterstützt.

