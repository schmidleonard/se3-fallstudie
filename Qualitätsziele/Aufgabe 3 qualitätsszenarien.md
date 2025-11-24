# Qualitätsszenarien

## Benutzerfreundlichkeit

### Szenario 1: Verwendungs-Szenario

**Qualitätsmerkmale:** Verständlichkeit, Erlernbarkeit, Fehlertoleranz

Ein neuer Sachbearbeiter soll während des Normalbetriebs (8–18 Uhr) bei seinem ersten Login über die Benutzeroberfläche des Bestandssystems in der Lage sein, einen Neuabschluss korrekt und ohne externe Hilfe durchzuführen, wobei die gesamte Einarbeitungszeit für die fünf Kernprozesse eine Woche nicht überschreiten darf.

---

### Szenario 2: Wachstums-Szenario

**Qualitätsmerkmale:** Verständlichkeit, Erlernbarkeit, Fehlertoleranz

Nach der Integration der neuen Sparte (Hausratversicherung) in der erweiterten Systemversion finden die Nutzer der Fachabteilung die neuen Prozesse auf der um zusätzliche Module erweiterten GUI intuitiv, was durch einen Usability-Test bestätigt wird, bei dem die Verständlichkeit der Oberfläche 80 % oder mehr beträgt.

---

### Szenario 3: Stress Szenario

**Qualitätsmerkmale:** Verständlichkeit, Erlernbarkeit, Fehlertoleranz

Wenn ein Sachbearbeiter im Standardbetrieb in der Eingabemaske zur Adressänderung eine falsche Eingabe in einem Pflichtfeld tätigt, erscheint in weniger als 2 Sekunden eine klare Fehlermeldung, woraufhin der Benutzer seine Eingabe korrigieren und den Vorgang fortsetzen kann, ohne dass der Prozess abbricht oder bereits getätigte Eingaben verloren gehen.

---

## Zuverlässigkeit & Verfügbarkeit

### Szenario 4: 

**Qualitätsmerkmale:** Robustheit, Fehlertoleranz, Wiederherstellbarkeit

Während der normalen Nutzung muss das Vertragsmodul für einen Sachbearbeiter, der einen Vertragsdatensatz aufruft, mit einer Verfügbarkeit von mindestens 99,9 % während der Kernarbeitszeit reagieren und den Datensatz fehlerfrei laden.

### Szenario 5: Wachstums-Szenario
**Qualitätsmerkmale:** Reife, Fehlertoleranz, Wiederherstellbarkeit

Im Rahmen eines Rolling-Updates wird eine neue Version des "Vertrags-Services" in die Private Cloud deployt, während gleichzeitig 500 Sachbearbeiter im System arbeiten. Das System leitet die Anfragen ohne Unterbrechung auf die neuen Instanzen um, sodass keine Downtime für den Endanwender spürbar ist (Zero-Downtime Deployment).

### Szenario 6: Stress-Szenario
**Qualitätsmerkmale:** Robustheit, Fehlertoleranz

Fällt am Monatsende (Spitzenlast) eine Instanz des Beitragsberechnungs-Services aufgrund eines Hardwarefehlers aus, erkennt die Container-Orchestrierung (Kubernetes) dies sofort. Das System startet automatisch eine neue Instanz, und die laufenden Transaktionen werden wiederholt, sodass der Batch-Lauf zur Beitragsberechnung trotzdem bis 06:00 Uhr morgens erfolgreich abgeschlossen wird.

---

## Sicherheit & Compliance

### Szenario 7: Verwendungs-Szenario
**Qualitätsmerkmale:** Vertraulichkeit, Integrität, Nachweisbarkeit

Ein Wirtschaftsprüfer fordert im Rahmen des Jahresabschlusses einen Nachweis über alle Änderungen an einem spezifischen Vertrag vom 12.03. des Vorjahres an. Das System generiert über den Audit Trail innerhalb von 5 Minuten einen manipulationssicheren Bericht, der zeigt, welcher Sachbearbeiter wann welche Änderung vorgenommen hat.

### Szenario 8: Wachstums-Szenario
**Qualitätsmerkmale:** Konformität, Integrität

Die Aufbewahrungsfrist für gekündigte Verträge läuft nach 10 Jahren ab (DSGVO). Obwohl das Datenvolumen auf 10 TB angewachsen ist, identifiziert und löscht (bzw. anonymisiert) das System im nächtlichen Wartungslauf automatisch alle betroffenen Datensätze zuverlässig, ohne die Performance des laufenden Betriebs am Folgetag zu beeinträchtigen.

### Szenario 9: Stress-Szenario
**Qualitätsmerkmale:** Vertraulichkeit, Authentizität

Ein Angreifer versucht, durch einen Brute-Force-Angriff (1000 Login-Versuche pro Minute) Zugriff auf das Endkunden-Portal zu erlangen. Das Identity & Access Management (IAM) erkennt das Muster innerhalb von 10 Sekunden, sperrt die IP-Adresse und alarmiert das Security-Team, während legitime Kunden sich weiterhin über SSO (Single Sign-On) einloggen können.

---

## Performance & Skalierbarkeit

### Szenario 10: Verwendungs-Szenario
**Qualitätsmerkmale:** Zeitverhalten, Antwortzeit

Ein Sachbearbeiter sucht während eines telefonischen Kundenkontakts nach einer Vertragsnummer. Nach Eingabe der Nummer werden die Vertragsdetails und der aktuelle Status in unter 2 Sekunden vollständig auf der Oberfläche angezeigt, um eine flüssige Gesprächsführung zu ermöglichen.

### Szenario 11: Wachstums-Szenario
**Qualitätsmerkmale:** Durchsatz, Skalierbarkeit

Nach der Übernahme der Unfallversicherungssparte erhöht sich die Anzahl der Verträge um 500.000. Der monatliche Beitragslauf, der die Rechnungen an SAP übergibt, skaliert horizontal und benötigt für die Verarbeitung der zusätzlichen Daten maximal 15 % mehr Zeit als vor der Migration, womit das Zeitfenster (Nachtlauf) eingehalten bleibt.

### Szenario 12: Stress-Szenario
**Qualitätsmerkmale:** Ressourcenverbrauch, Kapazität

Nach einem schweren Unwetterereignis melden sich 5.000 Kunden gleichzeitig über die App und das Portal, um Schäden zu melden (Lastspitze 5x höher als Normalbetrieb). Das System skaliert die Microservices für die Schadenannahme automatisch hoch, sodass die Reaktionszeit der API für die App konstant unter 3 Sekunden bleibt und keine Meldung verloren geht.

---

## Wartbarkeit & Erweiterbarkeit

### Szenario 13: Verwendungs-Szenario
**Qualitätsmerkmale:** Modifizierbarkeit, Testbarkeit

Ein Entwicklerteam muss einen kritischen Bug in der Rabatt-Berechnung beheben. Aufgrund der hohen Testabdeckung und der entkoppelten Architektur kann der Fix implementiert, durch die CI/CD-Pipeline getestet und innerhalb von 4 Stunden in die Produktion gebracht werden.

### Szenario 14: Wachstums-Szenario
**Qualitätsmerkmale:** Erweiterbarkeit, Anpassbarkeit

Das Marketing möchte eine neue Tarifgeneration für die Haftpflichtversicherung einführen. Ein Business-Analyst kann die neuen Tarifparameter (neue Risikoklassen) über das Produktsystem konfigurieren und aktivieren, sodass diese innerhalb von 2 Tagen für den Vertrieb verfügbar sind, ohne dass Core-Code im Bestandssystem angepasst werden muss.

### Szenario 15: Stress-Szenario
**Qualitätsmerkmale:** Analysierbarkeit, Modifizierbarkeit

Ein neuer Entwickler (Junior) kommt in das Team "Partnerverwaltung". Dank der klaren Domänenstruktur (DDD) und der dokumentierten API-Schnittstellen ist er in der Lage, innerhalb der ersten 2 Wochen ein neues Feature (z.B. ein neues Adressfeld) im Partner-Service zu implementieren und erfolgreich zu deployen, ohne Seiteneffekte im Vertrags-Service auszulösen.




